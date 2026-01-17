---
title: "OpenCV と python で顔トラッキングする"
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
#editPost:
#    URL: "https://github.com/<path_to_repo>/content"
#    Text: "Suggest Changes" # edit text
#    appendFilePath: true # to append file path to Edit link
---

Vtuber みたいなことをできるようにしてみたい．
顔認識ができればいいのか？

### 単純なマッチング

まずは単純に動画内を移動するアイコンをマッチングできるようにする.
動くプレミアムボールに対してトラッキングし，判定した場所に長方形を描く

[ソース](https://github.com/side-realms/image-processing/blob/main/tracking2.py)

![traking](/images/tracking.png)

### 顔認識

openCV で顔認識ができるらしい．
既存の学習セットがあったのでこれを食べさせればできる．(下記は photoAC の素材で顔判定をしたもの)

[ソース](https://github.com/side-realms/image-processing/blob/main/tracking2.py)

![traking](/images/result.jpg)

### カメラで顔認識

opencv でデバイスを開くようにすれば同じように検証できるらしい．
顔をトラッキングし続けて，顔を認識したらバックベアードのイラストを出すようにした

[ソース](https://github.com/side-realms/image-processing/blob/main/tracking4.py)
