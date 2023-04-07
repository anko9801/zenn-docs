---
title: "乱数と計算可能性"
---

## 計算可能性
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

## 暗号の性質
耐量子性
準同型性

- 選択平文攻撃 (Chosen-plaintext attack; CPA)
- 適応的選択平文攻撃 (Adaptive chosen-plaintext attack; CPA2)
- 選択暗号文攻撃 (Chosen-ciphertext attack; CCA1)
- 適応的選択暗号文攻撃 (Adaptive Chosen-ciphertext attack; CCA2)
- Side-channel attack

↓ めちゃくちゃわかりにくい、といってわかりやすさが上がる方法とは？定理証明だと思う.

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

## 乱数生成
誰にも予測できないような乱数を作るにはどうすればいいでしょうか？

例えばこんな例を考えてみましょう。
乱数を要求されたら円周率の $n$ 桁目からの数字を順番に返すような乱数生成器の OSS とその API があるとします。これは均一に分布しているし分散していて統計的には乱数と言えます。では果たしてこれは暗号として使うには安全でしょうか？

もちろん安全ではありません。$n$ がソースコードに書かれてあるので誰にでも予測できてしまいます。

では $n$ を誰にも分からないくらいとても大きくして秘密にすればどうでしょうか？
もちろん安全になります。でもよく考えてみてください。開発者は $n$ を知っているので開発者だけが簡単に予測することができてしまいます。これをバックドアといい、出来る限り避けるべきです。つまりオープンソースであることは必須条件です。

これより乱数に必要な条件としてはこうです。

1. 統計的に選ばれる確率が均一であり十分に分散している
2. 全てがオープンソースである
3. 誰にも予測できない

このような乱数を生成することはできるのでしょうか？この章ではそのような乱数生成器を紹介していきます。

### 真性乱数生成器 (TRNG)
最も原始的で簡単な方法がノイズを用いる方法です。ノイズといっても音のノイズではなく、熱のノイズや電気のノイズ、トンネル効果などを用いた量子の確率的挙動などです。それらノイズの情報をエントロピーが低くなるまで掻き集めて、乱数として返します。これを真性乱数生成器 (TRNG; True Random Number Generator) と呼びます。真性乱数でも意図的に負荷を掛けてノイズの予測を立てられることもあるので、一概にエントロピーが低いことを保証できないことを確認しておきます。

メリット
- 非決定的に生成するので次の値を予測しにくい

デメリット
- 生成に時間がかかる
- サイドチャネル攻撃などによってバイアスが掛かる可能性がある

### 決定論的乱数生成器 (DRBG)
TRNG だと乱数を生成するのに熱が揺らいでいるのを取らないといけないので時間がかかります。高速化するにはアルゴリズムで疑似的に乱数を生成することが必要です。これを決定論的乱数生成器 (DRBG; Deterministic Random Bit Generator) と呼びます。
ex.) XorShift、線形合同法、メルセンヌ・ツイスタ、LFSR

メリット
- 高速に生成できる

デメリット
- 決定論的に生成するので予測できる可能性がある

### 疑似乱数生成器 (PRNG)
それなら TRNG と DRBG を組み合わせれば長所と短所を互いに補ってよいじゃないかというのはごもっともで DRBG と TRNG を組み合わせた乱数生成器もしくは DRBG のみを疑似乱数生成器 (PRNG; Pseudo Random Number Generator) と呼びます。(実際は DRBG と PRNG は同じ意味らしいが便宜上この定義とする)

:::message
**練習問題**
`/dev/random` や `/dev/urandom` を用いて乱数を生成してみよう
:::

私は TRNG の仕組みをあまり知らないので DRBG だけを紹介していこうと思います。

DRBG は競プロのテストは XorShift を用います。今回はメルセンヌ・ツイスタを紹介します。線形合同法の攻撃については多くの解説記事があるのでここでは割愛します。良記事としては次のものがあります。

https://www.youtube.com/watch?v=WaAErTq7hWA

DRBG に対する全ての攻撃は内部状態をいかに復元するかが鍵となっています。

### LFSR (Linear Feedback Shift Register)

$\boldsymbol{a}_k^n = (a_k, a_{k+1}, \ldots, a_{k+n-1})\in\mathbb{F}_2^n$

$$
\begin{aligned}
a_{k+n} & = \boldsymbol{s}\cdot\boldsymbol{a}_k^{n} = \sum_{i=0}^{n-1} s_ia_{k+i} \\
x_k & = \boldsymbol{c}\cdot\boldsymbol{a}_k^{n} = \sum_{i=0}^{n-1} c_ia_{k+i}
\end{aligned}
$$

