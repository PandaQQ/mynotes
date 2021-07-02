# Mobile MircoService API Guide

> This topic is mainly focus on the Rancher 1.6 Installation for CentOS 7.6 with SpringCloud Alibaba MircoService Usage.

## 1. Docker Installation

>  One-Click Installation

```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

> And then you will found docker is not start, don't worry

```
service docker start
```

[Cannot connect to the Docker daemon at unix:/var/run/docker.sock. Is the docker daemon running?](https://stackoverflow.com/questions/44678725/cannot-connect-to-the-docker-daemon-at-unix-var-run-docker-sock-is-the-docker)

## 2. Rancher Installation

```
docker run -d --restart=unless-stopped -p 8080:8080 rancher/server:stable
```

> After Installed rancher server and then you need to login with your http://<Public IP>:8080

Add the host machine:

```
sudo docker run --rm --privileged -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/rancher:/var/lib/rancher rancher/agent:v1.2.11 http://<Public IP>:8080/v1/scripts/4A40B5567EA72C994C52:1609372800000:R3pg8rkAjRFutbmfE54DpJMs
```

## 3. Pull the docker Images

> Nacos Server  Docker Image

```
docker pull nacos/nacos-server

nacos port is 8848
```

> MySQL 5.7 Docker Image

```
docker pull nacos/nacos-mysql:5.7

docker exec -it affd8a505472 /bin/bash
```

> Redis 6.x Docker Image

```
docker pull redis

docker exec -it f0540dae02bf /bin/bash
```

> Sentinel Docker Image

```
docker pull bladex/sentinel-dashboard:1.8.0
sentinel port is 32100
```

> saber Docker Image

```
docker pull --platform linux/x86_64 mysql:5.7.26
docker build -t saber-db:3.0.3 .

docker run -p 3300:3306 --name saber-db \
--restart always \
-e MYSQL_ALLOW_EMPTY_PASSWORD=1 \
-e MYSQL_ROOT_PASSWORD=root \
-d saber-db:3.0.3

docker exec -it 74683d472176 /bin/bash
```

> Skywalking Docker Images

```
docker pull elasticsearch:7.5.1
docker pull apache/skywalking-oap-server:6.6.0-es7
docker pull apache/skywalking-ui:6.6.0
```

> Skywalking Deployment with ES7 as data storage

```
https://www.cnblogs.com/xiao987334176/p/13530575.html

docker run -d --name=es7 \
  -p 9200:9200 \
  -p 9300:9300 \
  -e "discovery.type=single-node" \
  -v /root/elasticsearch/data:/usr/share/elasticsearch/data \
  -v /root/elasticsearch/logs:/usr/share/elasticsearch/logs \
elasticsearch:7.5.1

docker cp es7:/usr/share/elasticsearch/data /root/elasticsearch/
docker cp es7:/usr/share/elasticsearch/logs /root/elasticsearch/

docker run --name oap --restart always -d \
--restart=always \
-e TZ=Asia/Shanghai \
-p 12800:12800 \
-p 11800:11800 \
--link es7:es7 \
-e SW_STORAGE=elasticsearch \
-e SW_STORAGE_ES_CLUSTER_NODES=es7:9200 \
apache/skywalking-oap-server:8.5.0-es7

docker run -d --name skywalking-ui \
--restart=always \
-e TZ=Asia/Shanghai \
-p 8088:8080 \
--link oap:oap \
-e SW_OAP_ADDRESS=oap:12800 \
apache/skywalking-ui:6.6.0
```

> Skywalking-8.5.0 with H2 as data storage

```
docker pull apache/skywalking-oap-server:8.5.0-es7
docker pull apache/skywalking-ui:8.5.0

docker run --name skywalking-oap --restart always -d -p 11800:11800 -p 12800:12800  -e SW_STORAGE=h2 -e TZ=Asia/Shanghai apache/skywalking-oap-server

docker run --name skywalking-ui --restart always -d -p 8080:8080 --link skywalking-oap:skywalking-oap -e SW_OAP_ADDRESS=skywalking-oap:12800 apache/skywalking-ui
```

### 4. All Ports needs Open

```
SSH	TCP	22	SSH	修改|删除
自定义	TCP	8080	Racher	修改|删除
自定义	TCP	8848	nacos	修改|删除
自定义	TCP	8000	skywalking	修改|删除
自定义	TCP	32100	sentinel	修改|删除
自定义	TCP	11800	sky-oap	修改|删除
自定义	TCP	9001	provider	修改|删除
自定义	TCP	5000	gateway	修改|删除
自定义	TCP	9002	consumer
```

### 5. Sentinel issues

```
Seems need to let Sentinel can acccess your springboot application with given port, for exmaple if your springboot port is 9001, so open to sentinel should be 19001
```

