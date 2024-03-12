---
title: Jenkins + Nginx 무중단 배포
date: 2024-02-10 23:30:00 +09:00
categories: [dev-ops, CI/CD]
tags:
  [
    Jenkins, 
    CI/CD, 
    devOps,
    Nginx,
    무중단배포
  ]
---

* * *

## ✅ 왜 무중단 배포를 해야할까?
무중단 배포란 소프트웨어 또는 웹 애플리케이션을 업데이트하거나 새로운 버전을 배포할 때,  
중단 없이 서비스를 계속 제공하는 배포 방식을 말하며 기존의 서비스가 동작하면서 새로운 업데이트가 이루어지기 때문에  
`사용자들은 전환 과정에서 서비스 중단을 경험하지 않게 된다.`

배포할때마다 서버가 멈추게 된다면 많은 문제들이 야기 될 것이다.

## ✅ 무중단 배포 종류

### 📌 로드 밸런싱
![LoadBalanceing.png](..%2F..%2Fassets%2Fimg%2FLoadBalanceing.png)

* 트래픽을 N개의 서버로 분산 시켜주는 방법  
* 서버 그룹간 트래픽을 고르게 분배하여 서버 부하 분산 가능
* 새로운 버전의 애플리케이션을 배포한 서버 그룹과 기존 버전의 애플리케이션을 동작시키는 서버 그룹을 구성하고,   
  로드 밸런서 설정을 변경하여 새로운 서버 그룹으로 트래픽을 전환하는 방식으로 무중단 배포를 할 수 있다. 

<br>

### 📌 롤링 업데이트
![rolling.png](..%2F..%2Fassets%2Fimg%2Frolling.png)

* 서버 그룹을 순차적으로 업데이트 하며 새로운 버전의 애플리케이션을 배포하는 방법
* 배포 중간 과정에서는 이전 버전과 업그레이드 버전이 공존할 수 있다.

<br>


### 📌 Blue/Green 배포
* 운영중인 구버전과 신버전의 인스턴스를 구성한 후 트래픽을 신버전쪽으로 전환하는 방식
* 시스템 자원이 두배로 필요함

<br>

### ⭐️ Blue/Green 적용 이유
1. 내 개발환경은 1개의 서버로만 구성되어 있어, 로드밸런싱을 통해 무중단 배포를 하는건 적합하지 않았다.
2. Rolling 방식은 구버전과 신버전이 공존한다는 치명적인 단점이 있어 고려 하지 않았다.
3. 1대의 서버로도 손쉽게 가능한 NginX를 이용한 Blue/Green 배포 채택

