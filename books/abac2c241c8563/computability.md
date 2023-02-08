---
title: "計算可能性とゼロ知識証明"
---

今までの暗号がこんなに攻撃できそうなのになぜ安全だと言えるのか？それは計算可能性に基づいているからです。

- 停止性問題

チューリング還元

1. $G$ が計算可能だと仮定。
2. $F$ は $G$ に還元できるので $F$ も計算可能。
3. $F$ は計算不能だという事実と矛盾する。
4. よって $G$ は計算不能である。

> **Prop.**
> $G$ が計算可能なら、$F$ は計算可能
> $F$ が計算不能なら、$G$ は計算不能

乱数を用いた確率的な計算不能性を用いて担保されることが多いです。例えば鍵生成や Nonce Salt, ワンタイムパッドなど。乱数はどうやって生成しているのか次の節で紹介していきます。

## 乱数生成

乱数生成器は大きく分けて2つに分けられます。

- アルゴリズムで生成する決定論的乱数生成器 (DRBG; Deterministic Random Bit Generator)
- 熱のノイズやリングオシレーター、量子力学的性質などを用いる真性乱数生成器 (TRNG; True Random Number Generator)

また、これらを組み合わせた乱数生成器もしくは DRBG のみを疑似乱数生成器 (PRNG; Pseudo Random Number Generator) と呼びます。

私は TRNG の仕組みをあまり知らないので DRBG だけを紹介していこうと思います。

DRBG にはいくつか種類がありますがメルセンヌ・ツイスタと標準化された生成器のみを紹介しようと思います。攻撃方法は全て同じで内部状態をいかに復元するかが鍵となっています。

他に有名な DRBG として競プロでよく使われる XorShift と線形合同法がありますが、XorShift は練習問題に回して、線形合同法とその攻撃については多くの解説記事があるのでここでは割愛します。良記事としては次のものがあります。

https://www.youtube.com/watch?v=WaAErTq7hWA

### メルセンヌ・ツイスタ
統計的に十分に分散していて長い周期を持つ高速な疑似乱数生成器の一種です。周期の長さは $2^{19937}-1$ とメルセンヌ数であり名前の由来になっています。実は日本人が作っています。

中身では 32 ビットのビットベクタで計算されていて初期状態 $\mathbf{x}_i$ $(i = 0,\cdots,n)$ を入力して漸化式から $\mathbf{x}_k$ を生成し、それぞれの $\mathbf{x}$ について後処理をした $\mathbf{y}$ を出力とします。

$$
\begin{aligned}
  \mathbf{x}_{k+n} & = \mathbf{x}_{k+m}\hspace{-10px}&& \oplus((\mathbf{x}_k\mid\mathbf{x}_{k+1})\gg 1)\oplus(\mathrm{LSB}(\mathbf{x}_{k+1})\mathop{\mathrm{AND}}\mathbf{a}) \\
  \mathbf{y} & \leftarrow \mathbf{x} && \oplus\ \,(\mathbf{x}\gg 11) \\
  \mathbf{y} & \leftarrow \mathbf{y} && \oplus((\mathbf{y}\ll\ \ 7) \mathop{\mathrm{AND}} \mathbf{b}) \\
  \mathbf{y} & \leftarrow \mathbf{y} && \oplus((\mathbf{y}\ll 15) \mathop{\mathrm{AND}} \mathbf{c}) \\
  \mathbf{y} & \leftarrow \mathbf{y} &&\oplus\ \,(\mathbf{y}\gg 18) \\
\end{aligned}
$$

ただし、$\mathbf{x}_k\mid\mathbf{x}_{k+1}$ は $\mathbf{x}_k$ の最上位ビットと $\mathbf{x}_{k+1}$ の下位 31 ビットを結合する演算、$\mathrm{LSB}(\mathbf{x}_{k+1})$ は $\mathbf{x}_{k+1}$ の最下位ビットを 32 ビットに展開する演算です。
初期シード $\mathbf{x}_0$ を元に初期状態とパラメータは次のようにします。

$$
\begin{aligned}
  n & = 624 \quad m = 397 \\
  \mathbf{a} & = \mathrm{0x9908B0DF} \quad \mathbf{b} = \mathrm{0x9D2C5680} \quad \mathbf{c} = \mathrm{0xEFC60000} \\
  x_i & = (x_{i-1} \oplus (x_{i-1}\gg 30))\times 1812433253 + i \pmod{2^{32}}
