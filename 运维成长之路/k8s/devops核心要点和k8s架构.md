

# DevOps(一种思想理念)

下面的所有环节打通

CI：持续集成

​		开发提交代码，构建工具自动进行构建，测试的过程，出现问题了反馈给开发

CD： 持续交付， Delivery

​		测试完成后，打包成可运维的最终产品，自动交给运维

CD： 持续部署， Deployment

​		交付完成后，自动将产品发布到线上

需求计划——架构设计——开发——build——测试——运维



故障自愈



# k8s组件

## master/node

- master:  3个
  - Apiserver    接收并处理pod请求
  - Scheduler   调度器，调度pod，两级调度，先预选node，再选最优node
  - Controller-Manager  3个 高可用 监控controller

- node:  

  - kubelet   **健康状态检测**   容器引擎  docker podman
  - kube-proxy **动态管理 service** 生成iptables ipvs规则
  
  node 运行 pod（豌豆荚射手）
  
  一个pod一般放一个容器，同一pod共享net uts  volume资源
  
  也可以放多个，一个为主程序的容器，其他为辅助

## Label  用来固定 pod

​		一个资源对象可以有多个标签label，每个label 都是键值对

​		key 不能超过63个字符，字母、数字开头、_ - . 

​		value  可以为空，只能数字或字母开头及结尾

​		environment=production/qa/dev

​		release=stable/canary/alpha/beta

​		tier=frontend/backend

## Label Selector 标签选择器

​		等值关系：=、 == 、  ! =

​		集合关系：key in (value1,value2,...)

​						key notin

​						key

​						!key 不存在

​		kubectl get pods -l "release notin (canary,beta,alpha)"

​		nodeSelector 节点标签选择器



 关于 pod 

​		尽量用 controller 去管理，不要手工管理。

​		自主式pod，如果宿主机挂了，pod也就消亡了。有生命周期，ip，hostname都会变

​		控制器管理的  pod

##  pod 控制器

 - ReplicationController

   ​	不推荐使用  滚动更新，副本数量。升级回滚

### ReplicaSet 推荐使用 简写 rs 无状态

   Deployment 可以控制多个 rs

控制节奏与控制逻辑

   扩容缩容 修改yaml 或者 kubectl scale

   升级就修改 image 版本，并且需要 pod 重建

   假设预设副本数量为5个，那么更新逻辑一般是先删一个，再重新创建一个

   有时候需要先创建一个，再删一个，能多不能少

   更新方式可以多样。注意，一定要设置 readiness

   蓝绿发布，再新建一个 rs，然后把原来的 rs 去掉

   金丝雀发布，只发布一个，如果过段时间没问题，继续更新

![image-20220406195906973](E:\【01 运维成长之路】\k8s\rs-更新.png)

   ```yaml
   # kubectl get rs
   apiVersion: apps/v1
   kind: ReplicaSet
   metadata:
       name: myapp  # myapp-随机字符串
       namespace: default
   spec:
     replicas: 2  # 根据标签选择器设置副本数量, b
     selector:
         matchLabels:
             app: myapp
             release: canary
     template:  # 模板标签一定要跟选择器的一样。可以更多
         metadata:
             name: myapp-pod # 会被controller 设置的替换
             labels:
                 app: myapp
                 release: canary
                 environment: qa
         spec:
              containers:
              -   name: myapp-container
                  image: ikubernetes/myapp:v1
                  ports:
                  - name: http
                    containerPort: 80
                  livenessProbe:
   ```


### Deployment (简写deploy) 可控制rs  无状态

####    strategy 更新策略

   ​	Recreate 删一个建立一个

   ​    RollingUpdate 滚动更新

