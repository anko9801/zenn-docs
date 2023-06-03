---
title: "Crypto 用ツール"
---

## Python
まずはバージョン管理マネージャー asdf を入れます。以下のリンクを読んで asdf のインストールをしてください。

https://asdf-vm.com/guide/getting-started.html

asdf を入れたら Python を入れます。

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

これで Python が入りました。:tada:
私が入れてるエコシステムはこれくらいです。
- Pylance (language support)
- Ruff (linter)

今回使うパッケージはこんな感じです。

```shell
$ pip install pycryptodome gmpy2 pwntools z3
```

## SageMath
SageMath とは

```shell
$ sudo apt-get install sagemath
```

チートシート

```python
# import
load('coppersmith.sage')

# 環
ZZ # 整数環 Z
Zmod(N) # 剰余環 Z/NZ
QQ # 有理数体 Q
RR # 実数体 R
CC # 複素数体 C

# 有限体 FiniteField Galois Field
GF(q)
GF(p).primitive_element()
F = GF(13)
F(9) * 200 == 9
R = GF(2^m)
R.fetch_int(12)

# 多項式 Polynomial Ring
R = Zmod(N)
P.<x, y> = PolynomialRing(R)
f = x^2 + 3*x + 3
f.small_roots()
p /= p.leading_coefficient()  # 最高次項 モニック化
p = p.monic()
ideal(f).groebner_basis()

# 剰余環 Quotient Ring
PR.<x> = PolynomialRing(GF(2))
QR.<x> = PR.quotient(x^32 + x^26 + x^23 + x^22 + x^16 + x^12 + x^11 + x^10 + x^8 + x^7 + x^5 + x^4 + x^2 + x^ + 1)


crt(remain_list, modulo_list)
b.nth_root(3)

# 行列
M = matrix(QQ, [
    [1, 2],
    [3, 4]
])
M.LLL()

A = matrix(Zn, [
    [a, b, c, d, e],
    [1, 0, 0, 0, 0],
    [0, 1, 0, 0, 0],
    [0, 0, 1, 0, 0],
    [0, 0, 0, 1, 0],
])

b11, b12, b13, b14, b15 = map(int, (A ^ e)[4])

D, P = m.eigenmatrix_left()
P^(-1)*D*P == m

Q = diagonal_matrix(weights)

B *= Q
B = B.LLL()
B /= Q

# 素因数分解
set(factor(n))

# 楕円曲線
EllipticCurve()
```


## SAT と SMT
命題論理式

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
& (\lnot B\lor C).
\end{aligned}
$$

次のような論理式は CNF ではありません。

$$
\begin{aligned}
& \lnot(B\lor C) \\
& (A\land B)\lor C \\
& A\land(B\lor(D\land E)).
\end{aligned}
$$

二重否定の除去、ド・モルガンの法則、分配法則などを用いて CNF に変換することができます。実際、次のアルゴリズムが知られています。

> **Tseitin Encoding**
> 任意の命題論理式を CNF に変換するアルゴリズムが存在する。

これより上で示した CNF でない論理式を CNF に変換でき、次のようになります。

$$
\begin{aligned}
& \lnot B\land\lnot C \\
& (A\lor C)\land(B\lor C) \\
& A\land(B\lor D)\land(B\lor E).
\end{aligned}
$$

CNF のような命題論理式について変数の値がどのようなときに真になるかを考えます。

> **充足可能性問題 (SAT; SATisfiability problem)**
> ある命題論理式が与えられたとき、それを満たす変数の値を定める問題を充足可能性問題 SAT という。

SAT は NP 完全です。

$$
(A\land\lnot B\land\lnot C)\lor(B\land C\land\lnot D)\land(\lnot B\lor\lnot C)
$$

これは $(A,B,C,D)=(\top,\bot,\bot,\top)$ のときのみ上の論理式を満たします。

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
かなり凄い、ネットワークの
- 形式手法 / モデル検査
  - TLA+
  - seL4
- シンボリック実行エンジン
- 自動定理証明支援系
- Differential Cryptoanalysis

SMT ソルバのアルゴリズムには EUF; Equality logic with Uninterpreted Functions や BitVector などがありますが、今回は BitVector のみを紹介します。

> **BitVector**
> $n$ ビットの四則演算などの演算について、それぞれのビットに関する論理式を立てることで SMT を SAT に置き換えることができる。

使われる演算としては $a\land b$, $\lnot a$, $a < b$, $a = b$, $a[i]$, $\sim a$, $a\mathop{\|}b$, $a\mathop{\&}b$, $a \oplus b$, $a \ll b$, $a \gg b$, $a + b$, $a - b$, $a \times b$, $a / b$, $\mathrm{ext}(a)$, $a\circ b$, $a[b:c]$, $c?a:b$ などなので、これらを論理式に落とせることは CPU を自作したことがある人ならわかると思います。

例えば 1 ビットの $a + b$ なら $a + b = a\oplus b = (a\land\lnot b)\lor(\lnot a\land b)$ と置き換えられます。

デファクトスタンダードな SMT ソルバに Z3 があります。
Z3 を使える言語は C, C++, JavaScript, Python, Rust などなど、今回は Python でやっていきます。

```python
bool, int, float, float32, double, real, string, array, set, enumeration, bitvector
BitVec()
IntVector()
Real()
from z3 import Solver, Context, RecFunction, RecAddDefinition, IntSort, Int, If, simplify

ctx = Context()
f = RecFunction("f", IntSort(ctx), IntSort(ctx))
x = Int("x", ctx)
RecAddDefinition(f, x, If(x <= 2, 1, f(x-1) + f(x-2)))

solver = Solver(ctx=ctx)
solver.add(f(x) == 10946)

print(solver.check())
print(solver.model())
```


## 参考
- [quickref](https://wiki.sagemath.org/quickref?action=AttachFile&do=get&target=quickref.pdf)
- [quickref-linalg.pdf (shinshu-u.ac.jp)](http://math.shinshu-u.ac.jp/~nu/nora/sage/doc/refcard/quickref-linalg/200905/ja-utf8/quickref-linalg.pdf)
- [SAT/SMTソルバの仕組み](https://www.slideshare.net/sakai/satsmt)
- [ミュンヘン工科大学の夏学期の自動推論に関する授業](https://www21.in.tum.de/teaching/sar/SS20/)
- [SATソルバ・SMTソルバの技術と応用](https://www.jstage.jst.go.jp/article/jssst/27/3/27_3_3_24/_pdf)
- [TokyoWesterns の z3 解説](https://wiki.mma.club.uec.ac.jp/CTF/Toolkit/z3py)

SAT / SMT ソルバの入力形式は DIMACS / SMT-LIBv2 を用います。

この資料は CC0 ライセンスです。