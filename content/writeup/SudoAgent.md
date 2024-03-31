---
title: "Agent Sudo"
date: 2024-02-17T20:49:48+09:00
draft: false
---

そういえばLerningしかやってなくてマシンを解いてないことに気付いたのでせっかくなら writeup を書きつつやっていく

### enum

- nmap

``` shell
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 ef:1f:5d:04:d4:77:95:06:60:72:ec:f0:58:f2:cc:07 (RSA)
|   256 5e:02:d1:9a:c4:e7:43:06:62:c1:9e:25:84:8a:e7:ea (ECDSA)
|_  256 2d:00:5c:b9:fd:a8:c8:d8:80:e3:92:4f:8b:4f:18:e2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Annoucement
|_http-server-header: Apache/2.4.29 (Ubuntu)
```

- gobuster

``` shell
/.hta                 (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/index.php            (Status: 200) [Size: 218]
/server-status        (Status: 403) [Size: 278]
```

### user.txt

- index.php にアクセスすると User-agent を変える，みたいな旨のメッセージがある
- burp のブルートフォースで「C」が見つかる
- agent_C_attention.php で chris が見つかる

- とりあえず FTP で hydra をする
  - ```hydra -t 16 -l chris -P /usr/share/wordlists/rockyou.txt 10.10.155.104 ftp```
- 画像が二枚手に入る
- binwalk で cutie.png から ZIP が見つかる（cute-alien.jpg は無反応）
  - ```dd ibs=1 obs=1 skip=34562 if=cutie.png of=hidden.zip```
- ZIP にパスワードがかかっている
  - ```zip2john hidden.zip > hash.txt```
  - ```john hash.txt```
- To_agentR.txt が見つかり，steg passphrase なるものが見つかる
  - ```steghide extract -sf cute-alien.jpg -p Area51```
- ssh のパスワードが見つかる
- ssh ログインする（最後に ！ が必要．正解との整合性がない．．．）
- user_flag を手に入れる
- 画像があるので google 検索（怖い画像だった．．．）

### root.txt

- linpeas.sh で (ALL, !root) /bin/bash が見つかる
- CVE : 2019-14287 がみつかる
- あとは PoC でOK

パズルみたいだ．
道具を使っただけでここまでできてしまうのは逆に不安でもある
