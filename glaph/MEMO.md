## メモ
influxDBになんのデータ入っているかメモだよ

## コマンドメモ
- DBにログイン
```
# influx
>
```

- DB切り替え
```
use telegraf
```

- テーブル参照
```
> SHOW MEASUREMENTS
name: measurements
name
----
! 適当に抜粋
cpu
disk
vsphere_datacenter_vmop
vsphere_datastore_disk
```

- タグ参照
```
! show tag keys from [テーブル名]
> show tag keys from vsphere_host_cpu
name: vsphere_host_cpu
tagKey
------
cpu
dcname
esxhostname
host
moid
source
vcenter
```

- フィールド参照
```
! show field keys from [テーブル名]
> show field keys from vsphere_host_cpu
name: vsphere_host_cpu
fieldKey                 fieldType
--------                 ---------
coreUtilization_average  float
costop_summation         integer
demand_average           integer
idle_summation           integer
latency_average          float
```
