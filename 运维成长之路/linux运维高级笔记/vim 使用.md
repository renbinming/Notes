vim 使用



打开文件

vim +5 /etc/profile  打开直接跳到指定行

vim +/start /etc/profile  打开跳到第一个搜索到匹配项的行



输入模式

i   光标处输入

a  光标后输入

o 光标下行输入

O 光标上行插入

I  行首输入

A 行尾输入



末行模式

:w /path/file   另存为文件



###  光标跳转

​		字符间跳转： 

​					h: 左   l: 右

​					j: 下   k:上           	19l   跳转19个字符

​		单词间跳转

​					w： 下一个单词词首

​					b：当前或后一个单词的词尾

​					e： 当前或前一个单词的词首

​		行首行尾跳转

​					^

​					0  绝对行首

​					$  

​	行间跳转

​				19G 跳转到第19行

​				G 最后一行



​				y

​				p

​				d

​				c



u 撤销   

ctrl+r  撤销上次撤销



vimtutor vim教程



### 末行模式



示例：

删除空格开头的字符

%s@^[:space:]\\+@@   

以空白字符开头的行首添加#

%s@^[[:space:]]\\+\[^[:space:]]@#&@   



enabled=0 替换成enabled=1

%s@\\(enabled\\|gpgcheck\\)=0@\1=1@g



### 多文件功能

​			:next 下一个文件

​			:prev 上一个

​			:first  :last

​			:qall    :wall

### 多窗口

ctrl+w 切换窗口，箭头→ ←

ctrl+w  s 水平分割窗口

v   垂直分割窗口



水平打开多个文件

vim -o file1 file2

垂直打开多个文件

vim -O file1 file2



/etc/vimrc  ~/.vimrc 配置文件



一些特性：

显示行号

​	set nu  set nonu

括号高亮

​	set sm   

自动缩进

​	set al

高亮搜索

​	set hlsearch

语法高亮

 syntax on

忽略字符大小写

set ic



tab 缩进4个字符

