---
title: "LITCTF2024"
date: 2024-08-13T23:18:46+09:00
draft: false
---

## Crypto

### simple otp

与えられた鍵と XOR する

```python
import random

encoded_with_xor = b'\x81Nx\x9b\xea)\xe4\x11\xc5 e\xbb\xcdR\xb7\x8f:\xf8\x8bJ\x15\x0e.n\\-/4\x91\xdcN\x8a'

random.seed(0)
key = random.randbytes(32)
#print(key ^ encoded_with_xor)

print(bytes(a ^ b for a, b in zip(key, encoded_with_xor)))
```

### privatekey

```txt
N = 91222155440553152389498614260050699731763350575147080767270489977917091931170943138928885120658877746247611632809405330094823541534217244038578699660880006339704989092479659053257803665271330929925869501196563443668981397902668090043639708667461870466802555861441754587186218972034248949207279990970777750209
e = 89367874380527493290104721678355794851752244712307964470391711606074727267038562743027846335233189217972523295913276633530423913558009009304519822798850828058341163149186400703842247356763254163467344158854476953789177826969005741218604103441014310747381924897883873667049874536894418991242502458035490144319
c = 71713040895862900826227958162735654909383845445237320223905265447935484166586100020297922365470898490364132661022898730819952219842679884422062319998678974747389086806470313146322055888525887658138813737156642494577963249790227961555514310838370972597205191372072037773173143170516757649991406773514836843206
```

e がデカい

```python
import owiener
from Crypto.Util.number import *

e = ...
n = ...
c = ...
d = owiener.attack(e, n)

print("d={}".format(d))

plain = pow(c, d, n)
print(long_to_bytes(plain).strip())
```

### pope shuffle

> it's like caesar cipher but better. Encoded: ࠴࠱࠼ࠫ࠼࠮ࡣࡋࡍࠨ࡛ࡍ࡚ࡇ࡛ࠩࡔࡉࡌࡥ

```python
s = "࠴࠱࠼ࠫ࠼࠮ࡣࡋࡍࠨ࡛ࡍ࡚ࡇ࡛ࠩࡔࡉࡌࡥ"
s2 = "LITCTF{"

for i in range(len(s)):
    print(chr(ord(s[i]) - 2024), end='')

print('')

for i in range(len(s2)):
    print(ord(s2[i]), end=',')
```

### Symmetric RSAPermalink

```python
#!/usr/bin/env python3
from Crypto.Util.number import long_to_bytes as ltb, bytes_to_long as btl, getPrime

p = getPrime(1024)
q = getPrime(1024)

n = p*q

e = p

with open("flag.txt", "rb") as f:
    PT = btl(f.read())

CT = pow(PT, e, n)
print(f"{CT = }")

for _ in range(4):
    CT = pow(int(input("Plaintext: ")), e, n)
    print(f"{CT = }")
```

e の使い方が変．
また，サーバにアクセスすると 4 回暗号文を得られる．

## Misc

### geoguessr1

google photo に入れたら中国の橋が見つかったので，google map で頑張って緯度経度を入れる

### geoguessr2

墓の写真を得る．
google map では墓地に入れなかったので上空写真からなんとなく場所をつかむ必要がある

### a little bit of tomcroppery

cropped な PNG の写真を得る．
binwalk で見るといくつか PNG が入っていることが分かる．

```binwalk -D '.*' image.png```

### endless

.mp3 が与えられる．
Flag を読み上げているのでこれを書き込むと flag になる．
想定解は分からない

### pokemon

ポケモンバトルが始まる．
オレはリザードン！

勝つと謎のおねえさんが出てきて旗を振る．
これをデコードする．

