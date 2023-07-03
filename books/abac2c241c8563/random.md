---
title: "ハッシュと乱数生成"
---

誰にも予測できないような乱数を作るにはどうすればいいでしょうか？

例えばこんな例を考えてみましょう。
乱数を要求されたら円周率の $n$ 桁目からの数字を順番に返すような乱数生成器の OSS とその API があるとします。これは均一に分布しているし分散していて統計的には乱数と言えます。では果たしてこれは暗号として使うには安全でしょうか？

もちろん安全ではありません。$n$ がソースコードに書かれてあるので誰にでも予測できてしまいます。

では $n$ を誰にも分からないくらいとても大きくして秘密にすればどうでしょうか？
もちろん安全になります。でもよく考えてみてください。開発者は $n$ を知っているので開発者だけが簡単に予測することができてしまいます。これをバックドアといい、出来る限り避けるべきです。実際 2013 年にアメリカの国家安全保障局 (NSA) がバックドアを用いて盗聴をしていたという告発があったようです。つまりオープンソースであることは必須条件です。

これより乱数に必要な条件としてはこうです。

1. 統計的に選ばれる確率が均一であり十分に分散している
2. 全てがオープンソースである
3. 誰にも予測できない

このような乱数を生成することはできるのでしょうか？この章ではそのような乱数生成器を紹介していきます。

## 乱数生成
### 真性乱数生成器 (TRNG)
最も原始的で簡単な方法がノイズを用いる方法です。ノイズといっても音のノイズではなく、熱のノイズや電気のノイズ、トンネル効果などを用いた量子的な確率的振る舞いなどです。それらノイズの情報をエントロピーが低くなるまで掻き集めて、乱数として返します。これを真性乱数生成器 (TRNG; True Random Number Generator) と呼びます。

メリット
- 非決定的に生成するので次の値を予測しにくい

デメリット
- 生成に時間がかかる
- サイドチャネル攻撃などによってバイアスが掛かる可能性がある

ただ真性乱数でも意図的に外から負荷を掛けてノイズの予測を立てられることもあるので、一概にエントロピーが低いことを保証できません。それを用いた攻撃をサイドチャネル攻撃と言います。

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
- Easy: `/dev/random` や `/dev/urandom` を用いて乱数を生成してみよう
- Medium: `/dev/random` と `/dev/urandom` の違いとは？
:::

私は TRNG の仕組みをあまり知らないので DRBG だけを紹介していこうと思います。

簡易的で代表的な DRBG は次のようなものがあります。

- 線形合同法
- XorShift
- LFSR
- メルセンヌ・ツイスタ

これらは高速に生成できるので安全性が求められない疑似乱数としては優秀です。XorShift やメルセンヌ・ツイスタは競プロのテスト生成でよく使われますね。

このような DRBG に対する攻撃の本質はすべて内部状態をいかに復元するかです。
CTF では基本逆操作を行うことで攻撃が成功します。また高難易度典型としてCrypto ツールで紹介する SMT を使ったり、論理演算の線形近似 (Walsh-Hadamard 変換など) を行うなどの手法もあります。

> **線形合同法**

> **XorShift**

> **LFSR (Linear Feedback Shift Register)**
> $\bm{a}_k^n = (a_k, a_{k+1}, \ldots, a_{k+n-1})\in\mathbb{F}_2^n$
>
> $$
\begin{aligned}
a_{k+n} & = \bm{s}\cdot\bm{a}_k^{n} = \sum_{i=0}^{n-1} s_ia_{k+i} \\
x_k & = \bm{c}\cdot\bm{a}_k^{n} = \sum_{i=0}^{n-1} c_ia_{k+i}
\end{aligned}
$$


