# 고급 라우팅 패턴

## 패러렐 라우트 Parallel Route

- 병렬 라우트.
- 하나의 화면 안에 여러 개의 페이지를 병렬로 함께 렌더링 시켜주는 패턴.
  - 페이지 : page.tsx 파일로 작성되는 컴포넌트.
  - 복잡한 구조의 UI 에 유용하게 활용됨.
- `@folderName`
  - 슬롯.
  - 병렬로 렌더링 될 페이지 컴포넌트를 보관하는 폴더.
  - url 의 경로에는 아무런 영향도 끼치지 않음.

    ![](https://velog.velcdn.com/images/938938/post/ea00df28-bb2a-4fac-ade6-a5592ac79714/image.png)

    - 이 경우 @sidebar 안의 page.tsx는, 부모 layout.tsx에 자동으로 props로 전달.
    - 이 때 props의 이름은 슬롯의 이름인 sidebar가 됨.

    ```tsx
    export default function Layout({
      children,
      sidebar,
    }: {
      children: ReactNode;
      sidebar: ReactNode;
    }) {
      return (
        <div>
          <div>{sidebar}</div>
          <div>{children}</div>
        </div>
      );
    }
    ```

  - 슬롯은 갯수 제한이 없음.
  - 슬롯 폴더 안에 경로에 적용되는 페이지 추가 가능.
    - 해당 경로에서 새로고침을 했을 때 404 페이지로 보내지는 오류 존재.
    - `default.tsx`
      - 404 페이지로 보내지는 오류를 방지하기 위해 세팅 가능.

## 인터셉팅 라우트 Intercepting Route

- 사용자가 특정 경로로 접속했을 때, 해당 요청을 가로채서 다른 페이지를 대신 렌더링하게 하는 라우팅 패턴.
  - 사용자가 동일한 경로에 접속하더라도 특정 조건을 만족하면 다른 페이지로 렌더링하게 함.
  - 조건 : 초기접속 요청이 아닐 때. 클라이언트 사이드 렌더링으로 접속했을 경우에만 발생.
    - ex. 인스타에서 게시글을 클릭했을 때는 해당 상세페이지를 별도로 띄어줌. / 해당 페이지에서 새로고침을 통해 초기 접속을 하면 게시글의 상세페이지로 완전히 이동.
- `(.)book/[id]`
  - `()` : 소괄호 뒤의 경로를 인터셉트 하라는 표시.(/book/1)
  - `(.)` : 상대 경로 의미. 동일한 경로상에 있는 book/[id]의 값을 인터셉트 하겠다는 의미.
    - `(..)` : 한 단계 위의 경로.
    - `(..)(..)` : 두 단계 위의 경로.
    - `(...)` : app 폴더 바로 밑에 있는 경로.
      ![](https://velog.velcdn.com/images/938938/post/e4b94873-5749-42ef-b513-868bdbe847e3/image.png)

### 모달 만들기

```tsx
export default function Modal({ children }: { children: ReactNode }) {
  const dialogRef = useRef<HTMLDialogElement>(null);
  const router = useRouter();

  useEffect(() => {
    if (!dialogRef.current?.open) {
      dialogRef.current?.showModal();
      dialogRef.current?.scrollTo({
        top: 0,
      });
    }
  }, []);

  return createPortal(
    <dialog
      onClose={() => router.back()}
      onClick={(e) => {
        // 모달의 배경이 클릭 되면 뒤로가기
        if ((e.target as any).nodeName === "DIALOG") {
          router.back();
        }
      }}
      className={style.modal}
      ref={dialogRef}
    >
      {children}
    </dialog>,
    document.getElementById("modal-root") as HTMLElement
    // 모달이 렌더링 될 위치.(돔 위치)
  );
}

// RootLayout.tsx
<div id="modal-root" />;
```

## 패러렐 & 인터셉팅 라우트

![](https://velog.velcdn.com/images/938938/post/e058df20-f077-4e83-a601-9d27933290f2/image.png)

- 이 경우 루트 레이아웃에 {modal} props 전달.
- 모달을 사용하지 않는 경우를 위해 default.tsx를 설정(return null)
