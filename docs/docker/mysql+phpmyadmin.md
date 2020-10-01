# 使用 Docker 安装 mysql5.6 + phpmyadmin


### 1.创建网络 net-mysql
```
sudo docker network create net-mysql
```
### 2. 创建容器 test-mysql 并加入网络 net-mysql

```
sudo docker run \ 
    --restart always \ 
    --name test-mysql \ 
    -e MYSQL_USER=root \ 
    -e MYSQL_PASSWORD=root \
    -e MYSQL_ROOT_PASSWORD=root 
    -p 3306:3306 \ 
    --network net-mysql \
    -d mysql:5.6
```

### 3. 创建容器 test-phpmyadmin 并加入网络 net-mysql
```
sudo docker run \ 
    --name test-phpmyadmin \
    -e MYSQL_USER=root \
    -e MYSQL_PASSWORD=root \
    -e MYSQL_ROOT_PASSWORD=root \ 
    -e PMA_HOST=test-mysql \ 
    -p 8080:80 --network net-mysql \ 
    -d phpmyadmin/phpmyadmin:latest
``` 