![alt text](/Nest/image/codeFactory.png)
출처 : https://fastcampus.co.kr/dev_online_nestjs

<br>

![alt text](/Nest/image/interceptor.png)

<br>

![alt text](/Nest/image/interceptor2.png)

<br>

## Interceptor란?

### Interceptor는 NestJS에서 유일하게 요청이 들어올때 그리고 응답이 나갈때 모두 요청을 수행할수 있는 미들웨어다.

<br>

## Interceptor는 아래의 기능들을 수행할수 있다.

- 함수 실행 전/후에 추가 로직을 바인딩한다.
- 함수에서 반환된 값을 변환한다.
- 함수에서 던진 에러를 변환한다.
- 함수의 기본 기능에 추가 기능을 연장한다.
- 조건에 따라 함수의 기능을 override 한다.

<br>

### Interceptor Response 핸들링은 기본적으로 RXJS를 사용한다.

<br>

## Interceptor 구현방법

```tsx
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from "@nestjs/common";
import { Observable } from "rxjs";
import { tap } from "rxjs/operators";

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log("Before...");
    const now = Date.now();
    return next
      .handle()
      .pipe(tap(() => console.log(`After... ${Date.now() - now}ms`)));
  }
}
```
