---
title: "ハッシュとSMT"
---

## ハッシュ関数
信頼できないソースの正当性を証明するものというのは世界中で必要とされています。
ハッシュ関数とは

- MD5
- SHA
  - 現代で最も使われている

暗号ではパスワードの保存や HMAC, 署名などに使われていたりします。ただ暗号で使うためには攻撃者がハッシュ値に対する元の入力に関する情報を得られないようにしないといけません。これを原像計算困難性といいます。

![](/images/hash.png)

これを満たしたハッシュというのは実際にあります。MD5 や SHA1, SHA-256 などなど、まぁそれぞれのハッシュの実装は結構荒っぽく作られてるので詳細は省きます。ただし攻撃するときの重要な性質として Merkle-Damgård construction というものがあるのでそれだけ知っておきましょう。

### Merkle-Damgård construction
ハッシュ関数は任意長のメッセージを固定長の出力に変換しないといけません。その為に入力を固定長のブロックに分割し、1つずつ内部状態に適用させます。
![](/images/length_extension.png)

MD5 や SHA-1 などよく使われるハッシュ関数はこれです。このときに成立する攻撃というのが伸長攻撃です。伸長攻撃には慎重につってね！フゥーワッ！

## ハッシュの応用
### HMAC (Hash-based MAC)
これを秘密鍵を用いて検査するコードのことをメッセージ認証コード (message authentication codes; MAC) と言います。その中でハッシュを用いる MAC を HMAC と呼びます。

秘密鍵 $K$ とハッシュ値長 $B$ として HMAC の値 $C$ を次のように定義します。

$$
\begin{aligned}
pad_{in} & := \overbrace{\mathrm{0x36}\|\cdots\|\mathrm{0x36}}^B \\
pad_{out} & := \overbrace{\mathrm{0x5C}\|\cdots\|\mathrm{0x5C}}^B \\
\mathrm{HMAC}(K, V) & := H(K \oplus pad_{out} \| H(K \oplus pad_{in} \| V))
\end{aligned}
$$

### CRC
ビット列を $\mathbb{F}_2[x]/(f(x))$ に変換して誤り検出する方法です。$f(x)$ を生成多項式と呼び、CRC-32 では次の生成多項式を用います。

$$
x^{32} + x^{26} + x^{23} + x^{22} + x^{16} + x^{12} + x^{11} + x^{10} + x^8 + x^7 + x^5 + x^4 + x^2 + x + 1
$$

具体的には2進数の入力を多項式に変換した $m\in\mathbb{F}_2[x]$、生成多項式の次数を $n$ とすると $mx^n$ を $\mathbb{F}_2[x]/(f(x))$ として解釈したものが出力となります。

例えば入力が $1011000_{(2)}$ で生成多項式が $f(x) = x^4 + x + 1$ の場合

$$
\begin{aligned}
m & = x^6 + x^4 + x^3 \\
mx^n & = (x^6 + x^4 + x^3)x^4 = x^{10} + x^8 + x^7 + x^5 = x^3 + x^2 + x + 1 \pmod{f(x)}
\end{aligned}
$$

より出力は $1111_{(2)}$ となります。

実装では多項式を反転させてビット演算に落とし込むことで高速化できます。

## 攻撃
### データベース
逆ハッシュ化はデータベースを用いている

### 伸長攻撃 (Length Extension Attack)
$H(m_1)$ $H(m_1\|m_2)$ を求める

### 誕生日攻撃 (Birthday Attack)
誕生日のパラドックスを用いた攻撃です。

> **Def. ハッシュ値の衝突**
> $H(m_1) = H(m_2)$ となる異なる $m_1, m_2$ が発見された状態

この衝突を恣意的に起こせるとハッシュ値で改ざんを検知してるシステムを騙すことが出来てしまいます。これは誕生日攻撃を用いて恣意的に起こすことができます。

