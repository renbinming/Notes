```yml
version: '3.9'
services:
  #服务名称
  nginx:
    # 如果有build，则是设置构建镜像名，无则拉取镜像
    images: nginx:1.0
    build:
      # build 的目录
      context: ./dir
      # Dockerfile 名称
      dockerfile：Dockerfile-nginx
    # 内核能力（capabilities）
    cap_add:
      - NET_ADMIN
    cap_drop:
      - SYS_ADMIN
    # 启动容器名称
    container_name：nginx
    # 依赖
    depends_on: 
      - db
    # dns
    dns:
      - 114.114.114.114
    # 覆盖容器的 entrypoint
    entrypoint: ["sleep", "3000"]
    # 启动策略
    restart: always
    env_file:
      - ./dbs.env
    # 指定容器启动时的环境变量
    environment:
      TZ: 'Asia/Shanghai'
      JAVA_OPTS: >
         -Dhudson.security.csrf.GlobalCrumbIssuerConfiguration.DISABLE_CSRF_PROTECTION=true
    ports:
      -
    networks:
      - define-network
    volumes: 
      - type: volume 
      - source: /var/lib/mysql 
      - target: /var/lib/mysql
    # 或者
    volumes: - /var/lib/mysql:/var/lib/mysql
networks:
  define-network:
    driver:
    
```

volumes
bind 与 volume 的区别在于，是否受dockerd管理

使用docker-compose 会默认创建一个名为指定名称的网络
如 docker-compose.yml 在  test 文件夹下，并指定network名称为bridge则会生成一个
test_bridge
未指定名称则为
test_default
```
NETWORK ID     NAME                     DRIVER    SCOPE
eef336aa6dd9   bridge                   bridge    local
8e7665471b03   docker_gwbridge          bridge    local
7516c4c8f6fc   host                     host      local
3fd72442d8ad   jenkins_docker_default   bridge    local
a20359df55aa   none                     null      local
```
