---
title: "楕円曲線暗号への攻撃"
---


### 楕円曲線暗号とは

楕円曲線暗号 (ECC) はRSA暗号と同時期に開発された暗号で1985年頃に Victor S. Miller と Neal Koblitz が同時期かつ独立に発明しました(ちなみにMiller-Rabin素数判定法のMillerはGary L. Millerで別人です)。特徴としては RSA 暗号よりも純粋に強い暗号であることや鍵長が短いことなどが挙げられます。

楕円曲線というのは次の関数からなる曲線のことです。

$$
y^2 = x^3 + ax + b
$$

掴みを見れたと思うのですがよく理解するには正確な定義が必要です。
数学的な話はあんまり

射影平面上の楕円曲線

射影平面で定義する理由

- 無限遠点を数学的に定義できる
- 実装時に割り算の遅延ができ高速化できる

ここにグラフ

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

さて、ここでこの楕円曲線上の加法を用いた次のような問題を作れます。

---

楕円曲線上の点 $P, Q$ に $Q=dP$ という関係があるとき $d$ を求めよ。

---

つまり楕円曲線の世界で「割り算」をしなさいという問題です。

実はこの問題はとても難しく、これを解く効率的なアルゴリズムは現在見つかっていません。これを楕円曲線上の離散対数問題 (ECDLP : Elliptic Curve Discrete Logarithm Problem) と呼びます。この ECDLP を利用して暗号の形にしたものが楕円曲線暗号です。

具体例としては色々ありますがCTFで最もよく出るのがECDHです。

## 楕円曲線の位数
Schoof
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

#### ECDH (Elliptic curve Diffie–Hellman key exchange)

暗号通信をする為に使われる暗号プロトコルです。

Alice と Bob は AES などの共通鍵暗号を用いて暗号通信しようとしていますが、始めに2人だけの秘密である共有鍵が必要です。しかしそれを直接共有してしまうと、第三者から鍵を盗聴されて通信を覗き見られてしまいます。そこで暗号を用いることで鍵を直接共有することなく共有鍵を構築することができます。この手法をディフィーヘルマン鍵共有 (DH) と呼び、DH の中でも ECDLP を安全性根拠とする DH を楕円曲線ディフィーヘルマン鍵共有 (ECDH) と呼びます。

具体的には次のような方法で ECDH を実現します。

ここに図

1. AliceとBobが楕円曲線のパラメータとベースポイント $G$ を共有する。
2. AliceとBobはそれぞれランダムな値 $d_A, d_B$ を生成し、$d_A, d_B$ を秘密鍵、$Q_A = d_AG, Q_B = d_BG$ を公開鍵として公開する。
3. AliceとBobは自分の秘密鍵と相手の公開鍵を掛けると $S = d_Ad_BG = d_AQ_B = d_BQ_A$ となり、$S$ の $x$ 座標をハッシュ化したものが Alice と Bob のみが知る共通鍵となる。

このように攻撃者は $(G, dG)$ が分かっても ECDLP が解けない為に $d$ が分からず、安全に共通鍵を共有することができます。

ここに実装

暗号標準を定める国際機関によって楕円曲線が
https://neuromancer.sk/std/

### 攻撃手法

この ECDLP を解くことができれば ECDH を含め、様々な楕円曲線暗号を解くことができます。さて主に攻撃対象となる楕円曲線暗号は以下のようなものがあります。

