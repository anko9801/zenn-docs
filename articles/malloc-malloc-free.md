---
title: "malloc.c を読む (malloc / free)"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CTF", "pwn", "Linux"]
published: true
---

`malloc()` は動的にメモリを確保して `free()` で開放してくれます。

```c
void *malloc(size_t size);
void free(void *ptr);
```

このシリーズではこれらの関数が内部でどのように処理されるのかを調べていきます。

- [malloc.c を読む (malloc / free)](https://zenn.dev/anko/articles/malloc-malloc-free)
- [malloc.c を読む (bins)](https://zenn.dev/anko/articles/malloc-each-bins)

今回は `malloc()` `free()` の全体像を紹介します。

注意としてここでの目的は全体を俯瞰して、詳細を詰めずとも各 bins の役割を理解し、攻撃手法を理解できるようにすることです。それに合わないマルチスレッドや最適化などにおける緻密なトリックやコーナーケースなどは暗黙的に実装されていると仮定します。その詳細についてはソースコードや他の資料を参考にしていただきたいです。

ここで扱う glibc のバージョンは v2.38 です。また glibc のソースコードはブラウザ上で読むことができます。

- https://elixir.bootlin.com/glibc/latest/source/malloc/malloc.c
- https://codebrowser.dev/glibc/glibc/malloc/arena.c.html

## malloc / free 全体像
まずはざっくりと！

`malloc()` `free()` ではプロセスのアドレス空間のうちヒープ領域を利用しています。まず `sbrk()` や `mmap()` システムコールによって OS からメモリプールを貰います。このメモリプールを効率的に分配する機構を bin と呼びます。それにはいくつかの種類があります。

| bins の種類 | チャンクサイズ (default) | 説明 | データ構造 |
| --- | --- | --- | --- |
| tcache bins | 0x20 ~ 0x410 | 最近アクセスしたチャンクが入れられる bins | 単方向リスト |
| fastbins | 0x20 ~ 0x80 | 頻繁に確保・解放されるような小さなチャンクを管理する bins | 単方向リスト |
| unsortedbin | 0x20 ~ | tcache bins や fastbins では扱えないものを入れて smallbins や largebins に渡す前に一時的に管理する bin | 双方向リスト |
| smallbins | 0x20 ~ 0x3f0 | 小さなチャンクを管理する bins | 双方向リスト |
| largebins | 0x400 ~ | 大きなチャンクを管理する bins | 双方向リスト + スキップリスト |

詳細はこのシリーズの bins において解説します。今はそういうものがあるんだと思っておいてください。

そして `malloc()` `free()` を理解するために読むべき関数は malloc.c 内の次の関数です。

| 関数名 | 説明 |
| --- | --- |
| `__libc_malloc()` | `malloc()` のエイリアス。主に tcache の処理を行う。 |
| `__libc_free()` | `free()` のエイリアス。 |
| `_int_malloc()` | malloc のほぼ全てが書かれてある。最も重要！ |
| `_int_free()` | free のほぼ全てが書かれてある。重要 of 重要！ |
| `sysmalloc()` | OS からメモリを貰う。mmap や MORECORE (sbrk) でヒープ領域を確保や拡張する。 |
| `systrim()` | OS にメモリを返す。top chunk から切り出して返す。 |
| `malloc_consolidate()` | fastbins の統合を行う。 |

実際の malloc.c は約 6000 行と結構長く、高速化の為にあまり抽象化されてないのでまずは擬似コードを読むことにしましょう。

また次の用語がよく出てくるのでここでまとめておきます。

| 用語 | 説明 |
| --- | --- |
| Remaindering | 要求のあったサイズと合致するチャンクが見つからない場合、既存のチャンクを要求サイズで分割する場合があります。残りの領域は新たなチャンクとして登録され、その内最新のものはアリーナの `last_remainder` に格納される。 |
| Exhausting | 要求のあったサイズ `N` と合致するチャンクが見つからず、サイズ `N + 0x10` のチャンクがある場合、分割せずにそのまま使うこと。 |
| Consolidation | 解放されたチャンクが隣り合っていたときに 2 つのチャンクを統合すること。これによってフラグメンテーションを防ぐ。 |
| フラグメンテーション | `malloc()` `free()` を繰り返すことで未使用のチャンクがバラバラになってしまうこと。 |

### malloc() の擬似コード
擬似コードを読んで各 bins や処理などの役割をよくよく考えてみましょう。

```c:malloc()
void *__libc_malloc (size_t bytes) {
    if (malloc の初期化をしてない)
        ptmalloc_init()
    if (tcache を初期化してない)
        tcache_init()

    if (tcache のサイズ)
        tcache から確保
    if (シングルスレッド)
        main_arena で _int_malloc()
    アリーナをロックして取得し _int_malloc()
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
            if (smallbins のサイズ
             && unsortedbin に last_remainder の 1 つしかない
             && last_remainder 以下のサイズ)
                last_remainder から切り出す

            unsortedbin から unlink
            if (同じサイズ) {
                if (tcahce が埋まってない)
                    unsortedbin から tcache に挿入して continue
                else
                    unsortedbin から確保
            }
            unsortedbin を smallbins, largebins に振り分け

            if (++tcache_unsorted_count > tcache_unsorted_limit)
                tcache から確保
            if (++iters >= 10000)
                break
        }
        tcache に挿入していたらそこから確保

        smallbins / largebins から best-fit で確保

        if (top chunk が要求サイズ以上)
            top chunk から切り出す
        else if (have_fastchunks)
            malloc_consolidate()
        else
            sysmalloc()
    }
}
```

これらを読むとよりマクロに考えれば次のような処理となります。

```mermaid
graph LR
    A(tcache bins) --> B(fastbins)
    B --> |smallbins| C(smallbins)
    B --> |largebins| D(fastbins の統合)
    C --> E(unsortedbin)
    D --> E
    E --> F(smallbins\nlargebins)
    F --> G(top chunk)
    G --> |fastbins| H(fastbins の統合)
    G --> I(sysmalloc)
    H --> E
```

こうして見ると大枠として smallbins / largebins から best-fit で割り当てて、足りなくなったら top chunk の切り出したり `sysmalloc()` で拡張します。それに加え、小さなチャンクを扱う fastbins では頻繁に確保・解放が行われ、 tcache bins や unsortedbin は局所参照性を高めるキャッシュ機構として機能します。

次に `sysmalloc()` を読んでいきましょう。

```c
static void *sysmalloc (INTERNAL_SIZE_T nb, mstate av) {
    if (アリーナがない
     || mmap_threshold (0x20000) 以上で mmap 回数が n_mmaps_max 回未満)
        sysmalloc_mmap()

    assert (初期状態 ||
            top chunk の前が使用中
         && top chunk サイズが MINSIZE 以上 nb + MINSIZE 以下
         && top chunk の末尾がページ境界)

    if (main_arena ではない) {
        if (mmap でヒープ拡張) {
            top chunk の拡張
        } else if (新しいヒープ領域を mmap で作成) {
            新しい top chunk を確保したヒープ領域にする
            古い top chunk の末尾に 1 つ fencepost を作って PREV_INUSE をマーク
            古い top chunk を _int_free()
        }
    } else {
        sbrk でヒープ拡張または mmap でヒープ領域を確保
        if (確保成功) {
            if (top chunk に隣接している)
                top chunk の拡張
            else {
                if (アリーナが連続な領域しか扱わない) {
                    MALLOC_ALIGNMENT (0x10) の alignment 調整
                    出来たら 2 度目の sbrk でページ境界まで伸ばす
                } else {
                    MALLOC_ALIGNMENT (0x10) の alignment 調整
                }
                新しい top chunk を確保したヒープにする
                古い top chunk の末尾に 2 つ fencepost を作って PREV_INUSE をマーク
                古い top chunk を _int_free()
            }
        }
    }
    top chunk から切り出す
}
```

このように mmap / sbrk を駆使してヒープ領域を獲得しています。House of Corrosion では top chunk の末尾がページ境界にあることを注意しないといけません。

### free() の擬似コード

`free()` についても思いを馳せてみましょう。

```c:free()
void __libc_free(void *mem) {
    if (mmap されたチャンク)
        munmap
    else
        _int_free()
}


static void _int_free (mstate av, mchunkptr p, int have_lock) {
    if (tcache bins の範囲内 && まだ埋まっていない)
        tcache bins に挿入
    if (global_max_fast 以下)
        fastbins に挿入

    if (!IS_MMAPED) {
        if (前のチャンクが free chunk)
            前のチャンクと統合して unlink
        if (次のチャンクが top chunk ではない) {
            if (次のチャンクが free chunk)
                次のチャンクと統合して unlink
            else
                次のチャンクの PREV_INUSE をクリアする

            unsortedbin に挿入
        } else {
            top chunk に統合する
        }

        if (統合後 FASTBIN_CONSOLIDATION_THRESHOLD (0x10000) 以上) {
            if (have_fastchunks)
                malloc_consolidate()
            if (top chunk が閾値 (0x20000) より大きい)
                systrim() または heap_trim()
        }
    } else {
        munmap
    }
}
```

`free()` では的確に統合して tcache bins, fastbins, unsortedbin に挿入します。よく見ると smallbins や largebins は関与してません。またある程度ヒープ領域が拡張されて top chunk が肥大化すると `free()` で無駄と判断して `systrim()` でヒープ領域を削減します。

## まとめ
`malloc()` `free()` の大枠は理解できたかと思います。これらの関数のより詳細については記事より直接ソースコードを読んで自分でまとめた方が絶対に良い理解と読解力が得られると思います。そしてソースコードを読み込んだ方はより良い記事を書いてほしいです！
