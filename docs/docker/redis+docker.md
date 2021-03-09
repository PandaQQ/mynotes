# 使用 Docker 安装 Redis
> 本文主要介绍如何使用docker 安装Redis

### 1. 新建data和conf两个文件夹，位置随意。
```
mkdir -p ~/redis/data
mkdir -p ~/redis/conf
```

### 2. 增加配置文件 redis.conf
> 在刚才新建的redis/conf中新建文件redis.conf，内容如下：
```
#bind 127.0.0.1 //允许远程连接
protected-mode no
appendonly yes //持久化
requirepass 123456 //密码 
```

### 3. 创建redis容器并启动
> 释义如下：
> –name：给容器起一个名
> -p：端口映射 宿主机:容器
> -v：挂载自定义配置 自定义配置:容器内部配置
> -d：后台运行
> redis-server --appendonly yes： 在容器执行redis-server启动命令，并打开redis持久化配置\

```
docker run --name my_redis -p 6379:6379 \
-v ~/redis/data:/data \
-v ~/redis/conf/redis.conf:/etc/redis/redis.conf \
-d redis redis-server /etc/redis/redis.conf

```

### 4. 启动成功，查看状态
> a. 通过docker ps查看启动状态，是否成功

> b. 执行docker exec -it my_redis redis-cli 命令，进入终端。 通过auth password进行登陆。 完成命令如下：

```
docker exec -it my_redis redis-cli
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> set name hello
OK
127.0.0.1:6379> get name
"hello"
```

> c. 使用Redis Desktop Manager客户端进行连接


### 5. 小结
- 当启动容器端口报错时，可以通过netstat -lntp | grep 6379查看哪个程序在占用
- 可以通过sudo kill 6379杀掉占用端口的程序
- 如果使用阿里云等，请务必把相应端口打开
