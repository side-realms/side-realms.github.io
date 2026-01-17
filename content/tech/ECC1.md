---
title: "楕円曲線暗号の実装 1"
date: 2023-09-27T18:40:23+09:00
math: true
draft: false
---
楕円曲線暗号を理解するために実装を行う

## 離散対数問題(DLP), DH 問題(DHP)

Alice と Bob が DH 鍵共有をするのは以下のような手順で行われる

1. あらかじめ使用する整数 $p, g$ を決める
   1. $p = 65537, g = 3$
2. Alice と Bob は秘密の値 $a, b$ をランダムに決定する
3. Alice: $A = g^a \mod p$ を計算し，Bob に渡す
4. Bob: $B = g^b \mod p$ を計算し，Alice に渡す
5. Alice は B を用いて $s1 = B^a \mod p = g^ab \mod p$ を計算する
6. Bob は A を用いて $s2 = A^b \mod p = g^ab \mod p$ を計算する
7. $s1 = s2$ なので鍵共有が行われる

この方法では，攻撃者は素数 $p$, 整数 $g, A, B$ を入手できる．
このとき，鍵は $s1, s2$ になるので，攻撃者は $a, b$ の値が分かれば鍵を知ることができる．
まず離散対数問題 (DLP) では， $A = g^a \mod p$ において $A, g, p$ が分かっていても $a$ を知ることが難しい．
また，DH 問題 (DHP) では，$g, p, A, B$ から $s1, s2$ の値を知ることが難しい．

## 楕円曲線暗号

楕円曲線は $y^2 = x^3 + ax + b$ と表される．
この楕円曲線上にある点同士での足し算ができる．

- ECDLP: 点 P, aP が与えられたときに a を求める問題
- ECDHP: 点 P, aP, bP が与えられたときに abP を求める問題

### ECDH 鍵共有

- Alice は乱数 a を選び，aP を Bob に送る
- Bob は乱数 b を選び， bP を Alice に送る
- Alice は abP を計算する
- Bob は abP を計算する

### 有限体演算の実装

楕円曲線上の計算は，通常有限体 \mathbb{F}_p (mod p による) 上で行われる．
まずはこの有限体上での基本的な演算を実装する必要がある．

```python
def mod_add(a, b, p):
    return (a + b) % p

def mod_sub(a, b, p):
    return (a - b) % p

def mod_mul(a, b, p):
    return (a * b) % p

def mod_inv(a, p):
    if a == 0:
        raise Exception("zero inverse")
    return (pow(a, p-2, p))

def mod_div(a, b, p):
    return mod_mul(a, mod_inv(b, p), p)
```

足し算，引き算，掛け算はそのまま実装すればいいが，割り算は難しい
有限体なので，普通に 1/x としても整数にならない．
ここで，$y = 1/x$ とすると，$y * x \mod p = 1$ といえる．
このような y を探す．

フェルマーの小定理を考えると，$x^{p-1} = 1$ となる．
$x(x^{p-2} = 1)$ なので，$x^{p-2} = y$ とすると，$xy = 1(\mod p)$ となる．
これを実装すればおけ

おしまい
