### 이정환 강사님의 한 입 크기로 잘라먹는 Next.js강의를 들으며 정리한 내용 (추후 내용 추가되면 수정).

<br><br>

# Page Router

## 장점

- 파일 시스템 기반이 간편한 페이지 라우팅 제공.

```tsx
/pagesㄴ
	index.tsx = ~/
    search.tsx = ~/search
    book
    	index.tsx = ~/book
        [id].tsx = ~/book/1 (dynamic routes)
        [...id].tsx = ~/book/1, ~/book/1/2 (catch all segment)
        [[...id]].tsx = ~/book, ~/book/1 (optional catch all segment)
```

- 다양한 방식의 사전 렌더링 제공
  - 서버사이드 렌더링 (SSR) : 요청이 들어올 때 마다 사전 렌더링을 진행 함.
    - 요청을 받을 때마다 페이지를 생성, 항상 최신의 데이터를 보장.
    - 상황에 따라 응답속도가 느려질 수 있음.
  - 정적 사이트 생성 (SSG) : 빌드 타임에 미리 페이질르 사전 렌더링 해 둠.
    - 사전 렌더링 때 많은 시간이 소요되더라도, 사용자의 요청에는 빠르게 응답 가능.
    - 최신 데이터 반영이 어려움.
  - 증분 정적 재생성 (ISR) : SSG 페이지를 일정 시간마다 재생성.
    - 일정 시간, 혹은 어떤 행동(revalidate 요청)을 기준으로 페이지를 재생성.

## 단점

- 페이지별 레이아웃 설정이 번거로움.

```tsx
// src/pages/_app.tsx
export default function App({
  Component,
  pageProps,
}: AppProps & {
  Component: NextPageWithLayout;
}) {
  const getLayout =
    Component.getLayout ?? ((page: ReactNode) => page);

  return (
    <GlobalLayout>
      {getLayout(<Component {...pageProps} />)}
    </GlobalLayout>
  );
}

// src/pages/search/index.tsx
...
Page.getLayout = (page: ReactNode) => {
  return <SearchableLayout>{page}</SearchableLayout>;
};

```

- 데이터 페칭이 페이지 컴포넌트에 집중.

  - getServerSideProps
  - getStaticProps

    - 서버로부터 데이터 호출, 페이지 컴포넌트에 props로 전달.
    - 복잡한 페이지의 경우 컴포넌트에 props로 데이터를 내려야해서 번거로워질 수 있음.

    ```tsx
       export const getServerSideProps = async (
          context: GetServerSidePropsContext
        ) => {
          // ssr로 동작하도록 자동으로 설정(getServerSideProps)
          const q = context.query.q;
          const books = await fetchBooks(q as string);
          return {
            props: { books },
          };
          // 반드시 객체타입의 props가 들어있어야만 함.
        };

        export default function Page({
          books,
        }: InferGetStaticPropsType<typeof getServerSideProps>) {
          return (
          );
        }

        export const getStaticPaths = () => {
          return {
            paths: [
              { params: { id: '1' } },
              { params: { id: '2' } },
              { params: { id: '3' } },
            ],
            // fallback: false, // 1,2,3 을 제외한 경로는 404 경로로 지정(not found)
            // blocking : 즉시 생성(SSR) , true : 즉시 생성 + 페이지만 미리 반환
            fallback: false,
          };
        };

        export const getStaticProps = async (context: GetStaticPropsContext) => {
          // ssr로 동작하도록 자동으로 설정(getServerSideProps)
          if (!book) {
            return {
              notFound: true,
            };
          }
          return {
            props: { book },
          };
        };
    ```

- 불 필요한 컴포넌트들도 JS Bundle에 포함.
  - 하이드레이션(hydration)을 위해 JS Bundle을 전달하는 과정에 불필요한 컴포넌트도 함께 전달.
    - 불필요한 컴포넌트 : 상호작용하는 기능이 없기 때문에, 하이드레이션 될 필요가 없는 컴포넌트.
    - 하이드레이션 : html로만 렌더링 되어있는 페이지에 js 코드를 연결, 상호작용 등의 이벤트를 연결하는 과정.
