# Docker 使用技巧
> 主要记录一些常用并且实用的小技巧

### 1  - Stop / remove all Docker containers
> One liner to stop / remove all of Docker containers:
```
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
```

### 2 - Monitor Docker logs with real-time
```
docker logs --follow {container_id}
```

### 3. Remove all docker images

```
docker rmi -f $(docker images -q)
```

### 4. Docker Usage:

> 查看运行中的容器

```
# 查看所有的容器用命令docker ps -a
docker ps
```

> 启动容器

```
# eg: docker start 9781cb2e64bd
docker start CONTAINERID[容器ID]
```

> stop容器

```
docker stop CONTAINERID[容器ID]
```

> 删除一个容器

```
docker rm CONTAINERID[容器ID]
```

> 查看Docker容器日志

```
# eg：docker logs 9781cb2e64bd
docker logs container‐name[容器名]/container‐id[容器ID]
```

> 复制文件

```
容器复制文件到物理机：docker cp 容器名称:容器目录 物理机目录
物理机复制文件到容器：docker cp 物理机目录 容器名称:容器目录
```





```
docker exec -it <container_id_or_name>  /bin/bash
```

