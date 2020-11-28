# 使用 Docker 安装 mysql5.7.31 + phpmyadmin
> 本文主要介绍如何使用docker 安装mysql和phpmyadmin

### 1.创建 conf 目录
```
mkdir -p ~/mysql/conf
cd ~/mysql/conf
```

### 2. 配置 my.conf

```
vim my.cnf
```

> my.cnf 配置如下:
```
[mysqld]
log-bin=/var/lib/mysql/mysql-bin
expire_logs_days=10

server-id=1

datadir=/var/lib/mysql

[mysql_safe]
log-error=/var/log/mysql/mysql.log

!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
```

### 3. 使用docker安装 mysql

```
sudo docker run -p 3306:3306 --name test-mysql \
--restart always \
-v ~/mysql/conf:/etc/mysql/conf.d \
-v ~/mysql/logs:/var/log \
-v ~/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=<!--your password--> \
-d mysql:5.7.31
```

### 3. 创建容器 phpmyadmin 并绑定 mysql容器
```
sudo docker run --name phpmyadmin -d \
--restart always \
--link test-mysql:db \
-p 80:80 \
phpmyadmin/phpmyadmin
``` 