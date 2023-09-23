---
title: "【CTF 探訪記】Crypto に使うツール"
emoji: "🦾"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CTF", "crypto", "Z3"]
published: true
---

この記事は各暗号を理解して解読できるようになるシリーズの一部です。一覧は次の記事を見てください。

https://zenn.dev/anko/articles/ctf-crypto-begginer

CTF の Crypto で使うツール、と言っても主に SageMath しか使いませんが次の 3 つを紹介していこうと思います。

- Python
- SageMath
- Z3

これらは計算機で数学や論理を解く非常に有用なツールで数学をベースにした Crypto も解けてしまうという訳です。

## Python
まずはバージョンマネージャーを入れましょう。これは様々なバージョンのコンパイラ、インタプリタを管理してくれるアプリケーションです。このソースコードはあるバージョンでは動いていたけれども次のバージョンで動かなくなったというのは IT 業界日常茶飯事なのでこれがあるとコードを書きやすくなります。

ここではその 1 つである asdf を入れましょう。

https://asdf-vm.com/guide/getting-started.html

そして asdf を入れたら Python を入れます。

```shell
$ asdf plugin-add python
$ asdf list-all python
$ asdf install python 3.11.2
$ asdf install python 2.7.18
$ asdf global python 3.11.2 2.7.18
$ asdf reshim
$ python -V
Python 3.11.2
$ python3 -V
Python 3.11.2
$ python2 -V
Python 2.7.18
```

これで Python が入りました！

あとはエコシステムも導入しておきましょう。私が入れてるエコシステムはこれくらいです。

- Ruff (Linter for NeoVim)
- Pylance (Language support for VSCode)

また crypto 分野でよく使うパッケージはこんな感じです。入れておきましょう。

```shell
$ pip install pycryptodome gmpy2 pwntools z3
```

## SageMath
SageMath は数学の幅広い計算処理を行える OSS のソフトウェアです。具体的には数値計算から計算機代数まで充実しています。中身としては SageMath 専用の言語を Python へトランスパイルして大量のライブラリと結合させて実行するというような感じです。

インストール方法はパッケージマネージャで入れるかビルドするかの 2 択あります。`apt` であれば次のようにインストール出来ます。

```shell
$ sudo apt-get install sagemath sagemath-doc sagemath-jupyter
```

始めるにはとりあえずチュートリアルを読みましょう。

https://doc.sagemath.org/html/ja/tutorial/tour.html

それもめんどくさい方は最も重要な代数構造の具体的対象は次のように生成できるということを知っていればあとは Python と同じように扱えると思っておいてください。

```python
ZZ                                            # 整数環 Z
Zmod(N)                                       # 剰余環 Z/NZ
GF(q)                                         # 有限体 F_q
QQ                                            # 有理数体 Q
RR                                            # 実数体 R
CC                                            # 複素数体 C
PR.<x> = GF(2)[]                              # 多項式環 F_2[x]
QR.<x> = PR.quotient(x^8 + x^4 + x^3 + x + 1) # 剰余環 F_2[x]/(x^8+x^4+x^3+x+1)
GF(2^8) == QR
```

sagemath には豊富なリファレンスがあるので大抵それを読めばわかります。チュートリアルを触るだけでもかなり十分です。

