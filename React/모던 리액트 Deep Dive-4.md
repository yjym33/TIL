![alt text](/React/image/모던리액트DeepDive.png) <br>
출처 : https://www.yes24.com/product/goods/123161563

### 싱글 페이지 애플리케이션

- 렌더링과 라우팅에 필요한 대부분의 기능을 서버가 아닌 브라우저의 JS에 의존하는 방식
- 최초 첫 페이지에서 데이터를 모두 불러온 뒤, 페이지 전환을 위한 모든 작업이 JS와 브라우저의 history.pushState와 history.replaceState로 이뤄진다.
- 페이지를 불러온 이후 하나의 페이지에서 모든 작업을 처리한다.
- <body/> 내부에 아무런 내용이 없고, 렌더링에 필요한 모든 내용을 JS 코드로 삽입한 이후 렌더링한다.
- 단점 : 최초 로딩해야 할 JS 리소스가 커진다.
- 장점 : 한 번 로딩된 후 리소스를 받아올 일이 적기 때문에 사용자에게 좋은 UI/UX를 제공한다.

- 전통적 방식의 애플리케이션
  - 페이지 전환 발생 시 새로운 HTML 페이지를 다운로드해 파싱한다.
  - 처음부터 새로 그려야하기 때문에 페이지가 전환될 때 부자연스러울 수 있다.

<br>

### 싱글 페이지 렌더링 방식의 등장

- CommonJS, AMD(Asynchronous Module Definition)

  - JS가 서서히 다양한 작업을 수행하게 되면서 JS를 모듈화하는 방안이 논의되며 등장했다.

- Backbone.js, AngularJS. Knockout.js

  - JS 수준에서 MVx 프레임워크 구현
  - JS의 역할과 규모가 점점 커졌다.

- 이후 React, Vue, Angular 등장하며 싱글 페이지 렌더링 방식이 인기를 얻었다.

  - 간편한 개발 경험, 시대적 요구

<br>

### JAM 스택

- 기존 웹 개발은 LAMP 스택으로 구성되어 있었다.

> Linux(운영체제), Apache(서버), MySQL(데이터베이스), PHP/Python(웹 프레임워크)

- 서버 의존적인 웹 애플리케이션은 확장성이 떨어진다.(서버도 확장해야 하기 때문에 + 클라우드 개념 부족)

새로운 프레임워크의 등장으로 JAM 스택이 등장했다.

> Javascript, API, Markup

- 프론트엔드에서 마크업(HTML, CSS)을 미리 빌드해 두고 정적으로 사용자에게 제공
- 작동이 모두 사용자의 클라이언트에서 실행되므로 서버 확장성에서 자유로워졌다.
- 이후 서버 자체도 JS로 구현하는 구조가 인기를 끌게 되었다.

<br>

## 서버 사이드 렌더링

- 최초에 사용자에게 보여줄 페이지를 서버에서 렌더링해 빠르게 사용자에게 화면을 제공하는 방식
- 웹페이지 렌더링 책임 소재가 다르다.
  - 싱글 페이지 애플리케이션 : 사용자에게 제공되는 JS번들에서 렌더링을 담당한다.
  - 서버 사이드 렌더링 : 렌더링에 필요한 작업을 모두 서버에서 수행한다. -> 비교적 안정적인 렌더링 가능
- 장점 :
  - 최초 페이지 진입이 비교적 빠르다(First Contentful Paint)
    - HTTP 요청 수행, HTML 그리는 작업 등을 서버에서 수행하는 것이 더 빠르다.
    - 서버가 사용자를 감당하지 못하고, 리소스를 확보하기 어렵다면 오히려 느릴 수 있다.
  - 메타데이터 제공이 쉽다.
    - 검색 엔진은 JS를 실행하지 않는다. -> 싱글 페이지 애플리케이션은 JS에 의존하기 때문에 불리하다.
    - 서버 사이드 렌더링은 검색 엔진에 제공할 정보를 서버에서 가공해 HTML 응답으로 제공할 수 있다.
  - 누적 레이아웃 이동(Cumulative Layout Shift)이 적다.
    - 누적 레이아웃 이동 : 페이지를 보여준 이후에 뒤늦게 HTML이 추가/삭제되어 화면이 덜컥이는 것
    - 싱글 페이지 애플리케이션 : 페이지 콘텐츠가 API에 의존하고, 응답 속도가 제각각이기 때문에 누적 레이아웃 이동이 발생할 수 있다.
    - SSR : API 요청이 완전히 완료된 이후에 완성된 페이지를 제공한다.
      - 하지만 API 요청이 모두 완료될 때까지 기다려야 하므로 최초 페이지 다운로드 속도가 느려질 수 있다.
    - 사용자의 디바이스 성능에 비교적 자유롭다.
      - 부담을 서버에 나눌 수 있기 때문
    - 보안에 좀 더 안전하다.
      - 애플리케이션의 모든 활동이 브라우저에 노출되는 위험이 있다.
      - SSR : 인증 또는 민감한 작업을 서버에서 수행하고 그 결과만 브라우저에 제공해 위협을 피할수 있따.

