---
title: "SimpleCTF"
date: 2024-02-21T02:31:08+09:00
draft: false
---

デュエマ，ゼニスというか昔のキャラが復活しているらしく，非常にやりたい\
動画見たら 4t ゼニスとかあって笑った

### enum

- nmap
  - 21/ftp
  - 80/http
  - 2222/ssh
- gobuster
  - simple

### CVE

- CMS 2.2.8 で構成されたサイトが見つかる
- CVE-2019-9053 で SQLi が通るっぽい
- PoC を指してuser name と password を手に入れる

### ssh

- user.txt が手に入る
- sudo -l で vim がパスワード無しで使えるらしい
  - ```sudo vim -c ':!/bin/sh'```
- root.txt が手に入る
