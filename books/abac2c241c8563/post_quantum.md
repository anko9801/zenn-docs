---
title: "耐量子暗号"
---

量子コンピュータにより DLP や素因数分解問題は Shor のアルゴリズムを用いて多項式時間で計算出来ることがわかりました。これにより従来多く用いられてきた公開鍵暗号である RSA 暗号や楕円曲線暗号は量子コンピュータによって解かれてしまいます。その為量子コンピュータに解かれないような暗号: 耐量子暗号を開発する必要が出てきました。

OpenSSH や [AWS Key Management Service](https://aws.amazon.com/jp/blogs/news/round-2-post-quantum-tls-is-now-supported-in-aws-kms/) でも耐量子暗号をサポートし

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

## 格子暗号

耐量子暗号の中では有力ですが耐量子性は明解ではないです。

SVP, GapSVP, SIVP, CVP, SIS, LWE, LWR, Approx-GCD, BGV, GSW などなど実にさまざまな計算困難な問題があり、それに対応する暗号がありますが、今回は耐量子暗号や完全準同型暗号でよく使われる LWE 問題に絞って紹介します。

### LLL 基底簡約のその先へ

第三章で基底簡約することにより SVP が解き易くなるということで LLL 基底簡約アルゴリズムまでやりましたが、LWE 問題の基礎である CVP を解く為にはより強い基底簡約が必要です。まずはそれらを紹介し CVP の解き方、LWE 格子暗号とその派生について学びます。

LLL 基底簡約は

> **HKZ (Hermite-Korkine-Zolotareff) 基底簡約**
> 1. サイズ基底簡約
> 2. すべての $1\leq i\leq n$ に対して $\|\boldsymbol{b}_i^*\| = \lambda_1(\pi_i(L))$ を満たすように基底ベクトルの交換

> **BKZ (Block Korkine-Zolotareff) 基底簡約**
> 1. サイズ基底簡約
> 2. すべての $1\leq k\leq n-\beta+1$ に対して $\beta$ 次元の格子 $L_{[k,k+\beta-1]} = \lbrace\pi_k(\boldsymbol{b}_k), \pi_k(\boldsymbol{b}_{k+1}), \ldots, \pi_k(\boldsymbol{b}_{k+\beta-1})\rbrace$ の基底が HKZ 基底簡約

BKZ を応用したさらなる基底簡約アルゴリズムは BKZ2.0, RSR, G6K などがあります。参考に元論文を置いておきます。

### 最近ベクトル問題 (Closest Vector Problem)
SVP は原点に最も近い格子点を求める問題でしたが CVP はある点に最も近い格子点を求める問題です。

> **CVP; Closest Vector Problem**
> CVP とは目標ベクトル $\boldsymbol{w}$ (格子点である必要はない) に対して格子上の最近ベクトル $\boldsymbol{v}$ を求める問題である。

厳密解を求めるのは難しいので近似解を求めて偶然厳密解となることを祈ります。

この近似解を解く代表的な方法に Babai’s nearest plane algorithm と Kannan's embedding method があります。

> **Babai’s nearest plane algorithm**
> $n$ 次元の格子について目標ベクトルに対して最も近い $n-1$ 次元超平面を $1$ つ選ぶことを帰納的に繰り返すことで CVP を解く。

$n$ 次元の格子を $n-1$ 次の $W\subset \Lambda$ とその直交補空間 $W^\top$ に分割します。

$$
\begin{aligned}
  \boldsymbol{w} & = \sum a_i\boldsymbol{b}_i \\
  U + \boldsymbol{y} & = \lbrace \boldsymbol{u} + \boldsymbol{y}\mid \boldsymbol{u}\in U\rbrace
\end{aligned}
$$

```python
def babai_cvp(B, w):
    n = B.nrows()
    BB, _ = B.gram_schmidt()
    e = w
    for i in range(n)[::-1]:
        c = (e.dot_product(BB[i]) / BB[i].dot_product(BB[i])).round()
        e -= c * B[i]
    return w - e


B = matrix([[1, 2, 3], [3, 0, -3], [3, -7, 3]])
w = vector([10, 6, 5])
v = babai_cvp(B, w)
print(v)
```

> **Kannan’s embedding method**
> CVP の目標ベクトル $w$ と解ベクトル $v$ の差 $e = w - v$ のノルムについて $\|e\| < \lambda_1/2$ が成り立つとき SVP を解くことで求まる。

次の行列を基底簡約することで

$$
\begin{pmatrix}
  \boldsymbol{B} & \boldsymbol{0}^\top \\
  \boldsymbol{w} & M
\end{pmatrix}
=
\begin{pmatrix}
b_{11} & \cdots & b_{1m} & 0 \\
\vdots & \ddots & \vdots & \vdots \\
b_{n1} & \cdots & b_{nm} & 0 \\
w_1 & \cdots & w_m & M
\end{pmatrix}
$$

```python
def kannan_cvp(B, w):
    n = B.nrows()
    M = 1
    BB = block_matrix([[B, matrix(ZZ, n, 1)], [w, M]])
    BB = BB.LLL()
    e = matrix(BB[0][0:n])
    return w - e


B = matrix([[1, 2, 3], [3, 0, -3], [3, -7, 3]])
w = matrix([10, 6, 5])
v = kannan_cvp(B, w)
print(v)
```

このような格子の分野において求解困難な問題はたくさんあり、これを用いた暗号はたくさんあります。CVP を応用した問題 LWE (Learning With Error) 問題を用いた暗号を LWE 格子暗号といいます。

### LWE格子暗号

機械学習理論から派生した求解困難な問題で

> **LWE 問題**
> $K$ を体とする。$\boldsymbol{A}\in K^{m\times n}, \boldsymbol{s}\in K^m$ を掛けて誤差ベクトル $\boldsymbol{e}\in K^n$ を与えた $\boldsymbol{b}\in K^n$ に対し $(\boldsymbol{A}, \boldsymbol{b})$ が与えられたときに $\boldsymbol{s}$ を求める問題を LWE 問題と呼ぶ。
>
> $$
\boldsymbol{A}\boldsymbol{s} + \boldsymbol{e} = \boldsymbol{b}
$$

この誤差がなければ逆行列を掛ければすぐに求まるのですが、誤差があることで問題が難しくなります。

$s$ や $e$ は $0$ から離れ過ぎると他の解と混ざってしまうので $0$ に近い乱数値が選ばれます。この乱数の分布は正規分布や二項分布などが用いられます。

$q$ を素数として $K = \mathbb{F}_q$ のとき LWE 問題、 $K = \mathbb{F}_q[x]/(x^n + 1)$ かつ $m = 1$ のとき Ring-LWE 問題、 $K = \mathbb{F}_q[x]/(x^n + 1)$ のとき Module-LWE 問題といいます。注意として $x^n + 1$ は $\mathbb{F}_q[x]$ において既約多項式であるから $\mathbb{F}_q[x]/(x^n + 1)$ は体となります。

これが $B \geq 2\sqrt{n}$ のとき計算困難な問題であることが知られています。


近似解は LLL して CVP 解く感じです。

```python
def solve_LWE(A, b):
    m = A.nrows()
    n = A.ncols()
    q = A.base_ring().order()
    BB = block_matrix([[A.change_ring(ZZ).transpose()], [q * matrix.identity(m)]])
    BB = BB.LLL()[n:]
    v = babai_cvp(BB, b.change_ring(ZZ))
    s = v[:n] * A[:n].transpose() ^ -1
    return s


q = 29
A = matrix(
    GF(q),
    [
        [1, 5, 21, 3, 14],
        [17, 0, 12, 12, 13],
        [12, 21, 15, 6, 6],
        [4, 13, 24, 7, 16],
        [20, 9, 22, 27, 8],
        [19, 8, 19, 3, 1],
        [18, 22, 4, 8, 18],
        [6, 28, 9, 5, 18],
        [10, 11, 19, 18, 21],
        [28, 18, 24, 27, 20],
    ],
)
b = vector(GF(q), [28, 2, 24, 16, 11, 14, 7, 28, 27, 13])
s = solve_LWE(A, b)
print(s)
```

### Module-LWE

今後簡単の為に $R_q = \mathbb{F}_q[x]/(x^n+1)$, $R = \mathbb{Z}[x]/(x^n+1)$ とおきます。

多項式同士の積で畳み込みを計算するので数論変換 NTT; Number Theoretic Transform を用いると次数 $n$ として $\mathcal{O}(n\log n)$ と高速に計算できる。

また LWE ではデータ圧縮をよく行う。$d < \lceil\log_2(q)\rceil$ として $\mathbb{F}_q\to\lbrace 0,\ldots,2^d-1\rbrace$ と圧縮し、次のような性質を満たす。

$$
\begin{aligned}
& x' = \mathrm{Decompress}_q(\mathrm{Compress}_q(x, d), d) \\
& |x' - x\bmod q|\leq\lceil q/2^{d+1}\rfloor
\end{aligned}
$$

つまり圧縮解凍をしたときの誤差が小さければ LWE の誤差に乗せることができるという意味です。
次のように定義すると上記性質を満たす。

$$
\begin{aligned}
\mathrm{Compress}_q(x, d) & = \lceil(2^d/q)x\rfloor \bmod q \\
\mathrm{Decompress}_q(x, d) & = \lceil(q/2^d)x\rfloor
\end{aligned}
$$

また $R_q$ であれば各要素に対して圧縮を行う。
圧縮 $\mathrm{Compress}_q(x, d)$

### CRYSTALS-KYBER
行列の圧縮には
シード値 $\rho$ から多項式を生成する関数 $Sam(\rho)$ を用いる。
$n = 256$ ビットの

鍵生成
1. 疑似乱数で $\boldsymbol{A}\in R_q^{k\times k}$ と二項分布で $\boldsymbol{s}, \boldsymbol{e}\in R^{k}$ を生成し、$\boldsymbol{t} = \boldsymbol{A}\boldsymbol{s} + \boldsymbol{e}$ を計算する
2. $(\boldsymbol{t}, \boldsymbol{A})$ を圧縮したものを公開鍵、$\boldsymbol{s}$ を秘密鍵とする

暗号化
平文 $m$ を用いて
1. 二項分布で $\boldsymbol{r}, \boldsymbol{e}_1\in R^k, e_2\in R$ を生成する
2. $\boldsymbol{t}$ を解凍して $(\boldsymbol{u}, v) = (\boldsymbol{A}^T\boldsymbol{r} + \boldsymbol{e}_1, \boldsymbol{t}^T\boldsymbol{r} + e_2 + \lceil\frac{q}{2}\rfloor \cdot m)$ を圧縮したものを暗号文として返す

復号
1. $\boldsymbol{u}, v$ を解凍して $\mathrm{Compress}_q(v - \boldsymbol{s}^T\boldsymbol{u}, 1)$ つまり $q / 2$ に近い値は $1$ 、$0$ に近い値は $0$ として返す

### NTRU
CRYSTALS と引き合いとして出されるのが NTT が開発した NTRU 暗号です。

> **Def. Ternary Polynomials**
> 係数が $\pm 1$ の多項式の集合である。具体的には $\mathcal{T}(d_{+1}, d_{-1})$ を $d_{+1}$ 個の $+1$ 係数と $d_{-1}$ 個の $-1$ 係数のある多項式の集合とする。

$$
\begin{aligned}
f & = 1+3x+4x^2+5x^3+2x^5 \in R_p \\
\phi(f) & = 1+3x−3x^2−2x^3+2x^5 \in R
\end{aligned}
$$

鍵生成
1. $f\in\mathcal{T}(d+1, d)$, $g\in\mathcal{T}(d, d)$ として $h = (f^{-1} \bmod q)g$

暗号化
1. $m \in R_p$ を Center lift する
2. $r\in\mathcal{T}(d, d)$ として $e = prh + m \pmod q$ を返す

復号
1. $a = fe\pmod q$ の Center lift をする
2. $m = a(f^{-1}\bmod p)\pmod p$

環とイデアルで割った部分環について何らかの方法で持ち上げることができるとき暗号を構成できる

## 符号暗号
符号理論にも様々な計算困難な問題がありますが、候補は誤り訂正符号の復号問題の計算困難性を利用した暗号が主要です。計算問題として SDP や DSDP の困難性を仮定しています。

> **SDP; Syndrome Decoding Problem**
> SDP とは符号長 $n$ として次元 $k$ をパリティ検査行列 $\boldsymbol{H}\in\mathbb{F}_2^{(n-k)\times n}$ とシンドローム $\boldsymbol{s}\in\mathbb{F}_2^{n-k}$ に対して $\boldsymbol{eH}^T = \boldsymbol{s}$ となるハミング重みが $w$ の $\boldsymbol{e}\in\mathbb{F}_2^n$ を求める問題である。

つまりハミング重み $w$ の符号 $\boldsymbol{e}$ をパリティチェックすると $\boldsymbol{s}$ となるような $\boldsymbol{e}$ を求める問題です。

これは LWE 問題の派生と見ることができます。

TODO: これがなぜ暗号となるのか？

それぞれの暗号は使う符号が異なります。逆に言えばそれぞれの暗号の違いはほぼそれくらいです。

| 暗号 | 符号 | 安全性 | 開発組織 |
| :-- | :-- | :-- | :-- |
| BIKE | QC-MDPC 符号 | IND-CPA | Intel |
| Classic McElience | Goppa 符号 | IND-CCA | TU Eindhoven |
| HQC | Quasi-Cyclic 符号 | IND-CCA | University of Limoges |

### Information Set Decoding

## 同種写像暗号
超特異同種写像ディフィー・ヘルマン鍵共有 (SIDH / SIKE)
CSIDH
現在 SIKE しかありませんが攻撃が見つかっている為選考に残るのは難しいです。

### SIKE
$p = w_A^{e_A}w_b^{e_B}f \pm 1$
$\mathbb{F}_{p^2}$ 上の超特異楕円曲線 $E$
位数 $w_A^{e_A}$ である点 $P_A, Q_A$ と位数 $w_B^{e_B}$ である点 $P_B, Q_B$

$2^a\approx 3^b$ となるような素数 $p = 2^a3^bf - 1$ を用いて超特異楕円曲線 $E_0/\mathbb{F}_{p^2}$、つまり位数が $\#E_0(\mathbb{F}_{p^2}) = (p + 1)^2$ となる楕円曲線を生成する。
$P_0, Q_0 \in E_0[2^a]$ $E_0[3^{e_3}]$
$3^b$ 同種写像 $\varphi: E_0\to E$

$$
\begin{aligned}
y^2 & = x^3 + x & y^2 & = x^3 + 6x^2 + x
\end{aligned}
$$

```python
# SIKEp377
p = 2^191 * 3^117 - 1
# SIKEp546
p = 2^273 * 3^172 - 1
# SIKEp697
p = 2^356 * 3^215 - 1

Fp2<I> = GF(p, 2)
assert I^2 == -1
R<x> = PolynomialRing(Fp2)
E = EllipticCurve(x^3 + 6*x^2 + x)
```

### SIKE への攻撃
去年

$$
y^2 = x^3 + x \qquad y^2 = x^3 + 6x^2 + x
$$

**Thm. Kani's theorem**


https://eprint.iacr.org/2022/975

## ハッシュ関数署名 (Hash-based signature)
ハッシュ値を用いるだけの退屈な署名です。

### Lamport 署名
Lamport 署名 とは 1979 年に Leslie Lamport が構築されたハッシュ関数署名です。これは 1 ビットごとに署名します。

1. 秘密鍵 $s_0, s_1$ を生成し、公開鍵 $p_i = H(s_i)$ を計算します
2. 1 ビットのメッセージ $m$ を用いて $s_m$ を署名として公開します
3. $p_m = H(s_m)$ となることを確認して検証します

非常にシンプルな署名です。ただ公開鍵に大量のメモリを割いているので、これを短い公開鍵で大量のメッセージを署名できるように改良したいです。

### One-time signature (Winternitz OTS)
1. 32 個の 256 ビット乱数を秘密鍵とし、それぞれ 256 回ハッシュ関数を通したものを公開鍵とする
2. メッセージを SHA-256 でハッシュ化した $N$ を 8 ビットごとに分けて $N = N_1\|\cdots\|N_{32}$ とする
3. 公開鍵をそれぞれ $256N_i$ 回ハッシュ化したものを署名として公開する

### Hash to Obtain Random Subset (HORS)
1. 256 個の乱数を秘密鍵とし、それぞれハッシュ化したものを公開鍵とする
2. メッセージを 128 ビットのハッシュ関数を通して 16 個の 8 ビットに分けて $N = N_1\|\cdots\|N_{16}$ とする
3. $N_i$ 番目の公開鍵を連結させたものを署名として公開する

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
> **MQ 問題**
> 有限体 $\mathbb{F}_q$ 上を係数とする $m$ 個の $n$ 変数の $2$ 次多項式の共通解をひとつ求めよ

グレブナー基底

## まとめ

## 参考文献
- Post-Quantum Cryptography
- 耐量子計算機暗号
- [量子コンピュータに耐性のある暗号技術の標準化動向：米国政府標準暗号について](https://www.imes.boj.or.jp/research/papers/japanese/19-J-04.pdf)
- [An efficient key recovery attack on SIDH](https://eprint.iacr.org/2022/975)
- [Constructing Digital Signatures from a One Way Function](https://www.microsoft.com/en-us/research/publication/constructing-digital-signatures-one-way-function/)


[1] Lamport, L. (1979). Constructing digital signatures from a one-way function (Vol. 238). Technical Report CSL-98, SRI International. [paper]

[2] Buchmann, J., Dahmen, E., & Hülsing, A. (2011, November). XMSS-a practical forward secure signature scheme based on minimal security assumptions. In International Workshop on Post-Quantum Cryptography (pp. 117–129). Springer, Berlin, Heidelberg.

[3] Hülsing, A., Butin, D., Gazdag, S., Rijneveld, J., & Mohaisen, A. "XMSS: eXtended Merkle Signature Scheme", RFC 8391, DOI 10.17487/RFC8391, May 2018, <https://www.rfc-editor.org/info/rfc8391>.

[4] Bernstein, D. J., Hopwood, D., Hülsing, A., Lange, T., Niederhagen, R., Papachristodoulou, L., … & Wilcox-O’Hearn, Z. (2015, April). SPHINCS: practical stateless hash-based signatures. In Annual international conference on the theory and applications of cryptographic techniques (pp. 368–397). Springer, Berlin, Heidelberg.

[5] Bernstein, D. J., Hülsing, A., Kölbl, S., Niederhagen, R., Rijneveld, J., & Schwabe, P. (2019, November). The SPHINCS+ signature framework. In Proceedings of the 2019 ACM SIGSAC conference on computer and communications security (pp. 2129–2146).

[6] Castelnovi, L., Martinelli, A., & Prest, T. (2018, April). Grafting trees: a fault attack against the SPHINCS framework. In International Conference on Post-Quantum Cryptography (pp. 165–184). Springer, Cham.

[7] https://github.com/kasperdi/SPHINCSPLUS-golang/blob/2e1d517cf3b3f17614ada1422e7c4cf0af3669d0/tweakable/sha256Tweak.go#L109