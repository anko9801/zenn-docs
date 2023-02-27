---
title: "耐量子暗号"
---

## 耐量子暗号
様々な暗号があります。
ラウンド 4

- 格子暗号 (Lattice Based Cryptography)
- 符号暗号 (Code Based Cryptography)
- 同種写像暗号 (Isogeny Based Cryptography)
- 多変数多項式暗号
- 共通鍵暗号ベース署名
- ハッシュ関数署名

## 格子暗号

> **HKZ (Hermite-Korkine-Zolotareff) 基底簡約**
> 1. サイズ基底簡約
> 2. すべての $1\leq i\leq n$ に対して $\|\boldsymbol{b}_i^*\| = \lambda_1(\pi_i(L))$ となるように基底ベクトルの交換

> **BKZ (Block Korkine-Zolotareff) 基底簡約**
> 1. サイズ基底簡約
> 2. すべての $1\leq k\leq n-\beta+1$ に対して $\beta$ 次元の格子 $L_{[k,k+\beta-1]} = \lbrace\pi_k(\boldsymbol{b}_k), \pi_k(\boldsymbol{b}_{k+1}), \ldots, \pi_k(\boldsymbol{b}_{k+\beta-1})\rbrace$ の基底が HKZ 基底簡約

### CVP; Closest Vector Problem

- SVP
- near SVP
- CVP
- near CVP

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

## 符号暗号

## 同種写像暗号
超特異同種写像ディフィー・ヘルマン鍵共有 (SIDH / SIKE)
CSIDH

### SIDH/SIKE
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

### CSIDH

## 参考文献
- Post-Quantum Cryptography
- 耐量子計算機暗号