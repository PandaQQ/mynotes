## GitLab, Harbora and GitLab-Runner Installation Guide

> This document is mainly focus on the 
>
> 1. GitLab Installation with docker on 192.168.20.11
>2. GitLab-Runner & Jenkins on 192.168.20.12
> 3. Harbor on 192.168.20.13

### Opening Ports:

| Virtual Machine | Running Instance        | Opening Ports         | Need Reserve Ports |
| --------------- | ----------------------- | --------------------- | ------------------ |
| 192.168.20.11   | Gitlab CE - latest      | 22, 2222, 8443, 8080, | 10010 ~ 10020      |
| 192.168.20.12   | Gitlab-Runner & Jenkins | 22, 8080, 50000,4545  | 10010 ~ 10020      |
| 192.168.20.13   | Harbor                  | 22, 8443, 8080        | 10010 ~ 10020      |

### All Instance Accounts Information

| Appliaction   | URL                | User Name / PWD   |
| ------------- | ------------------ | ----------------- |
| GitLab CE     | 192.168.20.11:8080 | root / Abcd@1234  |
| Harbor        | 192.168.20.13:8080 | admin / Abcd@1234 |
| GitLab-Runner | NULL               | NULL              |
| Jenkins       | 192.168.20.12:8080 | admin / Abcd@1234 |

### 1. Install GitLab CE

```   
$ GITLAB_HOME = /home/docker/gitlab            # 建立gitlab本地目录
$ docker run -d \
--hostname gitlab.gtjai-devops.com\            # 指定容器域名,创建镜像仓库用
-p 8443:443 \                                  # 容器443端口映射到主机8443端口(https)
-p 8080:80 \                                   # 容器80端口映射到主机8080端口(http)
-p 2222:22 \                                   # 容器22端口映射到主机2222端口(ssh)
--name gitlab \                                # 容器名称
--restart always \                             # 容器退出后自动重启
-v /home/docker/gitlab/config:/etc/gitlab \    # 挂载本地目录到容器配置目录
-v /home/docker/gitlab/logs:/var/log/gitlab \  # 挂载本地目录到容器日志目录
-v /home/docker/gitlab/data:/var/opt/gitlab \  # 挂载本地目录到容器数据目录
gitlab/gitlab-ce:latest                        # 使用的镜像:版本
```

> How to upgrade GitLab

```
$ docker stop gitlab
$ docker rm gitlab

$ docker run -d \
--hostname 192.168.20.11:8080 \     # 指定容器域名,创建镜像仓库用
-p 8443:443 \                           # 容器443端口映射到主机8443端口(https)
-p 8080:80 \                            # 容器80端口映射到主机8080端口(http)
-p 2222:22 \                            # 容器22端口映射到主机2222端口(ssh)
--name gitlab \                         # 容器名称
--restart always \                      # 容器退出后自动重启
-v /home/docker/gitlab/config:/etc/gitlab \    # 挂载本地目录到容器配置目录
-v /home/docker/gitlab/logs:/var/log/gitlab \  # 挂载本地目录到容器日志目录
-v /home/docker/gitlab/data:/var/opt/gitlab \  # 挂载本地目录到容器数据目录
gitlab/gitlab-ce:latest                 # 使用的镜像:版本

```

### 2. Install Gitlab-Runner

#### 2.1 Add Offical Image

```
# For RHEL/CentOS/Fedora
`curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash`
```

#### 2.2 Install

```
# Install latest version
# For RHEL/CentOS/Fedora
export GITLAB_RUNNER_DISABLE_SKEL=true; sudo -E yum install gitlab-runner

# Install given Version
# for RPM based systems
yum list gitlab-runner --showduplicates | sort -r
export GITLAB_RUNNER_DISABLE_SKEL=true; sudo -E yum install gitlab-runner-10.0.0-1
```

> After Installed git-runner, you can check the the gitlab-runner version

```
gitlab-runner -v

Version:      13.12.0
Git revision: 7a6612da
Git branch:   13-12-stable
GO version:   go1.13.8
Built:        2021-05-20T15:16:05+0000
OS/Arch:      linux/amd64
```

#### 2.3 Register gitlab-runner

```
sudo gitlab-runner register \
  --non-interactive  \
  --url "$https://gitlab.com/" \
  --registration-token "$PROJECT_REGISTRATION_TOKEN" \
  --executor "shell" \
  --description "devops-runner" \
  --tag-list "build,deploy" \
  --run-untagged="true" \
  --locked="false" \
  --access-level="not_protected"
  
  And Plz note that 
  - URL is 192.168.20.11:8080
  - token can copy from gitlab
```

#### 2.4 Commands

```
gitlab-runner register # 默认交互模式，非交互式添加 --non-interactive
gitlab-runner list # 列出保存在配置文件中的所有运行程序
gitlab-runner verify # 检查runner是否可以连接，但不验证GitLab服务是否正在使用runner

-delete 删除
gitlab-runner unregister # 使Gitlab取消已注册的runner

# 使用令牌注销
gitlab-runner unregister --url $url --token $token

# 使用名称注销（同名注销第一个）
gitlab-runner unregister --name test-runner

# 注销所有
gitlab-runner unregister --all-runners
```

### 3. Harbour Installation

#### 3.1  docker-compose

```
# 下载最新版 `Docker Compose`
sudo curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# 对二进制文件应用可执行权限：
sudo chmod +x /usr/local/bin/docker-compose
# 测试是否安装成功
docker-compose --version
```

#### 3.2 Install Harbour

```
wget https://github.com/goharbor/harbor/releases/download/v2.2.2/harbor-offline-installer-v2.2.2.tgz

tar xf harbor-offline-installer-v2.2.2.tgz 
cd harbor
cp harbor.yml.tmpl  harbor.yml
```

> And then edit harbour.yml file

```
hostname: 192.168.20.13

# And comment https related config

cd harbor
./install.sh --with-trivy --with-chartmuseum
```

#### 3.3 docker login failed

```
If you VM is 192.168.20.12 and you want to push docker images to harbour (192.168.20.12)

cat /etc/docker/daemon.json
{
	"insecure-registries":["192.168.20.13:8080"]
}

# reload json config and then reload docker
sudo systemctl daemon-reload
sudo service docker restart
# how to check insecure-registries is valid
docker info
```

#### 3.4 Push images Testing

```
docker login 192.168.20.13:8080
docker pull nginx
docker tag nginx 192.168.20.13:8080/my-test/nginx:latest
docker push 192.168.20.13:8080/my-test/nginx
```

### 4. Jenkins Install

```
docker pull jenkins/jenkins
mkdir -p /var/jenkins_mount
chmod 777 /var/jenkins_mount

docker run -d \ 
-p 8080:8080 \
-p 50000:50000 \
-v /var/jenkins_mount:/var/jenkins_home \ 
-v /etc/localtime:/etc/localtime \
--name myjenkins jenkins/jenkins
```

### 5. Reference:

- https://yxnchen.github.io/technique/Docker%E9%83%A8%E7%BD%B2GitLab%E5%B9%B6%E5%AE%9E%E7%8E%B0%E5%9F%BA%E6%9C%AC%E9%85%8D%E7%BD%AE/#%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C
- https://juejin.cn/post/6894156602224902157
- https://blog.csdn.net/u013948858/article/details/79974796
- https://cloud.tencent.com/developer/article/1647790
- https://hezhiqiang-book.gitbook.io/docker/harbor

