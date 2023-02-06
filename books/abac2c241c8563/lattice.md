---
title: "格子暗号と準同型暗号"
---

### LWE格子暗号

機械学習理論から派生した求解困難な問題で、有限体 $\mathbb{F}_q$ 上の秘密ベクトル $\mathbf{s} \in \mathbb{F}_q^n$ に関するランダムな連立線形「近似」方程式が与えられたとき、その秘密ベクトルを復元する問題である。

## 準同型暗号
格子ベースの暗号で完全準同型暗号
レベルn準同型 = 加法準同型 + n回の乗算準同型
- 乗算回数に制約がある完全準同型
- レベルnはn次方程式を計算できる
- レベル2準同型なら乗算が1度だけ可能

### Unpadded RSA
乗法準同型

$$
\begin{aligned}
\mathcal{E}(m_1)\times\mathcal{E}(m_2) &= m_1^em_2^e & \bmod n \\
&= (m_1m_2)^e & \bmod n \\
&= \mathcal{E}(m_1m_2)
\end{aligned}
$$

### ElGamal暗号
乗法準同型

$$
\begin{aligned}
\mathcal{E}(m_1)\times\mathcal{E}(m_2) &= (g^{r_1},m_1h^{r_1})(g^{r_2},m_2h^{r_2}) \\
&= (g^{r_1+r_2},m_1m_2h^{r_1+r_2}) \\
&= \mathcal{E}(m_1m_2)
\end{aligned}
$$

### Paillier暗号
加法準同型

$$
\begin{aligned}
\mathcal{E}(m_1)\times\mathcal{E}(m_2) &= (g^{m_1}r_1^n)(g^{m_2}r_2^n) & \bmod n^2 \\
&= g^{m_1+m_2}(r_1r_2)^n & \bmod n^2 \\
&= \mathcal{E}(m_1+m_2)
\end{aligned}
$$

### 岡本・内山暗号
加法準同型

### Lifted-ElGamal暗号
レベル2準同型暗号
位数 $p$ の楕円曲線 $E$, と生成元 $P\in E$
秘密鍵 $s\in\mathbb{F}_p$ と公開鍵 $sP$
平文 $m$ に対して乱数 $r$ をとり $c=(mP+rsP, rP)$
$c = (S, T)$ に対して $S-sT = (mP+rsP)-s(rP)=mP$ としDLPを解いて $m$ を得る

加法準同型性

$$
\begin{aligned}
\mathcal{E}(m_1)+\mathcal{E}(m_2) &= (m_1P+r_1sP,r_1P)+(m_2P+r_2sP,r_2P) \\
&= ((m_1+m_2)P, (r_1+r_2)sP, (r_1+r_2)P) \\
&= \mathcal{E}(m_1+m_2)
\end{aligned}
$$

乗法準同型性
$\mathcal{E}(m_1)\times\mathcal{E}(m_2) := (e(S_1, S_2), e(S_1, T_2), e(T_1, S_2), e(T_1, T_2))$

$$
\begin{aligned}
\mathcal{D}(c_1,c_2,c_3,c_4) &= \frac{c_1c_4^{s_1s_2}}{c_2^{s_2}c_3^{s_1}} = \frac{e(S_1,s_2)e(s_1T_1,s_2T_2)}{e(S_1,s_2T_2)e(s_1T_1,S_2)} \\
&=e(S_1-s_1T_1,S_2-s_2T_2) \\
&=e(mP_1,m'P_2)=e(P_1,P_2)^{mm'}
\end{aligned}
$$


[ペアリングベースの効率的なレベル2準同型暗号（SCIS2018） (slideshare.net)](https://www.slideshare.net/herumi/2scis2018?next_slideshow=86572957)


### Gentry
完全準同型
### TFHE
完全準同型

BGV CKKS BFV FHEW CKKS Bootstrapping TFHE