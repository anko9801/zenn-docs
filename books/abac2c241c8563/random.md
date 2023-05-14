---
title: "乱数生成とSMT"
---

誰にも予測できないような乱数を作るにはどうすればいいでしょうか？

例えばこんな例を考えてみましょう。
乱数を要求されたら円周率の $n$ 桁目からの数字を順番に返すような乱数生成器の OSS とその API があるとします。これは均一に分布しているし分散していて統計的には乱数と言えます。では果たしてこれは暗号として使うには安全でしょうか？

もちろん安全ではありません。$n$ がソースコードに書かれてあるので誰にでも予測できてしまいます。

では $n$ を誰にも分からないくらいとても大きくして秘密にすればどうでしょうか？
もちろん安全になります。でもよく考えてみてください。開発者は $n$ を知っているので開発者だけが簡単に予測することができてしまいます。これをバックドアといい、出来る限り避けるべきです。つまりオープンソースであることは必須条件です。

これより乱数に必要な条件としてはこうです。

1. 統計的に選ばれる確率が均一であり十分に分散している
2. 全てがオープンソースである
3. 誰にも予測できない

このような乱数を生成することはできるのでしょうか？この章ではそのような乱数生成器を紹介していきます。

## 乱数生成
### 真性乱数生成器 (TRNG)
最も原始的で簡単な方法がノイズを用いる方法です。ノイズといっても音のノイズではなく、熱のノイズや電気のノイズ、トンネル効果などを用いた量子的な確率的振る舞いなどです。それらノイズの情報をエントロピーが低くなるまで掻き集めて、乱数として返します。これを真性乱数生成器 (TRNG; True Random Number Generator) と呼びます。

メリット
- 非決定的に生成するので次の値を予測しにくい

デメリット
- 生成に時間がかかる
- サイドチャネル攻撃などによってバイアスが掛かる可能性がある

ただ真性乱数でも意図的に外から負荷を掛けてノイズの予測を立てられることもあるので、一概にエントロピーが低いことを保証できません。それを用いた攻撃をサイドチャネル攻撃と言います。

### 決定論的乱数生成器 (DRBG)
TRNG だと乱数を生成するのに熱が揺らいでいるのを取らないといけないので時間がかかります。高速化するにはアルゴリズムで疑似的に乱数を生成することが必要です。これを決定論的乱数生成器 (DRBG; Deterministic Random Bit Generator) と呼びます。
ex.) XorShift、線形合同法、メルセンヌ・ツイスタ、LFSR

メリット
- 高速に生成できる

デメリット
- 決定論的に生成するので予測できる可能性がある

### 疑似乱数生成器 (PRNG)
それなら TRNG と DRBG を組み合わせれば長所と短所を互いに補ってよいじゃないかというのはごもっともで DRBG と TRNG を組み合わせた乱数生成器もしくは DRBG のみを疑似乱数生成器 (PRNG; Pseudo Random Number Generator) と呼びます。(実際は DRBG と PRNG は同じ意味らしいが便宜上この定義とする)

:::message
**練習問題**
- Easy: `/dev/random` や `/dev/urandom` を用いて乱数を生成してみよう
- Medium: `/dev/random` と `/dev/urandom` の違いとは？
:::

私は TRNG の仕組みをあまり知らないので DRBG だけを紹介していこうと思います。

DRBG は競プロのテストは XorShift を用います。今回はメルセンヌ・ツイスタを紹介します。線形合同法の攻撃については多くの解説記事があるのでここでは割愛します。良記事としては次のものがあります。

https://www.youtube.com/watch?v=WaAErTq7hWA

DRBG に対する全ての攻撃は内部状態をいかに復元するかが鍵となっています。

### LFSR (Linear Feedback Shift Register)

$\boldsymbol{a}_k^n = (a_k, a_{k+1}, \ldots, a_{k+n-1})\in\mathbb{F}_2^n$

$$
\begin{aligned}
a_{k+n} & = \boldsymbol{s}\cdot\boldsymbol{a}_k^{n} = \sum_{i=0}^{n-1} s_ia_{k+i} \\
x_k & = \boldsymbol{c}\cdot\boldsymbol{a}_k^{n} = \sum_{i=0}^{n-1} c_ia_{k+i}
\end{aligned}
$$

