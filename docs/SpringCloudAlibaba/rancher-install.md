# Mobile MircoService API Guide

> This topic is mainly focus on the Rancher 1.6 Installation for CentOS 7.6 with SpringCloud Alibaba MircoService Usage.

### Version Management





## 1. Docker Installation

>  十分推荐，一键安装

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

