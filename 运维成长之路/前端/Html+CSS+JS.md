
html 5 多了一些标签
css 配置
	字体 font 
	大小 size
	位置 position
	颜色 color

URL Universal Resource Locator
http://host.company.com:80/a/b/c.html?user=Alice&year=2008#p2
schema
	http
	https
	file
	websocket
	mailto
Hierarchical portion
	可以是文件路径 /a/b/c.html
	可以是接口 /user/create
	可以是路由，跳转
Parameters 参数
	?user=Alice&year=2008

URL 编码，除了字母数字_-.~ 都会被转换成16进制%xx
URL 只能使用 [ASCII 字符集](https://www.w3school.com.cn/charsets/ref_html_ascii.asp "HTML ASCII 参考手册") 通过因特网进行发送。
escape 转义


JS

对象的属性可以是函数
let obj = {count: 0}
obj.increment = function (amount){
    this.count += amount
    return this.count;
}
this

函数也是对象，可以有属性properties
function func(args){

}
有方法 methods
toString()
call()
bind()