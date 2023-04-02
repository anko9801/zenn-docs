---
title: "暗号プロトコル"
---

## ゼロ知識証明
ゼロ知識証明の性質
- 完全性（Completeness）
    - 証明者の主張が真であるならば、検証者は真であることが必ずわかること。
- 健全性（Soundness）
    - 証明者の主張が偽であれば、検証者はかなり高い確率でそれが偽であること見抜けること。
- ゼロ知識性（Zero Knowledge）
	- あらゆる場合において、検証者が証明者から何らかの知識（情報）を盗もうとしても、証明者の主張が真であること以上の知識は得られない


名前の通り対話型は有名な洞窟の例のように証明者と検証者がやりとりを繰り返し、証明者が本当に正しい情報を持っているかを確率的に検証する方です。

一方、非対話型のゼロ知識証明は証明者と検証者はやりとりをせずに証明することが可能です。対話を行う代わりに証明者と検証者の間に第三者を置き、CRSと呼ばれる事前に公開される情報を証明者と検証者に送ります。証明者はそのCRSを用いて正しい情報を持っているという証明を生成し、それを検証者に一回だけ送ります。受け取った検証者はそれを検証するだけで非対話なゼロ知識証明が実現できます。事前にCRSを生成することを一般的に信頼されたセットアップといい、第三者は証明者、検証者にとって信頼される存在となります。

zk-SNARKs
- Succinct（簡潔）
    - 証明のサイズがステートメントのサイズと比べて非常に小さい
- Non-interactive（非対話型）
    - 証明者と検証者の間で何度も対話をする必要がない
- ARgument
    - 証明者の計算能力には限りがある
- Knowledge
    - 証明者は、知識なしでは証明を生成することは不可能である。

