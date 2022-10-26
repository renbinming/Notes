bash

解释型语言

通过运行一个子shell进程实现的



### 运行原理

```test.sh
#!/bin/bash

echo "hello"
```

实际上是运行 /bin/bash 进程，用这个进程解析脚本内容，然后在bash里执行命令



在终端下，sh test.sh 可不加执行权限，是认为test.sh为参数

而./test.sh 需要添加执行权限，是因为 shell 把它当成命令，来解释运行

shell 会将所有的命令，通过特定的解析来执行，读取PATH里的路径

直接运行 test.sh 会报错，因为找不到它的路径



### 配置文件

​	profile类： 为交互式登录的shell进程提供配置

​			全局：对所有用户生效

​						/etc/profile

​						/etc/profiled.d/*.sh

​			个人：  ~/.bash_profile

​			功能：

​					1、用于定义环境变量

​					2、 运行命令或者脚本

​	bashrc 类： 非交互式

​				全局：/etc/bashrc

​				~/.bashrc

​				功能：

​						1、定义本地变量

​						2、 定义命令别名

​	登录类型：

​			交互式： 直接通过终端登录打开的shell进程

​							使用su - root命令进行的登录切换

​			非交互式登录

​							su root

​							图形界面下打开的终端

​							运行脚本



交互式

/etc/profile ——> /etc/profile.d/* ——> ~/.bash_profile ——>  ~/.bashrc ——>  /etc/bashrc

非交互式

~/.bashrc ——> /etc/bashrc ——> /etc/profile.d/*

立即生效 source .bashrc



算术运算格式：

let VAR=$num1+$num2

​	let += 1

VAR=$[$num1+$num2]

VAR=$(($num1+$num2))

VAR=$(expr $num1 + $num2)



&& 如果左边的执行成功，则执行右边的命令

||   如果左边的执行失败，则执行右边的命令

-a 与

-o 或

！ 非



特殊变量

$0 路径

$# 参数

$*  

$@



LIST 生成方式

​	{1..10}

​	seq 1 2 10

​	