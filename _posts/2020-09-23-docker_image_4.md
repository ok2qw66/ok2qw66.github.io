---
title:  "django project docker image에 올리기(4)"
date:   2020-09-23 02:00:50 +0900
categories: 
  - Docker
---

# django project + mariadb 연동해서 docker image에 올리기(4)




> django project + mariadb 연동해서 docker image에 올리기(3)에서 django_playlist 소스가 포함되지 않은
>
> image 파일을 생성했다....

 

### django project source 포함한 image 파일 만들기



1. 지난시간에 만들어놓은 plist 이미지로 컨테이너 생성

   ```
   vagrant@swarm-worker2:~$ docker container run -td --name plist ok2qw66/plist:1.1
   c7dd17cca946e2926aede910f2092839b378823fa2288ee135050734073b54df
   ```

   

2.  컨테이너 내부로 진입

   ```
   vagrant@swarm-worker2:~$ docker container exec -it plist /bin/bash
   root@c7dd17cca946:~#
   ```

   

3. git clone 할 디렉터리로 이동하기

   ```
   root@c7dd17cca946:~# cd /home/django_playlist
   ```

    

4. github 사용하여 djanog project source 가져오기

   ```
   root@c7dd17cca946:/home/django_playlist# git clone https://github.com/ok2qw66/django_playlist.git .
   :
   :
   root@c7dd17cca946:/home/django_playlist# exit
   ```

   

5. ok2qw66/plist:1.1 image 파일로 덮어쓰기

   ```
   vagrant@swarm-worker2:~$ docker image ls
   REPOSITORY              TAG            IMAGE ID            CREATED             SIZE
   ok2qw66/plist           1.1            23e9de109441        7 seconds ago       1.49GB
   ok2qw66/plist           <none>         9eb3b2aaa300        47 hours ago        1.24GB
   ok2qw66/plist_mariadb   1.1            7c024eac9296        47 hours ago        492MB
   ```

   **===> TAG <none> 발생함!!!  ==> 제거해주자**

   

   - none tag 가지고 있는 이미지 파일 제거하기 (mariadb 와 연동되어 있어서 같이 삭제해야 함)

     ```
     vagrant@swarm-worker2:~$ docker image rm -f 9eb3b2aaa300 7c024eac9296
     ```

     

6. docker hub 에 image 올리기

   ```
   vagrant@swarm-worker2:~$ docker push ok2qw66/plist:1.1
   ```





### 다른 서버에서 이미지파일로 컨테이너 생성 후 project source 있는지 확인



1. ok2qw66/plist:1.1 이미지 파일 다운로드

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

   

2.  해당 이미지 파일로 컨테이너 생성

   ```
   vagrant@swarm-worker2:~$ docker run -d --name mariadb ok2qw66/plist_mariadb:1.1
   ```

   ```
   vagrant@swarm-worker2:~$ docker run -t -d --name plist \
   > --link mariadb \
   > -v /home/vagrant/django_playlist:/home/django_playlist \
   > -p 3306:3306 -p 8000:8000 \
   > ok2qw66/plist:1.1
   ```

   ```
   vagrant@swarm-worker1:~$ docker container ls -a
   CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                                            NAMES
   c7057f1a3991        ok2qw66/plist:1.1           "python3"                5 seconds ago       Up 3 seconds        0.0.0.0:3306->3306/tcp, 0.0.0.0:8000->8000/tcp   plist
   efc757d7512b        ok2qw66/plist_mariadb:1.1   "docker-entrypoint.s…"   38 seconds ago      Up 37 seconds       3306/tcp                                         mariadb
   ```

   

3. 컨테이너 내부로 접속하여 project source 있는지 확인

   ```
   vagrant@swarm-worker1:~$ docker exec -it plist /bin/bash
   root@c7057f1a3991:/# cd /home/django_playlist/
   root@c7057f1a3991:/home/django_playlist# ls
   README.md  django_src  manage.py  music.json  plist  requirements.txt
   ```

   ```
   root@c7057f1a3991:/home/django_playlist# python manage.py migratee
   root@c7057f1a3991:/home/django_playlist# python manage.py runserver 0.0.0.0:8000
   ```

   





> 이전에 만든 ok2qw66/plist_mariadb:1.1 과 
>
> 위에서 만든 ok2qw66/plist:1.1 파일을 연동하여 컨테이너 실행할 수 있도록
>
> docker-compose.yml 파일을 만들어보자 
>
> 이어서 계속...
