---
title: "ゼロ知識証明 1"
date: 2023-10-02T18:40:23+09:00
draft: false
---

## 知識の証明

主張「Prover はある知識 (秘密情報や解) をもつ」に関して，知識の正しさを証明したい．
単に主張された事柄が真であることを証明するのは対話証明だが，その知識の証明をすることになる．

## ZKP

ZKP は，その知識を Verifier に開示することなく知識を開示することなく(計算量的に再現不可能な形でのみ)証明したい．
ZKP は下記を満たす．

- 完全性: Prover が正しい知識をもっているなら，正当な手順で証明すれば Verifier はその知識が真実であることを疑いなく信頼できる
- 健全性: Prover は，Verifier を騙して偽の主張を真とみなすことはできない
- ゼロ知識: 主張が真か偽の他の情報を Verifer は受け取ることができない

## シグマプロトコル (Schnorr)

例えば知識 $w$ に関して，$g^w = y \mod p$ となるような $w$ をもっていることを証明したい．

1. 証明者 P は検証者 V に $a = g^r \mod p$ を送る
2. V は P にランダムな $e$ を送る
3. P は V に $z = r + ew \mod q$ を送る
4. V は $g^z = ay^e \mod p$ であることを確認する

このとき，公開されているのは大きな素数 $p$，位数 $q$ をもつ部分群の生成元 $g$，公開鍵 $y = g^w \mod p$．

### なぜこれが証明になるのか

- $z = r +ew \mod q$
- $a = g^r \mod p$
- $y = g^w \mod p$

$g^z = g^{r+ew} = g^r * g^{ew}$ だった．
このとき，$g^z = a * g^e \mod p$ となる．
証明者 P が本当に $z=r + ew$ を計算できているなら検証者側で $g^z$ が一致する．

## 実装

```python
#!/usr/bin/env python3
import random

p = 467
q = 233
g = 2

class Prover:
    def __init__(self, w):
        self.w = w
        self.r = None
        self.a = None

    def commit(self):
        self.r = random.randrange(q)
        self.a = pow(g, self.r, p)
        return self.a

    def respond(self, e):
        z = (self.r + e * self.w) % q
        return z


class Verifier:
    def __init__(self):
        pass

    def challenge(self):
        e = random.randrange(q)
        return e

    def verify(self, a, y, e, z):
        left = pow(g, z, p)
        right = (a * pow(y, e, p)) % p
        return (left == right)


def main():
    print("=== Schnorr Protocol Demonstration ===")
    w = random.randrange(q)
    y = pow(g, w, p)

    print(f"[Setup] p={p}, q={q}, g={g}")
    print(f"[Setup] Prover's secret w={w}")
    print(f"[Setup] y = g^w mod p = {y}")

    prover = Prover(w)
    verifier = Verifier()
    a = prover.commit()
    print(f"[Prover->Verifier] a = {a}")

    e = verifier.challenge()
    print(f"[Verifier->Prover] e = {e}")

    z = prover.respond(e)
    print(f"[Prover->Verifier] z = {z}")

    check = verifier.verify(a, y, e, z)
    if check:
        print("[Verifier] Proof is valid. Prover knows w.")
    else:
        print("[Verifier] Invalid proof. Prover might NOT know w.")


if __name__ == "__main__":
    main()
```
