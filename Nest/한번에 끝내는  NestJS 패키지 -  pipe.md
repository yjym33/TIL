![alt text](/Nest/image/codeFactory.png)
출처 : https://fastcampus.co.kr/dev_online_nestjs

## Pipe 이론

![alt text](/Nest/image/pipe1.png)

<br>

![alt text](/Nest/image/pipe2.png)

## Pipe란?

- 파이프는 Controller에 제공되는 argument들에 적용된다. 이 argument에는 @Body, @Param등 이미 우리가 사용하고 있는 입력받는 Annotation들이 모두 포함된다.
- Pipe는 argument 데이터를 가공한 후 Controller 메서드로 값들을 넘겨준다.
  - transformation : 데이터를 원하는 형태로 변형한다 (예: String에서 Integer로 변환)
  - validation : 입력된 값이 정상적인 값인지 확인한다. 아니라면 에러를 던진다.

![alt text](/Nest/image/pipe3.png)

<br>

## Global Pipe

```tsx
import { ValidationPipe } from "@nestjs/common";
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe()); // classValidator 사용하기 위해서
  await app.listen(3000);
}
bootstrap();
```

<br>

## Controller Pipe

```tsx
@Controller("movie")
@UsePipes(new ValidationPipe())
export class MovieController {
  constructor(private readonly movieService: MovieService) {}

  @Get()
  getMovies(@Query("title") title?: string) {
    return this.movieService.findAll(title);
  }
}
```

<br>

## Route Pipe

```tsx
@Controller("movie")
export class MovieController {
  constructor(private readonly movieService: MovieService) {}

  @Patch(":id")
  @UsePipes(new ValidationPipe())
  patchMovie(@param("id") id: string, @Body() body: UpdateMovieDto) {
    return this.movieService.update(+id, body);
  }
}
```

<br>

## Route Parameter Pipe (가장 많이 씀)

```tsx
@Controller("movie")
export class MovieController {
  constructor(private readonly movieService: MovieService) {}

  @Patch(":id")
  patchMovie(
    @param("id", ParseIntPipe) id: number,
    @Body() body: UpdateMovieDto
  ) {
    return this.movieService.update(id, body);
  }
}
```

<br>

![alt text](/Nest/image/pipe4.png)

<br>

![alt text](/Nest/image/pipe5.png)

<br>

### Custom Pipe

```tsx
import {
  PipeTransform,
  Injectable,
  ArgumentMetadata,
  BadRequestException,
} from "@nestjs/common";

@Injectable()
export class ParseIntPipe implements PipeTransform<string, number> {
  transform(value: string, metadata: ArgumentMetadata): number {
    const val = parseInt(value, 10);
    if (isNaN(val)) {
      throw new BadRequestException(
        `Validation failed. "${value}" is not a valid number.`
      );
    }
    return val;
  }
}
```

<br>

## 추가 개념정리 (GPT 기반)

## 1. Pipe란?

### 📌 정의

> NestJS에서 Pipe는 요청(Request) 데이터를 가공(변환, 검증, 필터링) 하기 위해 사용하는 클래스입니다.

- `@Body()`, `@Param()`, `@Query()` 등으로 받은 값에 대해:
  - **검증 (Validation)** – 값이 유효한지 검사
  - **변환 (Transformation)** – 형변환(예: 문자열 → 숫자) 수행
  - **예외 처리** – 잘못된 값이면 예외를 발생시켜 응답 제어

### 📌 작동 위치

- **컨트롤러 핸들러에 도달하기 전에** 실행됩니다.
- 요청 객체의 데이터를 미리 가공/검증한 후, 서비스로 전달되기 때문에 **보안성과 안정성을 향상**시킵니다.

---

<br>

## ✅ 2. 파이프의 주요 용도

| 용도               | 설명                              |
| ------------------ | --------------------------------- |
| 데이터 유효성 검사 | 값이 필수인지, 특정 형식인지 검사 |
| 데이터 변환        | 문자열 → 숫자, JSON 파싱 등       |
| 예외 처리          | 유효하지 않으면 예외(400 등) 응답 |

---

<br>

## ✅ 3. 내장 파이프 (NestJS 기본 제

| 파이프             | 설명                                          |
| ------------------ | --------------------------------------------- |
| `ValidationPipe`   | DTO를 기반으로 객체 유효성 검사               |
| `ParseIntPipe`     | 문자열을 정수로 변환 (ex: `"1"` → `1`)        |
| `ParseBoolPipe`    | 문자열을 boolean으로 변환 (`"true"` → `true`) |
| `DefaultValuePipe` | 값이 없을 경우 기본값 설정                    |
| `ParseUUIDPipe`    | UUID 형식 문자열 검증                         |

---

<br>

## ✅ 4. 실전 예제

### 📌 1) `ParseIntPipe` 사용 – URL 파라미터를 숫자로 변환

```tsx

@Get(':id')
getUser(@Param('id', ParseIntPipe) id: number) {
  return this.userService.getUser(id);
}
```

- `:id`는 기본적으로 문자열이므로 `ParseIntPipe`를 사용해 숫자로 변환
- 만약 변환이 실패하면 NestJS가 `400 Bad Request`로 자동 응답

---

### 📌 2) `ValidationPipe` + DTO를 통한 유효성 검사

### user.dto.ts

```tsx
import { IsString, IsInt, MinLength } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @MinLength(3)
  name: string;

  @IsInt()
  age: number;

```

### controller.ts

```tsx
import {
  Body,
  Controller,
  Post,
  UsePipes,
  ValidationPipe,
} from "@nestjs/common";
import { CreateUserDto } from "./user.dto";

@Controller("users")
export class UserController {
  @Post()
  @UsePipes(new ValidationPipe({ transform: true })) // transform을 사용하면 형변환도 가능
  create(@Body() createUserDto: CreateUserDto) {
    return createUserDto;
  }
}
```

- 유효성 검사 실패 시 `400 Bad Request` 응답
- DTO 클래스는 `class-validator` 라이브러리 기반

---

<br>

## ✅ 5. 커스텀 파이프 만들기

📌 커스텀 파이프 예제: 문자열을 대문자로 변환하는 파이프

```tsx
import { PipeTransform, Injectable, ArgumentMetadata } from "@nestjs/common";

@Injectable()
export class UppercasePipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    if (typeof value === "string") {
      return value.toUpperCase();
    }
    return value;
  }
}
```

### 사용

```tsx

@Get(':name')
getName(@Param('name', UppercasePipe) name: string) {
  return name; // 예: 'john' → 'JOHN'
}
```

---

<br>

## ✅ 6. 글로벌 파이프 설정 (모든 요청에 적용)

### main.ts

```tsx
import { ValidationPipe } from "@nestjs/common";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe({ transform: true }));
  await app.listen(3000);
}
```

---

## 🔁 요약

| 개념             | 설명                                                                     |
| ---------------- | ------------------------------------------------------------------------ |
| Pipe             | 요청 데이터를 가공(검증, 변환)하는 미들웨어 같은 구조                    |
| 적용 위치        | `@Param()`, `@Body()`, `@Query()` 등                                     |
| 사용 목적        | 입력 값 검증, 자동 형변환, 예외 응답 처리                                |
| 주요 내장 파이프 | `ValidationPipe`, `ParseIntPipe`, `ParseBoolPipe`, `DefaultValuePipe` 등 |
| 커스텀 가능      | `PipeTransform` 인터페이스로 직접 구현 가능                              |
