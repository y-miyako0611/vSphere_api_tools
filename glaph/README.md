## メモ
- exsi上のリソース状態をモニタリングしてみたいので環境構築のメモ書きです。
- こちらをやってみた感じ(https://github.com/jorgedlcruz/vmware-grafana)

## 必要なもの(動作環境など)
- esxi 6.7(CPU/メモリは各自の環境で大丈夫)
- vCenter 6.7(一番弱いスペックのやつ)
- CentOS8 Stream(4core/8GB 適当なスペックで作成)
- grafana(dnfで最新版)
- telegraf
- influxdb
- ~Prometheus(dnfで最新版)~
- ~vmware_exporter(pipで最新版)※vmware_exporter(CentOS8からvCenterのAPI叩くやつらしい)~

## Grafanaのインストール
- リソース状態の確認はGrafanaを使います。インストールおよび3000番ポートでアクセスするのでfirewallの許可しておく。
```
# dnf -y install grafana
# firewall-cmd --add-port=3000/tcp --permanent
# firewall-cmd --reload
# firewall-cmd --list-all
# systemctl enable grafana-server.service
# systemctl start grafana-server.service
```

## influxDBのインストール
- telegrafのデータはinfluxDBに貯めます。インストールおよび8060番ポートでアクセスするのでfirewallの許可しておく。
```
# cat <<EOF | sudo tee /etc/yum.repos.d/influxdb.repo
[influxdb]
name = InfluxDB Repository - RHEL 
baseurl = https://repos.influxdata.com/rhel/8/x86_64/stable/
enabled = 1
gpgcheck = 1
gpgkey = https://repos.influxdata.com/influxdb.key
EOF

# dnf -y install influxdb
# firewall-cmd --add-port=8060/tcp --permanent
# firewall-cmd --reload
# firewall-cmd --list-all
# systemctl enable influxdb
# systemctl start influxdb

! telegraf用のDB作成
# influx
Connected to http://localhost:8086 version 1.8.3
InfluxDB shell version: 1.8.3
> 
> CREATE DATABASE telegraf
> exit

```

## telegrafのインストール
- vCenterのデータ取集はtelegrafを使います。
```
# dnf -y install telegraf
# cd /etc/telegraf
!! telegraf.confを編集する。以下のセクションを編集などしていく
! # Configuration for sending metrics to InfluxDB
! [[outputs.influxdb]]

! 追加するやつ
urls = ["http://192.168.10.101:8086"]

! コメントアウト外す
database = "telegraf" # 上記で作成したDB名
retention_policy = "" # 必要かわからん
write_consistency = "any"　# 必要かわからん
timeout = "5s"　# 必要かわからん
```
- vCenterからリソース情報を取得するために設定追加
- 192.168.10.201はvCenterのアドレス
```
# /etc/telegraf/telegraf.d
# vi vsphere-stats.conf

## Realtime instance
[[inputs.vsphere]]
## List of vCenter URLs to be monitored. These three lines must be uncommented
## and edited for the plugin to work.
interval = "60s"
  vcenters = [ "https://192.168.10.201/sdk" ]
  username = "administrator@vsphere.local"
  password = "nya---n"

vm_metric_include = []
host_metric_include = []
cluster_metric_include = []
datastore_metric_exclude = ["*"]

max_query_metrics = 256
timeout = "60s"
insecure_skip_verify = true

## Historical instance
[[inputs.vsphere]]
interval = "300s"
  vcenters = [ "https://192.168.10.201/sdk" ]
  username = "administrator@vsphere.local"
  password = "nya---n"

  datastore_metric_include = [ "disk.capacity.latest", "disk.used.latest", "disk.provisioned.latest" ]
  insecure_skip_verify = true
  force_discover_on_init = true
  host_metric_exclude = ["*"] # Exclude realtime metrics
  vm_metric_exclude = ["*"] # Exclude realtime metrics

  max_query_metrics = 256
  collect_concurrency = 3
  
```

- telegrafの起動
```
# systemctl enable telegraf.service
# systemctl start telegraf.service

```


## 前提条件
esxi 6.7/vCenter 6.7/CentOS8 Stream(minimal)は準備していること前提となります。  
! 完了すると! http://Centos8のIP:9090 にアクセスできる。



## ~Prometheusのインストール~
~参考サイトを参照/firewalldで各ポート解放忘れないこと~

## ~vmware_exporter~
- ~vmware_exporterのインストールと起動~
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

## Grafanaに監視追加する
- 1.http://Centos8のIP:3000にアクセスする。(admin/admin)
- 2.`ホーム画面`-`Configuration(左歯車)`-`Data Sources`-`Add data source`-`Prometheus` で`Select`をクリック
- 3.Settings画面で`http://localhost:9090`を入力して`Save & test`
- 4.同じ画面の`Dashboards`タブをクリックして表示される項目を全て`import`する。
- 5.`Dashboards`の`Prometheus Status`をクリックしてグラフが表示されることを確認する。


## 
## 参考サイト
- https://www.server-world.info/query?os=CentOS_8&p=prometheus&f=1
- https://www.server-world.info/query?os=CentOS_8&p=prometheus&f=5
- https://qiita.com/hiromiarts/items/fbc2001479ce75246917
- https://knowledge.sakura.ad.jp/11633/
