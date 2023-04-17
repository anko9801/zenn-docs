---
title: "公開鍵暗号"
---

> **公開鍵暗号**
> 暗号化と復号で **別の鍵** を使い、暗号化で使う鍵を **公開** する方式
> ex.) RSA暗号, 楕円曲線暗号 など

- RSA 暗号
- 楕円曲線暗号

## 公開鍵暗号

落とし戸付き一方向性関数

最も有名な公開鍵暗号である RSA 暗号を紹介しましょう。

簡単に言うと、平文 $m$ の $e$ 乗を暗号化、暗号文 $c$ の $e^{-1}$ 乗を復号するとした暗号が RSA 暗号です。

$$
\begin{aligned}
c &= m^e & \pmod N \\
m &= c^d & \pmod N
\end{aligned}
$$

これを厳密に書くと次のようになります。

> **RSA 暗号**
> 1. 鍵生成: 大きな素数 $p, q$ を生成して $N = pq$ を計算し、整数 $e$ と $N$ を公開鍵として公開し、$p, q$ を秘密鍵とする。
> 2. 暗号化: 平文 $m$ に対して暗号文 $c$ を $c = m^e \bmod N$ として暗号化する。
> 3. 復号: 暗号文 $c$ に対して秘密鍵を用いて $\phi(N) = (p - 1)(q - 1)$ と $d = e^{-1} \pmod{\phi(N)}$ とおくと $m = c^d \bmod N$ のように復号する。

素因数分解の計算困難性を仮定すれば、復号は難しいと証明されています。これを用いた暗号をRSA暗号 (Rivest-Shamir-Adleman encryption) と呼びます。

## DH 鍵共有
共有鍵を作る為の操作である。共有鍵を作ることができれば共有鍵暗号を用いて通信できる。

1. AliceとBobが巡回群 $G$ とその生成元 $g$ を共有する。
2. AliceとBobはそれぞれ秘密鍵 $x_a, x_b$ を生成し、公開鍵 $y_a = g^{x_a}, y_b = g^{x_b}$ を公開する。
3. AliceとBobは自分の秘密鍵と相手の公開鍵を掛けると $s = g^{x_ax_b} = y_b^{x_a} = y_a^{x_b}$ となり、$s$ はAliceとBobのみが知る共有鍵となる。

ECDH だと $s$ の $x$ 座標をハッシュ化したものを共有鍵として使う。

## 電子署名
署名の方法にはいろいろありますが、暗号のレイヤーを抽象化すると

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