<br>

- 단점 :
  - 소스 코드를 작성 할 때 항상 서버를 고려해야 한다.
    - window에 대한 접근을 최소화해야 한다.
    - 라이브러리가 서버에 대한 고려가 되어있지 않다면 클라이언트에서만 실행되도록 해야한다.
      - 너무 많아지면 SSR의 장점을 잃는다.
    - 적절한 서버가 구축되어 있어야 한다.
    - 서비스 지연에 따른 문제가 있다.
      - 지연이 일어나면 렌더링 작업이 끝나기까지 사용자에게 어떠한 정보도 제공할 수 없다.
      - 병목 현상이 심해진다면 사용자 경험을 해치게 된다.

<br>

## SPA와 SSR을 모두 알아야 하는 이유

- 가장 뛰어난 SPA는 가장 뛰어난 SSR 애플리케이션보다 낫다.
- 평균적인 SPA는 평균적인 SSR 애플리키에션보다 느리다.
  - 최근에는 멀티 페이지 애플리케이션에서 발생하는 라우팅 문제를 해결하는 API가 브라우저에 추가되고 있다.
    - 페인트 홀딩(Paint Holding) : 같은 출처(origin)에서 라우팅이 일어날 경우 이전 페이지 모습을 잠깐 보여주는 기법
    - back forward cache(bfcache) : 브라우저 앞으로 가기, 뒤로가기 실행 시 캐시된 페이지를 보여주는 기법
    - Shared Element Transitions : 페이지 라우팅이 일어났을 때 두 페이지에 동일 요소가 있다면 해당 콘텍스트를 유지해 부드럽게 전환되게 하는 기법
- 현대의 SSR
  - 최초 웹사이트 진입 시 : SSR 방식으로 서버에서 완성된 HTML을 제공받는다.
  - 라우팅 : 서버에서 내려받은 JS를 바탕으로 SPA처럼 작동한다.
- 두 방식을 모두 이해해야 제대로 된 웹서비스를 구축할 수 있다.

<br>

## 서버 사이드 렌더링을 위한 리액트 API 살펴보기

리액트는 리액트 애플리케이션을 서버에서 렌더링할 수 있는 API도 제공한다.

- Node.js와 같은 서버 환경에서만 실행할 수 있다.
- Window 환경에서 실행 시 에러가 발생할 수 있다.

<br>

### renderToString

- 리액트 컴포넌트를 렌더링해 HTML 문자열로 변환하는 함수
- SSR을 구현하는 데 가장 기초적인 API

```jsx
const result = ReactDOMServer.renderToString(
  React.createElement("div", { id: "root" }, <SampleComponent />)
);
```

- useEffect같은 훅과 handleClick과 같은 이벤트 핸들러는 결과물에 포함되지 않는다.
  - 빠르게 브라우저가 렌더링할 수 있는 HTML을 제공하는 데 목적이 있다.
  - 클라이언트에서 실행되는 JS 코드를 포함시키거나 렌더링해주는 역할까지 하지는 않는다.
- 리액트의 SSR은 단순히 '최초 HTML 페이지를 빠르게 그려주는 데' 목적이 있다.

  - 인터랙션을 위해서는 별도의 JS 코드를 모두 다운로드, 파싱, 실행해야 한다.

- div#root의 속성 data-reactroot
  - 리액트 컴포넌트의 루트 엘리먼트가 무엇인지 식별하는 역할을 한다.
  - 이후 JS를 실행하기 위한 hydrate 함수에서 루트를 식별하는 기준점이 된다.