```python
class LFSR():
  def __init__(self, n):
    self.state = [getRandBit(1) for _ in range(n)]
    self.taps = [getRandBit(1) for _ in range(n)]
  def next(self):
    output = reduce(xor, [bit&tap for bit,tap in zip(self.state, self.taps)])
    self.state = self.state[1:] + [output]
    return output

class lfsr():
  def __init__(self, init, mask, length):
    self.init = init
    self.mask = mask
    self.lengthmask = 2**(length+1)-1

  def next(self):
    nextdata = (self.init << 1) & self.lengthmask
    i = self.init & self.mask & self.lengthmask
    output = 0
    while i != 0:
      output ^= (i & 1)
      i = i >> 1
    nextdata ^= output
    self.init = nextdata
    return output

def generate_bit_sequence(initial_state):
  state = initial_state
  while True:
    last_bit = state & 1
    yield last_bit
    middle_bit = state >> 3 & 1
    state = (state >> 1) | ((last_bit ^ middle_bit) << 4)
```

:::message
zer0lfsr, zer0lfsr+, zer0lfsr++ を解こう
:::

#### NLSR
非線形化を施すと行列形式で書くことができないので逆変換が難しくなる。

攻撃手法
1. 高次項は無視や線形近似をする (Walsh-Hadamard 変換)
2. 低次項は状態として定義する
3. annihilator
4. Correlation Attack
5. Z3

### メルセンヌ・ツイスタ (MT19937)
統計的に十分に分散していて長い周期を持つ高速な疑似乱数生成器の一種です。周期の長さは $2^{19937}-1$ とメルセンヌ数であり名前の由来になっています。実は日本人が作っています。

中身では 32 ビットのビットベクタで計算されていて初期状態 $\boldsymbol{x}_i$ $(i = 0,\cdots,n)$ を入力して漸化式から $\boldsymbol{x}_k$ を生成し、それぞれの $\boldsymbol{x}_k$ について後処理をした $\boldsymbol{y}$ を出力とします。

$$
\begin{aligned}
  \boldsymbol{x}_{k+n} & = \boldsymbol{x}_{k+m}\hspace{-10px}&& \oplus((\boldsymbol{x}_k\mid\boldsymbol{x}_{k+1})\gg 1)\oplus(\mathrm{LSB}(\boldsymbol{x}_{k+1})\mathop{\mathrm{AND}}\boldsymbol{a}) \\
  \boldsymbol{y} & \leftarrow \boldsymbol{x}_k \\
  \boldsymbol{y} & \leftarrow \boldsymbol{y} && \oplus\ \,(\boldsymbol{y}\gg 11) \\
  \boldsymbol{y} & \leftarrow \boldsymbol{y} && \oplus((\boldsymbol{y}\ll\ \ 7) \mathop{\mathrm{AND}} \boldsymbol{b}) \\
  \boldsymbol{y} & \leftarrow \boldsymbol{y} && \oplus((\boldsymbol{y}\ll 15) \mathop{\mathrm{AND}} \boldsymbol{c}) \\
  \boldsymbol{y} & \leftarrow \boldsymbol{y} &&\oplus\ \,(\boldsymbol{y}\gg 18) \\
\end{aligned}
$$

ただし、$\boldsymbol{x}_k\mid\boldsymbol{x}_{k+1}$ は $\boldsymbol{x}_k$ の最上位ビットと $\boldsymbol{x}_{k+1}$ の下位 31 ビットを結合する演算、$\mathrm{LSB}(\boldsymbol{x}_{k+1})$ は $\boldsymbol{x}_{k+1}$ の最下位ビットを 32 ビットに展開する演算です。パラメータと初期シード $x_0$ を元に初期状態を生成する漸化式は次のようにします。

$$
\begin{aligned}
  n & = 624 \quad m = 397 \\
  \boldsymbol{a} & = \mathrm{0x9908B0DF} \quad \boldsymbol{b} = \mathrm{0x9D2C5680} \quad \boldsymbol{c} = \mathrm{0xEFC60000} \\
  x_i & = (x_{i-1} \oplus (x_{i-1}\gg 30))\times 1812433253 + i \pmod{2^{32}}
\end{aligned}
$$

見ての通り、逆変換を行うことで任意の連続した 624 回の 32 ビット出力から内部状態を復元できてしまいます。
https://6715.jp/posts/6/

更に初期状態については 2 つの値さえ分かっていれば内部状態を復元できてしまいます。
https://www.ambionics.io/blog/php-mt-rand-prediction

これでは乱数を予測されてしまうので暗号には使えません。暗号で使えるような DRBG はあるのでしょうか？それらを次節で紹介します。

:::message
**練習問題**
CPCTF22/the luck 2 より
Python の `random` モジュールを予測してみよう
`secrets` モジュールとの違いを考えよう
https://github.com/python/cpython/blob/main/Lib/random.py
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

#### コラム
RTA

## 参考文献

- [メルセンヌ・ツイスタをわかった気になる](https://6715.jp/posts/5/)
- [Mersenne Twisterの出力を推測してみる](https://inaz2.hatenablog.com/entry/2016/03/07/194147)