​			maxUnavailable 最大不可用百分比

   管理无状态应用

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: myapp-deploy
     namespace: default
   spec:
     replicas: 2
     selector:
       matchLables:
         app: myapp
         release: canary
     template:
       metadata:
         labels:
           app: myapp
           release: canary
       spec:
         containers:
           name: 
           image:
   ```

使用deployment 创建的 rs 的name 是 template 的 hash 值

myapp-deploy-69dbh6749d-



通过修改yaml 文件的内容，再 kubectl apply -f deploy-demo.yaml 从而实现更新升级



打补丁更新，传json格式的内容

kubectl patch deployment myapp-deploy -p '{"spec":{"replicas":5}}'



#### 金丝雀更新发布

kubectl set image deployment myapp-deploy myapp=ikubernetes/myapp:v3 && kubectl rollout pause deployment myapp-deploy  # 滚动更新，并且暂停，就只更新一个pod

kubectl rollout status deployment myapp-deploy # 查看滚动更新状态

kubectl rollout resume deployment myapp-deploy # 继续更新

kubectl rollout undo deployment myapp-deploy --to-revision=1 # 回滚版本

### StatefulSet 有状态 如redis cluster 每个个体要单独对待

redis cluster 通过分配slot的方式，0-5000，5001-10000，10000-16383，3个节点无法互相取代



有状态的，每个pod副本都是单独管理，有持久化

运维操作步骤极其复杂，需要单独处理，强大的运维技能

redis cluster : node 1 使用 0 - 5000 slot，2 使用 5001 - 10000 slot

mysql 主从 

### DaemonSet （简写ds）适用于守护进程 如日志收集应用agent

守护进程   部署系统级别的应用  **每个节点只运行1个副本，也支持滚动更新**  无状态  持续运行在后台  例如 filebeat  zabbix-agent

只支持先删一个，再更新一个 

只要新增节点，就会自动增加daemon，因为必须要存在一个副本

**直接使用宿主机的网络命名空间，就可以不用使用service**

**spec.hostNetwork** 使用宿主机的ip 就能访问到 pod 

 

kubectl expose deployment redis --port=6379 # 创建名为redis的svc

kubectl get svc

```yaml
---
#一个文件可以定义多个关联资源
Deployment 定义个redis
再定义个service
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
      role: logstore
  template:
    metadata:
      labels:
        app: redis
        role： logstore
    spec:
      containers:
      - name: redis
        image:
        ports:
        - name: redis
          containerPort: 6379
---
apiVersion:
kind: 
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: filebeat-ds
  namespace: default
spec:
  selector:
    matchLables:
      app: filebeat
      release: stable
  template:
    metadata:
      labels:
        app: filebeat
        release: stable
    spec:
      containers:
      -  name: filebeat
         image:
         env:
         - name: REDIS_HOST
           value: redis.default.svc.cluster.local
           		 服务名。命名空间。k8s 域名
         - name: REDIS_LOG_LEVEL
           value: info
```

进入filebeat pod 里printenv 可查看环境变量，ps aux 看pod里的进程



滚动更新 maxUnavaiable

kubectl set image --help # 查看支持滚动更新的控制器

kubectl set image daemonsets filebeat-ds filebeat=ikubernetes/filebeat:5.6.6  # 更新镜像版本

kubectl get pods -w 



### Job  备份任务，一次性完成后自动删除pod

一次性任务

### Cronjob  周期任务 不需要持续运行

需要解决前一个定时任务未完成，后一个请求又来了

HPA    自动监控，自动扩容

- HorizontalPodAutoscaler

TPR：Third Party Resources  1.2+ —— 1.7

CDR： Custom Defined Resources  1.8+

Operator： 任何一个系统都应该把用户当傻瓜，操作尽量简单

## AddOns： 附件，附加组件

### CoreDNS，kube-dns

### kube-proxy

一直在watch api-server ，一旦 service 变动，kube-proxy 都需要立即转换规则

service 是一些ip规则，利用 iptables  ipvs 等，组合dns 去调度后端 pod，都是私有地址。实际上负责流量转发

   客户端请求 服务 service，由service去找可用pod

   pod 的ip地址和hostname经常会变，但是 label 是固定的

  service 通过 label selector 调度 pod，并且可以探测 pod 的 ip



各个pod 都是通过 service 访问，service 的域名

客户端——> service ——>  controller ——> pod

为什么要通过service与pod 通信，是因为pod是会变的



## Service

三种工作代理模式

​	userspace  通过 kube-proxy   1.1-

​	iptables 通过 iptables 转发流量

​	ipvs   1.11+

类型：

​	ExternalName  外部服务，非集群内

​	ClusterIP 集群内访问，如访问redis

​	NodePort   对外访问，使用这个

​	LoadBalancer  与云厂商结合使用

创建service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: default
spec:
  selector:   # 标签
    app: redis
    role: logstor
  clusterIP: 10.97.97.97 #
  type: ClusterIP
  ports:
  - port: 6379  # svc 上的端口
    targetPort: 6379 # pod 里的端口
-------
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: default
spec:
  selector:
    app: myapp
    release: canary
  clusterIP: 10.99.99.99
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080  # 不指定也会动态分配，确定不要被占用
```

