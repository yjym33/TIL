![alt text](/Nest/image/codeFactory.png)
출처 : https://fastcampus.co.kr/dev_online_nestjs

<br>

## Versioning

![alt text](/Nest/image/Versioning.png)

<br>

![alt text](/Nest/image/Versioning2.png)

<br>

![alt text](/Nest/image/Versioning3.png)

<br>

![alt text](/Nest/image/Versioning4.png)

<br>

![alt text](/Nest/image/Versioning5.png)

<br>

![alt text](/Nest/image/Versioning6.png)

<br>

![alt text](/Nest/image/Versioning7.png)

<br>

## 왜 Versioning(버전 관리)가 필요할까?

### 상황 설명

- Client A는 POST /movie 요청을 보내고, { name: string } 형식의 데이터를 전송합니다.
- 이후 서버는 새로운 기능을 도입하여, POST /movie 에서 { title : string, file: xxx.jpg } 형식을 요구합니다.
- client B는 이 새로운 포맷을 사용합니다.

- 문제 : 서로 다른 클라이언트가 같은 URL에 다른 데이터를 전송하고 있기 때문에 충돌이 발생 가능
  - 이 문제를 해결하기 위해 ** 버전관리(Versioning)** 를 도입하여 API의 호환성을 유지하고 클라이언트의 예상 동작을 보장합니다.

<br>

## NestJS에서 지원하는 Versioning 방식

### NestJS에서는 API 버전을 다음 방식으로 명시할 수 있습니다.

### 1. URI Versioning (가장 일반적인 방식)

```http

POST /v1/movie
POST /v2/movie

```

<br>

```ts
@Controller({ path: "movie", version: "1" })
export class MovieV1Controller {
  @Post()
  create(@Body() dto: MovieV1Dto) {}
}

@Controller({ path: "movie", version: "2" })
export class MovieV2Controller {
  @Post()
  create(@Body() dto: MovieV2Dto) {}
}
```

<br>

NestJS 설정

```ts
app.enableVersioning({
  type: VersioningType.URI,
});
```

<br>

- 장점 : 버전이 명확하게 URL에 드러나므로 클라이언트가 직관적으로 이해하기가 쉬움

<br>

---

### 2. Header Versioning

```http

POST /movie
Header: version: 1

```

<br>

NestJS 설정

```ts
app.enableVersioning({
  type: VersioningType.HEADER,
  header: "version",
});
```

<br>

---

### 3. Media Type Versioning (Accept Header 이용)

```http

POST /movie
Accept: application/json;v=2

```

<br>

NestJS 설정

```ts
app.enableVersioning({
  type: VersioningType.MEDIA_TYPE,
  key: "v",
});
```

<br>

## 🧠 언제 어떤 방식을 써야 하나요?

| 방식           | 장점                         | 단점                          | 사용 예                  |
| -------------- | ---------------------------- | ----------------------------- | ------------------------ |
| **URI**        | 명확한 URL 구분, 쉬운 디버깅 | URI 길어짐, URL 구조 변경     | 대부분의 REST API        |
| **Header**     | URL 깔끔                     | 클라이언트에서 헤더 설정 필요 | 모바일 앱, 내부 API      |
| **Media Type** | HTTP 표준에 적합             | 구현 복잡도 ↑                 | 기업 API, 고급 REST 설계 |

<br>

## NestJS 개발 시 핵심 포인트

- enableVersioning() 함수로 전역 설정을 꼭 해야 함

- @Controller({ version: '1' }) 와 같이 버전을 명시적으로 지정

- 버전 별로 DTO, 응답 형식을 나누어야 혼란 없음

- @Version('1') 데코레이터로 개별 메서드에 버전을 다르게 설정할 수도 있음
