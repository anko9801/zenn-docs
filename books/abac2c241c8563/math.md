---
title: "暗号技術を支える計算機代数"
---

ここが一番大きな関門です。ここを乗り切ればあとは楽です。

暗号では楕円曲線上の点や格子上の点などを考えたりしますが、それらには足し算や引き算ができたりして、数ではないが「数っぽいもの」がちらほら出てきます。それぞれの数っぽいものを研究してみると似たような性質がちらほらでてきます。これらに共通する性質を研究するのが代数学のモチベーションです。

その基礎となるものを次の節で学んでいきましょう。基礎とはいいつつ、その分野の大事なアイデアや考え方が詰まっているので

- 群論
- 格子
- 多項式
- 楕円曲線

について解説します。

資料の厳密さはその情報の信頼度を表していると思っています。しかし全てを厳密に書いてしまうと文章が記事では収まらなくなってしまいますので非本質的な部分は曖昧にしたり省きます。何が厳密で何が厳密ではないかは明確に書くつもりなのでそこは安心してください。厳密な証明や実装が気になった人は参考文献の本や規格書、元論文などを読んでみると良いと思います。

(かなり内容を絞って簡潔に説明したつもりなんですがこんな長さになってしまった、これ普通の内容にしたら本何冊書けるんだ...)

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

## 格子の基礎

図でイメージ掴むのが速いので図がほしい！冗長になりがちなので厳密性を落とします。

> **Def. 格子**
>  $n$ 個の線形独立なベクトル $\boldsymbol{b}_1,\ldots,\boldsymbol{b}_n\in\mathbb{R}^m$ について整数係数の線形結合によって生成されるベクトルの集合を格子 $\mathcal{L}$ と定義します。
>
> $$
\mathcal{L}(\boldsymbol{b}_1,\ldots, \boldsymbol{b}_n) := \left\{ \sum_{i=1}^{n} a_i\boldsymbol{b}_i\in\mathbb{R}^m\ \middle|\ a_i \in \mathbb{Z} \right\}
$$


$$
\boldsymbol{B} = \begin{pmatrix}
  \boldsymbol{b}_1 \\
  \vdots \\
  \boldsymbol{b}_n \\
\end{pmatrix} = \begin{pmatrix}
  b_{11} & \cdots & b_{1m} \\
  \vdots & \ddots & \vdots \\
  b_{n1} & \cdots & b_{nm} \\
\end{pmatrix}
$$

> **Prop.**
> 格子基底の基本変形に対し、格子は不変である。

化学の格子っぽいもの

### SVP; Shortest Vector Problem

一般の次元の格子上の非零なベクトルの中で最もノルムが小さなベクトルを見つけ出す問題です。
そのベクトルを $\boldsymbol{v}$ とおくと次のように表せられます。

$$
\boldsymbol{v} = v_1\boldsymbol{b}_1 + \ldots + v_n\boldsymbol{b}_n \qquad (v_1, \ldots , v_n \in \mathbb{Z}) \\
$$

この問題はNP困難

> **Def. 逐次最小**
> $n$ 次元格子 $L$ に対して、一次独立な格子ベクトル $\boldsymbol{b}_1,\ldots,\boldsymbol{b}_i\in L$ を用いて各 $1\leq i\leq n$ における逐次最小を次のように定義する。
>
> $$
\lambda_i(L) := \min_{\boldsymbol{b}_1,\ldots,\boldsymbol{b}_i\in L}\max\lbrace\|\boldsymbol{b}_1\|,\ldots,\|\boldsymbol{b}_i\|\rbrace
$$
>
> 特に任意の $1\leq i\leq n$ について $\|\boldsymbol{b}_i\| = \lambda_i(L)$ を満たすとき逐次最小ベクトルと呼び、それらが基底となっているとき逐次最小基底と呼ぶ。

### Gram-Schmidt の直交化 (GSO; Gram-Schmidt Orthonormalization)

Gram-Schmidt 直交化 (GSO; Gram-Schmidt Orthonormalization) とはベクトル空間 $\mathbb{R}^m$ の基底を直交基底に変換する方法です。 $\boldsymbol{b}_n$ の直交化は $\boldsymbol{b}_{1},\ldots, \boldsymbol{b}_{n-1}$ すべてと直交するように元の高さのまま移動させます。 GSO の Wikipedia の gif がわかりやすいです。

> **Def. GSO ベクトル**
> $n$ 次元格子 $L\subseteq \mathbb{R}^m$ の順序付き基底 $\{\boldsymbol{b}_{1},\ldots, \boldsymbol{b}_{n}\}$ に対する GSO ベクトル $\boldsymbol{b}_{1}^* ,\ldots, \boldsymbol{b}_{n}^ *\in\mathbb{R}^m$ を GSO 係数 $\mu_{i,j}$ を用いて次のように定義する。
>
> $$
\begin{aligned}
&\begin{dcases}
\boldsymbol{b}_1^* := \boldsymbol{b}_1 \\
\boldsymbol{b}_i^* := \boldsymbol{b}_i - \sum_{j=1}^{i-1} \mu_{ij} \boldsymbol{b}_j^* & (2\leq i\leq n) \\
\end{dcases} \\
&\quad \mu_{ij} := \frac{\langle \boldsymbol{b}_i, \boldsymbol{b}_j^* \rangle}{\| \boldsymbol{b}_j^*\|^2} \qquad (1\leq j<i\leq n)
\end{aligned}
$$

