---
title:  "수업정리(6)- Docker Swarm/Ingress Network/Discovery/ Swarm Volume"
date:   2020-10-03 02:00:50 +0900
categories: 
  - 수업정리
tags:
  - Docker
  - Swarm
  - Network
---

# docker swarm

![image-20200916160916888](https://user-images.githubusercontent.com/69428620/94989583-e89e3f00-05b0-11eb-8cfe-bb23dedbc997.png)


## swarm 환경 구성 후 할일

1.  C:\HashiCorp에서 cmd 창 3개 띄움 (swarm mananger/worker1/worker2)
2.  vagrant up (가상머신 기동)
3.  vagrant ssh (가상머신 접속)
4.  sudo su (루트 권한으로 실행)
5.  systemctl start docker.service   service docker restart (도커 서비스 실행)
6.  sudo ufw disable    systemctl stop firewalld (방화벽 중지 : 통신안됨)





> 만약 worker 노드가 down 상태라면??
>
> manager 노드에서 active 상태로 변경해주자!!
>
> ``root@swarm-manager:~# docker node update --availability active swarm-worker2``



##### **vagrant 명령어 참고**

| 설명                             | 명령어        |
| -------------------------------- | ------------- |
| vagrant up                       | 가상머신 가동 |
| vagrant ssh                      | 가상머신 접속 |
| vagrant halt                     | 가상머신 중지 |
| vagrant destroy                  | 가상머신 삭제 |
| vagrant snapshot save 스냅샷이름 | 스냅샷 찍기   |
| vagrant snapshot list            | 스냅샷 목록   |



## docker swarm 모드의 구조

- 매니저(manager) 노드와 워커(worker) 노드로 구성
- 워커 노드 ⇒ 실제 컨테이너가 생성되고 관리되는 도커 서버
- 매니저 노드 ⇒ 워커 노드를 관리하기 위한 도커 서버
- 매니저 노드는 워커 노드의 역할을 포함
- 클러스터를 구성하기 위해서는 최소 1개 이상의 매니저 노드가 존재해야 함



## docker swarm 환경 구성

### #1. vagrant init 하기

```
C:\swarm > vagrant init
```

### #2. vagrantfile 수정

- ubuntu 버전

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.hostname = "swarm-manager"
  config.vm.network "private_network", ip: "192.168.111.100"
  config.vm.synced_folder ".", "/vagrant_data", disabled: true
  config.vm.provision "shell", inline: $script
end

$script = <<SCRIPT
  apt update
  apt upgrade
  apt install -y docker.io
  usermod -a -G docker $USER
  service docker restart
  chmod 666 /var/run/docker.sock
SCRIPT
```

- centos 버전

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "generic/centos7"
  config.vm.hostname = "swarm-manager"
  config.vm.network "private_network", ip: "192.168.111.100"
  config.vm.synced_folder ".", "/vagrant_data", disabled: true
  config.vm.provision "shell", inline: $script
end

$script = <<SCRIPT
  yum install -y docker
  systemctl start docker.service
  systemctl stop firewalld
SCRIPT
```



### #3. vagrant 시작 / 접속 / 중지

```
C:\swarm> vagrant up
C:\swarm> vagrant ssh
C:\swarm> vagrant halt
```

### #4. manager / worker1 /worker2 만들기

#### 방법1. vagrantfile 각각 만들어서 실행

#2번 따라서 각각 vagrantfile로 노드 만들기

- 달라지는부분 : **config.vm.hostname 와 config.vm.network ip부분 ** 

```
config.vm.hostname = "swarm-manager/swarm-worker1/swarm-worker2"

config.vm.network "private_network", ip: "192.168.111.100/101/102"
```



#### 방법2. virtualbox 복제

이름: swarm-manager, swarm-worker1, swarm-worker2 (각각)

경로: c:\swarm

![image-20201001194542570](https://user-images.githubusercontent.com/69428620/94989597-09669480-05b1-11eb-96b4-d0dc3015a367.png)


##### 1) 각각 ip/hostname 설정하기

```
#3 가상머신(도커 스웜의 노드로 동작)을 실행 
#3-1 swarm_manager 시작

#3-2 swarm_worker1 시작
$ sudo vi /etc/netplan/50-vagrant.yml
IP를 192.168.111.101 으로 변경 후 저장
$ sudo netplan apply 
$ sudo hostnamectl set-hostname swarm-worker1
$ reboot

#3-3 swarm_worker2 시작
$ sudo vi /etc/netplan/50-vagrant.yml
IP를 192.168.111.102 으로 변경 후 저장
$ sudo netplan apply 
$ sudo hostnamectl set-hostname swarm-worker2
$ reboot
```

##### 2) bitvise로 각각 가상머신 접속

![image-20201001194738859](https://user-images.githubusercontent.com/69428620/94989611-18e5dd80-05b1-11eb-9e04-2d1c253203f6.png)


### #5. 매니저 역할의 서버에서 스웜 클러스터를 시작

```
[root@swarm-manager ~]# docker swarm init --advertise-addr 192.168.111.100
Swarm initialized: current node (ym6n02vuulxa9izewh25vdv57) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-5w9hi0j9ahwgnq78w29pbqvm8sdte7osgl261lf6hte4fatgxp-f105esf8p1dvu5cfvkwsb9ukj 192.168.111.100:2377
    ⇒ 새로운 워커 노드를 클러스터에 추가할 때 사용하는 비밀키(토큰)

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

### #6. 워커 노드를 추가 : #5 내용 붙여넣기

```
[root@swarm-worker1~]# docker swarm join --token SWMTKN-1-5w9hi0j9ahwgnq78w29pbqvm8sdte7osgl261lf6hte4fatgxp-f105esf8p1dvu5cfvkwsb9ukj 192.168.111.100:2377This node joined a swarm as a worker.
[root@swarm-worker2 ~]# docker swarm join --token SWMTKN-1-5w9hi0j9ahwgnq78w29pbqvm8sdte7osgl261lf6hte4fatgxp-f105esf8p1dvu5cfvkwsb9ukj 192.168.111.100:2377This node joined a swarm as a worker.
```

### #7. 토큰 확인 및 변경

- manager로 참여 토큰 확인

```
[root@swarm-manager ~]# docker swarm join-token manager
To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-5w9hi0j9ahwgnq78w29pbqvm8sdte7osgl261lf6hte4fatgxp-5rn4mxiq7va86d04tjqidanjd 192.168.111.100:2377
```

- worker로 참여 토큰 확인

```
[root@swarm-manager ~]# docker swarm join-token worker
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-5w9hi0j9ahwgnq78w29pbqvm8sdte7osgl261lf6hte4fatgxp-f105esf8p1dvu5cfvkwsb9ukj 192.168.111.100:2377

```

- manager참여 토큰 변경

```
[root@swarm-manager ~]# docker swarm join-token --rotate manager
Successfully rotated manager join token.

To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-5w9hi0j9ahwgnq78w29pbqvm8sdte7osgl261lf6hte4fatgxp-bdgqy50v9bt3reswv7upf87ok 192.168.111.100:2377
```





## 노드 설정

### 노드 삭제

ex) worker2를 제거하고 싶다면?

- worker2

```
vagrant@swarm-worker2:~$ docker swarm leave
Node left the swarm.
```

- manager

```
vagrant@swarm-manager:~$ docker node rm swarm-worker2
swarm-worker2

vagrant@swarm-manager:~$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
wn5j8nymx704prelscdq8fw54 *   swarm-manager       Ready               Active              Leader              19.03.6
i2boml3wjv87wjv35mpi0okgt     swarm-worker1       Ready               Active                                  19.03.6
```



### 워커 노드를 매니저 노드로 변경

#### 1. swarm-manaer 노드에서 조인 토큰 확인

```
vagrant@swarm-manager:~$ docker swarm join-token worker
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-5w9hi0j9ahwgnq78w29pbqvm8sdte7osgl261lf6hte4fatgxp-f105esf8p1dvu5cfvkwsb9ukj 192.168.111.100:2377
```

#### 2.  swarm-worker2 노드를 클러스터에 워크 노드로 추가

```
vagrant@swarm-worker2:~$ docker swarm join --token SWMTKN-1-5w9hi0j9ahwgnq78w29pbqvm8sdte7osgl261lf6hte4fatgxp-f105esf8p1dvu5cfvkwsb9ukj 192.168.111.100:2377
This node joined a swarm as a worker.
```

#### 3. swarm-manager 노드에서 워크 노드 추가를 확인

```
vagrant@swarm-manager:~$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
wn5j8nymx704prelscdq8fw54 *   swarm-manager       Ready               Active              Leader              19.03.6
i2boml3wjv87wjv35mpi0okgt     swarm-worker1       Ready               Active                                  19.03.6
8cjqbgqzlf26kszvcyxoy3wgu     swarm-worker2       Ready               Active                                  19.03.6
```

#### 4. promote 명령으로 swarm-worker1 노드를 매니저 노드로 승격

```
vagrant@swarm-manager:~$ docker node promote swarm-worker1
Node swarm-worker1 promoted to a manager in the swarm.
vagrant@swarm-manager:~$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
wn5j8nymx704prelscdq8fw54 *   swarm-manager       Ready               Active              Leader              19.03.6
i2boml3wjv87wjv35mpi0okgt     swarm-worker1       Ready               Active              Reachable           19.03.6
8cjqbgqzlf26kszvcyxoy3wgu     swarm-worker2       Ready               Active                                  19.03.6
```



### demote 명령으로 매너저 노드를 워커 노드로 변경

```
vagrant@swarm-manager:~$ docker node demote swarm-worker1
Manager swarm-worker1 demoted in the swarm.

vagrant@swarm-manager:~$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
wn5j8nymx704prelscdq8fw54 *   swarm-manager       Ready               Active              Leader              19.03.6
i2boml3wjv87wjv35mpi0okgt     swarm-worker1       Ready               Active                                  19.03.6
8cjqbgqzlf26kszvcyxoy3wgu     swarm-worker2       Ready               Active                                  19.03.6
```



## swarm 노드 다루기

- Active 상태

  - 새로운 노드가 swarm에 추가되면 기본설정 상태
  - 서비스의 컨테이너를 할당받기 가능

  - ``docker node update --availability active NODE_NAME``

- Drain 상태

  - 노드에 문제가 생겨 일시적으로 사용하지 않는 상태

  - 서비스의 컨테이너를 할당받지 않음

  - Drain상태로 변경 -> 실행중 컨테이너 중지되고, 다른 노드로 할당됨

  - Drain에서 Active로 변경해도 분산할당 안됨

    -> ``docker service scale`` 명령어 통해 사용컨테이너 균형을 재조정!

  - ``docker node update --availability drain NODE_NAME``

- Pause 상태

  - 서비스의 컨테이너를 할당받지 않음

  - Pause 상태로 변경 

    -> 가지고 있던 실행중인 컨테이너를 계속 실행함 & 새로 받지 않음

  - ``docker node update --avaiability pause NODE_NAME``

    

### 노드 라벨 추가

`docker node update --label-add` 옵션을 통해 라벨 지정

```
root@swarm-manager:~# docker node update --label-add storage=ssd swarm-worker1
swarm-worker1
root@swarm-manager:~# docker node inspect --pretty swarm-worker1
ID:                     rre9gp2t4t1ae10dcrvdwxur4
Labels:
 - storage=ssd
 	:
 	:
```



### 서비스 제약 설정

``docker service create --constraint``옵션을 통해 컨테이너가 할당될 노드를 선택할 수 있다



#### #1 . node.labels 제약조건

storage 키의 갑이 ssd 로 설정된 노드에 컨테이너 할당

```
root@swarm-manager:~# docker service create --name label_ssd --constraint 'node.labels.storage == ssd' --replicas=5 ubuntu:14.04 ping docker.com
```

```
root@swarm-manager:~# docker service ps label_ssd
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
el7pdsqqr90o        label_ssd.1         ubuntu:14.04        swarm-worker1       Running             Running 17 seconds ago
ke7be2mt6vh1        label_ssd.2         ubuntu:14.04        swarm-worker1       Running             Running 17 seconds ago
lpx88jgo1crk        label_ssd.3         ubuntu:14.04        swarm-worker1       Running             Running 17 seconds ago
wjmbi9lkubap        label_ssd.4         ubuntu:14.04        swarm-worker1       Running             Running 17 seconds ago
0u089zwjk48i        label_ssd.5         ubuntu:14.04        swarm-worker1       Running             Running 17 seconds ago
```

- storage 라벨이 ssd로 설정된 swarm-worker1에만 컨테이너 생성됨
- 제약조건 만족하는 노드 없을 경우, **컨테이너 생성되지 않음**



#### #2 node.id 제약조건

노드 ID를 명시(**ID 전체 입력해야함**)해서 컨테이너가 할당될 노드 선택

```
root@swarm-manager:~# docker node ls | grep swarm-worker2
rtglyf8jipxzuyag3g7smmyzo     swarm-worker2       Ready               Active                                  19.03.6

root@swarm-manager:~# docker service create --constraint 'node.id == rtglyf8jipxzuyag3g7smmyzo' --replicas=5 --name label_id ubuntu:14.04 ping docker.com
```

```
root@swarm-manager:~# docker service ps label_id
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
u7a2pc89do2h        label_id.1          ubuntu:14.04        swarm-worker2       Running             Running 27 seconds ago
nlqwribuml7x        label_id.2          ubuntu:14.04        swarm-worker2       Running             Running 27 seconds ago
ogx7xgeh2ejn        label_id.3          ubuntu:14.04        swarm-worker2       Running             Running 28 seconds ago
c3yuwumd3gvc        label_id.4          ubuntu:14.04        swarm-worker2       Running             Running 28 seconds ago
qv5oxyvzb0kr        label_id.5          ubuntu:14.04        swarm-worker2       Running             Running 28 seconds ago
```



#### #3 node.hostname과 node.role 제약조건

- swarm-worker1 에만 할당

```
root@swarm-manager:~# docker service create --constraint 'node.hostname == swarm-worker1' --name label_name --replicas=5 ubuntu:14.04 ping docker.com
```

- 매니저 노드가 아닌 worker 노드에 할당

```
root@swarm-manager:~# docker service create --constraint 'node.role != manager' --name label_notmanaget --replicas=5 ubuntu:14.04 ping docker.com
```



#### #4  engine.labels 제약조건

도커 데몬 자체에 라벨을 설정해 제한함 (도커 데몬 실행 옵션을 변경해야함)

![image-20201002022710991](https://user-images.githubusercontent.com/69428620/94989622-2bf8ad80-05b1-11eb-99d6-5703ce584c78.png)






## swarm mode service

> docker    VS    swarm mode
>
> 도커 명령어의 제어 단위 : 컨테이너
>
> 스웜 모드의 제어 단위 : 서비스



### 서비스

같은 이미지에서 생성된 컨테이너의 집합

서비스 내에 1개 이상의 컨테이너 존재하고, 각 컨테이너는 manager 노드와 worker 노드에 할당됨

각 노드에  할당된 컨테이너들을 **태스크 tastk**라고 한다 

**스웜 클러스터 내의 어떤 노드로 접근해도 서비스 가능**

**(nginx 컨테이너가 없는 노드로 접근해도 swarm으로 묶여있으면 다른 노드로 넘어가서 서비스 접근 가능 ) ===> ingress network를 사용하기 때문!**



1. 서비스 생성    ``docker service create --name SERVICE_NAME --replicas 2 -p 80:80 nginx``

2. 서비스 조회    ``docker service ls ``

3. 서비스 상세 정보 조회    ``docker service ps SERVICE_NAME``

4. 서비스 삭제    ``docker service rm SERVICE_NAME``

   (docker rm 과 달리 서비스 실행여부와 관계없이 삭제 가능)

5. 레플리카셋 4개로 세팅 ``docker service scale SERVICE_NAME=4``

   (증가/감소 하고 싶으면 원하는 개수로 세팅하면 됨)



> 서비스 모드
>
> 복제모드     VS     글로벌 모드

#1 복제(replicated) 모드

- 레플리카 셋의 수를 정의 그 만큼의 컨테이너를 생성
- 실제 서비스 제공에 일반적으로 사용하는 모드
- 외부에서 들어왔을 때 유효한 컨테이너 쪽으로 라우팅 & 로드 밸런싱
- ``docker service create --name SERVICE_NAME --replicas 2 ubuntu:14.04``

#2 글로벌(global) 모드

- swarm cluster 내에 모든 노드에 반드시 하나씩 컨테이너를 생성
- 레플리카 셋의 수를 별도로 지정하지 않음
- swarm cluster를 모니터링 하기 위한 용도로 사용
- ``docker service create --name SERVICE_NAME --mode global ubuntu:14.04``





## Swarm은 replicaset 수를 일정하게 유지한다

- 레플리카 수 항상 유지

  - 특정 노드 다운되면 다른 노드에 컨테이너가 생성된다

    => 다운된 노드 재시작해도 rebalance는 일어나지 않음

    => ``docker service scale``명령어로 컨테이너 수 줄이고 증가시켜 균형맞춤

  - 특정 컨테이너 제거되면 새로운 태스크(컨테이너) 생성된다





## 서비스 롤링 업데이트

```
root@swarm-manager:~# docker service create --name myweb2 --replicas 3 nginx:1.10
nw4k4517gtyrpyvjw4i6r11qd
overall progress: 3 out of 3 tasks
1/3: running
2/3: running
3/3: running
verify: Service converged

root@swarm-manager:~# docker service ps myweb2
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
l51u0t3koxdd        myweb2.1            nginx:1.10          swarm-manager       Running             Running 20 seconds ago
dxaj3owu5xgu        myweb2.2            nginx:1.10          swarm-worker1       Running             Running 18 seconds ago
pcpyz8h1ezjq        myweb2.3            nginx:1.10          swarm-worker2       Running             Running 19 seconds ago
```



- docker service update 명령어로 서비스의 설정 변경

```
root@swarm-manager:~# docker service update --image nginx:1.11 myweb2
myweb2
overall progress: 3 out of 3 tasks
1/3: running
2/3: running
3/3: running
verify: Service converged
root@swarm-manager:~# docker service ps myweb2
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE             ERROR               PORTS
zw70jkerzb5o        myweb2.1            nginx:1.11          swarm-manager       Running             Running 25 seconds ago
l51u0t3koxdd         \_ myweb2.1        nginx:1.10          swarm-manager       Shutdown            Shutdown 32 seconds ago
opbin8hk28xf        myweb2.2            nginx:1.11          swarm-worker1       Running             Running 34 seconds ago
dxaj3owu5xgu         \_ myweb2.2        nginx:1.10          swarm-worker1       Shutdown            Shutdown 41 seconds ago
ocepirnaw11z        myweb2.3            nginx:1.11          swarm-worker2       Running             Running 16 seconds ago
pcpyz8h1ezjq         \_ myweb2.3        nginx:1.10          swarm-worker2       Shutdown            Shutdown 23 seconds agoㅋㅇㅅ2	
```

- 이전으로 돌리는 롤백

```
root@swarm-manager:~# docker service rollback myweb2
myweb2
rollback: manually requested rollback
overall progress: rolling back update: 3 out of 3 tasks
1/3: running
2/3: running
3/3: running
verify: Service converged
root@swarm-manager:~# docker service ps myweb2
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE             ERROR               PORTS
4gbhjvignlsh        myweb2.1            nginx:1.10          swarm-worker1       Running             Running 21 seconds ago
1ohq8d5w4fz7         \_ myweb2.1        nginx:1.11          swarm-worker1       Shutdown            Shutdown 23 seconds ago
4w66949p0vwj         \_ myweb2.1        nginx:1.10          swarm-worker1       Shutdown            Shutdown 3 minutes ago
vm1pwavp5v7f        myweb2.2            nginx:1.10          swarm-worker2       Running             Running 24 seconds ago
10ggq6thb3b9         \_ myweb2.2        nginx:1.11          swarm-worker2       Shutdown            Shutdown 25 seconds ago
n2ivns6ky3r1         \_ myweb2.2        nginx:1.10          swarm-worker2       Shutdown            Shutdown 3 minutes ago
yqt149b28glg        myweb2.3            nginx:1.10          swarm-manager       Running             Running 19 seconds ago
a1adbtgtnozt         \_ myweb2.3        nginx:1.11          swarm-manager       Shutdown            Shutdown 20 seconds ago
c8xjcz8ws1iw         \_ myweb2.3        nginx:1.10          swarm-manager       Shutdown            Shutdown 3 minutes ago
```



- 롤링 업데이트 주기, 컨테이너 수, 업데이트 실패시 설정

```
root@swarm-manager:~# docker service create \
--replicas 4 \      		----> 컨테이너 4개 생성
--name myweb3 \  			----> 서비스 이름 myweb3
--update-delay 10s \ 		----> 10초 단위로 업데이트
--update-parallelism 2 \	----> 업데이트 시 한번에 2개 컨테이너에 수행
nginx:1.10					----> 컨테이너 생성시 사용하는 이미지
vv3l8otvdwlxwruowkp4u2ts9
overall progress: 4 out of 4 tasks
1/4: running
2/4: running
3/4: running
4/4: running
verify: Service converged
```

- 서비스의 롤링 업데이트 설정 확인

```
root@swarm-manager:~# docker service inspect --pretty myweb3

ID:             vv3l8otvdwlxwruowkp4u2ts9
Name:           myweb3
Service Mode:   Replicated
 Replicas:      4
Placement:


UpdateConfig:
 Parallelism:   2
 Delay:         10s
 On failure:    pause   -> 업데이트 도중 오류 발생시 업데이트 중지
 Monitoring Period: 5s
 Max failure ratio: 0
 Update order:      stop-first
 
 
RollbackConfig:
 Parallelism:   1
 On failure:    pause    
 Monitoring Period: 5s
 Max failure ratio: 0
 Rollback order:    stop-first 
ContainerSpec:
 Image:         nginx:1.10@sha256:6202beb06ea61f44179e02ca965e8e13b961d12640101fca213efbfd145d7575
 Init:          false
Resources:
Endpoint Mode:  vip
```



- 업데이트 도중 오류 발생해도 업데이트 진행하도록 설정

```
root@swarm-manager:~# docker service create --name myweb4 --replicas 4 
--update-failure-action continue nginx:1.10

root@swarm-manager:~# docker service inspect --pretty myweb4
	:
UpdateConfig:
 Parallelism:   1
 On failure:    continue   ---> 확인가능
 Monitoring Period: 5s
 Max failure ratio: 0
 Update order:      stop-first
```



## 컨테이너에 설정 정보 전달

#### 방법1.  -v 옵션을 통해 설정파일이나 값을 볼륨으로 공유

```
$ docker run -t -d --name yml_registry -p 5002:5000 --restart=always \
> -v $(pwd)/config.yml:/etc/docker/registry/config.yml registry:2.6
```



#### 방법2. -e 옵션을 통해 환경변수로 전달

```
$ docker run - t -d --name wordpressdb_hostvolume \
>-e MYSQL_ROOT_PASSWORD=password \
>-e MYSQL_DATABASE=wordpress \
>-v /home/wordpress_db:/var/lib/mysql \
> mysql:5.7
```



## swarm 모드는 secret과 config 기능 제공

> 파일 공유를 위해 설정 파일을 호스트마다 마련해두는 것은 매우 비효율적
>
> 비밀번호와 같은 민감한 정보를 환경변수로 설정하는 것은 보안에 취약함

- secret : 비밀번호, ssh 키, 인증서 키 등 보안에 민감한 데이터 전송
- config : nginx 나 registry 설정 파일 등 암호화할 필요가 없는 설정값에 사용





### secret

- 생성된 secret은 조회해도 실제 값 확인 불가
- 암호화된 상태로 저장됨

> 환경변수로 -e MYSQL_ROOT_PAASSWORD=223233434
>
> 지정하면 비밀번호가 노출된다    => 정보를 감추고 싶다
>
> ==> 정보가 들어가있는 객체를 하나 만들고 && 객체 내용을 컨테이너 안에 넣기





#### #1 secret 생성

```
root@swarm-manager:~# echo 1q2w3e4r | docker secret create my_mysql_password -
3tcxh1i8tiysx0yv917vwpnhj


root@swarm-manager:~# docker secret ls
ID                          NAME                DRIVER              CREATED             UPDATED
3tcxh1i8tiysx0yv917vwpnhj   my_mysql_password                       6 seconds ago       6 seconds ago


root@swarm-manager:~# docker secret inspect my_mysql_password
[
    {
        "ID": "3tcxh1i8tiysx0yv917vwpnhj",
        "Version": {
            "Index": 230
        },
        "CreatedAt": "2020-10-01T13:21:32.89688471Z",
        "UpdatedAt": "2020-10-01T13:21:32.89688471Z",
        "Spec": {
            "Name": "my_mysql_password",
            "Labels": {}
        }
    }
]
```



#### #2 생성한 secret 사용해서  mysql 컨테이너 생성

- --secret 옵션을 통해 공유된 값은 컨테이너 내부 /run/secrets/에 마운트 됨

- --secret source=my_mysql_password,target=mysql_root_password 처럼

  마운트 될 디렉터리도 지정 가능

```
root@swarm-manager:~# docker service create --name mysql --replicas 1 \
> --secret source=my_mysql_password,target=mysql_root_password \
> --secret source=my_mysql_password,target=mysql_password \
> -e MYSQL_ROOT_PASSWORD_FILE="/run/secrets/mysql_root_password" \
> -e MYSQL_PASSWORD_FILE="/run/secrets/mysql_password" \
> -e MYSQL_DATABASE="wordpress" mysql:5.7
```

#### #3 mysql 컨테이너가 어떤 노드에 있는지 확인

```
root@swarm-manager:~# docker service ps mysql
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
nkada0bk67hp        mysql.1             mysql:5.7           swarm-manager       Running             Running about a minute ago


root@swarm-manager:~# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                 NAMES
e9724cf552aa        mysql:5.7           "docker-entrypoint.s…"   2 minutes ago       Up 2 minutes        3306/tcp, 33060/tcp   mysql.1.nkada0bk67hpr2rd9kh1ger30
```

#### #4 mysql 컨테이너 안에 secret 확인

```
root@swarm-manager:~# docker exec mysql.1.nkada0bk67hpr2rd9kh1ger30 ls /run/secrets
mysql_password
mysql_root_password

root@swarm-manager:~# docker exec mysql.1.nkada0bk67hpr2rd9kh1ger30 cat /run/secrets/mysql_password
1q2w3e4r

root@swarm-manager:~# docker exec mysql.1.nkada0bk67hpr2rd9kh1ger30 cat /run/secrets/mysql_root_password
1q2w3e4r
```







### config

> config 사용하는 이유??

하나의 머신에서 하나의 컨테이너를 띄울때

docker run ... -v ./config.yml:/etc/docker/registry/config.yml



도커 스웜에서 컨테이너를 띄울때

노드마다 config.yml 파일이있어야한다.  ==> 비효율적

==<> 매니저 노드에 config.yml을 통해 config를 만들고 컨테이너에 할당

==> 매니저 노드에만 config파일이 있으면 된다







#### #1 사설 레지스트리 설정 파일 생성

```
root@swarm-manager:~# cat config.yml
version: 0.1
log:
  level: info
storage:
  filesystem:
    rootdirectory: /registry_data
  delete:
    enabled: true
http:
  addr: 0.0.0.0:5000
```

#### #2 config 생성 및 확인

```
root@swarm-manager:~# docker config create registry-config config.yml
kwcj1f1os66229k0g9isrgmr1

root@swarm-manager:~# docker config ls
ID                          NAME                CREATED             UPDATED
kwcj1f1os66229k0g9isrgmr1   registry-config     2 seconds ago       2 seconds ago
```

#### #3 config는 입력값을 BASE64로 인코딩해서 저장

```
root@swarm-manager:~# docker config inspect registry-config
[
    {
        "ID": "kwcj1f1os66229k0g9isrgmr1",
        "Version": {
            "Index": 244
        },
        "CreatedAt": "2020-10-01T13:34:18.602086612Z",
        "UpdatedAt": "2020-10-01T13:34:18.602086612Z",
        "Spec": {
            "Name": "registry-config",
            "Labels": {},
            "Data": "dmVyc2lvbjogMC4xCmxvZzoKICBsZXZlbDogaW5mbwpzdG9yYWdlOgogIGZpbGVzeXN0ZW06CiAgICByb290ZGlyZWN0b3J5OiAvcmVnaXN0cnlfZGF0YQogIGRlbGV0ZToKICAgIGVuYWJsZWQ6IHRydWUKaHR0cDoKICBhZGRyOiAwLjAuMC4wOjUwMDAK"
        }
    }
]
```

```
## 입력한 데이터 bas64로 디코딩해서 보기
root@swarm-manager:~# echo dmVyc2lvbjogMC4xCmxvZzoKICBsZXZlbDogaW5mbwpzdG9yYWdlOgogIGZpbGVzeXN0ZW06CiAgICByb290ZGlyZWN0b3J5OiAvcmVnaXN0cnlfZGF0YQogIGRlbGV0ZToKICAgIGVuYWJsZWQ6IHRydWUKaHR0cDoKICBhZGRyOiAwLjAuMC4wOjUwMDAK | base64 -d
version: 0.1
log:
  level: info
storage:
  filesystem:
    rootdirectory: /registry_data
  delete:
    enabled: true
http:
  addr: 0.0.0.0:5000
```



#### #4 config로 사설 레지스트리 생성

```
root@swarm-manager:~# docker service create --name yml_registry -p 5000:5000 \
> --config source=registry-config,target=/etc/docker/registry/config.yml registry:2.6
tw50bjd0xjrrwgqufcq4nrr7p
overall progress: 1 out of 1 tasks
1/1: running
verify: Service converged
root@swarm-manager:~# docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
tw50bjd0xjrr        yml_registry        replicated          1/1                 registry:2.6        *:5000->5000/tcp
```

```
root@swarm-manager:~# docker exec yml_registry.1.caq5zvb1b1pptgvi7h0pzntnz cat /etc/docker/registry/config.yml
version: 0.1
log:
  level: info
storage:
  filesystem:
    rootdirectory: /registry_data
  delete:
    enabled: true
http:
  addr: 0.0.0.0:5000
```



## 클러스터 컨테이너 배치상태 시각화

https://hub.docker.com/r/dockersamples/visualizer

```
vagrant@swarm-manager:~$ docker service create \
>   --name=viz \
>   --publish=8080:8080/tcp \
>   --constraint=node.role==manager \
>   --mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
>   dockersamples/visualizer
emrk6zvniul9apwwjoerdvjmj
overall progress: 1 out of 1 tasks
1/1: running   [==================================================>]
verify: Service converged
```







## ingress/overlay/docker_gwbridge network

### 매니저 노드에서 네트워크 목록 확인

```
root@swarm-manager:~# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
e5c54ca716c0        bridge              bridge              local
531dc4a747be        docker_gwbridge     bridge              local
a420af64f234        host                host                local
ywuvres5iicc        ingress             overlay             swarm
04767240199d        none                null                local
```

- docker_gwbridge : swarm에서 overlay 네트워크 사용할때 사용
- ingress:  load balacing과 rounting mesh 에 사용



### ingress network

- swarm cluster 생성하면 자동으로 등록되는 네트워크 => swarm에서만 유효
- swarm cluster 에 등록된 모든 노드에 ingress 네트워크 생성됨



![image-20201002022710991](https://user-images.githubusercontent.com/69428620/94989634-43379b00-05b1-11eb-8845-7035f4b45ead.png)


 **라우팅 메시**:어떤 노드에 접근하더라도 서비스 내의 컨테이너에 접근할 수 있게 설정 

ex. 	worker2에는 컨테이너가 없는상태 

​		  worker2로 접속 시 -> manager,worker1에서 서비스 중인 컨테이너로 연결시켜줌

**로드 밸런싱**:서비스 내의 컨테이너에 대한 접근을 라운드 로빈 방식으로 분산

ex.	manager ip로 계속 접속함 -> manager.worker1 로 돌아가면서 접속됨

​		-> 한 컨테이너만 사용하지 않고 여러개에 분산시켜서 접속함으로 부하 낮춤



### overlay network

여러 개의 도커 데몬을 하나의 네트워크 풀로 만드는 네트워크 가상화 기술의 하나

도커에 오버레이 네트워크를 적용하면 여러 도커에 존재하는 컨테이너가 서로통신할 수 있다.

==> 포트 포워딩 없어서 각 컨테이너 간 전송/연결이 가능!

**ingress network도 overlay network 중의 하나!**



여러개의 swarm 노드에 할당된 컨테이너는 overlay 네트워크의 서브넷에 해당하는 ip 대역을 할당받고 이 ip를 통해 서로 통신한다

컨테이너 내부의 네트워크에서 eth0과 연결됨





### docker_gwbridge network

overlay 네트워크를 사용하지 않는 컨테이너는 기본적으로 존재하는 브리지(bridge) 네트워크를 사용해 외부와 연결한다



그러나, ingress를 포함한 모든 overlay 네트워크는 bridge 네트워크와 다른

docker_gwbridge 네트워크와 함께 사용된다.



- 외부로 나가는 통신 담당
- overlay 네트워크의 트래픽 종단점(VTEP) 역할을 담당

- 컨테이너 내부의 네트워크에서 eth1과 연결됨

![image-20200917161629336](https://user-images.githubusercontent.com/69428620/94989640-4df23000-05b1-11eb-8ac6-89d0e0688b68.png)


![image-20200918095811808](https://user-images.githubusercontent.com/69428620/94989645-59ddf200-05b1-11eb-9f02-b7204101b3fd.png)






### 사용자 정의 overlay network

#### #1 사용자 정의 overlay 네트워크 생성

```
root@swarm-manager:~# docker network create \
>--subnet 10.0.9.0/24 \		--> overlay 네트워크의 subnet 지정
>-d overlay \				--> 네트워크 드라이버를 overlay로 지정
>myoverlay					--> overlay 네트워크 이름
ypzrzxzpyt9nnkif9tznzi5dp

```

#### #2 생성한 overlay 네트워크 확인

```
root@swarm-manager:~# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
e5c54ca716c0        bridge              bridge              local
531dc4a747be        docker_gwbridge     bridge              local
a420af64f234        host                host                local
ywuvres5iicc        ingress             overlay             swarm
ypzrzxzpyt9n        myoverlay           overlay             swarm
04767240199d        none                null                local
```

#### #3 overlay 네트워크를 서비스에 적용(--network 옵션 이용)해 컨테이너 생성

```
root@swarm-manager:~# docker service create --name overlay_service --network myoverlay --replicas 2 alicek106/book:hostname
```

#### #4 생성된 컨테이너에 할당된 overlay 네트워크 IP 주소 확인(eth0)

```
root@swarm-manager:~# docker service ps overlay_service
ID                  NAME                IMAGE                     NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
b4pst5f9aau2        overlay_service.1   alicek106/book:hostname   swarm-worker1       Running             Running 29 seconds ago
oyyvr12zb8sl        overlay_service.2   alicek106/book:hostname   swarm-manager       Running             Running 29 seconds ago
```

```
root@swarm-manager:~# docker exec overlay_service.2.oyyvr12zb8sldmjjn222h6s7n ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:0a:00:09:04
          inet addr:10.0.9.4  Bcast:10.0.9.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1450  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

```
root@swarm-worker1:~# docker exec overlay_service.1.b4pst5f9aau2c9f1pfjm6spp2 ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:0a:00:09:03
          inet addr:10.0.9.3  Bcast:10.0.9.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1450  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

==> 10.0.9.4 와 10.0.9.3 로 manager와 worker1에 떨어져 있지만, 동일한 네트워크 대역이다!

![image-20200917180054953](https://user-images.githubusercontent.com/69428620/94989652-64988700-05b1-11eb-9d5a-0835e966b111.png)






## Service Discovery

같은 컨테이너를 여러 개 만들어 사용 시 , 새로 생성된 컨테이너 발견(Service Discovery) 혹은 없어진 컨테이너 감지가 중요하다



일반적으로 주키퍼, etcd 등의 분산 코디네이터를 외부에 두고 사용하지만,

swarm 모드는 service discovery 기능을 자체적으로 지원한다



swarm 모드에서는 service이름으로 해당 service의 모든 컨테이너에 접근가능

=> 컨테이너 IP 주소 알 필요x , 새롭게 생성된 사실도 알필요 x



### #1 overlay 네트워크 생성

```
root@swarm-manager:~# docker network create -d overlay discovery
pkdrkddk4oghcdmhaaqtjun57
root@swarm-manager:~# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
e5c54ca716c0        bridge              bridge              local
pkdrkddk4ogh        discovery           overlay             swarm
531dc4a747be        docker_gwbridge     bridge              local
a420af64f234        host                host                local
ywuvres5iicc        ingress             overlay             swarm
04767240199d        none                null                local
```

### #2 discovery 네트워크 사용하는 server 서비스생성 

​	: 컨테이너의 호스트이름을 출력하는 웹서버 컨테이너 2개 생성

```
root@swarm-manager:~# docker service create --name server --replicas 2 --network discovery alicek106/book:hostname
```

### #3 discovery 네트워크 사용하는 clinet 서비스 생성

​	: server 서비스에 http 요청을 보낼 컨테이너 생성

```
root@swarm-manager:~# docker service create --name client --network discovery alicek106/book:curl ping docker.com
```

### #4  실행 노드 확인

```
root@swarm-manager:~# docker service ps server
ID                  NAME                IMAGE                     NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
6r384ogxg8tn        server.1            alicek106/book:hostname   swarm-manager       Running             Running 15 minutes ago
zgz63ckdrm8s        server.2            alicek106/book:hostname   swarm-worker1       Running             Running 15 minutes ago

root@swarm-manager:~# docker service ps client
ID                  NAME                IMAGE                 NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
2yitgahycwvc        client.1            alicek106/book:curl   swarm-worker2       Running             Running 3 minutes ago
```

### #5 client 컨테이너 내부에서 curl 명령어로 server 서비스 접근

```
root@swarm-worker2:~# docker container ls
CONTAINER ID        IMAGE                 COMMAND             CREATED             STATUS              PORTS               NAMES
1d62bb2c85bd        alicek106/book:curl   "ping docker.com"   5 minutes ago       Up 5 minutes                            client.1.2yitgahycwvcfutffcws5erzy
root@swarm-worker2:~# docker exec -it client.1.2yitgahycwvcfutffcws5erzy /bin/bash
```

```
root@1d62bb2c85bd:/# curl -s server | grep Hello
        <p>Hello,  99d7c205a7da</p>     </blockquote>
root@1d62bb2c85bd:/# curl -s server | grep Hello
        <p>Hello,  4a6338bc2639</p>     </blockquote>
```

-  overlay 네트워크에 속하면 swarm의 DNS가 이를 자동으로 변환(resolve)

-  요청 보낼때마다 다른 컨테이너에 접근

  (round-robbin 방식: server 컨테이너 중 하나로 매핑)



### #6 server 서비스 replica 수를 3개로 증가해도 확인가능

```
root@swarm-manager:~# docker service scale server=3
```

```
root@1d62bb2c85bd:/# curl -s server | grep Hello
        <p>Hello,  2164ee512b00</p>     </blockquote>
root@1d62bb2c85bd:/# curl -s server | grep Hello
        <p>Hello,  99d7c205a7da</p>     </blockquote>
root@1d62bb2c85bd:/# curl -s server | grep Hello
        <p>Hello,  4a6338bc2639</p>     </blockquote>
```

![image-20200917173115215](https://user-images.githubusercontent.com/69428620/94989667-78dc8400-05b1-11eb-92b2-f8bdd1c0d2f3.png)


### #7  sever 서비스의 VIP 확인

```
root@swarm-manager:~#  docker service inspect --format {{.Endpoint.VirtualIPs}} server
[{pkdrkddk4oghcdmhaaqtjun57 10.0.1.2/24}]
```

- server 라는 호스트이름이 3개의 IP를 가지는게 아니라 VIP(Virtaul IP)를 가진다

- swarm 의 내장 DNS 서버는 server라는 호스트 이름을 10.0.1.2 IP로 변환

- 컨테이너의 네트워크 네임스페이스 내부에서 실제 server 서비스 컨테이너의 IP로 포워딩해줌



### #8 VIP 방식이 아닌 도커의 내장 DNS 서버 기반으로 라운드 로빈 적용

```
root@swarm-manager:~# docker service create --name server-dnsrr --replicas 2 --network discovery \
> --endpoint-mode dnsrr alicek106/book:hostname
smqvifagyijnju9awjv8gfjg2
overall progress: 2 out of 2 tasks
1/2: running
2/2: running
verify: Service converged
```

- 애플리케이션에 따라 캐시 문제로 서비스 발견이 정상 작동이 안될때가 있음==> 가급적 VIP를 사용하자!







## Swarm volume

> docker에서는 run 명령어와 -v 옵션을 함께 사용 시,
>
> 호스트와 디렉터리 공유하는지/ 볼륨을 사용하는 지 구분이 없음

> swarm 모드에서는 호스트와 디렉터리 공유할지/docker volume을 사용할지 명확하게 명시한다



### volume 타입의 볼륨 생성

```
root@swarm-manager:~# docker service create --name ubuntu --mount type=volume,source=myvol,target=/root ubuntu:14.04 ping docker.com
```

-  --mount 옵션에 type값에 volume 지정 ==> docker volume을 사용

- source => 사용할 볼륨(존재하면 사용하고, 없으면 새로 생성)

  ==> source 명시 하지 않으면 임의의 16진수 이름의 볼륨 생성됨

- target => 컨테이너 내부에 mount될 디렉터리 위치



-- 볼륨의 공유할 컨테이너가 이미 존재하면 볼륨에 복사되고, 호스트에서 별도의 공간을 차지하게 된다

```
root@swarm-manager:~# docker service create --name ubuntu --mount type=volume,source=test,target=/etc/vim/ ubuntu:14.04 ping docker.com
```

```
root@swarm-manager:~# docker run -it --name test -v test:/root ubuntu:14.04
Unable to find image 'ubuntu:14.04' locally
14.04: Pulling from library/ubuntu
Digest: sha256:63fce984528cec8714c365919882f8fb64c8a3edf23fdfa0b218a2756125456f
Status: Downloaded newer image for ubuntu:14.04
root@940d1536562d:/# ls root
vimrc  vimrc.tiny
```

- volume-nocopy 옵션 주면 컨테이너에 볼륨내용이 복사되지 않음

```
root@swarm-manager:~# docker service create --name ubuntu --mount type=volume,source=test,target=/etc/vim/,volume-nocopy ubuntu:14.04 ping docker.com
```



- volume 제거 : ``docker volume rm -f  Volume_Name``
- volume 전체제거 : ``docker volume rm -f $(docker volume -q)``



### bind 타입의 볼륨 생성

- 호스트와 디렉터리 공유할 때 사용
- type 옵션값을 bind로 설정
- source 옵션을 반드시 명시해야 함(공유될 호스트 디렉터리 설정 필수!)

```
root@swarm-manager:~# docker service create --name ubuntu \
> --mount type=bind,src=/root/host/,dst=/root/container ubuntu:14.04 ping docker.com
```

- host의 /root/host 디렉터리를 컨테이너의 /root/container 에 mount



### swarm에서의 volume의 한계

- swarm cluster 의 모든 노드가 volume 데이터를 가지고 있어야 한다

  ==> 볼륨사용하기 비효율적 (볼륨 없는 노드에는 컨테이너 할당 x)

**해결법 : Persistent Storage 사용**



### Persistent Storage

- 호스트, 컨테이너와 별개로 외부에 존재해 네트워크로 mount할 수 있는 storage
- docker에서 제공 x --> thirdparty plugin 사용하거나 nfs,dfs 별도 구성필요









# 오타에러 조심하자!

verify : Detected task failure

===> 오타 에러일 확률

> 1. 명령어/파라미터값 오타
>
> 2. 참조하는 파일 내용 오타 (   **특히, go 형식 파일(.yml,yaml)에서 띄어쓰기 조심!**)
>
>    ex) level: info         -> level:info 해도 에러  & level : info 해도 에러임!!!!!
>
>   ![image-20200917155322472](https://user-images.githubusercontent.com/69428620/94989679-8a259080-05b1-11eb-90ac-c3014d32d7fd.png)


```
Error response from daemon: rpc error: code = DeadlineExceeded desc = context deadline exceeded
```

![image-20200917154401037](https://user-images.githubusercontent.com/69428620/94989687-94e02580-05b1-11eb-9b50-eb6e19e95358.png)


=======> ,  있으면 안되는 곳에 있어서 난 에러

