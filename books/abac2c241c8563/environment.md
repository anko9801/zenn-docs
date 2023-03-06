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

整数変数連立方程式を解く時はsageよりもsympyのほうが早く出来たりする（なぜかSageは`assume(x, 'integer')`を無視するんだよね

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