行列で書くと次のようになる。

$$
\begin{aligned}
\begin{pmatrix}
\boldsymbol{b}_1 \\
\vdots \\
\boldsymbol{b}_n \\
\end{pmatrix}
& =
\begin{pmatrix}
1 & 0 & 0 & & 0 \\
\mu_{21} & 1 & 0 & \cdots & 0 \\
\mu_{31} & \mu_{32} & 1 & & 0 \\
& \vdots & & \ddots & \vdots \\
\mu_{n1} & \mu_{n2} & \mu_{n3} & \cdots & 1 \\
\end{pmatrix}
\begin{pmatrix}
\boldsymbol{b}_1^ * \\
\vdots \\
\boldsymbol{b}_n^ * \\
\end{pmatrix} \\
\\
\boldsymbol{B} & = \boldsymbol{U}\boldsymbol{B}^*
\end{aligned}
$$

この $\boldsymbol{B}$、$\boldsymbol{B}^*$、$\boldsymbol{U}$ をそれぞれ **基底行列**、**GSO ベクトル行列**、**GSO 係数行列** と呼ぶことにします。また GSO 係数について

$$
\mu_{ij} = \begin{dcases}
  0 & (1\leq i<j\leq n)\\
  1 & (1\leq i=j\leq n) \\
  \frac{\langle \boldsymbol{b} _ i, \boldsymbol{b}_j^ * \rangle}{\| \boldsymbol{b}_j^*\|^2} & (1\leq j<i\leq n)
\end{dcases}
$$

と定義を拡大して $\boldsymbol{U} = (\mu_{ij})$

> **Prop. GSO ベクトルの基本性質**
> 1. 任意の $1\leq i<j\leq n$ に対して $\langle\boldsymbol{b}_i^*, \boldsymbol{b}_j^*\rangle = 0$ が成り立つ。
> 2. 任意の $1\leq i\leq n$ に対して $\|\boldsymbol{b}_i^*\|\leq\|\boldsymbol{b}_i\|$ が成り立つ。
> 3. 任意の $1\leq i\leq n$ に対して $\langle\boldsymbol{b}_1^* ,\ldots,\boldsymbol{b}_i^*\rangle_{\mathbb{R}} = \langle\boldsymbol{b}_1,\ldots,\boldsymbol{b}_i\rangle_{\mathbb{R}}$ が成り立つ。
> 4. $\mathrm{vol}(L) = \prod_{i=1}^n\|\boldsymbol{b}_i^*\|$ が成り立つ。

**Proof.**
まず 1 について $j = 1$ のとき証明せずとも成り立つ。$1\leq j\leq k$ のとき $\langle\boldsymbol{b}_i^*, \boldsymbol{b}_j^*\rangle = 0$ が成り立つと仮定して、$j = k+1$ のとき

$$
\begin{aligned}
  \langle\boldsymbol{b}_i^*,\boldsymbol{b}_{k+1}^*\rangle &
  = \left\langle\boldsymbol{b}_i^*,\boldsymbol{b}_{k+1} - \sum_{n=1}^k\mu_{k+1 n}\boldsymbol{b}_{n}^*\right\rangle \\
  & = \langle\boldsymbol{b}_i^*,\boldsymbol{b}_{k+1}\rangle-\mu_{
  k+1 i}\|\boldsymbol{b}_i^*\|^2\\
  & = 0
\end{aligned}
$$

が成り立つ。よって数学的帰納法より成り立つ。
2 に関しては定義式にノルムを取ることで分かる。

$$
\begin{aligned}
  \|\boldsymbol{b} _ 1^*\|^2 & = \|\boldsymbol{b}_1\|^2 \\
  \|\boldsymbol{b}_ i\|^2 & = \|\boldsymbol{b} _ i^ * \|^2 + \sum_{j=1}^{i-1}\mu _ {i,j}^2\|\boldsymbol{b}_j^ * \|^2\geq\|\boldsymbol{b} _ i^ * \|^2
\end{aligned}
$$

3 についてはまず $\langle\boldsymbol{b}_1,\ldots,\boldsymbol{b}_i\rangle_{\mathbb{R}}\subseteq\langle\boldsymbol{b}_1^*,\ldots,\boldsymbol{b}_i^*\rangle_{\mathbb{R}}$ について $i = 1$ は成り立ち、$1\leq i\leq k$ について成り立つとして以下より数学的帰納法から成り立つ。

$$
\boldsymbol{b}_k = \boldsymbol{b}_k^* + \sum_{j=1}^{k-1} \mu_{kj}\boldsymbol{b}_j^* \in\langle\boldsymbol{b}_1^*,\ldots,\boldsymbol{b}_i^*\rangle_{\mathbb{R}}
$$

同様に $\langle\boldsymbol{b}_1,\ldots,\boldsymbol{b}_i\rangle_{\mathbb{R}}\supseteq\langle\boldsymbol{b}_1^*,\ldots,\boldsymbol{b}_i^*\rangle_{\mathbb{R}}$ も数学的帰納法より成り立つ。