```python
class LFSR():
  def __init__(self, n):
    self.state = [getRandBit(1) for _ in range(n)]
    self.taps = [getRandBit(1) for _ in range(n)]
  def next(self):
    output = reduce(xor, [bit & tap for bit, tap in zip(self.state, self.taps)])
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

### メルセンヌ・ツイスタ (MT19937)
統計的に十分に分散していて長い周期を持つ高速な疑似乱数生成器の一種です。周期の長さは $2^{19937}-1$ とメルセンヌ数であり名前の由来になっています。実は日本人が作っています。

中身では 32 ビットのビットベクタで計算されていて初期状態 $\bm{x}_i$ $(i = 0,\cdots,n)$ を入力して漸化式から $\bm{x}_k$ を生成し、それぞれの $\bm{x}_k$ について後処理をした $\bm{y}$ を出力とします。

$$
\begin{aligned}
  \bm{x}_{k+n} & = \bm{x}_{k+m}\oplus((\bm{x}_k\mid\bm{x}_{k+1})\gg 1)\oplus(\mathrm{LSB}(\bm{x}_{k+1})\mathop{\mathrm{AND}}\bm{a}) \\
  \bm{y} & \leftarrow \bm{x}_k \\
  \bm{y} & \leftarrow \bm{y} \oplus\ \,(\bm{y}\gg 11) \\
  \bm{y} & \leftarrow \bm{y} \oplus((\bm{y}\ll\ \ 7) \mathop{\mathrm{AND}} \bm{b}) \\
  \bm{y} & \leftarrow \bm{y} \oplus((\bm{y}\ll 15) \mathop{\mathrm{AND}} \bm{c}) \\
  \bm{y} & \leftarrow \bm{y} \oplus\ \,(\bm{y}\gg 18) \\
\end{aligned}
$$

ただし、$\bm{x}_k\mid\bm{x}_{k+1}$ は $\bm{x}_k$ の最上位ビットと $\bm{x}_{k+1}$ の下位 31 ビットを結合する演算、$\mathrm{LSB}(\bm{x}_{k+1})$ は $\bm{x}_{k+1}$ の最下位ビットを 32 ビットに展開する演算です。パラメータと初期シード $x_0$ を元に初期状態を生成する漸化式は次のようにします。

$$
\begin{aligned}
  n & = 624 \quad m = 397 \\
  \bm{a} & = \mathrm{0x9908B0DF} \quad \bm{b} = \mathrm{0x9D2C5680} \quad \bm{c} = \mathrm{0xEFC60000} \\
  x_i & = (x_{i-1} \oplus (x_{i-1}\gg 30))\times 1812433253 + i \pmod{2^{32}}
\end{aligned}
$$

見ての通り、逆変換を行うことで任意の連続した 624 回の 32 ビット出力から内部状態を復元できてしまいます。
https://6715.jp/posts/6/

更に初期状態については 2 つの値さえ分かっていれば内部状態を復元できてしまいます。
https://www.ambionics.io/blog/php-mt-rand-prediction

連続する 624 個の 32 ビット出力なら一意に解けますが次のようなときはどうでしょう。

- 連続ではなかったら？
- 情報が足りず、一意でなくともいいなら？
- より一般の乱数生成では？

これらを論理的に計算するのはとても骨が折れます。

こういうときには SMT で解けます！
全探索より速いアルゴリズムがないときは SMT が強い

これでは乱数を予測されてしまうので暗号には使えません。暗号で使えるような DRBG はあるのでしょうか？それらを次節で紹介します。

:::message
**練習問題**
Python の `random.random()` を読んでみよう。
https://github.com/python/cpython/blob/main/Lib/random.py
:::

:::message
**演習問題**
[CTF archive](https://cryptohack.org/challenges/ctf-archive/) から出題
- Z3 を
- Twist and Shout (Zh3r0 CTF V2)
- import numpy as MT (Zh3r0 CTF V2)
- Real Mersenne (Zh3r0 CTF V2)
:::

## CSPRNG
暗号でも使えるような PRNG を暗号論的擬似乱数生成器 (CSPRNG; Cryptographically Secure Pseudo Random Number Generator) と呼びます。現在、標準化されている CSPRNG は [NIST SP 800-90A (Recommendation for Random Number Generation Using Deterministic Random Bit Generators)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-90a.pdf) に書かれてあります。

- Hash_DRBG
bcrypt などのハッシュ関数 $H$ とシード値 $V_0$ を用いて $V_{i+1} = H(V_i + 1)$ と生成する
- HMAC_DRBG
HMAC とシード値 $V_0$ を用いて $V_{i+1} = \mathrm{HMAC}(K, V_i)$ と生成する
- CTR_DRBG
AES-CTR の暗号化関数 $E_K$ とシード値 $V_0$ を用いて $V_{i+1} = E_K(V_i + 1)$ と生成する
- Dual_EC_DRBG (deprecated)
楕円曲線の加算を用いて生成する (後述)

これらに共通することとして乱数、Nonce、ユーザーによって指定される文字列を入力し、内部状態であるシード値を生成します。このシード値を用いて、指定されたビット数に達するまで乱数を生成し続けて連結させたものを出力します。何回か生成したらシード値の再生成 (reseed) を行い、エントロピーを上げます。

攻撃する方法としては今までと同様に内部状態を復元することで乱数予測することができます。

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

### コラム
乱数調整

## 安全性
このように疑似乱数や公開鍵暗号などは無限の計算能力を持つ攻撃者の前では SMT を用いて内部情報を取り出したり、公開鍵から秘密鍵を求めたりと攻撃できてしまうので、現代暗号の標準モデルでは限られた計算能力を持つ攻撃者 (確率的多項式時間チューリングマシン) を仮定します。

そして限られた計算能力しか持たない攻撃者にとって疑似乱数と本物の乱数を区別することができないなら、その疑似乱数は安全であるとします。これをより厳密に言うと次のようになります。

> **識別不可能性 (Indistinguishability)**
> 与えられた 2 つの確率分布 $A$ と $B$ がどのような確率的多項式時間アルゴリズムの識別器でも十分に近ければ (確率の対数の差が極限で 0 となるならば) 計算量的識別不可能であるという。

> **疑似乱数性**
> ある確率分布 $G$ と一様分布が計算量的識別不可能なら $G$ は疑似乱数性を持つという。

疑似乱数性を満たすなら安全な疑似乱数であるとすればよさそうです。

## ハッシュ関数
信頼できないソースの正当性を証明するものというのは世界中で必要とされています。

> **ハッシュ関数**
>

CRC

> **暗号学的ハッシュ関数**
>

- MD5
- [SHA; Secure Hash Algorithm](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.180-4.pdf)
  - 現代で最も使われている

暗号ではパスワードの保存や HMAC, 署名などに使われていたりします。ただ暗号で使うためには攻撃者がハッシュ値に対する元の入力に関する情報を得られないようにしないといけません。これを原像計算困難性といいます。

![](/images/hash.png)

これを満たしたハッシュというのは実際にあります。MD5 や SHA1, SHA-256 などなど、まぁそれぞれのハッシュの実装は結構荒っぽく作られてるので詳細は省きます。ただし攻撃するときの重要な性質として Merkle-Damgård construction というものがあるのでそれだけ知っておきましょう。

### Merkle-Damgård construction
ハッシュ関数は任意長のメッセージを固定長の出力に変換しないといけません。その為に入力を固定長のブロックに分割し、1つずつ内部状態に適用させます。
![](/images/length_extension.png)

MD5 や SHA-1 などよく使われるハッシュ関数はこれです。このときに成立する攻撃というのが伸長攻撃です。伸長攻撃には慎重につってね！フゥーワッ！

## ハッシュの応用
まず SHA-256 などの暗号学的ハッシュ関数を使う一番の例としては改ざん検知です。SHA-256 をそのまま貼り付けるのと HMAC

HMAC はハッシュを用いるメッセージ認証コード (Message Authentication Codes; MAC) で改ざん検知を行います。

> **HMAC; Hash-based MAC**
> 秘密鍵 $K$ とハッシュ値長 $B$ として次のように定義します。
>
> $$
\begin{aligned}
pad_{in} & := \overbrace{\mathrm{0x36}\|\cdots\|\mathrm{0x36}}^B \\
pad_{out} & := \overbrace{\mathrm{0x5C}\|\cdots\|\mathrm{0x5C}}^B \\
\mathrm{HMAC}(K, V) & := H(K \oplus pad_{out} \| H(K \oplus pad_{in} \| V))
\end{aligned}
$$

CRC とはビット列を有限体を用いて短いビット列に変換するハッシュ関数です。CRC はデータ転送時の誤り検出に用いられています。ただ CRC は暗号学的ハッシュ関数ではないので改ざんには弱いです。

> **巡回冗長検査 (CRC; Cyclic Redundancy Check)**
> 有限体 $\mathbb{F}_{2^n} \cong \mathbb{F}_2[x]/(f(x))$ 上の関数 $g(m) = mx^n$ をハッシュ関数とする冗長検査を CRC という。

有限体を構成する多項式 $f(x)\in\mathbb{F}_2[x]$ は次数 $n$ の既約多項式です。これを生成多項式といいます。例えば生成多項式が $f(x) = x^4 + x + 1$ (CRC-4-ITU) のときのハッシュ化において入力が $1011000_{(2)}$ なら $x^6 + x^4 + x^3$ と対応して次の計算から出力は $1111_{(2)}$ となります。

$$
\begin{aligned}
g(x^6 + x^4 + x^3) & = (x^6 + x^4 + x^3)x^4 = x^{10} + x^8 + x^7 \\
& = x^3 + x^2 + x + 1 \pmod{x^4 + x + 1}
\end{aligned}
$$

実際に使われる CRC-32 では次の生成多項式を用います。

$$
x^{32} + x^{26} + x^{23} + x^{22} + x^{16} + x^{12} + x^{11} + x^{10} + x^8 + x^7 + x^5 + x^4 + x^2 + x + 1
$$

実装では多項式を反転させてビット演算に落とし込むことで高速化できます。

> **ブロックチェーン**

## ハッシュへの攻撃
原像計算が攻撃

### データベース
大量の種類の平文とそのハッシュ値をデータベースに入れてハッシュ値から平文を出力するようなシステムを構成でき、それを逆ハッシュと言います。

### 伸長攻撃 (Length Extension Attack)
$H(m_1)$ から $H(m_1\|m_2)$ を求める

https://github.com/bwall/HashPump
https://pypi.org/project/hashpumpy/1.0/

### 誕生日攻撃 (Birthday Attack)
誕生日のパラドックスを用いた攻撃です。

> **Def. ハッシュ値の衝突**
> $H(m_1) = H(m_2)$ となる異なる $m_1, m_2$ が発見された状態

この衝突を恣意的に起こせるとハッシュ値で改ざんを検知してるシステムを騙すことが出来てしまいます。これは誕生日攻撃を用いて恣意的に起こすことができます。

誕生日攻撃というのはハッシュ値のビット数を $n$ として平文 $m$ とそのハッシュ値 $H(m)$ の対応を $O(2^{n/2})$ 程度集めるとハッシュが同じとなるような $m$ が $50\%$ の確率で見つかるという誕生日のパラドックスによる攻撃です。

https://github.com/corkami/collisions
https://github.com/cr-marcstevens/hashclash

### Differenctial Cryptoanalysis
暗号を解析しながら SMT を用いて解く
https://github.com/aappleby/smhasher/wiki/MurmurHash2Flaw

ランダムオラクルモデル
スタンダードモデル

## まとめ
また[準同型のハッシュ関数](https://github.com/benwr/bromberg_sl2)などもあり、発展も期待です。

## 参考文献
- [Length Extension Attackの原理と実装](https://ptr-yudai.hatenablog.com/entry/2018/08/28/205129)
- https://vividot-de.fi/entry/beanstalk-exploit
- [メルセンヌ・ツイスタをわかった気になる](https://6715.jp/posts/5/)
- [Mersenne Twisterの出力を推測してみる](https://inaz2.hatenablog.com/entry/2016/03/07/194147)

この資料は CC0 ライセンスです。