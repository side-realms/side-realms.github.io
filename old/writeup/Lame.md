---
title: "Lame"
date: 2024-05-22T21:27:57+09:00
draft: false
---

### user

- enum4linux

```shell
        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        tmp             Disk      oh noes!
        opt             Disk      
        IPC$            IPC       IPC Service (lame server (Samba 3.0.20-Debian))
        ADMIN$          IPC       IPC Service (lame server (Samba 3.0.20-Debian))
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            LAME
```

- tmp で anonymous ログインができる
- ファイルを見てみたがよく分からなかった
  - vmware の authlog のようなものがある
- samba 3.0.20 がある
  - metasploit で調べると exploit が見つかる
  - 刺さる

### root

- metasploit の exploit で root ログインだったので root も手に入れた
