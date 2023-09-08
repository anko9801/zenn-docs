---
title: "【CTF 探訪記】crypto 入門"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CTF", "crypto"]
published: false
---

CTF とは技術における高度な知識とパズルを解く発想力の両方を問う総合格闘技です！中でも Crypto は暗号という一見フシギなシステムを攻撃することで現代では欠かせなくなった暗号は本当に安全なのかを考えさせてくれる分野です。

ただ Crypto の知識を得る手段は実際に CTF に出て writeup を読んだり、大量の資料を読むことでしかがなかったと思います。そこで Crypto における体系的に解説する資料を作りました！私自身これが欲しくて助かりました！(?)

### 対象読者
これは辞書的なものなので全てを読むべきとは思っていません。対象者によって読むといいところも変わってきます。
- 暗号自体を軽く調べたい方
  - 最初: Introduction と最後: 暗号プロトコルを読むと幸せになれます。
- Crypto を解けるようになりたい方
  - 前半を読むと幸せになれます。
- 数学が好きな方や暗号に強い興味がある方
  - 全部読もう！

### 前提知識
- (必須) 線形代数の基礎的な知識
- (必須) Python の基礎的な構文
- (あるとよい) 群環体などの代数学
- (より理解したい方向け) 代数幾何学や楕円曲線論

### 目標
- CTF の Crypto で難しい問題が解けるようになる
- 最新の暗号研究を追うことができるようになる

### 謝辞
うしがぃさん、資料を提供していただきました！ありがとうございます。

## 暗号とは

まず暗号というのは自分と相手以外の人に内容が伝わらないように文章を送るための技術・理論の総称です。
ログインした状態で通信するときや個人情報が大量に含まれている可能性があります。
通信時に悪いことが出来ないとは具体的に何でしょうか？

- 機密性 (Confidentiality)
内容に関するあらゆる情報を得られない
- 完全性 (Integrity)
改ざんされない
- 真正性 (Authenticity)
なりすまされない

逆に「解読、改ざん、なりすまし」が出来なければ、通信において悪いことが出来ないということが証明されます。

### 解読されないとは
暗号というのは元来「**暗号化/復号は簡単でも解読は難しい**」という非対称性があります。

例えば換え字暗号は「文字置換は簡単でもどう置換したのかを調べるのは難しい」という非対称性、 RSA 暗号であれば「掛け算は簡単でも素因数分解は難しい」という非対称性、AES であれば「シャッフルは簡単でも元に戻すのは難しい」という非対称性があります。

その難しさこそが暗号の機密性です。

そしてその難しさは **カギ** が担っています。カギは暗号化/復号するときに必要なデータで、暗号化プログラムが全世界に公開されている現代暗号ではカギのない暗号だと誰もが復号できてしまいます。

このカギの取り扱いには慎重にならなければいけません。カギの取り扱い方によって暗号を大きく 2 つに分けられます。1 つは共通鍵暗号です。

> **共通鍵暗号**
> 暗号化と復号で **同じ鍵** を使い、暗号化で使う鍵を **秘密** にする方式
> ex.) AES, DES, ChaCha20 など

共通鍵暗号は高速に暗号化できる反面、最初に鍵を共有しなければならず、共有しているところを攻撃者に盗聴される危険があって安全に通信できません。そこで公開鍵暗号です。

> **公開鍵暗号**
> 暗号化と復号で **別の鍵** を使い、暗号化で使う鍵を **公開** する方式
> ex.) RSA暗号, 楕円曲線暗号 など

公開鍵暗号は暗号化と復号で別の鍵を用いるので、暗号鍵を公開しても攻撃者は復号することができません。このためある人以外は暗号化できてその人だけ復号できるという状態となります。その為、共通鍵を渡すときや秘密を安全に通信したいときに重宝します。

この 2 つの暗号を使い分けることで高速で安全な通信路を構築することができます。

このようにして機密性のある通信ができます。

### なりすまされないとは
信頼できない相手に対してどのように正当性を担保するのかという問題はインターネットが普及した現代では非常によくあります。

