---
title: "Boiler"
date: 2024-03-28T00:22:47+09:00
draft: false
---

## user

- FTP anonymous login
  - .info.txt
    - rabbit hole
- webmin
  - searchsploit
    - 特になし
- CMS
  - Joomla
    - Joomla 3.8
    - joomla-brute とかいろいろ試したけどクレデンシャルが手に入らず RCE 断念
    - エンドポイントにクレデンシャルが露出しているらしいがアクセスできなかった(ref: [How to bypass the admin login page in Joomla & RCE](https://systemweakness.com/how-bypass-admin-login-page-in-joomla-rce-d853d621ee0e))
  - gobuster をさらにかけると ./_test がアクセスできる
    - それ以外は rabbit hole(いい加減にしてくれ...)
  - sar2html が動いている
    - searchsploit で探すと RCE ができるらしい
    - ```ls``` で log.txt が見つかる
    - ```cat log.txt``` でクレデンシャルが見つかる
    - basterd:superduperp@$$
    - ssh する
- ssh
  - ```backup.sh``` が見つかる
  - パスワードがハードコードしてある
  - stoner にユーザを変える
  - ```.secret``` が見つかる

## root

- SUID
  - ```find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null```
  - ```find``` が使えるらしい
  - ```/usr/bin/find . -exec /bin/sh -p \; -quit```
