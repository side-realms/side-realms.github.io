---
title: "楕円曲線暗号の実装 4"
date: 2023-09-30T18:40:23+09:00
math: true
draft: false
---

## ECDH

```python
import random

def mod_div(a, b, p):
    return mod_mul(a, mod_inv(b, p), p)

def generate_keypair(G, n):
    d = random.randint(1, n-1)
    Q = d * G
    return d, Q

if __name__ == "__main__":
    p = 97
    a = 2
    b = 3
    curve = EllipticCurve(a, b, p)

    G = Point(17, 10, curve)
    n = 19

    # userA
    dA, QA = generate_keypair(G, n)
    print(f"userA: dA={dA}, QA={QA}")

    # userB
    dB, QB = generate_keypair(G, n)
    print(f"userB: dB={dB}, QB={QB}")

    shared_keyA = dA * QB
    shared_keyB = dB * QA

    assert shared_keyA == shared_keyB
    print("ECDH success")
    print(f"shared_key={shared_keyA}")
```

結果は以下

```sh
/bin/python /home/kali/CTF/miniCTF/ECC/ecc.py
userA: dA=11, QA=Point(47, 18)
userB: dB=7, QB=Point(49, 34)
ECDH success
shared_key=Point(23, 24)
```

よさそう

## ECDSA

1. 鍵生成
   1. 秘密鍵 d
   2. 公開鍵 Q = dG (G は基準点)
   3. ECDLP 困難性により，公開鍵から秘密鍵は得られない
2. 署名生成
   1. 署名するメッセージ m に対して，ハッシュ関数(今回は SHA256) でハッシュ値 e を得る
   2. 乱数 k を選択する
   3. 点 R = kG を求める
   4. R の x 座標を r とする(r != 0)
   5. k の逆元 k^-1 mod n を求める $$s = k^{-1}(e+dr) \mod n$$
   6. 署名は (r, s) の組
3. 署名検証
   1. ハッシュ値 e を再計算する
   2. $w = s^{-1} \mod n$
   3. $u1 = ew \mod n$
   4. $u2 = rw \mod n$
   5. $P = u1G + u2Q$
   6. P の x 座標を v とし，v = r なら署名は正当

```python
import random
import hashlib

class EllipticCurve:
    def __init__(self, a, b, p):
        self.a = a
        self.b = b
        self.p = p
        
        if 4 * a**3 + 27 * b**2 == 0:
            raise Exception("singular curve")
        
    def contain(self, point):
        if(point.infinity):
            return True
        
        x, y = point.x, point.y
        return (y**2 - x**3 -self.a*x - self.b) % self.p == 0
    
class Point:
    def __init__(self, x=None, y=None, curve=None, infinity=False):
        self.x = x
        self.y = y
        self.curve = curve
        self.infinity = infinity

        if not self.infinity and curve is not None:
            if not curve.contain(self):
                raise Exception("not on curve")
    
    def __eq__(self, other):
        if self.infinity and other.infinity:
            return True
        if self.infinity or other.infinity:
            return False
        return (self.x, self.y, self.curve) == (other.x, other.y, other.curve)
    
    def __repr__(self):
        if self.infinity:
            return "Point(infinity)"
        return f"Point({self.x}, {self.y})"
    
    def __add__(self, other):
        if self.curve != other.curve:
            raise Exception("mismatch curve")
        if self.infinity:
            return other
        if other.infinity:
            return self
        
        p = self.curve.p

        if self.x == other.x and self.y != other.y:
            return Point(infinity=True, curve=self.curve)
        
        if self != other:
            lam = mod_div(other.y - self.y, other.x - self.x, p)
        else:
            if self.y == 0:
                return Point(infinity=True, curve=self.curve)
            lam  = mod_div(3 * self.x**2 + self.curve.a, 2 * self.y, p)

        x3 = (lam**2 - self.x - other.x) % p
        y3 = (lam * (self.x - x3) - self.y) % p
        return Point(x3, y3, self.curve)
    
    def __rmul__(self, k):
        result = Point(infinity=True, curve=self.curve)
        addend = self

        while k:
            if k&1:
                result += addend
            addend = addend + addend
            k >>= 1
        return result


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

def generate_keypair(G, n):
    d = random.randint(1, n-1)
    Q = d * G
    return d, Q

def sign(m, G, n, private_key):
    e = hashlib.sha256(m.encode()).hexdigest()
    e = int(e, 16) % n
    print(e)
    r, s = 0, 0

    while True:
        k = random.randint(1, n-1)
        R =  k*G
        r = R.x % n
        if r == 0:
            continue
        try:
            k_inv = mod_inv(k, n)
        except Exception:
            continue
        s = (k_inv * (e + private_key * r)) % n
        if s != 0:
            break
        
    return(r, s)

def verify(m, signature, G, n, public_key):
    r, s = signature
    e = hashlib.sha256(m.encode()).hexdigest()
    e = int(e, 16) % n
    print(e)
    try:
        w = mod_inv(s, n)
    except Exception:
        return False
    
    u1 = (e*w)%n
    u2 = (r*w)%n

    point = u1*G + u2*public_key
    v = point.x % n
    print(v)
    return v==r

def compute_order(G):
    P = G
    order = 1
    while not P.infinity:
        P = P + G
        order += 1
    return order


if __name__ == "__main__":
    p = 97
    a = 2
    b = 3
    curve = EllipticCurve(a, b, p)

    G = Point(17, 10, curve)
    n = compute_order(G)
    print(f"G の位数: {n}")

    private_key = random.randint(1, n - 1)
    public_key = private_key * G
    print(f"秘密鍵: {private_key}")
    print(f"公開鍵: {public_key}")
    
    message = "Hello, ECDSA!"
    signature = sign(message, G, n, private_key)
    print(f"署名: {signature}")
    
    valid = verify(message, signature, G, n, public_key)
    print(f"署名検証: {valid}")
```

おしまい
