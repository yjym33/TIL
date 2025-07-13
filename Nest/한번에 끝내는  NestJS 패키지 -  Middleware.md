![alt text](/Nest/image/codeFactory.png)
출처 : https://fastcampus.co.kr/dev_online_nestjs

<br>

![alt text](/Nest/image/middleware.png)

<br>

![alt text](/Nest/image/middleware2.png)

<br>

## Middleware란?

- Middleware는 라우트 핸들러가 실행되기 전에 실행된다. Request와 Response 객체에 접근 할 수 있다.
- Middleware는 다음과 같은 작업을 할 수 있다.

- 자유롭게 코드 실행
- 요청과 응답 객체를 변경
- 요청 응답 사이클 중단
- 다음 미들웨어 실행하기

![alt text](/Nest/image/middleware3.png)

<br>

## Middleware 선언법

```tsx
import { Injectable, NestMiddleware } from "@nestjs/common";
import { Request, Response, NextFunction } from "express";

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log("Request...");
    next();
  }
}
```

<br>

## Middleware 적용법1

```tsx
import { Module, NestModule, MiddlewareConsumer } from "@nestjs/common";
import { LoggerMiddleware } from "./common/middleware/logger.middleware";
import { CatsModule } from "./cats/cats.module";

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggerMiddleware).forRoutes("cats");
  }
}

// LoggerMiddleware를 /cats 경로에 대해서만 적용
```

```tsx
import {
  Module,
  NestModule,
  RequestMethod,
  MiddlewareConsumer,
} from "@nestjs/common";
import { LoggerMiddleware } from "./common/middleware/logger.middleware";
import { CatsModule } from "./cats/cats.module";

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes({ path: "cats", method: RequestMethod.GET });
  }
}

// LoggerMiddleware를 /cats 경로의 GET 요청에 대해서만 적용
```

<br>

## Middleware 적용법2

```tsx
forRoutes({ path: "ab*cd", method: RequestMethod.ALL });

// 경로가 ab*cd 패턴에 일치하는 모든 요청에 대해 미들웨어를 적용
```

```tsx
consumer
  .apply(LoggerMiddleware)
  .exclude(
    { path: "cats", method: RequestMethod.GET },
    { path: "cats", method: RequestMethod.POST },
    "cats/(.*)"
  )
  .forRoutes(CatsController);

/*

LoggerMiddleware를 특정 경로에서 제외하고 나머지 요청에 대해 적용
.exclude()를 사용하여 GET 및 POST 메서드의 /cats 경로와 정규표현식 'cats/(.*)'로 일치하는 경로에 대해 미들웨어 적용을 제외
forRoutes(CatsController)를 통해 CatsController에 속한 나머지 경로에 대해서만 LoggerMiddleware를 적용

*/
```

```tsx
consumer.apply(cors(), helmet(), logger).forRoutes(CatsController);

// cors, helmet, logger와 같은 여러 미들웨어를 동시에 적용하는 예시
// forRoutes(CatsController)를 통해 CatsController의 모든 경로에 대해 해당 미들웨어들이 적용됨
```

<br>

## Middleware 적용법3

```tsx
const app = await NestFactory.create(AppModule);
app.use(logger);
await app.listen(3000);

// 위처럼 적용할수도 있지만, 이처럼 글로벌하게 적용할수도 있음
```
