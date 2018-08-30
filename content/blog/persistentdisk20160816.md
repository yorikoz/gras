+++
title = "GCP Persistent Disk更新情報(2016/08/16発表)"
date = "2016-08-24"
tags = ["googlecloudplatform","persistent disk"]
categories = ["Google Cloud Platform"]
banner = ""
author = "takeshi"
+++

2016/08/16にPersistent DiskのIOPSが25000まで拡張されたと正式に発表がありました。  
詳しくはこちらを[Google Cloud Platform Blog](https://cloudplatform.googleblog.com/2016/08/why-Google-Cloud-Platform-is-ready-for-your-enterprise-database-workloads.html)御覧ください。

確認して気付いた事を記載しておきます。

- SSD persistent diskにはCore数によるIOPSのLimitが設けられている

Instance core count | SSD PD read or write IOPS limit | SSD PD size needed to reach limit
----|----|----
15 or fewer cores | 15,000 IOPS | 500 GB
16 to 31 cores | 20,000 IOPS | 667 GB
32 cores | 25,000 IOPS | 834 GB

32CoreからSSDの最大IOPSを利用することが出来ます。

- persistent disk（標準デスク）についてはCoreによるLimitが無いことは確認出来てます。(2016/08/24時点)

- DiskSizeはいくつから25000IOPSか？

当然DiskSizeによってIOPSのLimitが変わります。  
こちらはpersistent disk（標準デスク）、SSD persistent diskともに制限がありますので以下にDiskSize別の性能を記載しております。

**SSD persistent disk**

Volume Size(GB) | Monthly Price | Sustained Random Read IOPS Limit | Sustained Random Write IOPS Limit | Sustained Read Throughput
Limit (MB/s) | Sustained Write Throughput
Limit (MB/s)
----|----|----|----|----|----
10	 | $1.7	 | 1500	 | 1500	 | 24	 | 24
50	 | $8.50	 | 1500	 | 1500	 | 24	 | 24
100	 | $17.00	 | 3000	 | 3000	 | 48	 | 48
200	 | $34.00	 | 6000	 | 6000	 | 96	 | 96
500	 | $85.00	 | 1500	 | 15000 | 	240	 | 240
667	 | $113.39	 | 20000	 | 20000 | 240	 | 	240
834	 | $141.78	 | 25000	 | 25000 | 240	 | 	240

834GB以上はSizeを変更しても性能変化はありません。

**persistent disk(標準ディスク)**

Volume Size (GB) | Monthly Price | Sustained Random Read IOPS Limit | Sustained Random Write IOPS Limit | Sustained Read Throughput Limit (MB/s) | Sustained Write Throughput Limit (MB/s)
----|----|----|----|----|----
50	 | $2	 | 37.5	 | 75	 | 6	 | 6
100	 | $4	 | 75	 | 150	 | 12	 | 12
200	 | $8	 | 150	 | 300	 | 24	 | 24
500	 | $20	 | 375	 | 750	 | 60	 | 60
1000	 | $40	 | 750	 | 1500	 | 120	 | 120
5000	 | $200	 | 3000	 | 7500	 | 180	 | 120
10000	 | $400	 | 3000	 | 15000	 | 180	 | 120

persistent diskに関しては5TBでRandomReadのLimitとなり10TBでRandomWriteの上限となります。

[Googleのドキュメント](https://cloud.google.com/compute/docs/disks/performance)を見るまでCore制限があると気づけなかったので25000IOPS使えないじゃーんって思ってる人がいましたら見てもらえればなって思います。