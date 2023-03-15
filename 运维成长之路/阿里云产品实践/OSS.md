
问题：传到oss的mp4文件，访问 变成直接下载，而不是预览

查看请求头

多了一个这个 Content-Disposition: attachment
2022年10月09号新开通 OSS 的用户，使用 OSS 域名访问，会强制加上
```
x-oss-force-download：true
Content-Disposition: attachment
```

解决方法：

绑定自定义域名
