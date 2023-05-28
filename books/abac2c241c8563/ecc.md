---
title: "楕円曲線暗号への攻撃"
---

楕円曲線の理論は群環体、ガロア理論、可換環論、代数幾何学と理解した先になるので付け焼き刃程度しか書けないです。網羅的かつ短くまとめるのが難しいので削れるところは曖昧に書きます、許してください。

## 楕円曲線

> **Def. 楕円曲線**
> $K$ を体、 $f(x) \in K[x]$ を 3 次方程式としたときに関数 $y^2 = f(x)$ を楕円曲線 $E/K$ という。また $K$ の標数が $2, 3$ でないとき $x, y$ の線形変換によって 2 次項を消すことができ、$a, b\in K$ を用いて次のように書ける。
>
> $$
E/K: y^2 = x^3 + ax + b
$$

ここにグラフ

楕円曲線上の点全体に加えてその曲線上に存在すると考えたいきわめて重要な無限遠点と呼ばれるものがあります。これは複素関数論において、複素平面に無限遠点を添加してリーマン球面を形成することと同じようなものです。

このことを正確に扱う為には体 $k$ 上の 3 次元アフィン空間 $\mathbb{A}_k^3$ の同値類として定義される射影平面 $\mathbb{P}_k^2$ の元 $(x:y:z)$ を座標と定義しますが、本質的ではないので詳細は省きます。 $(x/z, y/z) := (x:y:z)$ と対応すると考えればよいです。ここで無限遠点 $\mathcal{O}$ を $(1:0:0)$ と定義します。

> **Def. 楕円曲線上の和**
> 楕円曲線 $E$ 上の点同士の演算 $+: E\times E \to E$ の群 $(E, +)$ を定義する。まず単位元を無限遠点 $\mathcal{O}$、点 $P(x, y)$ の逆元を $-P = (x, -y)$ とする。このとき $P(x_1, y_1), Q(x_2, y_2)$ に対して $R(x_3, y_3) = P + Q$ を次のように定義する。
>
> $$
\begin{aligned}
x_3 &= \lambda^2 - x_1 - x_2 \\
y_3 &= \lambda(x_1 - x_3) - y_1 \\
\lambda &=
\begin{dcases}
\frac{y_2 - y_1}{x_2 - x_1} \quad (P \neq Q) \\
\frac{3x_1^2 + a}{2y_1} \quad (P = Q)
\end{dcases}
\end{aligned}
$$
>
> これは交換法則、可換則を満たし、可換群となる。

ここに図

特に実数体 $\mathbb{R}$ 上の楕円曲線のグラフ上で見ると、点 $P, Q$ に対して直線 $PQ$ ($P = Q$ のとき点 $P$ での接線) と曲線との交点について $y$ 座標を符号反転した点が $P + Q$ となります。

https://andrea.corbellini.name/ecc/interactive/reals-add.html

交換法則が成り立つので $nP := \overbrace{P + \cdots + P}^n$ と定義します。

ここに具体例

ここに実装

ここでは特に有理数体 $\mathbb{Q}$ または有限体 $\mathbb{F}_q$ のときについて考える。

torsion point

まずこの群の位数を求める為に群構造を明らかにする。

> **Prop. 楕円曲線の群構造**
> $m, n$ を整数として $E(\mathbb{F}_p) \cong \mathbb{Z}/m\mathbb{Z}\times\mathbb{Z}/n\mathbb{Z}$ となる。

**Proof.**

ところで Weierstrass の $\wp$ 関数というものがあり、次のように定義されます。

$L = \mathbb{Z}\omega_1 + \mathbb{Z}\omega_2$