<br>

### renderToStaticMarkup

- 리액트 컴포넌트를 기준으로 HTML 문자열을 만든다.
- renderToString이 루트 요소에 추가한 data-reactroot 등 리액트에서만 사용하는 추가적인 DOM 속성을 만들지 않는다.
- HTML의 크기를 약간이라도 줄일 수 있는 장점이 있다.

```jsx
const result = ReactDOMServer.renderToStaticMarkup(
  React.createElement("div", { id: "root" }, <SampleComponent />)
);
```

- renderToStaticMarkup의 결과물을 기반으로 hydrate를 수행하면 서버와 클라이언트 내용이 맞지 않는다는 에러가 발생한다.
  - renderToStaticMarkup은 순수한 HTML만 반환하기 때문
  - 이벤트 리스너가 필요 없는 완전히 순수한 HTML을 만들 때만 사용한다.

<br>

### renderToNodeStream

- renderToString과 결과물이 완전히 동일하지만 차이점이 있다.
  - renderToString, renderToStaticMarkup은 브라우저에서도 실행할 수 있지만, renderToNodeStream은 브라우저에서 사용할 수 없다. - 완전히 Node.js 환경에 의존한다.

> ReactDOMServer.renderToNodeStream() : The streaming API is not available in the browser. Use ReactDOMServer.renderToString() instead.

- 결과물의 타입이 다르다.

  - renderToString : string인 무자열
  - renderToNodeStream : Node.js의 ReadableStream(urf-8로 인코딩된 바이트 스트림, 서버환경에서만 사용할 수 있다.)

- 그렇다면 어디서 필요할까?
  - 스트림 : 큰 데이터를 다룰 때 데이터를 청크로 분할해 조금씩 가져오는 방식
  - renderToString으로 생성하는 HTML의 크기가 매우 크면 Node.js가 실행되는 서버에 부담이 될 수 있다.
  - 대신 스트림을 활용하면 데이터를 청크 단위로 분리해 순차적으로 처리해 부담을 덜 수 있다.
  - 리액트 SSR 프레임워크는 모두 renderToNodeStream을 채택하고 있다.

<br>

### renderToStaticNodeStream

- renderToNodeStream와 제공하는 결과물을 동일하나, 리액트 자바스크립트에 필요한 리액트 속성이 제공되지 않는다.
- hydrate할 필요 없는 순수 HTML 결과물이 필요할 때 사용하는 메서드

<br>

## 서버 사이드 렌더링을 위한 리액트 API 살펴보기

### hydrate

- renderToString과 renderToNodeStream으로 생성된 HTML 콘텐츠에 JS 핸들러나 이벤트를 붙이는 역할

<br>

### render

```jsx
import * as ReactDOM from "react-dom";
import App from "./App";

const rootElement = document.getElementById("root");

ReactDOM.render(<App />, rootElement);
```

- HTML 요소에 해당 컴포넌트를 렌더링하고, 여기에 이벤트 핸들러를 붙이는 작업까지 모두 한번에 수행

<br>

### hydrate

```jsx
import App from "./App";

// containerId를 가리키는 element는 서버에서 렌더링된 HTML의 특정 위치를 의미한다.
const element = document.getElementById(containerId);

// 해당 element를 기준으로 리액트 이벤트 핸들러를 붙인다.
ReactDOM.hydrate(<App />, element);
```

- 이미 렌더링된 HTML이 있다는 가정하에 작업이 수행되고, 렌더링된 HTML을 기준으로 이벤트를 붙이는 작업만 실행한다.
- 리액트 관련 정보가 없는 순수한 HTML 정보를 넘겨주면 에러가 발생한다.
  - 정상적으로 웹페이지는 만들어짐
  - hydrate 작업이 렌더링을 한 번 수행하면서 hydrate가 수행한 렌더링 결과물 HTML과 인수로 넘겨받은 HTML을 비교하는 작업을 수행하기 때문
  - 불일치가 발생하면 hydrate가 렌더링한 기준으로 웹페이지를 그린다.
  - 서버와 클라이언트에서 두 번 렌더링 하는 것이므로 바람직하지 않다.
