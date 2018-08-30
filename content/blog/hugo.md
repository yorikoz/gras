+++
title = "hugo使い始めました"
date = "2016-08-10"
tags = ["hugo"]
categories = ["お知らせ"]
banner = ""
author = "yusukeh"
+++

こんにちは株式会社grasysの代表しております長谷川です。

静的サイトジェネレーター [Hugo](https://gohugo.io/) を使い始めました。  
こういうのやり始めると止まらなくなるから良くない・・・うちはインフラの会社  
個人的には[Medium](https://medium.com/@yusuke_h)押しだったりするけど・・・（だがしかし、もう書いてないｗ）

> なぜ犬なのかはお答えしません！ｗ というより理由がありません！ｗ

[lektor](https://www.getlektor.com/), [ghost](https://ghost.org/), [Jekyll](https://jekyllrb.com/) といくつか候補を考えたんだけど**Hugo** にしました。（個人的にはghostでも良かったかもｗ）

themeは [casper](http://themes.gohugo.io/casper/) これを使わせて頂いてます。(ghostのdefault themeがcasperそっくりｗ)

### Google Cloud Storage

弊社は**google partner**なので**Google Cloud Storage**のWebsite Hostingを利用することにしました。

というか静的なWebサイトは全部これにしてます。

Documentは以下

[Static Website Examples, Troubleshooting and Tips](https://cloud.google.com/storage/docs/static-website)

subdomainはblog.grasys.io

### bucket生成

ex)
```
gsutil mb -c <Storage Classes> -l <Bucket Locations> <Bucket Name>
```

[Storage Classes Bucket Locations](https://cloud.google.com/storage/docs/storage-classes)

standardでasiaにmulti-regionalで作ります。

```
gsutil mb -c STANDARD -l ASIA gs://blog.grasys.io
```

※注意としてはGoogle Web Master ToolでDNSの所有者であることを設定しておく必要があります。->[Domain-Named Bucket Verification](https://cloud.google.com/storage/docs/domain-name-verification)

### ACL設定
```
gsutil defacl set public-read gs://blog.grasys.io
```

### DNS設定
Google Cloud DNSを利用してます。

digはこんな感じ

```
dig blog.grasys.io

blog.grasys.io.		3600	IN	CNAME	c.storage.googleapis.com.
```

### Hugo
hugoの設定して(かなり大変だったｗ)いろいろ準備してコンテンツのMarkdown書いて・・・

hugoのconfig.tomlがあるDirectoryまで移動して

```
$ hugo
Started building site
0 draft content
0 future content
2 pages created
0 non-page files copied
5 paginator pages created
2 tags created
0 categories created
in 78 ms

$ tree -L 1 hugo
hugo
├── archetypes
├── config.toml
├── content
├── data
├── layouts
├── public
├── static
└── themes

7 directories, 1 file
```

### Google Cloud StorageへUploadする！
js, css, htmlを圧縮しながらUpload

```
gsutil -m cp -R -z js,css,html ./hugo/public/* gs://blog.grasys.io/
```

---------------------------------------
Google Cloud PlatformのDocumentは時間経つとけっこう変わるので注意して！
