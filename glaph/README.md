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
! 完了すると! http://Centos8のIP:9090 にアクセスできる。

## Prometheusのインストール
参考サイトを参照/firewalldで各ポート解放忘れないこと

## Grafanaのインストール
参考サイトを参照/firewalldで各ポート解放忘れないこと

## vmware_exporter
- vmware_exporterのインストールと起動
```
# dnf -y install python36
# pip3 install vmware_exporter
# cd /root/;mkdir vcenter;cd vcenter;vi vcenter
-- ここから
default:
    vsphere_host: "192.168.10.201"
    vsphere_user: "administrator@vsphere.local"
    vsphere_password: "nya---n"
    ignore_ssl: True
    specs_size: 5000
    fetch_custom_attributes: True
    fetch_tags: True
    fetch_alarms: True
    collect_only:
        vms: True
        vmguests: True
        datastores: True
        hosts: True
        snapshots: True
-- ここまで
# vmware_exporter -c config.yml &
# firewall-cmd --add-port=9272/tcp --permanent
# firewall-cmd --reload
! http://Centos8のIP:9272　でメトリックスが見れるはず
```
- Prometheusの監視対象に追加する
```
#  vi /etc/prometheus/prometheus.yml
-- ここから
  # add vcenter
  - job_name: 'vcenter'
    static_configs:
    - targets: ['localhost:9272']    
-- ここまで

# systemctl restart prometheus.service
! http://Centos8のIP:9090 にアクセスしてvmware関連のグラフが取得できるようになっているのを確認する
```

## 
## 参考サイト
- https://www.server-world.info/query?os=CentOS_8&p=prometheus&f=1
- https://www.server-world.info/query?os=CentOS_8&p=prometheus&f=5
- https://qiita.com/hiromiarts/items/fbc2001479ce75246917
- https://knowledge.sakura.ad.jp/11633/
