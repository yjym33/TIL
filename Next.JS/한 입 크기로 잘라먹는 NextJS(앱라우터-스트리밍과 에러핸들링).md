# 스트리밍

- 잘게 쪼게진 작은 용량의 데이터를 클라이언트로 연속적으로 전송하는 기술.
- 모든 데이터가 불러와지지 않은 상태로도 데이터에 접근 가능.
  - 사용자에게 긴 로딩 없이 좋은 경험을 제공할 수 있음.
- 스트리밍.
  - 일단 뭐라도 빠르게 보여줄 수 있음.
  - 로딩바 같은 대체 UI 표시 등.
- Dynamic page에 자주 사용됨.

## 페이지 스트리밍 적용

```tsx
// /src/app/(with-searchbar)/search/loading.tsx
const Loading = () => {
  return <div>Loading...</div>;
};
export default Loading;
```

- dynamic page가 있는 경로에 `loading.tsx` 파일 추가.
  - 해당 dynamic page에 스트리밍이 적용 됨.

### 주의점

- loading.tsx는 같은 경로에 있는 페이지 뿐 아니라 해당 경로 아래에 있는 모든 페이지에 적용됨.(layout.tsx와 비슷하게 적용)
- async 로 이루어진, 비동기로 작동하도록 설정된 페이지에만 스트리밍을 제공.
- loading.tsx는 페이지 컴포넌트에만 적용 가능.
- 브라우저에서 쿼리 스트링이 변경될 때는 적용되지 않음.ex) search?q=abc 에서 search?q=123 이 될 땐 적용되지 않음.이 경우엔 suspense를 적용해야 함.

## 컴포넌트 스트리밍 적용

```tsx
async function SearchResult() {
  const response = await fetch(``, {
    cache: "force-cache",
  });
  if (!response.ok) {
    return <div>오류가 발생했습니다.</div>;
  }
  const books: BookData[] = await response.json();
  return (
    <div>
      {books.map((book) => (
        <BookItem key={book.id} {...book} />
      ))}
    </div>
  );
}

export default function Page({
  searchParams,
}: {
  searchParams: {
    q?: string;
  };
}) {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <SearchResult q={searchParams.q || ""} />
    </Suspense>
  );
}
```

- Suspense 컴포넌트로 감싼 컴포넌트는 스트리밍 되도록 설정됨.
  - fallback 으로 대체 UI 가 설정됨.
  - loading.tsx 와 동일하게 쿼리스트링이 변경될 때는 적용되지 않음.

    - key 값을 전달, 해당 key 값이 변경될 때마다 로딩상태로 돌아가도록 설정할 수 있음.

    ```
    <Suspense key={searchParams.q || ''} fallback={<div>Loading...</div>}>
      <SearchResult q={searchParams.q || ''} />
    </Suspense>
    ```

  - 하나의 페이지 내의 여러 컴포넌트에 각각 적용 가능.
  - fallback에 스켈레톤 UI 컴포넌트 전달 가능.
    - 스켈레톤 UI : 뼈대 역할을 하는 UI. 사용자 경험을 상승시킬 수 있음.

## 에러 핸들링

- try-catch 블록.
- `error.tsx` 파일 설정.
  - layout.tsx 와 같이 적용하려는 파일과 동일 경로에 생성.
  - layout.tsx 와 같이 경로 아래의 모든 페이지에 적용됨.
  ```tsx
  "use client";

  const Error = ({ error, reset }: { error: Error; reset: () => void }) => {
    useEffect(() => {
      console.error(error.message);
    }, [error]);
    return (
      <div>
        <h3>오류가 발생했습니다.</h3>
        <button onClick={() => reset()}>다시 시도</button>
      </div>
    );
  };

  export default Error;
  ```
  - `"use client"` 적용.
    - 서버와 클라이언트 측 모두의 오류를 대응하기 위함.
  - `reset` : 에러가 발생한 페이지를 복구하기 위해 다시 컴포넌트를 렌더링 시키는 함수.
    - 브라우저 측에서, 서버에서 전달받은 데이터를 이용해 화면을 다시 렌더링 시도.
    - 서버측에서 실행되는 서버 컴포넌트를 다시 실행하지는 않음.
      - `window.location.reload()`
      - `router.refresh()` : 현재 페이지에 필요한 서버 컴포넌트들을 다시 불러옴. 비동기로 작동.
      ```
        <button
          onClick={() => {
            startTransition(() => {
              router.refresh();
              reset(); // 에러 상태를 초기화, 컴포넌트를 다시 렌더링.
            }
          }}
         >
           다시 시도
         </button>
      ```
      - `startTransition` : 하나의 콜백함수를 전달. 해당 콜백함수 안에 들어있는 UI를 변경하는 작업을 모두 일괄적으로 수행.
  - error.tsx 가 실행되면, 같은 경로에 있는 layout.tsx 까지만 실행시켜 줌.
