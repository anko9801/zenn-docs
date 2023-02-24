---
title: "ハッシュとSMT"
---

## ハッシュ
ハッシュとは
ハッシュはデータを識別したいときに便利です。何か
暗号的に使われるハッシュ

- 衝突困難性
$H(m_1) = H(m_2)$ となる $m_1, m_2$ が見つけにくい

![](/images/hash.png)

HMAC (Hash-based MAC)

### Merkle-Damgård construction
## 攻撃
### 伸長攻撃
### Differenctial Cryptoanalysis

## SAT
充足可能性問題 (SAT; SATisfiability Problem)

- 論理式に帰着できるすべての問題のソルバ
- Differential Cryptoanalysis
- モデル検査
- 自動定理証明支援系
- シンボリック実行エンジン

SAT を解くには指数時間掛かると信じられている。指数時間の中でも高速化していく技術を学ぶ。

$$
((a\land\lnot b\land\lnot c)\lor(b\land c\land\lnot d))\land(\lnot b\lor\lnot c)
$$

$(a,b,c,d)=(\top,\bot,\bot,\top)$

### DPLL アルゴリズム
まずは単純に全探索を考える。リテラルが 1 つしかない節を単位節と呼ぶとして次の操作を繰り返す。

1. 単位節があればその変数の値は確定する
2. 単位節がなければ変数のどれか1つを深さ優先探索する
3. コンフリクトしたなら失敗、コンフリクトなしに変数を全て割り当てられたら成功とする

これは DPLL (Davis Putnam Logemann Loveland) アルゴリズムと呼ばれている。

$$
\begin{aligned}
\phi &= (a \lor \lnot b \lor d) \land (a \lor \lnot b \lor e) \\
& \land (\lnot b \lor \lnot d \lor \lnot e) \\
& \land (a \lor b \lor c \lor d) \land (a \lor b \lor c \lor \lnot d) \\
& \land (a \lor b \lor \lnot c \lor e) \land (a \lor b \lor \lnot c \lor \lnot e)
\end{aligned}
$$

### 節学習

DPLL に加え、コンフリクトしたときにその探索状態だと失敗することを条件に入れる。この操作を節学習と呼び、DPLL に節学習を加えたアルゴリズムは CDCL (Constrait-Driven Clause Learning) アルゴリズムと呼ばれる。

| $t$ | $L_t$       | 深さ  | reason                                       |
| :-: | :---------: | :---: | :------------------------------------------- |
| 0   | $\lnot x_1$ | 0     | $\lbrace\lnot x_1\rbrace$                    |
| 1   | $x_2$       | 1     | $\lbrace\rbrace$                             |
| 2   | $\lnot x_5$ | 1     | $\lbrace \lnot x_2,\lnot x_5\rbrace$         |
| 3   | $\lnot x_3$ | 2     | $\lbrace\rbrace$                             |
| 4   | $x_4$       | 2     | $\lbrace x_1,x_3,x_4\rbrace$                 |
| 5   | $x_6$       | 2     | $\lbrace x_1,\lnot x_2,\lnot x_4,x_6\rbrace$ |
| 6   | $\lnot x_6$ | 2     | $\lbrace x_3,\lnot x_4,x_5,\lnot x_6\rbrace$ |

ここで $t$ が 0 から 3 までの条件 $\lnot x_1\land x_2\land\lnot x_5\land\lnot x_3$ のときコンフリクトすることが分かる. するとその否定 $x_1\lor\lnot x_2\lor x_5\lor x_3 = \lbrace x_1,\lnot x_2,x_5,x_3\rbrace$ は必ず成立しなければならない為, 新たに条件として入れることができる.

[疑問] このときなぜ $\lnot x_4$ を入れてはいけないのかよくわからない. 資料には $x_6$ に依存しているからと書かれているが $\lbrace x_1,x_3,x_4\rbrace$ は $x_1, x_3$ によって決定されるので関係なさそう(入れた方が条件が弱くなるのは確かにそうだけれども, もし全てのリテラルがfalseでも $\lbrace x_1,x_3,x_4\rbrace$ とコンフリクトするので大丈夫そう). たぶん節内のリテラルを出来る限り少なくして探索時に有用に扱いたいんだと思う.

そして条件が多くなればなるほど探索をしなくて済むので高速化出来る.

その他最適化するアルゴリズムの基本的なアイデアとその名前を挙げるので興味ある方は調べてみるのも良さです。
1. 命題論理・述語論理の推論規則を用いて恒真命題か矛盾命題かを証明する。(Tableaux algorithm)
2. $A\lor L$ と $B\lor\lnot L$ があるなら $A\lor B$ を推論できる。特に $A = B$ のとき項数が少なくなる。(Resolution)

