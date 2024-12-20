---
title: "Rev を雑多に解く1"
date: 2024-05-06T05:03:30+09:00
draft: false
---

# SECCON Beginners 2023

### Half

```strings``` で見える

```ctf4b{ge4_t0_kn0w_the_bin4ry_fi1e_with_s4ring3}```

### Three

```C
  length = strlen(input);
  if (length == 0x31) {
    for (local_c = 0; local_c < 0x31; local_c = local_c + 1) {
      if (local_c % 3 == 0) {
        cVar1 = (char)*(undefined4 *)(flag_0 + (long)(local_c / 3) * 4);
      }
      else if (local_c % 3 == 1) {
        cVar1 = (char)*(undefined4 *)(flag_1 + (long)(local_c / 3) * 4);
      }
      else {
        cVar1 = (char)*(undefined4 *)(flag_2 + (long)(local_c / 3) * 4);
      }
      if (cVar1 != input[local_c]) {
        puts("Invalid FLAG");
        return 1;
      }
    }
    puts("Correct!");
    uVar2 = 0;
  }
  else {
    puts("Invalid FLAG");
    uVar2 = 1;
  }
```

パスワード比較の部分がある．
flag_0 とか flag_1 を見ればいいが，一文字ずつメモるのが大変すぎるので色々調べると python スクリプトがいけるらしい

```python
script = open('./three', 'rb').read()

flag_0 = 0x00102020 - 0x00100000
flag_1 = 0x00102080 - 0x00100000
flag_2 = 0x001020c0 - 0x00100000

flag = ''

for i in range(0x31):
    if(i%3 == 0):
        flag += chr(script[flag_0 + (i//3)*4])
    if(i%3 == 1):
        flag += chr(script[flag_1 + (i//3)*4])
    if(i%3 == 2):
        flag += chr(script[flag_2 + (i//3)*4])

print(flag)
```

```ctf4b{c4n_y0u_ab1e_t0_und0_t4e_t4ree_sp1it_f14g3}```

### Poker

```C
    if (99 < score) break;
    local_10 = local_10 + 1;
```

比較部分はここ

```asm
001022ae 89 45 fc        MOV        dword ptr [RBP + score],EAX
001022b1 83 7d fc 63     CMP        dword ptr [RBP + score],0x63
001022b5 7e 0c           JLE        LAB_001022c3
001022b7 e8 e4 ee        CALL       FUN_001011a0                                     undefined FUN_001011a0()
```

EAX の値が 0x63 より大きければいい

```C
pwndbg> set $rax = 0x64
```

```ctf4b{4ll_w3_h4v3_70_d3cide_1s_wh4t_t0_d0_w1th_7he_71m3_7h47_i5_g1v3n_u5}```

### Leak

指定した IP アドレスに接続し，/tmp/flag からバイト列を読み取る．
これをなんらかで暗号化し，送信している．

配布された .pcap で TCP follow すると hex 列が見えるのでコピーして，ローカルの /tmp/flag に代入する．
また，暗号化は XOR なので，暗号化された hex 列をまた代入してあげれば flag が再現される．

なので，ローカルの ```nc -nlvp 5000``` で待ち，もう一方で ```./leak``` を実行すればいい．

```python
data = bytes.fromhex('8e57ff5945da900628b2abfa497332334a7329413c34b7f66273250f954016fa47e9228da5cd3d53eeb4b3518ed289935be059cbfbb11b')

f = open('/tmp/flag', 'wb')
f.write(data)
f.close()
```
