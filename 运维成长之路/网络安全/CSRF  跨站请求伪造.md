CSRF  跨站请求伪造

原理：



只要能增删改，就有可能有漏洞



使用 burp 抓一个请求，CSRF poc ，



如何防御？

![image-20220802204531808](C:\Users\Binmi\AppData\Roaming\Typora\typora-user-images\image-20220802204531808.png)

referer 可以伪造

token可以防御