svc 资源记录：

SVC_NAME.NS_NAME.DOMAIN.LTD.

redis.default.svc.cluster.local.



kubectl apply -f svc.yaml

kubectl patch svc myapp -p '{"spec":{"sessionAffinity":"ClientIP"}}'  # 打补丁，更新，会话保持

删除service

kubectl delete svc myapp



### namespace

可以按照开发环境和生产环境区分

kubectl get namespace

kubectl create ns dev

kubectl create ns prod



# k8s三种网络模型

- 节点网络   node network 配置在设备上，事实上存在
- 集群网络   cluster network service 与 pod 不同网段，只存在于 iptables ipvs，只是规则
- pod网络    pod network 配置在设备上，事实上存在，即使是虚拟网卡   node里的pod 运行在同一网络

同一pod内的多个容器间用 lo 通信

各个pod之间的通信 

pod 与 service 之间的通信  



etcd 至少要3节点，保证高可用

master 也至少3个，保证高可用



CNI： container network interface

- flannel：网络配置，不支持网络策略，叠加网络
- calico： 网络配置，网络策略   基于BGP协议实现路由通信，配置复杂
- canel ： 网络配置+策略

资源类型+资源名称+命名空间

pod 启动后，会有就绪性检测，available 一开始是0

# 常用命令

kubectl get deployment

kubectl get pods -o wide

kubectl get pods -n kube-system -o wide # 指定命名空间

默认有三个命名空间

kubetctl get ns    # 查看命名空间

	- default     # 默认命名空间，不指定 -n 时都在这里
	- kube-public
	- kube-system # 系统级别的 pod 都在这里

排错

kubectl describe pod nginx -n test

镜像拉取失败，可以手动

请记住，pod 是运行在worker node 节点上的

docker pull 

  部署一个service，使用deployment 名称为 nginx-deploy 的

kubectl expose deployment nginx-deploy --name web --port=80 --target-port=80  --protocol=TCP

kubectl get service/svc  # 查看service

kubectl get svc -n kube-system # 查看 coredns 的 service 信息

kubectl describe svc nginx # 查看nginx  service的详细信息

​		有 label、 selector 等

kubectl get pods --show-labels  # 查看 标签

kubectl get pods -l app  # 过滤标签

kubectl get pods -L app,run  # 显示pod对应key的value

kubectl label pods pod-demo release=canary # 打标签

资源类型



kubectl get nodes --show-labels

​			标签key 可以有键前缀，必须是域名，不超过253字符

kubectl label nodes 

# 声明式资源清单定义

apiserver 只接收 JSON 格式的资源定义

yaml 格式提供配置清单，apiserver 可以自动将其转为 json 格式

只需要告诉机器最终结果

```yaml
apiVersion: group/version
	$ kubectl api-versions 可查看支持的version
	$ kubectl explain pods 可查看version和kind 该如何定义
	
kind: 资源类别
	Pod Deployment  StatefulSet 等都要注意大小写
metadata: 元数据
	name: pod-demo
	namespace: default
	labels:  # 传的map
		app: myapp
		tier: frontend  # 标签
spec:
	containers:
	- name: myapp
	  image: ikubernetes/myapp:v1
	- name: busybox
	  image: busybox:latest
	  command:  # 传的是list
	  - "/bin/sh"
	  - "-c"
	  - "sleep 3600"
	  - "echo $(date) >> /usr/share/nginx/html/index.html;sleep 5;"
	nodeSelector:
	  disktype: ssd  # 标签
```



