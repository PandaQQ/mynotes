# 使用 nginx + docker搭建静态资源服务器
> 记录下使用nigix + docker 搭建服务器的步骤， 方便日后部署查阅资

### Step 1  - 拉取 nginx 镜像
```
docker pull nginx
```

### Step 2 - 准备工作
```
# 创建一个存放静态资源的目录，比如我是使用了 /home/app/images( 建议不要使用 root 权限，不好使用 xftp 上传文件 )
mkdir /home/app/images
# 创建并编辑 nginx.conf，我的 nginx.conf 放在 /home/app/ 路径底下
vi nginx.conf
```

### Step - 3 nginx.conf
```
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   /images;
            index  index.html index.htm;
        }
    }

    include /etc/nginx/conf.d/*.conf;
}
```

### Step - 4 执行 docker 实例创建命令
```
# --name 实例名
# -p 是端口映射，80是 nginx 容器的默认端口，映射到主机的 4545 端口，也就是访问宿主机的 4545  端口，就会自动跳转到 nginx 容器的 80 端口，也就是 nginx 的入口
# -v 是挂载目录，它会让容器和宿主机共享目录，我们把静态资源放在 /home/app/images，容器内的路径 /images 也会更新静态资源，记得改成自己的真实路径
# 由于容器内没有 vim 和 vi ，甚至没有 yum ，所以我们无法在容器内修改，那么可以选择 将外部的 nginx 配置文件覆盖容器内的配置文件
docker run --name images_nginx -p 4545:80 -v /home/app/images:/images -v /home/app/nginx.conf:/etc/nginx/nginx.conf -d nginx
```

### Step - 5 最后通过 url 可以访问你的静态资源
```
http://ip:4545/image.png
```