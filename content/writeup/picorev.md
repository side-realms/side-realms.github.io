---
title: "picoCTF の rev 全部解く"
date: 2024-04-07T19:40:38+09:00
draft: false
---

CTF の問題を解いていると，解けはするけど異常に時間がかかる，みたいなことが多い．
rev に限れば，これは，ツールの使い方が甘かったり，つまみ食い的に rev をやってきたので常識が欠落していて遠回りしていたり，そもそもデコンパイル結果を読む筋力が全然なかったりすることが原因だと分かっている．
なので，数をこなそう作戦で，知り合いにオススメされた picoCTF の rev を全部解いてみる．
2~3 日くらいで完走できたら良い

### Transformation

- ```''.join([chr((ord(flag[i]) << 8) + ord(flag[i + 1])) for i in range(0, len(flag), 2)])``` をデコードする

```python
encoded_flag = "灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸弲㘶㠴挲ぽ"
flag = []

for i in range(len(encoded_flag)):
  tmp1 = ord(encoded_flag[i]) & 0xFF
  tmp2 = ord(encoded_flag[i]) >> 8
  flag += chr(tmp2)
  flag += chr(tmp1)

print("".join(flag))
```

### keygenme-py

- Arcane Calculator なるものが動いていて，ライセンスを入れるといいらしい

```python
def check_key(key, username_trial):

    global key_full_template_trial

    if len(key) != len(key_full_template_trial):
        return False
    else:
        # Check static base key part --v
        i = 0
        for c in key_part_static1_trial:
            if key[i] != c:
                return False

            i += 1

        # TODO : test performance on toolbox container
        # Check dynamic part --v
        if key[i] != hashlib.sha256(username_trial).hexdigest()[4]:
            return False
        else:
            i += 1

        if key[i] != hashlib.sha256(username_trial).hexdigest()[5]:
            return False
        else:
            i += 1

        if key[i] != hashlib.sha256(username_trial).hexdigest()[3]:
            return False
        else:
            i += 1

        if key[i] != hashlib.sha256(username_trial).hexdigest()[6]:
            return False
        else:
            i += 1

        if key[i] != hashlib.sha256(username_trial).hexdigest()[2]:
            return False
        else:
            i += 1

        if key[i] != hashlib.sha256(username_trial).hexdigest()[7]:
            return False
        else:
            i += 1

        if key[i] != hashlib.sha256(username_trial).hexdigest()[1]:
            return False
        else:
            i += 1

        if key[i] != hashlib.sha256(username_trial).hexdigest()[8]:
            return False



        return True
```

これを見て，あとは事前に定義されている文字列を当てはめればいい

### ARMssebly 0

ARM のアセンブリがわたされるのでクロスコンパイルする

```shell
aarch64-linux-gnu-as chall.S -o chall
```

Ghidra で見ると整数の大小を比較しているだけなので大きい方が答え

### vault-door-training

java のソースを読むだけ

### speeds and feeds

