---
title: "pivot_root の条件"
date: 2023-08-30T00:01:58+09:00
draft: true
---

pivot_root は Linux プロセスのルートファイルシステムを変更することができる．
古典的には chroot でも実現できるが脱獄できるので，Docker とかでは pivot_root がデフォルトで使われている．

