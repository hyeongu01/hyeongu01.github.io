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
import express from "express";
import * as controller from "./auth.controller";

const router = express.Router();

router.get("/naver/callback", controller.naverLogin);

export default router;
```
{:file="src/features/auth/auth.router.ts"}

```ts
import type {Request, Response} from "express";
import {CustomError, makeResponse} from "@common/CustomResponse";
import * as service from "./auth.service";
import {NaverLoginParamsSchema} from "@features/auth/auth.dto";

export const naverLogin = async (req: Request, res: Response) => {
    const result = NaverLoginParamsSchema.safeParse(req.query);
    if (!result.success) throw CustomError.BAD_REQUEST();

    const data = await service.naverLogin(result.data);
    return res.status(200).json(makeResponse({data}));
}
```
{: file='src/features/auth/auth.controller.ts'}

```ts
import config from "@config/config";
import {LoginParams, LoginResponse, NaverLoginParams, NaverProfile, NaverProfileSchema} from "@features/auth/auth.dto";
import {customError} from "@common/CustomResponse";
import axios from "axios";
import * as repository from "./auth.repository";
import {AuthProvider} from "@common/type";
import {User} from "@features/users/users.dto";
import { encodeJWT } from "@common/auth/jwt";
import {ulid} from "ulid";
import {createHash} from "crypto";

export const naverLogin = async (params: NaverLoginParams): Promise<LoginResponse> => {
    // code, state 분리
    const {code, state} = params;

    if (!config.naver) throw customError.SERVER_ERROR();

    // redis 에서 state 검증하는 코드 추가

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
    const result = NaverProfileSchema.safeParse(profileResult.data.response);
    if (!result.success) throw customError.SERVER_ERROR("네이버 프로필 조회 실패");
    const profile: NaverProfile = result.data;

    const loginParams: LoginParams = {
        providerId: profile.id,
        provider: AuthProvider.NAVER,
        name: profile.name,
        birthDate: (() => {
            const date = new Date(`${profile.birthyear}-${profile.birthday}`);
            return isNaN(date.getTime()) ? undefined : date
        })()
    };

    return await login(loginParams);
}

async function login(params: LoginParams): Promise<LoginResponse> {
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
```
{: file='src/features/auth/auth.service.ts'}


```ts
import * as z from "zod";

// 네이버 로그인 서비스 파라메터 스키마
export const NaverLoginParamsSchema = z.object({
    code: z.string(),
    state: z.string(),
});
// 네이버 로그인 서비스 파라메터 타입
export type NaverLoginParams = z.infer<typeof NaverLoginParamsSchema>;

// 로그인 결과 response
export type LoginResponse = {
    accessToken: string,
    refreshToken: string,
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
{: file='src/features/auth/auth.dto.ts'}




## 3. 결론
- 네이버 에플리케이션 등록 및 설정을 알아보았다.
- 네이버 소셜 로그인의 한 사이클을 구현해 보았다.
- `zod` 를 통해 형식 검증을 하였다.
- 본 프로젝트는 `Typescript`, `Prisma`, `Express` 를 사용하는 프로젝트의 탬플릿으로 설계되어있다. 
  - 추후 코드는 변경될 예정.

[![](https://gh-card.dev/repos/hyeongu01/TS-Express-Prisma-Backend-Architecture.svg?theme=dark)](https://github.com/hyeongu01/TS-Express-Prisma-Backend-Architecture)

