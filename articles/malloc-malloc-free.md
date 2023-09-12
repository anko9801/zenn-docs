---
title: "malloc.c を読む (malloc, free)"
emoji: "😊"
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

## malloc / free 全体像
まずはざっくりと！

読むべき関数は malloc.c 内の次の関数です。

| 関数名 | 説明 |
| --- | --- |
| `__libc_malloc()` | `malloc()` のエイリアス。主に tcache の処理を行う。 |
| `_int_malloc()` | malloc のほぼ全てが書かれてある。最も重要！ |
| `sysmalloc()` | その他 mmap など |
| `malloc_consolidate()` | fastbins の統合を行う。 |
| `__libc_free()` | `free()` のエイリアス。 |

### malloc() の擬似コード
実際のコードは約 6000 行と結構長いのでまずは擬似コードを読んで各 bins の役割を考えてみましょう。

```c
void *__libc_malloc (size_t bytes) {
    if (malloc の初期化をしてない)
        ptmalloc_init()
    if (tcache を初期化してない)
        tcache_init()

    if (tcache のサイズ)
        tcache から確保
    if (シングルスレッド)
        _int_malloc(&main_arena, bytes) を返す
    アリーナを mutex ロックして取得
    _int_malloc (arena_ptr, bytes) を返す
    失敗したら 1 回だけリトライする
}


static void *_int_malloc(mstate arena, size_t bytes) {
    if (arena がない)
        sysmalloc()
    if (global_max_fast 以下) {
        fastbins から確保
        他にアクセスしたチャンクを tcache に挿入する
        確保成功したらそれを返す
    }
    if (smallbins のサイズ) {
        smallbins から確保
        他にアクセスしたチャンクを tcache に挿入する
        確保成功したらそれを返す
    } else {
        malloc_consolidate()
    }

    for (;;) {
        iter = 0
        while (unsortedbin の末尾から取得) {
            if (smallbins のサイズ && unsortedbin に last_remainder の 1 つしかない && last_remainder 以下のサイズ)
                last_remainder から切り出す

            unsortedbin から unlink
            if (同じサイズ) {
                if (tcahce が埋まってない) {
                    tcache に挿入して continue
                } else {
                    確保して返す
                }
            }
            smallbins, largebins に振り分け

            if (++tcache_unsorted_count > tcache_unsorted_limit)
                tcache から確保
            if (++iters >= 10000)
                break;
        }
        if (tcache に挿入した)
            tcache から確保

        smallbins, largebins から確保

        if (top chunk size >= nb + MINSIZE) {
            top chunk から切り出す
        } else if (have_fastchunks) {
            malloc_consolidate()
        } else {
            sysmalloc()
        }
    }
}


static void *sysmalloc (INTERNAL_SIZE_T nb, mstate av) {
    if (アリーナがない || ある程度大きく mmap できる) {
        mmap して確保
    }

    if (main_arena ではない) {
        mmap でヒープを拡張
        mmap で新しいヒープを作る
        mmap で確保
    } else {
        sbrk でヒープを拡張
        アドレス空間の穴を mmap で確保
    }
    確保したヒープの top chunk を切り出す
}
```

smallbin の範囲のとき
fastbin → smallbin → unsortedbin → 上位の bin → top → fastbin consolidate → unsortedbin
largebin の範囲のとき
fastbin consolidate → unsortedbin → largebin → 上位の bin → top → sysmalloc()

### free() の擬似コード

長い関数は inline 化して高速化したいのかと思います。

```c
void __libc_free(void *mem) {
    if (mmap されたチャンク) {
        munmap
    } else {
        _int_free()
    }
}


static void _int_free (mstate av, mchunkptr p, int have_lock) {
    tcache
    if (get_max_fast 以下) {
        fastbins
    }
    tcache から確保
}
```

