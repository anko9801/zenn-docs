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
### BKZ (Block Korkine-Zolotareff) 基底簡約

### HKZ (Hermite-Korkine-Zolotareff) 基底簡約

1. サイズ基底簡約
2. 条件に合うように基底ベクトルの交換

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
CSIDH
超特異同種写像ディフィー・ヘルマン鍵共有 (SIDH or SIKE)

## 参考文献
- Post-Quantum Cryptography
- 耐量子計算機暗号