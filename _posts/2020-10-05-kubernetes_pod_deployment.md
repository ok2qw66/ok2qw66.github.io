---
title:  "수업정리(7)- kubernetes pod/deployment"
date:   2020-10-05 02:00:50 +0900
categories: 
  - 수업정리
tags:
  - Kubernetes
  - Pod
  - Deployment
toc: true
toc_sticky: true
---



## Kubernetes란?

- 대부분의 리소스를 "오브젝트"라고 불리는 형태로 관리
- 컨테이너의 집합(pods), pods를 관리하는 컨트롤러(replica set), 사용자(service account), 노드(node) 까지 하나의 오브젝트로 사용가능
- kubectl 명령어 또는 YAML 파일을 정의해서 kubernetes 사용
- 다수의 컨테이너들을 관리하는 오케스트레이션(Orchestration) 플랫폼

### Kubernetes에서 사용가능한 오브젝트

- 사용가능한 오프젝트 목록 확인: ``kubectl api-resources``
- 특정 오브젝트에 대한 상세설명 : ``kubectl explain OBJECT_NAME``



### Kubernetes 구성

https://m.blog.naver.com/sharplee7/221728275736

![image-20201003223123338](https://user-images.githubusercontent.com/69428620/95019443-4eb5bf80-06a0-11eb-921c-17987d15e254.png)

#### 마스터 노드

전체 Kubernetes 시스템을 제어,관리하는 Kubernetes control plane을 실행



#### 워커 노드(Worker Node)

실제 배포되는 컨테이너 애플리케이션을 실행

애플리케이션을 실행/모니터링하며 애플리케이션에 서비스 제공

##### 구성요소

- 컨테이너를 실행하는 도커, rkt, 다른 컨테이너 런타임(container runtime)
- API 서버와 통신하고 노드의 컨테이너를 관리하는 Kubelet
- 애플리케이션 구성요소 간에 네트워크 트래픽을 로드밸런싱하는 Kubernetes 서비스 프록시 



#### 컨트롤 플레인(Control Plane)

클러스터를 제어하고 작동시킴   **cf. 애플리케이션 실행은 노드에서 처리!**

##### 구성요소

- Kubernetes API 서버 : 사용자 컨트롤 플레인 구성요소와 통신

- 스케줄러 : 애플리케이션 배포 담당(애플리케이션 배포가능한 각 구성요소를 워크 노드에 할당)

- 컨트롤 매니저 : 클러스터 단의 기능 수행

  (구성 요소 복제본, 워커노드 추적, 노드 장애처리 등)

- etcd : 클러스터 구성을 지속적으로 저장하는 신뢰가능한 분산 데이터 저장소





## Kubernetest 환경 설정

https://kubernetes.io/ko/docs/tasks/tools/install-kubectl/

### #1. vagrantfile 생성

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.hostname = "ubuntu"
  config.vm.network "private_network", ip: "192.168.111.110"
  config.vm.synced_folder ".", "/home/vagrant/sync", disabled: true
  config.vm.provider "virtualbox" do |vb|
   vb.cpus = 2
   vb.memory = 2048
  end
  config.vm.provision "shell", inline: $script
end

$script = <<SCRIPT
  apt-get update
  apt-get upgrade
  apt install -y docker.io
  usermod -a -G docker vagrant
  service docker restart
  chmod 666 /var/run/docker.sock
  apt-get install -y apt-transport-https gnupg2
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
  echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | tee -a /etc/apt/sources.list.d/kubernetes.list
  apt-get update
  apt-get install -y kubectl
  curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube
  mkdir -p /usr/local/bin/
  install minikube /usr/local/bin/
SCRIPT
```



### #2. 설치 확인 및 minikube 실행

#### 1) 가상환경 접속

```
C:\kubernetes>vagrant up
C:\kubernetes>vagrant ssh
```

#### 2) docker, kubectl 설치 확인

``docker version`` 또는 ``docker --version``

``kubectl version`` 또는 ``docker version --short``

```
vagrant@ubuntu:~$ docker --version
Docker version 19.03.6, build 369ce74a3c
```

```
vagrant@ubuntu:~$ kubectl version --short
Client Version: v1.19.2
Server Version: v1.19.2
```

#### 3) minikube 실행 및 상태 확인

>  minikube란??

https://kubernetes.io/ko/docs/tasks/tools/install-minikube/

> kubernetes는 마스터노드와 하나이상의 워커노드로 구성되어있다.
>
> => 단순 개발테스트를 위해 개인이 플랫폼 구성하는것은 어렵다
>
> ==> minikube 사용하자!
>
> (마스터 노드의 일부기능과 개발,배포 위한 단일 워커 노드를 제공해주는 간단한 쿠버네티스 플랫폼)

- minikube 실행 : ``minikube start``

```
vagrant@ubuntu:~$ minikube start
😄  minikube v1.13.1 on Ubuntu 18.04 (vbox/amd64)
✨  Automatically selected the docker driver

🧯  The requested memory allocation of 1992MiB does not leave room for system overhead (total s
ystem memory: 1992MiB). You may face stability issues.
💡  Suggestion: Start minikube with less memory allocated: 'minikube start --memory=1992mb'

👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
💾  Downloading Kubernetes v1.19.2 preload ...
    > preloaded-images-k8s-v6-v1.19.2-docker-overlay2-amd64.tar.lz4: 486.36 MiB
🔥  Creating docker container (CPUs=2, Memory=1992MB) ...
🐳  Preparing Kubernetes v1.19.2 on Docker 19.03.8 ...
🔎  Verifying Kubernetes components...
🌟  Enabled addons: default-storageclass, storage-provisioner
🏄  Done! kubectl is now configured to use "minikube" by default
```

- minikube 상태확인: ``minikube status``

```
vagrant@ubuntu:~$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

- 중지 : ``minikube stop``
- 삭제: ``minikube delete``



# Kubernetes 명령어

| 설명                    | 명령어                                                       |
| ----------------------- | ------------------------------------------------------------ |
| 파드 생성               | kubectl apply -f YAML파일명                                  |
| 파드 확인               | kubectl get pods  또는 kubectl get po                        |
| 리소스 상세정보 확인    | kubectl describe pods POD_NAME                               |
| 파드 컨테이너 접속      | kubectl exec -it POD_NAME -- bash                            |
| 파드 특정 컨테이너 접속 | kubectl exec -it POD_NAME -c CONTAINER_NAME -- bash          |
| 파드 로그 확인          | kubectl logs POD_NAME                                        |
| 파드 오브젝트 제거      | kubectl delete -f YAML파일명<br />kubectl delete pods POD_NAME |





# Pod 포드

컨테이너 애플리케이션의 기본 단위

1개 이상의 컨테이너로 구성된 컨테이너 집합

여러개의 컨테이너가 하나의 애플리케이션으로 동작하도록 묶은 단위



pod: 여러개 컨테이너를 묶어서 감싸는 느낌?

kubenetes에서 쓰는 개념

컨테이너와 개념 동일하지만 연관이 있는 컨테이너를 연합시키기 위해서 사용

===> 여러개의 노드에서 사용가능 :  여러 머신에서 여러 컨테이너를 연결



> 컨테이너 VS  포드

컨테이너 = 하나의 어플리케이션을 동작하는 감싸는 덩어리

포드 pod = 컨테이너를 감싸는 덩어리 / 네트워크 등 공유하기 좋은 구조를 만듬

서비스를 위해서 컨테이너를 추상적으로 연결하는 것



## Pod를 사용하는 이유

> 왜 Pod를 사용할까?

**같은 리눅스 네임스페이스를 공유 => 컨테이너간의 연결이 쉽다**

일반적으로 포드 하나는 컨테이너 하나를 넣는다 & 서비스를 위해 컨테이너 여러개 넣을 수도 있다

===> 왜 포드를 사용하지??

===> 서비스를 위해서     컨테이너 간의 연계를 쉽게하기 위해서 

===> 연결을 위한 설정에 집중하는 것이 아니라  서비스에 집중할수있음



wordpress 이용하려면 db랑 연동해야 한다

ip 주소 넣어서 하면 복잡해진다... => 포드로 연결하면 쉽다







컴포즈 : 하나의 노드에서 여러개의 컨테이너를 쉽게 조작할때

일반적으로 연관성있는 컨테이너 동시에 띄울때

==> 단일 노드 : 하나의 머신에서 여러개의 컨테이너 연결



dind : docker in docker 

컨테이너 안에서 컨테이너를 띄운다









## 1개 nginx 컨테이너로 구성된 포드 생성 

### #1. yml파일 생성

```
vagrant@ubuntu:~/kubetest$ cat nginx-pod.yml
apiVersion: v1					-> YAML 파일에서 정의한 오브젝트의 API 버전
kind: Pod						-> 리소스 종류(kubectl api-resources 명령의 KIND 항목)
metadata:
  name: my-nginx-pod
spec:
  containers:					-> 리소스 생성을 위한 정보
    - name: my-nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
          protocol: TCP
```

### #2. 새로운 파드 생성 및 확인

- 파드 생성 : ``kubectl apply -f YAML파일명``
- 파드 확인 : ``kubectl get pods``또는 ``kubectl get po``
- 리소스 상세정보 확인: ``kubectl describe pods POD_NAME``

```
vagrant@ubuntu:~/kubetest$ kubectl apply -f nginx-pod.yml
pod/my-nginx-pod created

vagrant@ubuntu:~/kubetest$ kubectl get pods
NAME           READY   STATUS              RESTARTS   AGE
my-nginx-pod   0/1     ContainerCreating   0          5s
```

```
vagrant@ubuntu:~/kubetest$ kubectl describe pods my-nginx-pod
Name:         my-nginx-pod
Namespace:    default
Priority:     0
Node:         minikube/172.17.0.2
Start Time:   Fri, 18 Sep 2020 06:35:11 +0000
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           172.18.0.3
IPs:
  IP:  172.18.0.3			-> IP 확인!
Containers:
	:
```

### #3.클러스터 내부에 임시 포드 생성해서 nginx 동작 확인

```
vagrant@ubuntu:~$ kubectl run -it --rm debug --image=alicek106/ubuntu:curl \
> --restart=Never bash
```

- -it : pod 실행과 동시에 pod 내부 접속해서 입출력 가능
- --rm: pod 빠져나오는 동시에 삭제

### #4. debug 포드에서 my-nginx-pod로 요청 전달

```
root@debug:/# curl 172.18.0.3

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### #5. kubectl exec 명령으로 포드 컨테이너에 명령어 전달 

```
vagrant@ubuntu:~$ kubectl exec -it my-nginx-pod -- bash
root@my-nginx-pod:/# ls /etc/nginx/
conf.d          koi-utf  mime.types  nginx.conf   uwsgi_params
fastcgi_params  koi-win  modules     scgi_params  win-utf
```

### #6. 포드의 로그 확인

```
vagrant@ubuntu:~$ kubectl logs my-nginx-pod

/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
172.18.0.4 - - [04/Oct/2020:04:45:45 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.35.0" "-"
```

### #7. 오브젝트 삭제

yml파일 통해 삭제(yml파일은 그대로 존재) : ``kubectl delete -f YAML파일명``

pod 이름으로 삭제 : ``kubectl delete pods POD_NAME``

```
vagrant@ubuntu:~$ kubectl delete -f kubetest/nginx-pod.yml
pod "my-nginx-pod" deleted
```



## 완전한 애플리케이션으로서의 포드

nginx컨테이너가 실행되기 위해 리로더 프로세스(설정파일 변경사항 갱신) 나 로그수집프로세스 등 부가기능이 필요한 경우, **nginx와 함께 기능확장에 필요한 추가 컨테이너(sidecar container)를 포드에 포함**하면 완전한 애플리케이션으로 작업가능하다.

## 

## 2개 컨테이너로 구성된 pod 생성

### #1. yml 파일 생성

```
vagrant@ubuntu:~/kubetest$ cat nginx-pod-with-ubuntu.yml

apiVersion: v1
kind: Pod
metadata:
  name: my-nginx-pod
spec:
  containers:
   - name: my-nginx-container
     image: nginx:latest
     ports:
      - containerPort: 80
        protocol: TCP
   - name: ubuntu-sidecar-container
     image: alicek106/rr-test:curl
     command: [ "tail" ]
     args: [ "-f", "/dev/null" ]
```

### #2. pod 생성 및 확인

```
vagrant@ubuntu:~/kubetest$ kubectl apply -f nginx-pod-with-ubuntu.yml
pod/my-nginx-pod created
```

```
vagrant@ubuntu:~/kubetest$ kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
my-nginx-pod   2/2     Running   0          86s
```

### #3. 특정 컨테이너에 명령어 전달 : -c 옵션 사용

pod로 명령어를 전달시, 실행컨테이너 지정하지 않으면 default컨테이너 실행

**ex.** #2에서는 my-nginx-container와 ubuntu-sidecar-container 실행중

``kubectl exec -it my-nginx-pod -- bash``==> my-nginx-container 실행

```
vagrant@ubuntu:~/kubetest$ kubectl exec -it my-nginx-pod -- bash
Defaulting container name to my-nginx-container.
Use 'kubectl describe pod/my-nginx-pod -n default' to see all of the containers in this pod.

root@my-nginx-pod:/# curl localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
		:
```

- -c 옵션 사용해서 ubuntu-sidecar-container 접속

```
vagrant@ubuntu:~/kubetest$ kubectl exec -it my-nginx-pod -c ubuntu-sidecar-container -- bash
root@my-nginx-pod:/# hostname
my-nginx-pod
root@my-nginx-pod:/# curl localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
		:
```

## Pod를 사용하는 이유 : 같은 namespace 공유

컨테이너간의 통신을 편하게 할 수 있음

- nginx는 my-nginx-container에만 설치되어 있음

- 둘다 localhost로 nginx 통신 가능 

  ==> pod 내부 컨테이너는 같은 namespace 공유하기 때문





# 레프리카 셋 (Replica Set)

https://kubernetes.io/ko/docs/concepts/workloads/controllers/replicaset/

- 항상 정해진 개수의 pod가 실행 유지시켜줌
- 노드 장애 등 포드 사용불가하면 다른 노드에서 포드 새로 생성



## 레플리카 셋이 필요한 이유

kubectl delete 명령어로 pod 삭제하면 내부 컨테이너도 함께 삭제됨

=> 실제 서비스 환경에서는 여러개의 동일한 컨테이너가 따로 작동해야함

=> 동일한 여러개의 포드 생성해서 각 포드에 분배해야함

```
apiVersion: v1					
kind: Pod						
metadata:
  name: my-nginx-pod-a
spec:
  containers:					
    - name: my-nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
          protocol: TCP
---						-> 여러개 리소스 정의시 구분자로 사용
apiVersion: v1					
kind: Pod						
metadata:
  name: my-nginx-pod-b
spec:
  containers:					
    - name: my-nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
          protocol: TCP
```

==> 동일한 pod를 일일이 정의하는 것은 비효율적

==> 포드 삭제/장애 발생 시. 관리자가 직접 포드삭제/다시 생성해야 함



## 레플리카 셋 명령어

| 설명                       | 명령어                                                       |
| -------------------------- | ------------------------------------------------------------ |
| 레플리카셋 생성            | kubecl apply -f REPLICASET_YML_FILE                          |
| 레플리카셋 목록 확인       | kubectl get replicaset                                       |
| 레플리카셋 삭제            | kubectl delete replicaset REPLICASET_NAME<br />kubectl delete rs REPLICASET_NAME <br />kubectl delete -f REPLICASET_YML_FILE |
| 라벨 포함해서 pod목록 확인 | kubectl get pods --show-labels                               |
| 특정 pod 수정하기          | kubectl edit pods POD_NAME                                   |



## 레플리카 셋 정의

- selector : 유지할 pod를 식별하는 방법

  (selector label app 이름과 동일한 pod 개수를 찾아서 개수 유지)

  개수를 만족하지 못하면, template 내용에 맞춰 pod 생성해줌

- replicas: 유지해야하는 pod의 개수 명시

- spec.template:  pod 스펙, 템플렛, 생성할 pod를 명시

  (pod를 신규생성해야 할때 만들어야할 pod 내용을 알려줌)



### #1. yml 파일 생성

```
vagrant@ubuntu:~/kubetest$ cat replicaset-nginx.yml

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-nginx
spec:
  replicas: 3					-> 유지할 pod 개수
  selector:
    matchLabels:
      app: my-nginx-pods-label	-> 유지할 pod인지 label로 확인
  template:						-> 신규생성 pod에 대한 데이터
    metadata:
      name: my-nginx-pod
      labels:
        app: my-nginx-pods-label-> 유지할pod label 
    spec:
      containers:
        - name: my-nginx-container
          image: nginx:latest
          ports:
            - containerPort: 80
              protocol: TCP
```

### #2. 레플리카 셋 생성 및 확인

```
vagrant@ubuntu:~/kubetest$ kubectl apply -f replicaset-nginx.yml
replicaset.apps/replicaset-nginx created
```

```
vagrant@ubuntu:~/kubetest$ kubectl get pod
NAME                     READY   STATUS    RESTARTS   AGE
replicaset-nginx-d5jn4   1/1     Running   0          31s
replicaset-nginx-lxz5z   1/1     Running   0          31s
replicaset-nginx-sj5zh   1/1     Running   0          31s
```

레플리카셋 목록 확인 : ``kubectl  get replicaset``

```
vagrant@ubuntu:~/kubetest$ kubectl get replicaset
NAME               DESIRED   CURRENT   READY   AGE
replicaset-nginx   3         3         3       53s
```

### #3. pod 개수를 늘려서 실행

- #1 yml 파일에서 replicaset: 4로 변경 후 실행

```
vagrant@ubuntu:~/kubetest$ kubectl apply -f replicaset-nginx-4pods.yml
replicaset.apps/replicaset-nginx configured
```

- ``configured`` : 기존 리소스 수정했다는 의미

```
vagrant@ubuntu:~/kubetest$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
replicaset-nginx-d5jn4   1/1     Running   0          6m8s
replicaset-nginx-lxz5z   1/1     Running   0          6m8s
replicaset-nginx-phxqb   1/1     Running   0          89s
replicaset-nginx-sj5zh   1/1     Running   0          6m8s

vagrant@ubuntu:~/kubetest$ kubectl get replicaset
NAME               DESIRED   CURRENT   READY   AGE
replicaset-nginx   4         4         4       6m21s
```

### #4. 레플리카셋 삭제 -> pod도 삭제

``kubectl delete replicaset REPLICASET_NAME``

``kubectl delete -f REPLICASET_YML_FILE``

```
vagrant@ubuntu:~/kubetest$ kubectl delete rs replicaset-nginx
replicaset.apps "replicaset-nginx" deleted

vagrant@ubuntu:~/kubetest$ kubectl get pods
No resources found in default namespace
```





## 레플리카셋에서 정의한 Pod 개수 유지

## : label을 대상으로 pod 관리



### #1. yml 파일로 pod 생성

```
vagrant@ubuntu:~/kubetest$ cat nginx-pod-without-rs.yml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx-pod
  labels:
    app: my-nginx-pods-label
spec:
  containers:
    - name: my-nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
```

```
vagrant@ubuntu:~/kubetest$ kubectl apply -f nginx-pod-without-rs.yml
pod/my-nginx-pod created
```

- 라벨내용도 함께 출력

```
vagrant@ubuntu:~/kubetest$ kubectl get pods --show-labels
NAME           READY   STATUS    RESTARTS   AGE   LABELS
my-nginx-pod   1/1     Running   0          35s   app=my-nginx-pods-label
```



### #2. 동일label pod 3개 생성하는 레플리카 셋 생성

- replicaset yml 파일 내용

```
vagrant@ubuntu:~/kubetest$ cat replicaset-nginx.yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-nginx-pods-label
  template:
    metadata:
      name: my-nginx-pod
      labels:
        app: my-nginx-pods-label
    spec:
      containers:
        - name: my-nginx-container
          image: nginx:latest
          ports:
            - containerPort: 80
              protocol: TCP
```

- 레플리카셋 생성

```
vagrant@ubuntu:~/kubetest$ kubectl apply -f replicaset-nginx.yml
replicaset.apps/replicaset-nginx created
```

- 동일한 label이름의 pod가 이미1개 존재 => 2개만 추가생성됨

```
vagrant@ubuntu:~/kubetest$ kubectl get pods --show-labels
NAME                     READY   STATUS    RESTARTS   AGE     LABELS
my-nginx-pod             1/1     Running   0          3m41s   app=my-nginx-pods-label
replicaset-nginx-5spld   1/1     Running   0          63s     app=my-nginx-pods-label
replicaset-nginx-shsqk   1/1     Running   0          63s     app=my-nginx-pods-label
```

### #3.  pod 수동 삭제 후 다시 조회

```
vagrant@ubuntu:~/kubetest$ kubectl delete pods my-nginx-pod
pod "my-nginx-pod" deleted
```

- 1개 pod가 삭제됨 => replicaset에서 3개 유지시켜야 하므로 1개 추가생성

```
vagrant@ubuntu:~/kubetest$ kubectl get pods
NAME                     READY   STATUS              RESTARTS   AGE
replicaset-nginx-5spld   1/1     Running             0          3m10s
replicaset-nginx-b8svm   0/1     ContainerCreating   0          6s
replicaset-nginx-shsqk   1/1     Running             0          3m10s
```



### #4. #1에서 생성한 pod 라벨변경(관리대상에서 제외)

#### 1) #1에서 생성한 pod 의 라벨을 변경

```
vagrant@ubuntu:~/kubetest$ kubectl edit pods replicaset-nginx-5spld
```

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2020-10-04T05:28:12Z"
  generateName: replicaset-nginx-
  #  labels:						-> 라벨 주석처리
  #  app: my-nginx-pods-label		-> 라벨 주석처리
  name: replicaset-nginx-5spld
```

#### 2) pod 조회 

```
vagrant@ubuntu:~/kubetest$ kubectl get pods --show-labels
NAME                     READY   STATUS              RESTARTS   AGE     LABELS
replicaset-nginx-5spld   1/1     Running             0          4m50s   <none>
replicaset-nginx-b8svm   1/1     Running             0          106s    app=my-nginx-pods-label
replicaset-nginx-c6jst   0/1     ContainerCreating   0          7s      app=my-nginx-pods-label
replicaset-nginx-shsqk   1/1     Running             0          4m50s   app=my-nginx-pods-label
vagrant@ubuntu:~/kubetest$
```

- 1)의 작업으로 해당pod는 관리대상에서 제외

  => 새로운 pod 추가해서 pod 개수 유지시킴

#### 3) 레플리카셋 삭제 => 같은 라벨 pod만 삭제

```
vagrant@ubuntu:~/kubetest$ kubectl get replicaset
NAME               DESIRED   CURRENT   READY   AGE
replicaset-nginx   3         3         3       7m18s
vagrant@ubuntu:~/kubetest$ kubectl delete replicaset replicaset-nginx
replicaset.apps "replicaset-nginx" deleted
```

```
vagrant@ubuntu:~/kubetest$ kubectl get pods --show-labels
NAME                     READY   STATUS    RESTARTS   AGE     LABELS
replicaset-nginx-5spld   1/1     Running   0          8m30s   <none>
```

#### 4) 라벨 삭제된 파드는 직접 삭제

``kubectl delete pods POD_NAME``

```
vagrant@ubuntu:~/kubetest$ kubectl delete pod replicaset-nginx-5spld
pod "replicaset-nginx-5spld" deleted

vagrant@ubuntu:~/kubetest$ kubectl get pods
No resources found in default namespace.
```







# 디플로이먼트(Deployment)

https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/

- replica set ,pod 의 배포를 관리

  => 애플리케이션 업데이트,배포를 쉽게하기 위해 만든 객체

- 롤백 가능 (**replica set 과 차이**)

  ( deployment : 버전관리 --record 로 가능)



> Replicaset    VS   deployment

- yml 파일에서의 차이 => kind부분 빼고 다 동일하다
  
- kind: Deployment        kind: ReplicaSet
  
- pod를 일정한 개수만큼 유지 : replicaset

- pod를 새로운 버전 교체/이전버전으로 교체 : deployment

  ===> 묶어서 deployment로 사용한다!!!



## deployment 명령어

| 설명                            | 명령어                                                       |
| ------------------------------- | ------------------------------------------------------------ |
| 롤백기능 추가한 deployment 생성 | kubectl apply -f YML_FILE --record                           |
| deployment 목록 확인            | kubectl get deployment                                       |
| deployment 삭제                 | kubectl delete deployment DEPLOYMENT_NAME                    |
| pod 이미지 변경                 | kubectl set image deployment DEPLOYEMENT_NAME CONTAINER_NAME=UPDATE_CONTENT --record |
| 이전 버전으로 롤백              | kubectl rollout undo deployment DEPLOYMENT_NAME --to-revision=VERSION_NUMBER |
| 상세정보 확인                   | kubectl describe deploy DEPLOYMENT_NAME                      |
| 특정버전 상세정보 확인          | kubectl describe deploy DEPLOYMENT_NAME --revision=REVISION_NUMBER |



### #1. deployment 생성 -> pod,replicaset 함께 생성

-  yml 파일로 deployment 생성

```
vagrant@ubuntu:~/kubetest$ cat deployment-nginx.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-nginx
  template:
    metadata:
      name: my-nginx-pod
      labels:
        app: my-nginx
    spec:
      containers:
        - name: my-nginx-container
          image: nginx:1.10
          ports:
            - containerPort: 80
```

```
vagrant@ubuntu:~/kubetest$ kubectl apply -f deployment-nginx.yml
deployment.apps/my-nginx-deployment created
```

- pod,replicaset 함께 생성된 것 확인

```
vagrant@ubuntu:~/kubetest$ kubectl get deployment
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
my-nginx-deployment   3/3     3            3           42s
```

```
vagrant@ubuntu:~/kubetest$ kubectl get replicaset
NAME                             DESIRED   CURRENT   READY   AGE
my-nginx-deployment-844d67df58   3         3         3       56s
```

```
vagrant@ubuntu:~/kubetest$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
my-nginx-deployment-844d67df58-l76jt   1/1     Running   0          69s
my-nginx-deployment-844d67df58-qd75s   1/1     Running   0          69s
my-nginx-deployment-844d67df58-wbtjj   1/1     Running   0          69s
```

### #2. deployment 삭제 -> pod,replicaset 함께 삭제

- deployment 제거: ``kubectl delete deployment DEPLOYMENT_NAME``

```
vagrant@ubuntu:~/kubetest$ kubectl delete deployment my-nginx-deployment
deployment.apps "my-nginx-deployment" deleted

vagrant@ubuntu:~/kubetest$ kubectl get deployment
No resources found in default namespace.
```

- pod,replicaset 함께 제거된 것 확인

```
vagrant@ubuntu:~/kubetest$ kubectl get replicasets
No resources found in default namespace.

vagrant@ubuntu:~/kubetest$ kubectl get pods
No resources found in default namespace.
```



## deployment 사용 이유: rollback 기능

### #1. --record 옵션 추가해서 deployment 생성

- deployement 생성 : ``kubectl apply -f YML_FILE --record``

```
vagrant@ubuntu:~/kubetest$ kubectl apply -f deployment-nginx.yml --record
deployment.apps/my-nginx-deployment created
```



### #2. pod의 이미지 변경 : kubectl set image

``kubectl set image deployment DEPLOYEMENT_NAME CONTAINER_NAME=UPDATE_CONTENT --record``

- yml 파일 내용

```
grant@ubuntu:~$ cat kubetest/deployment-nginx.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx-deployment   		===> deployment 이름
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-nginx
  template:
    metadata:
      name: my-nginx-pod
      labels:
        app: my-nginx
    spec:
      containers:
        - name: my-nginx-container  ===> 해당 컨테이너 이름
          image: nginx:1.10
          ports:
            - containerPort: 80
```



```
vagrant@ubuntu:~$ kubectl get deployment
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
my-nginx-deployment   3/3     3            3           66s

vagrant@ubuntu:~$ kubectl set image deployment my-nginx-deployment my-nginx-container=nginx:1.11 --record
deployment.apps/my-nginx-deployment image updated
```

```
vagrant@ubuntu:~$ kubectl get replicasets
NAME                             DESIRED   CURRENT   READY   AGE
my-nginx-deployment-7dfcb65465   3         3         3       47s
my-nginx-deployment-844d67df58   0         0         0       6m7s
```

==> 롤백 이전 버전을 제거하지 않고 남겨둔다



### #3. 리비전 정보 확인

``--record=true`` 옵션으로 deployment 변경하면 변경사항 기록 가능

```
vagrant@ubuntu:~$ kubectl rollout history deployment my-nginx-deployment
deployment.apps/my-nginx-deployment
REVISION  CHANGE-CAUSE
1         kubectl apply --filename=deployment-nginx.yml --record=true
2         kubectl set image deployment my-nginx-deployment my-nginx-container=nginx:1.11 --record=true
```



### #4. 이전 버전의 replicaset 으로 롤백

``kubectl rollout undo deployment DEPLOYMENT_NAME --to-revision=VERSION_NUMBER``

```
vagrant@ubuntu:~$ kubectl rollout undo deployment my-nginx-deployment --to-revision=1
deployment.apps/my-nginx-deployment rolled back
```

```
vagrant@ubuntu:~$ kubectl get replicasets
NAME                             DESIRED   CURRENT   READY   AGE
my-nginx-deployment-7dfcb65465   0         0         0       3m37s
my-nginx-deployment-844d67df58   3         3         3       8m57s
```

```
vagrant@ubuntu:~$ kubectl rollout history deployment my-nginx-deployment
deployment.apps/my-nginx-deployment
REVISION  CHANGE-CAUSE
2         kubectl set image deployment my-nginx-deployment my-nginx-container=nginx:1.11 --record=true
3         kubectl set image deployment my-nginx-deployment nginx=nginx:1.11 --record=true
```



### #5. deployment 상세 정보 확인

``kubectl describe deploy DEPLOYMENT_NAME``

``kubectl describe deploy DEPLOYMENT_NAME --revision=REVISION_NUMBER``

```
vagrant@ubuntu:~$ kubectl describe deploy my-nginx-deployment
Name:                   my-nginx-deployment
Namespace:              default
CreationTimestamp:      Sun, 04 Oct 2020 05:55:06 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 3
```

```
vagrant@ubuntu:~$ kubectl rollout history deployment my-nginx-deployment --revision=3
deployment.apps/my-nginx-deployment with revision #3
Pod Template:
  Labels:       app=my-nginx
        pod-template-hash=844d67df58
  Annotations:  kubernetes.io/change-cause: kubectl set image deployment my-nginx-deployment nginx=nginx:1.11 --record=true
  Containers:
   my-nginx-container:
    Image:      nginx:1.10
    Port:       80/TCP
    Host Port:  0/TCP
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>
```
