
镜像越小越好，基础镜像能满足应用的运行环境即可
	例如：Java 类型的应用运行时只需要 JRE，不需要JDK

使用.dockerignore 文件
利用构建缓存，规则
-   从当前构建层开始，比较所有的子镜像，检查所有的构建指令是否与当前完全一致，如果不一致，则不使用缓存；
-   一般情况下，只需要比较构建指令即可判断是否需要使用缓存，但是有些指令除外（例如`ADD`和`COPY`）；
-   对于`ADD`和`COPY`指令不仅要校验命令是否一致，还要为即将拷贝到容器的文件计算校验和（根据文件内容计算出的一个数值，如果两个文件计算的数值一致，表示两个文件内容一致 ），命令和校验和完全一致，才认为命中缓存。

```Dockerfile
FROM centos:7
# 设置环境变量指令放前面
ENV PATH /usr/local/bin:$PATH
# 安装软件指令放前面
RUN yum install -y make
# 业务软件的配置，版本等经常变动的步骤放最后
```

时区设置
-   **Ubuntu 和Debian 系统**

Ubuntu 和Debian 系统可以向 Dockerfile 中添加以下指令：

```bash
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN echo "Asia/Shanghai" >> /etc/timezone
```

-   **CentOS系统**

CentOS 系统则向 Dockerfile 中添加以下指令：

```bash
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```


加快镜像构建速度
配置好repo文件，放置在build目录下
```bash
COPY CentOS7-Base-163.repo /etc/yum.repos.d/CentOS7-Base.repo
```


编写建议
1. RUN，复杂内容使用反斜杠结尾换行
2.  CMD 与 ENTRYPOINT
3. ADD和COPY
	COPY 可以更好地利用构建缓存，只支持基本的文件拷贝
	ADD 可以提取tar包
应该避免使用以下格式：

```bash
ADD http://pyropus.ca/software/memtester/old-versions/memtester-4.3.0.tar.gz /tmp/
RUN tar -xvf /tmp/memtester-4.3.0.tar.gz -C /tmp
RUN make -C /tmp/memtester-4.3.0 && make -C /tmp/memtester-4.3.0 install
```

下面是推荐写法：

```bash
RUN wget -O /tmp/memtester-4.3.0.tar.gz http://pyropus.ca/software/memtester/old-versions/memtester-4.3.0.tar.gz \
&& tar -xvf /tmp/memtester-4.3.0.tar.gz -C /tmp \
&& make -C /tmp/memtester-4.3.0 && make -C /tmp/memtester-4.3.0 install
```

4. WORKDIR 指定工作目录

示例：
[golang/Dockerfile at 4d68c4dd8b51f83ce4fdce0f62484fdc1315bfa8 · docker-library/golang (github.com)](https://github.com/docker-library/golang/blob/4d68c4dd8b51f83ce4fdce0f62484fdc1315bfa8/1.15/buster/Dockerfile)
[docker-nginx/Dockerfile at 9774b522d4661effea57a1fbf64c883e699ac3ec · nginxinc/docker-nginx (github.com)](https://github.com/nginxinc/docker-nginx/blob/9774b522d4661effea57a1fbf64c883e699ac3ec/mainline/buster/Dockerfile)
[docker-hylang/Dockerfile.python3.8-buster at f9c873b7f71f466e5af5ea666ed0f8f42835c688 · hylang/docker-hylang (github.com)](https://github.com/hylang/docker-hylang/blob/f9c873b7f71f466e5af5ea666ed0f8f42835c688/dockerfiles-generated/Dockerfile.python3.8-buster)
