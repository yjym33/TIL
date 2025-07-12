![alt text](/Nest/image/codeFactory.png)
출처 : https://fastcampus.co.kr/dev_online_nestjs

<br>

![alt text](/Nest/image/Token.png)

<br>

![alt text](/Nest/image/Token2.png)

<br>

![alt text](/Nest/image/Token3.png)

<br>

![alt text](/Nest/image/Token4.png)

<br>

![alt text](/Nest/image/Token5.png)

<br>

![alt text](/Nest/image/Token6.png)

<br>

![alt text](/Nest/image/Token7.png)

<br>

![alt text](/Nest/image/Token8.png)

<br>

![alt text](/Nest/image/Token9.png)

<br>

# 🔐 NestJS 기반 토큰 인증 흐름 및 토큰 이론 정리

<br>

## ✅ 토큰이란?

토큰(Token)은 클라이언트와 서버 간의 인증 및 권한 부여를 위해 사용되는 문자열입니다. NestJS에서는 주로 JWT(Json Web Token)를 활용하여 인증 시스템을 구현합니다.

---

<br>

## 🔑 토큰의 종류

| 종류              | 설명                               |
| ----------------- | ---------------------------------- |
| **Basic Token**   | 사용자 정보를 전달할 때 사용.      |
| **Access Token**  | 프라이빗 리소스를 접근할 때 사용.  |
| **Refresh Token** | Access Token을 재발급받을 때 사용. |

---

<br>

## 🔐 JWT(Json Web Token)란?

- 무상태(stateless) 인증 방식에 사용됨
- 인증에 필요한 정보가 자체적으로 포함되어 있음
- Header, Payload, Signature의 세 부분으로 구성
- 인증(Authentication)과 인가(Authorization)에 모두 사용 가능
- 인터넷을 통해 안전하게 전송 가능

<br>

## ✅ JWT 구조 예시

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkNvZGUgRmFjdG9yeSIsImlhdCI6MTUxNjIzOTAyMn0.
cWDGmiL3oVMO-9QZdutjOUaYs4loK97Mj2H9GnC396E

<br>

## NestJS 기반 JWT 인증 구현 예시

### 설치 패키지

```bash
npm install bcrypt
npm install -D @types/bcrypt
```

<br>

### 디렉토리 구조 예시

```cpp
src/
├── auth/
│   ├── auth.controller.ts
│   ├── auth.service.ts
│   ├── jwt.strategy.ts
│   └── auth.module.ts
├── users/
│   ├── users.service.ts
│   └── users.module.ts
```

### Step 1: auth.module.ts 설정

```ts
// auth.module.ts
import { Module } from "@nestjs/common";
import { JwtModule } from "@nestjs/jwt";
import { PassportModule } from "@nestjs/passport";
import { AuthService } from "./auth.service";
import { JwtStrategy } from "./jwt.strategy";
import { AuthController } from "./auth.controller";

@Module({
  imports: [
    PassportModule,
    JwtModule.register({
      secret: process.env.JWT_SECRET || "mysecret",
      signOptions: { expiresIn: "15m" }, // Access Token 만료시간
    }),
  ],
  controllers: [AuthController],
  providers: [AuthService, JwtStrategy],
})
export class AuthModule {}
```

<br>

### Step 2: auth.service.ts – 로그인 및 토큰 생성

```ts
// auth.service.ts
import { Injectable } from "@nestjs/common";
import { JwtService } from "@nestjs/jwt";

@Injectable()
export class AuthService {
  constructor(private jwtService: JwtService) {}

  async validateUser(username: string, password: string) {
    // 실제로는 DB에서 조회 후 패스워드 비교
    if (username === "test" && password === "1234") {
      return { userId: 1, username: "test" };
    }
    return null;
  }

  async login(user: any) {
    const payload = { username: user.username, sub: user.userId };

    const accessToken = this.jwtService.sign(payload, {
      expiresIn: "15m",
    });

    const refreshToken = this.jwtService.sign(payload, {
      expiresIn: "7d",
    });

    return { accessToken, refreshToken };
  }
}
```

<br>

### Step 3: auth.controller.ts – 로그인 API

```ts
// auth.controller.ts
import { Controller, Post, Body, UnauthorizedException } from "@nestjs/common";
import { AuthService } from "./auth.service";

@Controller("auth")
export class AuthController {
  constructor(private authService: AuthService) {}

  @Post("login")
  async login(@Body() body: { username: string; password: string }) {
    const user = await this.authService.validateUser(
      body.username,
      body.password
    );

    if (!user) {
      throw new UnauthorizedException("Invalid credentials");
    }

    return this.authService.login(user);
  }
}
```

<br>

### Step 4: jwt.strategy.ts – 토큰 검증 전략

```ts
// jwt.strategy.ts
import { Injectable } from "@nestjs/common";
import { PassportStrategy } from "@nestjs/passport";
import { ExtractJwt, Strategy } from "passport-jwt";

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: process.env.JWT_SECRET || "mysecret",
    });
  }

  async validate(payload: any) {
    return { userId: payload.sub, username: payload.username };
  }
}
```

<br>

### Step 5: 인증이 필요한 라우터 보호 (AuthGuard)

```ts
// app.controller.ts 또는 any controller
import { Controller, Get, UseGuards, Request } from "@nestjs/common";
import { AuthGuard } from "@nestjs/passport";

@Controller()
export class AppController {
  @UseGuards(AuthGuard("jwt"))
  @Get("profile")
  getProfile(@Request() req) {
    return req.user; // jwt.strategy.ts에서 validate한 값
  }
}
```

<br>

### Refresh Token으로 Access Token 재발급

```ts
// auth.controller.ts 추가
@Post('refresh')
async refresh(@Body() body: { refreshToken: string }) {
  try {
    const payload = this.authService.verifyRefreshToken(body.refreshToken);
    return this.authService.login(payload); // 새 토큰 반환
  } catch {
    throw new UnauthorizedException('Invalid refresh token');
  }
}
```

```ts
// auth.service.ts에 추가
verifyRefreshToken(token: string) {
  return this.jwtService.verify(token, { secret: process.env.JWT_SECRET });
}
```

<br>

## 보안 팁 (실무 적용 시 필수)

- HTTPS 필수로 적용 (토큰 탈취 방지)

- Access Token은 짧은 만료시간, Refresh Token은 상대적으로 김

- Refresh Token은 DB 또는 HttpOnly Cookie로 안전하게 저장

- 로그아웃 시 Refresh Token 무효화 처리 필요

- 민감 정보는 절대 Payload에 담지 말 것
