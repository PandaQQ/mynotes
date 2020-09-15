# 使用 GitLab 来对项目进行 CI/CD
> 主要介绍使用 Docker CE 安装 Gitlab 并进行 CI/CD

### 1. 搭建 GitLab CE
```
sudo docker run --detach \
    --publish 443:443 \
    --publish 80:80 \
    --publish 2222:22 \
    --name gitlab \
    --restart always \
    --volume /data/gitlab/config:/etc/gitlab \
    --volume /data/gitlab/logs:/var/log/gitlab \
    --volume /data/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest
```
### 2. 搭建 GitLab Runner
```
sudo docker run --rm -t -d -i -p 8084:8080 \
    -v /data/gitlab-runner:/etc/gitlab-runner \
    -v /var/run/docker.sock:/var/run/docker.sock \
    --add-host ad4aac43c567:172.17.0.2 \
    --name gitlab-runner \
    gitlab/gitlab-runner
```
- 1. 此处需注意--add-host请自行替换为GitLab CE Docker 容器 ID，此处是为了让 GitLab CE Docker 容器可以被 GitLab Runner 访问到, 如果使用公网 IP 可以忽略。
- 2. 如果通过/etc/hosts仍然无法解决HOST问题, 请自行更改/data/gitlab-runner/config.toml文件，在[runners.docker]节点下面添加extra_hosts = ["ad4aac43c567:172.17.0.2"]。

### 3.注册 Runner
```
sudo docker exec -it gitlab-runner  gitlab-runner register -n \
      --url http://192.168.1.13/ \
      --registration-token pfHxurfRMBctWwkqrt1c \
      --tag-list=docker-privileged \
      --description "dockersock" \
      --docker-privileged=false \
      --docker-image "docker:latest" \
      --docker-volumes /var/run/docker.sock:/var/run/docker.sock \
      --docker-volumes /root/m2:/root/.m2 \
      --executor docker
```

- url: GitLab CE 里面 CI 栏目查看
- registration-token: GitLab CE 里面 CI 栏目查看
- tag-list: 标签, 后续用于执行步骤时指定 Runner
- description: 描述
- docker-image: 外层使用的 Docker 镜像
- executor: 执行器

#### 4.编写 .gitlab-ci.yml

```
image: docker:stable

variables:
  DOCKER_DRIVER: overlay2

services:
  - docker:dind
#before_script:
#  - docker info

stages:
  - build
  - deploy

maven-build:  #   job名称， 可随意命名
  image: maven:3.5.3-jdk-8 # 使用maven容器运行，可以忽略
  stage: build  # 绑定这个job为`build`阶段
#  only:
#    - master # 仅在master分支变更时才触发执行
  tags:
    - docker-privileged # 使用标签名为 `maven`的`runner`执行
  script: "mvn package -B"  # 执行脚本，此处为打包操作
  artifacts:
    paths:
      - target/*.jar # 本阶段输出文件，此处输出打包后的fatjar

docker-deploy:
  stage: deploy
  tags:
    - docker-privileged # 使用标签名为`maven`的`runner`执行
  script:  # 这里的脚本逻辑是先把包含这个jar的文件编译成镜像，然后run it
    - docker build -t demo:1.0 .
    - app="test"
    - if docker ps | awk -v app="app" 'NR>1{  ($(NF) == app )  }'; then
    -  docker stop "$app" && docker rm -f "$app"
    - fi
    - docker run --name test -d -p 8081:8080 demo:1.0
```

#### 5.编写 .gitlab-ci.yml
> 使用docker compose 一件部署

```
version: '3'
services:
  gitlab:
    image: gitlab/gitlab-ce:latest
    container_name: gitlab
    restart: always
    volumes:
      - /data/gitlab/config:/etc/gitlab
      - /data/gitlab/logs:/var/log/gitlab
      - /data/gitlab/data:/var/opt/gitlab
    networks:
      - w-networks
    ports:
      - 443:443
      - 80:80
      - 2222:22

  gitlab-runner:
    image: gitlab/gitlab-runner
    container_name: gitlab-runner
    extra_hosts:
      - "ad4aac43c567:172.17.0.2" # Gitlab-CE
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /data/gitlab-runner/config:/etc/gitlab-runner
    networks:
      - w-networks
    ports:
      - 8080:8080

networks:
  w-networks:
    driver: bridge
```

#### 6.Reference

```
1.https://hocg.in/2018/06/20/%E4%BD%BF%E7%94%A8%20GitLab%20%E6%9D%A5%E5%AF%B9%20Spring%20Boot%20%E9%A1%B9%E7%9B%AE%E8%BF%9B%E8%A1%8C%20CI:CD/
```

#### 7. 部署过程中遇到的问题 (Gitlab Response 502)
> 这个问题是在一次运行gitlab VM 突然消失开始的，当我们的运维救起这部机的时候，gitlab就跑不起来了。后来经过努力～～～

```
# 修改 /data/gitlab/config/gitlab.rb
```
##! **recommend value is 1/4 of total RAM, up to 14GB.**
postgresql['shared_buffers'] = "12256MB"

### Advanced settings
postgresql['max_connections'] = 200
### Replication settings
postgresql['max_connections'] = 200
```