---
title: "計算可能性"
---

## 計算可能性
今までの暗号がこんなに攻撃できそうなのになぜ安全だと言えるのか？それは計算可能性が証明できるからです。

- 停止性問題

チューリング還元

1. $G$ が計算可能だと仮定。
2. $F$ は $G$ に還元できるので $F$ も計算可能。
3. $F$ は計算不能だという事実と矛盾する。
4. よって $G$ は計算不能である。

![](/images/computability.png)

> **Prop.**
> $F$ から $G$ へ還元できるとき
> $G$ が計算可能なら、$F$ は計算可能
> $F$ が計算不能なら、$G$ は計算不能


ランダムオラクルモデル

代表的な暗号の安全性の根拠
- RSA暗号 素因数分解
- ElGamal暗号 DLP
- 楕円曲線暗号 ECDLP

攻撃耐性

- 選択平文攻撃 (Chosen-plaintext attack; CPA)
- 適応的選択平文攻撃 (Adaptive chosen-plaintext attack; CPA2)
- 選択暗号文攻撃 (Chosen-ciphertext attack; CCA1)
- 適応的選択暗号文攻撃 (Adaptive Chosen-ciphertext attack; CCA2)
- Side-channel attack

↓証明したい

- 一方向性 (Onewayness; OW)
	- 暗号文から平文を求めるのが困難
- 強秘匿性 (Semantic Security; SS)
	- 暗号文から平文のどんな部分情報も漏れない
- 識別不可能性 (Indistinguishability; IND)
	- 暗号文が平文AとBのどちらのものかを区別できない
- 頑強性 (Non-Malleability; NM)
	- 暗号文が与えられた時、ある関係性を持った別の暗号文の生成が不可
	- stream cipher
	- RSA暗号 $Enc(m)\cdot t^e \bmod n = Enc(mt)$ padding(OAEP, PKCS 1)

乱数を用いた確率的な計算不能性を用いて担保されることが多いです。例えば鍵生成や Nonce Salt, ワンタイムパッドなど。乱数はどうやって生成しているのか次の節で紹介していきます。