$$
\boldsymbol{b}_k^* = \boldsymbol{b}_k - \sum_{j=1}^{k-1} \mu_{kj}\boldsymbol{b}_j^* \in\langle\boldsymbol{b}_1,\ldots,\boldsymbol{b}_i\rangle_{\mathbb{R}}
$$

よって $\langle\boldsymbol{b}_1^*,\ldots,\boldsymbol{b}_i^*\rangle_{\mathbb{R}}=\langle\boldsymbol{b}_1,\ldots,\boldsymbol{b}_i\rangle_{\mathbb{R}}$ となる。

4 については $\boldsymbol{B}=\boldsymbol{U}\boldsymbol{B}^*$ と $\det(\boldsymbol{U}) = 1$、GSO ベクトルの直交性より

$$
\begin{aligned}
\mathrm{vol}(\mathcal{L})^2 &= \det(\boldsymbol{B}\boldsymbol{B}^\top) \\
& = \det(\boldsymbol{U}\boldsymbol{B}^*(\boldsymbol{B}^*)^\top \boldsymbol{U}^\top) \\
& = \det(\boldsymbol{B}^*(\boldsymbol{B}^*)^\top) \\
& = \prod_{i=1}^n\|\boldsymbol{b}_i^*\|^2
\end{aligned}
$$

$\Box$

GSO ベクトルの基本性質 2, 4 より次のことが分かる。
> **Thm. Hadamardの不等式**
>
> $$
\mathrm{vol}(L)\leq\prod_{i=1}^n\|\boldsymbol{b}_i\|
$$
>
> 特に $\{\boldsymbol{b}_{1},\ldots, \boldsymbol{b}_{n}\}$ が直交基底$\iff\mathrm{vol}(L)=\prod_{i=1}^n\|\boldsymbol{b}_i\|$ である。

> **Def. 射影格子**
> $n$ 次元格子 $L\subseteq\mathbb{R}^m$ の基底 $\lbrace\boldsymbol{b}_1,\ldots, \boldsymbol{b}_n\rbrace$ に対し, 各 $1\leq l\leq n$ に対して $\langle\boldsymbol{b}_1,\ldots, \boldsymbol{b}_{l-1}\rangle_\mathbb{R}$ の直交補空間への直交射影を $\pi_l:\mathbb{R}^m\to\langle\boldsymbol{b}_1,\ldots, \boldsymbol{b}_{l-1}\rangle_\mathbb{R}^\bot$ とする。 定理 2 の 1,3 より
>
> $$
\begin{aligned}
\langle\boldsymbol{b}_1, \ldots, \boldsymbol{b}_{l-1}\rangle_\mathbb{R}^\bot &= \langle\boldsymbol{b}_1^*, \ldots, \boldsymbol{b}_{l-1}^* \rangle_\mathbb{R}^\bot = \langle\boldsymbol{b}_l^*, \ldots, \boldsymbol{b}_n^* \rangle_\mathbb{R} \\
\pi_l(\boldsymbol{b}_i) &= \sum_{j=l}^i \mu_{i,j}\boldsymbol{b}_j^* \\
\end{aligned}
$$
>
> となる。 すると集合 $\pi_l(L)$ は $\lbrace\pi_l(\boldsymbol{b}_l), \ldots, \pi_l(\boldsymbol{b}_n)\rbrace$ を基底に持つ $n-l+1$ 次元の格子であり, $\pi_l(L)$ を射影格子 (projected lattice) と呼ぶ。

### 最短ベクトルの数え上げ

まずは全探索してみます。
考えてみると帰納的に求めるのでは正確な最短ベクトルは求められないでしょう。
考えてみるとある基底 $\boldsymbol{b}_i$ に対し、それ以下の基底 $\boldsymbol{b}_1, \ldots \boldsymbol{b}_{i-1}$ で組み立てられたベクトル $\boldsymbol{v}$ に対し、$\boldsymbol{b}_i$ を用いて短くする

効率的に数え上げる為には基底簡約すると良いということが知られています。

> **Lagrange 基底簡約 (Gaussian Reduction)**
> 2 次元格子の厳密解については古くから知られている。ユークリッドの互除法を用いることで最も簡約された基底を得られる。
> $\|\boldsymbol{v}_1\| < \|\boldsymbol{v}_2\|$ となるように交換して
>
> $$
\begin{aligned}
  m & = \left\lfloor\frac{\boldsymbol{v}_1\cdot \boldsymbol{v}_2}{\|\boldsymbol{v}_1\|^2}\right\rceil \\
  \boldsymbol{v}_2 & \leftarrow \boldsymbol{v}_2 - m\boldsymbol{v}_1
\end{aligned}
$$
>
> を繰り返し $m = 0$ となるとき $\boldsymbol{v}_1$, $\boldsymbol{v}_2$ は最も簡約された基底となる。

最も簡約化されているかを証明する。

```python
def gaussian_reduction(v1, v2):
    while True:
        if v2.norm() < v1.norm():
            v1, v2 = v2, v1
        m = floor(v1.dot_product(v2) / v1.dot_product(v1))
        if m == 0:
            break
        v2 = v2 - m*v1
    return v1, v2


v1 = vector([846835985, 9834798552])
v2 = vector([87502093, 123094980])
```



