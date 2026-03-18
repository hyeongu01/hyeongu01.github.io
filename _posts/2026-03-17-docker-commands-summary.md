---
layout: post
title: "[docker] docker 명령어 총정리 (작성중)"
description: "Docker 사용 시 꼭 알아야 할 기본 및 실전 명령어를 정리했습니다. MySQL·Redis 컨테이너 실행 방법부터 dockerfile과 docker-compose 핵심 개념까지 한 번에 정리합니다."
categories: [backend, docker]
tags: [backend, docker]
---

## 0. 개요
- [ ] docker 를 사용하기 위한 필수 명령어 및 유용한 명령어 정리
- [ ] 자주 사용하는 mysql, redis 실행 명령어
- [ ] DockerFile 정리 링크
- [ ] docker-compose 정리 링크


## 1. Docker 생태계 요약

Docker 생태계에서 알아야 할 개념으로는 크게 4가지가 있다.
- `image`: 컨테이너의 설계도
  - 생성: `DockerFile`
  - 저장: `docker hub`
- `container`: 이미지를 실행한 결과
- `volum`: 컨테이너 외부에 데이터 저장하여 영속성 보장
- `docker-compose`: 여러 컨테이너를 함께 묶어서 실행/관리하는 도구


> 이 글에서는 위의 생태계를 사용하기 위한 docker CLI 명령어를 정리하고자 한다.
{: .prompt-info} 

## 2. Docker 명령어 리스트
### 이미지 (image)

```bash
docker pull <image>
docker build -t <name> .
docker images
docker rmi <image>
docker tag <source> <target>
docker push <image>
```
{:file="docker image"}

### 컨테이너 (container)
```bash
docker run <image>
docker run -d -p 3000:3000 --name <name> <image>
docker ps
docker ps -a
docker stop <container>
docker start <container>
docker restart <container>
docker rm <container>
docker logs <container>
docker exec -it <container> /bin/bash
```
{:file="docker container"}

### 볼륨 (volume)
```bash
docker volume create <name>
docker volume ls
docker volume inspect <name>
docker volume rm <name>
```
{:file="docker volume"}

### 네트워크
```bash
docker network ls
docker network create <name>
docker network inspect <name>
docker network connect <network> <container>
docker network disconnect <network> <container>
docker network rm <name>
```
{:file="docker network"}

### Docker Compose
```bash
docker compose up
docker compose up -d
docker compose down
docker compose ps
docker compose logs
docker compose build
```
{:file="docker compose"}

## 3. 상황별 사용 명령어
- `docker run` 의 상세 옵션
  - 실행 모드
    - `-d`: 백그라운드 실행 (-it 와 함께 사용 시 -d 가 우선된다.)
    - `-it`: 터미널 인터랙티브 모드
    - `--rm`: 컨테이너 종료 시 자동 삭제
    - `--name <name>`: 컨테이너 이름 지정
  - 네트워크
    - `-p <outPort:inPort>`: 컨테이너 내부 / 외부 포트 매핑 설정
    - `--network <name>`: 네트워크 지정
  - 볼륨
    - `-v <host>:<container>`
    - `-v <volume>:<container>`
  - 환경변수 / 설정
    - `-e KEY=value`: 환경변수 설정
    - `--env-file .env`: env 파일 로드

### docker hub 의 이미지를 사용
```bash
# 1. docker hub 의 이미지를 불러옴
docker pull mysql:latest
# (Optional) 불러온 이미지 확인
docker images
# 2. image 로 컨테이너 실행
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=1234 --name mysql-local mysql:latest
# (Optional) 생성된 컨테이너 확인
docker ps
# (Optional) mysql 접속
docker exec -it mysql-local mysql -u root -p
```

### dockerfile 사용
```bash
# 현재 경로에 dockerfile 존재
docker build -t <image name> .
# (Optional) 생성된 이미지 확인
docker images
# 이미지로 컨테이너 실행
docker run {options} --name <container name> <image name>
# (Optional) 컨테이너 로그 확인
docker logs <container name>
```


