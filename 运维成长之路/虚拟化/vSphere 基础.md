# ESXI 虚拟磁盘创建类型
## 一、虚拟磁盘创建类型
1、厚置备 延迟置零 zeroed thick （默认）
创建时为虚拟磁盘分配所需空间，使用了多少空间，就将分配的空间 zero 置零
例如：创建时200G，使用了50G，后面使用中，会将分配的空间置0

2、厚置备置零 eager zeroed thick
创建了200GB大小，会划分200GB 的空间，创建磁盘时更长，但后续使用性能最好，相当于独立硬盘

3、精简置备 thin
空间使用只增不减，容易磁盘爆满


## 二、空间回收准备
需要有足够的剩余空间

三、精简置备回收
关闭 ESXI，开启 SSH 登录， ssh esxi

    cd /vmfs/volumes/....
    执行命令
     vmkfstools --punchzero win2016_dev_14.vmdk


# ESXI 网络

![[Pasted image 20230203155702.png]]

# ESXI 存储
![[Pasted image 20230203155729.png]]


![[Pasted image 20230601151415.png]]


未共享： 虚拟机已占用，且不与其他虚拟机共享的存储
已用为什么是磁盘的两倍，因为做了快照。


文件格式

VMXF  
vmx 虚拟机配置文件
vmdk 虚拟磁盘文件，类似 ISO


```
#远程esxi
设置ssh 认证方式
启用ssh

#设置时区
vi
esxcli system time set -H 19 -m 17 -s 18

#定时关机
#/vmfs/volumnes/datastore1/autoshutdown.sh执行脚本路径
设置执行脚本

vim-cmd vmsvc/getallvms  查看所有vm
vim-cmd vmsvc/power.off 10 关闭vm，强制关机
vim-cmd vmsvc/power.shutdown 正常关机，需要安装vmware tools
/sbin/poweroff  关闭vm host宿主机

#!/bin/sh

#shutdown all VMs
for i in `vim-cmd vmsvc/getallvms|awk '{if(NR>1) print $1}'`;do vim-cmd vmsvc/power.shutdown  $i;done
echo "Closed all vms"
echo "Closing  Host Machine..."
#Poweroff Host
/sbin/poweroff

#设置定时任务
#!/bin/sh
/bin/echo "2    *   *   *   *   sh /vmfs/snapshot_creat.sh" >>/var/spool/cron/crontabs/root 
/bin/echo "30   *   *   *   *   sh /vmfs/snapshot_del.sh" >>/var/spool/cron/crontabs/root
kill $(cat /var/run/crond.pid)
crond
mv /etc/localtime /etc/localtime.bak
cp /vmfs/volumes/datastore1/Shanghai /etc/localtime 
/bin/echo "00   1   3   7   *   sh /vmfs/volumes/datastore1/shutdown.sh" >>/var/spool/cron/crontabs/root
exit 0

```
