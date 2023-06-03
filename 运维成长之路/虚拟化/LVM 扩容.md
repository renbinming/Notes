
虚拟机关机，扩展磁盘大小 ，从100g 扩到 500G，也可以添加磁盘

lsblk 查看整块盘大小变大了
df -h 

对新增磁盘进行分区
```
fdisk /dev/sda
Command (m for help): n
Partition number (4-128, default 4): 
First sector (209713152-838860766, default 209713152): 
Last sector, +sectors or +size{K,M,G,T,P} (209713152-838860766, default 838860766): 

Created a new partition 4 of type 'Linux filesystem' and of size 300 GiB.

Command (m for help): t
Partition number (1-4, default 4): 
Partition type (type L to list all types): 31
Command (m for help): w
The partition table has been altered.
Syncing disks.
```

lsblk 可以看到，新分区已经识别到
![[Pasted image 20230603164301.png]]

格式化分区
mkfs -t xfs /dev/sda4

创建物理卷 pv
pvcreate /dev/sda4 
按y 
WARNING: xfs signature detected on /dev/sda4 at offset 0. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/sda4.
  Physical volume "/dev/sda4" successfully created.


查看 pv
pvdisplay
pvs
pvscan

查看 vg 
vgs

扩展vg，将 /dev/sda4 加入到 almalinux vg 里面

vgextend almalinux /dev/sda4

vgdisplay 

查看逻辑卷  lv
lvdisplay

扩展逻辑卷
lvextend /dev/almalinux/data0 /dev/sda4
lvdisplay

写入文件系统
xfs_growfs /dev/almalinux/data0
