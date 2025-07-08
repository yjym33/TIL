# 풀 라우트 캐시(Full Route Cache)

- Next 서버측에서 빌드 타임에 특정 페이지의 렌더링 결과를 캐싱하는 기능.
- next의 페이지들은 어떤 기능을 사용하느냐에 따라 자동으로 분류됨.

- static page(정적 페이지)
  - 풀 라우트 캐시 적용.
  - 동적 페이지가 아니면 모두 정적 페이지로 설정(default).
- dynamic page(동적 페이지)

  - 특정 페이지가 접속 요청을 받을 때마다 매번 변화가 생기거나, 데이터가 달라질 경우.
  - 캐시되지 않는 Data Fetching을 사용할 경우.
  - 동적 함수(쿠키, 헤더, 쿼리스트링)을 사용하는 컴포넌트가 있는 경우.

  | 동적 함수 | 데이터 캐시 | 페이지 분류  |
  | --------- | ----------- | ------------ |
  | Yes       | No          | Dynamic Page |
  | Yes       | Yes         | Dynamic Page |
  | No        | No          | Dynamic Page |
  | No        | Yes         | Static Page  |

- Revalidate 가능.
  - 페이지를 구성하는 컴포넌트 중 하나의 요청이라도 revalidate 설정이 되어있다면 적용됨.
- Suspense
  - 쿼리 스트링 등을 다루는 컴포넌트의 경우, 빌드 중에 오류가 발생.
  - 해당 컴포넌트가 클라이언트 측에서만 설정되도록 로 감싸주어야 함.
  ```tsx
  <Suspense fallback={<div>Loading...</div>}>
    <Searchbar />
  </Suspense>
  ```
  - 사전 렌더링을 진행할 때, Suspense로 감싸인 컴포넌트는 렌더링하지 않음.
  - fallback으로 전달된 대체 ui로 대신됨.
  - Suspense로 감싸인 컴포넌트는 브라우저에 마운트가 되었을 때, 렌더링이 진행.

## 동적 경로 적용

- Static Page 로는 적용할 수 없음.
- 데이터 캐시를 적용해서 최적화 진행 가능.
- generateStaticParams 함수를 통해 정적인 파라미터 설정 가능.

```tsx
export function generateStaticParams() {
  return [{ id: "1" }, { id: "2" }, { id: "3" }];
}
```

- url 파라미터의 값을 명시할 땐 문자열로만 명시 가능.
- 위 파라미터로 생성된 페이지는 static page로 설정됨.
- 위 파라미터 범위를 벗어난 페이지(ex. id:4)는 dynamic하게 페이지가 생성됨.
  - 해당 페이지는 풀 라우트 캐시가 적용됨.
- generateStaticParams에 설정된 url 파라미터 이외는 모두 404 페이지로 설정하고 싶을 때 dynamicParams의 값을 false로 설정.(기본값 : true)

```tsx
export const dynamicParams = false;
```

# 라우트 세그먼트 옵션

- 강제로 특정 페이지의 동작을 강제로 설정.
- `dynamic` : 특정 페이지의 유형을 강제로 static, dynamic 페이지로 설정.

```tsx
export const dynamic = "auto";
```

- auto : 기본값, 아무것도 강제하지 않음.
- force-dynamic : 페이지를 강제로 Dynamic 페이지로 설정.
- force-static : 페이지를 강제로 Static 페이지로 설정.
  - 쿼리스트링 등의 값은 undefined로 설정됨.(오류 발생 가능성)
- error : 페이지를 강제로 Static 페이지로 설정.
  - 쿼리스트링 등으로 Static으로 설정하면 안되는 이유가 있다면 빌드 오류 발생.
- 특별한 이유가 없다면 권장되지는 않음.

# 클라이언트 라우터 캐시

- 브라우저에 저장되는 캐시.
- 페이지 이동을 효율적으로 진행하기 위해 페이지의 일부 데이터를 보관.
- 루트 레이아웃 등 공통되는 레이아웃에 적용.
- 새로고침을 하거나 탭을 껐다가 새로 켜는 경우엔 캐시가 사라짐.
