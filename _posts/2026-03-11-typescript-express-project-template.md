---
layout: post
title: "[express/typescript] 프로젝트 구조 (탬플릿) 에 대한 고민 및 결정"
categories: [backend, architecture]
tags: [typescript, backend, express]
---

## 개요
> 필자는 백엔드 개발자가 된지 6개월 정도 됬다. 지금까지 여러 프로젝트를 만들면서 여러구조를 사용하거나, 기존의 구조를 그대로 따라갔다.  
> 하지만, 아직까지 내가 스타일에 맞는 구조를 사용해 본 적이 없다. 그래서 여러 구조를 사용하며 결정하게 된 프로젝트 구조를 공유하고, 왜 그렇게 정했는지 풀어볼 것이다.
> 또한, 작은 프로젝트이지만 처음부터 끝까지 경험하기 위해 API 서버를 만들어 배포까지 해보려 한다.

- 목표
  - 프로젝트 구조 설립
  - 구조에 대한 상세 설명


## 본문
> 사실 필자는 가계부 App 을 위한 API 서버를 구현하는 중이었다. 구현하는 중 계속 프로젝트 구조가 맘에 안들었고, 쓰고 고치고를 여러차례 반복한 끝에 대략적으로 구조가 잡히게 되었다. 새로운 구조로 프로젝트를 새로 생성하면서, 왜 이렇게 결정했는지, 상세 구조는 어떤 느낌인지 적어두려고 한다.  
> 예상 독자는 미래의 나 혹은 저와 같은 고민을 하고 있는 모든 사람입니다.

### 주요 개념
> 여기서 주요 계층을 정의하고자 한다.  
> (필자는 Prisma ORM 을 사용하기에 관련 계층으로 추가하였다.)

- 설정 계층
  - `config`
    - 여러 주요 설정값, 민감한 키 값을 관리한다.
- API 처리 계층
  - `server`
    - 서버를 실행시키고, 서버에 필요한 요소들을 함께 실행시킨다. (orm client, redis client 등등)
  - `application`
    - router 를 하나의 app 으로 묶는다.
    - 공통 미들웨어, 표준 에러 관리를 추가한다.
  - `router`
    - 도메인별로 API를 설정하여 관리한다.
  - `controller`
    - 요청 형식을 검증하고, service 의 값을 응답한다.
  - `service`
    - 서버의 주요 연산이 이루어진다. 
    - DB 접근이 필요한 경우 repository 를 호출한다.
  - `repository`
    - DB 직접 접근하는 계층
  - `dto`
    - 여러 계층에서 사용되는 타입의 정의
    - `zod` 등을 활용한 형식 검증도 포함한다.
- (optional) API 문서화 계층 (swagger)
  - `swagger`
- (optional) Prisma 계층
  - `prisma`

### 프로젝트 구조
> 그래서 프로젝트 구조는 어떻게 하느냐?  
> (사실은 더 여러번 구조를 바꿨는데, 맨 처음 구조와 최종 구조만 비교할 예정)

<details>
<summary>시행착오 1. 단순한 나열</summary>
<div markdown="1">




- **문제점**
  - 하나의 기능을 추가하는데 여러 파일, 폴더를 이동하며 구현해야 함.
  - 기능이 많아질수록 원하는 파일을 찾기 어려워짐 -> **기능별로 파일을 분리하자**
  - swagger 문서를 관리할 수 있으면 좋을 것 같음 -> **swagger 문서를 잘 관리할 수 있는 방법을 찾아보자**

```text
config/
  config.ts
  db.config.ts
prisma/
  generated/
  schema.prisma
src/
  routers/              // API 들의 swagger 주석 포함
    auth.router.ts
    user.router.ts
    ...
  controllers/
    auth.controller.ts
    user.controller.ts
    ...
  services/
    auth.service.ts
    user.service.ts
    ...
  repositories/
    user.repository.ts
    ...
  dtos/
    auth.dto.ts
    user.dto.ts
    ...
  libs/
    jwt.ts
    swagger.ts
    dbModel.ts
    redis.ts
    ...
  app.ts
  server.ts
```

</div>
</details>


<details>
<summary>시행착오 2. 현재 (26.03.11) 최종 구조</summary>
<div markdown="1">

- **장점**
  1. domain 별로 구분되어 있어 특정 기능 하나를 구현하고자 한다면 한 폴더 내의 파일들만 수정하면 된다.
  2. 폴더별로 구분되어있기 때문에 여러 사람이 함께 작업할 때 구분이 명확하다.
  3. 폴더의 depth 가 낮다
- **한계점**
  1. 폴더의 depth 가 낮은만큼 파일의 길이가 길어질 수 있다.

```text
config/
  config.ts
  db.config.ts
prisma/
  generated/
  schema.prisma
src/
  common/
    CustomResponse.ts   // 표준 응답, 에러 객체 선언
    jwt.ts              // jwt 관련 함수
    passport.ts         // 미들웨어 프레임워크 `passport` 관련 선언 파일
  features/
    auth/
      auth.router.ts
      auth.controller.ts
      auth.service.ts
      auth.repository.ts
      auth.dto.ts
      auth.swagger.ts
    users/
      ...
  libs/
    logger.ts           // 로깅 객체 (필자는 `winston` 사용)
    models.ts           // db Client 파일
    redis.ts            // redis client 선언 파일
    swagger.ts          // swagger spec 선언 파일 (features 아래의 .swagger.ts 를 참조한다.)
  app.ts
  server.ts
```

</div>
</details>

### 프로젝트 구성
[Github: hyeongu01/TS-Express-Prisma-Backend-Architecture](https://github.com/hyeongu01/TS-Express-Prisma-Backend-Architecture)

<details>
<summary>탬플릿 업데이트 내역</summary>
<div markdown="1">

- \[2026.03.11\] 프로젝트 생성
</div>
</details>

> 필요한 구성이 필요한 경우 주기적으로 업데이트 하여 사용할 예정입니다.  
> 제안사항이 있다면 이메일로 주시면 확인 후 반영하겠습니다. (유용한 제안들 환영입니다!)
{: .prompt-tip }

## 결론
- 지금까지 경험하며 필요해 보이는 기능을 추가하였다.
- 프로젝트 구조는 효율성도 있겠지만, 본인 취향이 더 크게 작용하는 것 같다.
- 추후 프로젝트에서 사용해 보며 주기적인 관리를 하면 좋을 것 같다.