G-code が出てくるのでプロットする．
[いいサンプルサイト](https://ncviewer.com/)があった

### Shop

負の個数買うと所持金が増える

### Armeembly 1

```C
return 0xd2a - param_1
```

### ARMssembly 2

```C
  for (local_4 = 0; local_4 < param_1; local_4 = local_4 + 1) {
    local_8 = local_8 + 3;
  }
```

なんか何がしたいのか分からない問題が続くので，ちゃんと手を動かした問題だけのせる

### GDB baby step2

```shell
gdb debugger0_b

info functions

b main
layout asm
r
```

この時点で，main 関数の ret (関数抜け)のアドレスが分かるので

```shell
b *0x401142

c
info register rip
print/d $eax
```

### GDB baby step4

gdb で main に breakpoint を設置し，ステップ実行すると imul が見える

```shell
   0x401106 <func1>       endbr64 
   0x40110a <func1+4>     push   rbp
   0x40110b <func1+5>     mov    rbp, rsp
   0x40110e <func1+8>     mov    dword ptr [rbp - 4], edi
   0x401111 <func1+11>    mov    eax, dword ptr [rbp - 4]
 ► 0x401114 <func1+14>    imul   eax, eax, 0x3269
   0x40111a <func1+20>    pop    rbp
   0x40111b <func1+21>    ret    
```

ということで 0x3269

### Picker I

入力した関数を実行してくれるインスタンスが動いている．
win() を実行すると flag.txt を展開してくれるので，これを文字列に直せばいい

### Picker II

こんどは入力がフィルタされていて，「win」 が打てなくなっている．
一方で，main の eval はそのままなので，exec のようにコマンドインジェクションが通る．

```shell
==> print(open('flag.txt', 'r').read())
picoCTF{f1l73r5_f41l_c0d3_r3f4c70r_m1gh7_5ucc33d_95d44590}
'NoneType' object is not callable
```

### Picker III

必ず呼び出される ```call_func()``` 関数の最後に ```eval(func_name+'()')``` があるのでこれを利用する．

```python
def get_func(n):
  global func_table

  # Check table for viability
  if( not check_table() ):
    print(CORRUPT_MESSAGE)
    return

  # Get function name from table
  func_name = ''
  func_name_offset = n * FUNC_TABLE_ENTRY_SIZE
  for i in range(func_name_offset, func_name_offset+FUNC_TABLE_ENTRY_SIZE):
    if( func_table[i] == ' '):
      func_name = func_table[func_name_offset:i]
      break

  if( func_name == '' ):
    func_name = func_table[func_name_offset:func_name_offset+FUNC_TABLE_ENTRY_SIZE]
  
  return func_name
```

func_name が win になればいい

func_table を参照しているので，書き換えたい．

```shell
┌──(kali㉿kali)-[~/CTF]
└─$ nc saturn.picoctf.net 51363
==> 3
Please enter variable name to write: func_table
Please enter new value of variable: 'win                     read_variable                   write_variable                               '
==> 1
0x70 0x69 0x63 0x6f 0x43 0x54 0x46 0x7b 0x37 0x68 0x31 0x35 0x5f 0x31 0x35 0x5f 0x77 0x68 0x34 0x37 0x5f 0x77 0x33 07 0x31 0x37 0x68 0x5f 0x75 0x35 0x33 0x72 0x35 0x5f 0x31 0x6e 0x5f 0x63 0x68 0x34 0x72 0x67 0x33 0x5f 0x32 0x32 0x36 0x7d 
==>           
```

### packer

```strings``` するとなんか難読化された文字列がたくさん出てくる．
packer だし，```strings``` の結果にも ```UPX packer``` みたいな記述があるので，アンパックしてみる

```shell
upx -d out
strings out | grep flag
```

### asm1

アセンブリ，ちゃんとまとめたことがないので真面目にやる

```assembly
asm1:
    <+0>:   push   ebp
    <+1>:   mov    ebp,esp
    <+3>:   cmp    DWORD PTR [ebp+0x8],0x3fb
    <+10>:   jg     0x512 <asm1+37>
    <+12>:   cmp    DWORD PTR [ebp+0x8],0x280
    <+19>:   jne    0x50a <asm1+29>
    <+21>:   mov    eax,DWORD PTR [ebp+0x8]
    <+24>:   add    eax,0xa
    <+27>:   jmp    0x529 <asm1+60>
    <+29>:   mov    eax,DWORD PTR [ebp+0x8]
    <+32>:   sub    eax,0xa
    <+35>:   jmp    0x529 <asm1+60>
    <+37>:   cmp    DWORD PTR [ebp+0x8],0x559
    <+44>:   jne    0x523 <asm1+54>
    <+46>:   mov    eax,DWORD PTR [ebp+0x8]
    <+49>:   sub    eax,0xa
    <+52>:   jmp    0x529 <asm1+60>
    <+54>:   mov    eax,DWORD PTR [ebp+0x8]
    <+57>:   add    eax,0xa
    <+60>:   pop    ebp
    <+61>:   ret    

```

まず push ebp で，main 関数の上に今のベースポインタアドレス(ebp)を格納する．
このとき，push したので esp の値は -4 される．
次に，mov ebp,esp で，esp の値を ebp に格納する．
これにより，今の ebp(main 関数の底)が，esp(main 関数 + push ebp した値のスタックトップ)になるので，esp と ebp が同じ値になる．

```assembly
    <+0>:   push   ebp
    <+1>:   mov    ebp,esp
```

逆に，関数から戻るときはまず ```mov esp,ebp``` で esp(スタックトップ) に ebp(実行している関数のベースアドレス)の値を代入する．
次に ```pop ebp``` をする．
このとき pop されるのは，```push ebp``` で push された main関数の ebp なので，main 関数のベースアドレスが元に戻り，esp も +4 されて，関数から戻ってこれるようになる．

今回の場合は，引数がある．
引数は ebp+8 で表現されている．
(ebp の退避先，esp の退避先，引数の順)
引数は 0x2e0 で，0x2e0 < 0x3fb なので jg のジャンプは起こらない

```assembly
cmp    DWORD PTR [ebp+0x8],0x3fb
jg     0x512 <asm1+37>
```

次に 0x2e0 != 0x280 なので ZF = 0 で，移動する

```assembly
<+12>:   cmp    DWORD PTR [ebp+0x8],0x280
<+19>:   jne    0x50a <asm1+29>
```

jmp は何もなくても移動する

```assebmly
<+29>:   mov    eax,DWORD PTR [ebp+0x8]
<+32>:   sub    eax,0xa
<+35>:   jmp    0x529 <asm1+60>
```

```assembly
    <+60>:   pop    ebp
    <+61>:   ret    
```

### Bbbbloat

ghidra で見ると比較部分があるので，これをそのまま入力する

```C
if (local_48 == 0x86187)
```

### asm 2

```assembly
asm2:
    <+0>:    push   ebp
    <+1>:    mov    ebp,esp
    <+3>:    sub    esp,0x10
    <+6>:    mov    eax,DWORD PTR [ebp+0xc]
    <+9>:    mov    DWORD PTR [ebp-0x4],eax
    <+12>:    mov    eax,DWORD PTR [ebp+0x8]
    <+15>:    mov    DWORD PTR [ebp-0x8],eax
    <+18>:    jmp    0x50c <asm2+31>
    <+20>:    add    DWORD PTR [ebp-0x4],0x1
    <+24>:    add    DWORD PTR [ebp-0x8],0xd1
    <+31>:    cmp    DWORD PTR [ebp-0x8],0x5fa1
    <+38>:    jle    0x501 <asm2+20>
    <+40>:    mov    eax,DWORD PTR [ebp-0x4]
    <+43>:    leave  
    <+44>:    ret    
```

```assembly
    <+6>:    mov    eax,DWORD PTR [ebp+0xc]
    <+9>:    mov    DWORD PTR [ebp-0x4],eax
    <+12>:    mov    eax,DWORD PTR [ebp+0x8]
    <+15>:    mov    DWORD PTR [ebp-0x8],eax
```

引数を esp+0x10 で広げたスタック分に詰める．

```assembly
    <+31>:    cmp    DWORD PTR [ebp-0x8],0x5fa1
```

このあと， ebp-0x8 と 0x5fa1 が比較され，0x5fa1 の値を超えなければ 0xd1 が加算される．
なので，0x2d と 0x5fa1 の差を比べ，0xd1 を足した回数分，0x11 に 0x1 を足す．

- {(0x5fa1 - 0x2d) // 0xd1 } + 1 + 0x2d

### unpackme

たぶん pakcker だろうと思いながら解く．
radare2 の使い方を勉強しながら解いてみる．

正しい数字を入力するとフラグを返してくれるプログラムっぽい

```shell
┌──(kali㉿kali)-[~/CTF]
└─$ r2 unpackme-upx            
[0x00401c60]> aaa
[ ] Analyze all flags starting with sym. and entry0 (aa)
Cannot find basic block for switch case at 0x0043f6d3 bbdelta = 30
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Finding and parsing C++ vtables (avrr)
[x] Type matching analysis for all functions (aaft)
[x] Propagate noreturn information (aanr)
[x] Use -AA or aaaa to perform additional experimental analysis.
```

afl コマンドで main 関数があることが分かっているので main を解析する

```shell
[0x00401c60]> s sym.main
[0x00401e43]> pdf

-- snip --

│           0x00401e9c      488d3d61110b.  lea rdi, str.Whats_my_favorite_number__ ; 0x4b3004 ; "What's my favorite number? " ; int64_t arg1                                                                                            
│           0x00401ea3      b800000000     mov eax, 0
│           0x00401ea8      e8f3ec0000     call sym.__printf
│           0x00401ead      488d45c4       lea rax, [var_3ch]
│           0x00401eb1      4889c6         mov rsi, rax
│           0x00401eb4      488d3d65110b.  lea rdi, [0x004b3020]       ; "%d" ; const char *format
│           0x00401ebb      b800000000     mov eax, 0
│           0x00401ec0      e86bee0000     call sym.__isoc99_scanf     ; int scanf(const char *format)
│           0x00401ec5      8b45c4         mov eax, dword [var_3ch]
│           0x00401ec8      3dcb830b00     cmp eax, 0xb83cb <- これ

-- snip --
```

### forky

```C
{
  int *piVar1;
  
  piVar1 = (int *)mmap((void *)0x0,4,3,0x21,-1,0);
  *piVar1 = 1000000000;
  fork();
  fork();
  fork();
  fork();
  *piVar1 = *piVar1 + 0x499602d2;
  doNothing(*piVar1);
  return 0;
}
```

```doNothing``` に渡される引数の値が flag になっている．
piVar1 が明らかに鍵なので， fork() の動作を見る．

とは言っても単純で，プロセスが 2^4 = 16 個作られるだけ

1000000000 + 0x499602d2 * 16 = -721750240

### ARMssembly3

```C
undefined4 func1(uint param_1)

{
  uint local_14;
  undefined4 local_4;
  
  local_4 = 0;
  for (local_14 = param_1; local_14 != 0; local_14 = local_14 >> 1) {
    if ((local_14 & 1) != 0) {
      local_4 = func2(local_4);
    }
  }
  return local_4;
}
```

```func2``` は引数に 3 を足して返す関数．
param1 のビットを見て後ろから 1 なら +3 する，みたいな感じでやればいい

```python
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

num = 2541039191
i = 0

while num != 0:
    if num&1 != 0:
        i = i+3
    num = num >> 1
print(i)

```

### reverse_cipher

```C
void main(void)

{
  size_t sVar1;
  char flag [23];
  char local_41;
  int local_2c;
  FILE *fp_output;
  FILE *fp_flag;
  uint local_14;
  int local_10;
  char local_9;
  
  fp_flag = fopen("flag.txt","r");
  fp_output = fopen("rev_this","a");
  if (fp_flag == (FILE *)0x0) {
    puts("No flag found, please make sure this is run on the server");
  }
  if (fp_output == (FILE *)0x0) {
    puts("please run this on the server");
  }
  sVar1 = fread(flag,0x18,1,fp_flag);
  local_2c = (int)sVar1;
  if ((int)sVar1 < 1) {
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  for (local_10 = 0; local_10 < 8; local_10 = local_10 + 1) {
    local_9 = flag[local_10];
    fputc((int)local_9,fp_output);
  }
  for (local_14 = 8; (int)local_14 < 0x17; local_14 = local_14 + 1) {
    if ((local_14 & 1) == 0) {
      local_9 = flag[(int)local_14] + '\x05';
    }
    else {
      local_9 = flag[(int)local_14] + -2;
    }
    fputc((int)local_9,fp_output);
  }
  local_9 = local_41;
  fputc((int)local_41,fp_output);
  fclose(fp_output);
  fclose(fp_flag);
  return;
}
```

なんとなく変数名を直したのがこれ．
flag.txt からフラグを読んで，ちょっと計算して rev.txt に書き出すだけのプログラム．
なのでその計算に応じたソルバを書いてあげる

```python
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

ans = "w1{1wq84fb<1>49}"
cnt = 0

for i in ans:
  if cnt&1 == 0:
    tmp = ord(i) - 0x5
    print(chr(tmp), end="")
  else:
    tmp = ord(i) + 0x2
    print(chr(tmp), end="")
  cnt = cnt+1
```

### asm3

```assembly
asm3:
    <+0>:    push   ebp
    <+1>:    mov    ebp,esp
    <+3>:    xor    eax,eax // eax -> 0
    <+5>:    mov    ah,BYTE PTR [ebp+0xa] // ah <- 33
    <+8>:    shl    ax,0x10  // ax <- 3300 <- 0000
    <+12>:    sub    al,BYTE PTR [ebp+0xc] //ae  0x00 - 0xae = FFFFFF52, al <- 52
    <+15>:    add    ah,BYTE PTR [ebp+0xd] //ah 0x00 + 0x72 = 72, ah <- 72
    <+18>:    xor    ax,WORD PTR [ebp+0x10] // ax <- 0x7252 xor b139
    <+22>:    nop
    <+23>:    pop    ebp
    <+24>:    ret    
```

変数の積み方とリトルエンディアンの復習になった

初期状態は以下

| stack |
|:-----------|
| main ebp   |
| ret        |
| d73346ed(引数1)|
| d48672ae(引数2)|
| d3c8b139(引数3)|
| main       |

リトルエンディアンなので，例えば [ebp+0xa] では引数1 の 33 にアクセスできる

### asm4

アセンブリが渡されるが，多すぎて一つ一つ見るのは大変なのでコンパイルできるようにする．
32 bit なので ```gcc -masm=intel -m32 solve.c -o solve``` とする．
実行に ```sudo apt install libc6-dev-i386``` が必要．

```C
#include <stdio.h>
#include <stdlib.h>

int asm4(char* in)
{
    int val;

    asm (
        "nop;"
        "nop;"
        "nop;"
        //"push   ebp;"
        //"mov    ebp,esp;"
        "push   ebx;"
        "sub    esp,0x10;"
        "mov    DWORD PTR [ebp-0x10],0x246;"
        "mov    DWORD PTR [ebp-0xc],0x0;"
        "jmp    _asm_27;"
    "_asm_23:"
        "add    DWORD PTR [ebp-0xc],0x1;"
    "_asm_27:"
        "mov    edx,DWORD PTR [ebp-0xc];"
        "mov    eax,DWORD PTR [%[pInput]];"
        "add    eax,edx;"
        "movzx  eax,BYTE PTR [eax];"
        "test   al,al;"
        "jne    _asm_23;"
        "mov    DWORD PTR [ebp-0x8],0x1;"
        "jmp    _asm_138;"
    "_asm_51:"
        "mov    edx,DWORD PTR [ebp-0x8];"
        "mov    eax,DWORD PTR [%[pInput]];"
        "add    eax,edx;"
        "movzx  eax,BYTE PTR [eax];"
        "movsx  edx,al;"
        "mov    eax,DWORD PTR [ebp-0x8];"
        "lea    ecx,[eax-0x1];"
        "mov    eax,DWORD PTR [%[pInput]];"
        "add    eax,ecx;"
        "movzx  eax,BYTE PTR [eax];"
        "movsx  eax,al;"
        "sub    edx,eax;"
        "mov    eax,edx;"
        "mov    edx,eax;"
        "mov    eax,DWORD PTR [ebp-0x10];"
        "lea    ebx,[edx+eax*1];"
        "mov    eax,DWORD PTR [ebp-0x8];"
        "lea    edx,[eax+0x1];"
        "mov    eax,DWORD PTR [%[pInput]];"
        "add    eax,edx;"
        "movzx  eax,BYTE PTR [eax];"
        "movsx  edx,al;"
        "mov    ecx,DWORD PTR [ebp-0x8];"
        "mov    eax,DWORD PTR [%[pInput]];"
        "add    eax,ecx;"
        "movzx  eax,BYTE PTR [eax];"
        "movsx  eax,al;"
        "sub    edx,eax;"
        "mov    eax,edx;"
        "add    eax,ebx;"
        "mov    DWORD PTR [ebp-0x10],eax;"
        "add    DWORD PTR [ebp-0x8],0x1;"
    "_asm_138:"
        "mov    eax,DWORD PTR [ebp-0xc];"
        "sub    eax,0x1;"
        "cmp    DWORD PTR [ebp-0x8],eax;"
        "jl     _asm_51;"
        "mov    eax,DWORD PTR [ebp-0x10];"
        "add    esp,0x10;"
        "pop    ebx;"
        //"pop    ebp;"
        //"ret    ;"
        "nop;"
        "nop;"
        "nop;"
            :"=r"(val)
            : [pInput] "m"(in)
    );
    
    return val;
}

int main(int argc, char** argv)
{
    printf("0x%x\n", asm4("picoCTF_a3112"));
    
    return 0;
}
```

### Need For Speed

```C

void set_timer(void)

{
  __sighandler_t p_Var1;
  
  p_Var1 = __sysv_signal(0xe,alarm_handler);
  if (p_Var1 == (__sighandler_t)0xffffffffffffffff) {
    puts("\n\nSomething bad happened here. ");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  alarm(1);
  return;
}
```

alarm(1) によって，一秒後に signal が送られて実行が中断する．
そこで，set_timer() を実行しないようにしたい．

方法1: SIGALRM がセットされていることが分かっているので，```(gdb) handle SIGALRM ignore```
が使えるらしい．

方法2: set_timer() の呼び出しを nop に書き変えることでも回避できる．

```shell
┌──(kali㉿kali)-[~/CTF]
└─$ r2 -d need-for-speed 
[0x7f4ef7a0b4d0]> aaa
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Finding and parsing C++ vtables (avrr)
[x] Skipping type matching analysis in debugger mode (aaft)
[x] Propagate noreturn information (aanr)
[x] Use -AA or aaaa to perform additional experimental analysis.
[0x7f4ef7a0b4d0]> s main
[0x55894100091a]> pdf
            ; DATA XREF from entry0 @ 0x55894100067d
┌ 62: int main (int argc, char **argv);
│           ; var int64_t var_10h @ rbp-0x10
│           ; var int64_t var_4h @ rbp-0x4
│           ; arg int argc @ rdi
│           ; arg char **argv @ rsi
│           0x55894100091a      55             push rbp
│           0x55894100091b      4889e5         mov rbp, rsp
│           0x55894100091e      4883ec10       sub rsp, 0x10
│           0x558941000922      897dfc         mov dword [var_4h], edi ; argc
│           0x558941000925      488975f0       mov qword [var_10h], rsi ; argv
│           0x558941000929      b800000000     mov eax, 0
│           0x55894100092e      e8a5ffffff     call sym.header
│           0x558941000933      b800000000     mov eax, 0
│           0x558941000938      e8f2feffff     call sym.set_timer
│           0x55894100093d      b800000000     mov eax, 0
│           0x558941000942      e836ffffff     call sym.get_key
│           0x558941000947      b800000000     mov eax, 0
│           0x55894100094c      e85bffffff     call sym.print_flag
│           0x558941000951      b800000000     mov eax, 0
│           0x558941000956      c9             leave
└           0x558941000957      c3             ret
```

これで ```call sym.set_timer``` の場所が分かったので，nop に書きかえる．

```shell
[0x55894100091a]> s 0x558941000938
[0x558941000938]> wao nop
[0x558941000938]> pdf
            ; DATA XREF from entry0 @ 0x55894100067d
┌ 62: int main (int argc, char **argv);
│           ; var int64_t var_10h @ rbp-0x10
│           ; var int64_t var_4h @ rbp-0x4
│           ; arg int argc @ rdi
│           ; arg char **argv @ rsi
│           0x55894100091a      55             push rbp
│           0x55894100091b      4889e5         mov rbp, rsp
│           0x55894100091e      4883ec10       sub rsp, 0x10
│           0x558941000922      897dfc         mov dword [var_4h], edi ; argc
│           0x558941000925      488975f0       mov qword [var_10h], rsi ; argv
│           0x558941000929      b800000000     mov eax, 0
│           0x55894100092e      e8a5ffffff     call sym.header
│           0x558941000933      b800000000     mov eax, 0
│           0x558941000938      90             nop
..
│           0x55894100093d      b800000000     mov eax, 0
│           0x558941000942      e836ffffff     call sym.get_key
│           0x558941000947      b800000000     mov eax, 0
│           0x55894100094c      e85bffffff     call sym.print_flag
│           0x558941000951      b800000000     mov eax, 0
│           0x558941000956      c9             leave
└           0x558941000957      c3             ret
[0x558941000938]> dc
Keep this thing over 50 mph!
============================

Creating key...

Finished
Printing flag:
PICOCTF{Good job keeping bus #1f2ac4ec speeding along!}
(128407) Process exited with status=0x0
```

### OTP Implementation

なんか OTP っぽいプログラムが渡される

- jumble: ある規則に従って数字をこねくり回す
- main: さらにこねくり回す

これを逆から見ていけば所望の鍵が手に入るので，flag と xor すればいい

```python
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

from Crypto.Util.number import getPrime, bytes_to_long, long_to_bytes

flag = "1fcb81cd1f6f1e12b429092e3647153b6c212772554ca004145b82367e1e6b7870827dc249a319601776f727434e6b6227d1"
cmp_str = "mlaebfkoibhoijfidblechbggcgldicegjbkcmolhdjihgmmieabohpdhjnciacbjjcnpcfaopigkpdfnoaknjlnlaohboimombk"
candidate = "1234567890abcdef"

def jumble(param1):
     ans = param1
     if(0x60 < param1):
          ans = param1 + 0x9
     ans = (ans % 0x10) * 0x02
     if(0x0f < ans):
          ans = ans +  0x01
     return ans

def body(counter, chara, pre):
     if(counter == 0):
          after_jumble = jumble(ord(chara))
          result = after_jumble % 0x10
          #print("OK1")
     else:
          after_jumble = jumble(ord(chara))
          Val = after_jumble + pre >> 0x1f
          result = (after_jumble + pre + (Val >> 4) & 0xf) - (Val >> 4)
          #print("OK")
     counter = counter + 1
     return result

pre = 0
results = [0]*100
     
for c in range(100):
     for i in candidate:
          result = body(c, i, pre)
          result = chr(result + 0x61)
          if(cmp_str[c] == result):
               results[c] = i
               pre = ord(result)
               #print(chr(int(i, 16) ^ ord(cmp_str[c])))
               #print("OK")
               break

enc = ''

for i in range(100):
     print(long_to_bytes(int(results[i], 16) ^ ord(flag[i])))
     #print(chr(int(results[i], 16) ^ ord(flag[i])), end='')
```

### gogo

golang のデコンパイル結果こんな見にくいのか

まず ghidra で見ると checkPassword がある．

```C
  key[0] = 0x38;
  key[1] = 0x36;
  key[2] = 0x31;
  key[3] = 0x38;
                    /* /opt/hacksports/shared/staging/gogo_5_8320186217489444/problem_files/enter_pa ssword.go:39
                        */
  key[4] = 0x33;
  key[5] = 0x36;
  key[6] = 0x66;
  key[7] = 0x31;
                    /* /opt/hacksports/shared/staging/gogo_5_8320186217489444/problem_files/enter_pa ssword.go:43
                        */
  key[8] = 0x33;
  key[9] = 0x65;
  key[10] = 0x33;
  key[11] = 100;
                    /* /opt/hacksports/shared/staging/gogo_5_8320186217489444/problem_files/enter_pa ssword.go:47
                        */
  key[12] = 0x36;
  key[13] = 0x32;
  key[14] = 0x37;
  key[15] = 100;
                    /* /opt/hacksports/shared/staging/gogo_5_8320186217489444/problem_files/enter_pa ssword.go:51
                        */
  key[16] = 0x66;
  key[17] = 0x61;
  key[18] = 0x33;
  key[19] = 0x37;
                    /* /opt/hacksports/shared/staging/gogo_5_8320186217489444/problem_files/enter_pa ssword.go:55
                        */
  key[20] = 0x35;
  key[21] = 0x62;
  key[22] = 100;
  key[23] = 0x62;
                    /* /opt/hacksports/shared/staging/gogo_5_8320186217489444/problem_files/enter_pa ssword.go:59
                        */
  key[24] = 0x38;
  key[25] = 0x33;
  key[26] = 0x38;
  key[27] = 0x39;
                    /* /opt/hacksports/shared/staging/gogo_5_8320186217489444/problem_files/enter_pa ssword.go:63
                        */
  key[28] = 0x32;
  key[29] = 0x31;
  key[30] = 0x34;
  key[31] = 0x65;
                    /* /opt/hacksports/shared/staging/gogo_5_8320186217489444/problem_files/enter_pa ssword.go:67
                        */
                    /* /opt/hacksports/shared/staging/gogo_5_8320186217489444/problem_files/enter_pa ssword.go:68
                        */
  runtime.duffcopy_0x20_FUN_08090fe0(local_20,(uint4 *)main.statictmp_4);
  uVar1 = 0;
  iVar2 = 0;
                    /* /opt/hacksports/shared/staging/gogo_5_8320186217489444/problem_files/enter_pa ssword.go:70
                        */
  while( true ) {
    if (0x1f < (int)uVar1) {
                    /* /opt/hacksports/shared/staging/gogo_5_8320186217489444/problem_files/enter_pa ssword.go:75
                        */
      if (iVar2 == 0x20) {
                    /* /opt/hacksports/shared/staging/gogo_5_8320186217489444/problem_files/enter_pa ssword.go:76
                        */
        return;
      }
                    /* /opt/hacksports/shared/staging/gogo_5_8320186217489444/problem_files/enter_pa ssword.go:78
                        */
      return;
    }
                    /* /opt/hacksports/shared/staging/gogo_5_8320186217489444/problem_files/enter_pa ssword.go:71
                        */
                    /* /opt/hacksports/shared/staging/gogo_5_8320186217489444/problem_files/enter_pa ssword.go:70
                        */
                    /* /opt/hacksports/shared/staging/gogo_5_8320186217489444/problem_files/enter_pa ssword.go:71
                        */
    if ((param_2 <= uVar1) || (0x1f < uVar1)) break;
    if ((*(byte *)(param_1 + uVar1) ^ key[uVar1]) == *(byte *)((int)local_20 + uVar1)) {
                    /* /opt/hacksports/shared/staging/gogo_5_8320186217489444/problem_files/enter_pa ssword.go:72
                        */
      iVar2 = iVar2 + 1;
    }
    uVar1 = uVar1 + 1;
  }
```

key と main.statictmp_4 の中身を XOR しているっぽいので，まずこれを計算する．

```python
>>> for i in range(0x20):
...  print(chr(tmp[i]^input[i]),end='')
reverseengineericanbarelyforward
```

次に，鍵を unhash しろ，と出てくる．
コード中に md5 とあるのでハッシュを探すと goldfish が見つかる．

### Let's get dynamic

```C

bool main(void)

{

~~ snip ~~

  local_20 = *(long *)(in_FS_OFFSET + 0x28);
  local_98 = 0x396c109a7067b614;
  local_90 = 0x32ea1ab1495990f0;
  local_88 = 0xd09aa897d230c8fe;
  local_80 = 0x2c227b84b00f7d0b;
  local_78 = 0xb0a880f7d99ea817;
  local_70 = 0xc8f18206086afe7c;
  local_68 = 0x61;
  local_58 = 0x563f52ce0f15cd77;
  local_50 = 0x719435c3652ef38f;
  local_48 = 0x8bec9fe9be05a4c9;
  local_40 = 0x521c05fe8d590431;
  local_38 = 0xdbf1c3a6dadcef7b;
  local_30 = 0xc4fc8b585631f076;
  local_28 = 0x3f;
  fgets(user_input,0x31,stdin);
  counter = 0;
  while( true ) {
    str_length = strlen((char *)&local_98);
    if (str_length <= (ulong)(long)counter) break;
    local_118[counter] =
         (byte)counter ^
         *(byte *)((long)&local_98 + (long)counter) ^ *(byte *)((long)&local_58 + (long)counter) ^
         0x13;
    counter = counter + 1;
  }
  iVar1 = memcmp(user_input,local_118,0x31);
  if (iVar1 == 0) {
    puts("No, that\'s not right.");
  }
  else {
    puts("Correct! You entered the flag.");
  }
  if (local_20 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return iVar1 == 0;
}

```

入力文字列と既存のフラグを比較している．
文字数は 0x31.
途中にある while 文を注目すればよくて，local_xx にあるデータをこねくりまわしている．
ここまでは簡単なんだがデータがリトルエンディアンなのでスタックの最後から取り出さなければいけない．
ここにつまづいて 5 年かけました

```python
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

local_98 = [0x39,0x6c,0x10,0x9a,0x70,0x67,0xb6,0x14]
local_90 = [0x32,0xea,0x1a,0xb1,0x49,0x59,0x90,0xf0]
local_88 = [0xd0,0x9a,0xa8,0x97,0xd2,0x30,0xc8,0xfe]
local_80 = [0x2c,0x22,0x7b,0x84,0xb0,0x0f,0x7d,0x0b]
local_78 = [0xb0,0xa8,0x80,0xf7,0xd9,0x9e,0xa8,0x17]
local_70 = [0xc8,0xf1,0x82,0x06,0x08,0x6a,0xfe,0x7c]
local_68 = [0x61]

local_58 = [0x56,0x3f,0x52,0xce,0x0f,0x15,0xcd,0x77]
local_50 = [0x71,0x94,0x35,0xc3,0x65,0x2e,0xf3,0x8f]
local_48 = [0x8b,0xec,0x9f,0xe9,0xbe,0x05,0xa4,0xc9]
local_40 = [0x52,0x1c,0x05,0xfe,0x8d,0x59,0x04,0x31]
local_38 = [0xdb,0xf1,0xc3,0xa6,0xda,0xdc,0xef,0x7b]
local_30 = [0xc4,0xfc,0x8b,0x58,0x56,0x31,0xf0,0x76]
local_28 = [0x3f];

output = [0]*0x31
counter = 0
plus = 40

while(True):
   output[counter] = chr(plus ^ (local_30[7-counter]) ^ (local_70[7-counter]) ^ 0x13)
   counter = counter + 1
   plus = plus+1
   print(output)
   
print(output)
```

#### 別解

stack 表示を見ればいいらしい

```shell
pwndbg> b memcmp
Breakpoint 1 at 0x1060
pwndbg> r
~~snip~~
pwndbg> stack 20
00:0000│ rsp 0x7fffffffda18 —▸ 0x7ffff7fdb432 (_dl_fixup+642) ◂— mov qword ptr [rsp + 8], rax
01:0008│     0x7fffffffda20 —▸ 0x7ffff7ddeba0 ◂— 0x10001a0000487e /* '~H' */
02:0010│     0x7fffffffda28 —▸ 0x7ffff7e630f0 (memcmp) ◂— mov rax, qword ptr [rip + 0x137db9]
03:0018│     0x7fffffffda30 —▸ 0x7fffffffdca0 ◂— 0x31 /* '1' */
04:0020│     0x7fffffffda38 —▸ 0x7fffffffddf0 ◂— 0x1
05:0028│     0x7fffffffda40 ◂— 0x0
06:0030│     0x7fffffffda48 —▸ 0x7fffffffdf18 —▸ 0x7fffffffe290 ◂— 'COLORFGBG=15;0'
07:0038│     0x7fffffffda50 —▸ 0x555555557dd8 (__do_global_dtors_aux_fini_array_entry) —▸ 0x555555555130 (__do_global_dtors_aux) ◂— endbr64                                                                     
08:0040│     0x7fffffffda58 —▸ 0x7ffff7fdd443 (_dl_runtime_resolve_fxsave+67) ◂— mov r11, rax
09:0048│     0x7fffffffda60 —▸ 0x7fffffffdd20 ◂— 0x2f2f2f2f2f2f000a /* '\n' */
0a:0050│     0x7fffffffda68 —▸ 0x7fffffffdce0 ◂— 'picoCTF{dyn4m1c_4n4ly1s_1s_5up3r_us3ful_14bfa700}'
0b:0058│     0x7fffffffda70 ◂— 0x31 /* '1' */
0c:0060│     0x7fffffffda78 —▸ 0x7fffffffdce0 ◂— 'picoCTF{dyn4m1c_4n4ly1s_1s_5up3r_us3ful_14bfa700}'
0d:0068│     0x7fffffffda80 —▸ 0x7fffffffdd20 ◂— 0x2f2f2f2f2f2f000a /* '\n' */
0e:0070│     0x7fffffffda88 ◂— 0x0
0f:0078│     0x7fffffffda90 ◂— 0x410
10:0080│     0x7fffffffda98 ◂— 0x0
11:0088│     0x7fffffffdaa0 ◂— 0x37f
12:0090│     0x7fffffffdaa8 ◂— 0x0
13:0098│     0x7fffffffdab0 ◂— 0x0
```

なんて便利なんだ...

### Easy as GDB

```C
undefined4 FUN_000108c4(char *param_1,uint param_2)

{
  char *__dest;
  char *__dest_00;
  uint local_18;
  
  __dest = (char *)calloc(param_2 + 1,1);
  strncpy(__dest,param_1,param_2);
  FUN_000107c2(__dest,param_2,0xffffffff);
  __dest_00 = (char *)calloc(param_2 + 1,1);
  strncpy(__dest_00,&DAT_00012008,param_2);
  FUN_000107c2(__dest_00,param_2,0xffffffff);
  puts("checking solution...");
  local_18 = 0;
  while( true ) {
    if (param_2 <= local_18) {
      return 1;
    }
    if (__dest[local_18] != __dest_00[local_18]) break;
    local_18 = local_18 + 1;
  }
  return 0xffffffff;
}
```

入力文字列と内蔵文字列が一致していない or ループ回数が入力文字列数を超える で，
ループを抜ける．
つまり，入力した文字列がすべて正しければループ回数が入力文字列を超えるので
(local_18 (ループ回数)が param_2 (入力文字列数) を超える)，
ループ回数を見ることでフラグが分かる．
以下に gdb スクリプト

```python
import gdb
import string

gdb.execute('file ./brute')
gdb.execute('set pagination off')
gdb.Breakpoint('*0x565559a5')

pattern = string.printable
flag = ''

for i in range(30):
    for chara in pattern:
   payload = flag + chara
   gdb.execute('run ' + chara)
      
   for _ in range(i + 1):
            try:
                gdb.execute('continue')
            except gdb.error:
                pass

        msg = gdb.execute('i b', to_string=True)
        if 'hit {} times'.format(i + 2) in msg:
            flag += chara
            gdb.execute('continue')
            break

        print('flag:', flag + chara)
        print(msg)

print(flag)
gdb.execute('quit')
```

相対アドレスは忘れていたので注意

Ghidra で表示されているアドレスは gdb とズレてしまうことに注意する．
Ghidra は ```0x00010000``` からアドレスが始まる．
gdb の方は ```info proc mapping``` でアドレスの内容を見る

```shell
pwndbg> info proc mapping
process 614292
Mapped address spaces:

        Start Addr   End Addr       Size     Offset  Perms   objfile
        0x56555000 0x56556000     0x1000        0x0  r-xp   /home/kali/CTF/brute
        0x56556000 0x56557000     0x1000        0x0  r--p   /home/kali/CTF/brute
        0x56557000 0x56558000     0x1000     0x1000  rw-p   /home/kali/CTF/brute
...
```

で，Ghidra でブレークポイントを設置したい命令は ```000109a5 72 d1 JC LAB_00010978``` なので，
ベースアドレス +0x9a5 の ところに所望の命令があることが分かる．

あとはせいぜい 3 つなので 3 つとも試してみてブレークポイントが貼れる & スクリプトが正常に動作する
ものを選べばいいが，ダメ押しで ```x/16xb 0x565559a5``` みたいな感じでやると
逆アセンブルコードを表示してくれるので，Ghidra の表示と一致するものを選べばいい．

### ARMssembly 4

計算するだけ．
なんだこれは

### not crypto

```"I heard you wanted to bargain for a flag... whatcha got?``` という文字列を探して，おそらく main 関数は FUN_00101070 だと推測する．
入力は ```fread(local_198,1,0x40,_stdin);``` のように標準入力から受け取り，最後の方で ```iVar19 = memcmp(local_88,local_198,0x40);``` みたいな比較をしている．
local_198 がそのままであれば local_88 が flag になっているはずなので， memcmp のあたりでレジスタを見ればよさそう．

ブレークポイントを memcmp において(アドレスに注意する)，rdi を見るとフラグをゲト

### Ready Gladiator 2

Core wars らしい．
imp(小鬼) と呼ばれるプログラムに全勝すればいいらしい．

```text
;redcode
;name Imp Ex
;assert 1
JMP 0, <-5
end
```

なんか上手くいった．．．

### No way out

Unity 製のゲームが動いている．
ハシゴを上ると旗が見えるが，透明な壁があって通り抜けられない．

dll のデコンパイル，インターンで教わったな．．．と思いながら dnSpy をインストールする．
が，Unity 製の dll がどれをデコンパイルするべきか見当もつかなかったので writeup を見る．
```/Managed/Assembly-CSharp.dll``` を見ればいいらしい．
全部の exe でそうなんだろうか

dll をなんとなく見ると，以下の記述が目に付く．

```C#
if (Input.GetButton("Jump") && this.canMove && this.characterController.isGrounded && !this.isClimbing)
```

昔ゲームを作った時にジャンプ判定で苦労した記憶を掘り出して，ジャンプできるのはプレイヤーの着地判定がトリガーしているときだけだったことを思い出す．
なので && 以下をコメントアウトして再コンパイルする．
File -> Save Module を忘れずに(戒め)

ゲームを起動して旗に近づくとフラグが見える．

### FactCheck

```C
  local_20 = *(long *)(in_FS_OFFSET + 0x28);
  std::allocator<char>::allocator();
                    // try { // try from 001012cf to 001012d3 has its CatchHandler @ 00101975
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string
            ((char *)local_248,(allocator *)"picoCTF{wELF_d0N3_mate_");
  std::allocator<char>::~allocator(&local_249);
  std::allocator<char>::allocator();
                    // try { // try from 0010130a to 0010130e has its CatchHandler @ 00101996
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string
            ((char *)local_228,(allocator *)&DAT_0010201d);
  std::allocator<char>::~allocator(&local_249);
  std::allocator<char>::allocator();
                    // try { // try from 00101345 to 00101349 has its CatchHandler @ 001019b1
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string
            ((char *)local_208,(allocator *)&DAT_0010201f);
  std::allocator<char>::~allocator(&local_249);
  std::allocator<char>::allocator();
                    // try { // try from 00101380 to 00101384 has its CatchHandler @ 001019cc
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string
            ((char *)local_1e8,(allocator *)&DAT_00102021);
  std::allocator<char>::~allocator(&local_249);
  std::allocator<char>::allocator();
                    // try { // try from 001013bb to 001013bf has its CatchHandler @ 001019e7
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string
            ((char *)local_1c8,(allocator *)&DAT_0010201d);
  std::allocator<char>::~allocator(&local_249);
  std::allocator<char>::allocator();
                    // try { // try from 001013f6 to 001013fa has its CatchHandler @ 00101a02
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string
            ((char *)local_1a8,(allocator *)&DAT_00102023);
  std::allocator<char>::~allocator(&local_249);
  std::allocator<char>::allocator();
                    // try { // try from 00101431 to 00101435 has its CatchHandler @ 00101a1d
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string
            ((char *)local_188,(allocator *)&DAT_00102025);
  std::allocator<char>::~allocator(&local_249);
  std::allocator<char>::allocator();
                    // try { // try from 0010146c to 00101470 has its CatchHandler @ 00101a38
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string
            ((char *)local_168,(allocator *)&DAT_00102027);
  std::allocator<char>::~allocator(&local_249);
  std::allocator<char>::allocator();
                    // try { // try from 001014a7 to 001014ab has its CatchHandler @ 00101a53
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string
            ((char *)local_148,(allocator *)&DAT_00102029);
  std::allocator<char>::~allocator(&local_249);
  std::allocator<char>::allocator();
                    // try { // try from 001014e2 to 001014e6 has its CatchHandler @ 00101a6e
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string
            ((char *)local_128,(allocator *)&DAT_0010202b);
  std::allocator<char>::~allocator(&local_249);
  std::allocator<char>::allocator();
                    // try { // try from 0010151d to 00101521 has its CatchHandler @ 00101a89
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string
            ((char *)local_108,(allocator *)&DAT_0010202d);
  std::allocator<char>::~allocator(&local_249);
  std::allocator<char>::allocator();
                    // try { // try from 00101558 to 0010155c has its CatchHandler @ 00101aa4
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string
            ((char *)local_e8,(allocator *)&DAT_00102025);
  std::allocator<char>::~allocator(&local_249);
  std::allocator<char>::allocator();
                    // try { // try from 00101593 to 00101597 has its CatchHandler @ 00101abf
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string
            ((char *)local_c8,(allocator *)&DAT_0010202f);
  std::allocator<char>::~allocator(&local_249);
  std::allocator<char>::allocator();
                    // try { // try from 001015ce to 001015d2 has its CatchHandler @ 00101ada
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string
            ((char *)local_a8,(allocator *)&DAT_00102031);
  std::allocator<char>::~allocator(&local_249);
  std::allocator<char>::allocator();
                    // try { // try from 00101606 to 0010160a has its CatchHandler @ 00101af5
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string
            ((char *)local_88,(allocator *)&DAT_00102033);
  std::allocator<char>::~allocator(&local_249);
  std::allocator<char>::allocator();
                    // try { // try from 0010163e to 00101642 has its CatchHandler @ 00101b0d
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string
            ((char *)local_68,(allocator *)&DAT_0010201d);
  std::allocator<char>::~allocator(&local_249);
  std::allocator<char>::allocator();
                    // try { // try from 00101676 to 0010167a has its CatchHandler @ 00101b25
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::basic_string
            ((char *)local_48,(allocator *)&DAT_00102033);
  std::allocator<char>::~allocator(&local_249);
                    // try { // try from 00101699 to 0010185f has its CatchHandler @ 00101b3d
  pcVar2 = (char *)std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::
                   operator[]((ulong)local_208);
  if (*pcVar2 < 'B') {
    std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::operator+=
              (local_248,local_c8);
  }
  pcVar2 = (char *)std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::
                   operator[]((ulong)local_a8);
  if (*pcVar2 != 'A') {
    std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::operator+=
              (local_248,local_68);
  }
  pcVar2 = (char *)std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::
                   operator[]((ulong)local_1c8);
  cVar1 = *pcVar2;
  pcVar2 = (char *)std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::
                   operator[]((ulong)local_148);
  if ((int)cVar1 - (int)*pcVar2 == 3) {
    std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::operator+=
              (local_248,local_1c8);
  }
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::operator+=
            (local_248,local_1e8);
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::operator+=
            (local_248,local_188);
  pcVar2 = (char *)std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::
                   operator[]((ulong)local_168);
  if (*pcVar2 == 'G') {
    std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::operator+=
              (local_248,local_168);
  }
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::operator+=
            (local_248,local_1a8);
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::operator+=
            (local_248,local_88);
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::operator+=
            (local_248,local_228);
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::operator+=
            (local_248,local_128);
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::operator+=
            (local_248,'}');
  std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::~basic_string
            (local_48);
```

前半で文字列をメモリに当てこんでいて，後半でその各メモリの値をチェックしてフラグの文字列に追加するかどうかを判断している．
例えば以下

```C
pcVar2 = (char *)std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::
                   operator[]((ulong)local_208);
if (*pcVar2 < 'B') {
    std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::operator+=
              (local_248,local_c8);
  }
```

```local_208``` の値が ```<'B'``` なら，```local_c8 (2)``` を追加する，みたいな感じ．
最後に ```std::__cxx11::basic_string<char,std::char_traits<char>,std::allocator<char>>::operator+=(local_248,'}');``` があるのでここでおしまい．

### Keygenme

なんか libcrypto.so.1.1 が足りないって怒られたのでビルドする

```shell
wget https://www.openssl.org/source/old/1.1.0/openssl-1.1.0l.tar.gz
tar xvf openssl-1.1.0l.tar.gz
cd openssl-1.1.0l
./config
make
export LD_LIBRARY_PATH=./openssl-1.1.0l:$LD_LIBRARY_PATH
```

実行するとライセンスキーを求められる．
```printf("Enter your license key: ");``` みたいな記述がある関数があるのでこれが main ?

以下少し丁寧にやる

```C
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_98 = 0x7b4654436f636970;
  local_90 = 0x30795f676e317262;
  local_88 = 0x6b5f6e77305f7275;
  local_80 = 0x5f7933;
  local_ba = 0x7d;
                    /* 27words */
  sVar1 = strlen((char *)&local_98);
  MD5((uchar *)&local_98,sVar1,local_b8);
  sVar1 = strlen((char *)&local_ba);
  MD5((uchar *)&local_ba,sVar1,local_a8);
  local_d0 = 0;
  for (local_cc = 0; local_cc < 0x10; local_cc = local_cc + 1) {
    sprintf(local_78 + local_d0,"%02x",(ulong)local_b8[local_cc]);
    local_d0 = local_d0 + 2;
  }
  local_d0 = 0;
  for (local_c8 = 0; local_c8 < 0x10; local_c8 = local_c8 + 1) {
    sprintf(local_58 + local_d0,"%02x",(ulong)local_a8[local_c8]);
    local_d0 = local_d0 + 2;
  }
  for (local_c4 = 0; local_c4 < 0x1b; local_c4 = local_c4 + 1) {
    acStack_38[local_c4] = *(char *)((long)&local_98 + (long)local_c4);
  }
  acStack_38[27] = local_43;
  acStack_38[28] = local_62;
  acStack_38[29] = local_62;
  acStack_38[30] = local_78[0];
  acStack_38[31] = local_5b;
  acStack_38[32] = local_43;
  acStack_38[33] = local_6a;
  acStack_38[34] = local_60;
  acStack_38[35] = (undefined)local_ba;
  sVar1 = strlen(param_1);
  if (sVar1 == 0x24) {
    for (local_c0 = 0; local_c0 < 0x24; local_c0 = local_c0 + 1) {
      if (param_1[local_c0] != acStack_38[local_c0]) {
        uVar2 = 0;
        goto LAB_00101475;
      }
    }
    uVar2 = 1;
  }
```

local_98~~ みたいなとこがフラグの前半になっていて，残りのマジックナンバーを探そうという感じ．

```C
sVar1 = strlen((char *)&local_98);
MD5((uchar *)&local_98,sVar1,local_b8);
```

まず，```local_98 ~~``` には ```0x7b ~~``` のフラグが入っているので，最初は ```sVar1 = 17```
で，MD5 を計算している．
この結果は local_b8 から見れる．
逆に ```local_b1``` には ```0x7d``` しか入ってないので 1 文字だけ．

```C
local_d0 = 0;
  for (local_cc = 0; local_cc < 0x10; local_cc = local_cc + 1) {
    sprintf(local_78 + local_d0,"%02x",(ulong)local_b8[local_cc]);
    local_d0 = local_d0 + 2;
  }
  local_d0 = 0;
  for (local_c8 = 0; local_c8 < 0x10; local_c8 = local_c8 + 1) {
    sprintf(local_58 + local_d0,"%02x",(ulong)local_a8[local_c8]);
    local_d0 = local_d0 + 2;
  }
```

次にここを考える．
local_b8, local_a8 には既にハッシュ値が入っていて，local_78 の変数に格納していく．
ここで，local_78 は小さい方に伸びることを失念していて時間を使ってしまった．
つまり，local_78 -> local_76 みたいな感じで伸びていく．

```C
  for (local_c4 = 0; local_c4 < 0x1b; local_c4 = local_c4 + 1) {
    acStack_38[local_c4] = *(char *)((long)&local_98 + (long)local_c4);
}
```

ここは既存のフラグを代入しているだけ

```C
  acStack_38[27] = local_43;
  acStack_38[28] = local_62;
  acStack_38[29] = local_62;
  acStack_38[30] = local_78[0];
  acStack_38[31] = local_5b;
  acStack_38[32] = local_43;
  acStack_38[33] = local_6a;
  acStack_38[34] = local_60;
  acStack_38[35] = (undefined)local_ba;
```

マジックナンバーを入れるところ．
先に決めていたハッシュ値を見れば代入できる．

静的解析で頑張ったが，gdb でやったら 2 秒で終わった．
つまり，マジックナンバーを代入しているアドレスで止めて eax を見ればいい．

```C
001013c8 0f b6 45 c5     MOVZX      EAX,byte ptr [RBP + local_43]
001013cc 88 45 eb        MOV        byte ptr [RBP + local_1d],AL
001013cf 0f b6 45 a6     MOVZX      EAX,byte ptr [RBP + local_62]
001013d3 88 45 ec        MOV        byte ptr [RBP + local_1c],AL
001013d6 0f b6 45 a6     MOVZX      EAX,byte ptr [RBP + local_62]
```

静的解析，パズルで楽しいし筋トレにはなるけど眼の負担がエグいし動的解析が一瞬で終わった時虚無すぎる

### Wizardlike

マジでわかんね～と思いながら中見てたら ASCII コードが見えたので気合と根性．

### Classic Crackme 0x100

パスワード系の典型問題．
memcmp があるのでレジスタを見ればいいか，と思ったが，入力をこねくりまわしている & フラグがサーバ側にある，でちゃんと静的解析する必要がありそう

```C
  cmp_flag = 0x747774746971747a;
  local_60 = 0x7372667965697478;
  local_58 = 0x766f78757a74676c;
  local_50 = 0x6e7372626e64666c;
  local_48 = 0x647368687976726c;
  local_40 = 0x6e786f66727878;
  uStack_39 = 0x6c626a;
  setvbuf(stdout,(char *)0x0,2,0);
  printf("Enter the secret password: ");
  __isoc99_scanf(&DAT_00402024,user_input);
  local_c = 0;
  flag_length = strlen((char *)&cmp_flag);
  flag_length__ = (int)flag_length;
  local_18 = 0x55;
  local_1c = 0x33;
  local_20 = 0xf;
  local_21 = 'a';
  for (; local_c < 3; local_c = local_c + 1) {
    for (counter_input = 0; counter_input < flag_length__; counter_input = counter_input + 1) {
      local_28 = (counter_input % 0xff >> 1 & local_18) + (counter_input % 0xff & local_18);
      local_2c = ((int)local_28 >> 2 & local_1c) + (local_1c & local_28);
      iVar1 = ((int)local_2c >> 4 & local_20) +
              ((int)user_input[counter_input] - (int)local_21) + (local_20 & local_2c);
      user_input[counter_input] = local_21 + (char)iVar1 + (char)(iVar1 / 0x1a) * -0x1a;
    }
  }
  iVar1 = memcmp(user_input,&cmp_flag,(long)flag_length__);
```

重要なのは for を回している部分とハードコードされている情報はリトルエンディアンであること．
ソルバは以下

```python
import string

cmp_flag =[0x7a,0x74,0x71,0x69,0x74,0x74,0x77,0x74,0x78,0x74,0x69,0x65,0x79,0x66,0x72,0x73,0x6c,0x67,0x74,0x7a,0x75,0x78,0x6f,0x76,0x6c,0x66,0x64,0x6e,0x62,0x72,0x73,0x6e,0x6c,0x72,0x76,0x79,0x68,0x68,0x73,0x64,0x78,0x78,0x72,0x66,0x6f,0x78,0x6e,0x6a,0x62,0x6c]

local_18 = 0x55
local_1c = 0x33
local_20 = 0xf
local_21 = 'a'

user_input = [None] * 50
pattern = string.ascii_letters

for j in range(50):
   for chara in pattern:
      tmp_user_input = chara
      tmp_user_input = ord(tmp_user_input)
      for i in range(3):
            local_28 = (j % 0xff >> 1 & local_18) + (j % 0xff & local_18)
            local_2c = (local_28 >> 2 & local_1c) + (local_1c & local_28)
            iVar1 = (local_2c >> 4 & local_20) + (tmp_user_input - ord(local_21)) + (local_20 & local_2c)
            tmp_user_input = ord(local_21) + iVar1 + (iVar1 // 0x1a) * (-1*0x1a)
      if(hex(tmp_user_input) == hex(cmp_flag[j])):
         print(chara, end="")
         break
```

### WinAntiDbg0x100

Windows の実行ファイルをデバッグしてフラグを取ろう，みたいな感じらしい．
実行しながらやるといいらしいので勉強がてら IDA を使ってやってみる．

単純に IDA で実行すると以下のようなメッセージが出る

```text
Debugged application message: 

        _            _____ _______ ______  
       (_)          / ____|__   __|  ____| 
  _ __  _  ___ ___ | |       | |  | |__    
 | '_ \| |/ __/ _ \| |       | |  |  __|   
 | |_) | | (_| (_) | |____   | |  | |      
 | .__/|_|\___\___/ \_____|  |_|  |_|      
 | |                                       
 |_|                                       
  Welcome to the Anti-Debug challenge!

Debugged application message: ### Level 1: Why did the clever programmer become a gardener? Because they discovered their talent for growing a 'patch' of roses!

Debugged application message: ### Oops! The debugger was detected. Try to bypass this check to get the flag!
```

デバッガ検出の分岐があるっぽいので，メッセージから見て当該箇所を探す．

```asm
push    offset asc_6D3438 ; "\n"
call    ds:OutputDebugStringW
push    offset asc_6D343C ; "\n"
call    ds:OutputDebugStringW
call    sub_6D11B0
call    sub_6D1200
test    eax, eax
jnz     short loc_6D15E7
```

```test``` のところを右クリックすると ```Add breakpoint``` という表示があるのでブレークポイントを設置する．
で，実行すると止まってくれるので，IDA 側で ```eax``` を 0 に書き換えてあげればフラグが返ってくる．

### WinAntiDbg0x200

WinAntiDbg0x100 とほとんど同じで，分岐ポイントのフラグを書きかえればOK

### weirdSnake

python のデコンパイル結果が渡されるので手でデコンパイルする．

### breadth

二つの同じようなバイナリが与えられる．
diff を取ってみる．

```shell
objdump -d -M intel breadth.v1 > v1.txt
objdump -d -M intel breadth.v2 > v2.txt

diff v1-disassm.txt v2-disassm.txt 
2c2
< breadth.v1:     file format elf64-x86-64
---
> breadth.v2:     file format elf64-x86-64
164220,164225c164220,164225
<    95049:     48 8b 54 24 f0          mov    rdx,QWORD PTR [rsp-0x10]
<    9504e:     b8 3a 80 37 d0          mov    eax,0xd037803a
<    95053:     48 39 c2                cmp    rdx,rax
<    95056:     74 08                   je     95060 <fcnkKTQpF+0x20>
<    95058:     c3                      ret
<    95059:     0f 1f 80 00 00 00 00    nop    DWORD PTR [rax+0x0]
---
>    95049:     48 8b 44 24 f0          mov    rax,QWORD PTR [rsp-0x10]
>    9504e:     48 3d 3e c7 1b 04       cmp    rax,0x41bc73e
>    95054:     74 0a                   je     95060 <fcnkKTQpF+0x20>
>    95056:     c3                      ret
>    95057:     66 0f 1f 84 00 00 00    nop    WORD PTR [rax+rax*1+0x0]
>    9505e:     00 00 
```

当該部分にハードコードされているフラグが答えだった

飽きてきたので他の CTF の rev を解こうと思います
