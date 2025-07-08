### 이정환 강사님의 한 입 크기로 잘라먹는 Next.js강의를 들으며 정리한 내용 (추후 내용 추가되면 수정).

<br><br>

# 데이터 페칭

```tsx
export async function Page(props) {
  const data = await fetch("...");
  return <div>...</div>;
}
```

- 클라이언트 컴포넌트에는 Async 키워드 사용 불가.
  - 브라우저에서 동작시 문제를 일으킬 수 있기 때문에 권장X.
- 기존의 getServerSideProps, getStaticProps... 등을 대체 가능.
- props 등으로 넘겨줄 필요 없이 필요한 컴포넌트에서 직접 불러올 수 있음.
- `Fetching data where it's needed` : 데이터는 필요한 곳에서 직접 불러와라.
- .env 에 환경변수 설정

```tsx
	NEXT_PUBLIC_API_SERVER_URL=http://localhost:12345
    // NEXT_PUBLIC
    // 해당 접두사가 붙지 않으면 next는 자동으로 해당 환경변수를 서버 측에서만 사용할 수 있도록
    // private로 설정.
    // 클라이언트 컴포넌트에서도 접근할 수 있도록 하려면 해당 NEXT_PUBLIC이 필요함
```

<br><br>

# 데이터 캐시

- fetch 메서드를 활용해 불러온 데이터를 Next 서버에서 보관하는 기능.
  - 불필요한 데이터의 요청 수를 줄여서 서비스의 서능을 개선할 수 있음.
  - 오직 fetch 메서드에서만 활용 가능(axios에선 사용 불가).
- 서버 가동중에는 영구적으로 보관.

```tsx
const response  await fetch(`~/api`, {cache:'force-cache'});
```

- { cache : 'force-cache' }
  - 요청의 결과를 무조건 캐싱.
  - 한 번 호출된 이후에는 다시 호출되지 않음.
- { cache : 'no-store' }
  - 데이터 페칭의 결과를 저장하지 않는 옵션.
  - 캐싱을 아예 하지 않도록 설정.
  - default 값.(v15부터 적용)
- { next : { revalidate : 3 } }
  - 특정 시간을 주기로 캐시를 업데이트.(3초)
  - 페이지 라우터의 ISR 방식과 유사함.
- { next : { tags : ['a'] } }
  - 요청이 들어왔을 때 데이터를 최신화.
  - On-Demand Revalidate

<br><br>

# 리퀘스트 메모이제이션

Request Memoization : 요청을 기억한다.

- 하나의 페이지를 이루고 있는 여러개의 컴포넌트에서 발생하는 다양한 API 요청 중 중복으로 발생하는 요청을 캐싱. 요청 최적화.
- 하나의 페이지를 렌더링 하는 동안 중복된 API 요청을 캐싱하기 위해 존재.
- 렌더링이 종료되면 모든 캐시가 소멸.(=/= 데이터 캐시)
- 중복된 API 요청을 방지하는 것에만 목적이 있음.