```yaml
# kubectl get pod etcd-centos7-master -o yaml
# 看下系统里的 etcd 配置
apiVersion: v1
kind: Pod
metadata:
  annotations:  # 资源注释,只用于为对象提供元数据
    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.168.201.130:2379
    kubernetes.io/config.hash: be455af727036cf9cafbc44de3e3456b
    kubernetes.io/config.mirror: be455af727036cf9cafbc44de3e3456b
    kubernetes.io/config.seen: "2021-03-30T18:13:04.485287218+08:00"
    kubernetes.io/config.source: file
  creationTimestamp: "2021-03-30T10:13:05Z"
  labels:    # 标签
    component: etcd
    tier: control-plane
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .: {}
          f:kubeadm.kubernetes.io/etcd.advertise-client-urls: {}
          f:kubernetes.io/config.hash: {}
          f:kubernetes.io/config.mirror: {}
          f:kubernetes.io/config.seen: {}
          f:kubernetes.io/config.source: {}
        f:labels:
          .: {}
          f:component: {}
          f:tier: {}
        f:ownerReferences:
          .: {}
          k:{"uid":"ba6e688f-d96f-49af-9177-6a1ad0fef2e3"}:
            .: {}
            f:apiVersion: {}
            f:controller: {}
            f:kind: {}
            f:name: {}
            f:uid: {}
      f:spec:
        f:containers:
          k:{"name":"etcd"}:
            .: {}
            f:command: {}
            f:image: {}
            f:imagePullPolicy: {}
            f:livenessProbe:
              .: {}
              f:failureThreshold: {}
              f:httpGet:
                .: {}
                f:host: {}
                f:path: {}
                f:port: {}
                f:scheme: {}
              f:initialDelaySeconds: {}
              f:periodSeconds: {}
              f:successThreshold: {}
              f:timeoutSeconds: {}
            f:name: {}
            f:resources:
              .: {}
              f:requests:
                .: {}
                f:cpu: {}
                f:ephemeral-storage: {}
                f:memory: {}
            f:startupProbe:
              .: {}
              f:failureThreshold: {}
              f:httpGet:
                .: {}
                f:host: {}
                f:path: {}
                f:port: {}
                f:scheme: {}
              f:initialDelaySeconds: {}
              f:periodSeconds: {}
              f:successThreshold: {}
              f:timeoutSeconds: {}
            f:terminationMessagePath: {}
            f:terminationMessagePolicy: {}
            f:volumeMounts:
              .: {}
              k:{"mountPath":"/etc/kubernetes/pki/etcd"}:
                .: {}
                f:mountPath: {}
                f:name: {}
              k:{"mountPath":"/var/lib/etcd"}:
                .: {}
                f:mountPath: {}
                f:name: {}
        f:dnsPolicy: {}
        f:enableServiceLinks: {}
        f:hostNetwork: {}
        f:nodeName: {}
        f:priorityClassName: {}
        f:restartPolicy: {}
        f:schedulerName: {}
        f:securityContext: {}
        f:terminationGracePeriodSeconds: {}
        f:tolerations: {}
        f:volumes:
          .: {}
          k:{"name":"etcd-certs"}:
            .: {}
            f:hostPath:
              .: {}
              f:path: {}
              f:type: {}
            f:name: {}
          k:{"name":"etcd-data"}:
            .: {}
            f:hostPath:
              .: {}
              f:path: {}
              f:type: {}
            f:name: {}
      f:status:
        f:conditions:
          .: {}
          k:{"type":"ContainersReady"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
          k:{"type":"Initialized"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
          k:{"type":"PodScheduled"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
          k:{"type":"Ready"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
        f:containerStatuses: {}
        f:hostIP: {}
        f:phase: {}
        f:podIP: {}
        f:podIPs:
          .: {}
          k:{"ip":"192.168.201.130"}:
            .: {}
            f:ip: {}
        f:startTime: {}
    manager: kubelet
    operation: Update
    time: "2022-03-09T06:14:04Z"
  name: etcd-centos7-master  #
  namespace: kube-system     # 
  ownerReferences:
  - apiVersion: v1
    controller: true
    kind: Node
    name: centos7-master
    uid: ba6e688f-d96f-49af-9177-6a1ad0fef2e3
  resourceVersion: "441180"
  uid: 41a453a2-a913-4723-901d-9e485de495c5
spec:    # 最重要的,期望的状态
  containers: # required 必须要指明的
  - command:
    - etcd
    - --advertise-client-urls=https://192.168.201.130:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --initial-advertise-peer-urls=https://192.168.201.130:2380
    - --initial-cluster=centos7-master=https://192.168.201.130:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://192.168.201.130:2379
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://192.168.201.130:2380
    - --name=centos7-master
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    image: registry.aliyuncs.com/google_containers/etcd:3.4.13-0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /health
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 15
    name: etcd
    resources:
      requests:
        cpu: 100m
        ephemeral-storage: 100Mi
        memory: 100Mi
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /health
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 15
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/lib/etcd
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  hostNetwork: true
  nodeName: centos7-master
  preemptionPolicy: PreemptLowerPriority
  priority: 2000001000
  priorityClassName: system-node-critical
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    operator: Exists
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd
      type: DirectoryOrCreate
    name: etcd-data
status:   # 当前状态，current state 本字段由 k8s 集群维护
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2021-03-30T10:13:05Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2022-03-09T05:24:19Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2022-03-09T05:24:19Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2021-03-30T10:13:05Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://2e7d7de2cc04a41a9a3ca11ec6e6df1d5f87dca869aaa76c92455d7217d6e236
    image: registry.aliyuncs.com/google_containers/etcd:3.4.13-0
    imageID: docker-pullable://registry.aliyuncs.com/google_containers/etcd@sha256:4ad90a11b55313b182afc186b9876c8e891531b8db4c9bf1541953021618d0e2
    lastState:
      terminated:
        containerID: docker://1e565998da8e3c3cb3134ad034254b272d53fff62329fd3df0f12d076c5a1138
        exitCode: 0
        finishedAt: "2022-01-20T15:12:23Z"
        reason: Completed
        startedAt: "2022-01-20T08:47:05Z"
    name: etcd
    ready: true
    restartCount: 13
    started: true
    state:
      running:
        startedAt: "2022-03-09T05:22:28Z"
  hostIP: 192.168.201.130
  phase: Running
  podIP: 192.168.201.130
  podIPs:
  - ip: 192.168.201.130
  qosClass: Burstable
  startTime: "2021-03-30T10:13:05Z"
```



