---
title: "kubernetes"
date: 2023-09-03T14:14:38+09:00
draft: false
---

## kubernetes

コンテナを管理，スケールさせるためのツール（コンテナオーケストレーションツール）
<https://kubernetes.io/docs/concepts/overview/#why-you-need-kubernetes-and-what-can-it-do>

kubernetes では負荷分散やストレージオーケストレーション，セキュリティなどが機能としてある．
基本的に kubectl で kubernetes クラスタを操作する．

{{< figure src="https://www.cncf.io/wp-content/uploads/2020/09/Kubernetes-architecture-diagram-1-1-1024x698.png" caption="https://www.cncf.io/wp-content/uploads/2020/09/Kubernetes-architecture-diagram-1-1-1024x698.png" >}}

Contorol Plane は Node(Pod が実行されるマシンで，コンテナをまとめた単位)を制御する

### Control Panel

- API server: kubernetes クラスタを操作するための REST interface を提供する．Pod やサービスに対する操作はエンドポイントでプログラムされている
- Scheduler: リソース容量を監視して，Node のパフォーマンスが最適になるように管理する
- Controller manager: Node がダウンしたなどのイベントに対して宣言との差異を確認する
- kubelet: コンテナが実行されていることを確認するために pod の状態を追跡する
- kube proxy: サービスから Node に流入するトラフィックを管理する
- etcd: クラスタの状態を保存する


### Component

- Pod: コンテナをまとめるグループで，kubernetes の最小単位．Pod には IP アドレスが割り当てられていて，同じコンテナの Pod は同じリソース（メモリなど）を共有する．
- Deployment: Node 上で動く Pod の管理をする．希望する数の Pod を常に動かしておくなど
- Servise: 個々の Pod において，IP アドレスなど多くのものが生え変わりのたびに変化するので，このアドレスのルーティングなどを行う．Pods はユニークな存在ではないので，ダウンタイムを少なくする工夫が必要
- Ingress: 負荷分散．クラスタ外から来る通信をロードバランサによって制御する．制御されたトラフィックを service にルーティングする．

## やってみる

とりあえず windwos に minikube をインストールする．
今回は virtualbox を使う

```bash
C:\k8s_play>kubectl get nodes
NAME       STATUS   ROLES           AGE     VERSION
minikube   Ready    control-plane   7d22h   v1.27.3
```

よさそう．
次に pod を作成する manifest ファイルを作る

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```

```bash
C:\k8s_play>kubectl create -f pod-nginx.yaml
pod/nginx created

C:\k8s_play>kubectl get pod
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          7s

C:\k8s_play>kubectl delete pod/nginx
pod "nginx" deleted

C:\k8s_play>kubectl get pod
No resources found in default namespace.
```

pod が変化しているのでよさそう．

#### manifest component

- spec: オブジェクトに期待する状態を決める．name は名前だし image は使うイメージ
- container onject
  - command: コンテナで起動させたいプロセスを指定する．
  - args: 特に引数を指定する（と思う）
  - env: 環境変数を指定する
  - image: コンテナイメージ
  - name: コンテナ名
  - ports: コンテナが開くポート
  - workingDir: ワーキングディレクトリ

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    imagePullPolicy: Always
    command: []
    args: ["nginx", "-g", "daemon off;"]
    args:
    env:
    - name: woot
      value: wai
    ports:
    - containerPort: 80
      protocol: TCP
    workingDir: /tmp
```

```bash
C:\k8s_play>kubectl create -f pod-nginx.yaml
pod/nginx created

C:\k8s_play>kubectl get all
NAME        READY   STATUS              RESTARTS   AGE
pod/nginx   0/1     ContainerCreating   0          3s

C:\k8s_play>kubectl get all
NAME        READY   STATUS    RESTARTS   AGE
pod/nginx   1/1     Running   0          7s

C:\k8s_play>kubectl exec -- nginx ps
PID   USER     TIME  COMMAND
    1 root      0:00 nginx: master process nginx -g daemon off;
   30 nginx     0:00 nginx: worker process
   31 nginx     0:00 nginx: worker process
   32 root      0:00 ps

C:\k8s_play>kubectl exec nginx -- env
...
woot=wai
...
```

### Volume

コンテナはコンテナが削除されたり，再起動した場合に中身が消えてしまう．
Volume はデータの永続化のために使う．
また，コンテナ間のデータ共有にも使うことができる．
クラウドみたいなものだろうか...

