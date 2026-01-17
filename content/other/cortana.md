---
title: "YouTube を開くと叱ってくれるスクリプト"
date: 2023-08-28T17:57:25+09:00
# weight: 1
# aliases: ["/first"]
# tags: ["first"]
# author: "Me"
# author: ["Me", "You"] # multiple authors
#showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: ""
canonicalURL: "https://canonical.url/to/page"
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
# ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: false
# ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
#editPost:
#    URL: "https://github.com/<path_to_repo>/content"
#    Text: "Suggest Changes" # edit text
#    appendFilePath: true # to append file path to Edit link
---

YouTube 中毒なのでどうにかしたい

[ソース](https://github.com/side-realms/Dont-watch-youtube)

ウィンドウのステータスとか，マウスの位置みたいなパラメータは python では直接触ることができなくて，
dll 経由で操作する必要がある．
cdll の呼び出し規約は cdeclで， windll の呼び出し規約は stdcalなので，
スタックを関数がクリーンアップするか，呼び出し元がするかに
注意する必要がある（たぶん）
今回は WindowsAPI なので stdcall

_get_running_window() で開いているウィンドウ(プロセス)の情報を取得する．
無限ループで回し続けて，「Youtube」が含まれる(表記ゆれふくむ)プロセスを見つけたら，
mouse_move_close() でマウスを動かしてウィンドウを閉じる．
本来ならウィンドウを直接終了することもできるが，マウスの操作もしたいので今回はこれでいい

あとはトースト通知を出して，Cortana を呼び出しておしまい．Cortana 呼び出せるの知らなかったな
