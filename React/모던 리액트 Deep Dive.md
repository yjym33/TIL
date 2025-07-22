![alt text](/React/image/모던리액트DeepDive.png) <br>
출처 : https://www.yes24.com/product/goods/123161563

<br>

## 리액트를 위해 반드시 알아야하는 자바스크립트

### 자바스크립트의 데이터 타입

#### 원시 타입(primitive type)

- undefined : 선언되었지만 할당되지 않은 값
- null : 명시적으로 비어 있음을 나타내는 값. typeof 로 확인하면 object가 리턴된다.
- boolean : true, false 만을 가질 수 있는 데이터 타입
- number : 정수, 실수 구분 없는 숫자
- bigInt : number의 한계를 넘어 더 큰 숫자를 저장할 수 있게 해준다. 끝에 n을 붙이거나 - BigInt 함수를 사용한다.
- string : 텍스트 타입의 데이터
- symbol : 중복되지 않는 고유한 값을 만들기 위해 사용한다. Symbol()로 생성한다.

<br>

#### 객체 타입(reference type)

- 7가지 원시 타입 이외의 모든 것

<br>

#### 특징 및 차이점

- 원시 타입은 불변 형태의 값으로 저장한다. / 객체 타입은 변경 가능한 형태로 저장
- 원시 타입은 값을 복사 / 객체 타입은 값이 아닌 참조를 전달

<br>

### Object.is

- 인수 두 개가 동일한지 확인하고 반환하는 메서드
- 형변환을 하지 않는다.
- === 로 정확히 비교할 수 없는 특이한 케이스를 비교할 수 있다. (예: -0 === +0 은 true, object.is는 false 반환)
- 객체 비교는 동일하다.

### 리액트에서의 동등비교

- 리액트에서는 Object.is를 사용한다.
- ShallowEqual : Object.is를 사용해 한 번 비교한 뒤, 순회를 통해 1 depth까지 비교한다.

```js
function shallowEqual(objA: mixed, objB: mixed) {
  if (is(objA, objB)) {
    return true;
  }
  if (
    typeof objA !== "object" ||
    objA === null ||
    typeof objB !== "object" ||
    objB === null
  ) {
    return false;
  }

  // 얕은 비교 한 번 더 수행
  const keysA = Object.keys(objA);
  const keysB = Object.keys(objB);

  if (keysA.length !== keysB.length) {
    return false;
  }

  for (let i = 0; i < keysA.length; i++) {
    const currentKey = keysA[i];
    if (
      !hasOwnProperty.call(objB, currentKey) ||
      !is(objA[currentKey], objB[currentKey])
    ) {
      return false;
    }
  }
  return true;
}
```

- JSX props는 객체 -> props 값을 얕은 비교하여 값이 변했을 때 렌더링 수행
- 리액트는 얕은 비교까지만 구현 -> props에 더 깊은 객체를 넣으면 memo가 제대로 동작하지 않는다

<br>

## 함수

### 함수란?

- 작업을 수행하거나 값을 계산하는 드의 과정을 표현하고, 이를 하나의 블록으로 감싸 실행 단위로 만들어 놓은 것
- 함수 선언문
  - 함수 호이스팅이 일어난다.

```js
function add(a, b) {
  return a + b;
}
```

- 함수 표현식
  - 할당하려는 함수의 이름을 생략한다.
  - 함수를 변수에 할당하기 때문에 변수 호이스팅이 일어나 런타임 이전에 undefined로 초기화되고, 런타임에 함수가 할당된다.

<br>

- Function 생성자
  - 권장되지 않는다.

<br>

- 화살표 함수
  - constructor를 사용할 수 없다. 따라서 생성자 함수로 사용할 수 없다.
  - arguments가 존재하지 않는다.
  - 함수 자체의 바인딩을 갖지 않는다. 따라서 this를 참조하면 상위 스코프의 this를 그대로 따른다.

<br>

- 즉시 실행 함수(IIFE)
  - 함수를 정의하고 그 순간 즉시 실행되는 함수
  - 단 한 번만 호출되고 다시 호출할 수 없다. -> 이름을 붙이지 않는다.

<br>

- 고차 함수
  - 함수를 인수로 받거나 결과로 새로운 함수를 반환하는 역할을 하는 함수

<br>

### 함수를 만들 때 주의해야 할 사항

1. 함수의 부수효과를 최대한 억제한다.(순수함수)

   - 부수 효과가 없고, 동일한 인수를 받으면 동일한 결과를 반환해야 한다.
   - 작동 중 외부에 어떠한 영향도 미쳐서는 안된다.
   - 리액트 : 부수 효과를 처리하는 훅인 useEffect의 작동을 최소화하여 컴포넌트 안정성을 높인다.

2. 가능한 한 함수를 작게 만든다.

3. 누구나 이해할 수 있는 이름을 붙인다.

<br>

## 클래스

### 클래스란?

