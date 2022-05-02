---
title: "젠킨스를 사용한 CI/CI"
date: "2022-05-02"
---

## 0. 서론
현재 개발중인 프로젝트를 배포하기 위해 CI/CD 에 대해 알아봤다.

지금 온라인중인 서비스는 `django` 로만 만들어져있다.
템플릿을 사용해 정적인 페이지를 구성했는데 프론트와 백엔드 구분을 위해 vue.js - drf 구조로 변경했다.
```
# 현재 사용중인 기술 스택 

front-end : vue.js  
back-end : django  
server : ec2  
db : rds
```

맥북(local)에서 개발 완료된 소스를 도커를 통해 배포하고 싶었다.  
그리고 back-end 는 도커를 통해 배포하고, front-end 는 S3 를 통해 배포하기로 결정했다.  
이번에는 back-end 배포를 위해 한 작업을 정리한다.

[전체 출처](https://www.dongyeon1201.kr/9026133b-31be-4b58-bcc7-49abbe893044)

## 1. CI/CD 서버 구성
EC2 인스턴스를 하나 생성했다. t3.small 타입이였고, 나머지 구성 요소들은 현재 온라인중인 서버에서 사용중이던것을 그대로 사용했다.

## 2. CI/CD 서버에 jenkins 설치
아까 구성한 서버에 jenkins 서비스를 올릴것이다. 이것 또한 도커의 컨테이너를 사용한다면 설치나 관리하기 편할것이다.

일단 EC2 인스턴스에 docker 와 docker-compose 를 설치한다
```shell
# docker 설치

$ sudo yum update
$ sudo amazon-linux-extras install -y docker
$ sudo systemctl start docker
```

```shell
# docker-compose 설치

$ sudo curl -SL https://github.com/docker/compose/releases/download/v2.4.1/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

[docker-compose 설치](https://docs.docker.com/compose/install/)

그 다음에 젠킨스 컨테이너 생성을 위한 yml 파일을 생성한다. 
```shell
$ mkdir jenkins
$ vi jenkins/docker-compose.yml
```

docker-compose 에 대해 잘 모르기 때문에 일단 전체를 그대로 사용하고 넘어갔다.
```yml
# jenkins/docker-compose.yml

version: '3'

services:
    jenkins:
        image: jenkins/jenkins:lts
        container_name: jenkins_cicd
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock
            - /jenkins:/var/jenkins_home
        ports:
            - "8080:8080"
        privileged: true
        user: root
```

이제 `docker-compose up -d` 명령어로 컨테이너를 생성하고 실행한다.  
그렇게 하면 처음에 생성한 CI/CD 서버에 젠킨스 서비스가 실행된다.

다음과 같이 브라우저에 URL 을 입력하여 젠킨스 서비스 화면을 확인해 볼 수 있다.

> \[EIP]:8080 (ex: 10.0.0.0:8080)

## 3. jenkins 설정하기 
처음 보는 화면에서 젠킨스는 비밀번호부터 물어본다. 비밀번호는 이미 생성된 상태인데, 다음 명령어에서 확인 가능하다
```shell
$ docker-compose logs
```

![generated-password](/images/ci-cd-with-jenkins/generated-password.png)

이제 로그인 전 기본적인 설정이 필요하다. 직접 설정해야 할것은 크게 없고 다음으로 넘어가기만 하면 된다.

### 3.1 플러그인 설치
Install suggested plugins 을 눌러 필요한 플러그인이 자동으로 설치되도록 한다.  
![customize-jenkins](/images/ci-cd-with-jenkins/customize-jenkins.png)  
![getting-started](/images/ci-cd-with-jenkins/getting-started.png)

### 3.2 계정 생성
설치가 완료되면 계정을 생성해야 한다.  
![create-first-admin-user](/images/ci-cd-with-jenkins/create-first-admin-user.png)  

### 3.3 URL 설정
젠킨스 url 설정하는 부분. 잘 모르는 부분이라 default 대로 설정하고 넘겼다.  
![instance-configuration](/images/ci-cd-with-jenkins/instance-configuration.png)

### 3.4 완료
완료되었다!!  
![jenkins-is-ready](/images/ci-cd-with-jenkins/jenkins-is-ready.png)

### 3.5 메인화면 확인
내가 확인한 젠킨스 메인화면
![welcome-to-jenkins](/images/ci-cd-with-jenkins/welcome-to-jenkins.png)

## 4. github 와 jenkins 연동하기
github 의 특정 branch 가 업데이트 되면 jenkins 에서 확인 가능하고 빌드 가능하도록 연동한다.

### 4.1 jenkins ssh key 설정
#### 4.1.1 키 조합 생성
아래의 명령어로 젠킨스 컨테이너의 쉘에 접속한다. 컨테이너 이름인 `jenkins_cicd` 가 아니라 컨테이너 id 로도 접속 가능하다. 
```shell
sudo docker exec -it jenkins_cicd /bin/bash
```
