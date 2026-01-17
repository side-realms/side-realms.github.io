---
title: "Namespace"
date: 2023-08-30T10:04:30+09:00
draft: false
---

モンエナ スイカ味，ほんとうにまずい

## namespace

namespace は，プロセスをグループ化して隔離空間を作り出すのに一役買っている．
namaspace 大まかな機能があるわけではなく，隔離したいリソースごとに機能が分離している．
(mount namespace, PID namespace とか)

OS 起動時にはデフォルトの namespace が存在し，全てのプロセスが属している．
このうえで，そのリソースを隔離させたいかを指定してコンテナを作る．

### unshare

unshare --mount とか unshare --pid みたいな感じで namespace を作る．
ネットワーク以外はこれで隔離できるらしい
各プロセスが動作している namespace に関する情報は /proc/[PID]/ns で見れる．

![This is a image](/images/proc.png)

ここに表示されている数字が同じなら同じ namespace に属している．
この /proc/[PID]/ns は，プロセスを既存の namespace で動かしたいときに使うこともある．
setns は既存の namespace でプロセスを動かすことを実現するが(<https://tenforward.hatenablog.com/entries/2014/08/14>)，このときの引数にさっきの ns 指定される．


## mount namespace

ファイルシステムを特定のディレクトリに関連付けるための処理をマウントといい，
マウントが行われるディレクトリをマウントポイントという．
あるパーティション(例えば /dev/sdc1)にファイルシステムを作ってルートとしてマウントした場合，
ルートがマウントポイントになる．
この下に別のファイルシステムをマウントすることもできる．

mount namespace は，その namespace 内のプロセスから見えるマウントポイントを分離する．
なので，ある mount namespace から別の mount namespace を見たときにそのマウントポイントが見えない．
コンテナごとに独立したファイルシステムを使うことができる．

/root/hosts ファイルをバインドマウント先として /etc/hosts をマウントする．
ファイルを確認すると同じ内容が見えることが分かる．

![This is a image](/images/mount.png)

/proc/[PID]/mountinfo でもマウントされていることが分かる．

![This is a image](/images/mount2.png)

別の namespace でも見てみると /root/hosts はマウントされていない．

```bash
#:~$ ls -l /root/hosts
-rw-r--r-- 1 root root 0 Aug 30 02:33 /root/hosts
#:~$ cat /root/hosts
#:~$
```

なので， unshare で作った namespace 内のシェルではマウントされていることが分かるが，
別の namespace ではマウントされていることは分からなかった．

### pivot_root を使ったミニコンテナ

LXC でコンテナを作っておく

```bash
lxc-create --name=ubuntu --template=download
```

![This is a image](/images/rootfs.png)

unshare で mount namespace をつくる．
次に pivot_root で新しいファイルシステムを root にしたいので，バインドマウントでマウント用のディレクトリを作る．
(今回は new_mount_point)
最後に前の root をマウントする用のディレクトリを作って(old_root)，pivot_root する

![This is a image](/images/rootfs2.png)

ルートを echo してみると確かにコンテナのルートになっていることがわかる．
unmount old_root する必要がある．

![This is a image](/images/rootfs3.png)

unmount するとコンテナだけが残る．
ホストのマウントは見えていないので，mount namespace が作られていることが分かる．

```bash
root@ubuntu:/home/ubuntu/new_mount_point# cat /proc/self/mounts
/dev/sda3 / ext4 rw,relatime,errors=remount-ro 0 0
proc /proc proc rw,nosuid,nodev,noexec,relatime 0 0
```

### mountinfo

### マウントプロパゲーション

マウントしたときにそのマウントが他のマウントに影響するかどうかを設定することができる．

仮に shered を試してみる．
base と bind というディレクトリを作り，base を bind にバインドマウントする．
base に作ったディレクトリが bind 側からも見えている．

![This is a image](/images/bind.png)

tmpfs で一時的なファイルシステムを作って，bind/tmp に tmpfs マウントする．
bind/tmp 以下に新しいファイルを作ってみると，base 以下からも見えるようになる．
これは bind を shered にしたため，サブマウントがマウントに影響するようになったため．

![This is a image](/images/bind2.png)

他にあるのは

- private: マウントが波及しない
- slave: master -> slave 側は波及するが，逆は波及しない
- unbindable: マウントが波及しないだけでなく，バインドマウントを禁止する．DoS攻撃を防ぐ意味合いがある．

また，mount namespace を作った時の動作もみてみる．

shared と private ディレクトリを作って，マウントプロパゲーションをそれぞれ private, shared にする．

![This is a image](/images/mount3.png)

この上で unshare して namespace を作る

```bash
root@ubuntu:/home/ubuntu# unshare --mount --propagation unchanged /bin/bash
```

private と shared 配下に tmpfs をマウントしてみる．
同一 namespace からは priv も shrd も見える

![This is a image](/images/mount4.png)

違う namespace からは private が見えない．

![This is a image](/images/mount5.png)

逆からもやってみる

![This is a image](/images/mount6.png)

private がみえない

![This is a image](/images/mount7.png)

### UTS namespace

ホスト名とかドメインとかを隔離する

### IPC namespace

IPC リソースを隔離する．
IPCはプロセス間で行われるデータ通信のこと．

### PID namespace

PID を隔離する．
PID はプロセスに固有の番号なので一意に決まるが，隔離するとコンテナごとに PID をふることができる．
一方でホストからは一意の番号を見ることができ，二つの番号をもっていることになる(?)

### Network namespace

コンテナとホストで同一IPアドレス，同一ポートで待ち受けることができる
