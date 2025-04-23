---
title: "hash length extension attack"
date: 2023-10-01T18:40:23+09:00
draft: false
---

例えば，```user1:notadmin``` みたいな入力の先頭に秘密の言葉を加えてハッシュを取り，その内容を比較することで notadmin か admin かを判定する仕組みがあるとする．
このとき，通常であればユーザーは秘密の言葉を知り得ないので，```user1:admin``` みたいな文字列を作ることはできない．
一方で，md5 や sha1 のように Merkle–Damgård 構造をもっていると，内部構造がそのまま吐き出されるのでそれを利用して新しいハッシュを作ることができる．
つまり，secret:user1:notadmin のハッシュから secret:user1:notadmin:admin を作ることができる．

これは hashpump によって実現できる．
が，どうやら実装が消えている？ので自分で作る．
ひとまず sha1 だけ．

```python
#!/usr/bin/env python3
import struct
import sys
import base64

def left_rotate(n, b):
    return ((n << b) | (n >> (32 - b))) & 0xffffffff

class SHA1:
    def __init__(self, data=b'', _h0=None, _h1=None, _h2=None, _h3=None, _h4=None, message_byte_length=0):
        if _h0 is None:
            self.h0 = 0x67452301
            self.h1 = 0xEFCDAB89
            self.h2 = 0x98BADCFE
            self.h3 = 0x10325476
            self.h4 = 0xC3D2E1F0
        else:
            self.h0 = _h0
            self.h1 = _h1
            self.h2 = _h2
            self.h3 = _h3
            self.h4 = _h4
        self._unprocessed = b""
        self._message_byte_length = message_byte_length
        if data:
            self.update(data)

    def update(self, data):
        self._message_byte_length += len(data)
        self._unprocessed += data
        while len(self._unprocessed) >= 64:
            self._handle(self._unprocessed[:64])
            self._unprocessed = self._unprocessed[64:]

    def _handle(self, chunk):
        assert len(chunk) == 64
        w = list(struct.unpack(">16I", chunk))
        for i in range(16, 80):
            w.append(left_rotate(w[i-3] ^ w[i-8] ^ w[i-14] ^ w[i-16], 1))
        a, b, c, d, e = self.h0, self.h1, self.h2, self.h3, self.h4
        for i in range(80):
            if 0 <= i <= 19:
                f = (b & c) | ((~b) & d)
                k = 0x5A827999
            elif 20 <= i <= 39:
                f = b ^ c ^ d
                k = 0x6ED9EBA1
            elif 40 <= i <= 59:
                f = (b & c) | (b & d) | (c & d)
                k = 0x8F1BBCDC
            else:
                f = b ^ c ^ d
                k = 0xCA62C1D6
            temp = (left_rotate(a, 5) + f + e + k + w[i]) & 0xffffffff
            e = d
            d = c
            c = left_rotate(b, 30)
            b = a
            a = temp
        self.h0 = (self.h0 + a) & 0xffffffff
        self.h1 = (self.h1 + b) & 0xffffffff
        self.h2 = (self.h2 + c) & 0xffffffff
        self.h3 = (self.h3 + d) & 0xffffffff
        self.h4 = (self.h4 + e) & 0xffffffff

    def _padding(self, message_byte_length):
        padding = b"\x80"
        pad_len = (56 - (message_byte_length + 1) % 64) % 64
        padding += b"\x00" * pad_len
        padding += struct.pack(">Q", message_byte_length * 8)
        return padding

    def _produce_digest_ints(self):
        message = self._unprocessed + self._padding(self._message_byte_length)
        h0, h1, h2, h3, h4 = self.h0, self.h1, self.h2, self.h3, self.h4
        for i in range(0, len(message), 64):
            self._handle(message[i:i+64])
        digest_ints = (self.h0, self.h1, self.h2, self.h3, self.h4)
        self.h0, self.h1, self.h2, self.h3, self.h4 = h0, h1, h2, h3, h4
        return digest_ints

    def hexdigest(self):
        return "".join(["%08x" % i for i in self._produce_digest_ints()])

def sha1_padding(message_length):
    padding = b"\x80"
    pad_len = (56 - (message_length + 1) % 64) % 64
    padding += b"\x00" * pad_len
    padding += struct.pack(">Q", message_length * 8)
    return padding

def hashpump(old_hash, original_data, data_to_append, key_length):
    h = []
    for i in range(0, len(old_hash), 8):
        h.append(int(old_hash[i:i+8], 16))

    original_message_length = key_length + len(original_data)

    glue_padding = sha1_padding(original_message_length)

    new_message = original_data + glue_padding + data_to_append
    
    new_length = original_message_length + len(glue_padding)
    sha1_obj = SHA1(data=data_to_append,
                    _h0=h[0], _h1=h[1], _h2=h[2], _h3=h[3], _h4=h[4],
                    message_byte_length=new_length)
    new_hash = sha1_obj.hexdigest()
    return new_hash, new_message

if __name__ == "__main__":
    if len(sys.argv) != 5:
        print("Usage: {} <old_hash> <original_data> <data_to_append> <key_length>".format(sys.argv[0]))
        sys.exit(1)
    old_hash = sys.argv[1].strip()
    original_data = sys.argv[2].encode()
    data_to_append = sys.argv[3].encode()
    key_length = int(sys.argv[4])
    
    new_hash, new_message = hashpump(old_hash, original_data, data_to_append, key_length)
    encoded_new_message = base64.b64encode(new_message).decode('ascii')
    print("New hash: ", new_hash)
    print("New data (base64): ", encoded_new_message)
```

使い方は以下

```sh
python3 ./hashpump.py 00508f56ce332f48ed9f95fe189d08770851087d "username=guest&access_level=user&expiry=20251231" "&access_level=admin" 11
New hash:  a40abdabc31d68bdac6e1df509ee0ef74222a507
New data (base64):  dXNlcm5hbWU9Z3Vlc3QmYWNjZXNzX2xldmVsPXVzZXImZXhwaXJ5PTIwMjUxMjMxgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAHYJmFjY2Vzc19sZXZlbD1hZG1pbg==
```

おしまい
