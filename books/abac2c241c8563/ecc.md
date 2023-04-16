---
title: "楕円曲線暗号への攻撃"
---

楕円曲線の理論は群環体、ガロア理論、可換環論、代数幾何学と理解した先になるので付け焼き刃程度しか書けないです。

## 楕円曲線
> **Def. 楕円曲線**
> $K$ を体、 $f(x) \in K[x]$ を 3 次方程式としたときに関数 $y^2 = f(x)$ を楕円曲線という。また $K$ の標数が $2, 3$ でないとき $x, y$ の線形変換によって 2 次項を消すことができ、次のように書ける。
>
> $$
y^2 = x^3 + ax + b
$$

楕円曲線上の点全体に加えてその曲線上に存在すると考えたいきわめて重要な無限遠点と呼ばれるものがある。
これは複素関数論において、複素平面に無限遠点を添加してリーマン球面を形成することと同じようなものである。このことを正確に扱う為に今、射影座標を導入する。

楕円曲線上の点の座標について体 $k$ 上の 3 次元アフィン空間 $\mathbb{A}_k^3$ の同値類として定義される射影平面 $\mathbb{P}_k^2$ の元 $(x,y,z)$ と定義すると、簡潔で綺麗な定義や証明ができますが、本質的ではないので省きます。必要であれば $(x/z, y/z) := (x, y, z)$ と対応すると考えればよいです。

> **Def. 楕円曲線上の和**
> 楕円曲線 $E$ 上の点同士の演算 $+: E\times E \to E$ の群 $(E, +)$ を定義する。まず単位元を無限遠点 $O(1,0,0)$、点 $P(x, y)$ の逆元を $-P = (x, -y)$ とする。このとき $P(x_1, y_1), Q(x_2, y_2)$ に対して $R(x_3, y_3) = P + Q$ を次のように定義する。
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

**Proof.**

特に実数体 $\mathbb{R}$ 上の楕円曲線のグラフ上でこの和の構成を見ると、点 $P, Q$ に対して直線 $PQ$ ($P = Q$ のとき点 $P$ での接線) と曲線との交点について $y$ 座標を符号反転した点が $P + Q$ となります。

> **Thm. Hasse の定理**
> $p$ を素数, $E/\mathbb{F}_p$ を楕円曲線とする。
>
> $$
|\#E/\mathbb{F}_p - (p+1)|\leq 2\sqrt{p}
$$
> ただし、$\#E/\mathbb{F}_p$ で群位数を表す。

**Proof.**

> **Schoof のアルゴリズム**
> 楕円曲線 $E/\mathbb{F}_p$ の位数を求められる。

Hasse-Weil 定理より

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

## 超楕円曲線
ヤコビ多様体
Mumford 表現

$$
y^2 + h(x)y = f(x)
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

1. 公開: 楕円曲線 $E/\mathbb{F}_p$ とベースポイント $P\in E/\mathbb{F}_p$ を共有する
2. 鍵生成: Alice と Bob はそれぞれ疑似乱数 $d_A, d_B$ を生成し、$d_A, d_B$ を秘密鍵、$Q_A = d_AP, Q_B = d_BP$ を公開鍵として公開する
3. 鍵交換: Alice と Bob は自分の秘密鍵と相手の公開鍵を掛けると $S = d_Ad_BP = d_AQ_B = d_BQ_A$ となり、$S$ の $x$ 座標をハッシュ化したものが Alice と Bob のみが知る共通鍵となる

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
| Supersingular な曲線 $\#E/\mathbb{F}_p = p+1$ | MOV/FR Reduction | 埋め込み次数 $k$ を用いて $\mathbb{F}_{p^k}^\times$ 上の DLP に帰着できる |
| Singular な曲線 $\Delta(E/\mathbb{F}_p) = 0$ | Singular Curve Point Decompression Attack | $\mathbb{F}_p^+$ や $\mathbb{F}_p^\times, \mathbb{F}_{p^2}^\times$ 上の DLP に帰着できる |
| 楕円曲線上に存在しない点や位数の少ない点を指定できる | Invalid Curve Attack | 秘密鍵の情報を取り出すことができる |

### MOV/FR Reduction
楕円曲線が超特異 supersingular という性質を持つとき、ペアリングを用いて有限体上の DLP に帰着できるという方法です。

$$
y^2 = x^3 + (1 - b)x + b
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

> **MOV Reduction (Menezes-Okamoto-Vanstone Reduction)**
> 1. $E[n]\subseteq E(\mathbb{F}_{p^k})$ となる最小の $k$ を持ってくる
> 2. 位数 $n$ の $\alpha=e_n(P, Q)$ となるように $Q \in E[n]$ を取ってくる
> 3. $\beta = e_n(dP, Q)$
> 4. $\mathbb{F}_{p^k}^\times$ 上のDLPを $\alpha, \beta$ を用いて解く

> **Tate-pairing**
>
> $$
e: E(\mathbb{F}_p)[l]\times E(\mathbb{F}_{p^2})/lE(\mathbb{F}_{p^2})\to \mathbb{F}_{p^2}^\times/(\mathbb{F}_{p^2}^\times)^l
$$

FR Reduction (Frey-Rück Reduction)
Bilinear-paring

$E(\mathbb{F}_{p^k}^*)\cong\mathbb{Z}_{c_1n_1}\oplus\mathbb{Z}_{c_2n_1}$

### Anomalous な曲線を用いてはいけない
Anomalous の楕円曲線では SSSA Attack が有効です。
$\pi:\mathbb{P}^2(\mathbb{Q}_p) \to \mathbb{P}^2(\mathbb{F}_p)$
$\phi:\ker\pi \to \mathcal{E}(p\mathbb{Z}_p)$

### Singular な曲線を用いてはいけない

Singular な楕円曲線のとき、特異点という特殊な点ができます。その点を軸に ECDLP は FFDLP へ変わってしまうやんね。

> **Def. 特異点**
> ある関数 $F(x, y) = 0$ の特異点とは次を満たす $(X, Y)$ である。
>
> $$
\left.\frac{\partial F}{\partial x}\right|_{(X, Y)} = \left.\frac{\partial F}{\partial y}\right|_{(X, Y)} = 0
$$

このように微分値が不定となる点、自分自身と重なっている点です。楕円曲線では自分自身と 1 回交わることになるので 2 つのタイプに分けられます。1 つは普通に交わるノード、もう 1 つは自分自身と接しながら交わるカスプです。

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

## 参考文献
- [Imaginary hyperelliptic curve - Wikipedia](https://en.wikipedia.org/wiki/Imaginary_hyperelliptic_curve)
- https://blog.z.cash/new-snark-curve/ : BLS12-381: New zk-SNARK Elliptic Curve Construction
- Python での高速な実装 fastecdsa

この資料は CC0 ライセンスです。