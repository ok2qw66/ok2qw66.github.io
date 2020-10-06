---
title:  "django project docker image에 올리기(2)"
date:   2020-09-20 12:15:50 +0900
categories: 
  - test
tags: [Docker, Image Build, 실습]
toc: true
toc_sticky: true
toc_label: "Python 세팅"
---



# Python + Django 설치

<br>

# python 설치

<br>

## #1. python 이미지 불러오기

- 이미지 검색

```
vagrant@swarm-worker1:~$ docker search python
NAME             DESCRIPTION         STARS               OFFICIAL            AUTOMATED
python    Python is an interpreted, interactive, objec…   5491                [OK]
django    Django is a free web application framework, …   994                 [OK]
```

- 이미지 받기

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

- 이미지 확인

```
vagrant@swarm-worker1:~$ docker image ls python
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
python              latest              28a4c88cdbbf        7 days ago          882MB
```

<br>

## #2. python container 만들기

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

- 3306 포트는 MariaDB 연동하기위한 포트

<br>

## #3. Python 컨테이너에서 MariaDB와 연결확인

```
vagrant@swarm-worker1:~$ docker exec -it python /bin/bash
root@98773843bb34:/# python --version
Python 3.8.5
root@98773843bb34:/# ls /home/
django_playlist
```

### (1) 장고 설치 및 확인

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

### (2) apt-get update 및 upgrade하기

```
root@98773843bb34:/# apt-get update
root@98773843bb34:/# apt-get upgrade
```

### (3) mysqlclient 설치 및 버전 확인

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

### (4) Python 컨테이너와 MariaDB 연결 확인

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

<br>

## #4. 프로젝트 폴더 추가 및 third party 설치

### (1) git clone으로 프로젝트 가져오기

```
root@c7dd17cca946:/home/django_playlist# git clone https://github.com/ok2qw66/django_playlist.git .
:
:
```

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

### (2) settings.py 파일에서 MariaDB로 연결 변경하기

```
root@98773843bb34:/home/django_playlist# vi django_src/settings.py
```

![뒤에서3](https://user-images.githubusercontent.com/69428620/93616324-90dbd180-fa0f-11ea-9e08-63ef594fbe3d.png)

> mariadb ip 확인(172.17.0.2) 하고 settings.py 에 해당 ip입력하기

<br>

### (3) requirements.txt에 설치목록 설치하기

```
root@98773843bb34:/home/django_playlist# pip install -r requirements.txt
```

### (4) migrate 하기

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

### (5) 서버 띄우기

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

<br>

### (6) 서버 뜬 화면 확인!

![뒤에서1](https://user-images.githubusercontent.com/69428620/93616268-7b66a780-fa0f-11ea-8067-492be804fff6.png)

<br>
