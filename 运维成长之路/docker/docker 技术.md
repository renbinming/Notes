主机级虚拟化

​		内核级别隔离

vmware workstation

- Type-I： 直接在硬件装虚拟机管理器，不用装系统，hypervisor

- Type-II： 宿主机装管理软件，vmware workstation , virutal box

- 其他：kvm

  

容器级虚拟化

​		共享宿主机内核kernel



能实现隔离，但是资源开销大

减少中间环节，提升效率

**生产力**



用户空间，运行进程

kernel

host os

硬件平台



隔离：每个进程都以为自己是唯一用户

## Namespace 视图隔离



| namespace | 系统调用参数  | 隔离内容                   | 内核版本 |
| --------- | ------------- | -------------------------- | -------- |
| UTS       | CLONE_NEWUTS  | 主机名和域名               | 2.6.19   |
| IPC       | CLONE_NEWIPC  | 信号量、消息队列和共享内存 | 2.6.19   |
| PID       | CLONE_NEWPID  | 进程编号                   | 2.6.24   |
| Network   | CLONE_NEWNET  | 网络设备、网络栈、端口等   | 2.6.29   |
| Mount     | CLONE_NEWNS   | 挂载点（文件系统）         | 2.4.19   |
| User      | CLONE_NEWUSER | 用户和用户组               | 3.8      |



主机名和域名 uts

mount 挂载树

IPC   Inter-Process Communication，进程间通信，

PID，User，Net

文件树 + 进程树

## Cgroups (Control Groups) 资源限制

CPU 按比例或按核数分配资源

blkio: 块设备io

cpu

cpuacct：CPU资源使用报告

cpuset： 多处理器平台上的CPU集合

devices：设备访问

freezer： 挂起或恢复任务

memory：内存用量及报告

perf_event： 对cgroup 中的任务进行统一性能测试

net_cls： cgroup中的任务创建的数据报文的类别标识符



linuxC container 底层使用 lxc 基于 image 启动容器



docker 镜像仓库

docker-ce 

​	配置文件：/etc/docker/daemon.json

​	镜像加速器：

```
{
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
```





```shell
# docker info 详细信息
Containers: 26  # 当前运行容器数量
 Running: 0
 Paused: 0
 Stopped: 26
Images: 12
Server Version: 1.13.1   
Storage Driver: overlay2  # 存储驱动后端 aufs/overlay2
 Backing Filesystem: xfs
 Supports d_type: true
 Native Overlay Diff: true
Logging Driver: journald
Cgroup Driver: systemd
Plugins: 
 Volume: local
 Network: bridge host macvlan null overlay
Swarm: inactive
Runtimes: docker-runc runc
Default Runtime: docker-runc
Init Binary: /usr/libexec/docker/docker-init-current
containerd version:  (expected: aa8187dbd3b7ad67d8e5e3a15115d3eef43a7ed1)
runc version: 66aedde759f33c190954815fb765eedc1d782dd9 (expected: 9df8b306d01f59d3a8029be411de015b7304dd8f)
init version: fec3683b971d9c3ef73f284f176672c44b448662 (expected: 949e6facb77383876aeff8a6944dde66b3089574)
Security Options:
 seccomp
  WARNING: You're not using the default seccomp profile
  Profile: /etc/docker/seccomp.json
Kernel Version: 3.10.0-1160.21.1.el7.x86_64
Operating System: CentOS Linux 7 (Core)
OSType: linux
Architecture: x86_64
Number of Docker Hooks: 3
CPUs: 2
Total Memory: 1.777 GiB
Name: CentOS7-Master
ID: QAQY:B4P3:KXWQ:D4UJ:UQNQ:3YLU:UUH7:S2NU:NLL3:Q7AJ:SQ6W:2EDQ
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
Experimental: false
Insecure Registries:
 127.0.0.0/8
Live Restore Enabled: false
Registries: docker.io (secure)
```



alpine：微型发行版，体积小，空间小，缺少调试工具

尽量自己做镜像

busybox 数百种命令的合集

​		ln -s busybox ls  即可有一个命令ls

busybox 的镜像

docker0



**image 镜像含有启动容器所需要的文件系统及其内容**

- 采用分层构建机制，最底层为 bootfs，用于系统引导的文件系统，容器启动后会被卸载

- rootfs 

   传统模式，会挂载为只读模式，自检完成后为读写

  docker种，由内核挂载为只读模式，aufs 添加可读写层

```shell
常用命令

docker inspect 容器名 # 查看容器详细信息

docker [container] ps -a

docker run --name web1 -d nginx:1.14-alpine

docker exec -it kvstor1 /bin/sh

docker tag  38asdfaf2r88g  myimage:v9.1-1  可以打多个标签

docker commit -a "message" -c 'CMD ['/bin/httpd','-f',"-h","/data/html"]' -p 容器名 标签名

docker pull quay.io/coreos/flannel:v0.10.0-amd64



docker save -o  myimages.gz tag1 tag2 直接打包镜像

			load  直接导入镜像

docker network inspect web1
docker container inspect web1
```



