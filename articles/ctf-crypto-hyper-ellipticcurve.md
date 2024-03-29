---
title: "【CTF 探訪記】楕円曲線応用"
emoji: "🗂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

CM Method

Weil pairing の計算は比較的遅くて、この上位互換として Tate-Lichtenbaum Pairing というペアリングがあります。

> **Def. Tate-Lichtenbaum Pairing**
> 楕円曲線 $E/K$ に対し 整数 $m$
> $P\in E(K)$ と $T\in E(K)[m]$ $Q\in E(\overline{K})$ $mQ = P$
>
> $$
G_{\overline{K}/K}\to \mu_m \\
\sigma\mapsto e_m(Q^\sigma - Q, T)
$$
>
> $$
\begin{aligned}
& \tau: \frac{E(K)}{mE(K)}\times E(K)[m] \to \frac{K^\times}{(K^\times)^m} \\
& \tau(P, T) = f(P)\bmod (K^\times)^m
\end{aligned}
$$
>
> $$
K^\times/(K^\times)^m\to H^1(G_{\overline{K}/K}, \mu_m)
$$

> **Prop.**
> Tate-Lichtenbaum Pairing はペアリングである

**Proof.**
$\xi(\sigma) = e_m(Q^\sigma - Q, T)$ は $\xi: G_{\overline{K}/K}\to\mu_m$

$$
\begin{aligned}
\xi(\sigma\tau)
\end{aligned}
$$

$$
e_n(Q^\sigma - Q, T) = \frac{\sqrt[n]{\alpha}^\sigma}{\sqrt[n]{\alpha}} \qquad \forall \sigma\in G_{\overline{K}/K} \\
\tau(P, T) = \alpha \bmod (K^\times)^n
$$

## 同種写像暗号

> **同種写像の計算困難性**
> 楕円曲線 $E$ と同種写像 $\phi$ を生成し、公開情報 $(E, \phi(E))$ から秘密情報 $(E, \phi)$ を求めるのが計算量的に困難であるという仮定

ねじれ点のとりやすさから同種写像暗号では超特異曲線を利用します。
通常曲線を使った方式もあるがどれも非効率
モントゴメリーモデル

データサイズが小さい
$\mathbb{F}_{p^2}$ 上の超特異楕円曲線 $E$
$\#E_0(\mathbb{F}_{p^2}) = (p + 1)^2$

### 同種写像

> **Def. 同種写像**
> 楕円曲線 $E_1, E_2$ に対して有理多項式で表せる群準同型写像 $f: E_1 → E_2$ を同種写像とよぶ。
> 有理多項式の次数が $l$ のとき $l$-同種写像という。
> また有理多項式 $\hat{f}: E_2\to E_1$ 双対同種写像という。

同種写像の変換に対して位数が不変なのと同様に $j$-不変量も不変となります。

> **Thm. Tate の定理**
> 楕円曲線 $E_1, E_2$ に対して同種と $\#E_1 = \#E_2$ は同値

> **Def. $j$-不変量**
>
> $$
j_E = 1728\frac{4a^3}{4a^3 + 27b^2}
$$

楕円曲線同士の同型写像を具体的に計算する方法

https://eprint.iacr.org/2011/430.pdf

> **Thm. Vélu の公式**
> 楕円曲線 $E$, $E'$ に分離的な同型写像 $\phi: E\to E'$ が張られているとき、準同型定理より $E' = E/\ker\phi$ となります。$\phi$ の核 $F = \ker\phi$ の分割を $F = \lbrace\mathcal{O}\rbrace\cup F^+ \cup F^-$ とすると
>
> $$
E/F: y^2 = x^3 + (a - 5v)x + (b - 7w)
$$
>
> $$
\phi(x, y) = \left(x + \sum_{P\in F^+}\frac{v_P}{x - x_P} - \frac{u_P}{(x - x_P)^2}, y - \sum_{P\in F^+}\frac{2u_Py}{(x - x_P)^3} - v_P\frac{y - y_P - g_P^xg_P^y}{(x - x_P)^2}\right)
$$

**Proof.**
Laurent 展開
$\Box$

モントゴメリーモデルでは Costello-Hisil の公式

### 同種写像暗号

超特異同種写像 Diffie-Hellman 鍵共有
SIDH (Supersingular Isogeny Diffie-Hellman key exchange)
$p = l_A^{e_A}l_B^{e_B} - 1$
CSIDH (Commutative Supersingular Isogeny Diffie-Hellman)
$p = l_1l_2\cdots l_n - 1$
$n$-同種写像の計算量は $\mathcal{O}(n)$ 掛かります。次数 $2^{256}$ 程度の同種写像計算をする
この楕円曲線を生成するのに 1700 CPU時間使ったらしいです。

SIKE (Supersingular Isogeny Key Encapsulation)

$$
y^2 = x^3 + x \qquad y^2 = x^3 + 6x^2 + x
$$

```python
# SIKEp377
p = 2^191 * 3^117 - 1
# SIKEp546
p = 2^273 * 3^172 - 1
# SIKEp697
p = 2^356 * 3^215 - 1

Fp2<I> = GF(p, 2)
assert I^2 == -1
R<x> = PolynomialRing(Fp2)
E = EllipticCurve(x^3 + x)
E = EllipticCurve(x^3 + 6*x^2 + x)
```

> **Thm. Kani's theorem**

https://eprint.iacr.org/2022/975

