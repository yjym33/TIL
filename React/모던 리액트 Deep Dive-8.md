![alt text](/React/image/모던리액트DeepDive.png) <br>
출처 : https://www.yes24.com/product/goods/123161563

## [8장] 좋은 리액트 코드 작성을 위한 환경 구축하기

<br>

## 8.1 ESLint를 활용한 정적 코드 분석

버그나 예상치 못한 작동을 방지하기 위해선? 정적 코드 분석을 하자!!

- 정적 코드 분석: 코드의 실행과는 별개로 코드 그 자체만으로도 코드 스멜(잠재적으로 버그를 야기할 수 있는 코드)을 찾아내어 문제의 소지가 있는 코드를 사전에 수정하는 것

### ESLint란?

- ESLint는 자바스크립트 코드를 정적 분석해 잠재적인 문제를 발견하고 나아가 수정까지 도와주는 도구

- ESLint는 어떻게 코드를 분석할까?
  1. 자바스크립트 코드를 문자열로 읽는다.
  2. 자바스크립트 코드를 분석할 수 있는 파서(parser)로 코드를 구조화한다. → 기본값: espree
  3. 2번에서 구조화한 트리는 AST(Abstract Syntax Tree)라 하고, AST를 기준으로 각종 규칙과 대조한다.
  4. 규칙과 대조했을 때 이를 위반한 코드를 알리거나 수정한다.

```js
function hello(str) {}
```

```json
{
  "type": "Program",
  "body": [
    {
      "type": "FunctionDeclaration",
      "id": {
        "type": "Identifier",
        "name": "hello"
      },
      "params": [
        {
          "type": "Identifier",
          "name": "str"
        }
      ],
      "body": {
        "type": "BlockStatement",
        "body": []
      },
      "generator": false,
      "async": false
    }
  ],
  "sourceType": "script"
}
```

→ 단순히 한 줄밖에 안 되는 함수 내부 코드가 아무것도 없는 단순한 자바스크립트 코드임에도 불구하고 JSON으로 생성된 트리에 다양한 정보가 담겨 있다 <br>
→ 이러한 자세한 정보가 있어야만 ESLint, Prettier 같은 도구가 코드의 줄바꿈, 들여쓰기 등을 파악할 수 있다. 이를 **ESLint 규칙(rule)**이라고 하며, 특정한 규칙의 모음을 plugins라고 함.

<br>

### eslint-plugin과 eslint-config

- eslint-plugin

  - eslint-plugin이라는 접두사로 시작하는 플러그인은 규칙을 모아놓은 패키지

- eslint-config

eslint-plugin 을 한데 묶어서 완벽하게 한 세트로 제공하는 패키지 → 설정이 만만치 않아 이미 존재하는 eslint-config 를 설치하는 것이 일반적임… ex) eslint-config-airbnb, @titicaca/triple-config-kit, eslint-config-next

- eslint-config-airbnb

  - Airbnb 스타일 가이드에 따른 규칙 세트
  - React, JSX, ES6+ 문법에 대한 규칙 포함
  - 엄격한 코드 스타일 규칙 제공

- @titicaca/triple-config-kit

  - Triple의 내부 코드 스타일과 규칙 포함
  - 특정 환경에 맞춤화된 규칙 제공
  - 회사 내부 개발 표준을 유지하기 위한 설정

- eslint-config-next

  - Next.js 프로젝트에 특화된 규칙 포함
  - Next.js의 권장 코드 스타일 제공
  - 서버 사이드 렌더링과 정적 사이트 생성을 위한 규칙 포함

<br>

### 나만의 ESLint 규칙 만들기

- import React 삭제해보기 ~~

1. eslint 설치하기
2. 커스텀 eslint 플러그인 만들기
   - 프로젝트 디렉토리에 .eslintrc.js 생성
   - 커스텀 플러그인 파일 작성

```js
// eslint-plugin-custom-rules.js
module.exports = {
  rules: {
    "no-import-react": {
      create(context) {
        return {
          ImportDeclaration(node) {
            if (node.source.value === "react") {
              context.report({
                node,
                message: "import React를 넣지 말아요~",
                fix(fixer) {
                  return fixer.remove(node);
                },
              });
            }
          },
        };
      },
    },
  },
};
```

3. ESLint 설정 파일(.eslintrc.js) 수정

