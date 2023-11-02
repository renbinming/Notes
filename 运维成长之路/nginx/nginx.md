nginx 是多进程模型，一个 master 主进程，多个worker进程，但每个worker进程是单线程的

多进程是有独立的内存空间，进程间的数据交换需要通过 IPC(管道)、消息队列、信号量、Socket
多线程共享内存空间

epoll 模型，异步非阻塞


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


WAF
ModSecurity
https://github.com/SpiderLabs/ModSecurity

比较庞大，编译很费时间，所以最好编译成动态模块，在配置文件里用指令“load_module”加载：

```bash
load_module modules/ngx_http_modsecurity_module.so;
```

只有引擎还不够，要让引擎运转起来，还需要完善的防御规则，所以 ModSecurity 的第二个核心组件就是它的“**规则集**”。

ModSecurity 源码提供一个基本的规则配置文件“**modsecurity.conf-recommended**”，使用前要把它的后缀改成“conf”。

有了规则集，就可以在 Nginx 配置文件里加载，然后启动规则引擎：

```vbnet
modsecurity on;
modsecurity_rules_file /path/to/modsecurity.conf;
```

“modsecurity.conf”文件默认只有检测功能，不提供入侵阻断，这是为了防止误杀误报，把“SecRuleEngine”后面改成“On”就可以开启完全的防护：

```bash
#SecRuleEngine DetectionOnly
SecRuleEngine  On
```

基本的规则集之外，ModSecurity 还额外提供一个更完善的规则集，为网站提供全面可靠的保护。这个规则集的全名叫“**OWASP ModSecurity 核心规则集**”（Open Web Application Security Project ModSecurity Core Rule Set），因为名字太长了，所以有时候会简称为“核心规则集”或者“CRS”。

CRS 也是完全开源、免费的，可以从 GitHub 上下载：

```bash
git clone https://github.com/SpiderLabs/owasp-modsecurity-crs.git
```

其中有一个“**crs-setup.conf.example**”的文件，它是 CRS 的基本配置，可以用“Include”命令添加到“modsecurity.conf”里，然后再添加“rules”里的各种规则。

```bash
Include /path/to/crs-setup.conf
Include /path/to/rules/*.conf
```

你如果有兴趣可以看一下这些配置文件，里面用“SecRule”定义了很多的规则，基本的形式是“SecRule 变量 运算符 动作”。不过 ModSecurity 的这套语法“自成一体”，比较复杂，要完全掌握不是一朝一夕的事情，我就不详细解释了。

另外，ModSecurity 还有强大的审计日志（Audit Log）功能，记录任何可疑的数据，供事后离线分析。但在生产环境中会遇到大量的攻击，日志会快速增长，消耗磁盘空间，而且写磁盘也会影响 Nginx 的性能，所以一般建议把它关闭：

```bash
SecAuditEngine off  #RelevantOnly
SecAuditLog /var/log/modsec_audit.log
```