- 특정한 형태의 객체를 반복적으로 만들기 위해 사용한다.
- 데이터나 이를 조작하는 코드를 추상화할 수 있다.
- constructor
  - 생성자. 객체를 생성하는 데 사용하는 특수한 메서드
  - 단 하나만 존재할 수 있으며, 생략할 수도 있다.
- 프로퍼티
  - 클래스로 인스턴스를 생성할 때 내부에 정의할 수 있는 속성값
  - 기본적으로는 private, 타입스크립트를 사용하면 private, protected, public 사용이 가능하다.
- getter
  - 클래스에서 값을 가져올 때 사용한다.
- setter
  - 클래스 필드에 값을 할당할 때 사용한다.
- 인스턴스 메서드
  - 클래스 내부에서 선언한 메서드
  - JS의 prototype에 선언되므로 프로토타입 메서드라 부르기도 한다.
- 정적 메서드
  - 클래스의 이름으로 호출할 수 있는 메서드
  - 정적 메서드 내부의 this가 클래스 자신을 가리키기 때문
  - 전역 유틸 함수를 정적 메서드로 많이 활용한다.

```js
class Car(){
  static hello(){
    console.log('hi');
  }
}


const myCar = new Car();
myCar.hello() // 에러
Car.hello() // hi
```

<br>

## 클로저

### 클로저란?

- 함수 컴포넌트의 구조와 작동 방식, 훅의 원리, 의존성 배열 등 함수 컴포넌트의 대부분이 클로저에 의존한다.

```js
function outerFunction() {
  var x = "hello";
  function innerFunction() {
    console.log(x);
  }
  return innerFunction;
}

const innerFunction = outerFunction(); // 1) outerFunction은 innerFunction을 반환하며 종료
innerFunction(); // 'hello' - 2) x가 선언된 어휘적 환경에는 x 변수 존재 -> innerFunction()은 같은 환경에서 선언되었기 때문에 x라는 변수가 존재하는 환경을 기억한다.
```

### 스코프

- 변수의 유효 범위
- 전역 스코프 : 브라우저 환경 window, Node.js 환경 global
- 함수 스코프 : 자바스크립트는 함수 레벨 스코프를 가지고 있다.

<br>

### 클로저의 활용

```jsx
function Component() {
  const [state, setState] = useState();

  // useState 호출이 끝나도 setState는 state의 최신값을 알고 있다.
  function handleClick() {
    setState((prev) => prev + 1);
  }
}
```

<br>

## 이벤트 루프와 비동기 통신의 이해

### 싱글 스레드 자바스크립트

- 프로세스 : 프로그램의 상태가 메모리상에서 실행되는 작업 단위
- 스레드 : 하나의 프로그램에서 여러 개의 복잡한 작업 수행 필요 -> 더 작은 실행 단위인 스레드 등장. 스레드끼리는 메모리를 공유할 수 있다.
- 싱글 스레드 언어 = 코드의 실행이 하나의 스레드에서 순차적으로 이루어진다. 동기식으로 처리된다.

<br>

### 이벤트 루프

- JS 런타임 외부에서 비동기 실행을 돕기 위해 만들어진 장치
- 호출 스택(call stack) : JS에서 수횅해야 할 코드나 함수를 순차적으로 담아두는 스택
- 이벤트 루프(event loop)
  - 호출 스택이 비어 있는지 여부를 확인 -> 수행해야 할 코드가 있다면 JS 엔진을 이용해 실행한다.
  - 여기서 코드 실행과 호출 스택 확인은 모두 단일 스레드에서 일어난다.
- 태스크 큐(task queue)
  - 실행해야 할 태스크의 집합
  - 이벤트 루프는 태스크 큐를 한 개 이상 가지고 있다.
  - 호출 스택이 비었다면 태스크 큐에 대기중인 작업이 있는지 확인 -> 있다면 실행 가능한 가장 오래된 것부터 순차적으로 꺼내 실행한다.
- 비동기 함수는 메인 스레드가 아닌 태스크 큐가 할당되는 별도의 스레드에서 수행된다.
  - JS 코드 실행을 싱글 스레드에서 이루어진다.
  - 외부 Web API 등은 JS 코드 외부에서 실행되고, 콜백이 태스크 큐로 들어간다.
  - 이벤트 루프는 호출 스택이 비고, 콜백 실행 가능할 때 꺼내어 수행하는 역할을 한다.

<br>

## 태스크 큐 & 마이크로태스크 큐

- 이벤트 루프는 하나의 마이크로 태스크 큐를 가지고 있다.(예: Promise)
- 마이크로 태스크 큐는 기존 태스크 큐보다 우선권을 갖는다.(예: setTimeout은 Promise보다 늦게 실행된다.)
- 마이크로 태스크 큐가 빌 때까지는 기존 태스크 큐의 실행은 뒤로 미뤄진다.
- 대표적인 작업
  - 태스크 큐 : setTimeout, setInterval, setImmediate
  - 마이크로 태스크 큐 : process.nextTick, Promiese, queueMicroTask, MutationObserver