[](https://www.dcode.fr/semaphore-flag)

## rev

### forgotten message

```strings forgotten-message | grep LITCTF```

### kablewy

.js が動いている．
.js を整形して，tmp.txt に変換して以下のようにする．

```python
import re
import base64

f = open('tmp.txt', 'r')

boutput = []
st = ''

datalist = f.readlines()

for i in range(len(datalist)):
        matches = re.findall(r'"KGZ1bmN0aW9uKCl7InVzZSBzdHJpY3QiO2V2YWwoYXRvYigi(.*)IikpfSkoKTsK"', datalist[i])
        #matches = re.findall(r'const\s\w+\s=\s"(.*)"', datalist[i])
        if matches != []:
            boutput.append(str(base64.b64decode(matches[0]))[2:-1])

for i in range(len(boutput)):
    decoded = base64.b64decode(boutput[i])
    boutput[i] = re.findall(r"postMessage\('(.)'\)", str(decoded)[2:-1])
    st += (boutput[i][0])

print('flag: ' + st)

f.close()
```

### Burger Reviewer

.java が与えられる．
入力した文字列が flag と一致していてば OK．
.java の中身を逆算すると以下

```python
flag = 'LITCTF{XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX}'
flag = list(flag)

under = [13, 17, 19, 26, 29, 34, 39]

for i in under:
  flag[i] = '_'
  
meat = ['n', 'w', 'y', 'h', 't', 'f', 'i', 'a', 'i']
dif = [4, 2, 2, 2, 1, 2, 1, 3, 3]
m = 41

for i in range(len(meat)):
  m -= dif[i]
  flag[m] = meat[i]

veg1 = [10, 12, 15, 22, 23, 25, 32, 36, 38, 40]
veg = [0]*10

veg[0] = 9
veg[1] = 5
veg[2] = 4
veg[3] = 2
veg[4] = 2
veg[5] = 5
veg[6] = 3
veg[7] = 4
veg[8] = 7
veg[9] = 2

for i in range(len(veg1)):
  flag[veg1[i]] = str(veg[i])
 
#print(flag)
print(''.join(flag))

source = ['b', 'p', 'u', 'b', 'r', 'n', 'r', 'c']
a = 7
b = 20
i = 0

while a < b:
  flag[a] = source[i]
  flag[b] = source[i+1]
  a += 1
  b -= 1
  i += 2
  while(flag[a] != 'X'): a += 1
  while(flag[b] != 'X'): b -= 1
  print(''.join(flag))
  
print(''.join(flag))
```

### revsite1

wasm のフラグチェッカーをリバエンする必要がある．
はぁ．．．

chrome で見ると strcmp している部分がある．
ここにセットされた変数の値を見ればいいことがとりあえずわかる．
wasm は chrome でデバッグできるので，strcmp でブレークポイントを置き，memory inspector で中身を見る．(右側ペイン -> Scope -> Module -> memories -> $memory)
アドレスは変数の value から確認できる．

中身を見ると flag が露出している．

## web

### anti-inspect

```curl http://litctf.org:31779/```

%c を除かないとフラグにならない

### jwt-1

burp で通信を見ると，jwt トークンが見える．
こんな感じ ```eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiaWtpbW9ubyIsImFkbWluIjp0cnVlfQ.ekuW1alg7FNSAqKRPUIw4mymr5tItlcfmPxETpx7m4M```

で，これをデコードすると ```"admin": false``` という文字列が見えるので，ここを ```true``` に変える．
あとは謎に署名検証がされていないのでエンコードして通せばいい．

### jwt-2

同じように jwt トークンが見える．
今回は同じようにいかず，alg: none とかでもダメ．
ここで，index.ts を見ると以下のような記述がある

```js
app.get("/flag", (req, res) => {
  if (!req.cookies.token) {
    console.log('no auth')
    return res.status(403).send("Unauthorized");
  }

  try {
    const token = req.cookies.token;
    // split up token
    const [header, payload, signature] = token.split(".");
    if (!header || !payload || !signature) {
      return res.status(403).send("Unauthorized");
    }
    Buffer.from(header, "base64").toString();
    // decode payload
    const decodedPayload = Buffer.from(payload, "base64").toString();
    // parse payload
    const parsedPayload = JSON.parse(decodedPayload);
    // verify signature
    const expectedSignature = crypto.createHmac('sha256', jwtSecret).update(header + '.' + payload).digest('base64').replace(/=/g, '');
    if (signature !== expectedSignature) {
      return res.status(403).send('Unauthorized ;)');
    }
    // check if user is admin
    if (parsedPayload.admin || !("name" in parsedPayload)) {
      return res.send(
        fs.readFileSync(path.join(__dirname, "flag.txt"), "utf-8")
      );
    } else {
      return res.status(403).send("Unauthorized");
    }
  } catch {
    return res.status(403).send("Unauthorized");
  }
});
```

jwtSecret がハードコードされているので ```xook``` (Base64 encoded をオフにする)．
さらに ```admin = true``` に変える．
これで Verified になる．
ただし，コードは Base64 を受け付けているので ```_``` を ```\``` に直す必要がある．

### traversed

パストラバーサルっぽい．
```http://litctf.org:31778/?file=../../../../../../etc/passwd``` が通る．
```http://litctf.org:31778/?file=../../../flag.txt``` で OK
