---
title:  "django project docker image에 올리기(4)"
date:   2020-09-20 13:15:50 +0900
categories: 
  - test
tags: [Docker, Image Build, 실습]
toc: true
toc_sticky: true
toc_label: "DockerHub에 Docker Image 올리기"
---



# Python , MariaDB 이미지 파일을 Docker Hub에 업로드하기

<br>

# 컨테이너를 이미지 파일로 생성

## #1. python 컨테이너 이미지 파일로 생성

ok2qw66 도커 계정에 plist라는 이름으로 1.0 태그를 붙여서 이미지 파일 만들기

```
vagrant@swarm-worker1:~$ docker container commit python ok2w66/plist:1.0
```

## #2. mariadb 컨테이너 이미지 파일로 생성

```
vagrant@swarm-worker1:~$ docker container commit mariadb ok2w66/plist_mariadb:1.0
```

## #3. 이미지 파일 확인

```
vagrant@swarm-worker1:~$ docker image ls
REPOSITORY              TAG        IMAGE ID        CREATED           SIZE
ok2qw66/plist_mariadb   1.0      5dbd83374ac1    2 hours ago         492MB
ok2qw66/plist           1.0      223da4820311    2 hours ago         1.24GB
mariadb                 latest   cbbff8572fa8    3 days ago          406MB
python                  latest   28a4c88cdbbf    9 days ago          882MB
```

<br>

# docker hub에 이미지 파일 올리기

## #1. ok2qw66/plist:1.0 을 docker hub에 올리기

```
vagrant@swarm-worker1:~$ docker image push ok2w66/plist:1.0
```

## #2. ok2qw66/plist_mariadb:1.0 을 docker hub에 올리기

```
vagrant@swarm-worker1:~$ docker image push ok2w66/plist_mariadb:1.0
```

## #3. 도커허브 페이지에 접속해서 이미지 업로드 확인

![image-20200921000219789](https://user-images.githubusercontent.com/69428620/93718652-8f99d880-fbb8-11ea-9a11-1f02b09ed435.png)

<br>

# DockerHub 업로드 된 이미지 다운

> 1.0 버전에 미포함된 부분이 있어서 1.1 이미지파일 받아서 테스트 진행하겠습니다

<br>

## #1. plist 이미지 가져오기

```
vagrant@swarm-worker2:~$ docker image pull ok2qw66/plist:1.1
```

## #2. plist_mariadb 이미지 가져오기

```
vagrant@swarm-worker2:~$ docker image pull ok2qw66/plist_mariadb:1.1
```

## #3. image 확인

```
vagrant@swarm-worker2:~$ docker image ls
REPOSITORY              TAG         IMAGE ID            CREATED             SIZE
ok2qw66/plist_mariadb   1.1         5dbd83374ac1        2 hours ago         492MB
ok2qw66/plist           1.1         223da4820311        2 hours ago         1.24GB
```