- 불가피하게 불일치가 발생할 수 있는 경우
  - 해당 요소에 suppressHydrationWarning를 추가해 경고를 끌 수 있다.

```jsx
<div suppressHydrationWarning>{new Date().getTime()}</div>
```

<br>

## 서버 사이드 렌더링 예제 프로젝트

### index.jsx

```jsx
import { hydrate } from "react-dom";

import App from "./components/App";
import { fetchTodo } from "./fetch";

async function main() {
  const result = await fetchTodo();

  const app = <App todos={result} />;
  const el = document.getElementById("root");

  hydrate(app, el);
}

main();
```

- 어플리케이션의 시작점
- 서버로부터 받은 HTML을 hydrate를 통해 완성도니 웹 어플리케이션으로 만든다.
- 서버에서 완성한 HTML과 hydration 대상이 되는 HTML 결과물이 동일한지 비교하는 작업 -> 데이터를 한 번 더 조회한다.

<br>

### index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>SSR Example</title>
  </head>
  <body>
    __placeholder__
    <script src="https://unpkg.com/react@17.0.2/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@17.0.2/umd/react-dom.development.js"></script>
    <script src="/browser.js"></script>
  </body>
</html>
```

- 서버 사이드 렌더링을 수행할 때 기본이 되는 HTML 템플릿
- **placeholder**
  - 서버에서 리액트 컴포넌트를 기반으로 만든 HTML 코드를 삽입하는 자리
- unpkg
  - npm 라이브러리를 CDN으로 제공하는 웹 서비스
  - 번들링하지 않기 위해 간단히 처리
- browser.js
  - 클라이언트 리액트 애플리케이션 코드를 번들링했을 때 제공되는 리액트 JS 코드
  - 내부에 App.tsx, Todo.tsx, fetch 등 JS 코드가 포함돼 있다.
  - placeholder에 먼저 리액트에서 만든 HTML 삽입 -> 이후 browser.js 코드가 실행되면서 JS 이벤트 핸들러가 붙는다.

<br>

### server.ts

```ts
function main() {
  createServer(serverHandler).listen(PORT, () => {
    console.log(`Server has been started ${PORT}...`);
  });
}
```

- createServer

  - http 모듈을 이용해 간단한 서버를 만들 수 있는 Node.js 기본 라이브러리
  - 사용자의 요청 주소에 따라 어떠한 리소스를 내려줄 지 결정하는 역할을 한다.
  - 서버 사이드 렌더링을 위해 리액트 트리를 만드는 역할도 담당한다.

<br>

```jsx
async function serverHandler(req: IncomingMessage, res: ServerResponse) {
  const { url } = req;

  switch (url) {
    //...

    default: {
      res.statusCode = 404;
      res.end("404 Not Founc");
    }
  }
}
```

- serverHandler
  - createServer로 넘겨주는 인수
  - HTTP 서버가 라우트(주소) 별로 어떻게 작동할 지 정의하는 함수

<br>

```jsx
const { result } = await fetchTodo();
const rootElement = createElement(
  "div",
  { id: "root" },
  createElement(App, { todos: result })
);
const renderResult = renderToString(rootElement); // 리액트 컴포넌트 -> HTML
const htmlResult = html.replace("__placeholder__", renderResult); // replace

