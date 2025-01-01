# Docker-redis-cluster

这仅仅是一个普通的github项目，用于了解docker-compose

## Makefile

项目采用`make-file`启动，非常`legacy`;

看一下具体文件：

```makefile
help:
	@echo "Please use 'make <target>' where <target> is one of"
	@echo "  build         builds docker-compose containers"
	@echo "  up            starts docker-compose containers"
	@echo "  down          stops the running docker-compose containers"
	@echo "  rebuild       rebuilds the image from scratch without using any cached layers"
	@echo "  bash          starts bash inside a running container."
	@echo "  rm            Remove stopped containers."
	@echo "  restart       Restart services."
	@echo "  cli           run redis-cli inside the container on the server with port 7000"

build:
	docker-compose build

up:
	docker-compose up

down:
	docker-compose stop

rebuild:
	docker-compose build --no-cache

bash:
	docker-compose exec redis-cluster /bin/bash
	
remove:
	docker-compose rm

restart:
	docker-compose restart

cli:
	docker-compose exec redis-cluster /redis/src/redis-cli -p 7000
```

基本就是对 `docker-compose`命令进行了一层封装；

## Dockerfile

```dockerfile
# 基于 Redis 7.2.5 构建，使用指定的镜像哈希以确保版本一致性
FROM redis@sha256:e422889e156ebea83856b6ff973bfe0c86bce867d80def228044eeecf925592b

# 设置环境变量
ENV HOME /root                       # 设置默认用户目录
ENV DEBIAN_FRONTEND noninteractive   # 禁用交互式模式，适用于自动化构建

# 安装系统依赖
RUN apt-get update -qq && \ 
    apt-get install --no-install-recommends -yqq \
      net-tools supervisor ruby rubygems locales gettext-base wget gcc make g++ build-essential libc6-dev tcl && \
    apt-get clean -yqq
# - `net-tools`: 网络工具，例如 `ifconfig` 等。
# - `supervisor`: 用于管理进程的工具。
# - `ruby` 和 `rubygems`: 用于安装 Ruby gem 包，例如 Redis gem。
# - `locales` 和 `gettext-base`: 用于语言和区域设置。
# - `wget`: 用于下载文件。
# - `gcc` 和其他编译工具: 用于构建 Redis 源码和依赖。

# 确保 UTF-8 语言环境和本地化
RUN locale-gen en_US.UTF-8
ENV LANG       en_US.UTF-8
ENV LC_ALL     en_US.UTF-8
# 设置语言和区域为英文，编码为 UTF-8，避免因字符编码问题导致的错误。

# 配置 SSL 证书路径，解决 SHA1 安全性问题
ENV SSL_CERT_FILE=/usr/local/etc/openssl/cert.pem

# 安装 Redis gem，指定版本为 4.1.3
RUN gem install redis -v 4.1.3
# - `redis gem`: 提供与 Redis 交互的 Ruby 客户端。

# 下载并构建 Redis 的最新 7.2 分支版本
ARG redis_version=7.2

RUN wget -qO redis.tar.gz https://github.com/redis/redis/tarball/${redis_version} \
    && tar xfz redis.tar.gz -C / \
    && mv /redis-* /redis
# - 下载指定版本的 Redis 源码压缩包。
# - 解压源码到根目录。
# - 重命名解压后的目录为 `/redis`。

RUN (cd /redis && make)
# - 进入 `/redis` 目录并执行 `make` 命令以编译 Redis。

# 创建 Redis 配置目录和数据目录
RUN mkdir /redis-conf && mkdir /redis-data
# - `/redis-conf`: 存放 Redis 的配置模板文件。
# - `/redis-data`: 存放 Redis 的数据。

# 复制 Redis 和 Sentinel 的模板配置文件到镜像中
COPY redis-cluster.tmpl /redis-conf/redis-cluster.tmpl
COPY redis.tmpl         /redis-conf/redis.tmpl
COPY sentinel.tmpl      /redis-conf/sentinel.tmpl
# - 这些模板文件用于动态生成配置文件，例如 Redis 集群配置和哨兵配置。

# 添加启动脚本
COPY docker-entrypoint.sh /docker-entrypoint.sh
# - 入口脚本，用于在容器启动时执行初始化逻辑。

# 添加用于生成 Supervisor 配置文件的脚本
COPY generate-supervisor-conf.sh /generate-supervisor-conf.sh
# - 该脚本基于环境变量生成 Supervisor 的配置文件。

# 为入口脚本赋予执行权限
RUN chmod 755 /docker-entrypoint.sh

# 暴露 Redis 和其他服务的端口
EXPOSE 7000 7001 7002 7003 7004 7005 7006 7007 5000 5001 5002
# - `7000-7007`: Redis 集群的默认端口。
# - `5000-5002`: 自定义端口（可能用于管理或其他服务）。

# 设置容器的默认入口点
ENTRYPOINT ["/docker-entrypoint.sh"]
# - 指定默认入口脚本。

# 设置容器默认的启动命令
CMD ["redis-cluster"]
# - 默认启动 Redis 集群模式。
```

