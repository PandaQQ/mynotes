# PHP7.4-Apache with Docker

> This document is mainly focus on the docker configuration for php7.4-apache usage;

```
docker exec -it <container_id> /bin/bash
cd /usr/local/bin
./docker-php-ext-install pdo_mysql  
./docker-php-ext-install mysql
docker <container_id> restart
```



#### 1. Install mysql related packages

```
cd /usr/local/bin
./docker-php-ext-install pdo_mysql  
./docker-php-ext-install mysqli

./docker-php-ext-install calendar
./docker-php-ext-install exif

## for gd lib witj install directly is not work 
## ./docker-php-ext-install gd

apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd

./docker-php-ext-install gettext
./docker-php-ext-install sysvshm
./docker-php-ext-install sysvmsg
./docker-php-ext-install sysvsem
```



#### 2. Failed Installed Packages

```
./docker-php-ext-install intl
./docker-php-ext-install ldap
./docker-php-ext-install mcrypt
./docker-php-ext-install oci8
./docker-php-ext-install zip
./docker-php-ext-install xsl
./docker-php-ext-install xmlrpc
./docker-php-ext-install wddx
./docker-php-ext-install shmop
```



#### 3. Zend OPcache

```
docker-php-ext-configure opcache --enable-opcache
docker-php-ext-install opcache
```

#### 4. intl

```
apt-get install libicu-dev
docker-php-ext-configure intl && docker-php-ext-install intl
```