- 렌더링
  - 마이크로 태스크 큐를 실행한 뒤 렌더링이 일어난다.
  - 각 마이크로 태스크 큐 작업이 끝날 때마다 한 번씩 렌더링할 기회를 얻는다.

<br>

## 리액트에서 자주 사용하는 JS 문법

### 구조분해할당

- 배열 또는 객체의 값을 분해해 개별 변수에 즉시 할당하는 것
- useState : 배열 구조 분해 할당(원하는 이름으로의 변경이 자유롭다)
- 객체구조분해할당 : 트랜스파일 시 번들링 크기가 상대적으로 크다.

<br>

### 전개 구문

- 순회할 수 있는 값을 전개해 간결하게 사용할 수 있는 구문
- 객체 전개 연산자 : 트랜스파일 시 상대적으로 번들링 크기가 크다.

<br>

### 객체 초기자

```js
const a = 1;
const b = 2;

const obj = {
  a,
  b,
};

// {a: 1, b: 2}
```

- 트랜스파일 이후에도 큰 부담이 없다.

<br>

### Array 프로토타입 메서드

- Array 프로토타입 메서드
  - 각 아이템 순회하며 콜백 연산 결과로 구성된 새로운 배열 리턴
- Array.prototype.filter
  - 콜백 함수에서 truthy 조건을 만족하는 새로운 배열 리턴
- Array.prototype.reduce
  - 콜백 함수, 초깃값을 인수로 받음 -> 초깃값에 따라 배열, 객체 등을 리턴
- Array.prototype.forEach
  - 배열을 순회하면서 단순히 콜백 함수 실행
  - 에러, 프로세스 종료가 아닌 이상 중간에 멈출 수 없다.

<br>

## 타입스크립트

### 타입스크립트란?

- 기존 JS 문법에 타입을 가미한 것
- JS는 동적 타입 언어이기 때문에 코드 실행했을때만 에러 확인 가능 -> TS는 타입체크를 정적으로 빌드(트랜스파일) 타임에 수행

<br>

### 활용법

1. any 대신 unknown 사용
   - 불가피하게 아직 타입을 단정할 수 없는 경우에 unknown을 사용한다.
   - unknown
     - 모든 값을 할당할 수 있는 top type. 어떠한 값도 할당 가능
     - 바로 사용할 수 없고, 타입을 의도했던 대로 적절히 좁혀야 한다.

<br>

```js
// 불가능
function doSomething(callback: unknown) {
  callback();
}

// 가능
function doSomething(callback: unknown) {
  if (typeof callback === "function") {
    callback();
    return;
  }
  throw new Error("Callback은 함수여야 합니다.");
}
```

<br>

- never
  - bottom type. 어떠한 타입도 들어올 수 없다.
  - 클래스 컴포넌트에서 props에 어떠한 값도 받아들이고 싶지 않을 때 never로 타입을 설정해준다.

<br>

2. 타입 가드

- 타입을 좁히는 데 도움을 준다
- instanceof : 지정한 인스턴스가 특정 클래스의 인스턴스인지 확인 가능
- typeof : 특정 요소에 대해 자료형 확인
- in : property in object로 사용한다. 어떤 객체에 키가 존재하는 지 확인한다.

<br>

3. 제네릭

- 함수나 클래스 내부에서 단일 타입이 아닌 다양한 타입에 대응할 수 있도록 도와주는 도구
- 리액트의 useState 타입을 결정할 때 사용된다.
- 하나 이상 사용할 수 있다.

<br>

```jsx
function multipleGeneric<First, Last>(a1: First, a2: Last) : [First, Last]{
   return [a1, a2]
}

const [a, b] = multipleGeneric<string, boolean>('true', true);
a // string
b // boolean

```

<br>

4. 인덱스 시그니처

- 객체의 키를 정의하는 방식. 키에 원하는 타입을 부여할 수 있다.
- 키의 범위를 최대한 줄이고, 존재하지 않은 키로 접근하면 undefined를 리턴하기 때문에 키를 동적으로 선언되는 경우를 최대한 지양해야 한다.
- 객체의 키를 좁히는 방법

<br>

```jsx
// 1) record 사용
type Hello = Record<'hello' | 'hi', string>

const hello : Hello = {
  hello: 'hello',
  hi: 'hi'
}

// 2) 타입을 사용한 인덱스 시그니처
type Hello = {[key in 'Hello' | 'hi'] : string}

const hello : Hello = {
  hello: 'hello',
  hi: 'hi'
}
```

<br>

- Object.keys() 를 사용할 때는 에러가 날 수 있음
  - Object.keys()는 string[]을 반환한다.
  - JS는 값으로 타입을 비교하기 때문에(덕 타이핑, 구조적 타이핑) 객체의 타입에 열려있다.
  - 따라서 TS도 이러한 원칙에 따라 모든 키가 들어올 수 있는 가능성이 열려있기 때문에 string[]으로 타입 제공
