request 报文
```
GET /meta/world HTTP/1.1 
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9 
Accept-Encoding: gzip, deflate 
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6 Cache-Control: max-age=0 Connection: keep-alive Cookie: i_like_gitea=c1403b6b6fb2a752; lang=zh-CN; _csrf=DmqP4bpqwF_TDoGhIPLGAjf_evE6MTY2OTc4ODMwOTQ3NjE0NTk4Ng 
Host: localhost
Upgrade-Insecure-Requests: 1 
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36 Edg/107.0.1418.56
```




response 报文
```
HTTP/1.1 200 OK 
Server: nginx 
Date: Wed, 30 Nov 2022 06:58:16 GMT 
Content-Type: text/html; charset=UTF-8 
Transfer-Encoding: chunked 
Connection: keep-alive 
Vary: Accept-Encoding 
Cache-Control: no-cache 
Set-Cookie: _csrf=QinBVKG74h02nrKSkaEGOMfX1U06MTY2OTc5MTQ5NTkxOTExMTQ4Ng; Path=/; Expires=Thu, 01 Dec 2022 06:58:15 GMT; HttpOnly; SameSite=Lax Set-Cookie: macaron_flash=; Path=/; Max-Age=0; HttpOnly; SameSite=Lax 
X-Frame-Options: SAMEORIGIN 
Expires: Thu, 01 Jan 1970 00:00:01 GMT 
Content-Encoding: gzip
```


cookie


属性“**HttpOnly**”会告诉浏览器，此 Cookie 只能通过浏览器 HTTP 协议传输，禁止其他方式访问，浏览器的 JS 引擎就会禁用 document.cookie 等一切相关的 API，脚本攻击也就无从谈起了。

另一个属性“**SameSite**”可以防范“跨站请求伪造”（XSRF）攻击，设置成“SameSite=Strict”可以严格限定 Cookie 不能随着跳转链接跨站发送，而“SameSite=Lax”则略宽松一点，允许 GET/HEAD 等安全方法，但禁止 POST 跨站发送。


缓存
其实不止服务器可以发“Cache-Control”头，浏览器也可以发“Cache-Control”，也就是说请求 - 应答的双方都可以用这个字段进行缓存控制，互相协商缓存的使用策略。

当你点“刷新”按钮的时候，浏览器会在请求头里加一个“**Cache-Control: max-age=0**”。因为 max-age 是“**生存时间**”，max-age=0 的意思就是“我要一个最最新鲜的西瓜”，而本地缓存里的数据至少保存了几秒钟，所以浏览器就不会使用缓存，而是向服务器发请求。服务器看到 max-age=0，也就会用一个最新生成的报文回应浏览器。

Ctrl+F5 的“强制刷新”又是什么样的呢？

它其实是发了一个“**Cache-Control: no-cache**”，含义和“max-age=0”基本一样，就看后台的服务器怎么理解，通常两者的效果是相同的。


只要Cache-Control 允许缓存，就是静态资源，否则就是动态