\end{aligned}
$$

見ての通り、逆変換を行うことで連続した 624 回の 32 ビット出力から内部状態を復元できてしまいます。

https://6715.jp/posts/6/

更に初期状態については 2 つの値さえ分かっていれば内部状態を復元できてしまいます。
https://www.ambionics.io/blog/php-mt-rand-prediction

これでは乱数を予測されてしまうので暗号には使えません。暗号で使えるような DRBG はあるのでしょうか？それらを次節で紹介します。

### CSPRNG
暗号でも使えるような PRNG を暗号論的擬似乱数生成器 (CSPRNG; Cryptographically Secure Pseudo Random Number Generator) と呼びます。
標準化されている CSPRNG は次のようなものがあります。

- Hash_DRBG
- HMAC_DRBG
- CTR_DRBG
- Dual_EC_DRBG

それぞれの詳細や実装は以下の規格書を読んでもらうことにして、基本的なアイデアを載せます。

[NIST SP 800-90A (Recommendation for Random Number Generation Using Deterministic Random Bit Generators)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-90a.pdf)

#### Hash_DRBG

#### HMAC_DRBG
$$
\begin{aligned}

\end{aligned}
$$
$HMAC(K, V\|0x00\|input)$

#### CTR_DRBG
AES-CTR

#### Dual_EC_DRBG

NIST-p256, NIST-p384, NIST-p521 において点 $P, Q$ と初期シード $s_0$ を用いて乱数を生成する。

$$
\begin{aligned}
  s_{i+1} & = (s_iP)_x \\
  r_{i+1} & = (s_{i+1}Q)_x
\end{aligned}
$$

$r_i$ は 32 バイトであり、上位 2 バイトを削除した 30 バイトを連結させて出力する。

しかし、もし NSA がこの点について ECDLP が解けている場合、内部状態を復元できる為、バックドアとなります。
2006 年に NIST SP800-90A に組み込まれ、2013 年に利用すべきではないと勧告されています。

## ゼロ知識証明
ゼロ知識証明の性質
- 完全性（Completeness）
    - 証明者の主張が真であるならば、検証者は真であることが必ずわかること。
- 健全性（Soundness）
    - 証明者の主張が偽であれば、検証者はかなり高い確率でそれが偽であること見抜けること。
- ゼロ知識性（Zero Knowledge）
	- あらゆる場合において、検証者が証明者から何らかの知識（情報）を盗もうとしても、証明者の主張が真であること以上の知識は得られない


名前の通り対話型は有名な洞窟の例のように証明者と検証者がやりとりを繰り返し、証明者が本当に正しい情報を持っているかを確率的に検証する方です。

一方、非対話型のゼロ知識証明は証明者と検証者はやりとりをせずに証明することが可能です。対話を行う代わりに証明者と検証者の間に第三者を置き、CRSと呼ばれる事前に公開される情報を証明者と検証者に送ります。証明者はそのCRSを用いて正しい情報を持っているという証明を生成し、それを検証者に一回だけ送ります。受け取った検証者はそれを検証するだけで非対話なゼロ知識証明が実現できます。事前にCRSを生成することを一般的に信頼されたセットアップといい、第三者は証明者、検証者にとって信頼される存在となります。

zk-SNARKs
- Succinct（簡潔）
    - 証明のサイズがステートメントのサイズと比べて非常に小さい
- Non-interactive（非対話型）
    - 証明者と検証者の間で何度も対話をする必要がない
- ARgument
    - 証明者の計算能力には限りがある
- Knowledge
    - 証明者は、知識なしでは証明を生成することは不可能である。

[ZenGo-X/zk-paillier: A collection of Paillier cryptosystem zero knowledge proofs (github.com)](https://github.com/ZenGo-X/zk-paillier)

[zk-SNARKsの理論 (zenn.dev)](https://zenn.dev/kyosuke/articles/a1854b9be26c01df13eb)

### 参考文献

- [メルセンヌ・ツイスタをわかった気になる](https://6715.jp/posts/5/)
- [Mersenne Twisterの出力を推測してみる](https://inaz2.hatenablog.com/entry/2016/03/07/194147)