### 実装
効率的な CDCL の実装方法を学ぶ。

節は $x_k$ と $\lnot x_k$ は $2k$, $2k+1$ と表す。合計で $2n$ ビット必要になる。
各節はリテラルのポインタを格納するリンクリストを持つ
次に探索する変数の優先順位を表すアクティビティスコアを持つ
アクティビティスコアはコンフリクト時に各変数のそれに足されるものである。i回目のコンフリクト時には $\rho^{-i}$ 足される。(ex. $\rho=0.95$) 後の方になればなるほど増加するスピードが速くなり、古いコンフリクトは無視されるようになる。

殆どのSATソルバは連言標準形(CNF)を入力として受け取る。
任意の命題論理式をCNFに変換する為のアルゴリズムが存在する。(Tseitin Encoding)

$$
\begin{aligned}
\phi&\coloneqq((p\lor q)\land r)\to(\lnot s) \\
x_1&\leftrightarrow\lnot s \\
x_2&\leftrightarrow p\lor q \\
x_3&\leftrightarrow x_2\land r \\
x_4&\leftrightarrow x_3\to x_1 \\
T(\phi)&\coloneqq x_4\land(x_4\leftrightarrow x_3\to x_1)\land(x_3\leftrightarrow x_2\land r)\land(x_2\leftrightarrow p\lor q)\land(x_1\leftrightarrow\lnot s) \\
x_2\leftrightarrow p\lor q &= (x_2\to(p\lor q))\land((p\lor q)\to x_2) \\
&= (\lnot x_2\lor p\lor q)\land(\lnot(p\lor q)\lor x_2) \\
&= (\lnot x_2\lor p\lor q)\land((\lnot p\land \lnot q)\lor x_2) \\
&= (\lnot x_2\lor p\lor q)\land(\lnot p\lor x_2)\land(\lnot q\lor x_2) \\
\end{aligned}
$$

CNFの入力形式 DIMACS

## SMT
形式手法
- モデル検査
- 定理証明支援系

今回は BitVector しか使わないのでそれだけ解説する。他にも EUF (Equality logic with Uninterpreted Functions) に関するアルゴリズムや blahblah など色々あるので興味ある方は参考文献などを参照されて頂きたい。

### BitVector

| 構文 | 式 |
|:--:|:--|
| $a\land b$ | |
| $\lnot a$ | |
| $a < b$ | |
| $a = b$ | |
| $a[i]$ | |
| $\sim a$ | |
| $a\mathop{\|}b$ | |
| $a\mathop{\&}b$ | |
| $a \oplus b$ | |
| $a \ll b$ | |
| $a \gg b$ | |
| $a + b$ | 全加算器 |
| $a - b$ | |
| $a \times b$ | |
| $a / b$ | |
| $\mathrm{ext}(a)$ | |
| $a\circ b$ | |
| $a[b:c]$ | |
| $c?a:b$ | |

### 実装
SMTソルバの入力形式 SMT-LIBv2


## Z3
私が知っている Z3 を使える言語は Python, Rust です。ラッパを書けば

使えるデータ
- bool, int, float, float32, double, real, string
- array, set, enumeration, bitvector

```python
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

## 参考文献
SMT ソルバ全般
- [SAT/SMTソルバの仕組み](https://www.slideshare.net/sakai/satsmt)
- [ミュンヘン工科大学の夏学期の自動推論に関する授業](https://www21.in.tum.de/teaching/sar/SS20/)

SAT/SMTソルバのサーベイ論文
- [SATソルバ・SMTソルバの技術と応用](https://www.jstage.jst.go.jp/article/jssst/27/3/27_3_3_24/_pdf)
- [A Survey of Satisfiability Modulo Theory](https://arxiv.org/abs/1606.04786)

SMT-LIBv2
- [The SMT-LIBv2 Language and Tools: A Tutorial](http://smtlib.github.io/jSMTLIB/SMTLIBTutorial.pdf)
p20. SMT-LIBv2 の token が表になって並んでおり、どのような正規表現でマッチさせられるか掲載している
- [SMT-LIB The Satisfiability Modulo Theories Library](http://smtlib.cs.uiowa.edu/)
SMT ソルバに与える入力の形式 SMT-LIB v2 についてまとまっている Web サイト
- [SMT-LIB-benchmarks / QF_UF · GitLab (uiowa.edu)](https://clc-gitlab.cs.uiowa.edu:2443/SMT-LIB-benchmarks/QF_UF)
QF_UF のベンチマーク用入力が大量に用意されている