```python
class LFSR():
  def __init__(self, n):
    self.state = [getRandBit(1) for _ in range(n)]
    self.taps = [getRandBit(1) for _ in range(n)]
  def next(self):
    output = reduce(xor, [bit&tap for bit,tap in zip(self.state, self.taps)])
    self.state = self.state[1:] + [output]
    return output

class lfsr():
  def __init__(self, init, mask, length):
    self.init = init
    self.mask = mask
    self.lengthmask = 2**(length+1)-1

  def next(self):
    nextdata = (self.init << 1) & self.lengthmask
    i = self.init & self.mask & self.lengthmask
    output = 0
    while i != 0:
      output ^= (i & 1)
      i = i >> 1
    nextdata ^= output
    self.init = nextdata
    return output

def generate_bit_sequence(initial_state):
  state = initial_state
  while True:
    last_bit = state & 1
    yield last_bit
    middle_bit = state >> 3 & 1
    state = (state >> 1) | ((last_bit ^ middle_bit) << 4)
```

:::message
zer0lfsr, zer0lfsr+, zer0lfsr++ を解こう
:::

#### NLSR
非線形化を施すと行列形式で書くことができないので逆変換が難しくなる。

攻撃手法
1. 高次項は無視や線形近似をする (Walsh-Hadamard 変換)
2. 低次項は状態として定義する
3. annihilator
4. Correlation Attack
5. Z3

### メルセンヌ・ツイスタ (MT19937)
統計的に十分に分散していて長い周期を持つ高速な疑似乱数生成器の一種です。周期の長さは $2^{19937}-1$ とメルセンヌ数であり名前の由来になっています。実は日本人が作っています。

中身では 32 ビットのビットベクタで計算されていて初期状態 $\boldsymbol{x}_i$ $(i = 0,\cdots,n)$ を入力して漸化式から $\boldsymbol{x}_k$ を生成し、それぞれの $\boldsymbol{x}_k$ について後処理をした $\boldsymbol{y}$ を出力とします。

$$
\begin{aligned}
  \boldsymbol{x}_{k+n} & = \boldsymbol{x}_{k+m}\oplus((\boldsymbol{x}_k\mid\boldsymbol{x}_{k+1})\gg 1)\oplus(\mathrm{LSB}(\boldsymbol{x}_{k+1})\mathop{\mathrm{AND}}\boldsymbol{a}) \\
  \boldsymbol{y} & \leftarrow \boldsymbol{x}_k \\
  \boldsymbol{y} & \leftarrow \boldsymbol{y} \oplus\ \,(\boldsymbol{y}\gg 11) \\
  \boldsymbol{y} & \leftarrow \boldsymbol{y} \oplus((\boldsymbol{y}\ll\ \ 7) \mathop{\mathrm{AND}} \boldsymbol{b}) \\
  \boldsymbol{y} & \leftarrow \boldsymbol{y} \oplus((\boldsymbol{y}\ll 15) \mathop{\mathrm{AND}} \boldsymbol{c}) \\
  \boldsymbol{y} & \leftarrow \boldsymbol{y} \oplus\ \,(\boldsymbol{y}\gg 18) \\
\end{aligned}
$$

ただし、$\boldsymbol{x}_k\mid\boldsymbol{x}_{k+1}$ は $\boldsymbol{x}_k$ の最上位ビットと $\boldsymbol{x}_{k+1}$ の下位 31 ビットを結合する演算、$\mathrm{LSB}(\boldsymbol{x}_{k+1})$ は $\boldsymbol{x}_{k+1}$ の最下位ビットを 32 ビットに展開する演算です。パラメータと初期シード $x_0$ を元に初期状態を生成する漸化式は次のようにします。

$$
\begin{aligned}
  n & = 624 \quad m = 397 \\
  \boldsymbol{a} & = \mathrm{0x9908B0DF} \quad \boldsymbol{b} = \mathrm{0x9D2C5680} \quad \boldsymbol{c} = \mathrm{0xEFC60000} \\
  x_i & = (x_{i-1} \oplus (x_{i-1}\gg 30))\times 1812433253 + i \pmod{2^{32}}
\end{aligned}
$$

見ての通り、逆変換を行うことで任意の連続した 624 回の 32 ビット出力から内部状態を復元できてしまいます。
https://6715.jp/posts/6/

更に初期状態については 2 つの値さえ分かっていれば内部状態を復元できてしまいます。
https://www.ambionics.io/blog/php-mt-rand-prediction