:::message
**練習問題**
CryptoHack
:::

> **サイズ基底簡約**
> $n$ 次元格子 $L$ の基底 $\\{\boldsymbol{b_1},\ldots,\boldsymbol{b_n}\\}$ を GSO 係数 $\mu_{i,j}$ が
>
> $$
|\mu_{i,j}| \leq \frac{1}{2} \quad (1 \leq \forall j < \forall i \leq n)
$$
>
> を満たすとき、基底 $\\{\boldsymbol{b_1},\ldots,\boldsymbol{b_n}\\}$ はサイズ簡約されているという。
> GSO ベクトルを簡約 -> 基底ベクトルを簡約
> 1. $q = \lfloor\mu_{ij}\rceil$ として $\boldsymbol{b}_i\leftarrow\boldsymbol{b}_i - q\boldsymbol{b}_j$ と更新する。
> 2. GSO 係数について $\mu_{il}\leftarrow \mu_{il} - q\mu_{jl}$ と更新する。

```python
def size_reduction(B):
    n = B.nrows()
    _, mu = B.gram_schmidt()
    for i in range(n):
        for j in range(i - 1, -1, -1):
            if mu[i][j].abs() > 1 / 2:
                q = mu[i][j].round()
                B[i] -= q * B[j]
                mu[i] -= q * mu[j]
    return B

B = matrix([[5, -3, -7], [2, -7, -7], [3, -10, 0]])
print(size_reduction(B))
```

$$
\begin{pmatrix}
5 & -3 & -7 \\
2 & -7 & -7 \\
3 & -10 & 0
\end{pmatrix}\to
\begin{pmatrix}
 5 & -3 & -7 \\
-3 & -4 & 0 \\
 1 & -3 & 7
\end{pmatrix}
$$

> **LLL (Lenstra-Lenstra-Lovasz) 基底簡約**
> Lovasz 条件を $1/4 < \delta < 1$ としたときに任意の $2\leq k\leq n$ に対して次を満たすこととする。
>
> $$
\delta \|\boldsymbol{b}_{k-1}^*\|^2 \leq \|\pi_{k-1}(\boldsymbol{b}_k)\|^2
$$
>
> このとき次の 2 ステップを行えるまで繰り返す。
> 1. サイズ基底簡約
> 2. Lovasz 条件に合うように基底ベクトルの交換

```python
def LLL(B, delta=0.99):
    assert 1 / 4 < delta < 1
    n = B.nrows()
    b = [0 for _ in range(n)]
    BB, mu = B.gram_schmidt()

    i = 1
    while i < n:
        # size reduction
        for j in range(i - 1, -1, -1):
            if mu[i][j].abs() > 1 / 2:
                q = mu[i][j].round()
                B[i] -= q * B[j]
                mu[i] -= q * mu[j]

        b[i - 1] = BB[i - 1].dot_product(BB[i - 1])
        b[i] = BB[i].dot_product(BB[i])

        # Lovasz condition
        if b[i] >= (delta - mu[i][i - 1] * mu[i][i - 1]) * b[i - 1]:
            i += 1
        else:
            B.swap_rows(i - 1, i)
            BB, mu = B.gram_schmidt()
            i = max(i - 1, 1)
    return B
```

実際は LLL 簡約されていることを定義して、簡約されたときの上限などを示し、LLL 簡約された基底を返すアルゴリズムの well-defined 性や高速化を考えますが、前提知識などが不足していたり長くなるのでアルゴリズムとその特徴を天下り的に書いて終わらせます。

### ナップサック暗号 (Markle-Hellman Knapsack encryption)
一般の数列 $b_i$ に対して $c = \sum_i m_ib_i$ となる $c$ から $m_i$ を求めるのは難しいが $b_i$ が超増加列という性質を持つとき簡単になることを利用した暗号。

> **Def. 超増加列**
> 次の性質を満たす数列 $\lbrace w_i\rbrace$ を超増加列と呼ぶ。
>
> $$
\sum_{i=1}^n w_i < w_{n+1}
$$

鍵生成
1. 超増加列 $\lbrace w_i\rbrace$ を生成する。
2. 整数 $q, r$ について $q > \sum_i w_i$, $\gcd(r, q) = 1$ なるように生成する。
3. $b_i = rw_i \pmod q$ を計算し、$\lbrace b_i\rbrace, q$ を公開鍵、$\lbrace w_i\rbrace, r$ を秘密鍵とする。

暗号化
1. 平文 $m_i\in\lbrace 0, 1\rbrace$ を用いて次のように暗号文 $c$ を得る。

$$
c = \sum_i m_ib_i \pmod q
$$

復号
1. 暗号文 $c$ に対して秘密鍵 $r$ を用いて $c' = cr^{-1} \bmod q = \sum_i m_iw_i$ を計算する。
2. $i$ を降順に次のアルゴリズムを実行する。

