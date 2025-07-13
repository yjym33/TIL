![alt text](/Nest/image/codeFactory.png)
출처 : https://fastcampus.co.kr/dev_online_nestjs

<br>

## Guard 이론

![alt text](/Nest/image/Guard.png)

<br>

## Guard란?

- Guard는 권한등 조건을 확인한 후 요청이 라우트 핸들러로 전달될지 말지를 결정한다.
- 이 과정을 우린 흔히 “인가(Authorization)” 라고 부르며 요청을 보낸 사용자가 요청을 수행할 자격이 있는지 확인하게 된다.
- Middleware 에서도 Guard와 같은 기능을 수행할 수 있지만 Middleware는 실행 문맥이 부족하다. 어느 한 Middleware가 실행된 다음에 어떤 기능이 실행될지 알수가 없다. 반면에 Guard는 ExceutionContext 객체에 어떤 기능이 다음으로 실행될지 정확히 알수 있다.

![alt text](/Nest/image/Guard2.png)

<br>

## Guard 선언법

- TRUE를 반환할경우 Guard를 통과한다.
- FALSE를 반환할경우 Guard를 통과 못한다.

```tsx
import { Injectable, CanActivate, ExecutionContext } from "@nestjs/common";
import { Observable } from "rxjs";

@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(
    context: ExecutionContext
  ): boolean | Promise<boolean> | Observable<boolean> {
    return true;
  }
}

// canActivate 메서드: 이 메서드는 요청이 특정 핸들러에 접근할 수 있는지 여부를 결정합니다.
// 여기서는 항상 true를 반환하여 모든 요청을 허용하고 있음
```

<br>

## Guard 적용법1

- UseGuard 데코레이터를 사용해서 사용할 Guard를 지정 할 수 있다.
- 엔드포인트에 사용하고 싶으면 메서드 위에, 클래스 전체에 사용하고 싶으면 클래스 위에 적용하면 된다.

```tsx
@Controller("cats")
@UseGuards(RolesGuard)
export class CatsController {}
```

<br>

## Guard 적용법2

- Global하게 적용 할 수 있는 방법은 두가지가 존재한다.
- 우리가 이미 사용해본 useGlobalGuars를 사용하는 방법이 제일 간단하다.
- 디펜던시 인젝션이 필요하다면 AppModule의 providers에 제공 할 수도 있다.

```tsx
const app = await NestFactory.create(AppModule);
app.useGlobalGuards(new RolesGuard());
```

```tsx
import { Module } from "@nestjs/common";
import { APP_GUARD } from "@nestjs/core";

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: RolesGuard,
    },
  ],
})
export class AppModule {}
```
