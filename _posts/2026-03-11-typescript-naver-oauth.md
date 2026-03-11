---
layout: post
title: "[express/typescript] 네이버 소셜 로그인 구현"
date: 2026-03-11 10:19:00 +0900
categories: [backend, oauth]
tags: [typescript, backend, oauth]
---

## 0. 개요
- 구현 목표
  - 네이버 애플리케이션 설정
  - 소셜로그인 구현 및 테스트
- 기술 스택
  - `typescript`
  - `express`
  - `node.js`

## 1. 네이버 애플리케이션 생성
[NaverDevelopers - 애플리케이션 생성](https://developers.naver.com/apps/#/register)

1. 애플리케이션 이름  
    `CashMan` 으로 했다. (필자는 추후 가계부 App을 만들 것이다.)

2. 사용 API
  - 네이버 로그인: 소셜 로그인을 위한 회원정보 조회를 가능하게 한다.
    ![](/assets/img/2603/네이버_애플리케이션_등록_1.png)

3. 로그인 오픈 API 서비스 환경
  - PC 웹  
       - 추후 API 생성할 때 Callback URL 을 그에 맞도록 변경하면 된다.
  ![](/assets/img/2603/네이버_애플리케이션_등록_2.png)

1. 생성 페이지
    ![](/assets/img/2603/네이버_애플리케이션_등록_3.png)

    > `ClientID`, `Client Secret` 은 추후 인증에서 사용되니 잘 적어두고 `Client Secret` 은 유출에 주의하자.
    {: .prompt-tip } 

## 2. 백엔드 프로젝트에 적용
> 프로젝트 초기 설정은 아래 포스트 참고  
[\[express/typescript\] 프로젝트 구조 (탬플릿) 에 대한 고민 및 결정](https://hyeongu01.github.io/posts/2026/03/typescript-express-project-template/)
{: .prompt-tip } 

> 네이버 소셜로그인 흐름은 아래 포스트 참고  
[\[backend\] 네이버 소셜 로그인 흐름](https://hyeongu01.github.io/posts/2026/03/backend-naver-oauth/)
{: .prompt-tip } 






