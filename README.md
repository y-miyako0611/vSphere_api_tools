## 概要
vCenterのAPI使ってVM作成するやつ

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
|common_config.yml|OSにログインしてパッケージインストールとかする|
|create_guest_vm.yml|VMを作成する|
|show_guest_vm.yml|作成したVMの構成情報を取得する|
|tagging_guest_vm.yml|VMにタグ(vCenter管理用途)を付与する|
|vm_icmp_check.yml|pingでIPが存在しないか確認する|
|vm_reg_check.yml|同じ名前のVM(vCenter上の管理名)が存在しないか確認する|

## 追加のディスクアタッチについて

- 初回作成時はstate: poweredonでも複数ディスクアタッチ可能
- VMが電源OFFで存在するものに対してはstate: presentを指定することでディスクをアタッチできる。poweredonではディスクアタッチできない。

## IPアドレスの設定について
- API使ったIPアドレスの設定も上記同様。powerdonのタイミングでIPの設定はできない。
