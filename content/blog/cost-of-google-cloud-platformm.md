+++
title = "GCPとAWSの違いって何"
date = "2016-08-22"
tags = ["googlecloudplatform"]
categories = ["Google Cloud Platform"]
banner = ""
author = "takeshi"
+++

GCPとAWSの違いって何？と聞かれる事が多々あります。  
今回はGCPとAWSでsystemを構築した場合の簡単な比較を紹介します。  
弊社daisakuの記事を噛み砕いた感じで書いてます。[Google Cloud PlatformってAWSと比較してどれくらい安いの？ - grasys blog](https://blog.grasys.io/post/daisaku/cost-of-google-cloud-platform/)も参考にしてみてください。

## instance費用比較
弊社が主に扱っているIaaS環境のGoogle Compute Engine(GCE)とAmazon Elastic Compute Cloud(EC2)で比較します。  
例）簡単なSystem構成を例としています。  
``Load Balancer、ApplicationServer、BatchServer、CacheServer、DatabaseServer``
各instanceを比較しやすいように4Coreで計算します。LBについては100GBを通信したとして計算します。※2016/08/22現在の価格  
※AWSの費用算出はオンデマンドinstanceとなっています。  

**GCE** | 
----| ----
n1-standard-4(4Core Memory15GB) | $112.42
NetWork Load Balancer | $19.05
4instance+LB | **$468.73**

**EC2** | 
----| ----
m4.xlarge(4Core Memory16GB) | $254.04
Elastic Load Balancing | $20.51
4instance+LB | **$1036.67**

結果として大きな差がつきましたがGCEは継続利用期間で自動的に値引きが行われていますので気づかないでこの恩恵を受けてる方も多いかと思います。  
今回は比較として以下値引きが行われていない場合でも金額差は出ています  
**GCP $609.349**  
**AWS $1036.67**

今回は各役割のinstanceを1台ずつ計4instanceを例としましたが、将来的なスケールやDatabaseの冗長を考えると台数が増えた場合には差額が広がっていきます。

あくまでオンデマンドinstanceでの比較になりますのでリザーブドinstanceで前払いなどを行えば金額は変わります。

ちょこちょこGoogleとの違いについて書いていこうかなとおもってます。  
次は結構前に発表されてますがPersistentDisksのオンラインリサイズを書こうかな。多分GCPでしか今の所出来ないかと。