$$
\begin{aligned}
\wp(u) & = \frac{1}{u^2} + \sum_{\substack{\omega\in L \\ \omega \neq 0}}\left(\frac{1}{(u - \omega)^2} - \frac{1}{\omega^2}\right) \\
\wp'(u) & = - \sum_{\omega\in L}\frac{1}{(u - \omega)^3} \\
(\wp')^2 & = 4\wp^3 - 60\sum_{\substack{\omega\in L \\ \omega \neq 0}}\frac{1}{\omega^4}\wp - 140\sum_{\substack{\omega\in L \\ \omega \neq 0}}\frac{1}{\omega^6}
\end{aligned}
$$

> **Thm. Hasse の定理**
> 楕円曲線 $E/\mathbb{F}_q$ の位数 $\#E(\mathbb{F}_q)$ について次の条件で押さえられる。
>
> $$
|\#E(\mathbb{F}_q) - (q+1)|\leq 2\sqrt{q}
$$

詳しい証明は [Hasse's Theorem on Elliptic Curves](https://fse.studenttheses.ub.rug.nl/10999/1/opzet.pdf) に書いてあります。ここでは標数が 5 以上のときについての筋書きだけ記します。

**Proof.**
まずベースとなる楕円曲線 $E/\mathbb{F}_q$ を次のように定義する。

$$
E/\mathbb{F}_q: y^2 = x^3 + ax + b
$$

次に $s^2 = f(t) = t^3 + at + b$ とおき、有理関数体上の楕円曲線 $E^{tw}/\mathbb{F}_q(t)$ を定義する。

$$
E^{tw}/\mathbb{F}_q(t): f(t)y^2 = x^3 + ax + b
$$

これらは同型写像 $\phi: E/\mathbb{F}_q\ni (x, y)\mapsto (x, y/s) \in E^{tw}/\mathbb{F}_q(t)$ が存在するので同型 $E/\mathbb{F}_q \cong E^{tw}/\mathbb{F}_q(t)$ となる。ここで体を拡張した楕円曲線 $E/\mathbb{F}_q(t, s)$ における点 $(t, s)$ とそのフロベニウス写像の像 $(t^q, s^q)$ を同型写像 $\phi$ で移した点を $Q, P_0\in E^{tw}/\mathbb{F}_q(t)$ とおく。

$$
\begin{aligned}
Q & = (t, 1) \\
P_0 & = (t^q, s^{q-1}) = (t^q, (t^3 + at + b)^{(q-1)/2})
\end{aligned}
$$

これらを用いて点 $(x_n, y_n) = P_n = P_0 + nQ$ を生成する。このとき $x_n = f_n/g_n \in \mathbb{F}_q(t)$ と書けて $d_n$ を次のように定義する。

$$
d_n = \begin{cases}
0 & \mathrm{if}\ P_n = \mathcal{O} \\
\deg(f_n) & \mathrm{otherwise}
\end{cases}
$$

例えば $d_0 = \deg(f_0) = \deg(t^q) = q$ となる。このとき色々計算すると次の漸化式が成り立つことが分かる。

$$
d_{n-1} + d_{n+1} = 2d_n + 2
$$

また色々計算すると $d_{-1} = \#E(\mathbb{F}_q)$ と分かる。この $d_n$ に関する 3 つの式から $d_n$ が求まる。

$$
d_n = n^2 - (\#E(\mathbb{F}_q) - (q + 1))n + q
$$

$d_n$ は $n$ に関する二次方程式となり、その判別式 $D$ は次のようになる。

$$
D = (\#E(\mathbb{F}_q) - (q + 1))^2 - 4q
$$

$D > 0$ とすると 2 つの解 $\alpha$, $\beta$ が存在するが、整数 $n$ に対して $d_n \geq 0$ であるから 2 つの解の差は多くとも 1 である。また $D$ は整数であるから $D = (\beta - \alpha)^2 = 1$ となる。これより 2 つの解は $k$, $k+1$ と書けて $d_n = n^2 - (2k + 1)n + k(k+1)$ より方程式を比較して $q = k(k+1)$ となる。ただ $q$ は奇数に対し、 $k(k + 1)$ は偶数であるから不適である。
よって $D \leq 0$、つまり標数 5 以上の Hasse の定理が示された。

$$
|\#E(\mathbb{F}_q) - (q+1)|\leq 2\sqrt{q}
$$
$\Box$

例えば


> **Schoof のアルゴリズム**
> 楕円曲線 $E/\mathbb{F}_p$ の位数を $O(\log^8p)$ で求められる。

Hasse の定理より

$$
q+1-2\sqrt{q}\leq|E(\mathbb{F}_q)|\leq q+1+2\sqrt{q}
$$

$|E(\mathbb{F}_q)|=q+1+t$ とおける。$\mathbb{F}_q$ のフロベニウス写像 $\sigma$ のトレース $t$ を計算できれば位数が求まる。しかし $t$ は $2\sqrt{q}$ のオーダーであるため、直接計算できない。そこで素数 $l$ を剰余にとってそれぞれの $t$ の値を求め、中国剰余定理によって $t$ を求める。

具体的には特性多項式の $t$ に $0,\ldots,\frac{l-1}{2}$ の値を代入して確かめる。

$$
\begin{aligned}
\sigma_q^2-t\sigma_q+q &= 0 \\
\pm[t_l]\circ\sigma_q &= \sigma_q^2+[q_l]
\end{aligned}
$$

1つの $l$ 等分点 $P$ に対して成り立てばすべての $l$ 等分点に対して成り立つ。


Schoof のアルゴリズムはモジュラー多項式によって $O(\log^8p)$ から $O(\log^6p)$ へ高速化できるのですが、本筋とズレるので気になる方は楕円曲線について学習された上、論文などを参照してください。

上で示した楕円曲線は限定的な定義でこれより一般の楕円曲線や特殊な曲線として Weierstrass, Montgomery, twisted Edwards, Hessian などなどさまざまな曲線がありますが、あまり重要ではないので詳しくは CM の後で！


## 楕円曲線暗号

楕円曲線暗号 (ECC) はRSA暗号と同時期に開発された暗号で1985年頃に Victor S. Miller と Neal Koblitz が同時期かつ独立に発明しました(ちなみにMiller-Rabin素数判定法のMillerはGary L. Millerで別人です)。特徴としては RSA 暗号よりも純粋に強い暗号であることや鍵長が短いことなどが挙げられます。

さて、ここでこの楕円曲線上の加法を用いた次のような問題を作れます。

> **楕円曲線上の離散対数問題 (ECDLP : Elliptic Curve Discrete Logarithm Problem)**
> 楕円曲線上の点 $P, Q$ に $Q=dP$ という関係があるとき $d$ を求めよ。

つまり楕円曲線の世界で「割り算」をしなさいという問題です。

実はこの問題はとても難しく、これを解く効率的なアルゴリズムは現在見つかっていません。この ECDLP を利用して暗号の形にしたものが楕円曲線暗号です。

### ECDH (Elliptic Curve Diffie–Hellman key exchange)

暗号通信をする為に使われる暗号プロトコルです。

Alice と Bob は AES などの共通鍵暗号を用いて暗号通信しようとしていますが、始めに2人だけの秘密である共有鍵が必要です。しかしそれを直接共有してしまうと、第三者から鍵を盗聴されて通信を覗き見られてしまいます。そこで暗号を用いることで鍵を直接共有することなく共有鍵を構築することができます。この手法をディフィーヘルマン鍵共有 (DH) と呼び、DH の中でも ECDLP を安全性根拠とする DH を楕円曲線ディフィーヘルマン鍵共有 (ECDH) と呼びます。

具体的には次のような方法で ECDH を実現します。

ここに図

> **楕円曲線ディフィーヘルマン鍵共有 (ECDH)**
> 1. セットアップ: 楕円曲線 $E/\mathbb{F}_p$ とベースポイント $P\in E/\mathbb{F}_p$ を共有する
> 2. 鍵生成: Alice と Bob はそれぞれ疑似乱数 $d_A, d_B$ を生成し、$d_A, d_B$ を秘密鍵、$Q_A = d_AP, Q_B = d_BP$ を公開鍵として公開する
> 3. 鍵交換: Alice と Bob は自分の秘密鍵と相手の公開鍵を掛けると $S = d_Ad_BP = d_AQ_B = d_BQ_A$ となり、$S$ の $x$ 座標をハッシュ化したものが Alice と Bob のみが知る共通鍵となる

このように攻撃者は $(G, dG)$ が分かっても ECDLP が解けない為に $d$ が分からず、安全に共通鍵を共有することができます。

ここに実装

暗号標準を定める国際機関によって楕円曲線が
https://neuromancer.sk/std/

## ECDLP

DLP で書いた手法を用いることで解くことができます。Pollard-rho 法や GSBS 法は簡単に応用が効くので飛ばします。あとよく分かってないですけど数体ふるい法は ECDLP 上では有効ではないらしいので Pohlig-Hellman と Index Calculus Algorithm とその派生を紹介します。詳しい方ここらへん教えてください。

### Pohlig-Hellman

中国剰余定理を用いて大きな群を複数の小さな群の直積に分けます。楕円曲線暗号の楕円曲線の位数は細かく素因数分解できることが多いので有効な手法になります。

楕円曲線の位数 $\#E = p_1^{e_1}p_2^{e_2}\ldots p_k^{e_k}$ と素因数分解して $Q = dP$ となるとき次のように $z_i$ を定義する。

$$
d = z_0+z_1p_i+z_2p_i^2+\ldots+z_{e_i−1}p_i^{e_i−1} \quad \pmod{p_i^{e_i}} \\
$$

これより次の関係式が成り立つ。

$$
\frac{\#E}{p_i}Q = z_0\left(\frac{\#E}{p_i}P\right)
$$

この $z_0$ は ECDLP を用いて $\mathcal{O}(\sqrt{p_i})$ で求まります。次に $z_0,\ldots,z_{j-1}$ を知っているときに $z_j$ を計算する為に次のように変形します。

$$
\begin{aligned}
\frac{\#E}{p_i^{j+1}}Q & = (z_0+\cdots+z_{j}p_i^{j})\left(\frac{\#E}{p_i^{j+1}}P\right) \\
\frac{\#E}{p_i^{j+1}}Q & - (z_0+\cdots+z_{j−1}p_i^{j−1})\left(\frac{\#E}{p_i^{j+1}}P\right) = z_{j}\left(\frac{\#E}{p_i}P\right)
\end{aligned}
$$

これより ECDLP を解くことで $z_j$ が求まります。

```python
fact = factor(G.order())
ord = int(G.order())
dlogs = []
for p, e in fact:
    t = ord // p ^ e
    dlog = discrete_log(t * Q, t * G, operation="+")
    dlogs += [dlog]

print(crt(dlogs, primes))
```

### Index Calculus Algorithm
種数が大きい超楕円曲線上の ECDLP では Index Calculus Algorithm を応用することができます。次の Gaudry アルゴリズム

$B$ 以下の素数に代えて、次数 $s$ 以下の多項式の因子基底を用意して Mumford 表現に現れる多項式 $U$ が因子基底の要素に分解される場合に対して $B = \lbrace P_j\in C(\mathbb{F}_p)\setminus P_\infty\mid X(P_j)\neq X(P_i) for i \neq j\rbrace$

$$
\begin{aligned}
r_i\mathcal{D}_b & = \sum_{j=1}^n e_{ij}P_j^{e_{ij}} - mP_\infty \\
\begin{pmatrix}
r_i\mathcal{D}_b \\
\vdots \\
r_i\mathcal{D}_b
\end{pmatrix}
& = \begin{pmatrix}
e_{11} & \cdots & e_{m1} \\
\vdots & \ddots & \vdots \\
e_{1n} & \cdots & e_{mn}
\end{pmatrix}
\begin{pmatrix}
\log_{\mathcal{D}_b} P_i \\
\vdots \\
\log_{\mathcal{D}_b} P_i
\end{pmatrix} \\
\mathcal{D}_a + r\mathcal{D}_b & = \prod_{j=1}^n s_jP_j - mP_\infty \\
x = \log_{\mathcal{D}_b}\mathcal{D}_a & = \sum_{j=1}^ns_j\log_{\mathcal{D}_b}P_j - r \bmod N
\end{aligned}
$$

$\mathcal{O}(g!g^3p(\log p)^3 + g^3p^2(\log p)^2)$

### GHS-Weil descent 攻撃
楕円曲線の $\mathbb{F}_{p^k}$ 有理点群 $E(\mathbb{F}_{p^k})$ を種数 $g\geq k$ の代数曲線 $C$ の Jacobian の有理点群 $\mathcal{J}_C(\mathbb{F}_p)$ に埋め込み、 $\mathcal{J}_C(\mathbb{F}_p)$ 上で Gaudry アルゴリズムで解く

## 攻撃手法

この ECDLP を解くことができれば ECDH を含め、様々な楕円曲線暗号を解くことができます。さて主に攻撃対象となる楕円曲線暗号は以下のようなものがあります。

| アンチケース | 攻撃名   | 方法 |
| ---- | --- | ---- |
| なし | ECDLP | 単純に ECDLP を解く |
| 位数が Smooth number $\#E/\mathbb{F}_p = p_1^{e_1}p_2^{e_2}\ldots p_k^{e_k}$ | Pohlig Hellman Attack | 位数 $p_i$ の小さな ECDLP に分解できる |
| Anomalous な曲線 $\#E/\mathbb{F}_p = p$ | SSSA Attack | $\mathbb{F}_p^+$ 上の DLP に帰着できる |
| Supersingular な曲線 $\#E/\mathbb{F}_p = p+1$ | MOV / FR Reduction | 埋め込み次数 $k$ を用いて $\mathbb{F}_{p^k}^\times$ 上の DLP に帰着できる |
| Singular な曲線 $\Delta(E/\mathbb{F}_p) = 0$ | Singular Curve Point Decompression Attack | $\mathbb{F}_p^+$ や $\mathbb{F}_p^\times, \mathbb{F}_{p^2}^\times$ 上の DLP に帰着できる |
| 楕円曲線上に存在しない点や位数の少ない点を指定できる | Invalid Curve Attack / Small-Subgroup Attack | さまざまな少ない位数の点を収集して中国剰余定理 |

### Supersingular な曲線を用いてはならない (MOV/FR Reduction)
楕円曲線が超特異 supersingular という性質を持つとき、ペアリングを用いて有限体上の DLP に帰着できるという方法です。

$$
y^2 = x^3 + (1 - b)x + b
$$

> **Def. 双線形 (bilinear)**
> 任意の $a, b\in \mathbb{F}_q^\times$ と $P, Q\in E$ について次を満たすとき $e$ は双線形であるという。
>
> $$
e(aP, bQ) = e(P, Q)^{ab}
$$

> **Weil pairing**
> 楕円曲線の等分点群と同型な部分群が有限体の乗法群に含まれるためには有限体を拡大する必要がある。
>
> $$
e: E[m]\times E[m] \to \mu_m\subseteq\mathbb{F}_{q^d}^\times
$$
>
> 必要となる最小の拡大次数 $d$ を埋め込み次数という。

FFDLP に落とし込める
埋め込み次数が高いと ECDLP の方が計算量が小さくなってしまうので

> **超特異曲線の埋め込み次数**
> 超特異楕円曲線の埋め込み次数は $6$ 以下である。

**Proof.**

| $-t$ | $\#E(\mathbb{F}_{q^2})$ | $\#\mathbb{F}_{q^d}^\times$ | $d$ |
|:-:|:-:|:--|:-:|
| $0$ | $p^2 + 1$ | $p^4 - 1 = (p^2 + 1)(p^2 - 1)$ | $4$ |
| $p$ | $p^2 + p + 1$ | $p^3 - 1 = (p - 1)(p^2 + p + 1)$ | $3$ |
| $-p$ | $p^2 - p + 1$ | $p^6 - 1 = (p^3 - 1)(p + 1)(p^2 - p + 1)$ | $6$ |
| $2p$ | $(p + 1)^2$ | $p^2 - 1 = (p + 1)(p - 1)$ | $2$ |
| $-2p$ | $(p - 1)^2$ | $p - 1 = p - 1$ | $1$ |

$e_n(P, Q)$

> **Miller's algorithm**
> 1. $E[n]\subseteq E(\mathbb{F}_{p^k})$ となる最小の $k$ を持ってくる
> 2. 位数 $n$ の $\alpha=e_n(P, Q)$ となるように $Q \in E[n]$ を取ってくる
> 3. $\beta = e_n(dP, Q)$
> 4. $\mathbb{F}_{p^k}^\times$ 上のDLPを $\alpha, \beta$ を用いて解く

MOV Reduction (Menezes-Okamoto-Vanstone Reduction)

> **Tate-pairing**
>
> $$
e: E(\mathbb{F}_p)[l]\times E(\mathbb{F}_{p^2})/lE(\mathbb{F}_{p^2})\to \mathbb{F}_{p^2}^\times/(\mathbb{F}_{p^2}^\times)^l
$$

FR Reduction (Frey-Rück Reduction)
Bilinear-paring

$E(\mathbb{F}_{p^k}^*)\cong\mathbb{Z}_{c_1n_1}\oplus\mathbb{Z}_{c_2n_1}$

```python
def miller(E, P, Q, m):
  from six.moves import map
  """
  Calculate Divisor by Miller's Algorithm
  Args:
    E: The Elliptic Curve
    P: A point over E which has order m
    Q: A point over E which has order m to apply function f_P
    m: The order of P, Q on E
  Returns:
    f_P(Q)
  """
  def h(P, Q, R):
    # if \lambda is infinity
    if (P == Q and P.y == 0) or (P != Q and P.x == Q.x):
      return R.x - P.x
    L = P.line_coeff(Q)
    p = R.y - P.y - L * (R.x - P.x)
    q = R.x + P.x + Q.x - L * L
    return p / q
  if P == Q:
    return 1
  b = map(int, bin(m)[2:])
  next(b)
  f = 1
  T = P
  for i in b:
    f = f * f * h(T, T, Q)
    T = T + T
    if i:
      f = f * h(T, P, Q)
      T = T + P
  return f


def weil_pairing(E, P, Q, m, S=None):
  """
  Calculate Weil Pairing
  Args:
    E: The Elliptic Curve
    P: A point over E which has order m
    Q: A point over E which has order m
    m: The order of P, Q on E
    S: [Optional] A random point on E
  Returns:
    e_m(P, Q)
  """
  if S is None:
    S = E.random_point()
  from ecpy.utils.util import is_enable_native, _native
  from ecpy.fields.ExtendedFiniteField import ExtendedFiniteFieldElement
  if is_enable_native:
    P = _native.EC_elem(E.ec, tuple(P.x), tuple(P.y), tuple(P.z))
    Q = _native.EC_elem(E.ec, tuple(Q.x), tuple(Q.y), tuple(Q.z))
    S = _native.EC_elem(E.ec, tuple(S.x), tuple(S.y), tuple(S.z))
    if E.ec.type == 1:
      t = _native.FF_elem(0)
    elif E.ec.type == 2:
      t = _native.EF_elem(0, 0)
    _native.weil_pairing(t, E.ec, P, Q, S, m)
    if E.ec.type == 1:
      return t.to_python()
    elif E.ec.type == 2:
      t = t.to_python()
      return ExtendedFiniteFieldElement(E.field, t[0], t[1])
  else:
    fpqs = miller(E, P, Q + S, m)
    fps = miller(E, P, S, m)
    fqps = miller(E, Q, P - S, m)
    fqs = miller(E, Q, -S, m)
    return E.field._inv(fps * fqps) * fpqs * fqs


def tate_pairing(E, P, Q, m, k=2):
  """
  Calculate Tate Pairing
  Args:
    E: The Elliptic Curve
    P: A point over E which has order m
    Q: A point over E which has order m
    m: The order of P, Q on E
    k: [Optional] The Embedding Degree of m on E
  """
  from ecpy.utils.util import is_enable_native, _native
  if is_enable_native:
    P = _native.EC_elem(E.ec, tuple(P.x), tuple(P.y), tuple(P.z))
    Q = _native.EC_elem(E.ec, tuple(Q.x), tuple(Q.y), tuple(Q.z))
    if E.ec.type == 1:
      t = _native.FF_elem(0)
    elif E.ec.type == 2:
      t = _native.EF_elem(0, 0)
    _native.tate_pairing(t, E.ec, P, Q, m, k)
    if E.ec.type == 1:
      from ecpy.fields.Zmod import ZmodElement
      return ZmodElement(E.field, t.to_python())
    elif E.ec.type == 2:
      from ecpy.fields.ExtendedFiniteField import ExtendedFiniteFieldElement
      t = t.to_python()
      return ExtendedFiniteFieldElement(E.field, t[0], t[1])
  else:
    f = miller(E, P, Q, m)
    return f ** (((E.field.p ** k) - 1) // m)


def MapToPoint(E, y):
  """
  MapToPoint Function: Given by Boneh-Durfee's ID-based Encryption Paper.
  Args:
    E: The Elliptic Curve
    y: Any Value (should be E.field element)

  Returns:
    Correspond point of y on E
  """
  from ecpy.utils import cubic_root
  x = cubic_root(y**2 - 1)
  Q = E(x, y)
  return 6 * Q


def gen_supersingular_ec(bits=70):
  """
  Generate Super-Singluar Elliptic Curve
  Args:
    bits: The Security Parameter: log_2 p = bits

  Returns:
    A (Super Singular) Elliptic Curve, Extended Finite Field, l
    l is need to calculate Pairing
  """
  from ecpy.fields import ExtendedFiniteField
  from .EllipticCurve import EllipticCurve

  def _next_prime(n):
    from ecpy.util import is_prime
    """
    return next prime of n
    """
    while not is_prime(n):
      n += 1
    return n

  """
  If you have gmpy, use gmpy.next_prime
  in other hand, use slow function
  """
  try:
    from gmpy import next_prime
  except:
    next_prime = _next_prime

  def gen_prime():
    from ecpy.util import is_prime
    from random import randint
    while True:
      p = int(next_prime(randint(2**(bits - 1), 2**bits)))
      if is_prime(p * 6 - 1):
        break
    return p * 6 - 1, p

  p, l = gen_prime()
  F = ExtendedFiniteField(p, "x^2+x+1")
  return EllipticCurve(F, 0, 1), F, l


def find_point_by_order(E, l):
  """
  Find a Elliptic Curve Point P which has order l.
  Args:
    E: The Elliptic Curve
    l: Order of Point on E

  Returns:
    Point on E which has order l.
  """
  i = 3
  while True:
    r = E.get_corresponding_y(i)
    if r != None:
      P = E(i, r)
      if (P * l).is_infinity():
        return P
    i += 1


def symmetric_weil_pairing(E, P, Q, m):
  """
  Symmetric Weil Pairing
  \hat{e}(P, Q) = e(P, \phi(Q)) (\phi is Distortion Map)
  Args:
    E: The Elliptic Curve
    P: A point on E which has order m
    Q: A point on E which has order m
    m: The order of P, Q
  """
  return weil_pairing(E, P, Q.distortion_map(), m)


def symmetric_tate_pairing(E, P, Q, m, k=2):
  """
  Symmetric Tate Pairing
  \hat{e}(P, Q) = e(P, \phi(Q)) (\phi is Distortion Map)
  Args:
    E: The Elliptic Curve
    P: A point on E which has order m
    Q: A point on E which has order m
    m: The order of P, Q
    k: [Optional] The Embedding Degree of m on E
  """
  return tate_pairing(E, P, Q.distortion_map(), m)
```

### Anomalous な曲線を用いてはいけない
Anomalous の楕円曲線では SSSA Attack が有効です。

$$
\lambda_E: E(\mathbb{F}_p)\xrightarrow{u}E(\mathbb{Q}_p)\xrightarrow{\times p}\ker\pi\xrightarrow{Formal \log}p\mathbb{Z}_p\xrightarrow{\bmod{p^2}} p\mathbb{Z}_p/p^2\mathbb{Z}_p\cong \mathbb{F}_p
$$

$\psi(x:y:z) := x/y$

$$
\log_E(t) := t - \frac{a_1}{2}t^2 + \frac{a_1^2 + a_2}{3}t^3 - \frac{a_1^3 + 2a_1a_2 + a_3}{4}t^4 + \cdots
$$

$A := (X_1, Y_1)\in E(\mathbb{Z}/p^2\mathbb{Z})$ $\bmod p$ 写像 $\pi(A) = P$ となる
$(X_{p-1}, Y_{p-1}) := (p-1)A$

$X_{p-1} \neq X_1$ なら

$$
\lambda_E(P) = \left(\frac{X_{p-1} - X_1}{p}\bmod p\right)(Y_{p-1} - Y_1\bmod p)^{-1}
$$

```python
def hensel_lift(curve, P):
  from six.moves import map
  """
  Calculate Lifted Point using Hensel's Lemma
  Args:
    curve: The Elliptic Curve
    P: A point on curve
  Returns:
    The "lifted" Point
  """
  from six.moves import map
  from ecpy.utils import modinv
  x, y, _ = map(int, tuple(P))
  p = curve.field.p
  t = (((x * x * x + curve.a * x + curve.b) - y * y) // p) % p
  t = (t * modinv(2 * y, p)) % p
  return list(map(int, (x, y + (curve.field.p * t))))


def SSSA_Attack(F, E, P, Q):
  """
  Solve ECDLP using SSSA(Semaev-Smart-Satoh-Araki) Attack.
  Args:
    F: The Base Field
    E: The Elliptic Curve
    P: A point on E
    Q: A point on E
  Returns:
    Return x where satisfies Q = xP.
  """
  from .EllipticCurve import EllipticCurve
  from ecpy.fields import QQ, Zmod
  from ecpy.utils.util import modinv, is_enable_native, _native
  A = E.a
  # lP, lQ, ... is "lifted" P, Q, ...
  x1, y1 = hensel_lift(E, P)
  x2, y2 = hensel_lift(E, Q)
  lF = Zmod(F.p ** 2)
  lA = (y2 * y2 - y1 * y1 - (x2 * x2 * x2 - x1 * x1 * x1))
  lA = (lA * modinv(x2 - x1, lF.n)) % lF.n
  lB = (y1 * y1 - x1 * x1 * x1 - A * x1) % lF.n
  if not is_enable_native:
    modulo = F.p**2
    lE = EllipticCurve(lF, lA, lB)
    lP = lE(x1, y1)
    lQ = lE(x2, y2)
    lU = (F.p - 1) * lP
    lV = (F.p - 1) * lQ
    dx1 = ((int(lU.x) - x1) // F.p) % modulo
    dx2 = int(lU.y) - y1
    dy1 = ((int(lV.x) - x2) // F.p) % modulo
    dy2 = int(lV.y) - y2
    m = (dy1 * dx2 * modinv(dx1 * dy2, modulo)) % modulo
    return m % F.p
  else:
    modulo = F.p**2
    base = _native.FF(modulo)
    lE = _native.EC(base, lA, lB)
    lP = _native.EC_elem(lE, x1, y1)
    lQ = _native.EC_elem(lE, x2, y2)
    lU = _native.EC_elem(lE, 0, 1, 0)
    lV = _native.EC_elem(lE, 0, 1, 0)
    lE.mul(lU, lP, F.p - 1)
    lE.mul(lV, lQ, F.p - 1)
    lUx, lUy, lUz = lU.to_python()
    lVx, lVy, lVz = lV.to_python()
    lUx = (lUx * modinv(lUz, modulo)) % modulo
    lUy = (lUy * modinv(lUz, modulo)) % modulo
    lVx = (lVx * modinv(lVz, modulo)) % modulo
    lVy = (lVy * modinv(lVz, modulo)) % modulo
    dx1 = ((lUx - x1) // F.p) % modulo
    dx2 = lUy - y1
    dy1 = ((lVx - x2) // F.p) % modulo
    dy2 = lVy - y2
    m = (dy1 * dx2 * modinv(dx1 * dy2, modulo)) % modulo
    return m % F.p
```

### Singular な曲線を用いてはいけない

Singular な楕円曲線のとき、特異点という特殊な点ができます。その点を軸に ECDLP は FFDLP へ変わってしまうやんね。

> **Def. 特異点**
> ある関数 $f(x, y) = 0$ の特異点とは次を満たす $(X, Y)$ である。
>
> $$
\left.\frac{\partial f}{\partial x}\right|_{(X, Y)} = \left.\frac{\partial f}{\partial y}\right|_{(X, Y)} = 0
$$

このように微分値が不定となる点、グラフ上では関数の曲線が交差している点です。

楕円曲線の曲線は高々 1 回交わることになるので 2 つのタイプに分けられます。1 つは普通に交わるノード、もう 1 つは自分自身と接しながら交わるカスプです。

#### ノード

$y = 0$ 上の特異点が原点 $O(0, 0)$ となるように平行移動させると $y^2 = x^3 + kx^2$ となる。

$(\partial F/\partial x, \partial F/\partial y) = (3x^2 + 2kx, 2y)$ より特異点が原点しかないことがわかります。
このとき $y = \lambda x$ との交点を考えます。
まず接線となる。

$P = (\lambda^2 - k, \lambda(\lambda^2 - k))$

$f: E/\mathbb{F}_p \to \mathbb{F}_p^\times$

$$
\begin{aligned}
f(x,y) & = \frac{y + \sqrt{k}x}{y - \sqrt{k}x} \\
f(\infty) & = 1
\end{aligned}
$$

#### カスプ

どんな尖っている楕円曲線も平行移動や線形変換により $y^2 = x^3$ の形になります。

このとき $y = \lambda x$ との交点は $(\lambda^2, \lambda^3)$ 、接線は $y = 0$ となります。

$f:E/\mathbb{F}_p \to \mathbb{F}_p^+$

$$
f(x,y) = \frac{x}{y} \\
f(\infty) = 0
$$



### Invalid Curve Attack

:::message
**練習問題**
- tiramisu (Google CTF)
:::

## 超楕円曲線
ヤコビ多様体
Mumford 表現

$$
y^2 + h(x)y = f(x)
$$

$$
f(x) = x^{2g+1} + a_{2g}x^{2g} + \cdots + a_1x + a_0 \in\mathbb{F}_p[x] \\
C: y^2 = f(x)
$$

$\deg f = 2g + 1$ と書けるなら虚超楕円曲線と呼び、$g$ を種数

> **加算アルゴリズム**
> まず $d = \gcd(u_1, u_2, v_1 + v_2 + h) = s_1u_1 + s_2u_2 + s_3(v_1 + v_2 + h)$ を計算し、次のように加算を行う。
>
> $$
\begin{aligned}
u & = \frac{u_1u_2}{d^2} \\
v & = \frac{s_1u_1v_2 + s_2u_2v_1 + s_3(v_1v_2 + f)}{d} & \pmod u
\end{aligned}
$$

> **還元アルゴリズム**
> $\deg u > g$ なら次のように計算して $u$ をモニックとする。
>
> $$
\begin{aligned}
u' & = \frac{f - vh - v^2}{u} \\
v' & = - h - v & \pmod{u'}
\end{aligned}
$$

位数

$$
(\sqrt{p}-1)^{2g} \leq \#\mathcal{J}_C(\mathbb{F}_p) \leq (\sqrt{p}+1)^{2g}
$$

$\#\mathcal{J}_C(\mathbb{F}_p) \approx p^g$


## 参考文献
- [Imaginary hyperelliptic curve - Wikipedia](https://en.wikipedia.org/wiki/Imaginary_hyperelliptic_curve)
- https://blog.z.cash/new-snark-curve/ : BLS12-381: New zk-SNARK Elliptic Curve Construction
- Python での高速な実装 fastecdsa

この資料は CC0 ライセンスです。