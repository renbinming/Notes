## 基础

### 系统组成

kernel:

 驱动硬件——抽象成可用资源

​	分配资源，管理进程

​	时序复用：IO、网络、CPU

​	空间复用：内存、硬盘



内存有共享内存空间，放置共享库的文件



用户界面

​		GUI：

​				Gnome

​				KDE

​		CLI:   bash , zsh,sh,csh  （应用程序接口）

​				echo $SHELL  # 查看所用shell的类型

实际上是通过shell接口调用kernel，从而使用硬件资源

而**shell是解释命令，并调用命令，应用程序，本质上是一个解释器**



远程连接

​	ss -ntl : 查看监听于tcp协议



终端设备： terminal   查看设备 tty

​	物理终端，控制台，console             /dev/console

​	键盘连的：虚拟终端，tty                           /dev/tty# [1,6]

​	串口线连的：串行终端 ttyS							/dev/ttyS#

​	远程：伪终端 pts								/dev/pts/#



环境变量 PATH，从左到右查找命令对应的文件

内置命令，外部命令。内置无对应文件



### man   帮助文档

​	whatis man 可以查看手册有几个

​	SYNOPSIS: 语法格式

​		[]: 可选内容

​		<>: 必须提供的内容

​		a|b|c ： 选一个

手册分几种章节

1. man 1 用户命令
2. man 2  系统调用手册
3. man 3 C库调用
4. man 4 设备文件及特殊文件
5. man 5 文件格式
6. man 6 游戏帮助
7. man 7 杂项
8. man 9 管理工具及守护进程



### 匹配模式

​     * 任意长度任意字符

​     ? 单个 任意字符

​	 [] 指定范围内的任意单个字符

​	[^] 匹配范围外的任意单个字符





### 加密算法：

​		对称加密，加密解密用同一个算法

​		非对称加密，加解密用密钥对

​		**单向加密**： 只能加密，不能解密，提取数据特征码，定长输出

​					加密结果都一样，可以撞库破解。

​					1 - md5:   128bits

​					sha： sha224 sha256 sha384   6- sha512

​					在计算之时，加salt，添加随机数

​					

$加密算法$盐$密码

​					

### 用户和用户组管理

groupadd

useradd

创建 用户 的时候，实际上是拷贝 /etc/skel 下的 .bash_* 文件到

/home/user 下

如果用户下的bash文件丢失，就会出现 

-bash-4.2$

## 内核

支持模块化  .ko (kernel object)

支持模块运行时动态加载或卸载

www.kernel.org

组成部分

​	核心文件 /boot/vmlinuz-VERSION.x86_64

​	模块文件  /lib/modules/VERSION-RELEASE

​	ramdisk：

​			initramfs-3.10.0-1160.21.1.el7.x86_64.img

### 启动流程

1、加电自检POST  （ROM：CMOS  BIOS）

2、加载bootloader   Boot sequence 

查找引导设备，第一个有引导程序（bootloader）的设备

​	windows:  ntloader

​	linux:   

​		LILO: LIlinux LOader  最早的，现已不用 (安卓机好像在用这个)

​		GRUB： grand uniform bootloder

功能： 提供一个菜单，让用户选择要启动的系统或不同的内核版本，内核加载到RAM中，然后系统控制权交给内核

grub  

1. mbr

   1.5 mbr之后的扇区，识别stage2的文件系统

2. 磁盘分区 /boot/grub/



bl 存储在MBR中 MBR 最大支持2T硬盘分区

UEFI  取消了BIOS自检

GPT	可管理的硬盘容量=18EB(1EB=1024PB=1,048,576TB)

全称为Globally Unique Identifier Partition Table，也叫做GUID分区表，它是UEFI 规范的一部分		



3、

kernel

​	自身初始化：探测可识别到的所有硬件

​			加载硬件驱动；（可能会借助randisk加载驱动）

​			以只读方式挂载根文件系统

​			运行用户空间的第一个应用程序：/sbin/init

​			centos5  SysV init    /etc/inittab