1. 現在見つかっている最も高速なアルゴリズムで ECDLP を解く。計算量は $O(\sqrt{\#E/\mathbb{F}_p})$ となる。
2. $\#E/\mathbb{F}_p = p_1^{e_1}p_2^{e_2}\ldots p_k^{e_k}$ (位数が Smooth number なとき) ならば位数 $p_i$ の小さな ECDLP に分解できる。それぞれ計算量は $O(\sqrt{p_i})$ である。
3. $\#E/\mathbb{F}_p = p$ (Anomalous なとき) ならば $\mathbb{F}_p^+$ 上の DLP に帰着できる。
4. $\#E/\mathbb{F}_p = p+1$ (Supersingular なとき) ならば埋め込み次数 $k$ を用いて $\mathbb{F}_{p^k}^\times$ 上の DLP に帰着できる。実践上 $k$ はそこまで大きくならない。
5. $\Delta(E/\mathbb{F}_p) = 0$ (Singular なとき) ならば $\mathbb{F}_p^+$ や $\mathbb{F}_p^\times, \mathbb{F}_{p^2}^\times$ 上の DLP に帰着できる。
6. 楕円曲線上に存在しない点や位数の少ない点を指定できるならば秘密鍵の情報を取り出すことができる。

それぞれ

### 1. ECDLP

### 2. Polig-Hellman Attack

中国剰余定理を用いて大きな群を複数の小さな群の直積に分ける。

楕円曲線暗号の楕円曲線の位数は細かく素因数分解できることが多い。

楕円曲線の位数 $\#E = p_1^{e_1}p_2^{e_2}\ldots p_k^{e_k}$ と素因数分解できる。$Q = dP$ となるとき次のように $d_i$ を置く。

$$
\begin{aligned}
d_1 &= d \pmod{p_1^{e_1}} \\
d_2 &= d \pmod{p_2^{e_2}} \\
&\vdots \\
d_k &= d \pmod{p_k^{e_k}} \\
\end{aligned}
$$

それぞれの $d_i$ について次のように書ける。

$$
d_i=z_0+z_1p_i+z_2p_i^2+\ldots+z_{e_i−1}p_i^{e_i−1} \pmod{p_i^{e_i}} \quad (∀k:z_k \in [0,p_i))
$$

ここで $P_{i,j}=\frac{\#E}{p_i^j}P, Q_i=\frac{\#E}{p_i^j}Q$ とおくと

$$
Q_{i,1} = d_{i,1}P_{i,1} = (z_0+z_1p_i+z_2p_i^2+\ldots+z_{e_i−1}p_i^{e_i−1})P_{i,1} = z_0P_i
$$

となり, $z_0 < p_i$ である $Q_i = z_0P_i$ についてDLPを解けば良い。

他については $Q_i' = (Q_i - z_0P_i) / p_i$ とおいて同様に解けると思っていたけど除算は ECC の DDH 問題から不可能っぽいので $p_i^{e_i}$ 全探索するしかないかも。

```python
fact = factor(G.order())
ord = int(G.order())
dlogs = []
for p, e in fact:
    t = ord // p ^ e
    dlog = discrete_log(t * Q, t * G, operation="+")
    dlogs += [dlog]

print(crt(dlogs, primes))
```

### 3. MOV/FR Reduction
MOV Reduction (Menezes-Okamoto-Vanstone Reduction)
Weil-pairing
FR Reduction (Frey-Rück Reduction)
Tate-pairing

Tate-pairing, Bilinear-pairingのいい資料がみつからない
$R=dP$ となるECDLPを解く

Weil pairing $e_n: E[n]\times E[n]\to \mu_n\subseteq \mathbb{F}_{p^k}^*$

1. $E[n]\subseteq E(\mathbb{F}_{p^k})$ となる最小の $k$ を持ってくる
2. 位数 $n$ の $\alpha=e_n(P, Q)$ となるように $Q \in E[n]$ を取ってくる
3. $\beta = e_n(dP, Q)$
4. $\mathbb{F}_{p^k}^*$ 上のDLPを $\alpha, \beta$ を用いて解く

$k$ は多くとも6まで
$E(\mathbb{F}_{p^k}^*)\cong\mathbb{Z}_{c_1n_1}\oplus\mathbb{Z}_{c_2n_1}$



### 4. SSSA Attack / Supersingularな曲線を用いてはいけない
$E/\mathbb{F}_q, \gcd(t, q) > 1$ 埋め込み次数 $k\leq6$ 全埋込み次数は既知
$E[p]  \mathbb{Z}/p\mathbb{Z}$
$\pi:\mathbb{P}^2(\mathbb{Q}_p) \to \mathbb{P}^2(\mathbb{F}_p)$
$\phi:\ker\pi \to \mathcal{E}(p\mathbb{Z}_p)$


### 5. Singularな曲線を用いてはいけない

Singular な楕円曲線の場合、特異点が存在し、特異点には尖点 (cusp) 、節点 (node) の2種類がある。$c_4 = 0$ と尖点は同値である。

$y = 0$ 上の特異点が原点 $O(0, 0)$ となるように平行移動させる。すると任意の Singular な楕円曲線は $y^2 = x^3 + kx^2$ の形となる。

$$
\begin{aligned}
\begin{dcases}
E(\mathbb{F}_p) \rightarrow \mathbb{F}_p^+: (x,y)↦\frac{x}{y},&∞↦0 & (cuspのとき)\\
E(\mathbb{F}_p) \rightarrow \mathbb{F}_p^\times: (x,y)↦\frac{y + \sqrt{k}x}{y - \sqrt{k}x},&∞↦1 & (nodeのとき) \\
\end{dcases}
\end{aligned}
$$

↑これはなぜ？

### 6. Invalid Curve Attack


### Weil decsent
### ECFFT


より面白い話題として楕円曲線暗号にバックドアが仕込まれていたという事件がある。
- Dual EC DRBG

構成
- SIDH SIKE
- ECDH
- CSIDH

https://blog.z.cash/new-snark-curve/ :  BLS12-381: New zk-SNARK Elliptic Curve Construction
unefficient key recovery

Diffie Hellman
E/P/Q = E/Q/P
ECDDH
E/P E/Q -> E/PQ

Call Stack Graph
エラーが起きたところをいい感じにたどれる

同型写像
SIDH


Python での高速な実装 fastecdsa