res.setHeader("Content-Type", "text/html");
res.write(htmlResult);
res.end();
return;
```

- server.ts의 루트 라우터
  - hydrate가 되기 이전부터 이미 서버 사이드에서 완벽하게 렌더링돼서 하나의 HTML이 만들어짐

<br>

```jsx
switch (url) {
  case "/stream": {
    res.setHeader("Content-Type", "text/html");
    res.write(indexFront);

    const result = await fetchTodo();
    const rootElement = createElement(
      "div",
      { id: "root" },
      createElement(App, { todos: result })
    );

    const stream = renderToNodeStream(rootElement);
    stream.pipe(res, { end: false });
    stream.on("end", () => {
      res.write(indexEnd);
      res.end();
    });
    return;
  }
}
```

- server.ts의 /stream 라우터
  - rootElement를 만드는 과정은 동일
  - indexFront, indexEnd : index.html의 **placeholder** 부분을 반으로 나눈 코드
  - index.html의 앞선 절반을 우선 응답으로 기록
  - 이후 renderToNodeStream을 활용해 나머지 부분을 스트림 형태로 생성
  - 청크 단위로 생성하기 때문에 pipe와 res에 걸어두고 청크가 생성될 때마다 res에 기록
  - 이 스트림이 종료되면 index.html의 나머지 반쪽을 붙이고 최종 결과물을 브라우저에 제공

<br>

```jsx
switch (url) {
  // 브라우저에 제공되는 리액트 코드
  case "/broswer.js": {
    res.setHeader("Content-Type", "application/javascript");
    createReadStream("./dist/browser.js").pipe(res);
    return;
  }

  // 위 파일의 소스맵
  case "/browser.js.map": {
    res.setHeader("Content-Type", "application/javascript");
    createReadStream("./dist/browser.js.map").pipe(res);
    return;
  }

  default: {
    res.statusCode = 404;
    res.end("404 Not Found");
  }
}
```

- broswer.js : 애플리케이션에서 작성한 리액트 및 관련 코드를 제공하는 파일. 웹팩이 생성한다.
- broswer.js.map : browser.js와 관련된 소스맵 파일. 디버깅 용도로 쓰인다.

<br>

### webpack.config.js

- 각각 브라우저 코드와 서버 코드를 번들링하는 방식을 선언
- entry로 시작점 선언 -> 필요한 파일과 그에 맞는 loader 제공 -> 번들링에서 제외할 내용 선언 -> output으로 내보낸다.

<br>

## Next.js 톺아보기

### Next.js 시작하기

#### \_app 과 \_document

- \_app : 애플리케이션 페이지 전체를 초기화
  - 최초에서는 서버 사이드 렌더링, 이후에는 클라이언트에서 렌더링 실행
  - Next.js를 초기화하는 파일. Next.js 설정과 관련된 코드를 모아두는 곳
  - 사용용도
    - 에러 바운더리를 사용해 전역으로 에러 처리
    - reset.css 등 전역 CSS 선언
    - 모든 페이지에 공통으로 사용 또는 제공해야 하는 데이터 제공

<br>

- \_document: 애플리케이션의 HTML 초기화
  - 무조건 서버에서 실행된다. -> onClick 등 이벤트 핸들러를 추가할 수 없다.
  - HTML 설정과 관련된 코드를 추가하는 곳
  - 사용
    - <html>이나 <body>에 DOM 속성을 추가할 때
    - next/document의 head는 오직 \_document에서만 실행할 수 있고, - title을사용할 수 없다.
    - CSS-in-JS의 스타일을 서버에서 모아 HTML로 제공

<br>

### 서버 라우팅, 클라이언트 라우팅

- next/link로 이동하는 경우 클라이언트 렌더링 방식으로 작동한다.
- 최초 페이지 제공 = SSR, 라우팅 = CSR
- 따라서 <Link>를 사용하고, window.location.push 대신 router.push를 사용해야 한다.
- getServerSideProps가 없으면 서버에서 실행하지 않아도 되는 페이지로 처리한다.
  - 이 때 typeof window의 처리를 모두 object로 바꾼다.
  - 이후 빌드 시점에 미리 트리쉐이킹을 해버린다.

<br>

### Data fetching

- 서버 사이드 렌더링 지원을 위한 데이터 불러오기 전략
- pages/의 폴더에 있는 라우팅이 되는 파일에서만 사용 가능하다.
- 반드시 정해진 함수 명으로 export를 사용해 함수를 파일 외부로 내보내야 한다.
- 서버에서 미리 필요한 페이지를 만들어 제공하거나 해당 페이지에 요청이 있을 때마다 서버에서 데이터를 조회해 미리 페이지를 만들어 제공할 수 있다.

<br>

### getStaticPaths, getStaticProps

- 사용자와 관계없이 정적으로 결정된 페이지를 보여주고자 할 때 사용되는 함수
- 반드시 함께 있어야 사용할 수 있다.
- 빌드 시점에 미리 데이터를 불러온 뒤 정적인 HTML 페이지를 만들 수 있다.

```jsx
// /pages/post/[id]가 접근 가능한 주소를 정의하는 함수
export const getStaticPaths: GetStaticPaths = async () => {
  return {
    paths: [{ params: { id: "1" } }, { params: { id: "2" } }], // /post/1과 /post/2만 접근 가능하다.(그 외에는 404)
    fallback: false,
  };
};

