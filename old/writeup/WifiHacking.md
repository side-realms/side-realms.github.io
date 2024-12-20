---
title: "WifiHacking 101"
date: 2024-02-20T02:06:20+09:00
draft: false
---

BOØWY，空前の大ブーム

### aircrack-ng

- 4way-handshakeをキャプチャした pcap が得られるので，これをクラックすればいい
- Aircrack-ng ~~.cap -w ~~.txt で総当たりができるらしいが遅いので，HCAAPX ファイルに変換する方法が使えるらしい
  - ```aircrack-ng -j wifi ~~.cap```
  - ```aircrack-ng -a2 -b BSSID -w ~~.txt  ~~.hccapx```
- え，終わった．．．

手元の環境でもやりたい
