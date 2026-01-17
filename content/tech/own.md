---
title: "自作コンテナ1"
date: 2023-09-13T18:40:52+09:00
draft: false
---

缶コーヒー，飲めたことない

## 自作コンテナ

簡単に復習だけ

### namespace

カーネルのリソースを隔離するためのしくみ

- PID: PID は新しいプロセスができるたびにインクリメントされた整数がプロセスの識別子として付与される．
コンテナの中では独立した PID を見せたいが，同じ PID をもったプロセスが複数できると不便．
- MNT: マウントポイントに関するリソースの隔離．他の namespace に影響を与えることなくマウント，案マウントをすることができる
- NET: プロセスのネットワークスタックを分離する．メインのネットワーク namespace の他に仮想イーサネットペアを作り，namespace 上に仮想リンクを作る
- UTS: UTS namespace でホスト名とドメイン名を分離する．
- IPC: IPC に関するリソースを分離する．POSIX メッセージキュー
- USER: namespace ごとに UID/GID をマッピングすることができる．プロセスは，namespace の外では 0 以外の UID を持つ一方で，namespace では UID を 0 として持つことができ，root のように振舞うことができる．

### CGroup

システムリソースを使うプロセスをグループ化して，それぞれのグループで
リソースの使用量をコントロールできる．

### Layer filesystem

namespace と cgroup はコンテナがもつシステムの分離とリソース共有
の役割を果たすが，レイヤー layered filesystems は
マシンイメージ全体を効率的に移動することができる．
コンテナの rootfs はレイヤ構造で管理されている．
イメージは特定のファイルシステムに依存しない形で定義されていて，簡単に移植できる．

## 骨組み

```go
func main() {
 switch os.Args[1] {
 case "run": // 引数に run をとると execute を実行できる
  execute(os.Args[2], os.Args[3:]...)
 default:
  panic("you neer argument")
 }
}

func execute(cmd string, args ...string) {
 must(unix.Chroot("./rootfs")) // ./rootfs を root にする
 must(unix.Chdir("/")) // root(rootfs) に移動

 command := exec.Command(cmd, args...) // run の引数を実行する
 command.Stdin = os.Stdin
 command.Stdout = os.Stdout
 command.Stderr = os.Stderr

 must(command.Run()) // ねんのため
}

func must(err error) {
 if err != nil {
  panic(err)
 }
}
```

このままだと rootfs にバイナリがないので何も起こらない
（というかシェルが立ち上がらない）．
ので docker export した tar を rootfs に解凍する

```bash
ubuntu@ubuntu:~/build_container$ ls rootfs/

bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

```

```bash
ubuntu@ubuntu:~/build_container$ sudo go run main.go run sh

# ls
bin  boot  dev etc  home  lib lib32  lib64  libx32  media  mnt  opt  proc  root  run sbin  srv  sys tmp  usr  var

# pwd
/

# whoami
root

# touch /tmp/sample

# exit

ubuntu@ubuntu:~/build_container$ ls ./rootfs/tmp
sample
```

### namespace の実装

Go で linux namespace を設定するのは，
cmd の SysprocAttr に Cloneflags をセットすればいいらしい．
でも proc のマウント(ps のために必要)であったりホスト名の設定，
とかをあらかじめする必要がある．
そこで， namespace を設定した子プロセス上でマウントとかコマンドの実行を行う．

```go
command.SysProcAttr = &unix.SysProcAttr{
  Cloneflags: unix.CLONE_NEWNS | unix.CLONE_NEWPID,
 }
```

```bash
# namespace があるバージョン
ubuntu@ubuntu:~/build_container$ sudo go run main.go run sh
chroot...
Mount...
exec...
# ps
    PID TTY          TIME CMD
      1 ?        00:00:00 exe
      6 ?        00:00:00 sh
      7 ?        00:00:00 ps

# exit

# namespace がないバージョン
ubuntu@ubuntu:~/build_container$ sudo go run main.go run sh
chroot...
Mount...
exec...
# ps
    PID TTY          TIME CMD
  28429 ?        00:00:00 sudo
  28430 ?        00:00:00 go
  28494 ?        00:00:00 main
  28499 ?        00:00:00 exe
  28504 ?        00:00:00 sh
  28508 ?        00:00:00 ps

# exit

```
