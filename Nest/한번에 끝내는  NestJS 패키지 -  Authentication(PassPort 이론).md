![alt text](/Nest/image/codeFactory.png)
출처 : https://fastcampus.co.kr/dev_online_nestjs

<br>

![alt text](/Nest/image/Passport.png)

<br>

## Passport 사용 방식

```tsx
import { Injectable } from "@nestjs/common";
import { PassportStrategy } from "@nestjs/passport";
import { Strategy } from "passport-local";
import { AuthService } from "./auth.service";

@Injectable()
export class LocalStrategy extends PassportStrategy(Strategy) {
  constructor(private authService: AuthService) {
    super();
  }

  async validate(username: string, password: string): Promise<any> {
    const user = await this.authService.validateUser(username, password);
    if (!user) {
      throw new UnauthorizedException();
    }
    return user;
  }
}
```

## Passport 적용 방법

```tsx
@Controller('auth)
export class AuthController {
	@UseGuards(AuthGuard('local'))
	@Post('login')
	async login(@Reqeust() req) {
		return req.user;
	}
}
```

<br>

## 🧱 Passport의 핵심 개념

| 구성 요소                           | 설명                                                              |
| ----------------------------------- | ----------------------------------------------------------------- |
| **Strategy**                        | 인증 로직을 정의 (e.g., `LocalStrategy`, `JwtStrategy`)           |
| **serializeUser / deserializeUser** | 세션 사용 시 사용자 객체 직렬화/역직렬화                          |
| **Passport.authenticate()**         | 인증 로직을 실행하는 미들웨어                                     |
| **Guards (NestJS)**                 | NestJS에서는 Passport 전략을 `@UseGuards(AuthGuard(...))` 로 적용 |

<br>

> NestJS는 Passport를 내부적으로 Wrapping 하여 **전략(Strategy)**과 **AuthGuard**로 인증을 구성합니다.

<br>

## 📦 Passport 기반 인증 시스템 구현 (NestJS)

NestJS에서 Passport를 활용한 인증 시스템 구현 흐름입니다. 아래는 LocalStrategy 기반 로그인 처리 예제입니다.

<br>

### 🛠 설치

```bash
npm install @nestjs/passport passport passport-local
npm install --save-dev @types/passport-local

```

<br>

## Step 1. 사용자 서비스 구성 (users.service.ts)

사용자 데이터를 조회하는 로직을 구현합니다.

```ts
// src/users/users.service.ts

import { Injectable } from "@nestjs/common";

@Injectable()
export class UsersService {
  private readonly users = [
    { id: 1, username: "john", password: "changeme" },
    { id: 2, username: "jane", password: "guess" },
  ];

  async findOne(username: string) {
    return this.users.find((user) => user.username === username);
  }
}
```

<br>

## Step 2. LocalStrategy 정의 (local.strategy.ts)

- Passport의 Local 전략을 정의하여 사용자 인증 로직을 수행합니다.

```ts
// src/auth/local.strategy.ts

import { Strategy } from "passport-local";
import { PassportStrategy } from "@nestjs/passport";
import { Injectable, UnauthorizedException } from "@nestjs/common";
import { AuthService } from "./auth.service";

@Injectable()
export class LocalStrategy extends PassportStrategy(Strategy) {
  constructor(private authService: AuthService) {
    super(); // 기본값: username/password
  }

  async validate(username: string, password: string): Promise<any> {
    const user = await this.authService.validateUser(username, password);
    if (!user) {
      throw new UnauthorizedException("유효하지 않은 사용자입니다.");
    }
    return user; // req.user에 담김
  }
}
```

- validate()는 Passport가 자동으로 호출하며, 사용자 유효성 검사를 진행한 후 성공 시 req.user에 값을 저장합니다.

---

<br>

## 🧩 Step 3. AuthService 작성 (`auth.service.ts`)

> 사용자 인증 로직을 수행하는 서비스입니다. 비밀번호 비교, 사용자 확인 등을 처리합니다.

```ts
// src/auth/auth.service.ts

import { Injectable } from "@nestjs/common";
import { UsersService } from "../users/users.service";

@Injectable()
export class AuthService {
  constructor(private usersService: UsersService) {}

  async validateUser(username: string, password: string): Promise<any> {
    const user = await this.usersService.findOne(username);
    if (user && user.password === password) {
      const { password, ...result } = user;
      return result; // 비밀번호 제외 후 반환
    }
    return null;
  }
}
```

- 보안을 위해 실제 환경에서는 반드시 bcrypt.compare() 등을 사용해 암호를 비교해야 합니다.

<br>

## Step 4. Controller 구현 (auth.controller.ts)

인증 요청을 처리하는 컨트롤러입니다. LocalStrategy 인증이 성공되면 사용자 정보를 반환합니다.

```ts
// src/auth/auth.controller.ts

import { Controller, Post, Request, UseGuards } from "@nestjs/common";
import { AuthGuard } from "@nestjs/passport";

@Controller()
export class AuthController {
  @UseGuards(AuthGuard("local"))
  @Post("auth/login")
  async login(@Request() req): Promise<any> {
    return {
      message: "로그인 성공",
      user: req.user,
    };
  }
}
```

- @UseGuards(AuthGuard('local'))은 LocalStrategy에 등록된 validate() 함수를 실행시켜 사용자 인증을 수행합니다.

<br>

## Step 5. AuthModule 설정 (auth.module.ts)

Auth 관련 의존성을 선언하는 모듈입니다. 전략과 서비스, 컨트롤러를 함께 구성합니다.

```ts
import { Module } from "@nestjs/common";
import { PassportModule } from "@nestjs/passport";
import { AuthService } from "./auth.service";
import { LocalStrategy } from "./local.strategy";
import { AuthController } from "./auth.controller";
import { UsersModule } from "../users/users.module";

@Module({
  imports: [PassportModule, UsersModule],
  providers: [AuthService, LocalStrategy],
  controllers: [AuthController],
})
export class AuthModule {}
```

- UsersModule을 import 해야 의존성 주입(UsersService)이 동작합니다.

<br>

## users.module.ts 구성

```ts
// src/users/users.module.ts

import { Module } from "@nestjs/common";
import { UsersService } from "./users.service";

@Module({
  providers: [UsersService],
  exports: [UsersService], // 다른 모듈에서 사용 가능하게 export
})
export class UsersModule {}
```
