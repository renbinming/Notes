国内

```
gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
gem sources -a http://mirrors.aliyun.com/rubygems/
mkdir ~/.rvm/user -p
echo "ruby_url=https://cache.ruby-china.com/pub/ruby" > ~/.rvm/user/db
curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -
curl -sSL https://rvm.io/pkuczynski.asc | gpg2 --import -

curl -sSL https://get.rvm.io | bash -s stable
```

rvm -v 

rvm list known # 查看所有的版本
rvm install 2.6 # 安装 ruby 2.6
rvm install 3.0

rvm use ruby-3.0.0 --default # 设置默认的ruby版本


## 安装 redis-dump 工具
 可以将redis的数据导出，可指定 db 导入和导出
 优先使用 ruby 3.0 以上，对于 redis 高版本

gem install redis-dump
装完会有
redis-dump    redis-load    redis-report 

使用：
```
指定db0 导出
redis-dump -u 127.0.0.1:6379 -a password -d 0 > /root/redis_qu2_dbto1.json
指定db1 导入
cat /root/redis_qu2_dbto1.json| redis-load -u 127.0.0.1:6379 -a password -d 1
```
