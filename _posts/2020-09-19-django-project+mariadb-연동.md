---
layout: post
title:  "django project docker image에 올리기(2)"
slug: django project + mariadb
date:   2020-09-19 12:15:50 +0900
description: 도커 실습(1)
author: yejin
seo.type: BlogPosting
categories: Docker

---

# django project + mariadb 연동해서 docker image에 올리기(1)

## maria db 연동하기 및 장고 프로젝트 기본 세팅



### maria db 설치

1. mariadb 이미지 파일 받기

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

   ```
   vagrant@swarm-worker1:~$ docker image ls
   REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
   mariadb             latest              cbbff8572fa8        36 hours ago        406MB
   ```

   

2. mariadb 컨테이너 생성

   ```
   vagrant@swarm-worker1:~/datadir$ pwd
   /home/vagrant/datadir
   vagrant@swarm-worker1:~/datadir$ cd ~
   vagrant@swarm-worker1:~$ docker run --name mariadb \
   > -v /home/vagrant/datadir:/home/mariadb/DB \
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

   

3. 컨테이너 접속해서 mariadb 생성되었는지 확인하기

   (database 이름 : project_db / user : root  / pw : project)

   ```
   # mariadb 컨테이너에 접속하기
   vagrant@swarm-worker1:~$ docker exec -it mariadb /bin/bash
   
   root@276b09691217:/# echo $MYSQL_ROOT_PASSWORD
   project
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
   
   MariaDB [(none)]> select host,user from user;
   +-----------+-------------+
   | Host      | User        |
   +-----------+-------------+
   | %         | root        |
   | localhost | mariadb.sys |
   | localhost | root        |
   +-----------+-------------+
   3 rows in set (0.001 sec)
   
   # project db에 project 사용자가 외부에서 접속 가능하게 설정
   MariaDB [(none)]> grant all privileges on project.* to 'project'@'%';
   Query OK, 0 rows affected (0.001 sec)
   
   MariaDB [(none)]> flush privileges;
   Query OK, 0 rows affected (0.001 sec)
   
   MariaDB [(none)]> select host,user,password from mysql.user;
   +------------+-------------+-------------------------------------------+
   | Host       | User        | Password                                  |
   +------------+-------------+-------------------------------------------+
   | localhost  | mariadb.sys |                                           |
   | localhost  | root        | *FD0B2F9649853705D5A8A1D84AEA4B57B9590B23 |
   | %          | root        | *FD0B2F9649853705D5A8A1D84AEA4B57B9590B23 |
   | %          | project     | *FD0B2F9649853705D5A8A1D84AEA4B57B9590B23 |
   +------------+-------------+-------------------------------------------+
   4 rows in set (0.001 sec)
   
   MariaDB [(none)]> show grants for 'project'@'%';
   
   ```



### python 설치

1. python 이미지 불러오기

   ```
   vagrant@swarm-worker1:~$ docker search python
   NAME             DESCRIPTION         STARS               OFFICIAL            AUTOMATED
   python    Python is an interpreted, interactive, objec…   5491                [OK]
   django    Django is a free web application framework, …   994                 [OK]
   ```

   ```
   vagrant@swarm-worker1:~$ docker pull python
   Using default tag: latest
   latest: Pulling from library/python
   57df1a1f1ad8: Pull complete
   71e126169501: Pull complete
   1af28a55c3f3: Pull complete
   03f1c9932170: Pull complete
   65b3db15f518: Pull complete
   3e3b8947ed83: Pull complete
   a4850b8bdbb7: Pull complete
   416533994968: Pull complete
   1b580f9ce4ce: Pull complete
   Digest: sha256:e9b7e3b4e9569808066c5901b8a9ad315a9f14ae8d3949ece22ae339fff2cad0
   Status: Downloaded newer image for python:latest
   docker.io/library/python:latest
   ```

   ```
   vagrant@swarm-worker1:~$ docker image ls python
   REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
   python              latest              28a4c88cdbbf        7 days ago          882MB
   ```

   

2. python container 만들기

   ```
   vagrant@swarm-worker1:~$ docker run -t -d --name python \
   --link mariadb \
   -v /home/vagrant/django_playlist/:/home/django_playlist \
   -p 8000:8000 -p 3306:3306 \
   python:latest
   98773843bb3468eefe48a7a4166ea425312502c40cd7e31026f477a5295a1e21
   
   vagrant@swarm-worker1:~$ docker container ls
   CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                            NAMES
   98773843bb34        python:latest       "python3"                3 seconds ago       Up 3 seconds        0.0.0.0:3306->3306/tcp, 0.0.0.0:8000->8000/tcp   python
   276b09691217        mariadb:latest      "docker-entrypoint.s…"   30 minutes ago      Up 30 minutes       3306/tcp                                         mariadb
   ```

3. python 컨테이너에서 파이썬,볼륨,mariadb 연결 확인 및 third party 설치

   ```
   vagrant@swarm-worker1:~$ docker exec -it python /bin/bash
   root@98773843bb34:/# python --version
   Python 3.8.5
   root@98773843bb34:/# ls /home/
   django_playlist
   ```

   - 장고 설치 및 확인

     ```
     root@98773843bb34:/# pip install django
     Collecting django
       Downloading Django-3.1.1-py3-none-any.whl (7.8 MB)
          |████████████████████████████████| 7.8 MB 1.6 MB/s
     ```

     ```
     root@98773843bb34:/# pip show django
     Name: Django
     Version: 3.1.1
     Summary: A high-level Python Web framework that encourages rapid development and clean, pragmatic design.
     Home-page: https://www.djangoproject.com/
     Author: Django Software Foundation
     Author-email: foundation@djangoproject.com
     License: BSD-3-Clause
     Location: /usr/local/lib/python3.8/site-packages
     Requires: sqlparse, asgiref, pytz
     Required-by:
     ```

   - apt-get update 및 upgrade하기

     ```
     root@98773843bb34:/# apt-get update
     root@98773843bb34:/# apt-get upgrade
     ```

   - mysqlclient 설치 및 버전 확인

     ```
     root@98773843bb34:/# pip install mysqlclient
     Collecting mysqlclient
       Downloading mysqlclient-2.0.1.tar.gz (87 kB)
          |████████████████████████████████| 87 kB 525 kB/s
     Building wheels for collected packages: mysqlclient
       Building wheel for mysqlclient (setup.py) ... done
       Created wheel for mysqlclient: filename=mysqlclient-2.0.1-cp38-cp38-linux_x86_64.whl size=115814 sha256=977ca6dcc0715d0fd95305858b213305af95a979e798a2aa0e7d2682a25126fd
       Stored in directory: /root/.cache/pip/wheels/b9/b0/63/fb1bf1bfddaf244627c94fa2eb59e706d1170f4ca4f8b72d59
     Successfully built mysqlclient
     Installing collected packages: mysqlclient
     Successfully installed mysqlclient-2.0.1
     ```

     ```
     root@98773843bb34:/# pip show mysqlclient
     Name: mysqlclient
     Version: 2.0.1
     Summary: Python interface to MySQL
     Home-page: https://github.com/PyMySQL/mysqlclient-python
     Author: Inada Naoki
     Author-email: songofacandy@gmail.com
     License: GPL
     Location: /usr/local/lib/python3.8/site-packages
     Requires:
     Required-by:
     ```

   - mariadb 연결되어있는지 확인

     ```
     root@98773843bb34:~# ping mariadbb
     ping: mariadbb: Name or service not known
     root@98773843bb34:~# ping mariadb
     PING mariadb (172.17.0.2) 56(84) bytes of data.
     64 bytes from mariadb (172.17.0.2): icmp_seq=1 ttl=64 time=0.124 ms
     64 bytes from mariadb (172.17.0.2): icmp_seq=2 ttl=64 time=0.101 ms
     64 bytes from mariadb (172.17.0.2): icmp_seq=3 ttl=64 time=0.106 ms
     ^C
     --- mariadb ping statistics ---
     3 packets transmitted, 3 received, 0% packet loss, time 45ms
     rtt min/avg/max/mdev = 0.101/0.110/0.124/0.013 ms
     ```

     

4. 장고 프로젝트 파일  컨테이너 안에 옮기기 (왼쪽 하단의 upload 버튼 누르기)

   ![뒤에서5](https://user-images.githubusercontent.com/69428620/93616330-920cfe80-fa0f-11ea-974c-33e5cfdc541d.png)

   ```
   root@98773843bb34:~# cd /home/django_playlist/
   root@98773843bb34:/home/django_playlist# ls
   README.md  django_src  manage.py  plist  requirements.txt  venv
   ```

   > **vi 명령어가 안 먹힘... 고로 설치해야 함**
   >
   > ```
   > root@98773843bb34:~# apt-get install vim
   > ```

5. settings.py 파일에서 mariadb로 연결 변경하기

   ```
   root@98773843bb34:/home/django_playlist# vi django_src/settings.py
   ```

   ![뒤에서3](https://user-images.githubusercontent.com/69428620/93616324-90dbd180-fa0f-11ea-9e08-63ef594fbe3d.png)

   > mariadb ip 확인(172.17.0.2) 하고 settings.py 에 해당 ip입력하기

   

6. requirements.txt에 설치목록 설치하기

   ```
   root@98773843bb34:/home/django_playlist# pip install -r requirements.txt
   ```

7. migrate 하기

   ```
   root@98773843bb34:/home/django_playlist# python manage.py migrate
   /usr/local/lib/python3.8/site-packages/pydub/utils.py:170: RuntimeWarning: Couldn't find ffmpeg or avconv - defaulting to ffmpeg, but may not work
     warn("Couldn't find ffmpeg or avconv - defaulting to ffmpeg, but may not work", RuntimeWarning)
   Operations to perform:
     Apply all migrations: admin, auth, contenttypes, plist, sessions
   Running migrations:
     Applying contenttypes.0001_initial... OK
     Applying auth.0001_initial... OK
     Applying admin.0001_initial... OK
     Applying admin.0002_logentry_remove_auto_add... OK
     :
     
   root@98773843bb34:/home/django_playlist# python manage.py showmigrations plist
   /usr/local/lib/python3.8/site-packages/pydub/utils.py:170: RuntimeWarning: Couldn't find ffmpeg or avconv - defaulting to ffmpeg, but may not work
     warn("Couldn't find ffmpeg or avconv - defaulting to ffmpeg, but may not work", RuntimeWarning)
   plist
    [X] 0001_initial
    [X] 0002_auto_20200914_2100
    [X] 0003_auto_20200914_2129
   ```

8. 서버 띄우기

   ```
   root@98773843bb34:/home/django_playlist# python manage.py runserver 0.0.0.0:8000
   Watching for file changes with StatReloader
   Performing system checks...
   
   /usr/local/lib/python3.8/site-packages/pydub/utils.py:170: RuntimeWarning: Couldn't find ffmpeg or avconv - defaulting to ffmpeg, but may not work
     warn("Couldn't find ffmpeg or avconv - defaulting to ffmpeg, but may not work", RuntimeWarning)
   System check identified no issues (0 silenced).
   September 18, 2020 - 23:48:04
   Django version 3.1, using settings 'django_src.settings'
   Starting development server at http://0.0.0.0:8000/
   Quit the server with CONTROL-C.
   ```

   ![뒤에서2](https://user-images.githubusercontent.com/69428620/93616318-8f120e00-fa0f-11ea-89b3-b366503aafa4.png)

   > **포트 포워딩을 해줘야 HOST PC에서 접속 가능함**

   

9. 서버 뜬 화면 확인!

   ![뒤에서1](https://user-images.githubusercontent.com/69428620/93616268-7b66a780-fa0f-11ea-8067-492be804fff6.png)







### 참고 사이트

- db 사용자 계정 생성/권한

  [https://velog.io/@max9106/MySQL-%EC%9C%A0%EC%A0%80-%EC%83%9D%EC%84%B1-sgk5cplhit]: https://velog.io/@max9106/MySQL-%EC%9C%A0%EC%A0%80-%EC%83%9D%EC%84%B1-sgk5cplhit
  [https://www.codingfactory.net/11336]: https://www.codingfactory.net/11336
  [https://serverfault.com/questions/793058/can-not-access-mysql-docker]: https://serverfault.com/questions/793058/can-not-access-mysql-docker

- 장고 docker에 올리기

  [https://devyurim.github.io/development%20environment/docker/2018/08/09/docker-2.html]: https://devyurim.github.io/development%20environment/docker/2018/08/09/docker-2.html
  [https://www.daleseo.com/docker-compose-django/]: https://www.daleseo.com/docker-compose-django/

  







### 해결해야 할일..

1. elastic-search 컨테이너도 만들어서 묶어야함..
2. ffmpeg 없어서 노래 슬라이스가 안됨 ==> 리눅스 서버에 설치필요

3. mariadb 한글가능하게 설정
