---
title: "共通鍵暗号"
---

CTF では AES 暗号の攻撃のみに焦点を当てているものが多いのでそれだけでもいいですが、最新の暗号研究にも追い付きたい為に基礎的な知識は網羅的に解説します。

## Feistel 構造
暗号化

$$
\begin{align}
  R_{r+1} & = L_r \oplus F(R_r, k_r) \\
  L_{r+1} & = R_r
\end{align}
$$

復号化

$$
\begin{align}
  L_{r} & = R_{r+1} \oplus F(L_{r+1}, k_r) \\
  R_{r} & = L_{r+1}
\end{align}
$$

![](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d2/Feistel.png/220px-Feistel.png)


## SPN 構造
非線形置換
ブロック内での転置

S-box
撹拌性の検証をコードで実装する

:::message
**練習問題**
[CryptoHack の SYMMETRIC CIPHERS](https://cryptohack.org/challenges/aes/) で HOW AES WORKS の章を解いてください。
:::

## AES
- 共通鍵暗号の標準的な暗号
- AES-NI
  - 回路に組み込むことで高速化

## PKCS #7 パディング
~~最適化の為にパディングの最も左のバイトが判定の基準となる。逆にそれ以外は参照しない。~~
全部見るらしいので次のようなイディオムを使うとよいかも

```
from pwn import xor
ciphertext = xor(ciphertext[pad:], long_to_bytes(i+1), long_to_bytes(i+2))
```

Padding Oracle Attackでは最後の1文字総当り時に全く同じとき例外として落とす必要がある。

```
????????01
??????0202
????030303
...
0f...0f0f0f
??...??????1010...1010
```

AES オンラインシミュレータほしいかも

パディングした平文を $P$ として次のように複数のブロックに分割し、暗号化の際、それぞれのブロックで暗号化します。暗黙のうちに16バイトごと
$P = P_1\|\cdots\|P_n$

### AES-ECB (Electronic CodeBlock)
暗号化

$$
C_i = E_K(P_i)
$$

復号化

$$
P_i = D_K(C_i)
$$


![](https://ja.wikipedia.org/wiki/%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB:ECB_encryption.svg)

### AES-CBC (Cipher Block Chaining)
暗号化

$$
C_i = \begin{cases}
  IV & (i = 0) \\
  E_K(P_i\oplus C_{i-1}) & (i > 0)
\end{cases}
$$

復号化

$$
P_i = D_K(C_i)\oplus C_{i-1}
$$
### AES-OFB
### AES-CTR
### PCBC (Propagating Cipher Block Chaining)
### CFB (Cipher Feedback)

### AES-GCM (Galois/Counter Mode)
入力
- 平文 $P$
- 認証データ (AAD; Additional Authenticated Data) $A$
- IV (Initialization Vector)
出力
- 暗号文 $C$
- 認証タグ $T$

バイトを揃える為にゼロパディングが必要となる。あるバイト長 n でゼロパディングするとはビット長 $8n$ の倍数となるように論理左シフトする操作である。
$J_0$ を次のように定義する。

$$
J_0 = \begin{cases}
IV\|0^{31}1 & (\text{IV is 96 bits})\\
GHASH_H(IV) & (\text{IV isn't 96 bits, IV is zero padded as 128-bit block size})
\end{cases}
$$

AES の暗号化と相性をよくする為に入力 $P$ を 16 バイトごとに切り分ける。それぞれのブロックを $P_i$ とすると暗号文 $C_i$ は次のような計算で求める。これを GCTR と呼ぶ。

$$
C_i = E_k(J_0 + i) \oplus P_i \qquad (i = 1,\ldots,n)
$$

これで暗号文 $C = C_1\|\cdots\|C_n$ を作れた。
次に認証タグ $T$ を計算する。認証タグは正しく暗号化されているか検証するため、また認証情報を入れて正しい情報が送られてきているかを検証するタグである。
認証タグはガロア体 $GF(2^{128})$ 上で計算する。

$$
GF(2^{128}) = \mathbb{F}_2[x]/(x^{128} + x^7 + x^2 + x + 1)
$$

$A, C$ は16バイトでゼロパディングしたもの, $\mathrm{len}$ は文字列長を8バイトで出力する関数である。$A\|C\|\mathrm{len}(A)\|\mathrm{len}(C)$ から 16 バイトずつ切り出したものを前から順番に $X_i \quad (i = 1,\ldots,n)$ とすると $Y_0 = 0, H = E_k(0^{128})$ として

$$
Y_i = (X_i + Y_{i-1})\cdot H
$$

と計算する。これを GHASH と呼ぶ。最後にもう一度 GCTR を行う。

$$
Y_i = E_k(J_0 + (i - 1)) \oplus X_{i} \qquad (i = 1,\ldots,n)
$$

上から平文の長さだけ取ってくると認証タグとなる。

![[Pasted image 20221225174723.png]]

### AES-OFB

$$
C_i = E_k^i(\mathrm{iv})\oplus P_i
$$

### 解き方
Padding Oracle Attack を使って暗号/復号化関数 $E_k$ を作る。
すると鍵を考えなくてもいい感じになり、上の式を辿るだけで解けるようになる。


## 参考文献
- [暗号利用モード](https://ja.wikipedia.org/wiki/%E6%9A%97%E5%8F%B7%E5%88%A9%E7%94%A8%E3%83%A2%E3%83%BC%E3%83%89)
- [Recommendation for Block Cipher Modes of Operation: Galois/Counter Mode (GCM) and GMAC](https://nvlpubs.nist.gov/nistpubs/legacy/sp/nistspecialpublication800-38d.pdf)

Grover's algorithm: $2^{K}\to 2^{K/2}$
鍵長を倍の長さにすることで同じセキュリティを担保できる。

Differencial cryptanalysis
これはハッシュ関数の実装に踏み込む手法である. スケッチとしては何かしらのパラメータが同じなどの特殊な場合のとき, 簡約化ができ, 単純な算術演算による条件式をいくつか生成できる. これを SMT で解くらしい.
