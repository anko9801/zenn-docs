---
title: "malloc.c を読む (arena)"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CTF", "pwn", "Linux"]
published: false
---

`malloc()` でヒープ領域にあるメモリを確保してそのポインタを返し、`free()` はそのポインタのメモリを開放してくれます。

```c
void *malloc(size_t size);
void free(void *ptr);
```

このシリーズではこれらの関数が内部でどのように処理されるのかを調べていきます。

- [malloc.c を読む (malloc / free)](https://zenn.dev/anko/articles/malloc-malloc-free)
- [malloc.c を読む (bins)](https://zenn.dev/anko/articles/malloc-each-bins)

今回はアリーナの処理を中心に調べていきます。

シリーズは malloc.c を題材としていますがここでは arena.c を主に読むことになります。

ここで扱う glibc のバージョンは v2.38 です。また glibc のソースコードはブラウザ上で読むことができます。

- https://elixir.bootlin.com/glibc/latest/source/malloc/malloc.c
- https://codebrowser.dev/glibc/glibc/malloc/malloc.c.html

## アリーナとは
今まで見てきたようにヒープ領域ではデータをチャンクという単位で管理しており、そのチャンクを bin が管理していました。その bin を管理し、メモリプール全体を扱う機構がアリーナです。

アリーナの実体は `malloc_state` 構造体として定義されます。
main arena は次のように定義されています。

```c
struct malloc_state
{
  __libc_lock_define (, mutex);     // arena へのアクセスを serialize する
  int flags;                        // ヒープメモリが連続であるか

  int have_fastchunks;              // fastbins が空ではないことを表す真偽値
  mfastbinptr fastbinsY[NFASTBINS]; // fastbins の先頭チャンクへのポインタ

  mchunkptr top;                    // ヒープ領域の最後にある未使用の大きなチャンク
  mchunkptr last_remainder;         // 分割して確保した際に余った領域の最新のチャンク

  mchunkptr bins[NBINS * 2 - 2];    // unsortedbin smallbins largebins の先頭・末尾
  unsigned int binmap[BINMAPSIZE];  // これらを素早く見つける為に使われるビットベクタ

  struct malloc_state *next;        // arena の単方向リスト
  struct malloc_state *next_free;   // 使われていない arena の単方向リスト
  INTERNAL_SIZE_T attached_threads; // arena にアクセスしているスレッドの数

  INTERNAL_SIZE_T system_mem;       // arena によって現在確保されているメモリの合計値
  INTERNAL_SIZE_T max_system_mem;   // system_mem の最大値
};

static struct malloc_state main_arena =
{
  .mutex = _LIBC_LOCK_INITIALIZER,
  .next = &main_arena,
  .attached_threads = 1
};
```

## 各フィールドに関連する処理
### flags
アリーナの状態に関するフラグです。使うビットは `NONCONTIGUOS_BIT == 2` の 1 つだけだと思います。これは `MORECORE` したときに連続した領域を返すことを保証しないことを表すフラグです。フラグがクリアされているときは連続性をとことん利用して top chunk を大きくします。初期値は `MORECORE_CONTIGUOUS` から書き込まれ、mmap で拡張するとフラグが立ちます。

ちなみにこのフィールドは前に `max_fast` として、各アリーナにおける fastbins が扱うチャンクサイズのパラメータに使われていました。

### top
top chunk とは最上位つまり終端に置かれた特別なチャンクでどの bin にも含まれていません。

これは `malloc()` では他に割り当てられるチャンクがない場合にのみ切り出され、top chunk が肥大化した場合は `free()` 時に OS に返されます。top chunk は最初、`initiali_top` と呼ばれるサイズがゼロの bin を指しているので、最初の `malloc()` で拡張します。

```c
/* Conveniently, the unsorted bin can be used as dummy top on first call */
#define initial_top(M)              (unsorted_chunks (M))
```

### last_remainder
smallbins / largebins においてチャンクの分割を行ったときに残りのチャンク (remainder) を `last_remainder` に格納します。ただし残りが smallbins の大きさとなったチャンクしか扱わない。

```c
              /* Split */
              else
                {
                  remainder = chunk_at_offset (victim, nb);

                  /* We cannot assume the unsorted list is empty and therefore
                     have to perform a complete insert here.  */
                  bck = unsorted_chunks (av);
                  fwd = bck->fd;
		  if (__glibc_unlikely (fwd->bk != bck))
		    malloc_printerr ("malloc(): corrupted unsorted chunks 2");
                  remainder->bk = bck;
                  remainder->fd = fwd;
                  bck->fd = remainder;
                  fwd->bk = remainder;

                  /* advertise as last remainder */
                  if (in_smallbin_range (nb))
                    av->last_remainder = remainder;
                  if (!in_smallbin_range (remainder_size))
                    {
                      remainder->fd_nextsize = NULL;
                      remainder->bk_nextsize = NULL;
                    }
                  set_head (victim, nb | PREV_INUSE |
                            (av != &main_arena ? NON_MAIN_ARENA : 0));
                  set_head (remainder, remainder_size | PREV_INUSE);
                  set_foot (remainder, remainder_size);
                }
```


```c
/*
   Initialize a malloc_state struct.

   This is called from ptmalloc_init () or from _int_new_arena ()
   when creating a new arena.
 */

static void
malloc_init_state (mstate av)
{
  int i;
  mbinptr bin;

  /* Establish circular links for normal bins */
  for (i = 1; i < NBINS; ++i)
    {
      bin = bin_at (av, i);
      bin->fd = bin->bk = bin;
    }

#if MORECORE_CONTIGUOUS
  if (av != &main_arena)
#endif
  set_noncontiguous (av);
  if (av == &main_arena)
    set_max_fast (DEFAULT_MXFAST);
  atomic_store_relaxed (&av->have_fastchunks, false);

  av->top = initial_top (av);
}
```


## その他のパラメータ
```c
struct malloc_par
{
  /* Tunable parameters */
  unsigned long trim_threshold;   // trim する閾値
  INTERNAL_SIZE_T top_pad;        // アリーナを拡張するときのサイズ算出に使用
  INTERNAL_SIZE_T mmap_threshold; // 既にある領域を使わず mmap munmap を用いて管理するサイズの閾値
  INTERNAL_SIZE_T arena_test;
  INTERNAL_SIZE_T arena_max;

  INTERNAL_SIZE_T thp_pagesize;   // Transparent Huge Page のページサイズ
  INTERNAL_SIZE_T hp_pagesize;    // 通常のページサイズで mmap はこれに align される
  int hp_flags;

  int n_mmaps;                    // mmap した回数
  int n_mmaps_max;                // mmap できる上限
  int max_n_mmaps;                // これまでの n_mmaps の最大値
  int no_dyn_threshold;           // mmap_threshold の設定時に動的な変更をやめる為のフラグ

  INTERNAL_SIZE_T mmapped_mem;    // mmap() で確保した領域サイズの合計
  INTERNAL_SIZE_T max_mmapped_mem; // これまでの mmaped_mem の最大値

  char *sbrk_base;                // ヒープ領域のベースアドレス

  size_t tcache_bins;             // tcache bins の総数
  size_t tcache_max_bytes;        // 最大チャンクサイズ
  size_t tcache_count;            // 各 tcache bin が持てる最大のチャンク数
  size_t tcache_unsorted_limit;   // unsortedbin が持てる最大のチャンク数
};

/*
    DEFAULT_MXFAST             64 (for 32bit), 128 (for 64bit)
    DEFAULT_TRIM_THRESHOLD     128 * 1024
    DEFAULT_TOP_PAD            0
    DEFAULT_MMAP_THRESHOLD     128 * 1024
    DEFAULT_MMAP_MAX           65536
*/
static struct malloc_par mp_ =
{
  .top_pad = DEFAULT_TOP_PAD,
  .n_mmaps_max = DEFAULT_MMAP_MAX,
  .mmap_threshold = DEFAULT_MMAP_THRESHOLD,
  .trim_threshold = DEFAULT_TRIM_THRESHOLD,
#define NARENAS_FROM_NCORES(n) ((n) * (sizeof (long) == 4 ? 2 : 8))
  .arena_test = NARENAS_FROM_NCORES (1),
  .tcache_count = TCACHE_FILL_COUNT,
  .tcache_bins = TCACHE_MAX_BINS,
  .tcache_max_bytes = tidx2usize (TCACHE_MAX_BINS-1),
  .tcache_unsorted_limit = 0 /* No limit.  */
};
```