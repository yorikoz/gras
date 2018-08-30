+++
title = "vulsでRPM以外も脆弱性検知(grasys)"
date = "2016-08-17"
tags = ["vuls","security"]
categories = ["ネットワークセキュリティ","エンジニアブログ"]
banner = "/img/blog/vuls_logo.png"
author = "yusukeh"
+++

こんにちワン！grasys長谷川です。

今日は脆弱性スキャナのvulsのgrasysでの利用方法ネタを書きます。

![vuls](/img/blog/vuls_logo.png)

--------------------------------------------

[vuls](https://github.com/future-architect/vuls)

簡単な構成図は[Official](https://github.com/future-architect/vuls#architecture)にあります。

grasysではvulsを工夫して使っているので今回はその部分をご紹介します。

### grasysでの構成例
grasysではopsというオーケストレーションしたりするオペレーション用のインスタンスを生成しており、そこから``vuls scan``を発行しています。

![vuls_architecture.png](/img/blog/vuls_architecture.png)

### grasysだとそのままで使えない理由
grasysではサーバの構成をそれなりにカスタマイズしているのでvulsそのままでは使いにくい点があったりします。

検知対象が**RPM Package**のみだったりするので
SourceからBuildするような形だと対象外になってしまいます。

**RPMだけで構成してる方はここでブラウザをおもむろに閉じてくださいｗ**

------------------------------------------

## 簡単にgrasysの基本的なアーキテクチャを説明

grasysでは言語系のコンポーネントにRPMを極力利用しないようにしています。

目的としては単純に以下です。

- 長期間の運用考慮
- 言語のUpdateを止めたままその他のLibなど更新しやすくしたい（お客様の事情で言語のVersion上げられないこともあるので

OSのVersionが上がっても動く環境を再現しやすくするためにやっています。（ちょっと変わってるのかもしれないけどｗ

## 言語管理系

Lang	| Environment Tool
--------|-----------------
php		| [phpenv](https://github.com/CHH/phpenv)
python	| [pyenv](https://github.com/yyuu/pyenv)
ruby	| [rbenv](https://github.com/sstephenson/rbenv)
perl	| [plenv](https://github.com/tokuhirom/plenv)
java	| [jenv](https://github.com/gcuisinier/jenv)
erlang	| [kerl](https://github.com/kerl/kerl)
node.js	| [nodebrew](https://github.com/hokaccha/nodebrew) -> [nodenv](https://github.com/nodenv/nodenv)に移行中

かなりXXenvを多様していますｗ  
大好きXXenv！  
[anyenv](https://github.com/kennethreitz/autoenv)は自動管理するに少しやりにくいところがあったのでうちでは使ってません
開発環境とかだと自分で常にいじったりしてるだろうからいいんだろうけどね〜
上記以外で言語系ではないですが[autoenv](https://github.com/kennethreitz/autoenv)なども便利なので使ってます！

### 上記以外のgrasysの必須コンポーネント
- [consul](https://www.consul.io/)
- [serf](https://www.serf.io/)
- [git](https://git-scm.com/)
- etc…

----------------------------------------

# RPM以外をどうやって検知対象に入れているか
## /usr/local配下

以下あるインスタンスの``/usr/local``の状態です。

```
$ tree -L 1 /usr/local/
/usr/local/
├── bin
├── etc
├── games
├── hiredis -> /usr/local/hiredis-0.10.1
├── hiredis-0.10.1
├── include
├── lib
├── lib64
├── libexec
├── nginx -> nginx-1.10.1
├── nginx-1.10.1
├── plenv
├── pyenv
├── sbin
├── share
├── src
└── var
```
以下ができるような簡単なscriptを用意しています。

1. ``/usr/local``配下でsymlinkになっているものをreadlink
1. 実態のbasenameを取得
1. [Middleware Name], [Version], [Patch Version]にsplit
1. JSONにする
1. consul kvにsetする

以下のような情報が入ります。

```
$ consul-cli kv read vuls/[instance name] | jq .
{
  "host": {
    "node_name": "[instance name]",
    "tags": {
      "machine-type": "n1-standard-2",
      "numeric-project-id": xxxxxxxxxxxxxx,
      "zone": "asia-east1-a",
      "ip": "10.240.x.x",
      "project-id": "xxxxxx-xxxxxx-xxxxxx",
      "external-ip": "xxx.xxx.xxx.xxx",
      "servertype": "[tag]",
      "tags": [
        "[tag]"
      ]
    }
  },
  "middleware": [
    {
      "name": "python",
      "version": "2.7.11"
    },
    {
      "version": "5.20.1",
      "name": "perl"
    },
    {
      "version": "2.69",
      "name": "autoconf"
    },
    {
      "version": "1.15",
      "name": "automake"
    },
    {
      "version": "2.4",
      "name": "chrony"
    },
    {
      "name": "daemon",
      "version": "0.6.4"
    },
    {
      "name": "git",
      "version": "2.8.1"
    },
    {
      "name": "hiredis",
      "version": "0.10.1"
    },
    {
      "version": "3.0.7",
      "name": "redis"
    },
    {
      "version": "6.0",
      "name": "texinfo"
    },
    {
      "name": "consul",
      "version": "0.6.4"
    },
    {
      "version": "0.7.0",
      "name": "serf"
    },
    {
      "version": "0.7.3",
      "name": "dkron"
    },
    {
      "name": "inotifywait",
      "version": "3.13"
    },
    {
      "name": "sshguard",
      "version": "1.6.4"
    },
    {
      "version": "2.4.5",
      "name": "snoopy"
    }
  ],
  "update_time": "2016-08-12 18:58:53"
}
```

ここではpatch versionが含まれているMiddlewareがありませんが  
存在すれば抽出するようにごにょごにょしていますｗ

こんなのを適当にcronで毎日回しています。

## vulsのtomlを自動生成
grasysでは簡単なtemplate engineを利用してvulsのconfigを自動生成しています。

consul kvに入っているJSONはvulsのconfigのcpeNamesを自動設定するために利用しています。

![vuls config生成](/img/blog/vuls_config.png)

vuls configのserversセクション抜粋)

```
[servers]

[servers.[instance name]]
host = "[instance name]"

cpeNames = [
  "cpe:/a:xxxxxxx:[product]:[version]:[update]"
]
```

上記のcpeNamesの自動生成を取得するためにcve.sqlite3に発行するSQLのSampleです)

```
sqlite3 [cve.sqlite3]

SELECT
  cpe_name
FROM cpes 
WHERE product = "[product]"
AND version = "[version]"
[ AND update = "[update]" ]
```

上記を``vuls scan``するまえに自動生成し定時実行しています。

これで``phpenv``や``pyenv``といった言語コンポーネントの脆弱性も検知できるようにしています。
通知系はvulsのslack機能は使わずにgrasysの監視ツールに渡してslack通知しています。（slackの通知多くなっちゃうから・・・

# InitScript

いろいろサブコマンドとか覚えれられないので独自のInitScriptを用意して運用の軽減をしています。

```
/etc/init.d/vuls

/etc/init.d/vuls: vuls init script
help:

example:
  /etc/init.d/vuls [sub command]

sub command:
  start:                start server
  stop:                 stop server
  restart               restart server
  status                server status

  1st_setup             setup, update_week, reconfig, prepare, start
  full_setup            setup, update_full, reconfig, prepare, start

  reconfig:             make config
  setup:                setup cve/nve database
  prepare:              prepare instance

  scan:                 scan
  history:              scan history
  report:               for consul service report
  tui:                  Terminal User Interface

update cve database
  update_entire:        dictionary entire update
  update_month:         dictionary month update
  update_week:          dictionary week update
  update_full:          dictionary full update
```

# ちょっと残念なとこ・・・

各言語のversionによる脆弱性やMiddlewareの脆弱性は検知できるけども！

**Web Application Frameworkのversionまで自動で集めることができずｗｗｗ**

上記の対応については
Instance Tagに紐づくyamlファイルを拡張で持つことができるようにしてあり
そこに苦肉の策としてごにょごにょ書くことで対応するようにしてます・・・

example)

```
name: django
version: 1.6
```

``pip freeze``とかしちゃうと他の言語も全部するはめに・・・  
だからやらない！