連続する 624 個の 32 ビット出力なら一意に解けますが次のようなときはどうでしょう。

- 連続ではなかったら？
- 情報が足りず、一意でなくともいいなら？
- より一般の乱数生成では？

これらを論理的に計算するのはとても骨が折れます。

こういうときには SMT で解けます！
全探索より速いアルゴリズムがないときは SMT が強い

これでは乱数を予測されてしまうので暗号には使えません。暗号で使えるような DRBG はあるのでしょうか？それらを次節で紹介します。

:::message
**練習問題**
Python の `random.random()` を読んでみよう。
https://github.com/python/cpython/blob/main/Lib/random.py
:::

:::message
**演習問題**
[CTF archive](https://cryptohack.org/challenges/ctf-archive/) から出題
- Z3 を
- Twist and Shout (Zh3r0 CTF V2)
- import numpy as MT (Zh3r0 CTF V2)
- Real Mersenne (Zh3r0 CTF V2)
:::

## CSPRNG
暗号でも使えるような PRNG を暗号論的擬似乱数生成器 (CSPRNG; Cryptographically Secure Pseudo Random Number Generator) と呼びます。現在、標準化されている CSPRNG は [NIST SP 800-90A (Recommendation for Random Number Generation Using Deterministic Random Bit Generators)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-90a.pdf) に書かれてあります。

- Hash_DRBG
bcrypt などのハッシュ関数 $H$ とシード値 $V_0$ を用いて $V_{i+1} = H(V_i + 1)$ と生成する
- HMAC_DRBG
HMAC とシード値 $V_0$ を用いて $V_{i+1} = \mathrm{HMAC}(K, V_i)$ と生成する
- CTR_DRBG
AES-CTR の暗号化関数 $E_K$ とシード値 $V_0$ を用いて $V_{i+1} = E_K(V_i + 1)$ と生成する
- Dual_EC_DRBG (deprecated)
楕円曲線の加算を用いて生成する (後述)

これらに共通することとして乱数、Nonce、ユーザーによって指定される文字列を入力し、内部状態であるシード値を生成します。このシード値を用いて、指定されたビット数に達するまで乱数を生成し続けて連結させたものを出力します。何回か生成したらシード値の再生成 (reseed) を行い、エントロピーを上げます。

攻撃する方法としては今までと同様に内部状態を復元することで乱数予測することができます。

### Dual_EC_DRBG

NIST-p256, NIST-p384, NIST-p521 において点 $P, Q$ と初期シード $s_0$ を用いて乱数を生成します。

$$
\begin{aligned}
  s_{i+1} & = (s_iP)_x \\
  r_{i+1} & = (s_{i+1}Q)_x
\end{aligned}
$$

$r_i$ は剰余未満の数であり、その上位 2 バイト程を削除した数を連結させて出力します。P-256 なら 32 バイトの数なので下位 30 バイトを出力します。

しかし、もし NSA がこの点について ECDLP が解けている場合、内部状態を復元できる為、バックドアとなります。この為、2006 年に NIST SP800-90A に組み込まれましたが、2013 年に利用すべきではないと勧告されています。

### コラム
乱数調整

## 充足可能性問題 SAT / SMT
命題論理式

> **連言標準形 (CNF; Conjunctive Normal Form)**
> 変数 $A$ に関して $A, \lnot A$ をリテラルと定義される。このとき連言標準形とはリテラル $l_{i,j}$ に対して次のように書ける論理式である。
>
> $$
\bigwedge_i\bigvee_j l_{i,j}
$$

例えば次のような論理式は CNF である。

$$
\begin{aligned}
& A\land B \\
& \lnot A\land(B\lor C) \\
& (A\lor B)\land(\lnot B\lor C\lor\lnot D)\land(D\lor\lnot E) \\
& (\lnot B\lor C).
\end{aligned}
$$

次のような論理式は CNF ではない。

$$
\begin{aligned}
& \lnot(B\lor C) \\
& (A\land B)\lor C \\
& A\land(B\lor(D\land E)).
\end{aligned}
$$

> **Tseitin Encoding**
> 任意の命題論理式を CNF に変換するアルゴリズムが存在する。

これより次のように

$$
\begin{aligned}
& \lnot B\land\lnot C \\
& (A\lor C)\land(B\lor C) \\
& A\land(B\lor D)\land(B\lor E).
\end{aligned}
$$

> **充足可能性問題 (SAT; SATisfiability problem)**
> ある命題論理式が与えられたとき、それを満たす変数の値を定める問題である。

SAT は NP 完全である。

$$
(A\land\lnot B\land\lnot C)\lor(B\land C\land\lnot D)\land(\lnot B\lor\lnot C)
$$

これは $(A,B,C,D)=(\top,\bot,\bot,\top)$ のとき論理式を満たす。

まずは単純に全探索を考えます。リテラルが 1 つしかない節を単位節と呼ぶとして次の操作を繰り返します。

1. 単位節があればその変数の値は確定する
2. 単位節がなければ変数のどれか1つを深さ優先探索する
3. コンフリクトしたなら失敗、コンフリクトなしに変数を全て割り当てられたら成功とする

これは DPLL (Davis Putnam Logemann Loveland) アルゴリズムと呼ばれています。

この全探索に加え、コンフリクトしたときにその探索状態だと失敗することを条件に入れます。この操作を節学習と呼び、節学習されて条件が多くなると、探索をしなくて済むようになり高速化します。これを CDCL (Constrait-Driven Clause Learning) アルゴリズムと呼び、これにより SAT ソルバは画期的に速くなります。

この資料がわかりやすいなと思っています。

https://www.youtube.com/watch?v=d76e4hV1iJY&t=760s

実装は節 $x_k$, $\lnot x_k$ はそれぞれ $2k$, $2k+1$ と表して合計で $2n$ 個の配列が必要になり、各節はリテラルのポインタを格納するリンクリストを持ちます。各変数は探索する変数の優先順位を決定させるスコアを持ち、$i$ 回目にコンフリクト時にコンフリクトした変数のスコアに $\rho^{-i}$ だけ足されます。(ex. $\rho=0.95$) つまり後の方になればなるほど増加するスピードが速くなり、古いコンフリクトは無視されるようになります。

> **SMT; Satisfiability Modulo Theories**
> ある数式が与えられたとき、それを満たす変数の値を定める問題である。

SMT は SAT について実数、整数、リスト、配列、文字列など様々なデータ構造を含むより複雑な数式に一般化したものです。SMT ソルバは実際次のように応用されます。

- 論理式に帰着できるすべての問題のソルバ
- 形式手法 / モデル検査
  - TLA+
  - seL4
- 自動定理証明支援系
- シンボリック実行エンジン
- Differential Cryptoanalysis

SMT ソルバのアルゴリズムには EUF; Equality logic with Uninterpreted Functions や BitVector などがあり、今回は BitVector のみを紹介します。

> **BitVector**
> $n$ ビットの四則演算などの演算について、それぞれのビットに関する論理式を立てることで SMT を SAT に置き換えることができる。

使われる演算としては $a\land b$, $\lnot a$, $a < b$, $a = b$, $a[i]$, $\sim a$, $a\mathop{\|}b$, $a\mathop{\&}b$, $a \oplus b$, $a \ll b$, $a \gg b$, $a + b$, $a - b$, $a \times b$, $a / b$, $\mathrm{ext}(a)$, $a\circ b$, $a[b:c]$, $c?a:b$ などなので、これらを論理式に落とせることは CPU を自作したことがある人ならわかると思います。

例えば 1 ビットの $a + b$ なら $a + b = a\oplus b = (a\land\lnot b)\lor(\lnot a\land b)$ と置き換えられます。

デファクトスタンダードな SMT ソルバに Z3 があります。
私が知っている Z3 を使える言語は Python, Rust です。ラッパを書けばよさそう。

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

## 参考文献

- [メルセンヌ・ツイスタをわかった気になる](https://6715.jp/posts/5/)
- [Mersenne Twisterの出力を推測してみる](https://inaz2.hatenablog.com/entry/2016/03/07/194147)
- [SAT/SMTソルバの仕組み](https://www.slideshare.net/sakai/satsmt)
- [ミュンヘン工科大学の夏学期の自動推論に関する授業](https://www21.in.tum.de/teaching/sar/SS20/)
- [SATソルバ・SMTソルバの技術と応用](https://www.jstage.jst.go.jp/article/jssst/27/3/27_3_3_24/_pdf)
- [TokyoWesterns の z3 解説](https://wiki.mma.club.uec.ac.jp/CTF/Toolkit/z3py)

SAT / SMT ソルバの入力形式は DIMACS / SMT-LIBv2 を用います。

この資料は CC0 ライセンスです。