[Kani for begginers](https://www.math.auckland.ac.nz/~sgal018/kani.pdf)

https://ellipticnews.wordpress.com/2022/08/12/attacks-on-sidh-sike/
https://github.com/GiacomoPope/Castryck-Decru-SageMath

## 超楕円曲線
ヤコビ多様体
Mumford 表現

$$
y^2 + h(x)y = f(x)
$$

> **Def. 超楕円曲線とその群**
> $K$ を体、$f(x)\in K[x]$ を $\deg f = 2g + 1$ の重根を持たないモニック多項式としたときに $C: y^2 = f(x)$ を種数 $g$ の超楕円曲線という。
>
> $$
C: y^2 = x^{2g+1} + a_{2g}x^{2g} + \cdots + a_1x + a_0
$$
>
> これを満足する $P = (x, y)$ と唯一の無限遠点 $P_\infty$ を合わせて $C$ 上の点といい $P\in C$ とかく。

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

楕円曲線の場合と同様に超楕円曲線の位数について次のような定理が知られています。

> **Thm. 超楕円曲線の位数の範囲**
>
> $$
(\sqrt{p}-1)^{2g} \leq \#\mathcal{J}_C(\mathbb{F}_p) \leq (\sqrt{p}+1)^{2g}
$$

$\#\mathcal{J}_C(\mathbb{F}_p) \approx p^g$

同様に超楕円曲線にもその位数に関係するフロベニウス写像の特性多項式があり、$l$ 等分多項式を用いて $l$ 等分点を生成して Schoof のアルゴリズムを適用すれば位数が求まります。


被約因子

$$
\mathcal{D} = \sum_im_iP_i - \left(\sum_im_i\right)P_\infty
$$

$$
\mathcal{D} = \begin{cases}
0 \\
P_1 - P_\infty \\
2P_1 - 2P_\infty \\
P_1 + P_2 - 2P_\infty
\end{cases}
$$

## 楕円関数と楕円曲線と楕円積分の関係

楕円曲線上の点全体に加えてその曲線上に存在すると考えたいきわめて重要な無限遠点と呼ばれるものがあります。これは複素関数論において、複素平面に無限遠点を添加してリーマン球面を形成することと同じようなものです。

このことを正確に扱う為には体 $k$ 上の 3 次元アフィン空間 $\mathbb{A}_k^3$ の同値類として定義される射影平面 $\mathbb{P}_k^2$ の元 $(x:y:z)$ を座標と定義しますが、本質的ではないので詳細は省きます。 $(x/z, y/z) := (x:y:z)$ と対応すると考えればよいです。ここで無限遠点 $\mathcal{O}$ を $(1:0:0)$ と定義します。

**Proof.**
まず楕円関数を定義する。

> **Def. 楕円関数**
> 複素平面上の格子 $L$ を周期とする関数を楕円関数という。
>
> $$
\forall z\in\mathbb{C}, \forall \omega\in L, f(z + \omega) = f(z)
$$

ところで Weierstrass の $\wp$ 関数というものがある。

ある線形独立な複素数 $\omega_1, \omega_2\in \mathbb{C}$ に対し、格子 $L = \mathbb{Z}\omega_1 + \mathbb{Z}\omega_2$ を構成する。このとき Weierstrass の $\wp$ 関数を次のように定義する。

$$
\wp(z) = \wp(z, L) = \frac{1}{z^2} + \sum_{\substack{\omega\in L \\ \omega \neq 0}}\left(\frac{1}{(z - \omega)^2} - \frac{1}{\omega^2}\right)
$$

また $z$ に関して微分すると次のようになる。

$$
\begin{aligned}
\wp'(z) & = - 2\sum_{\omega\in L}\frac{1}{(z - \omega)^3} \\
\end{aligned}
$$

このとき $\wp(z), \wp'(z)$ が収束することを示す。総和の中身は $\omega \gg z$ において次のように近似できる。

$$
\begin{aligned}
\left|\frac{1}{(z - \omega)^2} - \frac{1}{\omega^2}\right| & = \left|\frac{z^2 - 2\omega z}{(z - \omega)^2\omega^2} \right| \approx \frac{1}{|\omega|^3} \\
\left|\frac{1}{(z - \omega)^3}\right| & \approx \frac{1}{|\omega|^3}
\end{aligned}
$$

これより総和を取っても $1/|\omega|$ 程度であるから $\wp(z), \wp'(z)$ は絶対収束することが分かる。また $\wp(z), \wp'(z)$ は総和を取っているので楕円関数である。
ここで色々計算すると次のような関係式が成り立つ。

$$
(\wp'(z))^2 = 4\wp^3 - 60\bigg(\sum_{\substack{\omega\in L \\ \omega \neq 0}}\frac{1}{\omega^4}\bigg)\wp - 140\bigg(\sum_{\substack{\omega\in L \\ \omega \neq 0}}\frac{1}{\omega^6}\bigg)
$$

これらの係数が収束することは明らかなので $g_2, g_3$ とおく。ここで $(\wp(z), \wp'(z))$ を平面上の点と見立てた曲線は (非特異な) 楕円曲線となる。

$$
y^2 = 4x^3 - g_2x - g_3
$$


[Isogenies of Elliptic Curves](https://www.math.auckland.ac.nz/~sgal018/crypto-book/ch25.pdf)
[CHOOSING THE CORRECT ELLIPTIC CURVE IN THE CM METHOD](https://eprint.iacr.org/2007/253.pdf)
