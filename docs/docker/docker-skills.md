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