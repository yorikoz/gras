+++
title = "Google Cloud PlatformってAWSと比較してどれくらい安いの？"
date = "2016-08-12"
tags = ["googlecloudplatform"]
categories = ["Google Cloud Platform"]
banner = ""
author = "daisaku"
+++

皆さま初めまして！grasys身長担当の鈴木です。

この記事にたどり着いて頂きありがとうございます。

今回ブログを書こうと思いたった理由としては、Google Cloud Platform がアツい！  
でもまだそんなに使われてない！もったいない！と思ったからです。

まだまだ知名度的にも全然AWSに勝てていないんですが、Google社はこれから  
広告の売り上げを超えると豪語しているくらいなんで、本腰入れてサービスを作っちゃってます。

そんな素晴らしいGoogle Cloud Platformを今日は紹介したいと思います。

特にコスト面を…(これ大事)

そして最初に謝っておきたいのが英語の翻訳は得意ではないので細かいニュアンスが間違っていたら  
ごめんなさい

### Google Cloud Platform(GCP)とAmazon Web Service(AWS)とのコスト比較

Enterprise Strategy Group Lab(ESG Lab)という会社が  
Google社の依頼を受けて著したこちらの[White Paper](https://cloud.google.com/files/esg-whitepaper.pdf)内で比較しています。

全て英文なので読むのが面倒くさい方用に要約してこちらに載せようと思います。

まず、ESG LabはGCPとAWSのコストを同じ条件で比較するために

- 2.5-2.6GHzのIntelプロセッサのVM
- 追加のネットワーク、ミドルウェア、DBは考慮に入れない
- GCPもAWSもどちらも公式のWEBカルキュレーターを使用

また、二つのサービスには色々な割引があることもわかってはいるが、  
AWSは長期間利用分の前払いが必要だとしてAWSを選ぶ際にはネガティブに働くとしています。

比較した結果、AWSのどの割引モデルを利用してもGCPが有利だという結果が出たようです。

以下、日本語に訳した図をアップしましたのでご参照ください。

(翻訳が心配な場合は[英語版](https://cloud.google.com/files/esg-whitepaper.pdf)をどうぞ) 
![GCPvs.AWScomparing](/img/blog/2016-08-12.png)

この図を見るとシングルインスタンスで比較した場合はおよそ半額以上の差がついていますね。

ここで注意して見ておきたいのは左から２番目の「スモールスタートアップ」の部分で、
実は**一番差がつきにくいインフラの大きさと言える**でしょう。

ビジネスをスタートする時は皆さん小さく始めて大きくしていくというスタンスになると思いますが、  
ここでAWSとGCPの価格を比較するとそこまで大きな価格優位性はでないので、結局まずはAWSから始めてみようということになるのでしょう。

しかし、ビジネスをするならいつか自身の作ったサービスを大きくしていきたいと考えているはずですよね。

そこで一番右を見てみましょう。

ここで言っているのは巨大なサービスとなった時の価格優位性です。  
なんとここでもシングルインスタンスと同様に半額以下になるのです。

大きなサービスになればインスタンス数も増え、毎月の支払額は相当な金額になっているはず。

そこでこの価格差は大きいと思いませんか？

### 「自身のサービスを大きくしたい！」とお考えの皆さんは最初からGCPにしないともったいない
…かもしれませんよ？

それではまた！

出典:[ESG Lab White Paper「Price Comparison:Google Cloud Platform vs. Amazon Web Services」](https://cloud.google.com/files/esg-whitepaper.pdf)