永続化に使う volume タイプにはいくつかある

- emptyDir: データの永続化には使えないが，コンテナ間のデータ共有に使う
- hostDir: データの永続化にも使える

```yaml
  volumes:
  - name: log-volume
    emptyDir:    
```

emptyDir を使った volume を作って，これにコンテナを対応させる

```yaml
    volumeMounts:
    - name: log-volume
      mountPath: /var/log/date-tail
```

これで両方のコンテナから deta-tail が参照できるようになる．
全然関係ないけど大文字小文字分かりづらくて泣く，なんで語頭は小文字なんだ

### init Container

pods の contaieners で指定したコンテナが起動する前に初期化する．
例えばあるコンテナ A でファイルが必要だが，もう一つのコンテナ B がその作成を担っているとき，
A が先に起動してしまうとファイルが無くて失敗してしまうが，これを init Container で防ぐ．

### Resource Requirements

pod は CPU や GPU リソースなど pod 内のコンテナに対してリソースの要求や制限を指定することができる
例えば以下

```yaml
resources:
  requests:
    cpu: 4
    memory: 128M
  limits:
    cpu: 0.5
    memory: 64M
```

- 要求：コンテナで使いたいリソース量で，確保されていてほしい量
- 制限：コンテナで超えてほしくないリソース量

### security context

pod やコンテナに対してアクセス制御やユーザの制限などを行うことができる．
例えば以下は volume のオーナーやパーミッションを指定することができる．

```yaml
securityContext:
    fsGroup: 1000
```

```bash
C:\k8s_play>kubectl apply -f nginx.yaml
pod/fsgroup-test created

C:\k8s_play>kubectl exec -it fsgroup-test -- ls -l /
/:
total 60
drwxr-xr-x    2 root     root          4096 Aug  7 13:09 bin
drwxrwsrwx    2 root     1000          4096 Sep  3 10:15 data <--
drwxr-xr-x    5 root     root           360 Sep  3 10:15 dev
drwxr-xr-x    1 root     root          4096 Sep  3 10:15 etc
drwxr-xr-x    2 root     root          4096 Aug  7 13:09 home
drwxr-xr-x    7 root     root          4096 Aug  7 13:09 lib
drwxr-xr-x    5 root     root          4096 Aug  7 13:09 media
drwxr-xr-x    2 root     root          4096 Aug  7 13:09 mnt
drwxr-xr-x    2 root     root          4096 Aug  7 13:09 opt
dr-xr-xr-x  257 root     root             0 Sep  3 10:15 proc
drwx------    2 root     root          4096 Aug  7 13:09 root
drwxr-xr-x    1 root     root          4096 Sep  3 10:15 run
drwxr-xr-x    2 root     root          4096 Aug  7 13:09 sbin
drwxr-xr-x    2 root     root          4096 Aug  7 13:09 srv
dr-xr-xr-x   12 root     root             0 Sep  3 10:15 sys
drwxrwxrwt    2 root     root          4096 Aug  7 13:09 tmp
drwxr-xr-x    7 root     root          4096 Aug  7 13:09 usr
drwxr-xr-x   12 root     root          4096 Aug  7 13:09 var
```

### Deployment

Pod の rolling update や rollback などを行う．
ReplicaSet を管理することで，Pod を指定された数（Replica 数）調整する．
足りない場合は追加し，多い場合は削除する．
この仕組みで Node の障害やクラッシュで pod が足りなくなった場合も自動的に Pod が追加される．

Deployment はコンテナイメージのアップデートがあった場合に
新しい仕様の ReplicaSet を作成し，全体の Pod を調整しながら新しい仕様の Pod に置き換える
（ローリングアップデート）．
Recreate の場合は全ての Pod を削除してから新しい Pod Template をもとに Pod を作成する．


```yaml
selector:
    matchLabels:
      app: nginx
  strategy:
    rollingUpdate:
      maxSurge: 50% # 複製数を超えていい許容量
      maxUnavailable: 0%
  template:
    metadata:
      labels:
        app: nginx
```

