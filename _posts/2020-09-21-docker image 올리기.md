---
layout: post
title:  "django project docker image에 올리기(3)"
slug: django project + mariadb
date:   2020-09-21 02:56:50 +0900
description: 도커 실습(1)
author: yejin
seo.type: BlogPosting
categories: Docker

---



# django project + mariadb 연동해서 docker image에 올리기(3)

## python , mariadb 이미지 파일로 docker hub에 업로드하기



### 컨테이너를 이미지 파일로 생성

1. python 컨테이너 이미지 파일로 생성

   ok2qw66 도커 계정에 plist라는 이름으로 1.0 태그를 붙여서 이미지 파일 만들기

   ```
   vagrant@swarm-worker1:~$ docker container commit python ok2w66/plist:1.0
   ```

2.  mariadb 컨테이너 이미지 파일로 생성

   ```
   vagrant@swarm-worker1:~$ docker container commit mariadb ok2w66/plist_mariadb:1.0
   ```

3. 이미지 파일 확인

   ```
   vagrant@swarm-worker1:~$ docker image ls
   REPOSITORY              TAG        IMAGE ID        CREATED           SIZE
   ok2qw66/plist_mariadb   1.0      5dbd83374ac1    2 hours ago         492MB
   ok2qw66/plist           1.0      223da4820311    2 hours ago         1.24GB
   mariadb                 latest   cbbff8572fa8    3 days ago          406MB
   python                  latest   28a4c88cdbbf    9 days ago          882MB
   ```

   



### docker hub에 이미지 파일 올리기

1. ok2qw66/plist:1.0 을 docker hub에 올리기

   ```
   vagrant@swarm-worker1:~$ docker image push ok2w66/plist:1.0
   ```

2. ok2qw66/plist_mariadb:1.0 을 docker hub에 올리기

   ```
   vagrant@swarm-worker1:~$ docker image push ok2w66/plist_mariadb:1.0
   ```

3.  도커허브 페이지에 접속해서 이미지 업로드 확인

   ![image-20200921000219789](https://user-images.githubusercontent.com/69428620/93718652-8f99d880-fbb8-11ea-9a11-1f02b09ed435.png)





### 다른 서버에서 이미지 파일 다운받아서 프로젝트 실행해보기

> 1.0 버전에 미포함된 부분이 있어서 1.1 이미지파일 받아서 테스트 진행하겠습니다

1. plist 이미지 가져오기

   ```
   vagrant@swarm-worker2:~$ docker image pull ok2qw66/plist:1.1
   ```

2. plist_mariadb 이미지 가져오기

   ```
   vagrant@swarm-worker2:~$ docker image pull ok2qw66/plist_mariadb:1.1
   ```

3. image 확인

   ```
   vagrant@swarm-worker2:~$ docker image ls
   REPOSITORY              TAG         IMAGE ID            CREATED             SIZE
   ok2qw66/plist_mariadb   1.1         5dbd83374ac1        2 hours ago         492MB
   ok2qw66/plist           1.1         223da4820311        2 hours ago         1.24GB
   ```



### 다른 서버에서 plist_mariadb 실행하기

1. plist_mariadb 컨테이너 실행

   ```
   vagrant@swarm-worker2:~$ docker run -d --name mariadb ok2qw66/plist_mariadb:1.1
   cb1355647dec0049ed6f0cd2a18e504ceba284d8d7a0b260dfd57d204620e236
   ```

2. 컨테이너 접속하기

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

3. mariadb 접속하기

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

   > project 사용자가 없다... 
   >
   > https://ok2qw66.github.io/posts/docker-django-mariadb-python 참고해서 다시생성해보자



### 다른 서버에서 plist 실행하기

1. plist 컨테이너 실행

   ```
   vagrant@swarm-worker2:~$ docker run -t -d --name plist \
   > --link mariadb \
   > -v /home/vagrant/django_playlist:/home/django_playlist \
   > -p 3306:3306 -p 8000:8000 \
   > ok2qw66/plist:1.1
   8a4332fd6b6fdc0ea108c589a2ceb439c4fd2f4e62187a52caa826a568dbee5a
   ```

2.  plist 컨테이너 실행 확인

   ```
   vagrant@swarm-worker2:~$ docker container ls
   CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                NAMES
   2c41b25e6d64        ok2qw66/plist:1.1           "python3"                5 seconds ago       Up 5 seconds        3306/tcp, 8000/tcp   plist
   83c3fec11b8c        ok2qw66/plist_mariadb:1.1   "docker-entrypoint.s…"   6 minutes ago       Up 6 minutes        3306/tcp             mariadb                                        mariadb
   ```

3. 컨테이너 접속하기 및 설치내용 확인

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



### 이미 실행중인 plist컨테이너에 볼륨 추가하기

1. 프로젝트 위치로 이동

   ```
   root@ac54a83c913b:/home/django_playlist# ls -al
   total 8
   drwxr-xr-x 2 root root 4096 Sep 18 13:18 .
   drwxr-xr-x 1 root root 4096 Sep 18 13:18 ..
   ```

   ===> volume 내용은 같이 복사가 안된다 ==> 따로 추가

2.  plist 컨테이너 멈추기

   ```
   vagrant@swarm-worker2:~$ docker stop plist
   plist
   ```

3.  sftp로 /home/vagrant/django_playlist 에 프로젝트 폴더내용 넣기

   ![image-20200921025132230](https://user-images.githubusercontent.com/69428620/93718647-8577da00-fbb8-11ea-84d3-172eb7885c88.png)

4.  plist 컨테이너 멈추기

   ```
   vagrant@swarm-worker2:~$ docker container start plist
   ```

5. 프로젝트 실행하기

   ```
   root@ac54a83c913b~# cd /home/django_playlist
   root@ac54a83c913b:/home/django_playlist# python manage.py migrate
   root@ac54a83c913b:/home/django_playlist# python manage.py runserver 0.0.0.0:8000
   ```
   
   ![image-20200921025903403](https://user-images.githubusercontent.com/69428620/93718633-76912780-fbb8-11ea-9958-7998d6b2dbf9.png)















### 참고 사이트

- 이미 실행중인 plist컨테이너에 볼륨 추가하기

  1) https://medium.com/sjk5766/%EC%8B%A4%ED%96%89%EC%A4%91%EC%9D%B8-container%EC%97%90-port-or-volume-%EC%B6%94%EA%B0%80-ae8889344c68






### 해결해야 할일..

1. elastsic search 컨테이너 생성해서 연동하기
2. ffmpeg 노래제목에 소괄호 () 들어가면 에러 뜸 ==> 예외처리해주기
3. 간혹 서버로그에서 한글들어간 파일/이미지파일  304에러가 뜸
4. /home/mariadb 폴더에 의미있는 파일이 들어가게 하고 싶다
5. 이미지 파일 만들 때 볼륨 내용도 같이 복사하기 가능한지 찾아보기
6. 이미지 파일 만들 때 -p, --link 등의 옵션이 포함될수 있는지 찾아보기
