17.05 +版本后可多阶段构建

使用场景——用于分离编译环境和运行环境

```dockerfile
# 设置了一个阶段名称 builder，编译后产生一个 http-server 文件
FROM golang:1.13 AS builder 
WORKDIR /go/src/github.com/wilhelmguo/multi-stage-demo/ 
COPY main.go . 
RUN CGO_ENABLED=0 GOOS=linux go build -o http-server . 

FROM alpine:latest 
WORKDIR /root/ 
COPY --from=builder /go/src/github.com/wilhelmguo/multi-stage-demo/http-server . 
CMD ["./http-server"]
```

指定阶段，将构建阶段停止在builder
$ docker build --target builder -t http-server:latest