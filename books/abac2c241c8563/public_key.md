---
title: "公開鍵暗号"
---

> **公開鍵暗号**
> 暗号化と復号で **別の鍵** を使い、暗号化で使う鍵を **公開** する方式
> ex.) RSA暗号, 楕円曲線暗号 など

- RSA 暗号
- 楕円曲線暗号

## 公開鍵暗号
暗号化と復号が別の鍵...と言われてもそんな暗号よくわからないと思うので最も有名な公開鍵暗号 RSA 暗号を紹介しましょう。

RSA 暗号とは $e$ 乗と $e^{-1}$ 乗を暗号化と復号の相互変換とした暗号です。

$$
\begin{aligned}
c &= m^e & \pmod N \\
m &= c^{e^{-1}} & \pmod N
\end{aligned}
$$

これを元に手順を書くと次のようになります。

> **RSA 暗号**
> - 鍵生成
>   1. 大きな素数 $p, q$ を生成して $N = pq$ と $\phi(N) = (p - 1)(q - 1)$ を計算する。
>   2. 整数 $e$ を決めて $d = e^{-1} \pmod{\phi(N)}$ を計算する。
>   3. $N, e$ を公開鍵として公開し、$p, q, \phi(N), d$ を秘密鍵とする。
> - 暗号化
>   1. 平文 $m$ に対して $c = m^e \bmod N$ と暗号化する。
> - 復号
>   1. 暗号文 $c$ に対して $m = c^d \bmod N$ と復号する。

これが RSA 暗号 (Rivest-Shamir-Adleman encryption) です。

ただ RSA 暗号は $N$ の素因数分解できると解かれてしまいます。

> **Prop.**
> 素因数分解が計算可能 (多項式時間で解ける) ならば RSA 暗号も計算可能である。

**Proof.**
素因数分解が解けるから $N = pq$ となる $p, q$ がわかる。これより $\phi(N) = (p-1)(q-1)$, $d = e^{-1} \pmod{\phi(N)}$, $m = c^d \pmod{N}$ は計算可能である。 $\Box$

逆に素因数分解出来なければ RSA 暗号が解けないことの証明はありますが、長くなります。一般に計算困難性を示すのは難しい問題です。

> **Prop.**
> 素因数分解が計算困難 (多項式時間で解けない) ならば RSA-OAEP は IND-CCA2 である。

https://eprint.iacr.org/2000/060.pdf

素因数分解の計算困難性を仮定すれば、解読できないことがわかりました。

現在、公開鍵暗号は **素因数分解問題** か **離散対数問題** をベースにした暗号がよく使われています。

> **素因数分解問題**
> ある整数 $N$ が素数 $p, q$ を用いて $N = pq$ の素因数分解が難しい

> **離散対数問題**
>

Trap-door function

### DH 鍵共有
共有鍵を作る為の操作である。共有鍵を作ることができれば共有鍵暗号を用いて通信できる。

1. AliceとBobが巡回群 $G$ とその生成元 $g$ を共有する。
2. AliceとBobはそれぞれ秘密鍵 $x_a, x_b$ を生成し、公開鍵 $y_a = g^{x_a}, y_b = g^{x_b}$ を公開する。
3. AliceとBobは自分の秘密鍵と相手の公開鍵を掛けると $s = g^{x_ax_b} = y_b^{x_a} = y_a^{x_b}$ となり、$s$ はAliceとBobのみが知る共有鍵となる。

ECDH だと $s$ の $x$ 座標をハッシュ化したものを共有鍵として使う。

### 電子署名
署名の方法にはいろいろありますが、暗号のレイヤーを抽象化すると

> **DSA**
> - 鍵生成
>   1. $p = 2q + 1$ となる素数 $p, q$ を生成すると $\mathbb{F}_p^\times \cong \mathbb{F}_q\times\mathbb{F}_2$ となる。
>   2. $g\in\mathbb{F}_p$ と $x\in\mathbb{F}_q$ を生成して $y = g^x \bmod p$ を計算する。
> - 署名
>   1. ランダムに $k$ を生成する。
>   2. $r = (g^k\bmod p)\bmod q$ と $s = k^{-1}(H(m) + xr) \bmod q$ を署名として公開する。
> - 検証
>   1. $v = (g^{s^{-1}H(m)}y^{s^{-1}r} \bmod p)\bmod q$ を計算して $r = v$ なら正当な署名となる。

:::message
**練習問題**
$k$ がランダムであるかはかなり重要です。
Easy: $k = m$ かつ $p, q$ 固定のとき $m$ を取り出すことができる。どうやって？ (WaniCTF DSA?)
:::

> **Fiat-Shamir 変換**
> 疑似乱数 $r$ を生成し, $e := H(g^r)$
> $y := r - se$ $(x, y)$
> $x' := g^yp^{H(x)}$
> $b := H(com)$

証明システムを非対話型にするために使用される有名なスキームで、検証者がランダムに選択するチャレンジの値を（ランダムオラクルモデルとして）暗号学的ハッシュ関数を使って決定論的に導出することで、証明システムのプロトコルを非対話型にする。
チャレンジのハッシュ計算の際の入力に、証明するステートメントに関するすべての公開値（ランダムなコミットメント値を含む）を含める必要があるということ（↑の例だと入力は、G、V、P、UserID、OtherInfo）。

#### Frozen Heart

Frozen Heartは、Fiat-Shamir変換でチャレンジのハッシュ計算の際の入力に、証明するステートメントに関するすべての公開値（ランダムなコミットメント値を含む）を含める必要があるということ（↑の例だと入力は、G、V、P、UserID、OtherInfo）を遵守していないプロトコルや実装による脆弱性。

詳しいことはすべてここに

- [Fiat-Shamir変換の安全でない適用による脆弱性Frozen Heart - Develop with pleasure! (hatenablog.com)](https://techmedia-think.hatenablog.com/entry/2022/04/19/193400#:~:text=Fiat%2DShamir%E5%A4%89%E6%8F%9B%E3%81%A8%E3%81%AF,%E9%9D%9E%E5%AF%BE%E8%A9%B1%E5%9E%8B%E3%81%AB%E3%81%99%E3%82%8B%E3%80%82)
- [Coordinated disclosure of vulnerabilities affecting Girault, Bulletproofs, and PlonK | Trail of Bits Blog](https://blog.trailofbits.com/2022/04/13/part-1-coordinated-disclosure-of-vulnerabilities-affecting-girault-bulletproofs-and-plonk/)

> **Schnorr Signature**
> 非対話型ゼロ知識証明な署名の一種。
>
> 1. 鍵生成 : 巡回群 $G$ 上で生成元 $g\in G$ と秘密鍵 $x\in\mathbb{N}$ を用いて公開鍵 $y = g^x$ を生成する。
> 2. 署名 : 疑似乱数 $k$ を生成し、署名したいメッセージ $M$ を用いて $e = H(g^k \| M), s = k - xe$ を計算して $(s, e)$ を署名値として公開する。
> 3. 検証 : $e' = H(g^sy^e \| M)$ を計算し、 $e = e'$ となれば署名が有効であると検証されたことになる。

位置情報共有
Intel HEXL
> **BLS署名**
> 1. 鍵生成 : 0以上r未満の乱数sを一つ取り秘密鍵とします。公開鍵はsQです。
> 2. 署名 : メッセージmに対してそのハッシュ値H(m)をとり、秘密鍵s倍して署名s H(m)を作ります。
> 3. 検証 : メッセージmと署名σをもらった人は自分でハッシュ値H(m)を計算し公開鍵sQを使って