- 基本
  - [数論のチュートリアル](https://doc.sagemath.org/html/ja/tutorial/tour_numtheory.html)
- 多項式
  - [多項式のチュートリアル](https://doc.sagemath.org/html/ja/tutorial/tour_polynomial.html)
  - [グレブナー基底など](https://doc.sagemath.org/html/en/reference/polynomial_rings/sage/rings/polynomial/multi_polynomial_ideal.html)
- 線形代数
  - [線形代数のチュートリアル](https://doc.sagemath.org/html/ja/tutorial/tour_linalg.html)
  - [General matrix Constructor and display options](https://doc.sagemath.org/html/en/reference/matrices/sage/matrix/constructor.html)
- 楕円曲線
  - [チュートリアル より進んだ数学](https://doc.sagemath.org/html/ja/tutorial/tour_advanced.html)
  - [Elliptic curves](https://doc.sagemath.org/html/en/reference/arithmetic_curves/index.html)

分からなければソースコードも読んだりビルドしたりして実験すれば完璧です。

## SAT / SMT
比較的簡単な暗号や共通鍵暗号、乱数生成においては条件に合うような唯一または複数の状態を全探索して見つけ出せれば解けてしまうということがよくあります。これを極限まで高速に解いてくれるのが SAT / SMT です。まずはその仕組みを理解していきましょう。

### 仕組み
まずは最も簡単な命題論理式、その中でも連言標準形と呼ばれる形の命題論理式を解くことを考えてみます。

> **連言標準形 (CNF; Conjunctive Normal Form)**
> 変数 $A$ に関して $A, \lnot A$ をリテラルと定義する。このとき連言標準形とはリテラル $l_{i,j}$ に対して次のように書ける命題論理式である。
>
> $$
\bigwedge_i\bigvee_j l_{i,j}
$$
>
> このとき $\bigvee_j l_{i,j}$ を節という。

例えば次のような論理式は CNF です。

$$
\begin{aligned}
& A\land B \\
& \lnot A\land(B\lor C) \\
& (A\lor B)\land(\lnot B\lor C\lor\lnot D)\land(D\lor\lnot E) \\
& (\lnot B\lor C)
\end{aligned}
$$

次のような論理式は CNF ではありません。

$$
\begin{aligned}
& \lnot(B\lor C) \\
& (A\land B)\lor C \\
& A\land(B\lor(D\land E))
\end{aligned}
$$

一般の命題論理式は二重否定の除去、ド・モルガンの法則、分配法則などを用いて CNF に変換することができます。実際、次のアルゴリズムが知られています。

> **Tseitin Encoding**
> 任意の命題論理式を CNF に変換するアルゴリズムが存在する。

これより上で示した CNF でない論理式を CNF に変換でき、次のようになります。

$$
\begin{aligned}
& \lnot B\land\lnot C \\
& (A\lor C)\land(B\lor C) \\
& A\land(B\lor D)\land(B\lor E)
\end{aligned}
$$

次に CNF のような命題論理式について変数の値がどのようなときに真になるかを考えます。

> **充足可能性問題 (SAT; SATisfiability problem)**
> ある命題論理式が与えられたとき、それを満たす変数の値を定める問題を充足可能性問題 SAT という。

例えば次の CNF について考えてみましょう。

$$
(A\land\lnot B\land\lnot C)\lor(B\land C\land\lnot D)\land(\lnot B\lor\lnot C)
$$

これは $(A,B,C,D)=(\top,\bot,\bot,\top)$ のときのみ上の論理式を満たします。

全探索に近いことをしないと解けないことがわかるでしょう。実際、この問題は NP 完全です。

SAT を解くアルゴリズムを SAT ソルバといい、CNF に変換すると何かと解析しやすいので CNF に絞って考えます。

まずは単純に全探索を考えます。リテラルが 1 つしかない節を単位節と呼ぶとして次の操作を繰り返します。

1. 単位節があればその変数の値は確定する
2. 単位節がなければ変数のどれか1つを深さ優先探索する
3. コンフリクトしたなら失敗、コンフリクトなしに変数を全て割り当てられたら成功とする

これは DPLL (Davis Putnam Logemann Loveland) アルゴリズムと呼ばれています。

この全探索に加え、コンフリクトしたときにその探索状態だと失敗することを条件に入れます。この操作を節学習と呼び、節学習されて条件が多くなると、探索をしなくて済むようになり高速化します。これを CDCL (Constrait-Driven Clause Learning) アルゴリズムと呼び、これにより SAT ソルバは画期的に速くなります。

この資料がわかりやすいなと思っています。

https://www.youtube.com/watch?v=d76e4hV1iJY&t=760s

実装は節 $x_k$, $\lnot x_k$ はそれぞれ $2k$, $2k+1$ と表して合計で $2n$ 個の配列が必要になり、各節はリテラルのポインタを格納するリンクリストを持ちます。各変数は探索する変数の優先順位を決定させるスコアを持ち、$i$ 回目にコンフリクト時にコンフリクトした変数のスコアに $\rho^{-i}$ だけ足されます。(ex. $\rho=0.95$) つまり後の方になればなるほど増加するスピードが速くなり、古いコンフリクトは無視されるようになります。

このように命題論理式は SAT ソルバを用いて解くことができます。SAT ソルバだけでもかなり有用なツールですが、ここではさらに発展させて SAT について実数、整数、リスト、配列、文字列など様々なデータ構造を含む、より複雑な数式に一般化した問題を考えます。

> **SMT; Satisfiability Modulo Theories**
> ある数式が与えられたとき、それを満たす変数の値を定める問題を SMT という。

SMT ソルバはいろいろな応用先があり、例えば次のようなものがあります。

- 数式に帰着できるすべての問題のソルバ
- 形式手法 / モデル検査
  - TLA+
  - seL4 Kernel
- シンボリック実行エンジン
- 自動定理証明支援系
- Differential Cryptoanalysis

SMT ソルバのアルゴリズムには EUF (Equality logic with Uninterpreted Functions) や BitVector などがありますが、今回は BitVector のみを紹介します。

> **BitVector**
> $n$ ビットの四則演算などの演算について、それぞれのビットに関する論理式を立てることで SMT を SAT に置き換えることができる。

使われる演算としては $a\land b$, $\lnot a$, $a < b$, $a = b$, $a[i]$, $\sim a$, $a\mathop{\|}b$, $a\mathop{\&}b$, $a \oplus b$, $a \ll b$, $a \gg b$, $a + b$, $a - b$, $a \times b$, $a / b$, $\mathrm{ext}(a)$, $a\circ b$, $a[b:c]$, $c?a:b$ などなので、これらを論理式に落とせることは CPU を自作したことがある人ならわかると思います。

例えば 1 ビットの $a + b$ なら $a + b = a\oplus b = (a\land\lnot b)\lor(\lnot a\land b)$ と置き換えられます。

### Z3

デファクトスタンダードな SMT ソルバに Z3 があります。Z3 をライブラリとして扱える言語は C, C++, JavaScript, Python, Rust などがあります。

詳しくは TokyoWesterns の Z3 解説が一番充実していると思います。
https://wiki.mma.club.uec.ac.jp/CTF/Toolkit/z3py

## 参考
- [quickref](https://wiki.sagemath.org/quickref?action=AttachFile&do=get&target=quickref.pdf)
- [quickref-linalg.pdf (shinshu-u.ac.jp)](http://math.shinshu-u.ac.jp/~nu/nora/sage/doc/refcard/quickref-linalg/200905/ja-utf8/quickref-linalg.pdf)
- [SAT/SMTソルバの仕組み](https://www.slideshare.net/sakai/satsmt)
- [ミュンヘン工科大学の夏学期の自動推論に関する授業](https://www21.in.tum.de/teaching/sar/SS20/)
- [SATソルバ・SMTソルバの技術と応用](https://www.jstage.jst.go.jp/article/jssst/27/3/27_3_3_24/_pdf)
