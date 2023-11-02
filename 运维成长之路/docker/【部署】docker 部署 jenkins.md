注意jdk 1.5 之后，不需要配置 CLASS_PATH 和 JRE_HOME
只需要配置 JAVA_HOME和PATH


/data0/jenkins_docker
docker-compose.yml
```
version: "3.1"
services:
  environment:
    TZ: Asia/Shanghai
  jenkins:
    user: root
    image: jenkins/jenkins:2.378
    container_name: jenkins
    ports:
      - 8080:8080
      - 50000:50000
    volumes:
      - ./data:/var/jenkins_home/
      - /var/run/docker.sock:/var/run/docker.sock
```

docker-compose up -d

修改jenkins插件镜像源
vim data/hudson.model.UpdateCenter.xml
```
<?xml version='1.1' encoding='UTF-8'?>
<sites>
  <site>
    <id>default</id>
    <url>https://mirrors.huaweicloud.com/jenkins/updates/update-center.json</url>
  </site>
</sites>
```

把maven 和 jdk 放到映射的 data下
![[Pasted image 20221118165139.png]]
配置maven源、maven/conf/settings.xml
```
    <mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
    </mirror>
```

配置maven jdk
```
<profiles>
    <profile>
        <id>jdk-17</id>
        <activation>
            <activeByDefault>true</activeByDefault>
            <jdk>17</jdk>
        </activation>
        <properties>
            <maven.compiler.source>17</maven.compiler.source>
            <maven.compiler.target>17</maven.compiler.target>
            <maven.compiler.compilerVersion>17</maven.compiler.compilerVersion>
        </properties>
    </profile>
</profiles>
```
jenkins 配置nginx反向代理

配置jdk 和maven
全局配置

![[Pasted image 20221118165004 1.png]]

新建任务，配置git 只读用户凭据

jenkins 插件：
ssh
publish over ssh
ssh pipeline steps
git
git parameters
Generic Webhook Trigger

创建ssh凭据
先生成RSA格式的私钥与公钥文件
-m PEM 指定为rsa格式，不然的话，almalinux 生成的是OPENSSH
ssh-keygen -m PEM -t rsa -C 'jenkins' -b 2048

-----BEGIN RSA PRIVATE KEY----- 这种可以用
-----BEGIN OPENSSH PRIVATE KEY-----   这种不能用

![[Pasted image 20221118094802.png]]
或者在添加SSH Remote host 的时候添加凭据



配置publish over ssh
![[Pasted image 20221122105729.png]]

![[Pasted image 20221122105820.png]]
![[Pasted image 20221122105908.png]]