誕生日攻撃というのはハッシュ値のビット数を $n$ として平文 $m$ とそのハッシュ値 $H(m)$ の対応を $O(2^{n/2})$ 程度集めるとハッシュが同じとなるような $m$ が $50\%$ の確率で見つかるという誕生日のパラドックスによる攻撃です。

### Differenctial Cryptoanalysis
暗号を解析しながら SMT を用いて解く

## 充足可能性問題 SAT
充足可能性問題 (SAT; SATisfiability Problem)
連言標準形(CNF)
任意の命題論理式をCNFに変換する為のアルゴリズムが存在する。(Tseitin Encoding)
CNFの入力形式 DIMACS

SAT を解くには指数時間掛かると信じられている。

$$
((a\land\lnot b\land\lnot c)\lor(b\land c\land\lnot d))\land(\lnot b\lor\lnot c)
$$

これは $(a,b,c,d)=(\top,\bot,\bot,\top)$

まずは単純に全探索を考えます。リテラルが 1 つしかない節を単位節と呼ぶとして次の操作を繰り返します。

1. 単位節があればその変数の値は確定する
2. 単位節がなければ変数のどれか1つを深さ優先探索する
3. コンフリクトしたなら失敗、コンフリクトなしに変数を全て割り当てられたら成功とする

これは DPLL (Davis Putnam Logemann Loveland) アルゴリズムと呼ばれています。

この全探索に加え、コンフリクトしたときにその探索状態だと失敗することを条件に入れます。この操作を節学習と呼び、節学習されて条件が多くなると、探索をしなくて済むようになり高速化します。これを CDCL (Constrait-Driven Clause Learning) アルゴリズムと呼び、これにより SAT ソルバは画期的に速くなりました。

私はこれが図としてわかりやすいなと思っています。

https://www.youtube.com/watch?v=d76e4hV1iJY

実装は節 $x_k$, $\lnot x_k$ はそれぞれ $2k$, $2k+1$ と表して合計で $2n$ 個の配列が必要になり、各節はリテラルのポインタを格納するリンクリストを持ちます。
各変数は探索する変数の優先順位を決定させるスコアを持ち、$i$ 回目にコンフリクト時にコンフリクトした変数のスコアに $\rho^{-i}$ だけ足されます。(ex. $\rho=0.95$) つまり後の方になればなるほど増加するスピードが速くなり、古いコンフリクトは無視されるようになります。

## SMT
- 形式手法
  - TLA+
- モデル検査
- 自動定理証明支援系
- シンボリック実行エンジン
- モデル検査
  - seL4
- 論理式に帰着できるすべての問題のソルバ
- Differential Cryptoanalysis

今回は BitVector しか使わないのでそれだけ解説する。他にも EUF (Equality logic with Uninterpreted Functions) に関するアルゴリズムや blahblah など色々あるので興味ある方は参考文献などを参照されて頂きたい。

### BitVector

それぞれの n bit 演算を論理式に落とし、 SAT に投げることで解く。
使われる演算としては $a\land b$, $\lnot a$, $a < b$, $a = b$, $a[i]$, $\sim a$, $a\mathop{\|}b$, $a\mathop{\&}b$, $a \oplus b$, $a \ll b$, $a \gg b$, $a + b$, $a - b$, $a \times b$, $a / b$, $\mathrm{ext}(a)$, $a\circ b$, $a[b:c]$, $c?a:b$ があるので、これらを論理式に落とせることは CPU を自作したことがある人ならわかると思う。
例えば $a + b$ なら全加算器を論理式に落とせばよい。

SMT ソルバの入力形式は SMT-LIBv2 を用います。

## Z3
デファクトスタンダードな SMT ソルバです。
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

## まとめ
また[準同型のハッシュ関数](https://github.com/benwr/bromberg_sl2)などもあり、発展も期待です。

## 参考文献
- [Length Extension Attackの原理と実装](https://ptr-yudai.hatenablog.com/entry/2018/08/28/205129)

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
- [TokyoWesterns の z3 解説](https://wiki.mma.club.uec.ac.jp/CTF/Toolkit/z3py)

この資料は CC0 ライセンスです。