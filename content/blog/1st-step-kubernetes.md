+++
title = "GKEでコンテナ作成／管理しよう！"
date = "2016-08-18"
tags = ["gke","kubernetes"]
banner = ""
author = "dokuma"
+++

GKEとは…

>Google Container Engine は、Docker コンテナの実行を支える強力なクラスタ マネージャおよびオーケストレーション システムとして機能します。ユーザーが定義する要件（CPU やメモリなど）に基づいてコンテナをクラスタにスケジューリングし、自動的に管理します。オープンソースの Kubernetes システム上に構築されており、ユーザーはオンプレミス、ハイブリッド、パブリック クラウド インフラストラクチャを柔軟に利用できます。

公式[^1]より引用

- Dockerはコンテナ単位の管理しかできない
- Kubernetes(以後k8sと略します)はコンテナをクラスタとして管理する仕組み
- Dockerとk8sをマネージメントしやすくサービス化したのがGKE

と私は解釈しています。

また、GKE自体の価格は、5ノードまで無料[^2]！ それ以上は ``$0.15HR/クラスタ`` だそうです。 お財布にも優しい！

## 必要なコンポーネントをインストール

gcloudコマンドは[インストール](https://cloud.google.com/sdk/docs/quickstart-mac-os-x)してる前提で進めるよ！

###### 最新のGoogle Cloud SDKにアップグレード

[ここ](https://cloud.google.com/container-engine/docs/before-you-begin)に詳しく書いてあるよ！

```
$ gcloud components update
```

###### 最新のkubectlをインストール
```
$ gcloud components update kubectl
```

###### 最新のDocker[^3]をインストール

[DockerのInstallationページ](https://docs.docker.com/engine/installation/)見て好きな環境に入れてね！

以降、GCPのProject IDをちょいちょい参照するので、作業するターミナルで変数をexportしておきます。

```
$ export PROJECT_ID=<your project id>
```

### イメージ作成

今回は機械学習用に使えそうなtensorflowのイメージを作ってみます。

[Docker Hub](https://hub.docker.com/r/tensorflow/tensorflow/)から公式リポジトリをpull

GPUは使わない、なるべく最新がいいので0.10.0rc0のタグを引っ張ってきました。

```
$ docker pull tensorflow/tensorflow:0.10.0rc0
0.10.0rc0: Pulling from tensorflow/tensorflow
56eb14001ceb: Pull complete
7ff49c327d83: Pull complete
6e532f87f96d: Pull complete
3ce63537e70c: Pull complete
1f4f6fd3e0d5: Pull complete
fe0ae0f3ab22: Pull complete
10a72d39e542: Pull complete
44cdafd451fb: Pull complete
f5b4418bcf1c: Pull complete
8f2a712621db: Pull complete
a3a976eeb78f: Pull complete
33446310bc4a: Pull complete
Digest: sha256:cac233641c1e3cf73a2049075269262f38448abe480f3ea7241b480bf083cf51
Status: Downloaded newer image for tensorflow/tensorflow:0.10.0rc0
```

イメージ確認

```
$ docker images
REPOSITORY                                  TAG                 IMAGE ID            CREATED             SIZE
tensorflow/tensorflow                       0.10.0rc0           6476695ec35f        2 weeks ago         794.1 MB
```

動作確認

```
$ docker run -d -p 8888:8888 tensorflow/tensorflow:0.10.0rc0
44150f5f513bdf11146ca33e19a8c26dfaf963e3b48014349487e5de72cd3f0e

$ docker ps -a
CONTAINER ID        IMAGE                             COMMAND                  CREATED             STATUS                        PORTS                              NAMES
44150f5f513b        tensorflow/tensorflow:0.10.0rc0   "/run_jupyter.sh"        11 seconds ago      Up 10 seconds                 6006/tcp, 0.0.0.0:8888->8888/tcp   evil_mccarthy
```

GKEプライベートリポジトリへPushするために、起動中コンテナからイメージを作成する。

```
$ docker commit 44150f5f513b gcr.io/${PROJECT_ID}/tensorflow:0.10.0rc0
sha256:9ed6746f08fa6dccc121371cfc9a864261e8fbbcad8ac2c9e02c58c89b404125

# イメージ確認
$ docker images
REPOSITORY                                  TAG                 IMAGE ID            CREATED             SIZE
gcr.io/<your project id>/tensorflow         0.10.0rc0           9ed6746f08fa        8 seconds ago       794.1 MB

# push
$ gcloud docker -- push gcr.io/${PROJECT_ID}/tensorflow:0.10.0rc0
```

この辺で図を…(；´Д｀)[^4]

![GKE architecture](/img/blog/architecture.png)

ざっくりこんな概念でございます！

### Cluster
コンテナを動作させるためのクラスタです。 後述するDeploymentsを作成するために必要なので作成しておきます。

```
# Cluster作成
$ gcloud container clusters create tensorflow-cluster001 \
  --zone asia-east1-a \
  --num-nodes 2 \
  --machine-type g1-small
Creating cluster tensorflow-cluster001...done.
Created [https://container.googleapis.com/v1/projects/<your project id>/zones/asia-east1-a/clusters/tensorflow-cluster001].
kubeconfig entry generated for tensorflow-cluster001.
NAME                   ZONE          MASTER_VERSION  MASTER_IP        MACHINE_TYPE  NODE_VERSION  NUM_NODES  STATUS
tensorflow-cluster001  asia-east1-a  1.3.4           104.155.230.147  g1-small      1.3.4         2          RUNNING
```

### Pod
Podはコンテナのグループです。GKEの場合、GCEインスタンスがNodeで、その上にコンテナが乗ります。さらにそのコンテナをまとめているのがPodです。

### Deployments
DeploymentsはPodの管理をします。

```
# Deployments作成
$ kubectl run rc-node --image=gcr.io/${PROJECT_ID}/tensorflow:0.10.0rc0 --replicas=2 --port=8888
deployment "rc-node" created

# Podの状態を確認
$ kubectl get pods
NAME                       READY     STATUS    RESTARTS   AGE
rc-node-2371076894-17kqc   1/1       Running   0          48s
rc-node-2371076894-cd4ut   1/1       Running   0          48s

# deploymentsの状態を確認
$ kubectl get deployments
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
rc-node   2         2         2            2           1m
```

### Service

Podに対するLBのようなもの。
```
# Service作成
$ kubectl expose deployment rc-node --port=8888 --type="LoadBalancer"

# Serviceの状態を確認
$ kubectl get services
NAME         CLUSTER-IP       EXTERNAL-IP       PORT(S)    AGE
kubernetes   10.255.240.1     <none>            443/TCP    2h
rc-node      10.255.243.181   104.199.197.199   8888/TCP   1m
```

注意点としては、```rc-node``` のEXTERNAL-IPは作成までに少し時間がかかるので、数十秒待ちましょう。

### 確認

``http://104.199.197.199:8888`` をブラウザで開くと…

ｷﾀ━━━(ﾟ∀ﾟ)━( ﾟ∀)━( 　ﾟ)━(　　)━(　　)━(ﾟ 　)━(∀ﾟ )━(ﾟ∀ﾟ)━━━!!

![tensorflow notebook](/img/blog/notebook-tensorflow.png)

### Scale
Podを増やします。

```
# Podの状態を確認
$ kubectl get pods
NAME                       READY     STATUS    RESTARTS   AGE
rc-node-2371076894-17kqc   1/1       Running   0          48s
rc-node-2371076894-cd4ut   1/1       Running   0          48s

# Podを一つ増やす
$ kubectl scale deployment/rc-node --replicas=3
deployment "rc-node" scaled

# Podの状態を確認
$ kubectl get pods
NAME                       READY     STATUS    RESTARTS   AGE
rc-node-2371076894-17kqc   1/1       Running   0          1h
rc-node-2371076894-8n3nc   1/1       Running   0          8s
rc-node-2371076894-cd4ut   1/1       Running   0          1h

# もっともっと！
$ kubectl scale deployment/rc-node --replicas=5
deployment "rc-node" scaled

# Podの状態を確認
$ kubectl get pods
NAME                       READY     STATUS    RESTARTS   AGE
rc-node-2371076894-0zr6q   1/1       Running   0          3s
rc-node-2371076894-17kqc   1/1       Running   0          1h
rc-node-2371076894-7j0ut   1/1       Running   0          3s
rc-node-2371076894-8n3nc   1/1       Running   0          4m
rc-node-2371076894-cd4ut   1/1       Running   0          1h
```

Nodeを増やします。

```
# Nodeを一つ増やす
$ gcloud container clusters resize tensorflow-cluster001 --size 3 --zone asia-east1-a
Pool [default-pool] for [tensorflow-cluster001] will be resized to 3.

Do you want to continue (Y/n)?  Y

Resizing tensorflow-cluster001...done.
Updated [https://container.googleapis.com/v1/projects/<your project id>/zones/asia-east1-a/clusters/tensorflow-cluster001].

# clusterの状態を確認
$ gcloud container clusters list
NAME                   ZONE          MASTER_VERSION  MASTER_IP        MACHINE_TYPE  NODE_VERSION  NUM_NODES  STATUS
tensorflow-cluster001  asia-east1-a  1.3.4           104.155.230.147  g1-small      1.3.4         3          RUNNING
```

Nodeを増やすのは少し時間がかかりました。2~3分お待ちくださいﾍ(ﾟ∀ﾟﾍ)ｱﾋｬ

俺はお掃除が嫌いなんだ！お掃除は自分で調べてやってねw

ヒント！

```
$ kubectl delete service rc-node
$ kubectl delete deployments rc-node
$ kubectl delete pods --all
$ gcloud container clusters delete tensorflow-cluster001
```

### 気になってるポイント

- Nodeに対するヘルスチェックがないけどいいのかなw
- [Node Pools](http://googlecloudplatform-japan.blogspot.jp/2016/06/google-container-enginegke.html)は今度触ってみよっと
- 今回PodとContainerが1対1の状態だけど、複数のContainerをのせるのどうやるんだろ
- terraformでも使えるねえ

### まとめ

イメージに監視とかも入れちゃえば、かなりいい感じになりそう。実際に動くアプリのイメージを使って動かしてみたいですね(｀ω´)ｸﾞﾌﾌ

覚える概念が多いので、とっつきにくいかも。``gcloud container`` がクラスタ管理、``kubectl ``がDeploymentsとPodのオペレーションといった感じで、使い分けがわかると頭の中でつながってくるかも。

Dockerに慣れていない人は、まずはローカルでDockerと触れ合うのもいいかもね。


参考サイト：
[Dockerイメージの理解とコンテナのライフサイクル](http://www.slideshare.net/zembutsu/docker-images-containers-and-lifecycle)

[GKEで半年運用してみた](http://www.slideshare.net/katsutoshinagaoka/gke-57322091)

[Google Container Engine で Webサーバ立ててScale、Fail Overさせてみる](http://qiita.com/awakia/items/95f0cde74301cebb4ff7)

[【GCP】Google Container Engineで Hello, World](http://otiai10.hatenablog.com/entry/2016/04/09/054008)

--------------------------------

[^1]: [GKE公式](https://cloud.google.com/container-engine/)
[^2]: ノードはGCEインスタンスを利用しており、ノードとして利用されている分のGCEインスタンス料金は別途かかります。
[^3]: [残念ながらDocker 1.8以降、CentOS 6系のサポートは打ち切られました](https://github.com/docker/docker/issues/14365)
[^4]: [k8s公式リポジトリより引用](https://github.com/kubernetes/kubernetes/blob/release-1.3/docs/design/architecture.md)

