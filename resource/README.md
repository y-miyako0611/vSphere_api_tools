## メモ

## コマンド
作業ディレクトリは適当に(/root/vcenter/test)
- vmのリソース情報を出力
```
#!/bin/bash

ID=`curl -k --request POST --url  https://192.168.10.201/rest/com/vmware/cis/session --header 'authorization: Basic hogehoge' --header 'content-type: application/json'   --header 'user-agent: vscode-restclient' | jq .value | sed 's/"//g'`
curl -k --request GET --url 'https://192.168.10.201/rest/vcenter/vm?filter.power_states=POWERED_ON' --header 'content-type: application/json' --header 'user-agent: vscode-restclient' --header "vmware-api-session-id: $ID" > list.txt
cat list.txt | jq -c '.value[] | {"VM_NAME": .name, "MEMORY": .memory_size_MiB, "CPU": .cpu_count}' > vms_resource.txt
rm -fr list.txt
```

-  esxiにログインして収容されているVMを確認する
```
#!/bin/bash
#
#
# usage) get_vms.sh esxi_ip_list.txt
ESXI_IP_LIST=$1
ESXI_ID=root
ESXI_PASS=abcdefg

_usage() {
        echo "usage) $0 <host_name>"
        exit 1
}

if [ $# -ne 1 ]
then
        _usage
fi
rm -fr esxi_vm_list.txt
for i in `cat $ESXI_IP_LIST`
do
    #sshpass -p $ESXI_PASS ssh -l $ESXI_ID $i "vim-cmd vmsvc/getallvms" | awk '{print "{\"VM_NAME\":\"" $2 "\","}' | grep -v Name | sed "s/$/ $i/g" >> esxi_vm_list.txt
    sshpass -p $ESXI_PASS ssh -l $ESXI_ID $i "vim-cmd vmsvc/getallvms" | awk '{print $2}' | grep -v Name | sed -e "s/^/{\"VM_NAME\":\"/g" -e "s/$/\", \"ESXI_IP\":\"$i\"}/g" >> esxi_vm_list.txt
done
```
