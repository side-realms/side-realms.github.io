---
title: "seccomp バイパス"
date: 2023-09-02T15:42:29+09:00
draft: true
---

新しく買ったヘッドホンのサイズが微妙に合わず，涙

## seccomp

seccomp は自プロセスが発行するシステムコールを制限することができる．
なので，仮に侵害された場合でも許可されたシステムコール以外は発行できなくなる．

<https://www.kernel.org/doc/Documentation/prctl/seccomp_filter.txt>
