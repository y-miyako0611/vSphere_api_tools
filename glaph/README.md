## メモ
esxiのリソース監視のメモ書き

## 必要なもの(動作環境など)
- esxi 6.7(CPU/メモリは各自の環境で大丈夫)
- vCenter 6.7(一番弱いスペックのやつ)
- CentOS8 Stream(4core/8GB 適当なスペックで作成)
- Prometheus(dnfで最新版)
- grafana(dnfで最新版)
- vmware_exporter(pipで最新版)※vmware_exporter(CentOS8からvCenterのAPI叩くやつらしい)

## 前提条件
esxi 6.7/vCenter 6.7/CentOS8 Stream(minimal)は準備していること前提となります。

## 

## 参考サイト
- https://www.server-world.info/query?os=CentOS_8&p=prometheus&f=1
- https://qiita.com/hiromiarts/items/fbc2001479ce75246917
- https://knowledge.sakura.ad.jp/11633/
