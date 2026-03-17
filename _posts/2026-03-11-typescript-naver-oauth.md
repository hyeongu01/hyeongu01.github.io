---
layout: post
title: "[express/typescript] 네이버 소셜 로그인 구현"
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
  - `zod`

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

4. 생성 페이지
    ![](/assets/img/2603/네이버_애플리케이션_등록_3.png)

    > `ClientID`, `Client Secret` 은 추후 인증에서 사용되니 잘 적어두고 `Client Secret` 은 유출에 주의하자.
    {: .prompt-tip } 

5. 맴버 관리
  - 네이버 계정 주인은 등록할 필요 없지만, 추후 테스터를 구할 때 여기서 맴버를 추가해 주면 된다.
    ![](/assets/img/2603/네이버_애플리케이션_등록_4.png)
  

## 2. 백엔드 프로젝트에 적용
> 프로젝트 초기 설정은 아래 포스트 참고  
[\[express/typescript\] 프로젝트 구조 (탬플릿) 에 대한 고민 및 결정](/posts/2026/03/typescript-express-project-template/)
{: .prompt-tip } 

> 네이버 소셜로그인 흐름은 아래 포스트 참고  
[\[backend\] 네이버 소셜 로그인 흐름](/posts/2026/03/backend-naver-oauth/)
{: .prompt-tip } 


```ts
import {Router} from "express";
import * as controller from "./auth.controller";

const router = Router();

// naver login callback API
router.get("/naver/callback", controller.naverLogin);
// naver login url 생성 및 Redirect API
router.get("/naver/login", controller.naverRedirect);

export default router;
```
{:file="src/features/auth/auth.router.ts"}
{:file="src/features/auth/auth.router.ts"}

```ts
```ts
import type {Request, Response} from "express";
import {makeResponse} from "@common/CustomResponse";
import * as service from "./auth.service";
import {validateNaverLoginParams} from "@features/auth/auth.dto";
import {generateNaverLoginURL} from "@common/auth/naverLogin";

export const naverLogin = async (req: Request, res: Response) => {
    const params = req.query;
    if (!validateNaverLoginParams(params)) throw validateNaverLoginParams.errors;

    const data = await service.naverLogin(params);
    return res.status(200).json(makeResponse({data}));
};

export const naverRedirect = async (req: Request, res: Response) => {
    const naverLoginUrl = await generateNaverLoginURL();
    return res.status(302).setHeader('Location', naverLoginUrl).end();
}

// generateNaverLoginURL(): 
//   무작위 state 를 ulid 로 생성하고, redis 에 저장하여 로그인 url 을 만든다.
//   state 는 추후 callback API 에서 redis 값으로 검증한다.
```
{:file="src/features/auth/auth.router.ts"}

```ts
import config from "@config/config";
import {
    type NaverLoginParams,
    type LoginResponse,
    type LoginParams,
    validateNaverProfile,
} from "@features/auth/auth.dto";
import {customError} from "@common/CustomResponse";
import axios from "axios";
import * as repository from "./auth.repository";
import {AuthProvider} from "@common/type";
import {User} from "@features/users/users.dto";
import {encodeJWT} from "@common/auth/jwt";
import {ulid} from "ulid";
import {createHash} from "crypto";
import redis from "@libs/redis";

export const naverLogin = async (params: NaverLoginParams): Promise<LoginResponse> => {
    // code, state 분리
    const {code, state} = params;

    if (!config.naver) throw customError.SERVER_ERROR();

    // redis 에서 state 검증하는 코드 추가
    const exists = await redis.del(`naverLoginState:${state}`);
    if (!exists) throw customError.UNAUTHORIZED("state is not valid");

    // access_token 발급 (네이버 서버 jwt)
    const tokenResult = await axios.get("https://nid.naver.com/oauth2.0/token", {
        params: {
            grant_type: "authorization_code",
            client_id: config.naver.clientId,
            client_secret: config.naver.clientSecret,
            redirect_uri: config.naver.redirectUri,
            code,
            state,
        }
    });
    const {access_token, token_type} = tokenResult.data;
    if (!access_token || !token_type) throw customError.SERVER_ERROR("네이버 토큰 발급 실패");

    // profile 조회
    const profileResult = await axios.get("https://openapi.naver.com/v1/nid/me", {
        headers: {
            Authorization: `${token_type} ${access_token}`,
        }
    });
    const profile = profileResult.data.response;
    if (!validateNaverProfile(profile)) throw validateNaverProfile.errors;

    const loginParams: LoginParams = {
        providerId: profile.id,
        provider: AuthProvider.NAVER,
        name: profile.name,
        birthDate: (() => {
            const date = new Date(`${profile.birthyear}-${profile.birthday}`);
            return isNaN(date.getTime()) ? undefined : date.toISOString().split("T")[0]
        })()
    };

    return await login(loginParams);
}

export async function login(params: LoginParams): Promise<LoginResponse> {
    const result: User | null = await repository.getUserByProvider(params.provider, params.providerId);
    const user: User = result ? result : await repository.createUser(params);

    const deviceId = ulid();
    const tokens = encodeJWT(user.id, deviceId);
    const hashedRefreshToken = createHash("sha256")
        .update(tokens.refreshToken)
        .digest("hex");

    await repository.createRefreshToken(user.id, hashedRefreshToken, deviceId);
    return tokens;
}

export const NaverProfileSchema = z.object({
    id: z.string(),
    name: z.string(),
    birthday: z.string().optional(),    // MM-dd 형식
    birthyear: z.string().optional(),   // yyyy 형식
});
export type NaverProfile = z.infer<typeof NaverProfileSchema>;

export const LoginParamsSchema = z.object({
    provider: z.string().uppercase(),
    providerId: z.string(),
    name: z.string(),
    birthDate: z.date().optional(),
});
export type LoginParams = z.infer<typeof LoginParamsSchema>;


export const CreateUserParamsSchema = z.object({
    name: z.string(),
    timezone: z.string().optional(),
    currency: z.string().length(3).optional(),
    birthDate: z.date().optional(),
    provider: z.string().uppercase(),
    providerId: z.string(),
})
export type CreateUserParams = z.infer<typeof CreateUserParamsSchema>;
```
{:file="src/features/auth/auth.service.ts"}



## 3. 결론
- 네이버 소셜 로그인 흐름을 이해하고, 그 과정을 검증하였다.
- redis 를 사용한 캐싱을 통해 state 검증 기능을 추가하였다.
- ajv 검증 라이브러리를 통해 입력 검증을 하였다.

> 전체 프로젝트는 아래 repo 에서 볼 수 있습니다.   
[\[Github\]: hyeongu01/CashMan-Backend-TS](https://github.com/hyeongu01/CashMan-Backend-TS)
{: .prompt-info } 