皆さんがよく使う **パスワード認証** はなりすましを防ぐ手段の 1 つです。
合言葉であるパスワードを知っている人しかログインできないようにすれば、ユーザーの正当性は大体担保できます。ただしこれはユーザーにとって分かりやすいシステムな反面、パスワードが使い回されていたり、簡潔なものに設定されているなどしていると、アカウントが乗っ取られてしまうことがあります。ですのでちゃんとサービス毎にそれぞれ違う複雑なパスワードを覚えて設定する...なんてことは私には無理なのでブラウザが生成した乱数値か FIDO を使うのがいいでしょう。

パスワード認証の他に **電子署名** というのもあります。
これは認証局から渡された電子証明書に署名した人だけが認証され、そうではない攻撃者は正当な署名を作ることができないようなシステムです。

1. ユーザーは信頼された第三者機関である認証局 (CA) に許可を得て、所有者情報や有効期限が書かれた電子証明書を発行してもらいます。
2. ユーザーは公開鍵暗号の秘密鍵を用いて署名し、公開鍵と署名をホストに渡します。
3. ホストは署名を公開鍵で復号したものを CA で検証してもらってユーザーの正当性を保証します。

逆に攻撃者がその人になりすます為には証明書を後から署名しなければなりませんが、ホストに保持された公開鍵に対応する秘密鍵を持っていないので署名は不可能です。

そもそもどうやって攻撃者の証明書を作るんだ？

![](/images/digital_signiture.png)

SSH の公開鍵認証ってしたことありますか？鍵のペアを生成して、公開鍵を `authorized_keys` に書いて `ssh` コマンド叩いて `fingerprint` がなんとかかんとかとか言われたりして接続できるやつです。あれは電子署名の簡易版で、ユーザー自身で証明書を発行/署名して、ホストだけが持つ公開鍵で検証してもらうという認証をしています。

TODO: GPG の話

最近この手の話題はよくあって、それらの単語と結びつけてざっと理解してみましょう。詳しくは traP とかいうサークルが解説記事いっぱい出してるんじゃないですかね。

