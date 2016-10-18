---
layout: post
title: Run docker under windows with or without shared folder
---

**Problematic:**
be able to launch docker from windows, edit my project files from a local folder, the files are updated on the docker instance.


**First attempt: **
 - share my local windows drive via Docker Settings dialog
 - mount my folder as a volume in my docker instance
 - result: the files are present on the docker instance but impossible to change the user/group of the files (stays root/root)
 - it seems that windows rights system interferes
 
** Second Attempt: **
 - let's use sshd to copy via sftp my files
 - but it's a very bad idea, as when I rebuild my docker instance, all the files will be lost and everything 
 will be needed to be copied again

** Third Attempt: **
 - use the priviledged mode of the docker compose file
 ```yaml
 version: '2'

services:
    php:
        privileged: true
        build: php7-fpm
        expose:
            - 9000
        links:
            - db:mysqldb
            - redis
        volumes:
            - ${MY_APP_PATH}:/var/www/app
...
```
the keypoint here is the priviledge option. With this you can launch a bash with privileged rights (true root user) that can do things like mount, apt-get, ...

Here the docker file associated to the service php
```bash
FROM php:7.0-fpm

RUN apt-get update && apt-get install -y git unzip wget cifs-utils vim \
    && docker-php-ext-install pdo_mysql opcache 


# opcache
RUN { \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=60'; \
		echo 'opcache.fast_shutdown=1'; \
		echo 'opcache.enable_cli=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini

RUN echo "realpath_cache_size = 4096k; realpath_cache_ttl = 7200;" > /usr/local/etc/php/conf.d/php.ini

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer && \
    composer --version

# Set timezone
RUN rm /etc/localtime && \
    ln -s /usr/share/zoneinfo/Europe/Paris /etc/localtime && \
    "date"

RUN echo 'alias sf="php app/console"' >> ~/.bashrc
RUN echo 'alias sf3="php bin/console"' >> ~/.bashrc
RUN echo 'alias ll="ls -al"' >> ~/.bashrc

#Share a directory between windows and docker
RUN mkdir -p /var/www/app
RUN { \
		echo "username=${SHARED_FOLDER_USER}"; \
		echo "password=${SHARED_FOLDER_PWD}"; \
		echo "domain=${SHARED_FOLDER_DOMAIN}"; \
	} > $HOME/.smbcredentials
RUN chmod 600 $HOME/.smbcredentials

#RUN mount -t cifs -o credentials=$HOME/.smbcredentials,iocharset=utf8,rw,uid=33,gid=33,serverino //192.168.0.25/anonymous-poll /var/www/app 

WORKDIR /var/www/app
```
the keypoints here are the installation of the package cifs-utils that allow the mount of samba folder
You need to share your folder in windows (in my case the directory anonymous-poll) and in the file $HOME/.smbcredentials, you set the username and password needed to connect to this samba folder

then you just have to launch these commands:
```bash
docker-compose down
docker-compose build
docker-compose up -d
docker exec --privileged -it dockersymfonymaster_php_1 /bin/bash
```

From the bash
```bash
mount -t cifs -o credentials=$HOME/.smbcredentials,iocharset=utf8,rw,uid=33,gid=33,serverino //192.168.0.25/anonymous-poll /var/www/app
```
it's OK you have your windows folder mounted in your docker instance with user/group 33 (www-data)


**Next step : be able to mount the directory during the "up" of the docker-compose**

**Problematic2:**

nodejs modules are problematic under windows as there is a lot of sub directories, sometimes you can have a problem with 
path that is limited to 260 characters
