---
title: "耐量子暗号"
---

量子コンピュータにより DLP や素因数分解問題は Shor のアルゴリズムを用いて多項式時間で計算出来ることがわかりました。これにより従来多く用いられてきた公開鍵暗号である RSA 暗号や楕円曲線暗号は量子コンピュータによって解かれてしまいます。その為量子コンピュータに解かれないような暗号: 耐量子暗号を開発する必要が出てきました。

[OpenSSH 9.0](https://www.zdnet.com/article/openssh-now-defaults-to-protecting-against-quantum-computer-attacks/) や [AWS Key Management Service](https://aws.amazon.com/jp/blogs/news/round-2-post-quantum-tls-is-now-supported-in-aws-kms/) でも耐量子暗号をサポートし普及しています。

耐量子暗号の仕組みや攻撃について紹介します。ただなぜ量子コンピュータで解かれないのかについてはよく分かっていないのでお答え出来ません。ここらへんについて詳しい方がいたらご教授頂きたいです。

## Post-Quantum Cryptography Standardization
耐量子暗号は英語で Post-Quantum Cryptography 略して PQC といいます。

NIST は全世界から PQC 標準化候補の方式を募集し、選考、標準規格の作成を行っています。

https://csrc.nist.gov/Projects/post-quantum-cryptography/post-quantum-cryptography-standardization

応募された方式は大きく 5 種類の暗号に分類できます。

- 格子暗号 (Lattice-based cryptography)
- 符号暗号 (Code-based cryptography)
- 同種写像暗号 (Isogeny-based cryptography)
- 多変数多項式暗号 (Multivariate polynomial cryptography)
  - 鍵自体やアルゴリズムはとても短いですが安全性は格子暗号に比べて低いらしく選考に外されたようです。これを紹介するか迷ってます。
- ハッシュ関数署名 (Hash-based signature)

現在 (2023/3/8) は NIST PQC Standarization の第 3 ラウンドが終了し、最終第 4 ラウンドの候補として次が挙げられています。

| 公開鍵暗号/鍵共有 | 署名 |
| :---- | :--- |
| CRYSTALS-KYBER (格子暗号) <br> BIKE (符号暗号) <br> Classic McElience (符号暗号) <br> HQC (符号暗号) <br> SIKE (同種写像暗号) | CRYSTALS-Dilithium (格子暗号) <br> FALCON (格子暗号) <br> SPHINCS+ (ハッシュ関数署名) |

今回はこれらの紹介をしていきたいと思います。
また量子の性質を用いて鍵配送問題を解決する量子暗号プロトコルがあります。これは誰かに見られたかを判定します。他の資料におまかせします。

[CRYPTREC 耐量子計算機暗号の研究動向調査報告書](https://www.cryptrec.go.jp/report/cryptrec-tr-2001-2022.pdf)

## 格子暗号

CRYSTALS-KYBER と引き合いとして出されるのが NTT が開発した NTRU 暗号です。
OpenSSH 9.0 格子暗号系の「Streamlined NTRU Prime」と、楕円曲線暗号系の「x25519」からなるハイブリッド手法がデフォルトとなる。

> **Def. Ternary Polynomials**
> 係数が $\pm 1$ の多項式の集合である。具体的には $\mathcal{T}(d_{+1}, d_{-1})$ を $d_{+1}$ 個の $+1$ 係数と $d_{-1}$ 個の $-1$ 係数のある多項式の集合とする。

$$
\begin{aligned}
f & = 1+3x+4x^2+5x^3+2x^5 \in R_p \\
\phi(f) & = 1+3x−3x^2−2x^3+2x^5 \in R
\end{aligned}
$$

> **NTRU**
> - 鍵生成
>   1. $f\in\mathcal{T}(d+1, d)$, $g\in\mathcal{T}(d, d)$ として $h = (f^{-1} \bmod q)g$
> - 暗号化
>   1. $m \in R_p$ を Center lift する
>   2. $r\in\mathcal{T}(d, d)$ として $e = prh + m \pmod q$ を返す
> - 復号
>   1. $a = fe\pmod q$ の Center lift をする
>   2. $m = a(f^{-1}\bmod p)\pmod p$

環とイデアルで割った部分環について何らかの方法で持ち上げることができるとき暗号を構成できる

## 符号暗号
符号理論にも様々な計算困難な問題がありますが、候補は誤り訂正符号の復号問題の計算困難性を利用した暗号が主要です。計算問題として SDP や DSDP の困難性を仮定しています。

> **SDP; Syndrome Decoding Problem**
> SDP とは符号長 $n$ として次元 $k$ をパリティ検査行列 $\bm{H}\in\mathbb{F}_2^{(n-k)\times n}$ とシンドローム $\bm{s}\in\mathbb{F}_2^{n-k}$ に対して $\bm{eH}^T = \bm{s}$ となるハミング重みが $w$ の $\bm{e}\in\mathbb{F}_2^n$ を求める問題である。

つまり $w$ 個の 1 があるビット列 $\bm{e}$ に行列を作用させると $\bm{s}$ となるような $\bm{e}$ を求める問題です。これは LWE 問題の派生と見ることができます。

それぞれの暗号は使う符号が異なります。逆に言えばそれぞれの暗号の違いはほぼそれくらいです。

| 暗号 | 符号 | 安全性 | 開発組織 |
| :-- | :-- | :-- | :-- |
| BIKE | QC-MDPC 符号 | IND-CPA | Intel |
| Classic McElience | Goppa 符号 | IND-CCA | TU Eindhoven |
| HQC | Quasi-Cyclic 符号 | IND-CCA | University of Limoges |

解くのは

> **Information Set Decoding**

## 同種写像暗号
楕円曲線の章で超特異同種写像ディフィー・ヘルマン鍵共有 (SIDH / SIKE) を扱いましたが、ここでは CSIDH を扱います。

## ハッシュ関数署名 (Hash-based signature)
ハッシュ値を用いるだけの退屈な署名です。

Lamport 署名 とは 1979 年に Leslie Lamport が構築されたハッシュ関数署名です。これは 1 ビットごとに署名します。

> **Lamport 署名**
> 1. 秘密鍵 $s_0, s_1$ を生成し、公開鍵 $p_i = H(s_i)$ を計算します
> 2. 1 ビットのメッセージ $m$ を用いて $s_m$ を署名として公開します
> 3. $p_m = H(s_m)$ となることを確認して検証します

非常にシンプルな署名です。ただ公開鍵に大量のメモリを割いているので、これを短い公開鍵で大量のメッセージを署名できるように改良したいです。

> **One-time signature (Winternitz OTS)**
> 1. 32 個の 256 ビット乱数を秘密鍵とし、それぞれ 256 回ハッシュ関数を通したものを公開鍵とする
> 2. メッセージを SHA-256 でハッシュ化した $N$ を 8 ビットごとに分けて $N = N_1\|\cdots\|N_{32}$ とする
> 3. 公開鍵をそれぞれ $256N_i$ 回ハッシュ化したものを署名として公開する

> **Hash to Obtain Random Subset (HORS)**
> 1. 256 個の乱数を秘密鍵とし、それぞれハッシュ化したものを公開鍵とする
> 2. メッセージを 128 ビットのハッシュ関数を通して 16 個の 8 ビットに分けて $N = N_1\|\cdots\|N_{16}$ とする
> 3. $N_i$ 番目の公開鍵を連結させたものを署名として公開する

### Merkle trees
それには Merkle が提案した二分木の一種 Merkle 木を使います。

HashMap とか HAMT

$$
h_{i+1,j/2} = H(h_{i,j}\|h_{i,j+1})
$$

### SPHINCS+
鍵生成コストを下げた
randomized tree-based stateless signature

## 多変数多項式暗号
> 松本勉と今井秀樹は隠れ単項式暗号系というエレガントな方法を 1988 年開催の国際会議 EUROCRYPT で発表した。これは現在、松本-今井暗号と呼ばれている。1996年、パタリン (J. Patarin) は松本-今井暗号を解読し、翌 97 年、松本-今井暗号を拡張した HFE (Hidden Field Equation) を提案した。この HFE は 2003 年に J.-C. Fauge`re と A. Joux により、グレブナー基底という一見直截的な攻撃法によって破られている。
> 暗号理論と楕円曲線より

> **MQ 問題**
> 有限体 $\mathbb{F}_q$ 上を係数とする $m$ 個の $n$ 変数の $2$ 次多項式の共通解をひとつ求めよ

> **Def. 松本-今井暗号**
> $v$

グレブナー基底

## まとめ

## 参考文献
- Post-Quantum Cryptography
- 耐量子計算機暗号
- [量子コンピュータに耐性のある暗号技術の標準化動向：米国政府標準暗号について](https://www.imes.boj.or.jp/research/papers/japanese/19-J-04.pdf)
- [An efficient key recovery attack on SIDH](https://eprint.iacr.org/2022/975)
- [Constructing Digital Signatures from a One Way Function](https://www.microsoft.com/en-us/research/publication/constructing-digital-signatures-one-way-function/)

この資料は CC0 ライセンスです。