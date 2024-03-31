---
title: "Chocolate"
date: 2024-03-16T13:28:17+09:00
draft: false
---

### user

- FTP に anonymous ログイン
  - steghide で引き出すと base64 エンコードされたパスワードリストがもらえる
- nmap したときに localhost うんぬん言われたところにアクセスすると key_rev_key をもらえる
  - strings
- charlie とパスワードで http にログインするとコードを実行できるフォームが見つかる
  - ```cat /home/charlie/teleport``` で ssh 鍵をもらう
  - ssh
  
### root

- vi が使える
  - ```sudo vi -c ':!/bin/sh' /dev/null```
