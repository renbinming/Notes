docker-compose 指定 JAVA_OPTS，启动未生效
![[Pasted image 20221129171722.png]]

使用镜像 jenkins/jenkins:2.378

而镜像使用jenkinsci/blueocean:1.25.7 则正常
![[Pasted image 20221129171830.png]]

实际上是修改了配置文件后，容器还是原来那个容器，需要删除掉原来的容器
docker-compose rm jenkins
docker-compose up -d

```yml
version: "3.1"
services:
  jenkins:
    user: root
    image: jenkins/jenkins:2.378
    container_name: jenkins
    restart: always
    environment:
      TZ: 'Asia/Shanghai'
      # 配置多个参数
      JAVA_OPTS: >
          -Dhudson.security.csrf.GlobalCrumbIssuerConfiguration.DISABLE_CSRF_PROTECTION=true
      #测试用，可关闭
      #    -Djava.util.logging.config.file=/var/jenkins_home/log.properties
    ports:
      - 8080:8080
      - 50000:50000
    volumes:
      - ./data:/var/jenkins_home/
      - /var/run/docker.sock:/var/run/docker.sock
```


### 关于容器删除后，maven 插件需要重新安装的问题
看看maven的配置文件，setting.xml
```xml
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->

```
默认是 ${user.home}/.m2/repository，由于容器使用root用户启动，因此 .m2 文件夹会在容器里的
/root/.m2 下
容器如果删除，maven 就不得不重新下载~~~

改一下
```
<localRepository>${env.JENKINS_HOME}/.m2/repository</localRepository>
```
docker cp jenkins:/root/.m2 data/

