![alt text](/Nest/image/codeFactory.png)
출처 : https://fastcampus.co.kr/dev_online_nestjs

<br>

## Pagination이란?

### Pagination이란 많은 데이터를 부분적으로 나눠서 불러오는 기술이다.

<br>

![alt text](/Nest/image/pagination.png)

![alt text](/Nest/image/pagination2.png)

![alt text](/Nest/image/pagination3.png)

![alt text](/Nest/image/pagination4.png)

<br>

![alt text](/Nest/image/pagination5.png)

<br>

![alt text](/Nest/image/pagination6.png)

![alt text](/Nest/image/pagination7.png)

![alt text](/Nest/image/pagination8.png)

<br>

![alt text](/Nest/image/pagination9.png)

![alt text](/Nest/image/pagination10.png)

![alt text](/Nest/image/pagination11.png)

<br>

![alt text](/Nest/image/pagination12.png)

![alt text](/Nest/image/pagination13.png)

![alt text](/Nest/image/pagination14.png)

![alt text](/Nest/image/pagination15.png)

![alt text](/Nest/image/pagination16.png)

<br>

![alt text](/Nest/image/pagination17.png)

![alt text](/Nest/image/pagination18.png)

![alt text](/Nest/image/pagination19.png)

![alt text](/Nest/image/pagination20.png)

## 추가 정리

## 1. 📌 핵심 개념

| 요소         | 설명                                                     |
| ------------ | -------------------------------------------------------- |
| `page`       | 현재 요청한 페이지 번호 (ex: 1페이지, 2페이지 등)        |
| `limit`      | 한 페이지당 출력할 항목 수 (ex: 10개씩)                  |
| `offset`     | 데이터베이스에서 건너뛸 레코드 수 (`(page - 1) * limit`) |
| `totalCount` | 전체 데이터 개수                                         |
| `totalPages` | 전체 페이지 수 (`Math.ceil(totalCount / limit)`)         |

<br>

## 2. 🧩 구현 흐름

- 클라이언트가 /items?page=2&limit=10 형태로 요청

- 서버는 해당 page, limit 값을 추출

- offset = (page - 1) \* limit 계산

- DB에서 limit, offset 기준으로 쿼리 실행

- 전체 데이터 수(totalCount)와 함께 응답

<br>

## 3. 구현예시(NestJS + Typeorm)

```ts
// dto/pagination.dto.ts
import { IsOptional, IsInt, Min } from "class-validator";

export class PaginationDto {
  @IsOptional()
  @IsInt()
  @Min(1)
  page?: number = 1;

  @IsOptional()
  @IsInt()
  @Min(1)
  limit?: number = 10;
}
```

```ts
// service.ts
async getItems(paginationDto: PaginationDto) {
  const { page, limit } = paginationDto;

  const [items, totalCount] = await this.itemRepository.findAndCount({
    skip: (page - 1) * limit,
    take: limit,
  });

  return {
    data: items,
    totalCount,
    totalPages: Math.ceil(totalCount / limit),
    currentPage: page,
  };
}
```

```ts
// controller.ts
@Get()
async getItems(@Query() paginationDto: PaginationDto) {
  return this.itemService.getItems(paginationDto);
}
```

<br>

## 페이지네이션 (page vs cursor 비교)

## 1. 페이지 기반 페이지네이션 (Page-based Pagination)

- 클라이언트가 page와 limit (또는 perPage)를 지정

- 서버는 (page - 1) \* limit 만큼 데이터를 건너뛰고, limit만큼 조회

- 대부분의 REST API가 기본적으로 채택하는 방식

<br>

```bash
GET /items?page=3&limit=10
```

```ts
skip = (page - 1) * limit;
take = limit;
```

<br>

## 장점

- 구현이 간단하고 직관적

- UI에서 "전체 페이지 수" 표시가 쉬움

- 특정 페이지로 직접 이동 가능

## 단점

- 데이터가 변경되면 페이지의 데이터가 밀릴 수 있음

  - 예: 2페이지를 보고 있는데 1페이지 데이터가 추가/삭제되면, 내용이 바뀜

- OFFSET 사용 시 성능 저하 (특히 페이지 번호가 클수록 느려짐)

<br>

## 2. 커서 기반 페이지네이션 (Cursor-based Pagination)

- 특정 레코드(예: ID, 타임스탬프)를 기준으로 다음 또는 이전 페이지를 가져옴

- 주로 정렬 가능한 고유 값(cursor) 을 기반으로 데이터 조회

- GraphQL, Twitter, Instagram 등에서 사용

```bash
GET /items?limit=10&after=1732
```

- 여기서 1732는 마지막으로 조회된 레코드의 ID 또는 createdAt 같은 정렬 기준

### 동작방식

```sql
SELECT * FROM items
WHERE id > :cursor
ORDER BY id ASC
LIMIT 10
```

## 장점

- 데이터가 실시간으로 변경돼도 결과가 밀리지 않음 → 정합성 유지

- 대규모 데이터에서 성능 우수

- 인덱스를 효과적으로 활용 가능

## 단점

- 특정 페이지로 이동이 어려움 (ex: "5페이지로 이동" 불가)

- 구현이 복잡할 수 있음 (특히 이전 페이지 구현)

- 클라이언트가 커서 값을 저장하고 관리해야 함