​			C6   upstart    /etc/inittab   /etc/init/*.conf

​			C7  systemd    /usr/lib/systemd/

启动级别

​	0  关机

​	1  单用户模式，维护模式

​	2 多用户模式，有网络功能，无NFS，维护模式

​	3 多用户模式，文本界面

​	4 预留

​	5  多用户模式，图形界面

​	6 重启

查看当前级别

 runlevel  

who -r

修复grub

挂载光盘，进入救援模式

chroot /mnt/sysimage

grub-install --root-directory=/boot /dev/sda



## 进程管理

理想状态： 70% 时间在执行代码，内核模式

内核的功能： 进程管理，文件系统，网络，内存，驱动，安全



### 进程优先级

​			0-139：

​					1-99 实时优先级

​					可通过nice值调整的 100-139: 静态优先级

​					**越接近100，优先级越高**

​					nice值  越小越优先（默认是0）

​							-20 ~ 19   0对应120





同一主机交互通信

​		shm   shared memory

​		signal

​		semerphor

不同主机通信

​		socket

​		rpc



CPU-Bound   CPU密集型，非交互式

IO-Bound  	IO密集型，交互式



管理工具：pstree  ps pidof  pgrep top glances htop pmap vmstat dstat kill pkill job bg fg nohup ...



pstree 查看系统进程

pmap 查看进程的内存表

### ps 

​	有些系统不用带- 有些必须带 -

​	BSD 一定不能带

​	UNIX 一定要带

​	GNU 要两个

​	a: 所有与终端相关的进程 (bsd风格)

​	x： 所有与终端无关的进程

​	u:  显示用户信息

​	-e: 显示所有进程 （unix风格） 相当于ax

​	-f: 显示更多字段，完整格式

​	-F: 显示更多更多字段

​	-H: 以层级结构显示进程

​	o: 自定义要显示的字段

#### 常用组合一：ps aux

```
USER        PID %CPU %MEM    VSZ   RSS  TTY      STAT START   TIME COMMAND
root          1  2.4  0.2 190936  3880 ?        Ss   20:36   0:01 /usr/lib/systemd/sys
root          2  0.0  0.0      0     0 ?        S    20:36   0:00 [kthreadd]  ###内核线程
```

VSZ 虚拟内存集

RSS 常驻内存集

page页，页框

STAT

​	R: running

​	S: interruptable sleeping

​	D: uninterruptable sleeping

​	T: stopped

​	Z: zombie

\+: 前台进程

| : 多线程进程

N: 低优先级进程

< :  高优先级进程

s: session leader   kill 掉，所有子进程都会被kill



#### 常用组合二：ps -ef

#### 常用组合三：ps -eFH

#### 常用组合四： ps axo   ps -eo

o: 自定义要显示的字段

​	常用的 pid, ni, pri, psr,pcpu,stat,comm,tty,ppid,rtprio

​	ni: nice值



pgrep

​	-U:  指定用户进程号

​	-l： 显示进程名

​	-a：显示完整格式的进程名，进程和参数列表

​	-t：与终端相关的进程

​	-P： 子进程的进程号

pgrep -U root -al



pidof ：

​	根据进程名，取pid

```
# pidof sshd
1545 1109 836
```



### top

​	P: 占据CPU百分比排序（默认）

​	M：按内存占用排序

​	T: 按占用CPU累计时间排序



​	-d： 指定刷新间隔

​	-b：显示一屏

​	交互式直接输入s：修改刷新时间间隔

​	k： 终止指定进程



load average: 0.16, 0.34, 0.50

过去1，5，15分钟的平均负载

等待运行的队列长度

**us： 用户空间 (应用程序，进程或线程)**

**sy： 内核使用 （系统调用）**

**高负载时，最好是在us占用，70%最佳**

wa: io等待

hi：硬件中断  si：软中断

st：被偷走的百分比，比虚拟化程序偷走的



缓冲和缓存的空间可以回收，真正的可用空间free+buffer



### vmstat

   Procs
       r: The number of runnable processes (running or waiting for run time). 

​		等待运行的任务的队列长度
​       b: Number of processes blocked waiting for I/O to complete.

​		不可中断睡眠的进程个数，被阻塞的任务队列长度

   Memory
       swpd: the amount of virtual memory used.
       free: the amount of idle memory.
       buff: the amount of memory used as buffers.
       cache: the amount of memory used as cache.
       inact: the amount of inactive memory.  (-a option)
       active: the amount of active memory.  (-a option)

   Swap
       si: Amount of memory swapped in from disk (/s).
       so: Amount of memory swapped to disk (/s).

​		活动如果特别频繁，则物理内存太小

   IO
       bi: Blocks received from a block device (blocks/s).

​		从块设备读入数据到系统的速度（kb/s）

​       bo: Blocks sent to a block device (blocks/s).

​		保存数据至块设备的速率（kb/s）

   System
       in: The number of interrupts per second, including the clock.

​		中断信号，打断cpu，请求CPU 

​	IO设备与CPU交互



​       cs: The number of context switches per second.

​			上下文切换的速率

   CPU
       These are percentages of total CPU time.
       us: Time spent running non-kernel code.  (user time, including nice time)
       sy: Time spent running kernel code.  (system time)
       id: Time spent idle.  Prior to Linux 2.5.41, this includes IO-wait time.
       wa: Time spent waiting for IO.  Prior to Linux 2.5.41, included in idle.
       st: Time stolen from a virtual machine.  Prior to Linux 2.6.11, unknown.

```
# vmstat 2 3
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 109108      0 757720    0    0   156    41 1033 2116  7  5 88  0  0
 5  0      0 109000      0 757720    0    0     0    14 1851 3802  6  4 90  0  0
 0  0      0 108984      0 757728    0    0     0    51 1859 3835  6  4 90  0  0
```



### dstat

-c CPU

-d dist

-g page

-m memory

-p process

-r io



iostat



作业控制

​	job：

​		前台作业：终端启动，启动后一直占用终端

​		后台作业：启动后，脱离终端



​		终端运行的job，如果终端挂了，那么任务也没了。

​		ctrl+z  送往后台，进入停止态

​		jobs		

​		fg (front 调回前端)    fg 2

​		bg（调往后台）



​		

hping 发包可以指定速率和包个数，慎用！容易攻击别的主机！
