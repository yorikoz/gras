+++
title = "古いcompute-image-packagesからの新しいgoogle-compute-engine Packageへの移行方法"
date = "2016-08-10"
tags = ["googlecloudplatform","compute-image-packages","google-compute-engine"]
categories = ["Google Cloud Platform"]
banner = ""
author = "yusukeh"
+++

こんにちは！grasys長谷川です！

Compute EngineにはGoogleが提供してくれているいくつかの管理Daemonがあります。  
ある時期からそれがUpdateされ大幅な変更が入りました。

これ困ってる人いないのかな・・・

### compute-image-packages
[github.com/GoogleCloudPlatform/compute-image-packages](https://github.com/GoogleCloudPlatform/compute-image-packages)

### Daemons
[compute-image-packages - Daemons](https://github.com/GoogleCloudPlatform/compute-image-packages#daemons)

### Old
- manage_accounts
- manage_addresses
- manage_clock_sync

ps)

```
/usr/bin/python /usr/share/google/google_daemon/manage_accounts.py
/usr/bin/python /usr/share/google/google_daemon/manage_addresses.py
/usr/bin/python /usr/share/google/google_daemon/manage_clock_sync.py
```

### New
- google_accounts_daemon
- google_ip_forwarding_daemon
- google_clock_skew_daemon

ps)

```
/usr/bin/python /usr/bin/google_accounts_daemon
/usr/bin/python /usr/bin/google_clock_skew_daemon
/usr/bin/python /usr/bin/google_ip_forwarding_daemon
```

### Upgradeする方法

**公式な方法じゃありませんのであしからず！責任取れません！ｗ**

正常にupgradeできてるかはわからないけど一応今のところ正常に稼働しているので大丈夫かな・・・

今回新しくyum repoが提供されるようになりました。

詳細は以下
[compute-image-packages - Package Distribution](https://github.com/GoogleCloudPlatform/compute-image-packages#package-distribution)

対応方法が書いてあります。  
今回は古い方がニーズがありそうなのでCentOS6ベースで書きます。

### yum repository file

```
DIST=6
tee /etc/yum.repos.d/google-cloud.repo << EOM
[google-cloud-compute]
name=Google Cloud Compute
baseurl=https://packages.cloud.google.com/yum/repos/google-cloud-compute-el${DIST}-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOM
```

> Google Documentのコピペっす・・・rootでやるなりsudoするなりしてください・・・すいませんｗ

``/etc/yum.repos.d/google-cloud.repo``これがyumのrepositoryファイルです。

### yum cache clean
yumのcacheあると邪魔かもしれないので適当にcache cleanしときます。

```$ sudo yum clean all```

### 古いrpmが入ってるか確認
```$ sudo rpm -qa | grep google```

以下のPackageが入ってたら古い！
- google-compute-daemon
- google-startup-scripts

古いrpmの削除

```
$ sudo rpm -e google-compute-daemon
$ sudo rpm -e google-startup-scripts
```

### 新しいrpmのInstall

徐ろに以下を叩く！

```
$ sudo yum install -y google-compute-engine google-compute-engine-init google-config
```

### 起動

```
$ sudo initctl start google-instance-setup
```

ps確認)

```
$ ps aux | grep google_

/usr/bin/python /usr/bin/google_accounts_daemon
/usr/bin/python /usr/bin/google_clock_skew_daemon
/usr/bin/python /usr/bin/google_ip_forwarding_daemon
```

initctl確認)

```
$ initctl status google-accounts-daemon
google-accounts-daemon start/running, process 2232
```

```
$ initctl status google-clock-skew-daemon
google-clock-skew-daemon start/running, process 2234
```

```
$ initctl status google-ip-forwarding-daemon
google-ip-forwarding-daemon start/running, process 2236
```

# ちなみに
- 2016/08/10 現在 Googleから公式なUpgrade手法は出ていません。
- manage_accountsにはauthorized_keysの生成時、極僅かな時間に発生する問題があります。
- サポート投げるとインスタンスを新しいImageから作り直すようにアナウンスされます・・・ぉぃ

なのではやめに新しいものにしましょう！

## updateすると起きる可能性があるもの

pythonをsystem以外のものをinstallして使っている条件においてgcloudを上げると、もしかしたらgcloudが動かなくなる可能性があります。

その場合は以下で動作するようになると思います。

### 対処方1

```
sudo pip install google-compute-engine

or

sudo pip install --upgrade google-compute-engine
```

pipが入ってなかったら別途Installして下さい。

### 対処法2

gcloudはShell Scriptになっているので以下で確認

```
which gcloud
/usr/local/share/google/google-cloud-sdk/bin/gcloud
```

環境変数が存在しそれをセットすることで回避できる可能性もあります。

```
# <cloud-sdk-sh-preamble>
#
#  CLOUDSDK_ROOT_DIR            (a)  installation root dir
#  CLOUDSDK_PYTHON              (u)  python interpreter path
#  CLOUDSDK_PYTHON_ARGS         (u)  python interpreter arguments
#  CLOUDSDK_PYTHON_SITEPACKAGES (u)  use python site packages
#
# (a) always defined by the preamble
# (u) user definition overrides preamble
```

上記以外のパターンもありそう・・・  
もうかれこれ2年以上Compute Engine触ってきてるけど  
さすがに記憶がない・・・

# ぼやき
CentOS6のSCL el6で提供されてるpython27-xxx系のPackageに  
含まれるbinaryがなぜにlinkerがsharedなのか理解ができない・・・