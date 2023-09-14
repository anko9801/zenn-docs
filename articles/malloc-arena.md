---
title: "malloc.c を読む (arena)"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CTF", "pwn", "Linux"]
published: false
---

`malloc()` は動的にメモリを確保して `free()` で開放してくれます。

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
プロセスは `sbrk()` や `mmap()` システムコールによって OS からメモリプールを獲得します。このメモリプールをアプリケーションに高速に分配する bin と呼ばれる機構があります。bin はメモリプールとは別にメモリプールの管理部で管理されています。そしてメモリプールとその管理部を合わせてアリーナ (arena) と呼びます。

アリーナの管理部は `malloc_state` 構造体に書かれてあります。初期状態ではヒープ領域にサイズゼロのメモリプールとグローバル領域に `main_arena` と呼ばれる管理部があります。

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

これは `malloc()` では他に割り当てられるチャンクがない場合にのみ切り出され、top chunk が肥大化した場合は `free()` 時に自動的に OS に返されます。top chunk は最初 `initiali_top` と呼ばれるサイズがゼロの bin を指しているので `malloc()` を呼ぶことで `sbrk()` が呼ばれてヒープ領域が拡張され、top chunk も拡張されます。

```c
/* Conveniently, the unsorted bin can be used as dummy top on first call */
#define initial_top(M)              (unsorted_chunks (M))

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

### last_remainder
smallbins / largebins / last_remainder においてチャンクの分割を行ったときの残りのチャンク (remainder) を `last_remainder` に格納して参照局所性を活かします。ただし残りが smallbins の大きさとなったチャンクしか扱いません。

### sbrk / mmap

```c
  INTERNAL_SIZE_T system_mem;       // arena によって現在確保されているメモリの合計値
  INTERNAL_SIZE_T max_system_mem;   // system_mem の最大値
```

### マルチスレッド
アリーナは `mutex` を用いてロックし、複数のスレッドがメモリプールを共有することができます。

ただし、あるスレッドがチャンクを確保しようとしたときに `main_arena` がロックされていた場合、`mmap()` で新たなヒープ領域を作ります。この為、最終的にヒープ領域の数はスレッド数くらいに落ち着きます。

`mmap()` で得られたヒープ領域では先頭に管理情報の実体 `heap_info` と管理部の実体 `malloc_state` を置き、各ヒープ領域は `malloc_state.next` `_heap_info.prev` によって双方向リストが作られます。そしてスレッドが終了したときには `next_free` に繋がれます。ちなみに `main_arena` では `heap_info` に相当するものはありません。

```c
typedef struct _heap_info
{
  mstate ar_ptr;            // このヒープ領域のアリーナへのポインタ
  struct _heap_info *prev;  // 前のヒープ領域
  size_t size;              // アリーナのバイト数
  size_t mprotect_size;     // PROT_READ|PROT_WRITE で mprotect されたバイト数
  size_t pagesize;          // ページサイズ
  char pad[-3 * SIZE_SZ & MALLOC_ALIGN_MASK]; // メモリ境界を揃える為のパディング
} heap_info;
```

また `mmap()` で得られたヒープ領域は 1MB でアラインされているため、どんなチャンクのアドレスでも先頭 5 nibbles をクリアすることで `_heap_info` `malloc_state` を見つけてアリーナの管理部にアクセスできます。ちなみに `main_arena` のチャンクはそれは出来ず、直接アクセスする他ありません。チャンクにこの違いを教える為にチャンクには `NON_MAIN_ARENA` というフラグがあるという訳です。

### アリーナのパラメータ
アリーナに関するパラメータは `malloc_par` 構造体に書かれてあります。

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

## まとめ
チャンクから始まり bins、アリーナまで紹介しました。これで malloc に関しては完璧なんじゃないでしょうか。

これで malloc.c の読書は一旦終わりです。気が向いたら詳細についてや他の malloc 関数についても紹介したいと思います。
