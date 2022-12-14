环境： Almalinux 8

# 问题
现在的service如下：
```
[Unit]
Description=webclient
After=syslog.target

[Service]

ExecStart=/usr/bin/java -jar /data0/java/world/webclient.jar
WorkingDirectory=/data0/java/world/
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```

	如果把springboot 做成systemd系统服务，控制台输出将会被 systemd 管理，而 systemd 默认把日志转发给 journal 管理，一旦日志过大，journal 会把之前其他服务的日志，给rotate掉，其他service就会报下面的警告
	
```
   Warning: Journal has been rotated since unit was started. Log output is incomplete or unavailable.

```

在官方文档有说明：
```
Note that, unlike when running as an `init.d` service, the user that runs the application, the PID file, and the console log file are managed by `systemd` itself and therefore must be configured by using appropriate fields in the ‘service’ script. Consult the [service unit configuration man page](https://www.freedesktop.org/software/systemd/man/systemd.service.html) for more details.
意思就是不同于init，pid 和控制台输出的日志，会被systemd管理
```


# 改进
使用改进后的 service，日志输出到指定目录，然后使用 logrotate 进行日志回滚
/etc/systemd/system/yczpro.service
```
[Unit]
Description=yczpro
After=syslog.target

[Service]
WorkingDirectory=/data0/java/world
ExecStart=/usr/local/jdk8/bin/java -jar /data0/java/world/yczpro.jar
StandardOutput=append:/data0/java/world/logs/yczpro.log
StandardError=append:/data0/java/world/logs/yczpro_error.log
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```

/etc/logrotate.d/yczpro
```
/data0/java/world/logs/*.log {
    daily
    rotate 30
    maxage 30
    copytruncate
    missingok
    notifempty
    compress
    dateext
    dateformat _%Y-%m-%d_%H-%M
    delaycompress
}
```
copytruncate # 用于还在打开中的日志文件，把当前日志备份并截断；是先拷贝再清空的方式，拷贝和清空之间有一个时间差，可能会丢失部分日志数据。
	像nginx，php，mysql 这种自带日志滚动的，可以不使用此项

delaycompress  # 表示，下一次滚动日志的时候压缩

### logrotate 使用
```
logrotate --help
Usage: logrotate [OPTION...] <configfile>
  -d, --debug               Don't do anything, just test and print debug messages
  -f, --force               Force file rotation
  -m, --mail=command        Command to send mail (instead of `/bin/mail')
  -s, --state=statefile     Path of state file
  -v, --verbose             Display messages during rotation
  -l, --log=logfile         Log file or 'syslog' to log to syslog
      --version             Display version information

Help options:
  -?, --help                Show this help message
      --usage               Display brief usage message

```


### 自定义 logrotate 时间

1.  配置文件放在 /etc/logrotate.d/指定目录下，或者 /etc/logrotate.daily.0
2.  配置crontab 
"59 23 * * * /usr/sbin/logrotate -f /etc/logrotate.daily.0/nginxLogrotate >/dev/null 2>&1"