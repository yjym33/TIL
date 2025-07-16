![alt text](/Nest/image/codeFactory.png)
출처 : https://fastcampus.co.kr/dev_online_nestjs

<br>

![alt text](/Nest/image/Caching.png)

<br>

![alt text](/Nest/image/Caching2.png)

<br>

![alt text](/Nest/image/Caching3.png)

<br>

![alt text](/Nest/image/Caching4.png)

<br>

![alt text](/Nest/image/Caching5.png)

<br>

![alt text](/Nest/image/Caching6.png)

<br>

## 캐싱이 왜 필요한가?

### 기본 동작 (캐싱 미사용 시)

- 클라이언트가 요청할 때마다 서버가 DB에 접근해 데이터를 가져옵니다.

- 동일한 요청이라도 매번 DB에 부하가 발생합니다.

- 요청이 많아질수록 DB가 과부하되고 성능 저하 및 장애 발생 가능성이 커집니다.

<br>

### 캐싱의 핵심 목적

- 자주 요청되는 데이터를 미리 저장해두고, DB 조회 없이 빠르게 제공하여 성능을 향상시키는 것

<br>

## 캐싱의 구조 흐름 요약

| 구조 형태                     | 설명                                                    |
| ----------------------------- | ------------------------------------------------------- |
| 클라이언트 → 서버 → DB        | 모든 요청이 DB까지 도달 (부하 ↑)                        |
| 클라이언트 → 서버 → 캐시 → DB | 서버가 먼저 캐시를 조회하고, 없을 때만 DB 접근 (부하 ↓) |

---

<br>

## 캐싱의 사용처

| 사용처 유형                | 설명                                                        |
| -------------------------- | ----------------------------------------------------------- |
| 랭킹 시스템                | 인기 영화, 최신 영화 등 자주 조회되는 순위 데이터           |
| 사용자 세션 정보           | 토큰 검증, 로그인 유지, 토큰 블랙리스트 등                  |
| 변화가 적은 상세정보       | 영화 상세 정보, 공지사항 등 거의 변하지 않는 데이터         |
| 외부 API 캐싱              | 외부 API 호출 결과를 저장하여 비용과 트래픽 절감            |
| Rate Limiting / Throttling | 사용자의 요청 횟수를 캐시에 저장해 일정 횟수 이상 요청 차단 |

<br>

## 캐싱의 장점

| 항목                 | 효과                                                |
| -------------------- | --------------------------------------------------- |
| 퍼포먼스 향상        | 데이터를 빠르게 가져오므로 서버 응답 시간이 감소    |
| Scalability (확장성) | DB 부하를 줄여 더 많은 사용자를 수용 가능           |
| 비용 절감            | 외부 리소스(API 등) 사용량 감소로 비용 절감         |
| UX 개선              | 빠른 응답으로 사용자 경험 개선 (끊김 없는 인터랙션) |

---

<br>

## 캐싱의 단점

| 항목               | 설명                                                          |
| ------------------ | ------------------------------------------------------------- |
| Stale 데이터       | 캐시는 최신 데이터가 아닐 수 있어 신선도 부족                 |
| 메모리 사용 증가   | 캐시는 빠른 접근을 위해 메모리에 저장됨 → 서버 자원 사용 증가 |
| 디자인 복잡성 증가 | 캐싱 전략 설계 및 무효화 정책 등 아키텍처 복잡성 증가         |
| 보안 리스크        | 토큰, 개인정보 등 민감한 데이터 캐싱 시 주의 필요             |

## 캐싱 적용 방법 (NestJS 기준)

### 1. 기본 메모리 캐시 (in-memory)

```bash
npm install cache-manager
```

<br>

```ts
// app.module.ts
import { CacheModule } from "@nestjs/common";

@Module({
  imports: [
    CacheModule.register({
      ttl: 60, // 캐시 생명주기 (초)
      max: 100, // 최대 캐시 수
    }),
  ],
})
export class AppModule {}
```

<br>

```ts
// service.ts
import { Inject, CACHE_MANAGER } from "@nestjs/common";
import { Cache } from "cache-manager";

@Injectable()
export class MovieService {
  constructor(@Inject(CACHE_MANAGER) private cacheManager: Cache) {}

  async getPopularMovies() {
    const cached = await this.cacheManager.get("popular-movies");
    if (cached) return cached;

    const movies = await this.movieRepo.findPopular();
    await this.cacheManager.set("popular-movies", movies, 60);
    return movies;
  }
}
```

### 2. Redis 캐시 연동

```bash
npm install cache-manager-redis-store ioredis
```

<br>

```ts
import * as redisStore from "cache-manager-redis-store";

CacheModule.register({
  store: redisStore,
  host: "localhost",
  port: 6379,
});
```

---

<br>

### 3. 글로벌 캐시 설정 (인터셉터 기반)

```ts
import { CacheInterceptor, CacheTTL } from "@nestjs/common";

@UseInterceptors(CacheInterceptor)
@Controller("movies")
export class MovieController {
  @Get("popular")
  @CacheTTL(60) // 60초 TTL
  getPopularMovies() {
    return this.movieService.getPopularMovies();
  }
}
```

<br>

## 🧠 정리

| 항목            | 요약                                                                |
| --------------- | ------------------------------------------------------------------- |
| 왜 필요한가     | 성능 향상, DB 부하 감소, UX 개선                                    |
| 언제 사용하는가 | 자주 조회되거나, 변화가 적거나, 외부 API 요청이 많은 경우           |
| 장점            | 빠른 응답, 비용 절감, 트래픽 최적화                                 |
| 단점            | 신선도 관리, 메모리 부담, 설계 복잡성                               |
| 적용 방법       | `CacheModule`, `cache-manager`, Redis 연동 등 다양한 전략 사용 가능 |