镜像仓库  quay.io

阿里云的容器镜像服务



## 容器网络

linux 内核可以虚拟出网卡接口，支持二层和三层

桥接：物理网卡当作交换机，与虚拟网卡连接，转发虚拟机的流量

大规模不用桥接，容易网络风暴

NAT：私有地址转到物理机的公有地址

出去时候 SNAT   到另一台主机做 DNAT



docker0  和 docker容器 是一对虚拟网卡，docker0 就相当于交换机。一个NAT桥

brctl show  

wget -O - -q http://172.17.0.2 相当于 curl http://172.17.0.2

#### 容器间通信



1. 默认创建的容器都连接到 docker0 网桥，docker0 下的容器都可以使用 ip 通信，如果使用 容器名称 通信，不能使用默认的 docker0 桥
2. docker0 桥，不能使用大规模，而是需要自己创建一个网络



#### docker 中网桥类型

bridge host null



#####  bridge 如docker0 桥

NAT网络，默认172.17.0.0，虚拟网络设备，一半在容器，一半在docker 0 

自定义docker0桥的网络属性 /etc/docker/daemon.json

```
{
    "bip": "10.0.0.1/16",
    "dns": ["114.114.114.114","8.8.8.8"]
    
}
```



##### host：开放式容器，共享物理机的namespace

##### none：封闭式容器，closed container 只有lo口

创建网桥

docker network create -d bridge (默认) 网络名称

查看网络

docker network ls

查看桥的信息

docker network inspect bridge

删除网络

docker network rm aa_web

删除未使用的网络

docker network prune 网络名称

在指定网络中运行多个容器

​		启动容器时指定

​        docker run -d --network 网络名称（需要先创建)

​		启动容器后添加

​        docker network connect  网络名称  容器id|容器名

#### 联盟式网络

joined container，共享部分namespace

UTS, NET IPC

```
docker run -it --name b1 --rm busybox:latest

docker run -it --name b2 --network container:b1 --rm busybox
ifconfig看到的ip是同一个ip，b2 容器与 b1 容器共享网络
```





### 交互式命令

-p 暴露端口，随机动态端口

docker run --name myweb --rm -p 80 httpd:v0.2

##### 查看端口

```shell
iptables -t nat -vnL 

docker port myweb #查看动态端口

-p <hostPort:containerPort  固定端口

-p 192.168.123.20::80

-p 80:80

-p 192.168.123.20:80:80
```

##### 查看容器网络

```shell
[root@DOCKER-HARBOR harbor]# docker network ls
NETWORK ID     NAME            DRIVER    SCOPE
edc36ad740ff   bridge          bridge    local
f19876f1c5ed   harbor_harbor   bridge    local
1d61ca1e8497   host            host      local
06f90ee3c54b   none            null      local

三种网络
bridge
host
none
```



##### 查看容器进程

```shell
[root@DOCKER-HARBOR harbor]# docker  top b2935087a917  #
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                41630               41611               4                   13:21               ?                   00:00:07            /opt/java/openjdk/bin/java -Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -classpath /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/tomcat -Dcatalina.home=/usr/local/tomcat -Djava.io.tmpdir=/usr/local/tomcat/temp org.apache.catalina.startup.Bootstrap start
```



连接远程docker  docker -H 192.168.123.128:2375 image ls



同一宿主机的不同网段容器通信，直接打开ip_forward 

iptables 规则允许通信



## 存储卷 volume

镜像层，只读+读写层

修改已存在的文件，文件会从只读，copy 一份到读写层，文被标记为删除，隐藏了而已



必须在容器启动时指定 -v 

1.  宿主机路径:容器内路径

   创建了后，容器内的原始数据会丢失

2. 使用别名方式，可以存在，也可以不存在

   可以保留容器的原始数据

   

   

   

```shell
   

# 查看所有存储卷

[root@DOCKER-HARBOR ~]# docker volume ls
   DRIVER    VOLUME NAME
   local     1c2fa36eee3a986dc5ab52b3041bcb4405d9c156291959f25b113741af1ab45c
   local     
# 查看数据卷详细内容


[root@DOCKER-HARBOR ~]# docker inspect 数据卷别名

# docker volume inspect 7a2330d2aa898278aad3ff5a30a60915ed628fd914c8f579aa2b6e4773bf60b7
[
    {
        "CreatedAt": "2022-09-10T14:31:19+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/data0/docker/volumes/7a2330d2aa898278aad3ff5a30a60915ed628fd914c8f579aa2b6e4773bf60b7/_data",
        "Name": "7a2330d2aa898278aad3ff5a30a60915ed628fd914c8f579aa2b6e4773bf60b7",
        "Options": null,
        "Scope": "local"
    }
]
```

   





