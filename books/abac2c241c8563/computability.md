---
title: "乱数と計算可能性"
---

今までの暗号がこんなに攻撃できそうなのになぜ安全だと言えるのか？それは計算可能性に基づいているからです。

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

乱数を用いた確率的な計算不能性を用いて担保されることが多いです。例えば鍵生成や Nonce Salt, ワンタイムパッドなど。乱数はどうやって生成しているのか次の節で紹介していきます。

ランダムオラクルモデル

代表的な暗号の安全性の根拠
- RSA暗号 素因数分解
- ElGamal暗号 DLP
- 楕円曲線暗号 ECDLP

## 乱数生成
誰にも予測できないような乱数を作るにはどうすればいいでしょうか？

例えばこんな例を考えてみましょう。
乱数を要求されたら円周率の $n$ 桁目からの数字を順番に返すような乱数生成器の OSS とその API があるとします。これは均一に分布しているし分散していて統計的には乱数と言えます。では果たしてこれは暗号として使うには安全でしょうか？

もちろん安全ではありません。$n$ がソースコードに書かれてあるので誰にでも予測できてしまいます。

では $n$ を誰にも分からないくらいとても大きくして秘密にすればどうでしょうか？
もちろん安全になります。でもよく考えてみてください。開発者は $n$ を知っているので開発者だけが簡単に予測することができてしまいます。これをバックドアといい、出来る限り避けるべきです。つまりオープンソースであることは必須条件です。

つまり乱数に必要な条件としてはこうです。

1. 統計的に選ばれる確率が均一であり十分に分散している
2. 全てがオープンソースである
3. 誰にも予測できない

このような乱数を生成することはできるのでしょうか？それらを

### 真性乱数生成器 (TRNG)
最も原始的で簡単な方法がノイズを用いる方法です。ノイズといっても音のノイズではなく、熱のノイズや電気のノイズ、量子などを使って、エントロピーが低くなるまでノイズの情報を掻き集めます。これを真性乱数生成器 (TRNG; True Random Number Generator) と呼びます。真性乱数でも意図的に負荷を掛けてノイズの予測を立てられることもあるので、一概にエントロピーが低いことを保証できないことを確認しておきます。

メリット
- 非決定的に生成するので次の値を予測しにくい

デメリット
- 生成に時間がかかる
- サイドチャネル攻撃などによってバイアスが掛かる可能性がある

### 決定論的乱数生成器 (DRBG)
TRNG だと乱数を生成するのに熱が揺らいでいるのを取らないといけないので時間がかかります。高速化するにはアルゴリズムで疑似的に乱数を生成することが必要です。これを決定論的乱数生成器 (DRBG; Deterministic Random Bit Generator) と呼びます。

メリット
- 高速に生成できる

デメリット
- 決定論的に生成するので予測できる可能性がある

### 疑似乱数生成器 (PRNG)
それなら TRNG と DRBG を組み合わせれば長所と短所を互いに補ってよいじゃないかというのはごもっともで DRBG と TRNG を組み合わせた乱数生成器もしくは DRBG のみを疑似乱数生成器 (PRNG; Pseudo Random Number Generator) と呼びます。(実際は DRBG と PRNG は同じ意味らしいが便宜上この定義とする)

:::message
演習
`/dev/random` や `/dev/urandom` を用いて乱数を生成してみよう
:::

私は TRNG の仕組みをあまり知らないので DRBG だけを紹介していこうと思います。

DRBG にはいくつか種類がありますがメルセンヌ・ツイスタと標準化された生成器のみを紹介しようと思います。他に有名な DRBG として競プロでよく使われる XorShift と線形合同法がありますが、XorShift は練習問題に回して、線形合同法とその攻撃については多くの解説記事があるのでここでは割愛します。良記事としては次のものがあります。

https://www.youtube.com/watch?v=WaAErTq7hWA

DRBG に対する全ての攻撃は内部状態をいかに復元するかが鍵となっています。

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

見ての通り、逆変換を行うことで任意の連続した 624 回の 32 ビット出力から内部状態を復元できてしまいます。
https://6715.jp/posts/6/

更に初期状態については 2 つの値さえ分かっていれば内部状態を復元できてしまいます。
https://www.ambionics.io/blog/php-mt-rand-prediction

これでは乱数を予測されてしまうので暗号には使えません。暗号で使えるような DRBG はあるのでしょうか？それらを次節で紹介します。

:::message
演習
Python の `random` モジュールを予測してみよう
ヒント
:::

### CSPRNG
暗号でも使えるような PRNG を暗号論的擬似乱数生成器 (CSPRNG; Cryptographically Secure Pseudo Random Number Generator) と呼びます。
標準化されている CSPRNG は次のようなものがあります。

- Hash_DRBG
- HMAC_DRBG
- CTR_DRBG
- Dual_EC_DRBG

これらに共通することとして乱数、Nonce、ユーザーによって指定される文字列を入力し、内部状態であるシード値を生成します。このシード値を用いて、指定されたビット数に達するまで乱数を生成し続けて連結させたものを出力します。何回か生成したらシード値の再生成 (reseed) を行い、エントロピーを上げます。攻撃についても同様に内部状態を復元することで乱数予測することができます。

それぞれの詳細や実装は以下の規格書を読んでもらうことにして、基本的なアイデアを掻い摘んで紹介します。
[NIST SP 800-90A (Recommendation for Random Number Generation Using Deterministic Random Bit Generators)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-90a.pdf)

#### Hash_DRBG
bcrypt などのハッシュ関数 $H$ とシード値 $V_0$ を用いて乱数を生成します。

$$
\begin{aligned}
  V_{i+1} & = H(V_i + 1)
\end{aligned}
$$

ハッシュ関数を繰り返し適用せず、インクリメントする理由はハッシュ関数を何回か適用すると元に戻る性質を持つとき脆弱になるからです(通常は Preimage Resistance よりそんなこと起こりえませんが)。

#### HMAC_DRBG
HMAC とシード値 $V_0$ を用いて乱数を生成します。

$$
\begin{aligned}
  V_{i+1} & = \mathrm{HMAC}(K, V_i)
\end{aligned}
$$

HMAC は SHA-256 を使うことが多いです。

#### CTR_DRBG
AES-CTR の暗号化関数 $E_K$ とシード値 $V_0$ を用いて乱数を生成します。

$$
\begin{aligned}
  V_{i+1} & = E_K(V_i + 1)
\end{aligned}
$$

#### Dual_EC_DRBG

NIST-p256, NIST-p384, NIST-p521 において点 $P, Q$ と初期シード $s_0$ を用いて乱数を生成します。

$$
\begin{aligned}
  s_{i+1} & = (s_iP)_x \\
  r_{i+1} & = (s_{i+1}Q)_x
\end{aligned}
$$

$r_i$ は剰余未満の数であり、その上位 2 バイト程を削除した数を連結させて出力します。P-256 なら 32 バイトの数なので下位 30 バイトを出力します。

しかし、もし NSA がこの点について ECDLP が解けている場合、内部状態を復元できる為、バックドアとなります。この為、2006 年に NIST SP800-90A に組み込まれましたが、2013 年に利用すべきではないと勧告されています。

ではこれらの乱数を用いてどのように安全性を担保しているのか

#### コラム
RTA

### 参考文献

- [メルセンヌ・ツイスタをわかった気になる](https://6715.jp/posts/5/)
- [Mersenne Twisterの出力を推測してみる](https://inaz2.hatenablog.com/entry/2016/03/07/194147)