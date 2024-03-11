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

## 왜 무중단 배포를 해야할까?
무중단 배포란 소프트웨어 또는 웹 애플리케이션을 업데이트하거나 새로운 버전을 배포할 때,  
중단 없이 서비스를 계속 제공하는 배포 방식을 말하며 기존의 서비스가 동작하면서 새로운 업데이트가 이루어지기 때문에  
`사용자들은 전환 과정에서 서비스 중단을 경험하지 않게 된다.`

배포할때마다 서버가 멈추게 된다면 많은 문제들이 야기 될 것이다.

## 무중단 배포 종류

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

## 아키텍처 구조
![dev-blog.infra.png](..%2F..%2Fassets%2Fimg%2Fdev-blog.infra.png)

* AWS EC2 (amazon linux)
* Jenkins
* NginX

크게 위와 같이 구성되어 있습니다.


<br><br>

1. Develop Branch Push
2. GitHub 가 Jenkins에 `WebHook` 전송
3. Jenkins Build Start
4. Nginx가



## 참고
* [전략 패턴 정리](https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%EC%A0%84%EB%9E%B5Strategy-%ED%8C%A8%ED%84%B4-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EC%9E%90)

