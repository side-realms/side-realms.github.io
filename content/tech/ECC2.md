---
title: "楕円曲線暗号の実装 2"
date: 2023-09-28T18:40:23+09:00
math: true
draft: false
---

楕円曲線の定義を行う

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

if __name__ == "__main__":
    p = 17
    a = 2
    b = 2

    curve = EllipticCurve(a, b, p)
    try:
        point1 = Point(5, 1, curve)
        print(f"{point1} is on the curve")
    except ValueError as e:
        print(e)

    infinity_point = Point(infinity=True, curve=curve)
    print(f"無限遠点: {infinity_point}")

    # contains メソッドのテスト
    print("無限遠点は曲線上か？", curve.contain(infinity_point))
```

楕円曲線の定義では非特異性の確認を行う．
$$4a^3 + 27b^2 \mod p \neq 0$$

さらに，無限遠の点のフラグを inifinity と定義する．

おしまい
