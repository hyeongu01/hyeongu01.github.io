---
layout: post
title: "[backend] 네이버 소셜 로그인 흐름"
date: 2026-03-09 11:19:00 +0900
categories: [backend, oauth]
tags: [typescript, backend, oauth]
---

---


{% include callout.html type="tip" content="이 글은 네이버 소셜 로그인을 구현하기 위해 알아야 할 흐름에 대한 정리이다." %}

### 개요

-   네이버 소셜 로그인 구현을 위한 개념 공부

---

### 네이버 어플리케이션 생성

1.  사용 API 설정

-   아래 사진과 같이 `네이버 로그인` 을 눌러 필요한 정보를 선택한다.
-   서비스에서 필요한 최소한의 정보를 선택하는게 권장된다.
-   필자는 필수로 사용자 이름과 선택으로 생년월일을 선택하였다.

| | |
|---|---|
| ![](/assets/img/2603/애플리케이션%20등록%20페이지-1.png) | ![](/assets/img/2603/애플리케이션%20등록%20페이지-1.png) |

2. 서비스, callback URL 설정

-   서비스 API 의 base url 을 적고, 네이버 요청이 왔을 때 콜백을 처리할 URL 을 지정한다.
-   일단 지정하지 않아도 되니 대충 적고 넘어가자! (네이버 소셜 로그인 흐름을 파악하고 지정)  

![](/assets/img/2603/서비스%20callback%20URL%20설정.png)

3. Client ID, Client Secret 확인

-   어플리케이션 상세정보 - 개요 - 애플리케이션 정보 에서 확인할 수 있다. 추후 인증에서 활용되니 잘 보관해 두자.
-   이 값들은 절대 유출되서는 안되는 중요한 값이기 때문에 github 에 공개로 올리는 불상사가 없도록 하자!

---

### 네이버 소셜 로그인 흐름

![](/assets/img/2603/네이버%20소셜로그인%20흐름.png)

---

### 네이버 개발자 문서

1\. 위의 사진에서 사용한 Naver API 를 상세 설명한다. [링크: 네이버 개발자 문서 - Open API](https://developers.naver.com/docs/common/openapiguide/apilist.md#%EB%84%A4%EC%9D%B4%EB%B2%84-%EB%A1%9C%EA%B7%B8%EC%9D%B8)


2\. Java, node.js 등등의 샘플 코드를 제공함. (오래된 코드 주의) [링크: 네이버 개발자 문서 - 샘플 코드](https://developers.naver.com/docs/login/web/web.md)