- MFA; Multi-Factor Authentication / 2FA
パスワードに加えて、デバイス認証やEメール、生体認証など、他の認証方法も用いた認証。
- OAuth 2.0 / OpenID Connect / SAML
他サービスへの認可と認証を行う為の規格。「Google でサインインする」や「Twitter と連携する」とかいうあれが認可認証です。
- FIDO / WebAuthn / [passkeys](https://blog.agektmr.com/2022/12/passkey.html)
公開鍵認証をブラウザなどで行う為の規格。パスワードなしでログインできるようにする。SoloKey, NitroKey など
- マイナンバーカード
電子証明書を搭載した IC チップ付き身分証。身分証に目が行きがちですが電子署名で確実な本人確認が可能なことが良いところ。
- セキュアブート
カーネルやブートマネージャーなどのファームウェアの電子署名を管理することで OS が正当なものか検知する仕組み。Linux をデュアルブートするときとかにお世話になりますね。
- TEE; Trusted Execution Envrionment
厳密にセキュアな領域を広げていくシステム。

このように真正性は保証されます。

### 改ざんされないとは
まず先に断っておくのですが、本質的に改ざんさせないということは **不可能** です。
暗号化された通信をしていて攻撃者は暗号文の内容がわからなくとも、適当に書き換えて送りつければ、平文も書き換わりますからね。何故かというと、暗号は復号可能性により暗号文と平文は一対一に対応しているので平文が変われば暗号文も変わるし、逆も同様です。

それでは改ざんされることは許すとして、改ざんされたことを検知できれば良いですよね。検知する手法としてはいくつかありますが、最も基本的な方法はハッシュ関数という関数を使う方法です。

> **ハッシュ関数**
> 一方向にしか計算できない関数、つまりあるデータのハッシュ値から元のデータが取り出せられないような関数です。例えば SHA256, MD5 などがあります。
TODO: 本当に一方向性は必要？

これを使って以下のようなことをします。

1. 暗号化された通信上でファイルとそのハッシュ値を送る
2. 送られてきたファイルのハッシュ値と送られてきたハッシュ値が一致するかどうか検査する

![](/images/hash2.png)

こうすることでデータがランダムに書き換わると送られてきたハッシュ値とは異なるハッシュ値となり、改ざんを検知できるという訳です。

しかし、これだけだと以前に送った暗号文に入れ替えるリプレイ攻撃に弱いので、ハッシュ関数をベースに疑似乱数 Nonce を取り入れた検知方法が主流です。

- MAC; Message Authentication Code
暗号学的に強い一方向性の関数を用いてメッセージ改ざん検知する関数。HMAC, AES-CMAC など。
- AEAD; Authenticated Encryption with Associated Data
暗号化とメッセージ改ざん検知を同時に行う暗号方式。AES-GCM, ChaCha20-Poly1308 など。

このように完全性が保証されます。

## あそんでみよう
実際の CTF ではよくわからない暗号やエンコーディング、古典暗号が出題されることがあるのですが、大抵ググるか CyberChef に投げれば解けてしまいます。

https://gchq.github.io/CyberChef/

換え字式暗号なら quipquip やも使います。

- [quipquip](https://quipqiup.com/)
- [](https://www.guballa.de/substitution-solver)
https://www.dcode.fr/

:::message
**練習問題**
CyberChef を使って何が書いてあるか当ててください。
`Li0uLiUyMC4lMjAtJTIwLi0tLS0uJTIwLi4uJTBBLi0uJTIwLiUyMC4tJTIwLS4uJTBBLi0uLS4tJTIwLS4uLi4tJTIwLi0tLSUyMC0uLS4lMjAuLi4tLSUyMC0uLSUyMC0tLiUyMC0tLS0uJTIwLi0lMjAtLi4uJTIwLi0lMjAtLS0tLiUyMC0tLS0tJTIwLS4tLiUyMC0tLSUyMC4tJTIwLS0tLS0lMjAuLi4uLiUyMC4uLS0tJTIwLS4uLi4lMjAuLS4uJTIwLi4uLiUyMC0uLi4uJTIwLi4uJTIwLi0lMjAuJTIwLS4uLi4lMjAuLi4tJTIwLSUyMC0tLSUyMC0uLi4uJTIwLi0tLiUyMC4uJTIwLi4tJTIwLS0tJTIwLi0=`
:::

## まとめ
暗号というのは何を守るのか、どうして必要なのかをさくっと解説しました。

これを聞いて次は中身がどうなっているのか気になりませんか？そして中身が分かってくると脆弱な点が見えてきて攻撃ができるようになってきます。

## Python
まずはバージョン管理マネージャー asdf を入れます。以下のリンクを読んで asdf のインストールをしてください。

https://asdf-vm.com/guide/getting-started.html

asdf を入れたら Python を入れます。

```shell
$ asdf plugin-add python
$ asdf list-all python
$ asdf install python 3.11.2
$ asdf install python 2.7.18
$ asdf global python 3.11.2 2.7.18
$ asdf reshim
$ python -V
Python 3.11.2
$ python3 -V
Python 3.11.2
$ python2 -V
Python 2.7.18
```

これで Python が入りました。:tada:
私が入れてるエコシステムはこれくらいです。
- Pylance (language support)
- Ruff (linter)

今回使うパッケージはこんな感じです。

```shell
$ pip install pycryptodome gmpy2 pwntools z3
```

## SageMath
SageMath とは

```shell
$ sudo apt-get install sagemath
```

チートシート

```python
# import
load('coppersmith.sage')

# 環
ZZ # 整数環 Z
Zmod(N) # 剰余環 Z/NZ
QQ # 有理数体 Q
RR # 実数体 R
CC # 複素数体 C

# 有限体 FiniteField Galois Field
GF(q)
GF(p).primitive_element()
F = GF(13)
F(9) * 200 == 9
R = GF(2^m)
R.fetch_int(12)

# 多項式 Polynomial Ring
R = Zmod(N)
P.<x, y> = PolynomialRing(R)
f = x^2 + 3*x + 3
f.small_roots()
p /= p.leading_coefficient()  # 最高次項 モニック化
p = p.monic()
ideal(f).groebner_basis()

# 剰余環 Quotient Ring
PR.<x> = PolynomialRing(GF(2))
QR.<x> = PR.quotient(x^32 + x^26 + x^23 + x^22 + x^16 + x^12 + x^11 + x^10 + x^8 + x^7 + x^5 + x^4 + x^2 + x^ + 1)


crt(remain_list, modulo_list)
b.nth_root(3)

# 行列
M = matrix(QQ, [
    [1, 2],
    [3, 4]
])
M.LLL()

A = matrix(Zn, [
    [a, b, c, d, e],
    [1, 0, 0, 0, 0],
    [0, 1, 0, 0, 0],
    [0, 0, 1, 0, 0],
    [0, 0, 0, 1, 0],
])

b11, b12, b13, b14, b15 = map(int, (A ^ e)[4])

D, P = m.eigenmatrix_left()
P^(-1)*D*P == m

Q = diagonal_matrix(weights)

B *= Q
B = B.LLL()
B /= Q

# 素因数分解
set(factor(n))

# 楕円曲線
EllipticCurve()
```


## SAT と SMT
命題論理式

> **連言標準形 (CNF; Conjunctive Normal Form)**
> 変数 $A$ に関して $A, \lnot A$ をリテラルと定義する。このとき連言標準形とはリテラル $l_{i,j}$ に対して次のように書ける命題論理式である。
>
> $$
\bigwedge_i\bigvee_j l_{i,j}
$$
>
> このとき $\bigvee_j l_{i,j}$ を節という。

例えば次のような論理式は CNF です。

$$
\begin{aligned}
& A\land B \\
& \lnot A\land(B\lor C) \\
& (A\lor B)\land(\lnot B\lor C\lor\lnot D)\land(D\lor\lnot E) \\
& (\lnot B\lor C).
\end{aligned}
$$

次のような論理式は CNF ではありません。

$$
\begin{aligned}
& \lnot(B\lor C) \\
& (A\land B)\lor C \\
& A\land(B\lor(D\land E)).
\end{aligned}
$$

二重否定の除去、ド・モルガンの法則、分配法則などを用いて CNF に変換することができます。実際、次のアルゴリズムが知られています。

> **Tseitin Encoding**
> 任意の命題論理式を CNF に変換するアルゴリズムが存在する。

これより上で示した CNF でない論理式を CNF に変換でき、次のようになります。

$$
\begin{aligned}
& \lnot B\land\lnot C \\
& (A\lor C)\land(B\lor C) \\
& A\land(B\lor D)\land(B\lor E).
\end{aligned}
$$

CNF のような命題論理式について変数の値がどのようなときに真になるかを考えます。

> **充足可能性問題 (SAT; SATisfiability problem)**
> ある命題論理式が与えられたとき、それを満たす変数の値を定める問題を充足可能性問題 SAT という。

SAT は NP 完全です。

$$
(A\land\lnot B\land\lnot C)\lor(B\land C\land\lnot D)\land(\lnot B\lor\lnot C)
$$

これは $(A,B,C,D)=(\top,\bot,\bot,\top)$ のときのみ上の論理式を満たします。

SAT を解くアルゴリズムを SAT ソルバといい、CNF に変換すると何かと解析しやすいので CNF に絞って考えます。

まずは単純に全探索を考えます。リテラルが 1 つしかない節を単位節と呼ぶとして次の操作を繰り返します。

1. 単位節があればその変数の値は確定する
2. 単位節がなければ変数のどれか1つを深さ優先探索する
3. コンフリクトしたなら失敗、コンフリクトなしに変数を全て割り当てられたら成功とする

これは DPLL (Davis Putnam Logemann Loveland) アルゴリズムと呼ばれています。

この全探索に加え、コンフリクトしたときにその探索状態だと失敗することを条件に入れます。この操作を節学習と呼び、節学習されて条件が多くなると、探索をしなくて済むようになり高速化します。これを CDCL (Constrait-Driven Clause Learning) アルゴリズムと呼び、これにより SAT ソルバは画期的に速くなります。

この資料がわかりやすいなと思っています。

https://www.youtube.com/watch?v=d76e4hV1iJY&t=760s

実装は節 $x_k$, $\lnot x_k$ はそれぞれ $2k$, $2k+1$ と表して合計で $2n$ 個の配列が必要になり、各節はリテラルのポインタを格納するリンクリストを持ちます。各変数は探索する変数の優先順位を決定させるスコアを持ち、$i$ 回目にコンフリクト時にコンフリクトした変数のスコアに $\rho^{-i}$ だけ足されます。(ex. $\rho=0.95$) つまり後の方になればなるほど増加するスピードが速くなり、古いコンフリクトは無視されるようになります。

このように命題論理式は SAT ソルバを用いて解くことができます。SAT ソルバだけでもかなり有用なツールですが、ここではさらに発展させて SAT について実数、整数、リスト、配列、文字列など様々なデータ構造を含む、より複雑な数式に一般化した問題を考えます。

> **SMT; Satisfiability Modulo Theories**
> ある数式が与えられたとき、それを満たす変数の値を定める問題を SMT という。

SMT ソルバはいろいろな応用先があり、例えば次のようなものがあります。

- 数式に帰着できるすべての問題のソルバ
かなり凄い、ネットワークの
- 形式手法 / モデル検査
  - TLA+
  - seL4
- シンボリック実行エンジン
- 自動定理証明支援系
- Differential Cryptoanalysis

SMT ソルバのアルゴリズムには EUF; Equality logic with Uninterpreted Functions や BitVector などがありますが、今回は BitVector のみを紹介します。

> **BitVector**
> $n$ ビットの四則演算などの演算について、それぞれのビットに関する論理式を立てることで SMT を SAT に置き換えることができる。

使われる演算としては $a\land b$, $\lnot a$, $a < b$, $a = b$, $a[i]$, $\sim a$, $a\mathop{\|}b$, $a\mathop{\&}b$, $a \oplus b$, $a \ll b$, $a \gg b$, $a + b$, $a - b$, $a \times b$, $a / b$, $\mathrm{ext}(a)$, $a\circ b$, $a[b:c]$, $c?a:b$ などなので、これらを論理式に落とせることは CPU を自作したことがある人ならわかると思います。

例えば 1 ビットの $a + b$ なら $a + b = a\oplus b = (a\land\lnot b)\lor(\lnot a\land b)$ と置き換えられます。

デファクトスタンダードな SMT ソルバに Z3 があります。
Z3 を使える言語は C, C++, JavaScript, Python, Rust などなど、今回は Python でやっていきます。

```python
bool, int, float, float32, double, real, string, array, set, enumeration, bitvector
BitVec()
IntVector()
Real()
from z3 import Solver, Context, RecFunction, RecAddDefinition, IntSort, Int, If, simplify

ctx = Context()
f = RecFunction("f", IntSort(ctx), IntSort(ctx))
x = Int("x", ctx)
RecAddDefinition(f, x, If(x <= 2, 1, f(x-1) + f(x-2)))

solver = Solver(ctx=ctx)
solver.add(f(x) == 10946)

print(solver.check())
print(solver.model())
```


## 参考
- [quickref](https://wiki.sagemath.org/quickref?action=AttachFile&do=get&target=quickref.pdf)
- [quickref-linalg.pdf (shinshu-u.ac.jp)](http://math.shinshu-u.ac.jp/~nu/nora/sage/doc/refcard/quickref-linalg/200905/ja-utf8/quickref-linalg.pdf)
- [SAT/SMTソルバの仕組み](https://www.slideshare.net/sakai/satsmt)
- [ミュンヘン工科大学の夏学期の自動推論に関する授業](https://www21.in.tum.de/teaching/sar/SS20/)
- [SATソルバ・SMTソルバの技術と応用](https://www.jstage.jst.go.jp/article/jssst/27/3/27_3_3_24/_pdf)
- [TokyoWesterns の z3 解説](https://wiki.mma.club.uec.ac.jp/CTF/Toolkit/z3py)

SAT / SMT ソルバの入力形式は DIMACS / SMT-LIBv2 を用います。

この資料は CC0 ライセンスです。