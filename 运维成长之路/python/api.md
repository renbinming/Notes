

```
pip install  mysqlclient==1.3.7
  pip install flask==2.3.2
  pip install flask-sqlalchemy==3.0.5
  pip install flask-mysqldb==1.0.1
  pip install oss2
```


### deploy 部署问题
如果 app.py 里用的
```
if __name__ == '__main__':
    main()
    process()
    app.run()
```
那么使用 gunicorn 等启动 app.py 的时候，不会执行 process() 这些函数
因为 代表的是直接调用 app.py 才会执行
而 gunicorn 是把 app.py 封装

```
pip install uwsgi 失败
      gcc: error: /opt/miniconda3/envs/test/lib/python3.8/config-3.8-x86_64-linux-gnu/libpython3.8.a: No such file or directory
      
```
系统环境是python3.6 的，python3-devel 也是3.6 的 

```
import os
import gevent.monkey
gevent.monkey.patch_all()

import multiprocessing

# debug = True
loglevel = 'debug'
bind = "0.0.0.0:7000"
# 在当前目录下创建一个log文件夹用来存放日志和运行的pid
accesslog = "logs/flask/access.log"
errorlog = "logs/flask/output.log"
# 进程守护True为开启后台模式，这里需要注意刚开始挂起最好使用False 看看脚本有没有反馈
#daemon = True

# 启动的进程数
workers = 2
worker_class = 'gevent'
x_forwarded_for_header = 'X-FORWARDED-FOR'
```