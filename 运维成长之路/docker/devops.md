## DevOps 核心（更快，更频繁的交付更稳定的软件）

借助自动化工具

运维的职责：将开发团队的code 进行测试后部署上线。希望系统稳定安全运行

![image-20220909174618019](C:\Users\Binmi\AppData\Roaming\Typora\typora-user-images\image-20220909174618019.png)

 ## CI 过程  持续集成

构建 测试

开发提交代码到git库

pom.xml

Dockerfile



jenkins 配置 maven

settings.xml

改三个地方

```xml
=========改镜像===========
<mirror>
    <id>nexus-aliyun</id>
    <mirrorOf>central</mirrorOf>
    <name>Nexus aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
=====================================
<profile>
	<id>jdk-11</id>
	<activation>
		<activeByDefault>true</activeByDefault>
		<jdk>11</jdk>
	</activation>
	<properties>
		<maven.compiler.source>11</maven.compiler.source>
		<maven.compiler.target>11</maven.compiler.target>
		<maven.compiler.compilerVersion>11</maven.compiler.compilerVersion>
	</properties>
</profile>
=============
  <activeProfiles>
    <activeProfile>jdk11</activeProfile>
  </activeProfiles>
```



jenkins 使用 docker 运行

maven jdk tar.gz 解压到 jenkins 主机下，要让容器里的 jenkins 可以访问到路径



maven clean packages -DskipTests # 打 jar 包

publish over ssh 插件

把 target/*.jar docker/Dockerfile docker/docker-compose.yml 传到目标服务器

docker 目录下，把需要打包的文件都放进来

```dockerfile
FROM daocloud.io/library/java:8u40-jdk
COPY mytest.jar /usr/local
WORKDIR /usr/local
CMD java -jar mytest.jar
```

```yml
version: '3.1'
services:
  mytest:
    build:
      context: ./
      dockerfile: Dockerfile     
    image: mytest:v1.0.0
    container_name: mytest
    ports:
      - 8081:8080 # 避免与宿主机端口冲突
    
```





cd /usr/local/test/docker   # 使用绝对路径

mv ../target/*.jar ./

docker-compose down

docker-compose up -d --build

docker image prune -f 清理none 镜像



多构建几次，images 名称未变，但是会出现 none 的镜像



## CD 过程  持续交付 部署



jenkins 使用 git parameter

基于git tag  拉取和构建工程



git checkout $tag 



``` yml
version: '3.1'
services:
  db:
    image: postgres
    container_name: db
    ports:
      - 5432:5432
    networks:
      - sonarnet
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
  sonarqube:
    image: sonarqube:8.9.6-community
    container_name: sonaqube
    depends_on:
      - db
    ports:
      - 9000:9000
    networks:
      - sonarnet
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://db:5432/sonar
      SONAR_JDBC_USERNAME: sonar
      SONAR_JDBC_PASSWORD: sonar
networks:
  sonarnet:
    driver: bridge
  
```

sonaqube 连接 postgres





jenkins 容器使用 docker  

/var/run/docker.sock

chown root:root  /var/run/docker.sock

chmod o+rw   /var/run/docker.sock  安全？

```yml
version: '3.1'
services:
  jenkins:
    image:
    container_name:
    ports:
      - 8080:8080
      - 50000:50000
    volumes:
      - ./data/:/var/jenkins_home/
      - /var/run/docker.sock:/var/run/docker.sock
      - /usr/bin/docker:/usr/bin/docker
      - /etc/docker/daemon.json:/etc/docker/daemon.json
```



daemon.json



## pipeline

使用 Jenkinsfile，可以放到 git 库代码里面

有pipeline 语法生成器



k8s 1.19版本，支持dockers