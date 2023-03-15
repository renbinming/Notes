使用的是percona
1. 全备份数据
2. 停止原有版本mysql
systemctl stop mysqld
3. 移除原有mysql
rpm -qa | grep Percona-Server | xargs rpm -e --nodeps
4. 启用 8.0
 percona-release setup ps80
 5. 安装 percona 8.0
 yum install percona-server-server -y

替换配置文件，5.7 与 8.0 的配置文件不同
启动mysql

 8.0.16-7 版本后，直接启动