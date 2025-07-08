### 이정환 강사님의 한 입 크기로 잘라먹는 Next.js강의를 들으며 정리한 내용 (추후 내용 추가되면 수정).

<br><br>

# 서버 액션

- 브라우저에서 호출할 수 있는 서버에서 실행되는 비동기 함수.

```tsx
export default function Page() {
  const saveName = async (formDat:FormData)=>{
    "use server";
    const name = formData.get('name');
    await saveDB({ name: name });
  }
  return (
  )
}
```

- `"use server"`
  - 해당 함수는 next 서버에서만 실행되는 서버 액션으로 실행 됨.
  - 컴포넌트 안의 함수가 아니라 따로 해당 함수를 파일로 설정할 때는, 파일의 최상단에 위치.
- 장점.
  - 코드가 간결해짐.
  - 서버측에서만 실행되는 함수이기 때문에 보안상으로 민감하거나 중요한 데이터를 다룰 때 유용하게 활용 가능.

<br><br>

## 재검증하기.

- 리뷰 등을 작성했을 때, 해당 컴포넌트를 리렌더링 해서 작성된 리뷰를 바로 보여주기.

```tsx
try {
  const res = await fetch(`${API_URL}/review`, {
    method: "POST",
    body: JSON.stringify({}),
  });
  console.log(res.status);
  revalidatePath(`/book/${bookId}`);
} catch (err) {
  console.error(err);
  return;
}
```

- `revalidatePath`

  - 넥스트 서버측에게 페이지를 다시 생성하게 요청해서, 서버 액션의 결과를 바로 화면에 나타나게 할 수 있음.
  - 서버측에서만 호출 가능.
  - 해당 경로에 해당하는 페이지를 모두 재검증, 해당 페이지에 포함된 모든 캐시를 무효화 시킴.
    - 풀 라우트 캐시 포함, 해당 캐시는 제거되고 다시 풀 라우트 캐시를 생성하진 않음.(다음 페이지 요청이 있을 때에 다이나믹 페이지로 생성, 그 때 풀 라우트 캐시가 설정됨.)=> 무조건 최신의 데이터를 보장하기 위함.

  ```tsx
  revalidatePath(`/book/${bookId}`);
  // 1. 특정 주소의 해당하는 페이지만 재검증.
  revalidatePath(`/book/[id]`, "page");
  // 2. 특정 경로의 모든 동적 페이지를 재검증.
  revalidatePath(`/(with-searchbar)`, "layout");
  // 3. 특정 레이아웃을 갖는 모든 페이지를 재검증.
  revalidatePath(`/`, "layout");
  // 4. 모든 데이터 재검증
  ```

- `revalidateTag` : 태그를 기준으로 데이터 캐시 재검증.

  - 효율적으로 데이터 캐시를 재검증 가능.

  ```tsx
  revalidateTag(`review-${bookId}`);
  // 5. 태그를 기준으로 데이터 캐시 재검증.
  const response = await fetch(`${API_URL}`, {
    next: { tags: [`review-${bookId}`] },
  });
  ```

  <br><br>

## 클라이언트 컴포넌트에서의 서버액션

- `useActionState` : 폼 태그의 상태를 핸들링하는 여러 기능 보유.
  - state
  - formAction
  - isPending
    ```tsx
    const [state, formAction, isPending] = useActionState(
      createReviewAction, // 핸들링하려는 폼의 action 함수
      null // 폼 상태의 초기값
    );
     ...
     <form className={style.form_container} action={formAction}>
       <input name="bookId" value={bookId} hidden />
       <textarea
         disabled={isPending}
         ..
       />
       <div className={style.submit_container}>
         <button disabled={isPending} type="submit">
           {isPending ? "..." : "작성하기"}
         </button>
       </div>
     </form>
    ```
    ```tsx
    // createReviewAction.ts
    export const createReviewAction = async (_: any, formData: FormData) => {
    // 첫번째 인자로 status를 받아야 함.(사용하지 않을 땐 _ 로 대체)
    if (!content || !author) {
      return {
        status: false,
        error: '리뷰 내용과 작성자를 입력해주세요.',
      };
    }
    try {
      const res = await fetch(`${API_URL}/review`, {
      });
      ...
      return {
        status: true,
        error: '',
      };
    } catch (err) {
      return {
        status: false,
        error: `리뷰 저장에 실패했습니다. : ${err}`,
      };
    }
    };
    ```