[ZenGo-X/zk-paillier: A collection of Paillier cryptosystem zero knowledge proofs (github.com)](https://github.com/ZenGo-X/zk-paillier)

[zk-SNARKsの理論 (zenn.dev)](https://zenn.dev/kyosuke/articles/a1854b9be26c01df13eb)

## 署名
### Fiat-Shamir 変換

Fiat-Shamir変換は、証明システムを非対話型にするために使用される有名なスキームで、検証者がランダムに選択するチャレンジの値を（ランダムオラクルモデルとして）暗号学的ハッシュ関数を使って決定論的に導出することで、証明システムのプロトコルを非対話型にする。

チャレンジのハッシュ計算の際の入力に、証明するステートメントに関するすべての公開値（ランダムなコミットメント値を含む）を含める必要があるということ（↑の例だと入力は、G、V、P、UserID、OtherInfo）。

#### Frozen Heart

Frozen Heartは、Fiat-Shamir変換でチャレンジのハッシュ計算の際の入力に、証明するステートメントに関するすべての公開値（ランダムなコミットメント値を含む）を含める必要があるということ（↑の例だと入力は、G、V、P、UserID、OtherInfo）を遵守していないプロトコルや実装による脆弱性。

詳しいことはすべてここに

- [Fiat-Shamir変換の安全でない適用による脆弱性Frozen Heart - Develop with pleasure! (hatenablog.com)](https://techmedia-think.hatenablog.com/entry/2022/04/19/193400#:~:text=Fiat%2DShamir%E5%A4%89%E6%8F%9B%E3%81%A8%E3%81%AF,%E9%9D%9E%E5%AF%BE%E8%A9%B1%E5%9E%8B%E3%81%AB%E3%81%99%E3%82%8B%E3%80%82)
- [Coordinated disclosure of vulnerabilities affecting Girault, Bulletproofs, and PlonK | Trail of Bits Blog](https://blog.trailofbits.com/2022/04/13/part-1-coordinated-disclosure-of-vulnerabilities-affecting-girault-bulletproofs-and-plonk/)

### Schnorr Signature

非対話型ゼロ知識証明な署名の一種。

巡回群 $G$ 上で署名を行う。各パラメータは次のように定義する。

- 生成元 $g$
- 秘密鍵 $x$
- 公開鍵 $y = g^x$
- メッセージ $M$

#### 署名

乱数 $k$ を生成し、次の値を計算する。

$$
\begin{aligned}
r & = g^k & e & = H(r \| M) & s & = k - xe
\end{aligned}
$$

このとき $(s, e)$ を署名値として公開する。

#### 検証

公開されている値 $(g, y, M)$ と署名 $(s, e)$ を用いて次の値を計算する。

$$
\begin{aligned}
r' &= g^sy^e & e' &= H(r' \| M)
\end{aligned}
$$

ここで $e = e'$ となれば署名が有効であると検証されたことになる。

## 秘密分散 (Secret Sharing)
ある秘密情報を $n$ 個に分割して $t$ 個わかれば復元できる。
BLS署名

### Shamir's secret sharing
$t-1$ 次多項式 $f(x) = \sum_i a_ix^i \in \mathbb{F}_p[x]$ を用いて多項式上の $n$ 点 $(x_i, f(x_i))$ を生成する。

### Blakley's secret sharing

## 紛失通信 (Oblivious Transfer)
Goldwasser–Micali cryptosystem


## SSH

OpenSSH 9.0 ポスト量子暗号化時代への対応として、格子暗号系の「Streamlined NTRU Prime」と、楕円曲線暗号系の「x25519」からなるハイブリッド手法がデフォルトとなっている。

ed448
ed25519


Gnu Privacy Guard: GPG
master key
sub key
Pritty Good Privacy: PGP鍵
[GPGで自分用の秘密鍵を1つに統一する · JoeMPhilips](http://joemphilips.com/post/gpg_memo/)

物理的に奪われた場合など
パソコンは奪われて鍵は奪われてないとき有効
SoloKey, Nitrokey

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


## DH
共有鍵を作る為の操作である。共有鍵を作ることができれば共有鍵暗号を用いて通信できる。

1. AliceとBobが巡回群 $G$ とその生成元 $g$ を共有する。
2. AliceとBobはそれぞれ秘密鍵 $x_a, x_b$ を生成し、公開鍵 $y_a = g^{x_a}, y_b = g^{x_b}$ を公開する。
3. AliceとBobは自分の秘密鍵と相手の公開鍵を掛けると $s = g^{x_ax_b} = y_b^{x_a} = y_a^{x_b}$ となり、$s$ はAliceとBobのみが知る共有鍵となる。

ECDH だと $s$ の $x$ 座標をハッシュ化したものを共有鍵として使う。

## 暗号スイート
通信に使われるプロトコル/暗号の組み合わせのことです。

スイートはお菓子 Sweet ではなくて装備一式 Suite です。



## SSL/TLS

OCSP Stapling 証明書をハンドシェイク時に

## サービス
マイナンバーカード
コンテナイメージの署名 Sigstore

## 暗号の歴史

> 1974年, カリフォルニア大学バークレー校の計算機科学の学部学生で 22 歳になったばかりのマークル (Merkle) は, 指導教官の理解のない状態で孤立しながらも, 1つの画期的なアイデアにど到達した. それは、盗聴されてもいい通信路を使って2人の参加者が安全に秘密情報を共有する方法についてのアイデアであった。
> この問題は、インターネットの原型である米国国防省のプロジェクトアーパネット(ARPANET) が 1969 年に始まったことに触発されたものでありゅ。アーパネットは米国全土の様々な大学にあるコンピュータを通信回線で結んでコンピュータネットワークを構築するプロジェクトであった。ここでの基本方針はプロトコルだけを決めておき、あとは自主的に大学間でネットワークを構築する草の根ネットワークである。ネットワーク全体を管理する機能はなく、通信のセキュリティやプライバシーを守る機能もない。アーパネットは、1969 年に米国西海岸の 2 つの大学 (UCLA とスタンフォード大学) の間で最初のネットワークが構築され、1970 年代にはカリフォルニア大学バークレー校など西海岸の大学から全米の大学へと広がっていった。
> 1970 年代初頭にアーパネットプロジェクトを知り、マークルが気づいた問題はセキュリティ機能がないネットワークを使った安全な通信は可能であるかということであった。いま、事前に何ら共有情報を保持していない 2 人の利用者 A と B が安全でないネットワークで結ばれているとしよう。このネットワーク上で $A$ と $B$ が通信を行い、
1969年 ARPANET

共通鍵暗号
暗号化と復号で **同じ鍵** を使い、暗号化で使う鍵を **秘密** にする方式
ex.) DES, AES, ChaCha20-Poly1308

- DES
  - 最初に標準化された共通鍵暗号
- Triple-DES
  - DES の脆弱性を受けて改良したもの
- AES
  - 現代の共通鍵暗号の標準

公開鍵暗号
暗号化と復号で **別の鍵** を使い、暗号化で使う鍵を **公開** する方式
ex.) RSA暗号, Rabin暗号

- RSA暗号
- 楕円曲線暗号

DH鍵共有
公開鍵暗号を用いて共通鍵暗号を渡すプロトコル

TLS/SSL
HTTPS や SMTPS などで使用されている暗号プロトコル

1995年 SSL 3.0 が公開
- Netscape Communications 社が開発
1999年 TLS 1.0 の制定
- 国際標準化の為に SSL から名称変更
- 既に SSL という名称が定着していた為、SSL という名前も残る
SSL は TLS の前身で現在は使用されていない。

## 参考文献

- [Schnorr署名 ―― 30年の時を超えて注目を集める電子署名](https://blog.visvirial.com/articles/721)
- [RFC8235](https://datatracker.ietf.org/doc/html/rfc8235)
