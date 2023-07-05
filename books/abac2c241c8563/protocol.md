---
title: "より高機能な暗号"
---

今まで数学と暗号だけをやってきましたが、ようやく暗号をベースにした様々な仕組みを理解していきます。レイヤーが上がるので、出来るだけ数学は排除します。

開発中に〇〇させたくないなと思ったときにどんな技術を使えばよいのかを参照してもらえたらなと思います。その為にもライブラリ情報なども集めたいので詳しい方いたら教えていただきたいです。

[CRYPTREC 暗号技術ガイドライン（高機能暗号）](https://www.cryptrec.go.jp/report/cryptrec-gl-2005-2022.pdf)

## TL; DR

| したいこと | 技術 | ライブラリ |
| :-: | :-- | :-: |
| 秘匿性 | 暗号化通信, 権限管理 | |
| 改ざん防止 | MAC, AEAD | TLS/SSL で実装済み |
| なりすまし防止 | 認証, 電子署名 | |
| サイバー攻撃の耐性強化 | 秘密分散 | |
| 重要機密情報の交換 | 紛失通信 | |
| 情報を渡さずに処理<br>データ資産の安全な活用 | 準同型暗号 + ゼロ知識証明 | concrete |
| データベース暗号化 | | |
| 匿名化 |  | |
| | ブロックチェーン + ゼロ知識証明 | |
| 利用者情報を用いる | 関数型暗号 | |
| 通信盗聴防止 | 量子暗号 | |

## 関数型暗号
> **ID ベース暗号**
> **属性ベース暗号**
> **関数型暗号**

## 秘密計算
クラウドでプライバシーを保護しつつ、

- 秘密分散
- 紛失通信
- 準同型暗号
- 検索可能暗号

ここで目標を提示する。
秘匿検索システム
特許検索、紛失検索、医学データベース等
Beacon検索, ゲノム配列解析, 医療関連文書検索, 決定木の評価, 化合物検索

ゲノムは 40 塩基で個人情報と成り得ます。
保有するゲノム配列をデータベースと照合して、疾患リスクを算出したい。
ただゲノム配列は秘密
互いのゲノム配列を秘密にしたまま検索できないのか
指紋情報

両者の持つ機密情報を不正させずに共有することを実現する紛失通信を学ぶ。
信頼できる第三者がいれば、両者の機密情報をそこに渡し、両者に配ればよい。

まで企業に眠っていたデータ資産を、安全に活用することができたり
企業データについてクラウドなどの外部インフラを使うことへのハードルが下がったり、
企業間、部署間のデータ連携を、お互いのデータを秘匿したまま可能にしたり、
といったことが可能になります。

今まで分断されていたサプライチェーン間のデータに対し、秘密計算技術を用いてそれぞれのデータを連携させ、今まで存在していた煩雑なオペレーションを簡潔にしたり、今まで結合できなかったデータが結合できることにより可能になった、部門を横断したデータに対してより質の高いデータ分析をすることが可能になったりするでしょう。

https://www.ntt.com/business/services/secihi.html

> **Def. 秘密分散 (Secret Sharing)**
> 秘密情報を $n$ 個に分割して $t$ 個以上わかれば情報を復元でき、$t$ 個未満であれば全く復元出来ないように分散できるもの。

> **秘密分散の例 1 (Shamir's secret sharing)**
> 定数項 $a_0$ が秘密情報である $t-1$ 次多項式 $f(x) = \sum_i a_ix^i \in \mathbb{F}_p[x]$ とその $n$ 個の点 $\lbrace(x_i, f(x_i))\mid i = 1,\ldots,n\rbrace$ を生成する。
>
> これは $n$ 個の点の中で $t$ 個以上が分かればラグランジュの補間公式を用いて係数が分かり、$t$ 個未満であれば分からないので秘密分散の性質を満たす。

> **秘密分散の例 2 (Blakley's secret sharing)**
> $t$ 次元空間上で秘密情報を含む点を通る $n$ 枚の平行でない $t-1$ 次元超平面を生成する。
>
> これは $n$ 枚中 $t$ 枚以上分かると $1$ つの点で交わり、$t$ 枚未満だと分からないので秘密分散の性質を満たす。 (でも例 1 とは違って次元が絞られるのでは？)

秘密分散によっていいこと
- 1 つのサーバーを攻撃されて漏洩が起こったとしても情報不足で秘密が守られる。

> **Def. 紛失通信 (Oblivious Transfer)**
> 送信者は $n$ 個のメッセージを保有し、受信者はその内 $k$ 個を受信したいとする。このとき送信者が送った $n$ 個のメッセージのうち、どの $k$ 個を受信したか送信者が分からないような通信を k-out of-n 紛失通信という。

> **紛失通信の例 1 (Rabin-OT)**
> 1. アリスは秘密鍵と公開鍵を生成し、公開鍵とコミットメント $r_1, r_2$ を送る。
> 2. ボブは送られてきたコミットメントの内一方 $r\in\lbrace r_1, r_2\rbrace$ を選び、乱数 $x$ を暗号化した値 $E(x)$ との和 $q = E(x) + r\bmod N$ を送る
> 3. アリスは $y = D(q - r_i\bmod N)$ $c_i = m_i + y_i$
> 4. ボブは $m_b = c_b - x\bmod n$

紛失通信によって防げること
- 一方が情報を送った後、もう一方が不正に送り返さない
送信者は送った情報のうち、どれが届いたのかわからない
受信者は送られてきた情報のうち、一つの情報以外は得られない
- 一方が不正に誤った情報を送る


紛失通信プロトコルを用いて秘匿共通集合演算プロトコルというものを実現できる。

> **Def. 秘匿共通集合演算 (Private Set Intersection)**
>

> **検索可能暗号 (Publickey Encryption with Keyword Serach)**
> 暗号 + 全順序準同型
完全一致
部分一致
大小比較
平均値の集計
合計の集計
正規表現を用いた検索
データにつけたタグを検索
データに含まれる文字列を検索
異なるテーブルデータに対して結合しつつ検索
複雑な複合クエリを用いた検索
正規表現いけそう

:::message
**練習問題**
zer0pts CTF 2020 より dirty laundry
:::

## 準同型暗号
古典的な暗号は情報を守ることだけの機能だけしか持たなかったが、現代的な暗号は多機能な属性をもつようになった。その中でも実現すると社会が変わるようなものが完全準同型暗号である。
準同型暗号には次のような種類がある。

| 暗号 | 説明 |
| :--: | :-- |
| 加法準同型暗号 | 暗号化関数 $\mathcal{E}$ が加法準同型 $\mathcal{E}(x + y) = \mathcal{E}(x)\mathcal{E}(y)$ を満たす |
| 乗法準同型暗号 | 暗号化関数 $\mathcal{E}$ が乗法準同型 $\mathcal{E}(xy) = \mathcal{E}(x)\mathcal{E}(y)$ を満たす |
| レベル $n$ 準同型暗号 | 加法準同型かつ $n$ 回までの乗法準同型演算が成り立つ暗号 |
| 完全準同型暗号 | 加法準同型かつ乗法準同型な暗号 |

レベル $n$ 準同型暗号は $n$ 次方程式を計算できる。

準同型暗号ができることによって何ができるか

例えば準同型暗号には次のようなものがあります。

> **Unpadded RSA (乗法準同型暗号)**
>
> $$
\begin{aligned}
\mathcal{E}(m_1)\times\mathcal{E}(m_2) &= m_1^em_2^e & \bmod n \\
&= (m_1m_2)^e & \bmod n \\
&= \mathcal{E}(m_1m_2)
\end{aligned}
$$

> **ElGamal暗号 (乗法準同型暗号)**
>
> $$
\begin{aligned}
\mathcal{E}(m_1)\times\mathcal{E}(m_2) &= (g^{r_1},m_1h^{r_1})(g^{r_2},m_2h^{r_2}) \\
&= (g^{r_1+r_2},m_1m_2h^{r_1+r_2}) \\
&= \mathcal{E}(m_1m_2)
\end{aligned}
$$

> **Paillier暗号 (加法準同型暗号)**
>
> $$
\begin{aligned}
\mathcal{E}(m_1)\times\mathcal{E}(m_2) &= (g^{m_1}r_1^n)(g^{m_2}r_2^n) & \bmod n^2 \\
&= g^{m_1+m_2}(r_1r_2)^n & \bmod n^2 \\
&= \mathcal{E}(m_1+m_2)
\end{aligned}
$$

> **Lifted-ElGamal暗号 (レベル 2 準同型暗号)**
> 楕円曲線 $E/\mathbb{F}_p$, と生成元 $P\in E$
> 秘密鍵 $s\in\mathbb{F}_p$ と公開鍵 $sP$
> 平文 $m$ に対して乱数 $r$ をとり $c=(mP+rsP, rP)$
> $c = (S, T)$ に対して $S-sT = (mP+rsP)-s(rP)=mP$ としDLPを解いて $m$ を得る
> 加法準同型性
>
> $$
\begin{aligned}
\mathcal{E}(m_1)+\mathcal{E}(m_2) &= (m_1P+r_1sP,r_1P)+(m_2P+r_2sP,r_2P) \\
&= ((m_1+m_2)P, (r_1+r_2)sP, (r_1+r_2)P) \\
&= \mathcal{E}(m_1+m_2)
\end{aligned}
$$
>
> 乗法準同型性
>
> $$
\mathcal{E}(m_1)\times\mathcal{E}(m_2) := (e(S_1, S_2), e(S_1, T_2), e(T_1, S_2), e(T_1, T_2))
$$
>
> $$
\begin{aligned}
\mathcal{D}(c_1,c_2,c_3,c_4) &= \frac{c_1c_4^{s_1s_2}}{c_2^{s_2}c_3^{s_1}} = \frac{e(S_1,s_2)e(s_1T_1,s_2T_2)}{e(S_1,s_2T_2)e(s_1T_1,S_2)} \\
&=e(S_1-s_1T_1,S_2-s_2T_2) \\
&=e(mP_1,m'P_2)=e(P_1,P_2)^{mm'}
\end{aligned}
$$

### 完全準同型暗号
格子ベースの暗号で完全準同型暗号

#### ブートストラップ
1. ノイズを削減したい暗号文 $c$ について、(ビット単位などに分割した上で) $c$ をさらに暗号化する
2. 上で得られた $c$ の暗号文 $\hat{c}$ について、「$c$ を復号して平文を得る」という操作に対応する準同型演算を $\hat{c}$ に施す。
3. 暗号文 $\hat{c}$ の内部で $c$ がその平文 $m$ へ変換され、結果として $m$ の新たな暗号文が得られる。

> **TFHE**
> 現状で最も高速な部類に入る完全準同型暗号です。

https://eprint.iacr.org/2018/421.pdf
https://nindanaoto.github.io/

$\mathbb{T} := \mathbb{R}/\mathbb{Z}$ は環ではないです。
群 $+:\mathbb{T}\times\mathbb{T}\to\mathbb{T}$
作用 $\times:\mathbb{T}\times\mathbb{Z}\to\mathbb{T}$
$\mathbb{Z}_N[X] := \mathbb{Z}[X]/(X^N + 1)$
$\mathbb{T}_N[X] := \mathbb{R}[X]/(X^N + 1)\bmod 1$

> **Gadget Decomposition function**
> $\|d - \bm{v}\cdot H\|$ を最小化する $\bm{v}$ を求めるアルゴリズム
approximate decomposition

LWE は次のようなものだった。

$\bm{a}$: $U_{\mathbb{T}^n}$
$e$: $\mathcal{D}_{\mathbb{T},\alpha}$
$\bm{s}$: $U_{\mathbb{B}^n}$

$$
\begin{aligned}
b & = \bm{a}\cdot\bm{s} + m + e \\
m & \in\mathbb{B}, \mu = \frac{1}{8}\in\mathbb{T} \\
b & = \bm{a}\cdot\bm{s} + \mu(2\cdot m - 1) + e \\
m & = \frac{sgn(b - \bm{a}\cdot\bm{s}) + 1}{2}
\end{aligned}
$$

Torus Ring-LWE
SampleExtractIndex

$$
\begin{aligned}
\lceil Bg\cdot(a_0, a_1, b)\rfloor\cdot\frac{\mu}{Bg} & = (a_0^r, a_1^r, b^r) \approx \mu\cdot(a_0, a_1, b) \\
b^r - \bm{a}^r\cdot\bm{s} & = \mu(b - \bm{a}\cdot\bm{s} + e_r)
\end{aligned}
$$

Decomposition

$$
\begin{aligned}
& \arg\min\sum_{j=0}^{N-1}\left(a_j - \sum_{i=1}^l\frac{\overline{a}_{ij}}{Bg^i}\right)^2 \\
\overline{a}_{ij}& \in\left[-\frac{Bg}{2}, \frac{Bg}{2}\right) \\
\lceil Bg\cdot(a, b)\rfloor \\
(a, b)\approx(\overline{a}_1, \ldots, \overline{a}_l, \overline{b}_1, \ldots, \overline{b}_l)
\begin{pmatrix}
\frac{1}{Bg} & 0 \\
\frac{1}{Bg^2} & 0 \\
\vdots & \\
\frac{1}{Bg^l} & 0 \\
0 & \frac{1}{Bg} \\
0 & \frac{1}{Bg^2} \\
\vdots & \\
0 & \frac{1}{Bg^l} \\
\end{pmatrix}
\end{aligned}
$$

Blind Rotate

比較演算ができるのでデータベースに載せて高速に検索することができます。

最速の実装で NAND 回路が 10 ms 程度です。これは普通の回路の数万倍程度でかなり遅いです。専用ハードウェアを開発すれば 1 万倍の高速化され、実用に近づきます。それを研究しているのが


位置情報共有
Intel HEXL

これで準同型暗号に興味を持った方は Zama プロジェクトを調べてみると楽しいと思います。誰にも知られずに人工知能の推論や

## TEE
> **Def. TEE**
> TEE Intel SGX
> Controlled-Channel Attacks、Sealing Replay Attack、Foreshadow、Plundervolt、LVI、ÆPIC Leakなど

## プロトコル
### SSH

OpenSSH 9.0 ポスト量子暗号化時代への対応として、格子暗号系の「Streamlined NTRU Prime」と、楕円曲線暗号系の「x25519」からなるハイブリッド手法がデフォルトとなっている。

### SSL/TLS
HTTPS や SMTPS などで使用されている暗号プロトコル
OCSP Stapling 証明書をハンドシェイク時にやる

1995年 SSL 3.0 が公開
- Netscape Communications 社が開発
1999年 TLS 1.0 の制定
- 国際標準化の為に SSL から名称変更
- 既に SSL という名称が定着していた為、SSL という名前も残る
SSL は TLS の前身で現在は使用されていない。

**暗号スイート**
通信に使われるプロトコル/暗号の組み合わせのことです。
スイートはお菓子 Sweet ではなくて装備一式 Suite です。

公開鍵暗号を用いて共通鍵暗号を渡すプロトコル

草川恵太先生が第一人者でしょうか
興味がある人は暗号に特化した論文サイト [ePrint Archive](https://eprint.iacr.org/) を覗いてみると良いと思います。

**Goldwasser–Micali cryptosystem**

## 参考文献

- [Schnorr署名 ―― 30年の時を超えて注目を集める電子署名](https://blog.visvirial.com/articles/721)
- [RFC8235](https://datatracker.ietf.org/doc/html/rfc8235)
- https://letsencrypt.org/docs/challenge-types/
- [ペアリングベースの効率的なレベル2準同型暗号（SCIS2018） (slideshare.net)](https://www.slideshare.net/herumi/2scis2018)

この資料は CC0 ライセンスです。