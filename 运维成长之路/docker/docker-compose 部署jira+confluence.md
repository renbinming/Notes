## 1. 预先准备
目录结构如下：
atlassian-agent.jar ，懂的人自然懂
mysql-connector-java-8.0.29 .jar   需要的数据库连接包
Dockerfile 如下

![[Pasted image 20221205170251.png]]
confluence
```dockerfile
FROM atlassian/confluence-server:7.8.3
COPY "atlassian-agent.jar" /opt/atlassian/confluence/
COPY "mysql-connector-java-8.0.29.jar"  /opt/atlassian/confluence/confluence/WEB-INF/lib/
RUN echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/confluence/atlassian-agent.jar ${CATALINA_OPTS}"' >> /opt/atlassian/confluence/bin/setenv.sh
```
jira
```dockerfile
FROM atlassian/jira-software:9.3.1
COPY mysql-connector-java-8.0.29.jar /opt/atlassian/jira/lib/
COPY atlassian-agent.jar /opt/atlassian/jira/
RUN echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/jira/atlassian-agent.jar ${CATALINA_OPTS}"' >> /opt/atlassian/jira/bin/setenv.sh

```
docker-compose.yml
```yml
version: '3.9'
services:
  confluence:
    image: local/confluence:7.8.3
    build:
      context: ./confluence
      dockerfile: Dockerfile
    container_name: confluence
    restart: unless-stopped
    environment:
      TZ: Asia/Shanghai
      CATALINA_OPTS: -Xms512m -Xmx1024m -Datlassian.plugins.enable.wait=300
    ports:
      - 127.0.0.1:8090:8090
    volumes:
      - /data0/confluence/data:/var/atlassian/application-data/confluence
    depends_on:
      - db

  jira:
    image: local/jira:9.3.1 
    build:
      context: ./jira
      dockerfile: Dockerfile
    container_name: jira
    restart: unless-stopped
    environment:
      TZ: Asia/Shanghai
      CATALINA_OPTS: -Xms512m -Xmx1024m -Datlassian.plugins.enable.wait=300
    ports:
      - 127.0.0.1:8080:8080
    volumes:
      - /data0/jira/data:/var/atlassian/application-data/jira
    depends_on:
      - db

  db:
    image: mysql:8.0.29
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: 'yourpassword'
      TZ: Asia/Shanghai
    restart: unless-stopped
    ports:
      - 3306:3306
    volumes:
      - /data0/mysql/data:/var/lib/mysql
      - ./mysql/conf:/etc/mysql/conf.d # 目录对应目录
 
```

```cnf
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql

default-storage-engine=INNODB
character_set_server=utf8mb4
skip_name_resolve                 = 1
max_allowed_packet=256M

sql_mode                          = "NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION"
max_connect_errors           = 100000
max_connections              = 1000
thread_cache_size            = 50
back_log                     = 100
max_length_for_sort_data     = 1024
max_sort_length              = 1024
sort_buffer_size             = 4M
join_buffer_size             = 4M
read_rnd_buffer_size         = 4M
tmp_table_size               = 32M
max_heap_table_size          = 32M
interactive_timeout          = 31536000
wait_timeout                 = 31536000

### Binlog ###
server_id                    = 1
sync_binlog                  = 1
gtid_mode                    = ON
enforce_gtid_consistency     = ON
binlog_format                = ROW
binlog_expire_logs_seconds   = 604800
binlog_cache_size            = 8M
max_binlog_cache_size        = 10G
max_binlog_size              = 1G


###### Innodb ######
transaction_isolation                 = READ-COMMITTED
innodb_data_file_path                 = ibdata1:1024M:autoextend
innodb_buffer_pool_size               = 500M
innodb_log_file_size                  = 2G
innodb_log_files_in_group             = 2
innodb_log_buffer_size                = 8M
innodb_file_per_table                 = 1
innodb_stats_persistent               = 1
innodb_flush_log_at_trx_commit        = 1
innodb_max_dirty_pages_pct            = 75
innodb_lock_wait_timeout              = 120
innodb_print_all_deadlocks            = 1
innodb_buffer_pool_load_at_startup    = 1
innodb_buffer_pool_dump_at_shutdown   = 1
innodb_flush_method                   = O_DIRECT
innodb_doublewrite                    = 1
innodb_io_capacity                    = 2000
```

## 2. 启动容器

进入到目录中，docker-compose up -d

## 3. 创建数据库
CREATE DATABASE jira CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'jira'@'%' IDENTIFIED BY 'yourpassword';
grant all privileges on 'jira'.* to jira@'%';

## 4. 配置nginx反向代理

## 5. 后台配置jira和confluence

## 6. 配置key license

java -jar atlassian-agent.jar -d -m defatldgkx@zero.com -p conf -o http://192.168.1.165 -s 


JVM 参数，内存8G情况下，不是 -Xms512m -Xmx1024m 调整到2G，就没问题了，调整到2G，实际占用内存会超过2G！
5000 个问题以内，jira 调 768M 就够用了，默认2048m，一开启就把内存拿走了