```js
module.exports = {
  root: true,
  env: {
    browser: true,
    es2021: true,
  },
  extends: [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:@typescript-eslint/recommended",
  ],
  parserOptions: {
    ecmaFeatures: {
      jsx: true,
    },
    ecmaVersion: 12,
    sourceType: "module",
  },
  plugins: [
    "react",
    "@typescript-eslint",
    "./eslint-plugin-custom-rules", // 여기서 커스텀 플러그인을 추가하기
  ],
  rules: {
    "custom-rules/no-import-react": "error", // 커스텀 규칙을 활성화
  },
};
```

4. eslint 실행 (import React 구문이 있는지 검사하고 자동으로 제거)

```bash
npx eslint --fix .

```

### 주의할 점

- Prettier와의 충돌

  - Prettier는 코드 포매팅을 도와주는 도구
    - ESLint는 코드의 잠재적인 문제가 될 수 있는 부분을 분석
    - Prettier는 포매팅과 관련된 작업, 즉 줄바꿈, 들여쓰기, 작은따옴표와 큰따옴표 등.. Prettier는 자바스크립트뿐만 아니라 HTML, CSS, JSON 등 다양한 언어에도 적용 가능하다. 자바스크립트의 경우 두 도구 모두 처리할 수 있기 때문에 충돌하는 규칙으로 인해 에러가 발생할 수 있다!!
  - 규칙이 충돌하지 않게 나눠서 선언하기
  - 자바스크립트나 타입스크립트는 ESLint에, 그 외 파일은 Prettier에 맡기기

- 규칙에 대한 예외 처리, 그리고 react-hooks/no-exhaustive-deps

  - 일부 코드에서 특정 규칙을 임시로 제외시키고 싶다면 eslint-disable- 주석을 사용하기 (특정 줄, 파일 전체, 혹은 특정 범위에 걸쳐 제외하는 것 가능)
  - 리액트에서 이런 규칙을 가장 많이 사용하는 곳 중 하나가 eslint-disable-line no-exhaustive-deps
    - useEffect나 useMemo와 같이 의존 배열이 필요한 훅에 의존성 배열을 제대로 선언했는지 확인하는 역할
    - 개발 시 이 의존성 배열이 너무 길어지거나, 혹은 빈 배열을 넣어서 컴포넌트가 마운트되는 시점에 한 번만 강제로 실행되게 하고 싶을 때, 혹은 임의로 판단해 없어도 괜찮다고 생각될 때 등에 사용된다.

의존성 배열에 값이 없어도 괜찮다고 임의로 판단하지말자. → 버그 야기할 위험성 높음

<br>

의존성 배열이 너무 긴경우 → useEffect를 쪼개자. → 의존성 배열을 너무 길게 설정하면, 해당 배열의 모든 값이 변경될 때마다 useEffect가 실행될 수 있다.

마운트 시점에 한 번만 실행하고 싶은 경우 → 클래스형 컴포넌트에서 주로 사용되던 생명주기 형태의 접근 방법으로, 함수형 컴포넌트의 패러다임과는 맞지 않을 수 있다. 또한, 상태 불일치가 일어날 수도 있다.

<br>

- ESLint 버전 충돌

  예) create-react-app을 실행하면 설치되는 react-scripts의 5.0.1 버전에는 ESLint 8에 의존성을, eslint-config-triple은 ESLint 7에 의존성을 두고 있다.

  eslint-config, eslint-plugin이 지원하는 ESLint 버전을 확인하고, 설치하고자 하는 프로젝트에서 어떤 ESLint를 사용하고 있는지 살펴보자.

<br>

## 8.2 리액트 팀이 권장하는 리액트 테스트 라이브러리

- 테스트 : 개발자가 만든 프로그램이 코딩을 한 의도대로 작동하는지를 확인하는 일련의 작업
- 백엔드의 테스트는 일반적으로 서버나 데이터베이스에서 원하는 데이터를 올바르게 가져올 수 있는지, 데이터 수정 간 교착 상태가 경쟁 상태가 발생하지는 않는지, 데이터 손실이나 특정 상황에서의 장애 여부 등을 확인하는 과정이 주를 이룬다. 일반적으로 화이트박스 테스트로, GUI가 아닌 AUI(Application User Interface)에서 주로 수행해야 하기 때문에 어느 정도 백엔드에 대한 이해가 있는 사람만 가능하다.

