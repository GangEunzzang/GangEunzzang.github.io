---
title: Docker-Compose 명령어 정리
date: 2024-08-05 10:30:00 +09:00
categories: [dev-ops, Docker]
tags:
  [
    Docker,
    컨테이너,
    도커,
    명령어,
    docker-compose,
    도커컴포즈
  ]
---

* * *

도커컴포즈를 사용하면서 자주 사용하는 명령어를 정리해보았습니다.


## ✅ 도커 명령어 정리

### 📌 도커 컴포즈 실행
  
  ```shell
  docker-compose up -d
  ```

- `-d`: 백그라운드 실행
- `-f`: 컴포즈 파일 지정
- `-p`: 프로젝트 이름 지정
- `--build`: 이미지 빌드
- `--force-recreate`: 컨테이너 강제 재생성
- `--no-deps`: 의존성 무시
- `--no-recreate`: 컨테이너 재생성 무시
- `--no-build`: 이미지 빌드 무시
- `--abort-on-container-exit`: 컨테이너 종료시 실행 중지
- `--remove-orphans`: 미사용 컨테이너 삭제
- `--scale`: 컨테이너 개수 지정
- `--timeout`: 타임아웃 지정

### 📌 도커 컴포즈 중지 및 컨테이너 삭제

  ```shell
  docker-compose down
  ```

### 📌 도커 컴포즈 로그 확인

  ```shell
  docker-compose logs 
  docker-compose logs container_name
  ```

- `-f`: 실시간 로그 확인
- `--tail`: 로그 라인 수 지정

### 📌 도커 컴포즈 컨테이너 목록 확인

  ```shell
  docker-compose ps
  ```

### 📌 도커 컴포즈 컨테이너 중지

  ```shell
  docker-compose stop
  docker-compose stop container_name
  ```

### 📌 도커 컴포즈 컨테이너 시작

  ```shell
  docker-compose start
  docker-compose start container_name
  ```

### 📌 도커 컴포즈 컨테이너 재시작

  ```shell
  docker-compose restart
  docker-compose restart container_name
  ```

### 📌 도커 컴포즈 서비스 구성 확인

  ```shell
  docker-compose config
  ```

### 📌 도커 컴포즈 포트 확인

  ```shell
docker-compose port container_name
  ```





## 마치며
* 쓸만한 명령어 알게되면 주기적으로 추가 예정 !
* [도커 공식문서](https://docs.docker.com/reference/cli/docker/compose/up/)
