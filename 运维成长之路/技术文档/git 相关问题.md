
## 1. git hook 简易版
服务端hook只有三种
pre-receive
update
post-receive

只有 post-receive 是在push到仓库后，执行的
pre-receive 是在 push 之前
update 是在 pre-receive 之后，依然在push成功之前，根据分支执行多次，其余都只一次
可以把 hook 放在post-receive.d 下
post-receive.d/post-receive
```
#!/bin/sh
#
# An example hook script to prepare a packed repository for use over
# dumb transports.
#
# To enable this hook, rename this file to "post-update".

HOST="xxx.xxx.xxx.xxx"
DIR="/data0/web/www.xxx.xxx"

ssh -i /root/.ssh/id_rsa  -o 'StrictHostKeyChecking no' root@$HOST "cd $DIR && git pull && chown www.www -R $DIR"
```

## 2. git 仓库单库过大问题

git 仓库，单库 近 2 g
使用 nginx 反代 到 gitea 的时候，clone 或 push 的时候会失败
排查后发现，/dev/shm 的占用过大，超过了限制

nginx 错误日志为
```
2023/03/28 10:17:01 [crit] 783098#0: *35414 pwritev() "/dev/shm/client_body_temp/0000001176" has written only 5089 of 8184, client: 192.168.1.144, server: gitea.wwwtech.com, request: "POST /git-receive-pack HTTP/1.1", host: ""
```

设置的 client_max_body_size 改为 2g
client_body_temp_path    /dev/shm/client_body_temp

程序运行原理，读取硬盘上的问题到内存，再从内存输出？
加大内存能解决，但是仓库以后过大了，不能一直加内存吧

设置为内存，可以

使用ssh 方式克隆代码，建议


## 3. 使用 LFS 解决大文件占用仓库的问题

git lfs 

https://github.com/Git-LFS/Git-LFS/wiki/Tutorial
https://gitee.com/help/articles/4235#article-header4
git lfs
git lfs install
 git config lfs.https://gitea.wwwtech.cn/meta/u3d-xrmeimei-client.git/info/lfs.locksverify true

git config lfs.url  修改 lfs URL 地址

### 跟踪lfs
git lfs track *.png
*.fbx
*.tga

### 迁移到 LFS 上，会修改历史提交
git lfs migrate import --include-ref=main --include="*.png"  

### 不修改历史提交

git lfs migrate import --no-rewrite *.png


### 对于 不同 LFS url，可以先不拉lfs，clone 后再拉lfs
GIT_LFS_SKIP_SMUDGE=1 git clone http://xxxxx/url
git lfs pull


LFS 解决了大文件存储的问题，但是带来了存储空间释放的问题。本地仓库删除了资源后，LFS 上并不会删除资源，可以后台删除，
另一个问题，非本地想拉取代码，拉取 lfs 资源，该如何拉取