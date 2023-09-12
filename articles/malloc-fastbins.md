---
title: "malloc.c を読む (fastbins)"
emoji: "👏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

`malloc` 関数でヒープ領域にあるメモリを確保してそのポインタを返す。 `free` 関数はそのポインタのメモリを開放してくれる。

```c
void *malloc(size_t size);
void free(void *ptr);
```

これらの関数が内部でどのように処理されるのかを調べていく。

[**最新の malloc.c のソースコード**](https://elixir.bootlin.com/glibc/latest/source/malloc/malloc.c) を用意しましょう。

今回は malloc 関数の中身を中心に調べていく。

glibc のバージョンは v2.38 です。

### fastbins とは
最近解放された小さなチャンクを保持している配列です。
単方向リスト

consolidate 統合された状態が保たれるようにする

MORECORE sbrk

| パラメータ | 説明 |
| --- | --- |
| `FASTBIN_CONSOLIDATION_THRESHOLD` | `FASTBIN_CONSOLIDATION_THRESHOLD` のチャンクが free されたときに自動的に周辺にある可能性のある fastbins の consolidation を行う。It is defined at half the default trim threshold as a compromise heuristic to only attempt consolidation if it is likely to lead to trimming. However, it is not dynamically tunable, since consolidation reduces fragmentation surrounding large chunks even if trimming is not used. |
| `NONCONTIGUOS_BIT` | NONCONTIGUOUS_BIT indicates that MORECORE does not return contiguous regions.  Otherwise, contiguity is exploited in merging together, when possible, results from consecutive MORECORE calls. The initial value comes from MORECORE_CONTIGUOUS, but is changed dynamically if mmap is ever used as an sbrk substitute. |
| `TRIM_FASTBINS` | 小さなチャンクも trim メモリ少なくなる代わりに処理が遅くなる TRIM_FASTBINS controls whether free() of a very small chunk can immediately lead to trimming. Setting to true (1) can reduce memory footprint, but will almost always slow down programs that use a lot of small chunks. Define this only if you are willing to give up some speed to more aggressively reduce system-level memory footprint when releasing memory in programs that use many small chunks.  You can get essentially the same effect by setting MXFAST to 0, but this can lead to even greater slowdowns in programs using many small chunks. TRIM_FASTBINS is an in-between compile-time option, that disables only those chunks bordering topmost memory from being placed in fastbins. |

malloc_consolidate
`global_max_fast`

```
An array of lists holding recently freed small chunks.  Fastbins are not doubly linked.  It is faster to single-link them, and since chunks are never removed from the middles of these lists, double linking is not necessary. Also, unlike regular bins, they are not even processed in FIFO order (they use faster LIFO) since ordering doesn't much matter in the transient contexts in which fastbins are normally used.

Chunks in fastbins keep their inuse bit set, so they cannot be consolidated with other free chunks. malloc_consolidate releases all chunks in fastbins and consolidates them with other free chunks.
```


```c
typedef struct malloc_chunk *mfastbinptr;
#define fastbin(ar_ptr, idx) ((ar_ptr)->fastbinsY[idx])

/* offset 2 to use otherwise unindexable first 2 bins */
#define fastbin_index(sz) \
  ((((unsigned int) (sz)) >> (SIZE_SZ == 8 ? 4 : 3)) - 2)


/* The maximum fastbin request size we support */
#define MAX_FAST_SIZE     (80 * SIZE_SZ / 4)

#define NFASTBINS  (fastbin_index (request2size (MAX_FAST_SIZE)) + 1)

#define FASTBIN_CONSOLIDATION_THRESHOLD  (65536UL)

#define NONCONTIGUOUS_BIT     (2U)

#define contiguous(M)          (((M)->flags & NONCONTIGUOUS_BIT) == 0)
#define noncontiguous(M)       (((M)->flags & NONCONTIGUOUS_BIT) != 0)
#define set_noncontiguous(M)   ((M)->flags |= NONCONTIGUOUS_BIT)
#define set_contiguous(M)      ((M)->flags &= ~NONCONTIGUOUS_BIT)

/* Maximum size of memory handled in fastbins.  */
static uint8_t global_max_fast;

/*
   Set value of max_fast.
   Use impossibly small value if 0.
   Precondition: there are no existing fastbin chunks in the main arena.
   Since do_check_malloc_state () checks this, we call malloc_consolidate ()
   before changing max_fast.  Note other arenas will leak their fast bin
   entries if max_fast is reduced.
 */

#define set_max_fast(s) \
  global_max_fast = (((size_t) (s) <= MALLOC_ALIGN_MASK - SIZE_SZ)	\
                     ? MIN_CHUNK_SIZE / 2 : ((s + SIZE_SZ) & ~MALLOC_ALIGN_MASK))

static inline INTERNAL_SIZE_T
get_max_fast (void)
{
  /* Tell the GCC optimizers that global_max_fast is never larger
     than MAX_FAST_SIZE.  This avoids out-of-bounds array accesses in
     _int_malloc after constant propagation of the size parameter.
     (The code never executes because malloc preserves the
     global_max_fast invariant, but the optimizers may not recognize
     this.)  */
  if (global_max_fast > MAX_FAST_SIZE)
    __builtin_unreachable ();
  return global_max_fast;
}

#ifndef TRIM_FASTBINS
#define TRIM_FASTBINS  0
#endif
```

### malloc

まずは `_int_malloc()` での fastbins の処理は

If the size qualifies as a fastbin, first check corresponding bin. This code is safe to execute even if av is not yet initialized, so we can try it without checking, which saves some time on this fast path.

最も


```c:L1972
/* ------------------ Testing support ----------------------------------*/

static int perturb_byte;

static void
alloc_perturb (char *p, size_t n)
{
  if (__glibc_unlikely (perturb_byte))
    memset (p, perturb_byte ^ 0xff, n);
}

static void
free_perturb (char *p, size_t n)
{
  if (__glibc_unlikely (perturb_byte))
    memset (p, perturb_byte, n);
}
```