// 위에서 정의한 페이지를 기준으로 해당 페이지로 요청이 왔을 때 제공할 props를 반환하는 함수
export const getStaticProps: GetStaticProps = async ({ params }) => {
  const { id } = params;

  const post = await fetchPost(id);

  return {
    props: { post },
  };
};

// getStaticProps가 반환한 post를 렌더링하는 역할
export default function Post({ post }: { post: Post }) {
  // post로 페이지 렌더링
}
```

- fallback : 미리 빌드해야 할 페이지가 너무 많은 경우에 사용 가능한 옵션
  - paths에 미리 빌드할 몇 개의 페이지만 리스트로 반환하고, true나 "blocking"으로 값을 선언할 수 있다.
  - next build를 실행할 때 path에 기재되어 있는 페이지만 빌드, 나머지는 다르게 작동
    - true: 미리 빌드하지 않은 페이지에 접근할 경우 빌드 전까지 fallback 컴포넌트를 보여준다.
    - "blocking" : 단순히 빌드 완료까지 사용자를 기다리게 한다.

<br>

### getServerSideProps

- 서버에서 실행되는 함수. 무조건 페이지 진입 전에 이 함수를 실행한다.
- 빌드 시에도 서버용 JS 파일을 별도로 만든다.
- Next.js의 SSR은 getServerSideProps의 실행과 함께 이뤄지며, 이 정보를 기반으로 페이지가 렌더링된다.

```jsx
  export default function({post} : {post: Post}){
      // 렌더링
  }

  export const getServerSideProps : GetServerSideProps = async(context) => {
      const {
          query: {id: ''},
      } = context
      const post = await fetchPost(id.toString())
      return {
          props: {post}
      }
  }

```

- 실행 결과물

```jsx
<body>
  <div id="__next" data-reactroot="">
    <h1>안녕하세요</h1>
    <h1>반갑습니다</h1>
  </div>
  <!-- 생략 -->
  <script id="__NEXT_DATA__" type="application/json">
    {
      "props": {
        "pageProps": {
          "post": { "title": "안녕하세요", "contents": "반갑습니다." }
        },
        "__N_SSP": true,
        "page": "/post/[id]",
        "query": { "id": "1" },
        "buildId": "development",
        "isFallback": false,
        "gssp": true,
        "scriptLoading": []
      }
    }
  </script>
</body>
```

- **NEXT_DATA** 스트립트

- getServerSideProps의 정보인 props 뿐만 아니라 현재 페이지 정보. query 등 Next.js 구동에 필요한 정보가 담겨있다.
- 처음 서버에서 fetch로 가져온 정보를 결과물인 HTML에 script 형태로 내려준다.
- 반복해서 fetch 하지 않아도 되고, 시점 차이로 인한 결과물 차이도 막을 수 있다.
- 이 정보는 window 객체에도 저장된다.(window.**NEXT_DATA**)

<br>

- 일반적인 리액트의 JSX와 다르게 getServerSideProps의 props로 내려줄 수 있는 값은 JSON으로 제공할 수 있는 값으로 제한된다.

  - HTML에 정적으로 작성해서 내려주기 때문
  - class나 Date 등은 제공할 수 없다.
  - 값에 대한 가공이 필요하다면 실제 페이지나 컴포넌트에서 해야 한다.

  <br>

- 무조건 서버에서만 실행된다.

  - window, document 등 브라우저에서만 접근할 수 있는 객체에는 접근할 수 없다.
  - API 호출 시 domain 없이 fetch 요청을 할 수 없다.(서버는 자신의 호스트를 유추할 수 없기 때문)
  - 에러 발생 시 500.tsx 등 미리 정의해 둔 에러 페이지로 리다이렉트 된다.

<br>

### getInitialProps

- getStaticProps나 getServerSideProps가 나오기 전에 사용할 수 있었던 유일한 수단이었다.
- 현재는 사용이 권장되지 않는다.
- \_app.tsx나 \_error.tsx와 같이 Next.js의 특성상 사용이 제한돼 있는 페이지에서만 사용하는 것이 좋다.
- 라우팅에 따라 서버, 클라이언트 모두에서 실행 가능한 메서드
