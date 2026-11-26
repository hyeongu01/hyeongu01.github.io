---
layout: post
title: "[CI/CD] Github Actions 를 활용한 Docker 이미지 배포 자동화 (작성중)"
categories: [backend, CI/CD]
tags: [backend, docker, github_actions, CI/CD]
---

## 0. 개요
> 기존 프로젝트는 직접 EC2 인스턴스에 들어가서 Dockerfile 빌드하고, 배포를 하였다.

- 문제점
  - 소스코드 전체가 인스턴스에 들어가게 된다.
  - 운영 중 재배포를 할 때 빌드를 서버에서 하기 때문에 과부화 문제가 발생할 수 있다.
  - **이전 버전의 이미지를 관리할 수 없다.**
    - **이전 버전의 이미지, 커밋정보 등을 관리할 수 있는 방법의 필요성!**

---
## 1. 전체 아키텍쳐
![alt text](assets/img/2603/github_actions_architecture.png)

- 전체적인 동작
  1. User 가 Repository 에 push, tag 를 올린다.
  2. Github 에서 해당 프로젝트의 `/.github/workflows` 의 모든 `yml` 파일을 읽어 실행한다.
    - `yml` 문서에서 실행할 트리거, 실행할 환경, 명령어 등을 설정할 수 있다.
- `/.github/workflows/work.yml` 의 구조
  - github 인증 및 레포지토리 브랜치 코드 pull
  - GHCR 인증
  - docker build & push

---
## 2. Github Actions 기본 개념
- workflow 구조 (on, env, jobs)
  - workflow 파일을 작성하기 위해선 `.github/workflows/*.yml` 위치에 파일을 생성한다.
  - 최상위 명령어
    - name: Action 이름을 지정한다.
      ```yml
      name: "Docker Image CI"

      on: 
      # ...
      ```
      {:file=".github/workflows/docker-image.yml"}
      ![alt text](assets/img/2603/workflow_name.png)

    - on
      - 해당 파일의 작업 (jobs) 을 실행할 트리거
      - 대표적인 트리거 종류:
        - push
        - pull_request
        - tag
      - Github docs: 워크플로우를 활성화하는 이벤트 트리거
        - [https://docs.github.com/ko/actions/reference/workflows-and-actions/events-that-trigger-workflows](https://docs.github.com/ko/actions/reference/workflows-and-actions/events-that-trigger-workflows)
    - env
      ```yml
        on:
          push:
            branches: ["main"]
        
        env:
          NAME: value_1
        ```
      - workflow 에서 사용할 변수를 선언한다.
      - {% raw %}`${{ env.NAME }}`{% endraw %} 형식으로 사용 가능
    - jobs
      - 작업을 선언할 수 있다.
      - 작업을 여러개 선언하면 각 작업은 병렬적으로 실행되고, needs 를 활성화하면 지정한 job 이 끝날때까지 기다렸다가 실행한다.
- 트리거 종류
- jobs 병렬/순차 실행

---
## 3. GHCR 이미지 빌드 & 푸시


---
## 4. 컨테이너 이미지 태그 전략

### Commit hash 값 이용

### 시멘틱 버전


---
## 5. 이미지에 커밋 정보 남기기


---
## 6. 오래된 이미지 자동 정리


---
## 7. 마무리


