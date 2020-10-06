---
title:  "django project docker image에 올리기(5)"
date:   2020-09-23 13:15:50 +0900
categories: 
  - test
tags: [Docker, Image Build, 실습]
toc: true
toc_sticky: true
toc_label: "Docker Image 테스트"
---

# DockerHub에서 받은 Docker Image로 테스트

<br>

## #1. 이미지 확인

```
vagrant@swarm-worker1:~$ docker image pull ok2qw66/plist:1.1
vagrant@swarm-worker1:~$ docker image pull ok2qw66/plist_mariadb:1.1
```

```
vagrant@swarm-worker1:~$ docker image ls
REPOSITORY              TAG            IMAGE ID            CREATED             SIZE
ok2qw66/plist           1.1            23e9de109441        37 minutes ago      1.49GB
ok2qw66/plist_mariadb   1.1            7c024eac9296        2 days ago          492MB
```

## #2. plist_mariadb 컨테이너 접속 및 테스트

```
vagrant@swarm-worker2:~$ docker run -d --name mariadb \ 
-v /home/vagrant/mariadb_vol:/var/lib/mysql \
ok2qw66/plist_mariadb:1.1
cb1355647dec0049ed6f0cd2a18e504ceba284d8d7a0b260dfd57d204620e236
```

### (1) plist_mariadb 컨테이너 접속

```
vagrant@swarm-worker2:~$ docker exec -it mariadb /bin/bash
root@cb1355647dec:/# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
22: eth0@if23: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

==> ip 도 동일하게 들어간다

### (2) mariadb 접속하기

```
root@cb1355647dec:/# mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 4
Server version: 10.5.5-MariaDB-1:10.5.5+maria~focal mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| project_db         |
+--------------------+
4 rows in set (0.001 sec)

MariaDB [(none)]> select user,host from mysql.user;
+-------------+-----------+
| User        | Host      |
+-------------+-----------+
| root        | %         |
| mariadb.sys | localhost |
| root        | localhost |
+-------------+-----------+
3 rows in set (0.001 sec)
```

> project 사용자가 없다...  다시 생성해보자

<br>

## #3. plist 컨테이너 접속 및 테스트

### (1) plist 컨테이너 실행

```
vagrant@swarm-worker2:~$ docker run -t -d --name plist \
> --link mariadb \
> -v /home/vagrant/django_playlist:/home/django_playlist \
> -p 3306:3306 -p 8000:8000 \
> ok2qw66/plist:1.1
8a4332fd6b6fdc0ea108c589a2ceb439c4fd2f4e62187a52caa826a568dbee5a
```

### (2) plist 컨테이너 실행 확인

```
vagrant@swarm-worker2:~$ docker container ls
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                NAMES
2c41b25e6d64        ok2qw66/plist:1.1           "python3"                5 seconds ago       Up 5 seconds        3306/tcp, 8000/tcp   plist
83c3fec11b8c        ok2qw66/plist_mariadb:1.1   "docker-entrypoint.s…"   6 minutes ago       Up 6 minutes        3306/tcp             mariadb                                        mariadb
```

### (3) 컨테이너 접속하기 및 설치내용 확인

```
vagrant@swarm-worker2:~$ docker exec -it plist /bin/bash

root@ac54a83c913b:/# python --version
Python 3.8.5

root@ac54a83c913b:/# ffmpeg -version
ffmpeg version 4.1.6-1~deb10u1 Copyright (c) 2000-2020 the FFmpeg developers
built with gcc 8 (Debian 8.3.0-6)
		:
		
root@ac54a83c913b:/# pip show django
Name: Django
Version: 3.1
Summary: A high-level Python Web framework that encourages rapid development and clean, pragmatic design.
Home-page: https://www.djangoproject.com/
Author: Django Software Foundation
Author-email: foundation@djangoproject.com
License: BSD-3-Clause
Location: /usr/local/lib/python3.8/site-packages
Requires: sqlparse, pytz, asgiref
Required-by: djangorestframework, django-cors-headers, django-bootstrap4
```

<br>

### (4) 프로젝트 실행하기

```
root@ac54a83c913b~# cd /home/django_playlist
root@ac54a83c913b:/home/django_playlist# python manage.py migrate
root@ac54a83c913b:/home/django_playlist# python manage.py runserver 0.0.0.0:8000
```

![image-20200921025903403](https://user-images.githubusercontent.com/69428620/93718633-76912780-fbb8-11ea-9958-7998d6b2dbf9.png)

<br>



## 해결해야 할일..

1. elastsic search 컨테이너 생성해서 연동하기
2. ffmpeg 노래제목에 소괄호 () 들어가면 에러 뜸 ==> 예외처리해주기
3. 간혹 서버로그에서 한글들어간 파일/이미지파일  304에러가 뜸
4. /home/mariadb 폴더에 의미있는 파일이 들어가게 하고 싶다
5. 이미지 파일 만들 때 볼륨 내용도 같이 복사하기 가능한지 찾아보기
6. 이미지 파일 만들 때 -p, --link 등의 옵션이 포함될수 있는지 찾아보기
7. ok2qw66/plist_mariadb:1.1 과  ok2qw66/plist:1.1 파일을 연동하여 컨테이너 실행할 수 있도록 docker-compose.yml 파일 만들기
