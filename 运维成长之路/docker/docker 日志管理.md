默认容器日志在
/var/lib/docker/container/ {container_id}/{container_id}.json 里
也可以设置  journal 管理着日志
docker -f 容器名，读取的就是 json 日志
默认日志不做回滚
因此
需要设置，不然日志占用越来越大
```
/etc/docker/daemon.json

{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn"
  ],
  "log-driver": "json-file",
  "log-opts": {
        "max-size": "200m",
        "max-file": "14"
  },
  "data-root": "/data0/docker"
}
```

设置完成后，重启docker进程，并且容器需要重新创建！
已有容器不生效!
生效效果
```shell
docker inspect jira
"HostConfig": {
            "Binds": [
                "/data0/jira/data:/var/atlassian/application-data/jira:rw"
            ],
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {
                    "max-file": "14",
                    "max-size": "200m"
                }
            },
            "Net
```