docker 管理的卷，自己管理

​		 docker run --name b2 -it -v /data  busybox

​		容器 /data 目录 对应宿主机上的 /var/lib/docker/volumes/....

docker run --name b1 -it -v 宿主机目录:docker目录  busybox:latest

​		/data0/volumes 对应 容器里的 /data 目录



docker run --volume-from



​		docker inspect -f  {{.Mounts}} b2

​		docker inspect -f {{.NetworkSettings.IPaAddress}} b2

有状态需要持久 mysql





   ```

# 设定支撑容器，不需要启动，只需要创建即可
docker run --name inframecontainer -v /data/infracon/volume:/data/web/html busybox

# nginx 加入到支撑容器
docker run --name nginx --network container:infracon --volumes-from infracon busybox

   ```

## Dockerfile 镜像构建

Dockerfile

```dockerfile
FROM busybox:latest
MAINTAINER 
ENV  DOC_ROOT /data/web/html

COPY index.html $DOC_ROOT
COPY yum.repos.d /etc/yum/repos.d/
#ADD http://nginx.org/download/nginx-1.20.2.tar.gz /usr/local/src # 会下载
#ADD nginx-1.20.2.tar.gz  /usr/local/src # 会展开
WORKDIR /usr/local/src/
ADD nginx-1.20.2.tar.gz ./
# 运行于镜像文件构建中
RUN cp aaa /tmp
VOLUME /data/mysql/
EXPOSE 80/tcp
# CMD 只有最后一个生效，运行在容器启动时
CMD 
```

docker build -t tagname:labelname .



${NAME:-tom} 如果 NAME 没值，就把值 tom 赋给 NAME

```

```

●RUN
	●用于指定docker build过程中运行的程序，其可以是任何命令
	●RUN <command>或
	●RUN [<executable>), "<param1>", "<param2>"]
	● **第一种格式中，<command> 通常是一个shell 命令**， 且默认以"/bin/sh -c”来运行它，这意味着此进程
在容器中的PID不为1，不能接收Unix信号，因此，当使用docker stop <container>命令停止容器
时，此进程接收不到SIGTERM信号;
	●第二种语法格式中的**参数是一个JSON格式的数组**，其中<executable>为要运行的命令，后面的
<paramN>为传递给命令的选项或参数;然而，此种格式指定的命令不会以"/bin/sh -c"来发起
	**因此常见的shell 操作如变量替换${} 以及通配符(, *等)替换将不会进行**;不过，如果要运行的命令
	依赖于此shell特性的话，可以将其替换为类似下面的格式。
	●RUN ["/bin/bash", "-c", "<executable>", "<param1>"]

示例：

CMD  /bin/httpd -f -h  /data/web/html

CMD ["/bin/httpd",'-f','-h /data/web/html']

CMD ["/bin/sh", "-c", "/bin/httpd",'-f','-h ${WEB_DOC_ROOT}' ]



exec COMMAND  执行命令，退出shell，进程号设为为1

docker exec -it   web2 /bin/sh 



**推荐使用数组方式，可以在运行容器时传参**

CMD ['参数1'，''参数2]

ENTRYPOINT 

参数传递给 entrypoint

差别

**运行CMD时，覆盖Dockerfile里的命令**

**docker run mycentos:15 ls /data0**

**运行ENTRYPOINT，**

**docker run --entrypoint=ls mycentos:15 /data0** 

前面是指令，后面是参数



结合使用

ENTRYPOINT ['ls']   # 写容器的固定指令

CMD ['/apps/data']   # 给entrypoint 传递参数



ONBUILD 触发器，别人使用镜像时触发的动作

​					一般执行RUN ADD

```dockerfile
FROM openjdk:8-jre
ENV APP_PATH=/apps
WORKDIR $APP_PATH
ADD ems-0.0.1-SNAPSHOT.jar $APP_PATH/apps.jar  # 添加的时候改名字
EXPOSE 8989
ENTRYPOINT ['java', '-j']
CMD ['apps.jar']
```





### daemon.json

```json
{
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn"
  ],
    "insecure-registries": ["192.168.201.134:80"],
  "data-root": "/data0/docker"  # 修改默认存储路径

}
```





docker 镜像推送到harbor流程

docker login -u 用户名 -p  harbor地址

docker tag 自定义镜像  harbor地址:harbor项目名称:harbor镜像名称

docker push harbor地址:harbor项目名称:harbor镜像名称





删除多余tag

docker rmi 192.168.201.134:80/registry/nginx:alpine

docker stop container_id  # 根据容器id删除，先停止容器

docker rm container_id  # 删除容器



docker start container_id|container_name



docker save  nginx1 -o nginx.tar

docker load -i nginx.tar