## ✅ 아키텍처 구조
[프로젝트 코드 보기](https://github.com/GangEunzzang/dev-blog)

![dev-blog.infra.png](..%2F..%2Fassets%2Fimg%2Fdev-blog.infra.png)

* AWS EC2 (amazon linux)
* Jenkins
* NginX
* Java 17/ Spring boot 2.x/ Gradle
* Spring 에서 Health Check를 위한 `Controller(/profile)` 및 `actuator 라이브러리`가 추가
* 인바운드 방화벽 세팅

크게 위와 같이 구성되어 있습니다.

## ✅ Jenkins 설정
먼저 SSH에 Jenkins가 설치되었다는 가정하에 진행 하겠습니다.  


### 📌 SSH 접속 키 등록
Jenkins에서 SSH를 접속하기 위해선 SSH Key가 필요합니다.

* **Jenkins 관리 -> 시스템 설정**

![PublishSSH.png](..%2F..%2Fassets%2Fimg%2FdevOps%2FPublishSSH.png)
- `Key`: ec2를 생성할 때 받은 ssh 접속 키인 pem 파일 내용 삽입
- `Name`: Jenkins에서만 식별할 임의의 `SSH Server`의 이름
- `HostName`: 실제로 접속할 원격 서버 IP
- `UserName`: 원격 서버의 user 이름
- `Remote Directory`: 원격 서버에 접속할때 기본 디렉토리 주소


### 📌 GitHub 계정 등록
Jenkins에서 빌드시 필요한 `GitHub Repository`의 제어 권한을 가진 계정을 등록해야 합니다.  
만약 해당 Repository가 `Public`이고 단순히 다운로드만 해도 된다면 굳이 아래의 계정등록을 진행하지 않아도 됩니다.

#### 1. GitHub Token 생성
* `GitHub` -> `Settings` -> `Developer Settings` -> `Personal access tokens` -> `Fine-grained tokens(Beta)`  -> `Gererate new token`
[토큰생성 링크 바로가기](https://github.com/settings/personal-access-tokens/new)

![GithubToken.png](..%2F..%2Fassets%2Fimg%2FdevOps%2FGithubToken.png)
- `Token name`: 토큰 이름
- `Expiration`: 토큰 만료기간
- `Resource owner`: 조직 or 개인 선택 가능
- `Repository access`: 제어 가능한 Repository 지정

Generate token 클릭하면 아래와 같이 토큰이 발급됩니다.
![makeToken.png](..%2F..%2Fassets%2Fimg%2FdevOps%2FmakeToken.png)
토큰은 다시는 볼 수 없으므로 잘 저장해야 합니다.

<br>

#### 2. Jenkins Credentials 등록
* `Jenkins 관리(settings)` -> `시스템 설정` -> `Github Servers`
![jenkinsGitCredentials.png](..%2F..%2Fassets%2Fimg%2FdevOps%2FjenkinsGitCredentials.png)

- `Kind`: Secret text 선택 
- `Secret`: GitHub에서 생성한 토큰 입력
- `ID`: 내가 지정하는 식별자값 입력

![gibhubTestSuccess.png](..%2F..%2Fassets%2Fimg%2FdevOps%2FgibhubTestSuccess.png)

`Test connection` 클릭 후 테스트 통과하면 등록 성공 !

<br>

#### 3. GitHub Webhooks 등록
* `Github Repository` -> `Settings` -> `Webhooks` 접속

![webhooks.png](..%2F..%2Fassets%2Fimg%2FdevOps%2Fwebhooks.png)
여기에 서버 IP + jenkins Port 입력해주면 된다.

그럼 GitHub에 Push Event 발생시 Webhooks 등록된 곳에 통지해준다.


<br>

### 📌 Jenkins 빌드 생성
* `새로운 Item` -> `Freestyle project` 생성

아래는 저의 Jenkins 빌드 설정 정보입니다.

![jenkinsBuildInfo.png](..%2F..%2Fassets%2Fimg%2FdevOps%2FjenkinsBuildInfo.png)

- `General`
  - GitHub Project 지정
  - 오래된 빌드 삭제(선택)
- `소스 코드 관리`
  - 등록한 Jenkins Credentials Git 지정 
- `빌드 유발`
  - master 브랜치에 push 이벤트 발생시 자동으로 빌드하도록 트리거 설정
- `Build Steps`
  - 프로젝트에서 암호화된 값 yml 치환 (Jenkins Global Variables 등록)   
  - yml 파일은 [여기](https://github.com/GangEunzzang/dev-blog/tree/master/src/main/resources)서 확인할 수 있습니다.
  - Gradle Clean + Build jar 생성
- `빌드후 조치`
  - 등록한 ssh 접속 정보 선택 
  - 만든 jar 파일 jenkins -> ssh 로 복사
  - 쉘 실행 (deploy2.sh)


<br>


## ✅ NginX 설정

### 📌 설치

EC2에 아래 명령어로 Nginx를 설치합니다.
> sudo yum install Nginx

설치가 완료되면 아래 명령어로 실행합니다.
> sudo service nginx start

Nginx가 잘 구동중인지 확인해봅니다.
> ps -ef |grep nginx

<br>

자신의 퍼블릭IPv4 주소 입력후 아래와 같은 화면이 보이면 NginX서버가 제대로 실행이 되고있음을 알 수 있습니다.  

![NginX.png](..%2F..%2Fassets%2Fimg%2FdevOps%2FnginX.png)


### 📌 설정

NginX는 `리버스 프록시`로도 사용될 수 있습니다.   
리버스 프록시는 클라이언트로부터의 요청을 서버로 전달하는 역할을 합니다.  
기본 포트인 80번으로 들어오는 요청을 어디로 포워딩할지 지정할 수 있습니다.  
이를 통해 `blue/green 배`포와 같은 배포 전략을 구현할 수 있습니다.

#### 1. service-url.inc 파일 생성
```shell
cd /etc/Nginx/conf.d  ## 없을시 폴더 생성

vi service-url.inc  
```
> set $service_port 8081;

위 값을 입력한 후 저장해줍니다. (ESC > wq! > Enter)  

위 값은 배포시 리버스 프록시가 바라볼 포트를 동적으로 지정해주기 위한 설정입니다.  
<br>

#### 2. con.f 파일 수정

![nginXConf.png](..%2F..%2Fassets%2Fimg%2FdevOps%2FnginXConf.png)

방금 입력한 파일 include 후 proxy_pass 기본 주소를 변경해줍니다.

```shell
vi /etc/Nginx/Nginx.conf

 include /etc/Nginx/conf.d/service-url.inc;

        location / {
                proxy_pass http://127.0.0.1:$service_port;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
        }
```


<br>

## ✅ Shell Script 생성

먼저 파일의 위치는 `Jenkins에서 지정한 위치`와 동일해야 합니다.  
저는 `/home/ec2-user` 에 생성하였습니다.

### 📌 deploy2.sh 

```shell
#!/bin/bash

NOW_TIME=`date +"%Y%m%d - %H:%M:%S"`

echo -e "\n\n\n\n=============== <실행중인 서버 체크 시작> ===============" >> deploy.log
echo " 실행시간 : $NOW_TIME " >> deploy.log

## 변수셋팅
BASE_PATH="/home/ec2-user"
JAR_PATH="$(ls -t ${BASE_PATH}/dev-blog-0.0.1-SNAPSHOT.jar | head -1)"

echo " 서버 정보 체크 " >> deploy.log

CURRENT_PROFILE=$(curl -s http://localhost/profile)

echo "  > 현재 서버: $CURRENT_PROFILE" >> deploy.log

## profile에 따른 셋팅
if [ $CURRENT_PROFILE == blue ]
then

  echo " green 서버로 변경합니다." >> deploy.log

  SET_PROFILE=green
  SET_PORT=8082

elif [ $CURRENT_PROFILE == green ]
then

  echo " blue 서버로 변경합니다." >> deploy.log

  SET_PROFILE=blue
  SET_PORT=8081

else

  echo " 현재 실행중이지 않습니다. : $CURRENT_PROFILE"
  echo " blue 서버로 변경합니다." >> deploy.log

  SET_PROFILE=blue
  SET_PORT=8081
fi

echo "  > Profile: $SET_PROFILE" >> deploy.log
echo "  > Port: $SET_PORT" >> deploy.log



#====================== 프로세스 종료 ===============================

echo -e "\n\n================< 프로세스 종료 시작 >===============" >> deploy.log
APP_NAME="dev-blog-0.0.1-SNAPSHOT.jar"
DEPLOY_APP=$SET_PROFILE-$APP_NAME
DEPLOY_APP_PATH=$BASE_PATH/$DEPLOY_APP

ln -fs $JAR_PATH $DEPLOY_APP_PATH

## 실행중인 프로세스 kill
if pgrep -f "$DEPLOY_APP" > /dev/null; then
  sudo pkill -f "DEPLOY_APP"
  echo " $DEPLOY_APP_PATH  프로세스 종료" >> deploy.log
  sleep 5

else
  echo " 현재 구동중인 애플리케이션이  없습니다." >> deploy.log
fi


#========================== 배포=================================

echo -e "\n\n================< 배포 시작 >===============" >> deploy.log
java -jar $DEPLOY_APP --spring.profiles.active=$SET_PROFILE > app.log 2>&1 &
echo " $SET_PROFILE 10초후 Health Check 시작 " >> deploy.log

sleep 10

for reCnt in {1..10}
do
  response=$(curl -s http://localhost:$SET_PORT/actuator/health)
  upCount=$(echo $response | grep 'UP' | wc -l)


  if [ $upCount -ge 1 ]; then
    echo " > Health Check 성공!! " >> deploy.log
    break
  else
    echo " > Health Check의 응답이 없거나 status가 UP이 아닙니다." >> deploy.log
    echo " > Health Check Response : ${response} "
  fi

  if [ $reCnt -eq 10 ]
  then
    echo "  > Health Check 실패.." >> deploy.log
    echo "  > Nginx에 연결하지 않고 종료합니다. " >> deploy.log
    exit 1
  fi

  echo "  > Health Check 연결 실패... 재시도! - $reCnt / 10" >> deploy.log
  sleep 10

done


## nginx proxy_pass change.
sleep 3
/home/ec2-user/switch.sh

```

deploy.sh 쉘 스크립트는 다음과 같은 역할을 합니다.  

1. Nginx의 `Reverse Proxy 포트` 확인
2. 포트 확인 후 연결되어 있지 않은 profile 종료
3. jar 파일 실행 (종료한 profile과 같은 포트로)
4. 서버가 정상 구동 확인 `Health Check`


<br>

### 📌 switch.sh

해당 쉘의 위치는 위에 deploy2.sh 에서 마지막 줄에 지정한 경로에 존재해야 합니다.
```shell
#!/bin/bash
echo -e "\n\n=============== <Nginx Reverse proxy 변경> ==============" >> deploy.log

CURRENT_PROFILE=$(curl -s http://localhost/profile)

echo "  > 현재 서버:  $CURRENT_PROFILE" >> deploy.log

if [ $CURRENT_PROFILE == blue ]; then
  SET_PORT=8082
  echo " 변경  서버 : Green"

elif [ $CURRENT_PROFILE == green ]; then
  SET_PORT=8081
  echo "변경 서버 : Blue"

else
  echo "  > 일치하는 Profile이 없습니다.  : $CURRENT_PROFILE"
  echo "  > 기본 Profile로  세팅합니다BLUE - (8081)"
  SET_PORT=8081
  echo " 기본 서버 : Blue"
fi

echo "  > set port : $SET_PORT" >> deploy.log
echo "set \$service_port $SET_PORT;" | sudo tee /etc/nginx/conf.d/service-url.inc

echo " Nginx reload" >> deploy.log
sudo systemctl reload nginx

echo " ********** 실행 종료 ********** " >> deploy.log

exit 0

```

switch.sh 쉘 스크립트는 다음과 같은 역할을 합니다.

1. 포트 확인 후 다른 포트로 변경
   : ex) 8081이면 8082로 변경  
   : ex) 8082이면 8081으로 변경  

2. 위에서 생성 해준 `service-url.inc` 파일의 값을 위의 값으로 업데이트 해줍니다.
3. Nginx Reload (1초 이내 실행됨)

<br><br>

## 마치며
무중단 배포는 처음해봤는데, 생각보다 어려웠던 거 같다...  (빌드 한 50번 가까이 실패....)  
근데 무엇이든 배우면 배울수록 큰 흐름은 비슷하다고 느껴진다.  
다음에 기회가 된다면 AWS Deploy를 이용한 무중단 배포도 도전해보고 싶다.


<br><br>

## 참고
* [이동욱님 블로그](https://jojoldu.tistory.com/267)
