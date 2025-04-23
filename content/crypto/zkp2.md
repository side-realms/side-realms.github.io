---
title: "ゼロ知識証明 2"
date: 2023-10-03T01:34:21+09:00
draft: false
---

shnnor protocol において，同じ a に対して複数のチャレンジ e を成功させると w が取り出せることがある

```python
import socket
import json
import random
from Crypto.Util.number import inverse, long_to_bytes

p = xxx
q = yyy

def readline_json(sockfile):
    line = sockfile.readline()
    if not line:
        return None
    line = line.strip()
    if not line:
        return None
    try:
        return json.loads(line)
    except json.JSONDecodeError:
        return None

def main():
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.connect((HOST, PORT))
        sockfile = s.makefile("r")

        while True:
            data = readline_json(sockfile)
            if data is not None:
                break
        a = data["a"]
        y = data["y"]
        print("[Server] a =", a, "y =", y)

        e = random.getrandbits(511)
        print("[Client] Send e =", e)
        s.sendall((json.dumps({"e": e}) + "\n").encode())

        while True:
            data = readline_json(sockfile)
            if data is not None and "z" in data:
                break
        z = data["z"]
        print("[Server] z =", z)

        while True:
            data = readline_json(sockfile)
            if data is not None and "a2" in data:
                break
        a2 = data["a2"]
        print("[Server] a2 =", a2)

        e2 = random.getrandbits(511)
        print("[Client] Send e2 =", e2)
        s.sendall((json.dumps({"e": e2}) + "\n").encode())

        while True:
            data = readline_json(sockfile)
            if data is not None and "z2" in data:
                break
        z2 = data["z2"]
        print("[Server] z2 =", z2)

        diff_z = (z - z2) % q
        diff_e = (e - e2) % q

        inv_diff_e = inverse(diff_e, q)
        w_val = (diff_z * inv_diff_e) % q

        print("[+] Extracted w as integer:", w_val)

if __name__ == "__main__":
    main()
```
