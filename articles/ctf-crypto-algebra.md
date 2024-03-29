---
title: "【CTF 探訪記】計算機代数"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CTF", "crypto"]
published: false
---

- 素数生成と素因数分解
- Grobner
# はじめに

すぐそこに秘密の世界がある。この記事を読んでいるなら、あなたは暗号化された通信を通してサーバーから電気線を通り、遥々その端末に届いている文字を読んでいます。
こんなことを考えてほしい。機器から暗号を使って通信をしていますが、私達が触れていないだけで暗号は誰でも解けるものであり、近くを通った人が自分の通信を盗聴しているかもしれない。ゾクッとしませんか。

ってことで暗号の中でも指折りの世界で活躍している「RSA暗号」を詳しくみて本当の世界をいきましょう。

CTFの中でもCryptoはメタ的な能力よりかは数学力に重きが置かれる分野です。最近では発想だけではなくテクニックを知らないと解けないような問題が多くなってきました。そこでこの記事ではRSA関連の問題でよく使うテクニックを体系的に1から習得します。

テクニック
gcd, 中国剰余定理, 終結式, グレブナー基底, LLL, Coppersmith

# 時計の世界

様々な暗号を語る上で最も根幹を成すのが、Nを自然数として「Nを法とする」算術です。

この算術は日々触れていて例えば時間について考えてみましょう。

午前10時に仕事を始めて、8時間働くとすると、仕事が終わるのはいつでしょう。

10 + 8 = 18 だから「18時に終わる」と答えるのが自然だろう。特に18から12を引いて 18 - 12 = 6 で午後6時に終わるとも答える。また角度でもこの算術が出てくきます。2つの角度の和が360度よりも大きければ、そこから360を引いて、1から360までの間に収まるようにします。例えば 450 - 360 = 90 だから、450度の回転は90度の回転に等しい。

このように私達は時間についても角度と同じようなタイプの計算を使っています。これを時間であれば「12を法とする」足し算、角度であれば「360を法とする」足し算をしています。

同様に任意の自然数 $N$ について $N$ を法とする足し算をすることができます。
0からN-1までの数を集めた集合を定義します。

$$
\mathbb{Z}/N\mathbb{Z} = \{0, 1, ..., N - 2, N - 1\}
$$

例えば $\mathbb{Z}/12\mathbb{Z}$ はAM/PMの時間を表す数となります。

これらの数の集合に対し、加法(足し算)を定義します。この集合の中から任意の2つの数が与えられたとして、それらを足し算したものが N より大きくなるなら、ここから N を引き算して結果がこの中に入るようにする。この演算を入れることでこの集合は群となります。

加法の単位元   $0$
加法の逆元     $-a = N - a$
加法の結合法則 $(a + b) + c = a + (b + c)$

乗法(掛け算)も定義できますが群にはなりません。乗法の逆元が常に存在する訳ではないからです。例えば0やNの約数となる場合です。

乗法の単位元   $1$
乗法の逆元     $aa^{-1} = 1$
乗法の結合法則 $(a \times b) \times c = a \times (b \times c)$

具体的にどういうときに逆元が存在しないのかを考えてみます。

mod N での a の逆元が存在する条件は、N と a とが互いに素であることです。

$$
\begin{aligned}
ax &= 1 & \pmod N \\
ax - kN &= 1 \\
(x, k)
\end{aligned}
$$

mod 12 なら 1, 5, 7, 11
mod 13 なら 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12

一般にNが素数pのときは $\mathbb{Z}/p\mathbb{Z}$ から0を除いた集合 $(\mathbb{Z}/p\mathbb{Z})^\times$ に対して乗法も群になります。

$$
(\mathbb{Z}/p\mathbb{Z})^\times = \{1, 2, ..., p - 2, p - 1\}
$$

> def. 累乗根
ある数 $a$ に対し $x^2 = a \pmod N$ となるような $x$ を $a$ の平方根 $\sqrt{a}$ とする。また $x^n = a \pmod N$ となる $x$ を $a$ の累乗根 $\sqrt[n]{a}$ とする。

> def. 対数
ある数 $e, a$ に対し $e^x = a \pmod N$ となるような $x$ を $e$ を底とする $a$ の対数 $\log_e{a}$ とする。

$$
\begin{aligned}
3 + 4 &= 0 & \pmod 7 \\
3 - 4 &= -1 = 6 & \pmod 7 \\
3 \times 4 &= 12 = 5 & \pmod 7 \\
3 \div 4 &= 3 \times 4^{-1} = 3 \times 2 = 6 & \pmod 7 \\
3^4 &= 9^2 = 2^2 = 4 & \pmod 7 \\
\sqrt{4} &= 2, 5 & \pmod 7 \\
\sqrt[3]{6} &= 3, 5, 6 & \pmod 7 \\
\log_3{6} &= 3 & \pmod 7 \\
\end{aligned}
$$

さてコンピューターはそれぞれどのようにして計算するのでしょうか。以下の表のようになります。

| 演算                   | 方法                          | 計算量                      |
| :--------------------  | :---------------------------- | :-------------------------: |
| 足し算   $a + b$         | 足してN以上になったらN引く    | $\mathcal{O}(1)$            |
| 引き算   $a - b$         | 引いて0未満になったらN足す    | $\mathcal{O}(1)$            |
| 掛け算   $a \times b$    | 掛けてNで割った余り           | $\mathcal{O}(1)$            |
| 割り算   $a \div b$      | 拡張ユークリッドの互除法      | $\mathcal{O}((\log N)^2)$   |
| 累乗     $a ^ b$         | 繰り返し二乗法                | $\mathcal{O}(\log N)$       |
| 平方剰余               | $(p-1)/2$ 乗                  | $\mathcal{O}(\log N)$       |
| 平方根   $\sqrt{a}$    | Tonelli Shanksのアルゴリズム  | $\mathcal{O}((\log N)^2)$   |
| 累乗根   $\sqrt[e]{a}$ | 離散対数問題                  | $\mathcal{O}(\sqrt{N})$     |
| 対数     $\log_e{a}$   | 離散対数問題                  | $\mathcal{O}(\sqrt{N})$     |

