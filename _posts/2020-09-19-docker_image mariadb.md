---
title:  "django project docker image에 올리기(1)"
date:   2020-09-19 12:15:50 +0900
categories: 
  - test
tags: [Docker, Image Build, 실습]
toc: true
toc_sticky: true
toc_label: "MariaDB 세팅"
---

# 프로젝트 설명 및 mariaDB 세팅

<br>

##  프로젝트 설명

github 주소 : [https://github.com/multicampus-cloud/django_playlist](https://github.com/multicampus-cloud/django_playlist)

- 프로젝트 설명

  YouTube에서 국내외에 관계없이 수 많은 노래들을 들을 수 있다.<br>

  그 중에는 간혹 공식 음원 사이트에 없는 국내 미발매 곡들도 있다.  <br>또한 Apple Music에는 있지만 Melon에는 없는, 혹은 그 반대 경우인 곡들도 있어

  두 사이트를 함께 사용하기에 불편한 점이 있다.<br>

  이 점에서 착안하여 YouTube URL을 통해 음원을 서버에 다운받아 노래를 재생시킬 수 있는 플레이리스트 프로젝트를 수행하게 되었다.

- Docker Image를 만드는 데에 필요한 프로젝트 주요 환경
  - Python 3.7 이상
  - Django Framework
  - MariaDB
  - Elastic-search
  - ffmeg

<br>

**다음 실습은 Docker 환경에서 위에 설명한 Django Project를 Docker Image로 만드는 과정입니다!<br> 실습 전에 Docker 설치 및 DockerHub 계정이 있어야 합니다!**

<br>

## MariaDB 설치

<br>

### #1. MariaDB 이미지 파일 받기

- MariaDB 이미지 받기

```
vagrant@swarm-worker1:~$ docker pull mariadb:latest
latest: Pulling from library/mariadb
e6ca3592b144: Pull complete
534a5505201d: Pull complete
990916bd23bb: Pull complete
c62d6bd206a2: Pull complete
ba34deb445c3: Pull complete
47b4f6570cf0: Pull complete
28b039c5139e: Pull complete
d9f0e67eb23f: Pull complete
69df87d9330c: Pull complete
0222c6c4a452: Pull complete
6ab34d00af2f: Pull complete
bb48a5d6fdc6: Pull complete
Digest: sha256:3c18e067d60fc9fa2b669d0820176840248d85ce51ff9ebb0f3869f61939194c
Status: Downloaded newer image for mariadb:latest
docker.io/library/mariadb:latest
```

- MariaDB 이미지 확인

```
vagrant@swarm-worker1:~$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mariadb             latest              cbbff8572fa8        36 hours ago        406MB
```

<br>

### #2. mariadb 컨테이너 생성 (미리 /home/vagrant/mariadb 폴더 생성x)

```
vagrant@swarm-worker1:~$ docker run --name mariadb \
> -v /home/vagrant/mariadb:/var/lib/mysql \
> -e MYSQL_ROOT_PASSWORD='project' \
> -e MYSQL_DATABASE=project_db \
> -d mariadb:latest
276b09691217d06f357fe45b46821ac426f32d1ea781eb33b8937c9ae0bbfa90
```

```
vagrant@swarm-worker1:~$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
276b09691217        mariadb:latest      "docker-entrypoint.s…"   4 seconds ago       Up 3 seconds        3306/tcp            mariadb
```

<br>

### #3. 컨테이너 접속해서 mariadb 생성되었는지 확인하기

```
vagrant@swarm-worker1:~$ docker exec -it mariadb /bin/bash

root@276b09691217:/# echo $MYSQL_ROOT_PASSWORD
project
```

<br>

### #4. root 계정으로 mairadb 접속해서 project 계정만들기

(database 이름 : project_db / user : root  / pw : project)

```
root@276b09691217:/# mysql -u root -p
Enter password: project
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 6
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
4 rows in set (0.000 sec)

MariaDB [(none)]> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [mysql]> select host,user from user;
+-----------+-------------+
| Host      | User        |
+-----------+-------------+
| %         | root        |
| localhost | mariadb.sys |
| localhost | root        |
+-----------+-------------+
3 rows in set (0.001 sec)

#project 계정 생성
MariaDB [(none)]> create user 'project'@'%' identified by 'project';
Query OK, 0 rows affected (0.001 sec)
```
- project db에 대한 모든 권한을 project 계정에게 주기

```
MariaDB [(none)]> grant all privileges on project_db.* to 'project'@'%';
   Query OK, 0 rows affected (0.001 sec)

   MariaDB [(none)]> flush privileges;
   Query OK, 0 rows affected (0.001 sec)

   MariaDB [(none)]> select host,user from mysql.user;
   +-----------+-------------+
   | Host      | User        |
   +-----------+-------------+
   | %         | project     |
   | %         | root        |
   | localhost | mariadb.sys |
   | localhost | root        |
   +-----------+-------------+
   4 rows in set (0.001 sec)

   MariaDB [(none)]> show grants for 'project'@'%';
   +---------------------------------------------------------------------------------+
   | Grants for project@%                                                            |
   +---------------------------------------------------------------------------------+
   | GRANT USAGE ON *.* TO `project`@`%` IDENTIFIED BY PASSWORD '*FD0B2F9649853705D5A8A1D84AEA4B57B9590B23' |
   | GRANT ALL PRIVILEGES ON `project_db`.* TO `project`@`%`                         |
   +---------------------------------------------------------------------------------+
   2 rows in set (0.000 sec)
```

<br>

### #5. project 계정으로 mariadb 접속하기

```
root@09b89dd518c0:~# mysql -u project -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 5
Server version: 10.5.5-MariaDB-1:10.5.5+maria~focal mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| project_db         |
+--------------------+
2 rows in set (0.000 sec)

MariaDB [(none)]> use project_db;
Database changed
MariaDB [project_db]> show tables;
Empty set (0.000 sec)
```

<br>

### #6. 한글 설정해주기

#### (1) apt-get update 및 upgrade 하기

```
root@98773843bb34:/# apt-get update
root@98773843bb34:/# apt-get upgrade
```

<br>

#### (2) vi 편집 가능하도록 vim 설치

```
root@98773843bb34:~# apt-get install -y vim
```

<br>

#### (3) MariaDB 한글로 설정파일 변경

```
root@276b09691217:~# cd /etc/mysql/
root@276b09691217:/etc/mysql# cp my.cnf my.cnf.ori
root@276b09691217:/etc/mysql# ls
conf.d        debian.cnf   mariadb.conf.d  my.cnf.ori
debian-start  mariadb.cnf  my.cnf
root@276b09691217:/etc/mysql# cat my.cnf
```

```
[client-server]
# Port or socket location where to connect
# port = 3306
socket = /run/mysqld/mysqld.sock

# Import all .cnf files from configuration directory
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mariadb.conf.d/

[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4

[mysqld]
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server = utf8
```

<br>

#### (4) MariaDB 컨테이너 재시작

```
vagrant@swarm-worker1:~$ docker restart mariadb
vagrant@swarm-worker1:~$ docker exec -it mariadb /bin/bash
```

<br>

#### (5) 한글 설정 확인하기

```
root@276b09691217:/etc/mysql# mysql -u root -p
Enter password: project
MariaDB [(none)]> show variables like 'c%';
+----------------------------------+----------------------------+
| Variable_name                    | Value                      |
+----------------------------------+----------------------------+
| character_set_client             | utf8mb4                    |
| character_set_connection         | utf8mb4                    |
| character_set_database           | utf8                       |
| character_set_filesystem         | binary                     |
| character_set_results            | utf8mb4                    |
| character_set_server             | utf8                       |
| character_set_system             | utf8                       |
| character_sets_dir               | /usr/share/mysql/charsets/ |
| check_constraint_checks          | ON                         |
| collation_connection             | utf8mb4_general_ci         |
| collation_database               | utf8_unicode_ci            |
| collation_server                 | utf8_unicode_ci            |
| column_compression_threshold     | 100                        |
| column_compression_zlib_level    | 6                          |
| column_compression_zlib_strategy | DEFAULT_STRATEGY           |
| column_compression_zlib_wrap     | OFF                        |
| completion_type                  | NO_CHAIN                   |
| concurrent_insert                | AUTO                       |
| connect_timeout                  | 10                         |
| core_file                        | OFF                        |
+----------------------------------+----------------------------+
20 rows in set (0.001 sec)
```





### 참고 사이트

- db 사용자 계정 생성/권한

  1) https://velog.io/@max9106/MySQL-%EC%9C%A0%EC%A0%80-%EC%83%9D%EC%84%B1-sgk5cplhit
  
  
  2) https://velog.io/@max9106/MySQL-%EC%9C%A0%EC%A0%80-%EC%83%9D%EC%84%B1-sgk5cplhit
  
  
  3) https://www.codingfactory.net/11336
  
  4) https://serverfault.com/questions/793058/can-not-access-mysql-docker

- mariadb 한글 설정해주기

  1) https://velog.io/@hongji3354/Docker%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%B4%EC%84%9C-MariaDB-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0