alpha 内测   beta 公测 stable 稳定版



查看定义  yaml 的 说明

kubectl explain pods

kubectl explain pods.metadata

Object 表示一个对象



kubectl explain deployment

KIND:     Deployment
VERSION:  apps/v1 

说明kind 和 version 都是固定的



排错

正常运行

kubectl logs etcd-centos7-master -n kube-system # 查看日志



|      | Docker     | K8s     |
| ---- | ---------- | ------- |
|      | Entrypoint | command |
|      | Cmd        | args    |

Cmd 是传给 entrypoint 的

**Container 的 优先于 image 设置**







pod 的生命周期

初始化容器 init c ——> init c2 ——>主容器 main container

​											post start   /存活状态检测  liveness probe/ 就绪状态检测 readiness probe/pre stop

## Containers

##  livenessProbe 三种探针 存活状态检测

- ​	httpGet

   HTTPGetAction

- ​	tcpSocket

  TCPSocketAction

- ​	exec

  ExecAction



默认10秒一次，探测 3次，超时1秒

initialDelaySeconds 容器启动后，探测之前的时间（ 容器启动初始化需要时间，立即探测有可能失败）

## readinessProbe  就绪状态检测 必须做

必须要做的，因为不做的话，很有可能有部分会访问到失败的请求

用户访问到service，service 通过 label 选择pod，而pod 未作检测的将会直接连接到service上，提供服务。而做了检测，只有检测成功，才会提供服务

```yaml
spec:
    containers:
    - name: c1
      image:
      imagePullPolicy: Always、Never 、IfNotPresent
      lifecycle:
        postStart: # 启动容器后需要做的操作
          exec:
            command: ["/bin/sh","-c",""]
      command: ["/bin/httpd"]
      args: ["-f","-h /data/web/html"]
```

lifecycle

​	preStart

​    postStart

​    preStop

​    postStop



Helm 不能只依靠这个，只依赖这个只会成为下一个背锅的