$$
\begin{aligned}
m_i & \leftarrow \begin{cases}
0 & (c' \leq w_i) \\
1 & (c' > w_i)
\end{cases} \\
c' & \leftarrow c' - w_i
\end{aligned}
$$

これが低密度のとき LLL を用いて攻撃する方法があります。LO 法 (Lagarias-Odlyzko Algorithm) と CLOS 法 (Coster LaMacchia Odlyzko Schnorr) と呼ばれています。

LO 法は次の行列を LLL 簡約して左端が $0$, それ以外が $0, 1$ のみである行があれば、それは復号結果の候補となる。

$$
\begin{pmatrix}
\boldsymbol{b} & I \\
-c & \boldsymbol{0}
\end{pmatrix} =
\begin{pmatrix}
b_1 & 1 &&& \\
b_2 && 1 && \\
\vdots &&& \ddots & \\
b_n &&&& 1 \\
-c &&&& \\
\end{pmatrix}
$$

CLOS 法は基底行列をちょっと変更して精度を向上させた方法です。まず $0, 1$ はノルムに非対称性があるので $-1, 1$ に変更し、左端を優先的に $0$ にするよう伝える為適当な大きな定数 $K$ を掛ける。

$$
\begin{pmatrix}
K\boldsymbol{b} & 2I \\
-Kc & -\boldsymbol{1}
\end{pmatrix} =
\begin{pmatrix}
Kb_1 & 2 &&& \\
Kb_2 && 2 && \\
\vdots &&& \ddots & \\
Kb_n &&&& 2 \\
-Kc & -1 & -1 & \cdots & -1 \\
\end{pmatrix}
$$

### Approximate GCD Problem
$r_i \approx 2^\lambda$, $p \approx 2^{\lambda + \log\lambda}$, $q \approx 2^{\lambda\log\lambda}$

$$
x_i = q_ip + r_i
$$

$$
\begin{pmatrix}
2^{\rho+1} & x_1 & x_2 & x_3 & x_4 \\
& -x_0 &&& \\
&& -x_0 && \\
&&& -x_0 & \\
&&&& -x_0  \\
\end{pmatrix}
$$

## 多項式
ここでは剰余上の方程式 $\mathbb{Z}/N\mathbb{Z}[x]$ の解を求める方法について解説します。

まず多変数連立 $1$ 次方程式のときは行列の $\mathrm{rank}$ と変数の数が一致するときガウスの消去法を用いて逆行列を計算できます。

$$
\begin{aligned}
A\boldsymbol{x} &= \boldsymbol{b} & \pmod N \\
\boldsymbol{x} &= A^{-1}\boldsymbol{b} & \pmod N
\end{aligned}
$$

次に与えられる多変数 1 次方程式が変数の数より少ないときについて考えます。まずある式に他の式を代入することで 1 つの式に帰着できます。

$$
a_1x_1 + a_2x_2 + \ldots + a_nx_n = b \pmod N
$$

まずは 2 変数のとき一次不定方程式 $ax + by = c$ は $\gcd(a, b)\mid c$ のとき $(x, y) = (bm + x_0, -am + y_0)$ $m\in\mathbb{Z}$ である。グラフ上で考えると

つまり、方程式の代表解を格子問題に帰着させることができます。

$$
\begin{pmatrix}
  1 & 0 & 0 & a \\
  0 & 1 & 0 & b \\
  0 & 0 & X & c \\
\end{pmatrix}
$$

少ない次数の方程式を LLL で解く方法については良記事があります。

https://qiita.com/kusano_k/items/5509bff6e426e5043591

:::message
**練習問題**
$|x|, |y|\leq\sqrt{n}$

$$
ax = b \pmod n
$$

:::

更に $n$ 次方程式を格子問題に帰着する方法を考えます。

$$
a_nx^n + \ldots + a_1x + a_0 = 0 \pmod N
$$

まず考えるのはそれぞれの次数について $x_i = x^i$ とそれぞれ変数で置いて LLL に通す方法です。精度は落ちますが次数が少ないときには有効です。

これを改良する方法を考えます。まず次の補題を示します。

> **Thm. Howgrave-Graham の補題**
> $N$ を法、 $g(x) \in \mathbb{Z}[x]$ を整数多項式とし、含まれる単項式の数を $\omega$ とする。$g(x)$ に対してある $X$ が存在し、$g(x_0) = 0 \pmod{N}$ なる $x_0 \in \mathbb{Z}$ について $|x_0| \leq X$ であると仮定する。このとき
>
> $$
\|g(xX)\| < \frac{N}{\sqrt{\omega}}
$$
>
> が成立するならば $g(x_0) = 0$ が整数方程式として成立する。ただし
>
> $$
\|g(x)\| = \left\|\sum_{i=0}^{\deg g(x)}g_i\right\| = \sqrt{\sum_{i=0}^{\deg g(x)}g_i^2}
$$
>
> であり、 $\deg g(x)$ は $g(x)$ の次数である。

**Proof.**

$$
\begin{aligned}
|g(x_0)| &= \left|\sum_{i=0}^{\deg g(x_0)}g_ix_0^i\right| \\
&\leq \sum_{i}|g_ix_0^i| \\
&\leq \sum_{i}|g_i|X^i \\
&= \sum_{i}(1\cdot|g_i|X^i) \\
&\leq \sqrt{\sum_{i, g_i \neq 0}1} \sqrt{\sum_{i}(|g_i|X^i)^2} && \left(\because \text{Cauchy–Schwarz の不等式}\right) \\
&= \sqrt{\omega}\|g(xX)\| < N && \left(\because \|g(xX)\| < \frac{N}{\sqrt{\omega}}\right)
\end{aligned}
$$

$g(x_0) = 0 \pmod N$ より $g(x_0) = 0$ となる。 $\Box$

「剰余の方程式は係数がある程度小さければそのまま整数方程式となるよ」という主張をしています。ここで勘のいい人は LLL を用いて係数を小さくすれば整数方程式に変換できて解けるのでは...！？と気付くでしょう。実際に考えてみましょう。

とりあえず状況を整理すると、 LLL に入れる値は各係数として、 LLL を使う為には複数の方程式が必要になってきます。そしてそれらの方程式は同じ解を持つ必要があります。現在、その解が分からないのですが、どうしたらそんな方程式が作れるでしょうか。

実は $\bmod {N}$ では難しいので、$\bmod {N^m}$ に持ち上げることで同じ解の方程式を増やすことができます。

> **Lemma.**
> $N$ を法、$f(x)$ を多項式とする。自然数 $m, l$ について
>
> $$
g_{i,j}(x) := N^{m−i}x^j f^i(x) \ (0 \leq i \leq m, 0 \leq j\leq l)
$$
>
> とおく。このとき、 $f(x_0) = 0 \pmod N$ をみたす $x_0 \in \mathbb{Z}$ について、 $g_{i,j}(x_0) = 0 \pmod{N^m}$ となる。

**Proof.**
$f(x_0) = 0 \pmod N$ なので $f(x_0) = kN$ とおける。

$$
\begin{aligned}
g_{i,j}(x_0) &= N^{m−i}x_0^j f^i(x_0) \\
&= N^{m−i}x_0^j (kN)^i \\
&= k^ix_0^j N^m \\
g_{i,j}(x_0) &= 0 \pmod{N^m} \\
\end{aligned}
$$

$\Box$

これで方程式を増やすことができました！ちゃんと LLL で動くかちょっと不安ですがとりあえずやってみます。

小さくしたい方程式は $g_{i,j}(xX)$ であることに注意して。
$g_{i,j}(x)$ の $k$ 次の係数のことを $g_{i,j}^{(k)}$ と表すことにします。

$$
\begin{pmatrix}
g_{0,0}^{(0)} & g_{0,0}^{(1)}X & g_{0,0}^{(2)}X^2 & \cdots & g_{0,0}^{(n)}X^n \\
&& \vdots \\
g_{0,l}^{(0)} & g_{0,l}^{(1)}X & g_{0,l}^{(2)}X^2 & \cdots & g_{0,l}^{(n)}X^n \\ \\
g_{1,0}^{(0)} & g_{1,0}^{(1)}X & g_{1,0}^{(2)}X^2 & \cdots & g_{1,0}^{(n)}X^n \\
&& \vdots \\
g_{1,l}^{(0)} & g_{1,l}^{(1)}X & g_{1,l}^{(2)}X^2 & \cdots & g_{1,l}^{(n)}X^n \\ \\
&& \vdots \\
g_{m,l}^{(0)} & g_{m,l}^{(1)}X & g_{m,l}^{(2)}X^2 & \cdots & g_{m,l}^{(n)}X^n \\
\end{pmatrix}
$$

これを LLL に通してあげると無事小さな値の方程式が返ってきます。これが Howgrave-Graham の補題を満たしていれば整数方程式となります。後は増減表書いたりして探索すれば解けます。

これらの操作は Coppersmith の定理と呼ばれています。

> **Thm. Coppersmith の定理**
> $N$ を法とし $f(x)$ をモニックな 1変数 $\delta$ 多項式とする。このとき $f(x_0) = 0 \pmod{N}$ と次の条件を満たすような $x_0$ を効率よく求めることができる
>
> $$
|x_0| \leq N^{\frac{1}{\delta}}
$$

さらに Coppersmith の定理には拡張できることが2つあります。

- 未知の法について解ける
  - 既知の法の約数を法とする式の解を求められます。約数の法が小さいほど方程式に対する制約がゆるくなります。
- 多変数の方程式も解ける
  - 変数の数が多いほど方程式に対する制約がキツくなります。

これらは Howgrave-Graham の補題 などを見直すことで簡単に拡張できます。興味ある方は考えてみてください。

これらをまとめて Coppersmith Method と呼びます。

Berlekamp-Zassenhause 法

これを使って様々な攻撃ができます。

解きたい方程式の法の数の下限 $\beta$ と解が存在しうる上限 $X$ を決めて関数を与えると解が返ってきます。

:::message
**練習問題**

:::

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

:::message
**練習問題**

:::

## 楕円曲線の基礎

楕円曲線というのは次の関数からなる曲線のことです。

$$
y^2 = x^3 + ax + b
$$

ここにグラフ

射影平面で定義する理由

- 無限遠点を数学的に定義できる
- 実装時に割り算の遅延ができ高速化できる

楕円曲線上の点 $P, Q$ に対し、直線 $PQ$ と曲線との交点の $y$ 座標を符号反転した点を $P + Q$ とします。

ここに図

この定義を踏まえて、具体的に式を立てて計算すると $P(x_1, y_1), Q(x_2, y_2)$ として $R(x_3, y_3) = P + Q$ は次のように計算できます。ただし、$P = Q$ のときの直線 $PQ$ は曲線に対する点 $P$ の接線と考えます。

$$
\begin{aligned}
x_3 &= \lambda^2 - x_1 - x_2 \\
y_3 &= \lambda(x_1 - x_3) - y_1 \\
\lambda &=
\begin{dcases}
\frac{y_2 - y_1}{x_2 - x_1} \quad (P \neq Q) \\
\frac{3x_1^2 + a}{2y_1} \quad (P = Q)
\end{dcases}
\end{aligned}
$$

ここに具体例

また加法について群となることが証明でき結合法則 $P + (Q + R) = (P + Q) + R$ が成り立つことが証明でき、$nP = \underbrace{P + P + \ldots + P}_n$ と書けます。これを楕円曲線上のスカラー積と呼びます。

ここに具体例

$8P$

ここに実装

### 楕円曲線の位数
> **Schoof のアルゴリズム**
> 楕円曲線 $E/\mathbb{F}_p$ の位数を求められる。

Hasse-Weil 定理より

$$
q+1-2\sqrt{q}\leq|E(\mathbb{F}_q)|\leq q+1+2\sqrt{q}
$$

$|E(\mathbb{F}_q)|=q+1+t$ とおける。$\mathbb{F}_q$ のフロベニウス写像 $\sigma$ のトレース $t$ を計算できれば位数が求まる。しかし $t$ は $2\sqrt{q}$ のオーダーであるため、直接計算できない。そこで素数 $l$ を剰余にとってそれぞれの $t$ の値を求め、中国剰余定理によって $t$ を求める。

具体的には特性多項式の $t$ に $0,\ldots,\frac{l-1}{2}$ の値を代入して確かめる。

$$
\begin{aligned}
\sigma_q^2-t\sigma_q+q &= 0 \\
\pm[t_l]\circ\sigma_q &= \sigma_q^2+[q_l]
\end{aligned}
$$

1つの $l$ 等分点 $P$ に対して成り立てばすべての $l$ 等分点に対して成り立つ。


## 離散対数問題
計算機で解くことの難しい部類 NP 完全の問題です。

> **離散対数問題 (DLP: Discrete Logarithm Problem)**
> 位数 $N$ の巡回群 $G$ について $g, y\in G$ が与えられるので $g^x = y$ となる最小の $x\in \mathbb{N}$ を求める問題。

- 有限体 $\mathbb{F}_p$ の DLP は FFDLP; Finite Field DLP と呼ばれる。巡回群 $\mathbb{F}_p$ の位数は $p-1$ となる。
- 楕円曲線 $E$ 上での DLP は ECDLP; Elliptic Curve DLP と呼ばれる。巡回群 $E/\mathbb{F}_p$ の位数は Hasse の定理より $|\#E/\mathbb{F}_p - (p+1)|\leq 2\sqrt{p}$ に制限される。

### Baby-step Giant-step

半分全列挙を用いる方法。

$m = \lceil\sqrt{N}\rceil$ とおく。DLP の解 $n$ を $m$ で割って $n = qm + r$ とおく。

$$
\begin{aligned}
y & = g^{qm + r} & (q, r\in[0, m-1])
\end{aligned}
$$

このとき $yg^{-r}$, $g^{qm}$ を全列挙し、どちらかのリストの要素をもう1つのリストで検索して $yg^{-r} = g^{qm}$ となる組を探索し、解を得る。この計算量は $O(\sqrt{N}\log N)$ となる。

### Pollard's $\rho$ 法

誕生日のパラドックスを用いる方法。

> **Prop. 誕生日のパラドックス**
> 誕生日が同じ 2 人を見つけたいときに確率 $P$ を超えるには人を何人集めればよいのかという問題です。鳩ノ巣原理から $366$ 人いれば必ず同じ誕生日の人が出てきます。$50\%$ を超えるには $23$ 人で十分です。

若干曖昧ですが期待値の上界の証明です。

**Proof.**
$N$ 種類の元から $k$ 個の元を取ってきたとき $k-1$ 個までそれぞれ相違なり, $k$ 個目で同じとなる確率は $t \ll 1$ のときの近似 $1 - t\approx e^{-t}$ を行うことで次のようになる。

$$
P(A) = \frac{k}{N}\prod_{i = 0}^{k-1}\left(1-\frac{i}{N}\right) \leq \frac{k}{N}\prod_{i = 0}^{k-1}e^{-i/N} = \frac{k}{N}e^{-k(k-1)/2N} \leq \frac{k}{N}e^{-k^2/2N}
$$

試行回数 $k$ に対する期待値は $t = k/\sqrt{N}$ と変数変換し、ガウス積分することで求まる。

$$
E(A) \leq \sum_{k=1}^N k\cdot\frac{k}{N}e^{-k^2/2N} = \sum_{k=1}^N t^2e^{-t^2/2} \leq \sqrt{N}\int_0^\infty t^2e^{-t^2/2}dt = \sqrt{\frac{\pi N}{2}}
$$

よって期待値は大体 $\sqrt{\frac{\pi N}{2}}$ となる為、$N = 365$ を代入すると 50\% を超えるには 23.95 人が必要となる。$\Box$

このように $N$ 種類のボールが入った袋から無作為に取ってきたら同じ種類のボールが 2 つ取れるような個数が $\mathcal{O}(\sqrt{N})$ であることを利用して計算量を落とすことを考えます。まず大枠としては次のようなアルゴリズムです。

1. 疑似乱数関数 $f(a)$ を決めて数列 $a_0, a_{i+1} = f(a_i)$ を生成する。
2. $a_i = a_j$ となる $i, j\ (0\leq i<j<N)$ を発見したとき以下の方法で DLP が求まる。

まず、このアルゴリズムで使われる代表的な疑似乱数関数 $f(x)$ について紹介します。まず巡回群 $G$ を $G_1, G_2, G_3$ に振り分けて、次のように定義します。

$$
f(a)=
\begin{cases}
ya & (a \in G_1) \\
a^2 & (a \in G_2) \\
ga & (a \in G_3)
\end{cases}
$$

このとき $a_0 = g$ とすると $a_i = g^{s_i}y^{t_i} = g^{s_i + xt_i}\ (s_i, t_i \in \mathbb{N})$ と書ける。$a_i = a_j$ のとき

$$
\begin{aligned}
a_ia_j^{-1} & = g^{(s_i + xt_i) - (s_j + xt_j)} = 1 \\
x &= \frac{s_i - s_j}{t_j - t_i} & \pmod N
\end{aligned}
$$

となり $x$ が分かる。期待計算量は $\mathcal{O}(N^{1/4})$ です。

Pollard-$\rho$ 法の $\rho$ は文字 $\rho$ の形が $a_i$ の由来となっています。

### Pollard's Kangaroo 法 ($\lambda$ 法)
$\rho$ 法は動く点が1つの値だったのに対し、 $\lambda$ 法は2つの値がランダムに動いていき、一方がもう一方の点に衝突したとき DLP が解ける。

$$
\begin{aligned}
x_0 & = g^\alpha & y_0 & = y \\
x_{i+1} & = x_ig^{f(x_i)} & y_{i+1} & = y_ia^{f(y_i)} \\
\end{aligned}
$$

$x_i = y_j$ となるとき $x = \alpha + \sum_{k=1}^{i} f(x_k) - \sum_{k=1}^{j} f(y_k)$ となる。
見つからなければ $N$ や $f$ を取り替えて繰り返す。

同じく期待計算量は $\mathcal{O}(N^{1/4})$ です。

### Pohlig-Hellman

> **Prop.**
> 巡回群の位数が $|G| = \prod_{i = 1}^n p_i^{e_{i}}$ と素因数分解できるとき $G \cong \prod_{i = 1}^n \mathbb{Z}/p_i^{e_{i}}\mathbb{Z}$ となる。

アーベルの構造定理により証明できる。詳細は群論を学んでほしい。
これより中国剰余定理から $\mathcal{O}(\max{p_i^{e_i}})$ に落ちる。

### 指数計算法 (Index Calculus Algorithm)

因子基底 $p_j$
1. 小さな素因数 $p_j$ を用いて $yg^k = \prod_{j = 1}^m p_j^{e_{j}} \pmod p$ と書けるような $k$ を見つける。
2. $g^{k_i} = \prod_{j = 1}^m p_j^{e_{ij}} \pmod{p}$ と素因数分解できるような $k_i$ を $n$ 個以上見つける。

すると次のように書ける。

$$
\begin{aligned}
  g^{k_i} & = \prod_{j = 1}^m p_j^{e_{ij}} & \pmod p \\
  k_i & = \sum_{j = 1}^m e_{ij}\log_g{p_j} & \pmod{p-1} \\
\begin{pmatrix}
  k_1 \\
  \vdots \\
  k_n \\
\end{pmatrix} & =
\begin{pmatrix}
  e_{11} & \cdots & e_{m1} \\
  \vdots & \ddots & \vdots \\
  e_{1n} & \cdots & e_{mn}\\
\end{pmatrix}
\begin{pmatrix}
  \log_g p_1 \\
  \vdots \\
  \log_g p_m \\
\end{pmatrix} & \pmod{p-1}
\end{aligned}
$$

これよりガウスの消去法から $\log_g p_1, \ldots, \log_g p_n$ が求まる。よって次の式より $x$ が求まる。

$$
x = \sum_{j = 1}^me_j\log_g{p_j} - k \pmod {p-1}
$$

計算量は $\exp((\sqrt{2}+c)(\log n)^{1/2}(\log\log n)^{1/2})$ となる。

### 数体ふるい法

## DH
共有鍵を作る為の操作である。共有鍵を作ることができれば共有鍵暗号を用いて通信できる。

1. AliceとBobが巡回群 $G$ とその生成元 $g$ を共有する。
2. AliceとBobはそれぞれ秘密鍵 $x_a, x_b$ を生成し、公開鍵 $y_a = g^{x_a}, y_b = g^{x_b}$ を公開する。
3. AliceとBobは自分の秘密鍵と相手の公開鍵を掛けると $s = g^{x_ax_b} = y_b^{x_a} = y_a^{x_b}$ となり、$s$ はAliceとBobのみが知る共有鍵となる。

ECDH だと $s$ の $x$ 座標をハッシュ化したものを共有鍵として使う。

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
