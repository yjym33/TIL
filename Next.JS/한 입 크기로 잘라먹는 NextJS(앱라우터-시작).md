### 이정환 강사님의 한 입 크기로 잘라먹는 Next.js강의를 들으며 정리한 내용 (추후 내용 추가되면 수정).

<br><br>

```tsx
$ npx create-next-app@rc
npx : Node Package Executor
@rc : 출시 후보 버전 명시(release candidate) (현재(241004) v15)
```

![](https://velog.velcdn.com/images/938938/post/ee4a9b26-e872-4556-b533-69208eed2141/image.png)

<br><br>

# 페이지 라우팅 설정

- `page.tsx` 파일만 페이지로 설정 됨.(search.tsx, index.tsx 등 X)
  - ~/search : app/search/page.tsx
  - ~/book/1 : app/book/[id]/page.tsx

<br><br>

## 쿼리 스트링 사용

- `localhost:3000/search?q=1`
- 쿼리스트링이나 파라미터 등 경로와 함께 명시되는 값은 자동으로 Props로 전달.

```tsx
const SearchPage = (props) => {
  console.log(props);
  return <div>SearchPage</div>;
};
// { params: {}, searchParams: { q: '1' } }

// 혹은

const SearchPage = ({
  searchParams,
}: {
  searchParams: {
    q?: string;
  };
}) => {
  return <div>SearchPage</div>;
};

export default SearchPage;
```

<br><br>

# 레이아웃 설정

- `layout.tsx` 파일으로 레이아웃 적용.
- /app/search/layout.tsx : /search/page.tsx의 레이아웃으로 설정됨.
  - /search로 시작하는 모든 페이지에 적용. (/search/setting/page.tsx 등)
  - /search/setting/layout.tsx 가 존재할 경우, 중첩되어서 적용.

<br><br>

## 라우트 그룹

- `()` 소괄호로 감싸인 폴더.
  - 경로 상으로는 아무런 영향도 끼치지 않음.
  - 각기 다른 경로를 가지는 페이지를 하나의 폴더 안에 묶을 수 있음.
  - 일정 페이지에만 동일한 layout을 적용할 때 사용.

<br><br>

# 리액트 서버 컴포넌트

<br>

## 서버 컴포넌트란?

- 서버 측에서만 실행되는 컴포넌트.(브라우저에서는 실행되지 않음.)
  - 상호작용이 필요하지 않은 컴포넌트.(하이드레이션이 될 필요가 없는 컴포넌트.)
  - jsBundle에 포함되지 않음. 렌더링에 시간이 줄어듬.
  - 상호작용이 필요한 컴포넌트 : 클라이언트 컴포넌트.
- 서버 컴포넌트 : 서버측에서 사전 렌더링을 진행할 때 딱 한 번만 실행.
- 클라이언트 컴포넌트 : 사전 렌더링 진행 때, 하이드레이션 진행 때 총 두 번 실행.
  - 페이지의 대부분을 서버 컴포넌트로 구성할 것을 권장.
  - 기본 설정은 서버 컴포넌트, 클라이언트 컴포넌트일 경우에만 설정하면 됨.
  - `"use client"`
  - useHook은 클라이언트 컴포넌트에서만 사용 가능.
- Link는 html 고유의 기능. javascript의 기능에 해당되지 않기 때문에 클라이언트 컴포넌트를 적용해야하는 상호작용에 해당하지 않음.

<br><br>

## 주의사항

- 서버 컴포넌트에는 브라우저에서 실행될 코드가 포함되면 안됨.
  - 브라우저에서 실행되는 기느을 담고 있는 라이브러리도 호출 불가.
- 클라이언트 컴포넌트는 클라이언트에서만 실행되지 않음.
  - 사전 렌더링 때 서버에서 한 번, 하이드레이션을 위해 브라우저에서 한 번 실행.
  - 서버와 클라이언트 모두에서 실행됨.
- 클라이언트 컴포넌트에서 서버 컴포넌트를 import 할 수 없음.

  - 해당 경우 서버 컴포넌트는 클라이언트 컴포넌트로 자동으로 변경됨.
  - import가 아닌 children으로 전달되는 경우는 변경되지 않음.

  ```tsx
  <ClientComponent>
    <ServerComponent />
  </ClientComponent>
  // 이 경우 서버 컴포넌트는 서버 컴포넌트로 작동함.
  ```

- 서버 컴포넌트에서 클라이언트 컴포넌트로 직렬화 되지 않는 props는 전달 불가.
  - 직렬화(Serialization)
    - 객체, 배열, 클래스 등의 복잡한 구조의 데이터를 네트워크 상으로 전송하기 위해 아주 단순한 형태(문자열, byte)로 변환하는 것.
    - ex) const person = {name:'abc', age:10}=> {"name":"abc","age":10}
    - 함수는 직렬화가 불가능 함.

<br><br>

# 네비게이팅

- `Link`를 이용해서 이동.
- 프로그래매틱(Programmatic)한 페이지 이동.
  - 이벤트 핸들러 등을 통해서 이동하는 방법.
  - useRouter 이용 (from 'next/navigation')
    - `from 'next/router'`은 페이지 라우터 버전의 라우터.