```bash
C:\k8s_play>kubectl get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   2/2     2            2           1s

C:\k8s_play>kubectl get rs # ReplicaSet
NAME               DESIRED   CURRENT   READY   AGE
nginx-77b4fdf86c   2         2         2       9s

C:\k8s_play>kubectl get po # Pod
NAME                     READY   STATUS    RESTARTS   AGE
nginx-77b4fdf86c-qkmz2   1/1     Running   0          15s
nginx-77b4fdf86c-rdfv5   1/1     Running   0          15s
```

manifest の内容を変えて更新してみる

```bash
C:k8s_play>kubectl apply -f nginx.yaml
deployment.apps/nginx configured

C:k8s_play>kubectl get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   2/2     0            2           2m9s

C:k8s_play>kubectl get rs
NAME               DESIRED   CURRENT   READY   AGE
nginx-77b4fdf86c   2         2         2       2m22s

C:k8s_play>kubectl get po # pause で止めているので特に変化はない
NAME                     READY   STATUS    RESTARTS   AGE
nginx-77b4fdf86c-qkmz2   1/1     Running   0          2m35s
nginx-77b4fdf86c-rdfv5   1/1     Running   0          2m35s

C:k8s_play>kubectl rollout resume deploy nginx
deployment.apps/nginx resumed

C:k8s_play>kubectl get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   2/2     2            2           3m45s

C:k8s_play>kubectl get rs # 77b4fdf86c -> 7544d696d4
NAME               DESIRED   CURRENT   READY   AGE
nginx-7544d696d4   2         2         2       28s
nginx-77b4fdf86c   0         0         0       3m51s

C:k8s_play>kubectl get po # 完全に置き換わっている
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7544d696d4-6s2tb   1/1     Running   0          25s
nginx-7544d696d4-bqblr   1/1     Running   0          29s

C:k8s_play>kubectl rollout history deploy nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

下記は rollout しているときのログ
旧：qkmz2
旧：rdfv5
新：6s2tb
新：bqblr

```bash
C:\k8s_play> kubectl get po -w
NAME                     READY   STATUS    RESTARTS   AGE
nginx-77b4fdf86c-qkmz2   1/1     Running   0          3m8s
nginx-77b4fdf86c-rdfv5   1/1     Running   0          3m8s # <- 旧が動いている
nginx-7544d696d4-bqblr   0/1     Pending   0          0s # <- 新が用意される
nginx-7544d696d4-bqblr   0/1     Pending   0          0s
nginx-7544d696d4-bqblr   0/1     ContainerCreating   0          0s
nginx-7544d696d4-bqblr   1/1     Running             0          3s # <- maxSurge が 50% なので新しいものが一つ追加される
nginx-77b4fdf86c-qkmz2   1/1     Terminating         0          3m26s # <- 旧が一つ止まる
nginx-7544d696d4-6s2tb   0/1     Pending             0          0s # <- 新が用意される
nginx-7544d696d4-6s2tb   0/1     Pending             0          0s
nginx-7544d696d4-6s2tb   0/1     ContainerCreating   0          0s
nginx-77b4fdf86c-qkmz2   0/1     Terminating         0          3m27s
nginx-77b4fdf86c-qkmz2   0/1     Terminating         0          3m28s
nginx-77b4fdf86c-qkmz2   0/1     Terminating         0          3m28s
nginx-77b4fdf86c-qkmz2   0/1     Terminating         0          3m28s
nginx-7544d696d4-6s2tb   1/1     Running             0          3s
nginx-77b4fdf86c-rdfv5   1/1     Terminating         0          3m30s
nginx-77b4fdf86c-rdfv5   0/1     Terminating         0          3m30s
nginx-77b4fdf86c-rdfv5   0/1     Terminating         0          3m31s
nginx-77b4fdf86c-rdfv5   0/1     Terminating         0          3m31s
nginx-77b4fdf86c-rdfv5   0/1     Terminating         0          3m31s
```

rollback する

```bash
C:\k8s_play>kubectl rollout undo deploy nginx --to-revision 1
deployment.apps/nginx rolled back

C:\k8s_play>kubectl get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   2/2     2            2           9m55s

C:\k8s_play>kubectl get rs
NAME               DESIRED   CURRENT   READY   AGE
nginx-7544d696d4   0         0         0       6m33s
nginx-77b4fdf86c   2         2         2       9m56s

