# RabbitMQ  with CI4



#### Dockerfile-php8.1

```
FROM php:8.1-apache

# Set timezone to Shanghai
ENV TZ Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone


# 1.0.1 Add: pdo_mysql, mysqli, calendar, exif gettext sockets mysqli pdo_mysql shmop sysvmsg sysvsem sysvshm
RUN docker-php-ext-install -j$(nproc)  calendar exif gettext \
sockets mysqli pdo_mysql shmop sysvmsg sysvsem sysvshm

# 1.0.2 Add: gd
RUN apt-get update && \
apt-get install -y --no-install-recommends libfreetype6-dev libjpeg62-turbo-dev libpng-dev && \
rm -r /var/lib/apt/lists/* && \
docker-php-ext-configure gd --with-freetype --with-jpeg && \
docker-php-ext-install -j$(nproc) gd

# 1.0.3 Add: zip
RUN apt-get update && \
apt-get install -y --no-install-recommends libzip-dev && \
rm -r /var/lib/apt/lists/* && \
docker-php-ext-install -j$(nproc) zip

# 1.0.4 Add: Zend OPcache
RUN docker-php-ext-configure opcache --enable-opcache && docker-php-ext-install opcache

# 1.0.5 Add: intl
RUN apt-get update && \
apt-get install -y --no-install-recommends libicu-dev && \
rm -r /var/lib/apt/lists/* && \
docker-php-ext-configure intl &&  \
docker-php-ext-install -j$(nproc) intl

ADD conf/apache.conf /etc/apache2/sites-available/000-default.conf

# 1.0.7 Add: ldap
RUN set -x \
    && apt-get update \
    && apt-get install -y libldap2-dev \
    && rm -rf /var/lib/apt/lists/* \
    && docker-php-ext-configure ldap --with-libdir=lib/$(gcc --dumpmachine)/ \
    && docker-php-ext-install ldap \
    && apt-get purge -y --auto-remove libldap2-dev

# 1.0.8 Add: soap xsl
RUN apt-get update && \
apt-get install -y --no-install-recommends libxml2-dev libtidy-dev libxslt1-dev && \
rm -r /var/lib/apt/lists/* && \
docker-php-ext-install -j$(nproc) soap xsl


# 1.0.9 Add: CodeIngiter rewrite engine setting
RUN a2enmod rewrite

# 1.0.10 Add: xdebug
RUN pecl install xdebug \
    && docker-php-ext-enable xdebug \
    && echo "xdebug.mode=debug" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.client_host = host.docker.internal" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# 1.0.10 Add: Composer
RUN php -r "copy('https://install.phpcomposer.com/installer', 'composer-setup.php');" 
RUN php composer-setup.php
RUN php -r "unlink('composer-setup.php');"
RUN mv composer.phar /usr/local/bin/composer

# 1.0.11 Add Git / ZIP / UNZIP
RUN apt update && \
    apt -qy install git unzip zlib1g-dev libzip-dev
RUN docker-php-ext-install sockets pcntl zip

# 1.0.12 Clean apt-get
RUN apt-get clean \
    && rm -r /var/lib/apt/lists/*

#ADD startScript.sh /startScript.sh
# The printf command below creates the script /startScript.sh with the following 3 lines. 
# #!/bin/bash
# mv /codeigniter4 /var/www/html
# /usr/sbin/apache2ctl -D FOREGROUND
RUN printf "#!/bin/bash\nmv /codeigniter4 /var/www/html\n/usr/sbin/apache2ctl -D FOREGROUND" > /startScript.sh
RUN chmod +x /startScript.sh

RUN cd /var/www/html

RUN composer create-project codeigniter4/appstarter codeigniter4 v4.1.5
RUN chmod -R 0777 /var/www/html/codeigniter4/writable

RUN mv codeigniter4 /

RUN apt-get clean \
    && rm -r /var/lib/apt/lists/*
    
EXPOSE 80
VOLUME ["/var/www/html", "/var/log/apache2", "/etc/apache2"]

CMD ["bash", "/startScript.sh"]


# MIT
LABEL Author="Qi Zheng"
LABEL Version="1.0"
LABEL Description="PHP 8.1 + Apache hsu ecampus development environment - with ARM Version"
```



#### Build & Run CI4 Docker with PHP8.1 + Apache

```
# Build Docker Image:
docker build -f Dockerfile-php8.1 -t pandaqq/codeigniter:4.1.5 .

# Run Docker:
docker run --name my-ci-dev -d \
-p 81:80 \
-v ~/Documents/rabbitmq-dev/www:/var/www/html \
codeigniter:4.1.5

# Push Docker Image:
docker push pandaqq/codeigniter:4.1.5
```





#### Reference

- https://github.com/php-amqplib/php-amqplib/blob/master/docker/php/Dockerfile (php-amqp Dockerfile)
- https://github.com/atsanna/codeigniter4-docker   (CodeIgniter4+PHP8.0 Dockerfile) 
- https://github.com/pierrecdn/phpipam/issues/61 (Can't build image for ARM #61)
- https://www.w3cschool.cn/codeigniter4/codeigniter4-x96a39js.html (CI4 教程)