それぞれの具体的な方法はググれば良解説が山ほど出てくるのでそちらを参考にしてください。特に [整数論のpdf](https://kirika-comp.hatenablog.com/entry/2018/03/12/210446) がおすすめです。

上のように剰余上での計算は $\bmod N$ と書いていますが、逆によく使っている整数 $\mathbb{Z}$ 上での演算は $\mathrm{over}\ \mathbb{Z}$ と書くことにします。

// TODO
$\mathrm{over}\ \mathbb{Z}$ -> $\bmod N$
$\bmod N$ -> $\mathrm{over}\ \mathbb{Z}$

CTFの問題

まとめ
二つの世界 $\bmod N, \mathrm{over}\ \mathbb{Z}$ を学んだ。世界線を変えていけ。

# 剰余上の掛け算と足し算

剰余のことが少しわかった所で累乗について少し深く掘り下げてみます。

累乗に関する有名な定理としてフェルマーの小定理(最終定理じゃないよ)があります。

フェルマーの小定理
$p$ が素数であり、かつ $a$ と $p$ が互いに素のとき次の式が成り立つ。

$$
a^{p-1} = 1 \pmod p
$$

数式だけだとあまり良くわからないので具体的に $\bmod 7$ で考えてみます。 $a^k$ を計算した表がこちらになります。

| k ＼ a |  1  |  2  |  3  |  4  |  5  |  6  |
|:-----:|:---:|:---:|:---:|:---:|:---:|:---:|
|   1   |  1  |  2  |  3  |  4  |  5  |  6  |
|   2   |  1  |  4  |  2  |  2  |  4  |  1  |
|   3   |  1  |  1  |  6  |  1  |  6  |  6  |
|   4   |  1  |  2  |  4  |  4  |  2  |  1  |
|   5   |  1  |  4  |  5  |  2  |  3  |  6  |
|   6   |  1  |  1  |  1  |  1  |  1  |  1  |

このようにどんな $a$ を使って計算しても必ず $a^6 = 1$ になります。これが一般の素数 $p$ で法を取ったなら必ず $p-1$ 乗で $1$ となるという不思議な定理です。これには様々な証明方法があるので他の方の良解説を読むと良いです。こちらは特に [高校数学の美しい物語](https://manabitimes.jp/math/680) がおすすめです。

ここではこの定理についてもう少し膨らませてみます。

始めに必要な記号について定義します。数学では $\bmod N$ は $\mathbb{Z}/N\mathbb{Z}$ と書きます。そして掛け算/割り算で良い性質を満たすように $\mathbb{Z}/N\mathbb{Z}$ から $0$ だけを抜いた剰余を $(\mathbb{Z}/N\mathbb{Z})^\times$ と書きます。また、暗黙の了解として変数名を $p$ や $q$ と書いたらそれは素数だと思ってください。

ここで $(\mathbb{Z}/p\mathbb{Z})^\times$ に対して対数を取ります。具体例として $(\mathbb{Z}/11\mathbb{Z})^\times$ 上で $\log_2 n$ を計算してみます。

| $n\in(\mathbb{Z}/11\mathbb{Z})^\times$ |   1   |   2   |   3   |   4   |   5   |   6   |   7   |   8   |   9   |  10   |
|:----------------------------------:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|
|               $n = 2^k$                | $2^0$ | $2^1$ | $2^8$ | $2^2$ | $2^4$ | $2^9$ | $2^7$ | $2^3$ | $2^6$ | $2^5$ |
|              $\log_2 n$              |   0   |   1   |   8   |   2   |   4   |   9   |   7   |   3   |   6   |   5   |

じっくり見てみると、対数を取ったもの同士の $\bmod 10$ 上での足し算はそれに対応する値同士の $\bmod 11$ 上の掛け算と一致しているということが見えてきます。例えば、対数を取った数 $8, 4$ を足すと $8 + 4 = 12 = 2 \pmod {10}$ ですが、それに対応する値 $3, 5$ を掛けると $3 \times 5 = 15 = 4 \pmod {11}$ となり、$2 = \log_2{4}$ と対応しています。

証明
剰余上の指数法則 $2^a \times 2^b = 2^{a+b} \pmod p$ は明らか。また、原始根 $a$ であれば $a^k \bmod{p} \ (k=0, \ldots, p-2)$ はそれぞれ違う値を取り、フェルマーの小定理より $a^{p-1} = 1 \pmod{p}$ であるから、 $\bmod p-1$ が必要十分条件となります。つまり任意の $p$ に対して $(\mathbb{Z}/p\mathbb{Z})^\times$ で原始根が存在することを言えれば積と和が対応していることが言えます。任意の素数の法について原始根が必ず存在するという定理については少し長くなるので他の記事をぜひ参考にしてください。

これで一般の $\bmod p$ で掛け算は対数を取れば $\bmod p - 1$ の足し算に変換できるということが成り立つということが分かりました。このような性質を同型と呼び、記号 $≅$ を使って表します。

$$
(\mathbb{Z}/p\mathbb{Z})^×≅\mathbb{Z}/(p−1)\mathbb{Z}
$$

また、法の数を素数から一般の自然数に一般化することができ、これをカーマイケルの定理と呼びます。証明は [せきゅーんさんの記事](https://integers.hatenablog.com/entry/2016/07/24/163831) [記事2](https://integers.hatenablog.com/entry/2017/06/08/191649)がおすすめです。

$$
\begin{aligned}(\mathbb{Z}/p_1^{e_1}\ldots p_n^{e_n}\mathbb{Z})^× &≅ \prod_{i=1}^{n}(\mathbb{Z}/p_i^{e_i}\mathbb{Z})^× \\
(\mathbb{Z}/2^e\mathbb{Z})^× &≅ \left\{
\begin{array}{ll}
\mathbb{Z}/1\mathbb{Z} & (e = 1) \\
\mathbb{Z}/2\mathbb{Z} & (e = 2) \\
\mathbb{Z}/2\mathbb{Z} \times \mathbb{Z}/2^{e-2}\mathbb{Z} & (e \geq 3)
\end{array}
\right.\\
(\mathbb{Z}/p^e\mathbb{Z})^× &≅ \mathbb{Z}/p^{e-1}(p−1)\mathbb{Z} \\
(\mathbb{Z}/2^ep_1^{e_1}\ldots p_n^{e_n}\mathbb{Z})^×&≅\mathbb{Z}/\mathrm{lcm}(2^{e-2}, p_1^{e_1-1}(p_1−1), \ldots , p_n^{e_n-1}(p_n−1))\mathbb{Z} & (e \geq 3)\\
\end{aligned}
$$

こうして掛け算を足し算に置き換えられ、問題が簡単になることが多いです。

具体例
RSA暗号はこれを使えば簡単に解けます。
$c = m^e \pmod {pq}$ という式について原始根 $a$ で対数を取るとカーマイケルの定理より

$$
\begin{aligned}
c &= m^e & \pmod {pq} \\
\log_a c &= \log_a{m^e} = e\log_a{m} & \pmod{\mathrm{lcm}(p-1, q-1)}\\
\log_a m &= (\log_a{c})/e & \pmod{\mathrm{lcm}(p-1,q-1)} \\
m &= a^{(\log_a{c})/e} & \pmod{pq}
\end{aligned}
$$

と計算できます。

ここまでは数学上の話でしたが、ここから計算機科学に移ります。

確かに、掛け算を足し算に変換することで簡単な問題になります。しかし、それ以前にコンピュータが剰余上の対数を計算する事は結構難しく、離散対数問題 (DLP: Discrete Logarithm Problem) と呼ばれ、現在見つかっている最も速いアルゴリズムでも指数時間が掛かります。

では簡単に解くことを諦める、という事はなく、他に1つ方法があります。直接数に対応させなくても、ある数だけ累乗すると1や-1になることを使うことである程度情報を引き出すことができます。

例えば Tonelli Shanks のアルゴリズムは平方剰余を使っています。

$(\mathbb{Z}/13\mathbb{Z})^×≅\mathbb{Z}/12\mathbb{Z}≅\mathbb{Z}/2^2\mathbb{Z}\times\mathbb{Z}/3\mathbb{Z}$

| $\times$ |  0  |  1  |  2  |
|:--------:|:---:|:---:|:---:|
|    0     |  0  |  4  |  8  |
|    1     |  9  |  1  |  5  |
|    2     |  6  | 10  |  2  |
|    3     |  3  |  7  | 11  |

(逆に累乗を求めることは簡単という非対称性を用いた暗号が ElGamal暗号 です。(ここでは紹介しません))

まとめ
$\bmod pq$ -> $\bmod \mathrm{lcm}(p-1, q-1)$
$\mathbb{Z}/N\mathbb{Z} = \{0, 1, 2, \ldots , N - 1\}$
$(\mathbb{Z}/p\mathbb{Z})^×≅(\mathbb{Z}/(p−1)\mathbb{Z})^+$
カーマイケルの定理
DLP
平方剰余が

### 素数生成

暗号として機能する素数の大きさは $2^{512}$ や $2^{1024}$ 程度のオーダーとなっています。素数定理より、ある数 $n$ が素数である確率は約 $1/\log n$ です。例えば $n=2^{512}$ で2.8%、 $n=2^{1024}$ で1.4%となります。つまり、500回乱数を生成すれば99.65%で素数を見つけられるということです。
素数判定のアルゴリズムは多くありますが、ここではMiller-Rabin素数判定法を紹介します。

#### Miller–Rabin 素数判定法

素数判定法とはその名の通り、数を与えるとそれが素数かどうかが分かる判定法です。その中で Miller-Rabin 素数判定法は与えられた数 $n$ が素数かどうかを計算時間 $O(k\log^3 n)$ で誤り率 $4^{-k}$ 以下で判定する確率的素数判定アルゴリズムです。

$n$ が素数のとき、$n-1$ はそれを $2$ で割れるだけ割った数を $d$ として $n-1 = 2^sd$ と書けます。フェルマーの小定理より $a≠0 \pmod n$ のとき

$$
\begin{aligned}
a^{n-1} &= a^{2^sd} ≡ 1 \quad \pmod n \\
a^{2^sd}-1 &= (a^d-1)(a^d+1)(a^{2d}+1)(a^{4d}+1)\cdots(a^{2^{s-1}d}+1)\\
&≡ 0 \\
\end{aligned}
$$

これより次の2式のどちらかが成り立ちます。

$$
\begin{aligned}
\left\{
\begin{array}{ll}
a^d &≡ 1 & \pmod n \\
a^{2^rd} &≡ -1 & \pmod n \qquad (\exists r \in \mathbb{Z}, 0\leq r\leq s-1)
\end{array}
\right.
\end{aligned}
$$

この対偶をとると、「ある $a$ をとってきて次の2式をどちらも満たすとき

$$
\begin{aligned}
\left\{
\begin{array}{ll}
a^d &\neq 1 & \pmod n\\
a^{2^rd} &\neq -1 & \pmod n \qquad (\forall r \in \mathbb{Z}, 0\leq r\leq s-1)
\end{array}
\right.
\end{aligned}
$$

$n$ は合成数である」と言えます。

これを用い、次のステップを実行することで確率的な素数判定ができます。
1. $1\leq a \leq n-1$ でaの値をランダムにとってくる。
2. 上の条件を満たしたらcompositeと返す。
3. 満たさなければprobably primeと返す。

これを繰り返すことで判定の精度が高まります。この処理をMiller–Rabin素数判定法といって、実行時間は $O(k\log^3 n)$ 、FFTベースの乗算で $Õ(k\log^2 n)$ となります。

具体例を考えてみましょう。
判定時にprobably primeを返す時 p、compositeを返す時 c として具体値を入れると次のようになります。
$n = 25$ (合成数)のとき
$n-1 = 24 = 2^3 \times 3$ より $s = 3, d = 3$

| a | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 | 17 | 18 | 19 | 20 | 21 | 22 | 23 | 24 |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| $a^3 \bmod 25$ | 1 | 8 | 2 | 14 | 0 | 16 | 18 | 12 | 4 | 0 | 6 | 3 | 22 | 19 | 0 | 21 | 13 | 7 | 9 | 0 | 11 | 23 | 17 | 24 |
| $a^6 \bmod 25$ | 1 | 14 | 4 | 21 | 0 | 6 | 24 | 19 | 16 | 0 | 11 | 9 | 9 | 11 | 0 | 16 | 19 | 24 | 6 | 0 | 21 | 4 | 14 | 1 |
| $a^{12} \bmod 25$ | 1 | 21 | 16 | 16 | 0 | 11 | 1 | 11 | 6 | 0 | 21 | 6 | 6 | 21 | 0 | 6 | 11 | 1 | 11 | 0 | 16 | 16 | 21 | 1 |
| 判定 | p | c | c | c | c | c | p | c | c | c | c | c | c | c | c | c | c | p | c | c | c | c | c | p |

$n = 17$ (素数)のとき
$n-1 = 16 = 2^4 \times 1$ より $s = 4, d = 1$

| a | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| $a \bmod 17$ | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 |
| $a^2 \bmod 17$ | 1 | 4 | 9 | 16 | 8 | 2 | 15 | 13 | 13 | 15 | 2 | 8 | 16 | 9 | 4 | 1 |
| $a^4 \bmod 17$ | 1 | 16 | 13 | 1 | 13 | 4 | 4 | 16 | 16 | 4 | 4 | 13 | 1 | 13 | 16 | 1 |
| $a^8 \bmod 17$ | 1 | 1 | 16 | 1 | 16 | 16 | 16 | 1 | 1 | 16 | 16 | 16 | 1 | 16 | 1 | 1 |
| 判定 | p | p | p | p | p | p | p | p | p | p | p | p | p | p | p | p |

かなり正確に判定できていることがわかるでしょう。

素数については必ず成功し、合成数のときは誤る可能性が $1/4$ 以下ということが示せるので、 $k$ 回試行すれば誤り率は $4^{-k}$ 以下となります。つまり、ある値に対して10回素数判定法を回せば99.9999046%成功するということです。


ここが一番大きな関門です。ここを乗り切ればあとは楽です。

暗号では楕円曲線上の点や格子上の点などを考えたりしますが、それらには足し算や引き算ができたりして、数ではないが「数っぽいもの」がちらほら出てきます。それぞれの数っぽいものを研究してみると似たような性質がちらほらでてきます。これらに共通する性質を研究するのが代数学のモチベーションです。

- 群論
- 計算可能性

について解説します。

資料の厳密さはその情報の信頼度を表していると思っています。しかし全てを厳密に書いてしまうと文章が記事では収まらなくなってしまいますので非本質的な部分は曖昧にしたり省きます。何が厳密で何が厳密ではないかは明確に書くつもりなのでそこは安心してください。厳密な証明や実装が気になった人は参考文献の本や規格書、元論文などを読んでみると良いと思います。

## 記号
$\mathbb{B} = \lbrace 0, 1\rbrace$: バイナリの値の集合
$\mathbb{N}$: 自然数の集合
$\mathbb{Z}$: 整数の集合
$\mathbb{Q}$: 有理数の集合
$\mathbb{R}$: 実数の集合
$\mathbb{C}$: 複素数の集合

## 群論の基礎

> **Def. 群**
> 空集合でない集合 $G$ と $G$ 上の演算が定義されていて次を満たすとき $G$ は群であるという。
> 1. 単位元と呼ばれる元 $e\in G$ があり、任意の元 $a\in G$ に対し $ae = ea = a$ となる。この $e$ は $e$ や $1$ のように書く。$G$ の群であることを強調して $1_G$ と書くこともある。
> 2. 任意の元 $a\in G$ に対し逆元と呼ばれる元 $b\in G$ があり、$ab = ba = e$ となる。 この $b$ を $a^{-1}$ と書く。
> 3. 結合法則 $a(bc) = (ab)c$

具体例

1. $\mathbb{N}$ 上の加法について単位元は $0$ で結合法則は成り立ちますが、一般に逆元が存在しないので群とはなりません。$\mathbb{Z}$, $\mathbb{Q}$, $\mathbb{R}$, $\mathbb{C}$ は加法について可換群となります。
2. $\mathbb{Z}\setminus\lbrace 0\rbrace$ 上の乗法について単位元は $1$ で結合法則は成り立ちますが、一般に逆元が存在しないので群とはなりません。$\mathbb{Q}\setminus\lbrace 0\rbrace$, $\mathbb{R}\setminus\lbrace 0\rbrace$, $\mathbb{C}\setminus\lbrace 0\rbrace$ は乗法について可換群となります。これらの集合を $\mathbb{Q}^\times$, $\mathbb{R}^\times$, $\mathbb{C}^\times$ と書きます。
3. 平行移動・回転対称性

群 $G$ と $a\in G$, $n\in\mathbb{N}$ として

$$
\begin{aligned}
  a^0 &:= 1_G \\
  a^n &:= \overbrace{a\cdots a}^{n} \\
  a^{-n} &:= (a^n)^{-1}
\end{aligned}
$$

そして結合法則より指数法則 $a^m \cdot a^n = \overbrace{a\cdots a}^{m}\cdot \overbrace{a\cdots a}^{n} = a^{m + n}$

$(xy)z$ の $z$ を前に持ってきて $z(xy)$ などとすることは一般にはできません。

> **Def. 部分群**
> ある群 $G$ の部分集合 $G'$ が同じ演算において群を成すとき $G'$ は $G$ の部分群であるという。

> **Prop. 部分群の条件**
> 単位元逆元

> **Def. 巡回群**
> 群 $G$ についてある元 $g\in G$ を用いて $G = \lbrace g^n\mid n\in\mathbb{Z}\rbrace$ となるとき、$G$ を巡回群と呼ぶ。

> **Prop.**
> 巡回群 $G$ において任意の元 $g\in G$ で $g^{|G|} = 1$ が成り立つ。

フェルマーの小定理やオイラーの定理やカーマイケルの定理は上記の命題により成り立つ(後ほど証明する)。またねじれ群

> **繰り返し二乗法**
> 巡回群 $G$ において元 $g\in G$ を用いて $g^n$ を $\mathcal{O}(k\log{n})$ で求められる。ただし 1 回の演算に $\mathcal{O}(k)$ 掛かるとする。

**Proof.**

> **Def. 同値関係**
> 集合 $S$ 上の関係 $\sim$ が次の条件を満たすとき $\sim$ を同値関係という。
> 1. 反射律 $a\sim a$
> 2. 対称律 $a\sim b\implies b\sim a$
> 3. 推移律 $a\sim b, b\sim c\implies a\sim c$
>
> 同値関係 $\sim$ について $x\in S$ に対し
>
> $$C(x) = \lbrace y\in S\mid y\sim x\rbrace$$
>
> を同値類といい、同値類全体の集合を同値関係による商 $S/\sim$ と呼ぶ。

> **Def. 剰余類**
> $G_2$ を群 $G_1$ の部分群とする。$x\sim y$ を $x^{-1}y\in G_2$ と定義したとき、同値関係 $\sim$ による $x\in G_2$ の同値類を左剰余類といい $xH$ と書く。またその同値関係による商を $G/H$ と書く。
> 同様に $x\sim y$ を $yx^{-1}\in G_2$ と定義したとき、同値類を右剰余類といい $Hx$ と書き、同値関係による商を $H\backslash G$ と書く。

> **Prop.**
> $G_2$ が群 $G_1$ の部分群とおく。このとき次が成り立つ。
> 1. $|G/H| = |H\backslash G|$
> 2. $\forall g\in G\ |gH| = |Hg| = |H|$

**Proof.**
写像 $\alpha: G/H\to H\backslash G$ を $\alpha(gH) = Hg^{-1}$ と定義する。

> **Thm. ラグランジュの定理**
>
> $$|G| = (G:H)|H|$$

> **Def. 正規部分群**

> **Def.**
> 準同型とは取れる
> 同型とは

確認ですが 2 つの群 $G_1, G_2$ について同型 $G_1\cong G_2$ と同値 $G_1 = G_2$ は違います。同値が集合や演算の中身までみて等しいことですが、同型は中身を気にせず全単射準同型が張れるということが定義です。使い分けましょう。

> **Thm. 準同型定理**
> 群 $G_1, G_2$ とその間に準同型 $\phi: G_1 \to G_2$ があるとするとき次が成り立つ。
>
> $$G_1/\mathrm{Ker}(\phi) \cong \mathrm{Im}(\phi)$$

### 中国剰余定理

> **Thm. 中国剰余定理 (CRT; Chinese Remainder Theorem)**
> $m, n \neq 0$ が互いに素な整数なら、$\mathbb{Z}/mn\mathbb{Z} \cong \mathbb{Z}/m\mathbb{Z}\times\mathbb{Z}/n\mathbb{Z}$

**Proof.**
写像 $\phi: \mathbb{Z}/mn\mathbb{Z} \to \mathbb{Z}/m\mathbb{Z}\times\mathbb{Z}/n\mathbb{Z}$ を次のように定義する。

$$
\phi(x + mn\mathbb{Z}) = (x + m\mathbb{Z}, x + n\mathbb{Z})
$$

これは準同型となる。また $m, n$ が互いに素であるから $ma + nb = 1$ となる $a, b$ が存在する。ここで任意の $x, y$ に対し $z = may + nbx$ とおくと

$$
\begin{aligned}
  z & = may + (1 - ma)x = x + ma(y - x) &&\in x + m\mathbb{Z} \\
    & = (1 - nb)y + nbx = y + nb(x - y) &&\in y + n\mathbb{Z}
\end{aligned}
$$

となる。これを元に写像 $\psi(x + m\mathbb{Z}, y + n\mathbb{Z}) = z + mn\mathbb{Z}$ を構成すると $\psi$ は $\phi$ の逆写像であり、$\phi$ は全単射となる。よって $\phi$ は同型写像であり、$\mathbb{Z}/mn\mathbb{Z}$ と $\mathbb{Z}/m\mathbb{Z}\times\mathbb{Z}/n\mathbb{Z}$ は同型である。 $\Box$

これより次のことが言えます。

> **Prop.**
> $n = p_1^{e_1}\cdots p_k^{e_k}$ と素因数分解出来るとき、次が成り立つ。
>
> $$\mathbb{Z}/n\mathbb{Z} \cong \mathbb{Z}/p_1^{e_1}\mathbb{Z}\times\cdots\times\mathbb{Z}/p_k^{e_k}\mathbb{Z}$$

例えば $\mathbb{Z}/15\mathbb{Z} \cong \mathbb{Z}/3\mathbb{Z}\times\mathbb{Z}/5\mathbb{Z}$ となるので法が15の数と法が3, 5の数のペアは1対1に対応させることができます。

| $\times$ |  0  |  1  |  2  |  3  |  4  |
|:--------:|:---:|:---:|:---:|:---:|:---:|
|    0     |  0  |  6  | 12  |  3  |  9  |
|    1     | 10  |  1  |  7  | 13  |  4  |
|    2     |  5  | 11  |  2  |  8  | 14  |

数自体だけではなく加法、乗法についても対応します。

$$
\begin{aligned}
8 &+ 9 = 2 & \pmod{15} \\
&\downarrow \\
(2, 3) &+ (0, 4) = (2, 2) & \pmod{(3, 5)} \\
\end{aligned}
$$

$$
\begin{aligned}
8 &\times 9 = 12 & \pmod{15} \\
&\downarrow \\
(2, 3) &+ (0, 4) = (0, 2) & \pmod{(3, 5)} \\
\end{aligned}
$$

![](/images/crt.jpg)
// TODO 3 5 でやると分かりやすい

大きな剰余ではなく複数の小さな剰余に分割して考えた方が探索すべき数は圧倒的に減ります。このような考え方を Crypto ではよく行います。
その為には相互に剰余を変換できなければ話になりません。

大きな剰余 $n$ と複数の小さな剰余 $m_i (i=1,\ldots,k)$ について $m_i|n$ が成り立つとします。

まず小さくするときは $(x \bmod n)\bmod m_i$ とすればよいです。これを**還元**と呼びます。

$$
\begin{aligned}
a \bmod{pq} \bmod{p} &= (a - k_1pq) - k_2p \\
&= a - (k_1q + k_2)p \\
&= a \bmod{p} \\
\end{aligned}
$$

となるからです。注意すべきなのは2つが約数の関係となる剰余でしかこのような式は有効ではないです。例えば有効ではない式として $20 \bmod 15 \bmod 9 \neq 20 \bmod 9$ があります。

逆に小さな剰余から大きな剰余にするにはどうやって計算すればいいでしょうか。このように小さな群から大きな群へ移す操作は**持ち上げ (lift)** とよばれていて、この群を持ち上げるには次の多項式時間のアルゴリズムが知られています。

> **Garner のアルゴリズム**
> 整数 $m_1,\ldots,m_k$ に対し、ある整数 $x\in [0, \mathop{\mathrm{lcm}} m_i)$ の $m_i$ に関する剰余 $r_i = x \bmod m_i$ が与えられれば $x$ を $\mathcal{O}(k\log(\max m_i) + k^2)$ で求められる。

基本的なアイデアとしては互いに素な剰余 $m_1, m_2$ に対して

$$
\begin{aligned}
x & = q_1m_1 + r_1 \\
r_2 & = q_1m_1 + r_1 & \pmod{m_2} \\
q_1 & = (r_2 - r_1)m_1^{-1} & \pmod{m_2} \\
x & = (r_2 - r_1)(m_1^{-1} \bmod{m_2})m_1 + r_1 & \pmod{m_1m_2} \\
\end{aligned}
$$

例えば「3 で割ったあまりが 2」かつ「5 で割ったあまりが 3」であるようなものは $2 + (3 - 2)2\times3 = 8 \pmod{15}$ となります。

これについては良記事があります。
https://qiita.com/drken/items/ae02240cd1f8edfc86fd

:::message
**練習問題**
- $(ap)^e \pmod{pq}$ は $p$ で割り切れることを中国剰余定理によって証明せよ。
- 剰余 $m_i$ が互いに素ではないときの Garner のアルゴリズムを実装せよ。
:::

### 乗法群

> **Prop.**
> $p$ を素数とおくと
>
> $$
(\mathbb{Z}/p\mathbb{Z})^\times \cong \mathbb{Z}/(p−1)\mathbb{Z}
$$

**Proof.**
$(\mathbb{Z}/p\mathbb{Z})^\times$ において位数 $p - 1$ の元 (原始根) が存在することを示す。
まず $n$ が $p - 1$ の約数であるとき $x^n = 1$ は $n$ 個の解を持つことを示す。仮定より $p - 1 = nk$ とおけ、次のそれぞれの式について解の個数について考える。

$$
x^{p-1} - 1 = (x^n - 1)((x^n)^{k-1} + \ldots + x^n + 1)
$$

$x^{p-1} - 1 = 0$ はフェルマーの小定理より $p - 1$ 個
$(x^n)^{k-1} + \ldots + x^n + 1 = 0$ は代数学の基本定理より $n(k-1)$ 個以下
よって $x^{n} - 1 = 0$ は解の個数を比較して $n$ 個存在する。

これより $p-1$ と互いに素な数の個数だけ原始根が存在する。

原始根 $a$ を1つ選び、写像 $\phi: \mathbb{Z}\to(\mathbb{Z}/p\mathbb{Z})^\times$ を $\phi(k) = a^k$ とすると、これは準同型である。全射である。$\mathrm{Ker}(\phi) = (p-1)\mathbb{Z}$。よって $(\mathbb{Z}/p\mathbb{Z})^\times \cong \mathbb{Z}/(p−1)\mathbb{Z}$ である。$\Box$

具体例として $\mathbb{Z}/7\mathbb{Z}$ での対数 $\log_3 n$ について考えてみます。

|     $n$    |   $1$   |   $2$   |   $3$   |   $4$   |   $5$   |   $6$   |
|:----------:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|
|  $n = 3^k$ | $3^0$ | $3^2$ | $3^1$ | $3^4$ | $3^5$ | $3^3$ |
| $\log_3 n$ |   $0$   |   $2$   |   $1$   |   $4$   |   $5$   |   $3$   |

じっくり見てみると、対数を取ったもの同士の $\mathbb{Z}/6\mathbb{Z}$ での足し算はそれに対応する値同士の $\mathbb{Z}/7\mathbb{Z}$ の掛け算と一致しているということが見えてきます。

$$
\begin{aligned}
4 &\times 6 = 3 \pmod 7 \\
&\downarrow \log_3 \\
4 &+ 3 = 1 \pmod 6 \\
\end{aligned}
$$

知っておくと便利な定理があります。

> **Thm. Carmichael の定理**
>
> $$
\begin{aligned}
(\mathbb{Z}/p_1^{e_1}\ldots p_n^{e_n}\mathbb{Z})^\times &\cong (\mathbb{Z}/p_1^{e_1}\mathbb{Z})^\times \times \ldots \times (\mathbb{Z}/p_n^{e_n}\mathbb{Z})^\times\\
(\mathbb{Z}/p^e\mathbb{Z})^× &\cong \begin{cases}
\lbrace 1\rbrace & (p = 2, e = 1) \\
\mathbb{Z}/2\mathbb{Z} \times \mathbb{Z}/2^{e-2}\mathbb{Z} & (p = 2, e \geq 2) \\
\mathbb{Z}/p^{e-1}(p−1)\mathbb{Z} & (p > 2) \\
\end{cases}
\end{aligned}
$$

これについても良記事があります。
https://integers.hatenablog.com/entry/2016/07/24/163831
https://integers.hatenablog.com/entry/2017/06/08/191649
### Tonelli Shanks のアルゴリズム

> **Prop. 平方剰余**
> $\mathbb{Z}/p\mathbb{Z}$ において $n^{(p-1)/2}$ から $n$ の対数 $\mathbb{Z}/(p-1)\mathbb{Z}$ の偶奇を判別できる。

**Proof.**
位数が $p-1$ であるから $n$ の位数が 2 の倍数のとき $n^{(p-1)/2} = 1$、そうでないとき $n^{(p-1)/2} = -1$ となる。$\Box$

| $n$ | $2^0$ | $2^1$ | $2^2$ | $2^3$ | $2^4$ | $2^5$ | $2^6$ | $2^7$ | $2^8$ | $2^9$ | $2^{10}$ | $2^{11}$ |
|:--------:|:-:|:-:|:--:|:--:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| $\sqrt{n}$ | $2^0, 2^6$ | なし | $2^1, 2^7$ | なし | $2^2, 2^8$ | なし | $2^3, 2^9$ | なし | $2^4, 2^{10}$ | なし | $2^5, 2^{11}$ | なし |

中国剰余定理から上のように分解でき、$\bmod 3$ なら $2$ の逆数が定義できるので $2^{-1} \bmod 3  = 2$ 乗します。すると $10^2 = 9 \pmod{13}$ となり、これは $9 = 2^8 \pmod{13}$ で下の表をみるとしっかり $8$ は $5, 11$ のある $2 \bmod 3$ の行にあります。ここから上下に動かさずに左右だけを動かして本当の平方根を求めます。

| $\times$ | 0 | 1 |  2 |  3 |
|:--------:|:-:|:-:|:--:|:--:|
|     0    | 0 | 9 |  6 |  3 |
|     1    | 4 | 1 | 10 |  7 |
|     2    | 8 | 5 |  2 | 11 |

例えば $\sqrt{10} \pmod{13}$ については $\sqrt{10} = \sqrt{2^{10}} = 2^5, 2^{11} = 6, 7 \pmod{13}$ となります。このように DLP を解いて半分にしたものを累乗させれば平方根が求まります。この計算量は $\mathcal{O}(\sqrt{N})$ であり、$N$ のビット数に対して指数時間掛かる。これに対し、多項式時間のアルゴリズムが見つかっている。

> **Tonelli Shanks のアルゴリズム**
> $\mathbb{F}_p$ において平方根を $\mathcal{O}((\log p)^2)$ で求められる。また $k$ 乗根は $\mathcal{O}(\min(p^{1/4}, \sqrt{k})\log k(\log p)^2 + \min(p^{1/4}, \sqrt{k}))$ で求められる。

ここでは平方根のみを考えます。

平方根について定式化すると $x^2 = a \pmod p$ となる $x$ を求める問題です。まず $2$ と $p-1$ は互いに素ではない為、$2^{-1}\pmod{p - 1}$ は計算できません。代わりに中国剰余定理で $2$ の累乗部分だけ分解して

$$
(\mathbb{Z}/p\mathbb{Z})^\times \cong \mathbb{Z}/(p-1)\mathbb{Z} \cong \mathbb{Z}/q\mathbb{Z} \times \mathbb{Z}/2^Q\mathbb{Z}
$$

$x = a^{(q + 1)/2}\pmod{p}$ を計算します。$\mathbb{Z}/q\mathbb{Z} \times \mathbb{Z}/2^Q\mathbb{Z}$ において $a = (a_1, a_2)$ とおくと、$a, x, x^2, a^{-1}x^2$ はそれぞれ次のようにかけます。

$$
\begin{aligned}
  a & = (a_1, a_2) \\
  x & = \frac{q + 1}{2}(a_1, a_2) \\
  x^2 & = (q + 1)(a_1, a_2) = ((q + 1)a_1, (q + 1)a_2) \\
  & = (a_1, (q + 1)a_2) \\
  a^{-1}x^2 & = (0, qa_2)
\end{aligned}
$$

(本当は同型関数 $\phi: (\mathbb{Z}/p\mathbb{Z})^\times \to \mathbb{Z}/q\mathbb{Z} \times \mathbb{Z}/2^Q\mathbb{Z}$ を用いて $\phi(a) = (a_1, a_2)$ と書きますが、分かりやすさの為に省略します。)

これより $\mathbb{Z}/q\mathbb{Z}$ について解が合いました。

次に $\mathbb{Z}/2^Q\mathbb{Z}$ を合わせます。

まず誤差を $e = a^{-1}x^2$ とおき、$e, e^{2}, e^{4}, e^{8}, e^{16}, \ldots, e^{2^Q}$ を計算していくといつかは $1$ となります。ちょうどそのとき $\mathbb{Z}/2^Q\mathbb{Z}$ が $0$ となります。

$$
\begin{aligned}
  e & = (0, e_2) \\
  e^{2^s} & = (0, 2^se_2)
\end{aligned}
$$

このときの $s$ を用いると誤差 $e$ について $1$ となっている一番下のビットは $Q - s$ ビット目とわかります。

ここで平方剰余でない数 $u$ を取ると $u, u^{2^tq}$ はそれぞれ次のようになります。ただし $u$ は非平方剰余だから $qu_2$ は奇数です。

$$
\begin{aligned}
  u & = (u_1, u_2) \\
  u^{2^tq} & = (0, 2^tqu_2) \\
\end{aligned}
$$

これより $1$ となっている一番下のビットが $t$ の数を作ることができます。$1$ となっている一番下のビットが $Q-s-1$ の数を作って $x$ に掛けます。

$$
x \leftarrow xu^{2^{Q-s-1}q}
$$

すると

$$
\begin{aligned}
  e & = (0, 2^{Q-s}qu_2e_2) \\
  e^{2^s} & = (0, 2^Qqu_2e_2) = (0, 0)
\end{aligned}
$$

となり、$e$ の $1$ となっている一番下のビットが大きくなっています。これを繰り返すことで誤差が次第に無くなっていき、最後には $x$ が $a$ の平方根となります。 $\Box$

最終的には次のアルゴリズムとなります。

1. $x = a^{(q + 1)/2}$ を計算する。
2. 誤差 $e = a^{-1}x^2$ で $1$ となっている一番下のビット $s$ を調べる。
3. 非平方剰余 $u$ を用いて $x \leftarrow x\cdot \mathrm{pow}(u^q, 2^{Q - s - 1})$ とする。
4. $e = 1$ となるまで 2, 3 を繰り返す。

たぶんこの説明だと分かりづらいと思うので具体例を考えます。

$p = 13$ のとき $\mathbb{Z}/12\mathbb{Z} = \mathbb{Z}/3\mathbb{Z}\times\mathbb{Z}/2^2\mathbb{Z}$ となるので次のような表となる。
|  $2^Q\backslash q$  | 0   | 1   | 2   |
| --- | --- | --- | --- |
| 0   | 0   | 4   | 8   |
| 1   | 9   | 1   | 5   |
| 2   | 6   | 10  | 2   |
| 3   | 3   | 7   | 11  |

ここで原始根 $2$ を選ぶと $(\mathbb{Z}/13\mathbb{Z})^\times$ と $\mathbb{Z}/3\mathbb{Z}\times\mathbb{Z}/2^2\mathbb{Z}$ の対応表が完成する。

| $2^Q\backslash q$ | 0         | 1           | 2          |
| ----------------- | --------- | ----------- | ---------- |
| 0                 | $2^0=1$   | $2^4=3$     | $2^8 = 9$  |
| 1                 | $2^9=5$   | $2^1=2$     | $2^5=6$    |
| 2                 | $2^6=12$  | $2^{10}=10$ | $2^2=4$    |
| 3                 | $2^3 = 8$ | $2^7=11$    | $2^{11}=7$ |

[Tonelli-Shanks のアルゴリズム - 37zigenのHP](https://37zigen.com/tonelli-shanks-algorithm/)

## 素数生成

暗号として機能する素数の大きさは $2^{512}$ や $2^{1024}$ 程度のオーダーとなっています。素数定理より、ある数 $n$ が素数である確率は約 $1/\log n$ です。例えば $n=2^{512}$ で2.8%、 $n=2^{1024}$ で1.4%となります。つまり、500回乱数を生成すれば99.65%で素数を見つけられるということです。
素数判定のアルゴリズムは多くありますが、ここではMiller-Rabin素数判定法を紹介します。

> **Miller-Rabin 素数判定法**
> 与えられた数 $n$ が素数かどうかを計算時間 $O(k\log^3 n)$ で誤り率 $4^{-k}$ 以下で判定する確率的素数判定アルゴリズムです。

$n$ が素数のとき、$n-1$ はそれを $2$ で割れるだけ割った数を $d$ として $n-1 = 2^sd$ と書けます。フェルマーの小定理より $n$ と互いに素な数 $a$ を用いて

$$
a^{n-1} - 1 = a^{2^sd} - 1 = (a^d-1)(a^d+1)(a^{2d}+1)(a^{4d}+1)\cdots(a^{2^{s-1}d}+1) = 0
$$

これより次の2式のどちらかが成り立ちます。

$$
\begin{cases}
a^d = 1 & \pmod n \\
a^{2^rd} = -1 & \pmod n \qquad (\exists r \in \mathbb{Z}, 0\leq r\leq s-1)
\end{cases}
$$

この対偶をとると、「ある $a$ をとってきて次の2式をどちらも満たすとき

$$
\begin{cases}
a^d \neq 1 & \pmod n\\
a^{2^rd} \neq -1 & \pmod n \qquad (\forall r \in \mathbb{Z}, 0\leq r\leq s-1)
\end{cases}
$$

$n$ は合成数である」と言えます。

これを用い、次のステップを実行することで確率的な素数判定ができます。
1. $1\leq a \leq n-1$ で $n$ と互いに素な $a$ をランダムにとってくる。
2. 上の条件を満たしたらcompositeと返す。
3. 満たさなければprobably primeと返す。

これを繰り返すことで判定の精度が高まります。この処理をMiller–Rabin素数判定法といって、実行時間は $O(k\log^3 n)$ 、FFTベースの乗算で $Õ(k\log^2 n)$ となります。

具体例を考えてみましょう。
判定時にprobably primeを返す時 p、compositeを返す時 c として具体値を入れると次のようになります。
$n = 25$ (合成数)のとき
$n-1 = 24 = 2^3 \times 3$ より $s = 3, d = 3$

| a | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 | 17 | 18 | 19 | 20 | 21 | 22 | 23 | 24 |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| $a^3 \bmod 25$ | 1 | 8 | 2 | 14 | 0 | 16 | 18 | 12 | 4 | 0 | 6 | 3 | 22 | 19 | 0 | 21 | 13 | 7 | 9 | 0 | 11 | 23 | 17 | 24 |
| $a^6 \bmod 25$ | 1 | 14 | 4 | 21 | 0 | 6 | 24 | 19 | 16 | 0 | 11 | 9 | 9 | 11 | 0 | 16 | 19 | 24 | 6 | 0 | 21 | 4 | 14 | 1 |
| $a^{12} \bmod 25$ | 1 | 21 | 16 | 16 | 0 | 11 | 1 | 11 | 6 | 0 | 21 | 6 | 6 | 21 | 0 | 6 | 11 | 1 | 11 | 0 | 16 | 16 | 21 | 1 |
| 判定 | p | c | c | c | c | c | p | c | c | c | c | c | c | c | c | c | c | p | c | c | c | c | c | p |

$n = 17$ (素数)のとき
$n-1 = 16 = 2^4 \times 1$ より $s = 4, d = 1$

| a | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| $a \bmod 17$ | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 |
| $a^2 \bmod 17$ | 1 | 4 | 9 | 16 | 8 | 2 | 15 | 13 | 13 | 15 | 2 | 8 | 16 | 9 | 4 | 1 |
| $a^4 \bmod 17$ | 1 | 16 | 13 | 1 | 13 | 4 | 4 | 16 | 16 | 4 | 4 | 13 | 1 | 13 | 16 | 1 |
| $a^8 \bmod 17$ | 1 | 1 | 16 | 1 | 16 | 16 | 16 | 1 | 1 | 16 | 16 | 16 | 1 | 16 | 1 | 1 |
| 判定 | p | p | p | p | p | p | p | p | p | p | p | p | p | p | p | p |

かなり正確に判定できていることがわかるでしょう。

素数については必ず成功し、合成数のときは誤る可能性が $1/4$ 以下ということが示せるので、 $k$ 回試行すれば誤り率は $4^{-k}$ 以下となります。つまり、ある値に対して10回素数判定法を回せば99.9999046%成功するということです。

## 環論の基礎

- イデアル
- 加群

加群
ベクトル空間

### イデアル
素イデアル
極大イデアル

### 多項式環
$A$

## 有限体
$\mathbb{F}_p^n \neq \mathbb{F}_{p^n}$ です。



## 多項式の解
さて多変数連立 $n$ 次方程式の場合はどうでしょう。Coppersmith Method も使うこともできますが、私の体感的には精度が出にくいです。これに対して使われる道具は多項式 GCD, 終結式, Gröbner 基底があります。

### 多項式 GCD

まずは多項式 GCD です。
例えば $x$ について同じ解を持つ次のような方程式を考えてみましょう。

> **Half GCD**
> 2 つの多項式の最大公約式を $\mathcal{O}(N(\log{N})^2)$ で求められる。(N is 何)

$a = qb + r$

$$
\begin{pmatrix}
  0 & 1 \\
  1 & -q
\end{pmatrix}
\begin{pmatrix}
  a \\
  b
\end{pmatrix}
= \begin{pmatrix}
  b \\
  a - qb
\end{pmatrix}
$$



### 終結式
式増やしてgcd
合成数 mod でも使える
グレブナー基底は mod p のみ

### Gröbner 基底

Gröbner 基底のお気持ちは多項式を基底簡約するものです。

$$
T = \lbrace x_1^{e_1}x_2^{e_2}\cdots x_l^{e_l}\mid e_1,e_2,\cdots,e_l\in\mathbb{Z}_{\geq 0}\rbrace
$$

- $\mathrm{lt}(f)$: 先頭項 (leading term)
- $\mathrm{lc}(f)$: 先頭係数 (leading coefficient)
- $\mathrm{lm}(f)$: 先頭単項式 (leading monomial)

$f = 5x_1^2x_3 - 2x_1x_3 + 3x_2x_3 \in \mathbb{Q}[x_1, x_2, x_3]$ のとき $\mathrm{lt}(f) = x_1^2x_3$, $\mathrm{lc}(f) = 5$, $\mathrm{lm}(f) = 5x_1^2x_3$ となる。


Buchberger の業績を Gröbner 教授が奪って発表した

> **Buchberger's Algorithm**
> 1. $G = F$
> 2.
> 3.

表に解き方をまとめるとこんな感じです。

|            |           1変数          |               多変数              |
|:----------:|:------------------------:|:---------------------------------:|
| 1次方程式 | 拡張ユークリッドの互除法 |                LLL                |
|  n次方程式 |    Coppersmith Method    | 多項式GCD, 終結式, Gröbner基底 |

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

## 安全性

理想的に安全な暗号というのは無限の計算能力をもってしても破れないような暗号です。これを情報理論的に安全な暗号といいます。

エントロピーとは乱雑さ

> **Def. 情報理論的安全性**
> 任意の入力に対して暗号文の長さと暗号文のエントロピーが一致するとき情報理論的に安全な暗号であるという。

ただ RSA 暗号などの公開鍵暗号は攻撃者に無限の計算能力があれば総当りで公開鍵から秘密鍵を計算することができてしまいます。なので情報理論的に安全ではありません。しかし攻撃者はそこまでの計算能力はないので事実上不可能です。これを性質よく定義するにはどうしたらいいでしょうか。

情報量と計算量という概念を使うことでよりよい安全性を定義できます。
これは暗号を解くには情報理論的に計算効率が最悪な時間、つまり指数時間 $\mathcal{O}(2^N)$ が必要であるということを指しています。

現代暗号の標準モデルでは限られた計算能力を持つ攻撃者 (確率的多項式時間チューリングマシン) を仮定します。

> **Def. 確率的多項式時間チューリングマシン**
> 計算量が多項式で表される確率的なアルゴリズムを確率的多項式時間アルゴリズム (PPT; Probabilistic Polynomial Time) といい、

そして限られた計算能力しか持たない攻撃者にとって疑似乱数と本物の乱数を区別することができないなら、その疑似乱数は安全であるとします。これをより厳密に言うと次のようになります。

> **Def. 識別不可能性 (Indistinguishability)**
> 与えられた 2 つの確率分布 $A$ と $B$ がどのような確率的多項式時間アルゴリズムの識別器でも十分に近ければ (確率の対数の差が極限で 0 となるならば) 計算量的識別不可能であるという。

> **Def. 計算量的エントロピー**
> ある確率分布 $G$ の計算量的エントロピーを $G$ と識別不可能な確率分布のエントロピーとする。

疑似乱数性を満たすなら安全な疑似乱数であるとすればよさそうです。

> **Def. 疑似乱数性**
> ある確率分布 $G$ と一様分布が計算量的識別不可能なら $G$ は疑似乱数性を持つという。

> **Def. 計算量的安全性**
> 任意の入力に対して暗号文の長さと暗号文の計算量的エントロピーが一致するとき計算量的に安全な暗号であるという。

解決がとても難しい $P = NP$ 問題が
多くの計算量理論の研究者は $P \neq NP$ と予想している

$P = NP$ のときすべての暗号が多項式時間で解けるようになってしまう世界線を考えるのが話題になりますが、そうだとしても多項式の次数がめちゃくちゃ大きくなるだろうという予想をされています。今の暗号理論は指数時間と多項式時間で解けるかどうかが安全性に直結するので定義や理論については大幅に見直しが必要になりますが、安全性を再定義出来てしまえばそこまで脅威でもないのかなと思います。
### 安全性
このように疑似乱数や公開鍵暗号などは無限の計算能力を持つ攻撃者の前では SMT を用いて内部情報を取り出したり、公開鍵から秘密鍵を求めたりと攻撃できてしまうので、現代暗号の標準モデルでは限られた計算能力を持つ攻撃者 (確率的多項式時間チューリングマシン) を仮定します。

そして限られた計算能力しか持たない攻撃者にとって疑似乱数と本物の乱数を区別することができないなら、その疑似乱数は安全であるとします。これをより厳密に言うと次のようになります。

> **識別不可能性 (Indistinguishability)**
> 与えられた 2 つの確率分布 $A$ と $B$ がどのような確率的多項式時間アルゴリズムの識別器でも十分に近ければ (確率の対数の差が極限で 0 となるならば) 計算量的識別不可能であるという。

> **疑似乱数性**
> ある確率分布 $G$ と一様分布が計算量的識別不可能なら $G$ は疑似乱数性を持つという。

疑似乱数性を満たすなら安全な疑似乱数であるとすればよさそうです。

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

## まとめ

これで剰余上の演算は一通りできるようになりました。さてコンピューターはこれらをどのようにして計算するのでしょうか。代表的なアルゴリズムで組んだ場合だと以下の表のようになります。(簡単の為、基本的な演算の計算量はビット数に依らないとする)

| 演算                   | 方法                          | 計算量                      |
| :--------------------  | :---------------------------- | :-------------------------- |
| 足し算   $a + b$       | 足して $N$ 以上になったら $N$ 引く | $O(1)$            |
| 引き算   $a - b$       | 引いて $0$ 未満になったら $N$ 足す | $O(1)$            |
| 掛け算   $a \times b$  | 掛けて $N$ で割った余り           | $O(1)$            |
| 割り算   $a \div b$    | 拡張ユークリッドの互除法      | $O(\log^2 N)$   |
| 累乗     $a ^ e$       | 繰り返し二乗法                | $O(\log N)$       |
| 平方根   $\sqrt{a}$    | Tonelli Shanksのアルゴリズム  | $O(\log^2 N)$   |
| 累乗根   $\sqrt[e]{a}$ | Tonelli Shanksのアルゴリズム     | $O(\min(N^{1/4},\sqrt{e})\log{e}\log^2{N})$     |
| 対数     $\log_e{a}$   | 離散対数問題            | $O(\sqrt{N})$     |

累乗は多項式時間しか掛かりませんが、累乗根や対数は指数時間掛かるということを頭の隅に置いておいてください。

### 数体ふるい法
Schirokauer のアルゴリズム
$L_q[1/3, (64/9)^{1/3}]$

$$
\begin{aligned}
\phi(\delta^{q - 1}) = ut^x\phi(\gamma^{q - 1}) \\
x = -\log_t u \pmod{q - 1}
\end{aligned}
$$



## 参考文献
- [Finding a Small Root of a Univariate Modular Equation](https://static.aminer.org/pdf/PDF/000/192/854/finding_a_small_root_of_a_univariate_modular_equation.pdf)
- [katagaitai workshop 2018 winter](http://elliptic-shiho.github.io/slide/katagaitai_winter_2018.pdf)
- [Factoring Integers with Elliptic Curves - HW Lenstra, Jr.](https://wstein.org/edu/Fall2001/124/lenstra/lenstra.pdf)
- Polynomial-Time Algorithms for Prime Factorization and Discrete Logarithms on a Quantum Computer https://arxiv.org/abs/quant-ph/9508027
- The Arithmetic of Elliptic Curves
- Introduction to Cryptography Buchmann
- クラウドを支えるこれからの暗号技術
- [整数論テクニック集のpdf](https://kirika-comp.hatenablog.com/entry/2018/03/12/210446)
- [General purpose integer factoring](https://eprint.iacr.org/2017/1087)
- [katagaitai workshop #7 crypto ナップサック暗号と低密度攻撃](https://www.slideshare.net/trmr105/katagaitai-workshop-7-crypto)

この資料は CC0 ライセンスです。