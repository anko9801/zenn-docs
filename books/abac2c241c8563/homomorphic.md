---
title: "準同型暗号"
---

古典的な暗号は情報を守ることだけの機能だけしか持たなかったが、現代的な暗号は多機能な属性をもつようになった。その中でも実現すると社会が変わるようなものが完全準同型暗号である。

## 準同型暗号
準同型暗号には次のような種類がある。

| 暗号 | 説明 |
| :--: | :-- |
| 加法準同型暗号 | 暗号化関数 $\mathcal{E}$ が加法準同型 $\mathcal{E}(x + y) = \mathcal{E}(x)\mathcal{E}(y)$ を満たす |
| 乗法準同型暗号 | 暗号化関数 $\mathcal{E}$ が乗法準同型 $\mathcal{E}(xy) = \mathcal{E}(x)\mathcal{E}(y)$ を満たす |
| レベル $n$ 準同型暗号 | 加法準同型かつ $n$ 回までの乗法準同型演算が成り立つ暗号 |
| 完全準同型暗号 | 加法準同型かつ乗法準同型な暗号 |

レベル $n$ 準同型暗号は $n$ 次方程式を計算できる。

準同型暗号ができることによって何ができるか

### 具体例
加法/乗法準同型暗号の具体例を並べる。

乗法準同型暗号
- Unpadded RSA

$$
\begin{aligned}
\mathcal{E}(m_1)\times\mathcal{E}(m_2) &= m_1^em_2^e & \bmod n \\
&= (m_1m_2)^e & \bmod n \\
&= \mathcal{E}(m_1m_2)
\end{aligned}
$$

- ElGamal暗号

$$
\begin{aligned}
\mathcal{E}(m_1)\times\mathcal{E}(m_2) &= (g^{r_1},m_1h^{r_1})(g^{r_2},m_2h^{r_2}) \\
&= (g^{r_1+r_2},m_1m_2h^{r_1+r_2}) \\
&= \mathcal{E}(m_1m_2)
\end{aligned}
$$

加法準同型暗号
- Paillier暗号

$$
\begin{aligned}
\mathcal{E}(m_1)\times\mathcal{E}(m_2) &= (g^{m_1}r_1^n)(g^{m_2}r_2^n) & \bmod n^2 \\
&= g^{m_1+m_2}(r_1r_2)^n & \bmod n^2 \\
&= \mathcal{E}(m_1+m_2)
\end{aligned}
$$

## レベル $n$ 準同型暗号
Lifted-ElGamal暗号
レベル $2$ 準同型暗号
楕円曲線 $E/\mathbb{F}_p$, と生成元 $P\in E$
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

$$
\mathcal{E}(m_1)\times\mathcal{E}(m_2) := (e(S_1, S_2), e(S_1, T_2), e(T_1, S_2), e(T_1, T_2))
$$

$$
\begin{aligned}
\mathcal{D}(c_1,c_2,c_3,c_4) &= \frac{c_1c_4^{s_1s_2}}{c_2^{s_2}c_3^{s_1}} = \frac{e(S_1,s_2)e(s_1T_1,s_2T_2)}{e(S_1,s_2T_2)e(s_1T_1,S_2)} \\
&=e(S_1-s_1T_1,S_2-s_2T_2) \\
&=e(mP_1,m'P_2)=e(P_1,P_2)^{mm'}
\end{aligned}
$$




## 完全準同型暗号
格子ベースの暗号で完全準同型暗号

### ブートストラップ
1. ノイズを削減したい暗号文 $c$ について、(ビット単位などに分割した上で) $c$ をさらに暗号化する
2. 上で得られた $c$ の暗号文 $\hat{c}$ について、「$c$ を復号して平文を得る」という操作に対応する準同型演算を $\hat{c}$ に施す。
3. 暗号文 $\hat{c}$ の内部で $c$ がその平文 $m$ へ変換され、結果として $m$ の新たな暗号文が得られる。


### Gentry
最初期の完全準同型暗号です。

### TFHE
現状で最も高速な部類に入る完全準同型暗号です。

$\mathbb{T} := \mathbb{R}/\mathbb{Z}$ は環ではないです。
群 $+:\mathbb{T}\times\mathbb{T}\to\mathbb{T}$
作用 $\times:\mathbb{T}\times\mathbb{Z}\to\mathbb{T}$
$\mathbb{Z}_N[X] := \mathbb{Z}[X]/(X^N + 1)$
$\mathbb{T}_N[X] := \mathbb{R}[X]/(X^N + 1)\bmod 1$

> **Gadget Decomposition function**
> $\|d - \boldsymbol{v}\cdot H\|$ を最小化する $\boldsymbol{v}$ を求めるアルゴリズム
approximate decomposition

LWE は次のようなものだった。

$\boldsymbol{a}$: $U_{\mathbb{T}^n}$
$e$: $\mathcal{D}_{\mathbb{T},\alpha}$
$\boldsymbol{s}$: $U_{\mathbb{B}^n}$

$$
\begin{aligned}
b & = \boldsymbol{a}\cdot\boldsymbol{s} + m + e \\
m & \in\mathbb{B}, \mu = \frac{1}{8}\in\mathbb{T} \\
b & = \boldsymbol{a}\cdot\boldsymbol{s} + \mu(2\cdot m - 1) + e \\
m & = \frac{sgn(b - \boldsymbol{a}\cdot\boldsymbol{s}) + 1}{2}
\end{aligned}
$$

Torus Ring-LWE
SampleExtractIndex

$$
\begin{aligned}
\lceil Bg\cdot(a_0, a_1, b)\rfloor\cdot\frac{\mu}{Bg} & = (a_0^r, a_1^r, b^r) \approx \mu\cdot(a_0, a_1, b) \\
b^r - \boldsymbol{a}^r\cdot\boldsymbol{s} & = \mu(b - \boldsymbol{a}\cdot\boldsymbol{s} + e_r)
\end{aligned}
$$

Decomposition

$$
\begin{aligned}
& \arg\min\sum_{j=0}^{N-1}\left(a_j - \sum_{i=1}^l\frac{\overline{a}_{ij}}{Bg^i}\right)^2 \\
\overline{a}_{ij}& \in\left[-\frac{Bg}{2}, \frac{Bg}{2}\right) \\
\lceil Bg\cdot(a, b)\rfloor \\
(a, b)\approx(\overline{a}_1, \ldots, \overline{a}_l, \overline{b}_1, \ldots, \overline{b}_l)
\begin{pmatrix}
\frac{1}{Bg} & 0 \\
\frac{1}{Bg^2} & 0 \\
\vdots & \\
\frac{1}{Bg^l} & 0 \\
0 & \frac{1}{Bg} \\
0 & \frac{1}{Bg^2} \\
\vdots & \\
0 & \frac{1}{Bg^l} \\
\end{pmatrix}
\end{aligned}
$$

Blind Rotate

これで準同型暗号に興味を持った方は Zama プロジェクトを調べてみると楽しいと思います。誰にも知られずに人工知能の推論や

## 参考文献

- [ペアリングベースの効率的なレベル2準同型暗号（SCIS2018） (slideshare.net)](https://www.slideshare.net/herumi/2scis2018)
