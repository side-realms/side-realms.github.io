---
title: "楕円曲線暗号の実装 3"
date: 2023-09-29T18:40:23+09:00
draft: false
---

点の演算を実装する（加算，倍加）

## 点の加法と倍加

2つの点 $P=(x_1, y_1)$ と $Q=(x_2, y_2)$ を考える（無限遠点でない）．
まず，P, Q を結ぶ直線を考える．この直線は傾き $\lambda$ を用いて以下のように表す

$$y = \lambda x + c$$

傾き $\lambda$ は以下のように定義される．

- $P \neq Q$
  - $\lambda = \frac{y_2-y_1}{x_2-x_1}$
- $P=Q$
  - $\lambda = \frac{3x_1^2 + a}{2y_1}$

楕円曲線上では， P+Q は 3 つめの交点 R の x 軸に対して反転した点として定義される．

```python
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

if __name__ == "__main__":
    p = 97
    a = 2
    b = 3
    curve = EllipticCurve(a, b, p)

    P = Point(17, 10, curve)
    Q = Point(95, 31, curve)

    R = P + Q
    print(f"{P} + {Q} = {R}")
    assert R == Point(1, 54, curve)

    # 同一点の倍加テスト
    R = P + P
    print(f"{P} + {P} = {R}")
    assert R == Point(32, 90, curve)

    # スカラー倍テスト（例: 7 * P）
    R = 7 * P
    print(f"7 * {P} = {R}")
    assert R == Point(49, 34, curve)

    print("ok")
```

結果は以下

```sh
Point(17, 10) + Point(95, 31) = Point(1, 54)
Point(17, 10) + Point(17, 10) = Point(32, 90)
7 * Point(17, 10) = Point(49, 34)
ok
```

よさそう