C:\k8s_play>kubectl get po
NAME                     READY   STATUS    RESTARTS   AGE
nginx-77b4fdf86c-6b7sm   1/1     Running   0          14s
nginx-77b4fdf86c-fbwgf   1/1     Running   0          10s
```

ちゃんと古い ReplicaSet を使っている

```bash
nginx-77b4fdf86c-6b7sm   0/1     Pending             0          0s
nginx-77b4fdf86c-6b7sm   0/1     Pending             0          0s
nginx-77b4fdf86c-6b7sm   0/1     ContainerCreating   0          1s
nginx-77b4fdf86c-6b7sm   1/1     Running             0          4s
nginx-7544d696d4-bqblr   1/1     Terminating         0          6m24s
nginx-77b4fdf86c-fbwgf   0/1     Pending             0          0s
nginx-77b4fdf86c-fbwgf   0/1     Pending             0          0s
nginx-77b4fdf86c-fbwgf   0/1     ContainerCreating   0          0s
nginx-7544d696d4-bqblr   0/1     Terminating         0          6m25s
nginx-7544d696d4-bqblr   0/1     Terminating         0          6m25s
nginx-7544d696d4-bqblr   0/1     Terminating         0          6m25s
nginx-7544d696d4-bqblr   0/1     Terminating         0          6m25s
nginx-77b4fdf86c-fbwgf   1/1     Running             0          3s
nginx-7544d696d4-6s2tb   1/1     Terminating         0          6m23s
nginx-7544d696d4-6s2tb   0/1     Terminating         0          6m24s
nginx-7544d696d4-6s2tb   0/1     Terminating         0          6m24s
nginx-7544d696d4-6s2tb   0/1     Terminating         0          6m24s
nginx-7544d696d4-6s2tb   0/1     Terminating         0          6m24s
```

## Service

Pod への接続に関して負荷分散などをするオブジェクト．

- clusterIP: kubernetes 内の通信で利用することができ，クラスタ内で IP アドレスが払い出され，それを利用して Pod 間の通信を行う．
- NodePort: Node のランダムなポートを使って外部サーバとの通信を行う
- LoadBalancer: NodePort の service を作成し，外部の LoadBalancer で転送をしてもらう
- ExternalName: 外部のサービスに対してエイリアスを作成する

下記はserviceの例で，適当な Pod と 適当な Pod で通信する．

```yaml
spec:
  selector:
    app: nginx
  ports:
  - port: 80
```

```yaml
  selector:
    app: nginx
  ports:
  - port: 80
  type: NodePort
```

```bash
C:\k8s_play> kubectl get all
NAME                        READY   STATUS    RESTARTS   AGE
pod/nginx-55f598f8d-8r8cv   1/1     Running   0          2m20s
pod/nginx-55f598f8d-jp76l   1/1     Running   0          2m20s
pod/nginx-55f598f8d-q5v7r   1/1     Running   0          2m20s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        2m27s
service/nginx        ClusterIP   10.101.95.207   <none>        80/TCP         74s
service/nginx-np     NodePort    10.111.133.95   <none>        80:30626/TCP   14s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   3/3     3            3           2m20s

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-55f598f8d   3         3         3       2m20s

C:\k8s_play> kubectl run alpine -it --rm --image alpine -- ash
If you don't see a command prompt, try pressing enter.
/ # env
...
NGINX_NP_SERVICE_HOST=10.111.133.95  <- 一緒
...
NGINX_SERVICE_HOST=10.101.95.207 <- 一緒
...

/ # wget -O - $NGINX_SERVICE_HOST
Connecting to 10.101.95.207 (10.101.95.207:80)
writing to stdout
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the ng

```

### ヘルスチェック

- LivenessProbe: コンテナの生存チェックをする．これが通らない場合は再生成される．
- ReadinessProbe: コンテナの Ready 状態をチェックする．init や 処理中でリクエストが返せない場合に対応する．

- exec: コンテナ内でコマンドを実行できる．この終了ステータスが 0 なら healthy ということになる．
- httpGet: HTTP の GET リクエストを発行する．レスポンスステータスで判断する
- topSocket: TCP コネクションが開けるかチェックする．

```yaml
livenessProbe:
      httpGet:
        port: 80
        path: /
```

```bash
Liveness:       http-get http://:80/ delay=0s timeout=1s period=5s #success=1 #failure=5
```

index.html を消した場合

```bash
Warning  Unhealthy  1s (x6 over 26s)    kubelet            Liveness probe failed: HTTP probe failed with statuscode: 403 <- http ステータスコード
```
