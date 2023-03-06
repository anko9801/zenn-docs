---
title: "環境構築"
---

## Python
まずはバージョン管理マネージャー asdf を入れます。以下のリンクを読んで asdf のインストールをしてください。

https://asdf-vm.com/guide/getting-started.html

それでは Python

```shell
asdf plugin-add python
asdf list-all python
asdf install python 3.11.2
asdf install python 2.7.18
asdf global python 3.11.2 2.7.18
asdf reshim
```

```shell
$ python -V
Python 3.11.2
$ python3 -V
Python 3.11.2
$ python2 -V
Python 2.7.18
```

linter などは
- Pylance
- Ruff

```shell
pip install pycryptodome gmpy2 pwntools z3
```

## SageMath

```shell
sudo apt-get install sagemath
```

```python
# import
load('coppersmith.sage')

# 整数環, 剰余環, 有限体
ZZ, Zmod(N), GF(q)

R = GF(2^m)
R.fetch_int(12)
# 多項式
R = Zmod(N)
P. < x, y > = PolynomialRing(R)
f = x ^ 2 + 3*x + 3
f.small_roots()
p /= p.leading_coefficient()  # 最高次項 モニック化
p = p.monic()
ideal(f).groebner_basis()

crt(remain_list, modulo_list)
b.nth_root(3)

# 行列
M = matrix(QQ, [[1, 2], [3, 4]])
M.LLL()

A = matrix(Zn, [
    [_a, _b, _c, _d, _e],
    [1, 0, 0, 0, 0],
    [0, 1, 0, 0, 0],
    [0, 0, 1, 0, 0],
    [0, 0, 0, 1, 0],
])

b11, b12, b13, b14, b15 = map(int, (A ^ e)[4])

D, P = m.eigenmatrix_left()
P ^ (-1)*D*P == m

Q = diagonal_matrix(weights)

B *= Q
B = B.LLL()
B /= Q

# 素因数分解
set(factor(n))

# 楕円曲線
EllipticCurve()
```

FiniteField Galois Field
GF(p).primitive_element()

有限体。`Field = GF(13)` みたいな感じで作って、 `Field(9)` みたいな感じでインスタンス化する。 `Field(9) * 200 == 9`とかになる

[** PolynomialRing]
多項式環。整数上の多項式だったり有限体上の多項式だったするので、 `PR.<x> = PolynomialRing(ZZ)` って書いたり、 `PR.<x> = PolynomialRing(GF(20000))` って書いたりする

[** 剰余環(QuotientRing)]
多項式環を作ってから、剰余環を作る。
code:sage
 PR.<x> = PolynomialRing(GF(2))
 QR.<x> = PR.quotient(x^32 + x^26 + x^23 + x^22 + x^16 + x^12 + x^11 + x^10 + x^8 + x^7 + x^5 + x^4 + x^2 + x^ + 1)

[* Zp上の[quadratic residue]を求める]

square_root_mod_prime という関数がある。pの条件にいろいろなアルゴリズムを適用して効率的に平方剰余を求める。

特筆したいのは [$ p \mod 4 \equiv 3]のときと[$ p \mod 16 \equiv 1]のとき。
http://doc.sagemath.org/html/en/reference/finite_rings/sage/rings/finite_rings/integer_mod.html#sage.rings.finite_rings.integer_mod.square_root_mod_prime

code:modsqrt
 sage: p = next_prime(2**512)
 sage: p
 13407807929942597099574024998205846127479365820592393377723561443721764030073546976801874298166903427690031858186486050853753882811946569946433649006084171
 sage: y = getrandbits(500)
 sage: p % 4
 3
 sage: x = Mod(y*y, p)
 sage: x
 7427119115233134249944321954430159226671297124068251970205418931370131288785824942153709639322308405052842071730444238640093199607268137920681247102765246
 sage: from sage.rings.finite_rings.integer_mod import square_root_mod_prime
 sage: z = square_root_mod_prime(x, p=p)
 sage: z
 3073334465288394130574905125748160850388311389033237305733932921701934149185507919467233929798203728295688487377798377649820120555112278717850922475040
 sage: (z*z) % p == (y*y) % p
 True


$Z_{p^k}$ 上の quadratic residue を求める

square_root_mod_prime_power という関数がある
http://doc.sagemath.org/html/en/reference/finite_rings/sage/rings/finite_rings/integer_mod.html#sage.rings.finite_rings.integer_mod.square_root_mod_prime_power

code:modesqrtpower
 sage: p2 = p ** 2
 sage: y = getrandbits(768)
 sage: y
 1144776144634711213915815436342983403687527578588146564559560225448129227020193092625403552282662074892960581873756097335297322968816516814280840472692629659854617454374162992641915481205178971794270991714039594713106635977091268916L
 sage: p2
 179769313486231590772930519078902473361797697894230657273430081157732675805500963132708477322407536021120113879871393357658789768814416622492847430639476135548957384814430421380051950478165216024326171811091664303054708946946973913520433391685552272677504015463314271147575309020901508290327321376975136757241
 sage: x = Mod(y*y, p2)
 sage: x
 55874972658617988255940982575238219553607987562376547685566558253939769790491518790223658359140601723753235170613507653732154866442837634568025587703533930970495785300938083684580710036092263959017842810264745974397913767527728459719720752682069639257710085602432068747531285968922943668510633383794002373191
 sage: z = square_root_mod_prime_power(x, p=p, e=2)
 sage: z
 179769313486231590772930519078902473361797697894230657273430081157732675805499818356563842611193620205683770896467705830080201622249857062267399301412455942456331981262147759305158989896291459926990874488122847786240428106474281283860578774231178109684862099982109092175781038029187468695614214740998045488325
 sage: (z*z) % p2 == (y*y) % p2
 True

 整数変数連立方程式を解く時はsageよりもこっちのほうが早く出来たりする（なぜかSageは`assume(x, 'integer')`を無視するんだよね

```python
from sympy.solvers import solve
from sympy import symbols

x, y = symbols("x, y", integer=True)
solutions = solve([x + y - 12, x*y - 4060])
for s in solutions:
  print(s[x], s[y])
```

## 参考
- [quickref](https://wiki.sagemath.org/quickref?action=AttachFile&do=get&target=quickref.pdf)
- [quickref-linalg.pdf (shinshu-u.ac.jp)](http://math.shinshu-u.ac.jp/~nu/nora/sage/doc/refcard/quickref-linalg/200905/ja-utf8/quickref-linalg.pdf)
