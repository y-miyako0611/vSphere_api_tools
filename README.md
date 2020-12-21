## 概要
vCenterのAPI使ってVM作成するやつ  
どっかの会社で使ってるやつと同じ。  

## その他
以下はテスト用  
https://github.com/y-miyako0611/vsphere-public

## テンプレート作成について(CentOS7 minimal)
Ansibleからapi叩いてテンプレート作るとNICがONにならない事象あった。とりあえず以下をやる
```
! これでapi経由でIP設定できる
yum -y install open-vm-tools perl

! 上記でIP設定できるけどNICがONにならない事象ある(以下はOKだった)　なんかのバージョン依存かな・・・
!! dhcp有効 | v4 OFF v6 ON | v4 ON(IP降ったもの) v6 OFF| 一度既存NIC削除 は動いた 

```

## awxのワークフロー
組み込みたいテンプレートにsurveyが定義されているとだめ？


## 各種プレイブックについて

| ファイル名| 内容|
| :----| ----|
|guest_os_add_disk.yml|２個目以降のディスクをアタッチする|
|guest_os_common_config.yml|VMにログインして色々設定する|
|guest_os_create.yml|指定された情報で作成する|
|guest_os_icmp_check.yml|VMに割り当てるIPが存在しないか確認する|
|guest_os_info.yml|VMのタグ情報取得する|
|guest_os_poweron.yml|VMの電源をONする|
|guest_os_register_check.yml|vmware上に同じ名前のVMが存在しないか確認する|
|guest_os_tagging.yml|VMにタグを付与する|

## 追加のディスクアタッチについて

- 初回作成時はstate: poweredonでも複数ディスクアタッチ可能
- VMが電源OFFで存在するものに対してはstate: presentを指定することでディスクをアタッチできる。poweredonではディスクアタッチできない。

## IPアドレスの設定について
- API使ったIPアドレスの設定も上記同様。powerdonのタイミングでIPの設定はできない。