프론트엔드의 테스트는 주로 블랙박스 테스트가 이뤄지고 코드와 상관없이 의도된 대로 작동하는지를 확인하는 데에 좀 더 초점이 맞춰져 있다. 단순히 함수나 컴포넌트 수준에서 유닛 테스트를 할 수도 있고, 사용자가 하는 작동을 모두 흉내 내서 테스트 할 수도 있다.

- 화이트박스 테스트 : 내부 구조와 작동 원리를 이해하고 있는 상태에서 소프트웨어를 테스트하는 접근 방식. 주로 개발자들이 코드를 작성하고 디버깅하는 데 사용된다.
- 블랙박스 테스트 : 소프트웨어의 내부 구조나 작동 방식을 몰라도 기능적인 측면에서 테스트를 수행하는 접근 방식. 외부에서 소프트웨어를 사용하는 사용자의 관점에서 테스트 케이스를 설계한다.

### React Testing Library란?

- DOM Testing Library를 기반으로 만들어진 테스팅 라이브러리
- 리액트 테스팅 라이브러리를 활용하면 리액트 컴포넌트가 원하는 대로 렌더링되고 있는지 확인할 수 있다.

<br>

> DOM Testing Library는 jsdom 기반 <br><br>**jsdom 이란?** 순수하게 자바스크립트로 작성된 라이브러리. HTML이 없는 자바스크립트만 존재하는 환경, Node.js같은 환경에서도 HTML과 DOM을 사용할수 있도록 해주는 라이브러리

<br>

- **자바스크립트 테스트의 기초**
  1. 테스트할 함수나 모듈을 선정한다.
  2. 함수나 모듈이 변화하길 기대하는 값을 적는다
  3. 함수나 모듈의 실제 반환 값을 적는다.
  4. 3번의 기대에 따라 2번의 결과가 일치하는지 확인한다.
  5. 기대하는 결과를 반환한다면 테스트는 성공이고, 만약 기대와 다른 결과를 반환하면 에러를 던진다.

<br>

```js
// 예시 함수: 두 숫자의 합을 계산하는 함수
function add(a, b) {
  return a + b;
}

// 테스트 케이스
const expected = 5; // 기대하는 값
const result = add(2, 3); // 실제 반환된 값

// 결과 비교
if (result === expected) {
  console.log("테스트 성공");
} else {
  throw new Error(`테스트 실패: 기대값 ${expected}, 실제값 ${result}`);
}
```

<br>

> 테스트를 하려면 “작성한 코드가 예상대로 작동한다면 성공했다는 메시지가 출력되고, 실패하면 에러를 던진다.” 이 작동을 해주는 라이브러리가 필요하다.

- Node.js는 assert라는 모듈을 기본적으로 제공하고, 이 모듈을 사용하면 위와 같이 작동하도록 만들 수 있다.

<br>

```js
const assert = require("assert");

// 예시 함수: 두 숫자의 합을 계산하는 함수
function add(a, b) {
  return a + b;
}

// 테스트 케이스: add 함수를 테스트
// strictEqual: actual과 expected 값을 엄격하게 비교하여 동일한지 확인
assert.strictEqual(add(2, 3), 5, "add 함수는 2와 3의 합이 5여야 합니다.");

// 테스트 성공 시 출력
console.log("테스트 성공");
```

- 어설션 라이브러리: 테스트 결과를 확인할 수 있도록 도와주는 라이브러리
- 어설션: 테스트 코드에서 실행 결과가 기대한 대로 나왔는지를 확인하고 검증하는 데 사용됨
- assert, should.js, expect.js, chai 등…
  - equal: 단순 동등 비교
  - deepEqual: 객체 자체가 동일한지 비교
  - notEqual: 같이 않은지 비교
  - throws: 에러 던지는지 여부

> 테스트 코드가 정상적으로 작동하고, 테스트도 모두 통과하겠지만 무엇을 테스트했는지,무슨 테스트를 어떻게 수행했는지 등 테스트에 관한 실제 정보를 알 수 없다. \*\*좋은 테스트 코드는 다양한 테스트 코드가 작성되고 통과하는 것뿐만 아니라 어떤 테스트가 무엇을 테스트하는지 일목요연하게 보여주는 것도 중요하다.
> -> 스팅 프레임워크 (Jest, Mocha, Karma, Jasmine)
