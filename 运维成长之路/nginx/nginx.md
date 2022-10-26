nginx 的四大组成

​		编译后的 nginx 二进制文件

​		nginx conf 配置文件

​		access.log 日志文件

​		error.log 错误文件



nginx 源码包

contrib/vim/*  ~/.vim/ 可支持vim 可读性 nginx语法

--with-_module 默认不开启的模块，选中则开启



./configure --prefix=/usr/local/webserver/nginx 

make





autoindex on; #开启目录索引结构

log_format main #不同日志，记录不同格式

$limit_rate  # 限制客户端请求的速率



反向代理

proxy_set_header



对称加密

​		加密解密使用同一个密钥





#### 封多个ip请求

```shell
## map spider ip

map $http_x_forwarded_for $deny_status {
    default 0;
    ~192.168.0.1 1;
    ~192.168.0.3 1;
}

if ($deny_status = 1){
    return 444;
}
```

#### 防火墙禁用ip

```
firewall-cmd --zone=public --add-rich-rule="rule family='ipv4' source address='' reject" --permanent
firewall-cmd --reload
```

