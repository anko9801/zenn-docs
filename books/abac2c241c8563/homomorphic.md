---
title: "準同型暗号"
---

## 準同型暗号
準同型暗号には次のような種類がある。

- 乗法準同型暗号
- 加法準同型暗号
- レベル n 準同型暗号
- 完全準同型暗号

## 乗法準同型暗号
最もよくある準同型暗号
### Unpadded RSA

$$
\begin{aligned}
\mathcal{E}(m_1)\times\mathcal{E}(m_2) &= m_1^em_2^e & \bmod n \\
&= (m_1m_2)^e & \bmod n \\
&= \mathcal{E}(m_1m_2)
\end{aligned}
$$

### ElGamal暗号

$$
\begin{aligned}
\mathcal{E}(m_1)\times\mathcal{E}(m_2) &= (g^{r_1},m_1h^{r_1})(g^{r_2},m_2h^{r_2}) \\
&= (g^{r_1+r_2},m_1m_2h^{r_1+r_2}) \\
&= \mathcal{E}(m_1m_2)
\end{aligned}
$$

## 加法準同型暗号
あまり見かけない準同型暗号です。
### Paillier暗号
加法準同型

$$
\begin{aligned}
\mathcal{E}(m_1)\times\mathcal{E}(m_2) &= (g^{m_1}r_1^n)(g^{m_2}r_2^n) & \bmod n^2 \\
&= g^{m_1+m_2}(r_1r_2)^n & \bmod n^2 \\
&= \mathcal{E}(m_1+m_2)
\end{aligned}
$$

## レベルn準同型暗号
レベルn準同型 = 加法準同型 + n回の乗算準同型
- 乗算回数に制約がある完全準同型
- レベルnはn次方程式を計算できる
- レベル2準同型なら乗算が1度だけ可能

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