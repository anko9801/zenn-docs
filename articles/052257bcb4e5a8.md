---
title: "【CTF 探訪記】セキュリティ機構"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CTF", "pwn"]
published: false
---

NX, ASLR, RELRO, PIE, Stack Canary などのセキュリティ機構が ELF ファイルで指定されているかを調べてくれるスクリプトです。
https://github.com/slimm609/checksec.sh
checksec コマンド
```shell
$ checksec --file=a.out
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      Symbols         FORTIFY Fortified       Fortifiable   FILE
Full RELRO      Canary found      NX enabled    PIE enabled     No RPATH   No RUNPATH   38) Symbols       No    0               0    a.out
```

### Stack-Smashing Protection

関数呼び出し時に return address の次に Master Canary から得られるランダムな値 (Canary) を配置し、return 前に Canary が変化したか検証し、変わっていたら`__stack_chk_fail` 関数を呼び出して例外を送出する。

https://www.youtube.com/watch?v=UTC2iWxQ4qc

### NX bit (No eXecute bit)
読み書きのフラグの他に CPU が特定のセグメントを実行できないようにするフラグを追加する。ソフトウェア上で W^X (Write xor Execute) が成り立つようにフラグを立てることで shellcode による攻撃は完全に出来ないようになった。Windows では DEP (Data Execution Prevention) と呼ばれている。このフラグを書き換えるには Linux の場合 mprotect(2)、Windows の場合 VirtualProtectEx 関数が使える。

### RELRO (RELocation Read-Only)
glibc などの動的ライブラリをリンクするとき、lazy binding といってシンボル名の検索とアドレスの解決については呼び出し時まで遅延させてプログラムの起動を早める。ただ GOT overwrite と呼ばれる脆弱性が生まれる為、起動時に全て行い、書き込み禁止とするのが RELRO です。

|  | lazy binding | RELRO |
| --- | --- | --- |
| No RELRO | Yes | No |
| Partial RELRO | Yes | Yes |
| Full RELRO | No | Yes |

### ASLR (Address Space Layout Randomization)

スタックやヒープ、動的ライブラリが置かれるベースアドレスをランダムに決める。これらのアドレスを知るには以下のコマンドを叩く。毎回変わることが分かる。

```
$ cat /proc/<process ID>/maps              // = gdbのi proc map = pwndbgのvmmap
```

### PIE (Position-Independent Executables)

実行ファイルそのものが置かれるベースアドレスをランダムに決められる。

### ASCII-armor

共有ライブラリのベースアドレスを `0x00XXXXXX` のように `\x00` を含めることで BOF によって書き込むことを難しくする機構。

### Control Flow Integrity

ROP, JOP 対策として導入された CPU のセキュリティ機構。Intel, ARM, RISC-V における CFI (Control Flow Integrity) 拡張をまとめる。

- Intel CET Shadow Stack
    - call 命令で return address を shadow stack に push し、ret 命令で pop して一致しなければ例外を送出する。既存のコードを変更せずに適用できるのが強み。
- Intel CET Indirect Branch Tracking (IBT)
    - jump 先に endbranch 命令を埋め込み、関数ポインタの先が endbranch 命令を指していない場合には例外が送出される機構。JOP の検知ができる。
- ARM Branch Target Identification (BTI)
    - br/blr 命令によって jump した先が bti 命令以外であれば例外を送出する機構。PTE の GP ビットが立っているときにそのアドレス範囲での BTI が有効になる。JOP の検知ができる。
- ARM Pointer Authentication (PAC)
    - PAC 命令で 64 bit ポインタの上位 8 bit にタグ, 3-23 bit にアドレスの署名を埋め込み、AUTH 命令で認証し、無効な署名を持つ場合に例外が送出される機構。署名は QARMA アルゴリズムが推奨されている。ROP の検知ができる。
- ARM Memory Tagging Extension (MTE)
    - メモリにタグを割り当て、ポインタの上位 4 bit にタグを埋め込み、アクセス時にそれらが不一致の場合には例外が送出される機構。Use After Free の検知ができる。
- RISC-V CFI Shadow Stack
    - 特殊なメモリ領域に Shadow Stack を確保し、return address のみの読み書きをする。sspush, sspop, sschkra 命令によって return address のプッシュ、ポップ、比較をし、不一致ならば例外を送出する。
- RISC-V CFI Landing Pads
    - 新しい命令 lpsll, lpcll 命令を追加して jalr 命令の分岐前と分岐後で label を照合することで想定された組み合わせかどうかを判定し、不一致なら例外を送出する。デメリットとしてツールチェイン側が実装する際にバグが発生しそうなことが挙げられる。