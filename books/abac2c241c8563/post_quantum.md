---
title: "耐量子暗号"
---

NIST による選考

- 格子暗号 (Lattice-based cryptography)
- 符号暗号 (Code-based cryptography)
- 同種写像暗号 (Isogeny-based cryptography)
- 多変数多項式暗号 (Multivariate polynomial cryptography)
- ハッシュ関数署名 (Hash-based signature)

NIST 第 3 ラウンドが終了し、最終第 4 ラウンドの候補としては次のようになっています。

| 公開鍵暗号/鍵共有 | 署名 |
| :---- | :--- |
| CRYSTALS-KYBER (格子暗号) <br> BIKE (符号暗号) <br> Classic McElience (符号暗号) <br> HQC (符号暗号) <br> SIKE (同種写像暗号) | CRYSTALS-Dilithium (格子暗号) <br> FALCON (格子暗号) <br> SPHINCS+ (ハッシュ関数署名) |

私もすべて理解している訳ではないので知っていることだけを紹介します。

> なんでもは知らないわよ。知っていることだけ。

## 格子暗号

第三章で基底簡約することにより SVP が解き易くなるということで LLL 基底簡約アルゴリズムまでやりましたが、暗号で使われる問題である CVP を解く為にはより強い基底簡約が必要です。まずはそれらを紹介し CVP の解き方、LWE 格子暗号とその派生について学びます。

### LLL 基底簡約のその先へ

LLL 基底簡約は

> **HKZ (Hermite-Korkine-Zolotareff) 基底簡約**
> 1. サイズ基底簡約
> 2. すべての $1\leq i\leq n$ に対して $\|\boldsymbol{b}_i^*\| = \lambda_1(\pi_i(L))$ となるように基底ベクトルの交換

> **BKZ (Block Korkine-Zolotareff) 基底簡約**
> 1. サイズ基底簡約
> 2. すべての $1\leq k\leq n-\beta+1$ に対して $\beta$ 次元の格子 $L_{[k,k+\beta-1]} = \lbrace\pi_k(\boldsymbol{b}_k), \pi_k(\boldsymbol{b}_{k+1}), \ldots, \pi_k(\boldsymbol{b}_{k+\beta-1})\rbrace$ の基底が HKZ 基底簡約

BKZ を応用したさらなる基底簡約アルゴリズムは BKZ2.0, RSR, G6K などがあります。参考に元論文を置いておきます。

### 最近ベクトル問題 (Closest Vector Problem)
SVP は原点に最も近い格子点を求める問題でしたが CVP は任意の点に対して最も近い格子点を求める問題となります。

> **Def. CVP; Closest Vector Problem**
>

- (near) SVP
- (near) CVP

> **Kannan's embedding method**
> CVP の目標ベクトル $w$ と解ベクトル $v$ の差 $e = w - v$ のノルムについて $\|e\| < \lambda_1/2$ が成り立つとき SVP を解くことで求まる。

$$
\begin{pmatrix}
  B & \boldsymbol{0} \\
  w & M
\end{pmatrix}
$$

> **Babai’s Algorithm**

$$
\begin{aligned}
  w = \sum a_ib_i
\end{aligned}
$$


### LWE格子暗号

機械学習理論から派生した求解困難な問題で、有限体 $\mathbb{F}_q$ 上の秘密ベクトル $\boldsymbol{s} \in \mathbb{F}_q^n$ に関するランダムな連立線形「近似」方程式が与えられたとき、その秘密ベクトルを復元する問題である。

> **LWE 問題**
>

### CRYSTALS-KYBER

### NTRU
CRYSTALS と引き合いとして出されるのが NTT が開発した NTRU 暗号です。
簡略化の為、多項式環の剰余環 $R = \mathbb{Z}[x]/(x^n-1)$、$R_p = \mathbb{F}_p[x]/(x^n-1)$ とおきます。

> **Def. Ternary Polynomials**
> 係数が $\pm 1$ の多項式の集合である。具体的には $\mathcal{T}(d_{+1}, d_{-1})$ を $d_{+1}$ 個の $+1$ 係数と $d_{-1}$ 個の $-1$ 係数のある多項式の集合とする。

$$
\begin{aligned}
f(x) & = 1+3x+4x^2+5x^3+2x^5 \in R_p \\
Lift(f) & = 1+3x−3x^2−2x^3+2x^5 \in R
\end{aligned}
$$

## 符号暗号

## 同種写像暗号
超特異同種写像ディフィー・ヘルマン鍵共有 (SIDH / SIKE)
CSIDH

### SIKE
$p = w_A^{e_A}w_b^{e_B}f \pm 1$
$\mathbb{F}_{p^2}$ 上の超特異楕円曲線 $E$
位数 $w_A^{e_A}$ である点 $P_A, Q_A$ と位数 $w_B^{e_B}$ である点 $P_B, Q_B$

```python
a = 110
b = 67
p = 2^a*3^b - 1
Fp2<I> = GF(p, 2)
assert I^2 eq -1
R<x> = PolynomialRing(Fp2)
E = EllipticCurve(x^3 + 6*x^2 + x)
```

## 多変数多項式暗号 (Multivariate polynomial cryptography)


## ハッシュ関数署名 (Hash-based signature)
SPHINCS+

## 参考文献
- Post-Quantum Cryptography
- 耐量子計算機暗号