`ENTRYPOINT`和`CMD`的关系非常微妙：

1. 单独只有 `CMD["echo","hello"]`,会被识别出主命令 `echo`以及**默认参数** `hello`，但是默认参数是会被覆盖的，即可以被 docker run命令提供的参数覆盖，但是主命令依然会被保留：例如我执行 `docker run myimage Hi` 输出： `Hi`；但是如果docker run 提供的命令是可执行命令，则整个CMD都会被替换掉；
2. 单独只有 `ENTRYPOINT ["echo", "Hello"]`,执行 `docker run myimage "Goodbye"`,输出 `Hello Goodbye`，背后原理是：`ENTRYPOINT`后的指令会作为 **固定主命令**，我们在docker run命令中提供的参数会被附加到 ENTRYPOINT后面
3. 同时有 ENTRYPOINT 和 CMD：`ENTRYPOINT ["echo"] CMD ["Hello"]`，则`CMD`命令会作为 `ENTRYPOIINT`的参数，并且整个CMD都可以被docker run的参数替换掉；

另外`docker-compose`文件中也能够制定CMD、ENDPOINT，在docker-compose文件中设定的一切对于dockerfile文件中的设置都是 **覆盖**效果；

## Docker-compose

```yml
version: '2'
services:
  redis-cluster:
    environment:
     IP: ${REDIS_CLUSTER_IP}
     SENTINEL: ${REDIS_USE_SENTINEL}
     STANDALONE: ${REDIS_USE_STANDALONE}
    build:
      context: .
      args:
        redis_version: '7.2.5'
    hostname: server
    ports:
      - '7000-7050:7000-7050'
      - '5000-5010:5000-5010'
```

我们可以看到，`dockerfile`与`dockercompose`文件都能够添加环境变量，一个是创建镜像是加入的，一个是创建容器时添加的，最终的效果都是添加到执行环境中

## 容器交互

对于需要相互交互的容器之间，我们尽量将其放置在一个`docker-compose`文件当中，这样他们就能够直接互相访问对方的服务；docker-compose还为我们提供了更加友好的方式；在 `Docker Compose` 中，为每个服务起的名字可以直接在其他服务的内部使用，原因是 Docker Compose 默认会为每个服务创建一个 DNS 解析名称，并将其添加到服务所在的网络中。

即：我们使用exec进入其中任意一个容器，就可以直接使用 `curl http://容器名`访问其他服务了！！！，这是非常方便的；docker-compose只负责帮助我们讲容器名翻译成ip；



但是对于不在同一个docker-compose文件中定义的服务，但是在同一主机的docker上，其要是想要互相访问应该怎么办呢？

docker也提供的简便的方案，我们可以直接使用**主机IP:端口**的方式进行服务访问（`切记ufw要放开相应端口的权限`）

