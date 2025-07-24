![alt text](/React/image/모던리액트DeepDive.png) <br>
출처 : https://www.yes24.com/product/goods/123161563

<br>

# 리액트 훅 깊게 살펴보기

<br>

## UseState

- 함수 컴포넌트 내부에서 상태를 정의하고, 이 상태를 관리할 수 있게 해주는 훅
- 매번 실행되는 함수 컴포넌트 환경에서 state의 값을 유지하고 사용하기 위해 클로저를 활용한다.

```jsx
// 내부 작동 자체만을 구현
const MyReact = function () {
  const global = {};
  let index = 0;

  function useState(initialState) {
    if (!global.states) {
      // 애플리케이션 전체 states 배열 초기화. 최초 접근이면 빈 배열로 초기화
      global.states = [];
    }

    // states 정보를 조회해서 현재 상태값 있는지 확인, 없으면 초깃값으로 설정
    const currentState = global.states[index] || initialState;
    global.states[index] = currentState;

    const setState = (function () {
      // 현재 index를 클로저로 가둬놔서 이후에도
      // 계속해서 동일한 index에 접근할 수 있도록 한다.
      let currentIndex = index;
      return function (value) {
        global.states[currentIndex] = value;
        // 컴포넌트 렌더링(코드 생략)
      };
    })();

    // useState를 쓸 때 마다 index를 하나씩 추가한다.
    // index는 setState에서 사용된다.
    // 즉 하나의 state마다 index가 할당되어 있어 그 index가 배열의 값(global.states)을 가리킨다.

    index = index + 1;

    return [currentState, setState];
  }
};
```

<br>

### 게으른 초기화(lazy initialization)

- useState에 변수 대신 함수를 넘기는 것
- useState의 초깃값이 복잡하거나 무거운 연산을 포함하고 있을 때 사용한다
  - localStorage, sessionStorage에 대한 접근
  - map, filter, find 등 배열 접근
  - 초기값 계산을 위해 함수 호출이 필요할 때 등
- 게으른 초기화 함수는 처음 만들어질 때만 사용된다.(이후 리렌더링 발생해도 실행이 무시된다.)

<br>

```jsx
import { useState } from "react";

export default function App() {
  const [state, setState] = useState(() => {
    // App 컴포넌트가 처음 구동될 때만 실행되고, 이후 리렌더링 시에는 실행되지 않는다.
    console.log("복잡한 연산..");

    return 0;
  });

  function handleClick() {
    setState((prev) => prev + 1);
  }

  return (
    <div>
      <h1>{state}</h1>
      <button onClick={handleClick}>+</button>
    </div>
  );
}
```

<br>

## useEffect

- 애플리케이션 내 컴포넌트의 여러 값들을 활용해 동기적으로 부수 효과를 만드는 훅
- 부수효과가 '언제' 일어나는지보다 어떤 상태값과 함께 실행되는지 살펴보는 것이 중요하다.

### useEffect란?

- 첫 번째 인수 : 실행할 부수 효과가 포함된 함수 / 두 번째 인수 : 의존성 배열
- 함수 컴포넌트는 렌더링 시마다 고유의 state와 props를 가지고 있다.
- useEffect는 state와 props의 변화 속에서 일어나는 렌더링 과정에서 실행되는 부수 효과 함수 이다.

<br>

### 클린업 함수의 목적

- 클린업 함수는 이전 state를 참조해 실행된다.
- 새로운 값과 함께 렌더링된 뒤에 실행된다.(함수가 정의됐을 당시에 선언됐던 이전 값을 보고 실행한다.)
- useEffect는 콜백이 실행될 때마다 이전의 클린업 함수가 존재한다면 그 클린업 함수를 실행한 뒤 콜백을 실행한다. (이전 상태를 청소해주는 개념)

<br>

의존성 배열

- 의존성 없는 useEffect vs 그냥 실행

```jsx
// 1
function Component(){
  console.log('렌더링됨')
}

// 2
function Component(){
  useEffect(() => {
    console.log('렌더링됨)
  })
}
```

- SSR 관점에서 useEffect는 클라이언트 사이드에서 실행되는 것을 보장해준다.(window 객체 접근 가능)
- useEffect는 컴포넌트의 렌더링이 완료된 이후에 실행된다.
  - 1번(함수 내부 직접 실행) : 컴포넌트가 렌더링되는 도중에 실행된다.
  - 따라서 1번은 서버에서도 실행된다.
  - 함수 컴포넌트의 반환을 지연시키는 행위 -> 렌더링을 방해한다.

<br>

### 구현

```jsx
const MyReact = (function(){
  const global = {}
  let index = 0

  function useEffect(callback, dependencies){
    const hooks = global.hooks

    // 이전 훅 정보가 있는지 확인한다.
    let previousDependencies = hooks[index]

    // 변경됐는지 확인한다.
    // 이전 값이 있다면 이전 값을 얕은 비교로 비교해 변경이 일어났는지 확인한다.
    // 이전 값이 없다면 최초 실행이므로 변경이 일어난 것으로 간주해 실행을 유도한다.
    let isDependenciesChanged = previousDependencies
      ? dependencies.some(
        (value, idx) => !Object.is(value, previousDependencies[idx]).
      )
      : true

    // 변경이 일어났다면 첫 번째 인수인 콜백 함수를 실행한다.
    if(isDependenciesChanged){
      callback()

      // 다음 훅이 일어날 때를 대비하기 위해 index 추가
      index ++

      // 현재 의존성을 훅에 다시 저장한다.
      hooks[index] = dependencies
    }
  }

  return { useEffect }
})()
```

<br>

### 사용시 주의할 점

- eslint-disable-line react-hooks/exhaustive-deps 주석 자제하기

  - useEffect 인수 내부에서 사용하는 값 중 의존성 배열에 포함돼 있지 않은 값이 있을 때 경고
  - 무시 = 컴포넌트의 state, props와 같은 어떤 값의 변경과 useEffect 부수 효과가 별개로 작동하게 된다.
  - 메모이제이션을 활용해 변화를 막거나 적당한 실행 위치를 고민해봐야 한다.

- 첫 번째 인수에 함수명을 부여하기
  - useEffect가 복잡하고 하는 일이 많다면 인수를 기명 함수로 바꿔주는 것이 좋다.
- 거대한 useEffect 만들지 않기

  - 부수 효과의 크기가 커질수록 애플리케이션 성능에 악영향을 미친다.
  - 적은 의존성 배열을 사용하는 여러 개의 useEffect로 분리하는 것이 좋다.
  - 여러 변수가 들어가야 하면 -> useCallback과 useMemo로 사전에 정제한 내용들만 useEffect에 담아두자

- 불필요한 외부 함수를 만들지 마라
  - useEffect 내에서 사용할 부수 효과라면 내부에서 만들어 정의해 사용하는 편이 훨씬 좋다

<br>

> 콜백 인수로 비동기 함수를 바로 넣을 수 없다

<br>

- 비동기 함수의 응답 속도에 따라 결과가 이상하게 나타날 수 있다.

```jsx
// error
useEffect(() => {
  const response = await fetch('...some url')
  const result = await response.json()
  setData(result)
},[])

// good
useEffect(() => {
  let shouldIgnore = false

  async function fetchData(){
    const response = await fetch('...some url')
    const result = await response.json()
    if(!shouldIgnore){
      setData(result)
    }
  }

  fetchData()

  return () => {
    shouldIgnore = true
  }
},[])
```

- useEffect의 경쟁 상대(race condition)라 한다.
- 비동기 함수가 내부에 존재하게 되면 useEffect 내부에서 함수가 생성되고 실행되는 것을 반복 -> 클린업 함수에서 이전 비동기 함수 처리를 하는 것이 좋다.
- state의 경쟁 상태 야기, cleanup 함수 실행순서도 보장할 수 없기 때문에 편의성을 위해 비동기 함수를 인수로 받지 않는다.

<br>

## useMemo

- 비용이 큰 연산에 대한 결과를 저장(메모이제이션)해 두고, 이 저장된 값을 반환하는 훅

```jsx
import { useMemo } from "react";

const memoizedValue = useMemo(() => expensiveComputation(a, b), [a, b]);
```

- 인수
  - 첫 번째 인수 : 어떠한 값을 반환하는 생성 함수
  - 두 번째 인수 : 해당 함수가 의존하는 값의 배열
- 렌더링 발생 시 의존성 배열의 값이 변경되지 않았으면 함수를 재실행하지 않고 이전에 기억해둔 값을 반환한다.
- 의존성 배열 값이 변경되었다면 첫 번째 인수 함수를 실행한 뒤 그 값을 반환하고 다시 기억해둔다.
- 무거운 연산을 다시 수행하는 것을 막을 수 있다.

<br>

## useCallback

- 인수로 넘겨받은 콜백 자채를 기억해 특정 함수를 새로 만들지 않고 다시 재사용하는 훅

```jsx
const ChildComponent = memo(({ name, value, onChange }) => {
  return (
    <>
      <h1>
        {name} {value ? "켜짐" : "꺼짐"}
      </h1>
      <button onClick={onChange}>toggle</button>
    </>
  );
});

function App() {
  const [status1, setStatus1] = useState(false);
  const [status2, setStatus2] = useState(false);

  const toggle1 = () => {
    setStatus1(!status1);
  };

  const toggle2 = () => {
    setStatus2(!status2);
  };

  return (
    <>
      <ChildComponent name="1" value={status1} onChange={toggle1} />
      <ChildComponent name="2" value={status2} onChange={toggle2} />
    </>
  );
}
// memo를 사용해 메모이제이션했지만 App의 자식 컴포넌트 전체가 렌더링된다.
// 한 버튼을 클릭해도 둘 다 렌더링된다.
// state 값이 바뀌면서 App이 리렌더링 -> onChange로 넘기는 함수가 재생성되고 있기 때문이다.

// 위 예제에서 useCallback 추가
const toggle1 = useCallback(() => {
  setStatus1(!status1);
}, [status1]);

const toggle2 = useCallback(() => {
  setStatus2(!status2);
}, [status2]);

// 의존성이 변경됐을 때만 함수가 재생성된다.
```

<br>

## useRef

- useState와 동일하게 컴포넌트 내부에서 렌더링이 일어나도 변경 가능한 상태값을 저장한다.
- 컴포넌트가 렌더링될 때만 생성된다.
- 컴포넌트 인스턴스가 여러 개라도 각각 별개의 값을 바라본다.
- useState와의 차이점
  - 반환값인 객체 내부에 있는 current로 값에 접근 또는 변경할 수 있다.
  - 그 값이 변하더라도 렌더링을 발생시키지 않는다.

<br>

```jsx
  const inputRef = useRef()

console.log(inputRef.current) // useRef의 최초 기본값은 DOM이 아니고 useRef()로 넘겨받은 인수이다. -> 선언된 당시에는 렌더링 전이라 return 전이기 때문에 undefined

useEffect(() => {
console.log(inputRef.current) // <input type="text"></input>
}, [inputRef])

return <input ref={inputRef} type="text"/>
}

```

<br>

- useState의 이전 값을 저장하는 usePrevious() 같은 훅을 구현할 때 유용하다.

<br>

```jsx
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => {
    ref.current = value;
  }, [value]); // value가 변경되면 그 값을 value에 넣어준다
  return ref.current;
}

function SomeComponent() {
  const [counter, setCounter] = useState(0);
  const previousCounter = usePrevious(counter);

  function handleClick() {
    setCounter((prev) => prev + 1);
  }

  return (
    <button onClick={handleClick}>
      {counter} {previousCounter}
    </button>
  );

  // 0 undefined
  // 1, 0
  // 2, 1
  // 3, 2
}
```

<br>

### useRef 구현

```jsx
export function useRef(initialValue) {
  currentHook = 5;
  return useMemo(() => ({ current: initialValue }), []);
}
```

<br>

## useContext

### context란?

- props drilling을 해결하기 위해 등장한 개념
- 명시적인 props 전달 없이도 하위 컴포넌트 모두에서 자유롭게 원하는 값을 사용할 수 있게 해준다.

### useContext

- 콘텍스트를 함수 컴포넌트에서 사용할 수 있게 해준다.

### 사용 시 주의할 점

- Provider에 의존성을 가지게 되어 컴포넌트 재활용이 어려워진다.
- useContext를 사용하는 컴포넌트를 최대한 작게 하거나 재사용되지 않을 만한 컴포넌트에서 사용해야 한다.
- 컨텍스트가 미치는 범위는 필요한 환경에서 최대한 좁게 만들어야 한다.
- 렌더링이 최적화 되지는 않는다.

<br>

## useReducer

- useState와 비슷한 형태를 띠지만 좀 더 복잡한 상태값을 미리 정의해 놓은 시나리오에 따라 관리할 수 있다
- 반환값은 useState와 동일하게 길이가 2인 배열이다.
  - state : 현재 useReducer가 가지고 있는 값
  - dispatcher : state를 업데이트하는 함수. setState는 단순히 값을 넘겨주지만 여기서는 action을 넘겨준다.
- 2 ~ 3개의 인수를 필요로 한다.
  - reducer : 기본 action을 정의하는 함수
  - initialState : useReducer의 초깃값
  - init : 초깃값을 지여내서 생성시키고 싶을 때 사용하는 함수(optional). 존재하면 게으른 초기화가 일어나며 initialState를 인수로 init 함수가 실행된다.

```jsx
type State = {
  count: number,
};

type Action = { type: "up" | "down" | "reset", payload?: State };

// count를 받아 초깃값을 어떻게 정의할지 연산
function init(count: State): State {
  return count;
}

const initialState: State = { count: 0 };

// state, action을 기반으로 state가 어떻게 변경될 지 정의
function reducer(state: State, action: Action): State {
  switch (action.type) {
    case "up":
      return { count: state.count + 1 };
    case "down":
      return { count: state.count - 1 > 0 ? state.count - 1 : 0 };
    case "reset":
      return init(action.payload || { count: 0 });
    default:
      throw new Error(`Unexpected action type ${action.type}`);
  }
}

export default function App() {
  const [state, dispatcher] = useReducer(reducer, initialState, init);

  function handleUpButtonClick() {
    dispatcher({ type: "up" });
  }

  function handleDownButtonClick() {
    dispatcher({ type: "down" });
  }

  function handleResetButtonClick() {
    dispatcher({ type: "reset", payload: { count: 1 } });
  }

  return (
    <div className="App">
      <h1>{state.count}</h1>
      <button onClick={handleUpButtonClick}>+</button>
      <button onClick={handleDownButtonClick}>-</button>
      <button onClick={handleResetButtonClick}>reset</button>
    </div>
  );
}
```

- 사용 목적
  - 복잡한 형태의 state를 사전에 정의된 dispatcher로만 수정할 수 있게 만들어준다.
  - state 값을 변경하는 시나리오를 제한적으로 두고 이에 대한 변경을 빠르게 확인할 수 있게끔 한다.

<br>

## useImperativeHandle

### forwareRef

- ref를 하위 컴포넌트로 전달하고 싶을 때, 일관성을 제공하기 위해 사용한다.
- 완전한 네이밍의 자유가 주어진 props 보다는 forwardRef를 사용하면 좀 더 확실히 ref를 전달할 것을 예측할 수 있다.

### useImperativeHandle

- 부모에게서 넘겨받은 ref를 원하는대로 수정할 수 있는 훅

<br>

```jsx
const Input = forwareRef((props, ref) => {
  // ref의 동작을 추가로 정의할 수 있다.
  useImperativeHandle(
    ref,
    () => ({
      alert: () => alert(props.value),
    }),
    [props.value]
  );

  return <input ref={ref} {...props} />;
});

function App() {
  const inputRef = useRef();
  const [text, setText] = useState("");
  function handleChange(e) {
    setText(e.target.value);
  }

  return (
    <>
      <Input ref={inputRef} value={text} onChange={handleChange} />
      <button onClick={handleClick}>Focus</button>
    </>
  );
}
```

<br>

### useLayoutEffect

> 이 함수의 시그니처는 useEffect와 동일하나, 모든 DOM의 변경 후에 동기적으로 발생한다.

- 두 훅의 형태나 사용 예제가 동일하다
- DOM 변경 = 렌더링
- 실행 순서
  - 리액트가 DOM 업데이트
  - useLayoutEffect를 실행 - 브라우저에 변경 사항이 반영되기 전에 실행
  - 브라우저에 변경 사항을 반영
  - useEffect를 실행 - 브라우저에 변경 사항이 반영된 이후에 실행
- useLayoutEffect의 실행이 종료될 때까지 기다린 다음에 화면을 그린다.
  - 리액트 컴포넌트는 useLayoutEffect가 완료될 때까지 기다린다.
  - 성능문제 발생 가능성
- DOM은 계산됐지만 이것이 화면에 반영되기 전에 하고 싶은 작업이 있을 때 등 반드시 필요할 떄만 사용해야 한다.
  - DOM 요소를 기반으로 한 애니메이션, 스크롤 위치 제어 등

<br>

## useDebugValue

- 리액트 애플리케이션을 개발하는 과정에서 사용되는 훅
- 디버깅하고 싶은 정보를 이 훅에 사용하면 리액트 개발자 도구에서 볼 수 있다.

```jsx
function useDate() {
  const date = new Date();

  useDebugValue(date, (date) => `현재 시간 : ${date.toISOString()}`);
  return date;
}

export default function App() {
  const date = useDate();
  const [counter, setCounter] = useState(0);

  function handleClick() {
    setCounter((prev) => prev + 1);
  }

  return (
    <div className="App">
      <h1>
        {counter} {date.toISOString()}
      </h1>
      <button onClick={handleClick}>+</button>
    </div>
  );
}
```

- 사용자 정의 훅 내부의 내용에 대한 정보를 남길 수 있는 훅
- 두 번째 파라미터 : 포매팅 함수 -> 이에 대한 값이 변경됐을 때만 호출된다.
- 다른 훅 내부에서만 실행할 수 있다.(컴포넌트 레벨에서 실행하면 작동하지 않는다.)

<br>

## 훅의 규칙(rules-of-hooks)

> 1. 최상위에서만 훅을 호출해야 한다. 반복문, 조건문, 중첩된 함수 내에서 훅을 실행할 수 없다. 이 규칙을 따라야만 컴포넌트가 렌더링될 때마다 항상 동일한 순서로 훅이 호출되는 것을 보장할 수 있다.
> 2. 훅을 호출할 수 있는 것은 리액트 함수 컴포넌트, 혹은 사용자 정의 훅 두 가지 경우 뿐이다. 일반 JS 함수에서는 훅을 사용할 수 없다.

- 훅에 대한 정보 저장은 index 같은 키를 기반으로 구현된다. = 순서에 아주 큰 영향을 받는다.

<br>

```jsx
function Component(){
    const [count, setCount] = useState(0)
    const [required, setRequired] = useState(false)

    useEffect(() => {
        // do something
    },[count, required])
}

// 파이버에는 이렇게 저장된다.
{
    memoizedState: 0, // setCount 훅
    baseState: 0,
    queue: { /* ... */ },
    baseUpdate: null,
    next: { // setRequired 훅
        memoizedState: false,
        baseState: false,
        queue: { /* ... */ },
        baseUpdate: null,
        next: { // useEffect 훅
            memoizedState: {
                tag: 192,
                create: () => {},
                destroy: undefined,
                daps: [0, false],
                next: { /* ... */ },
            },
        baseState: null,
        queue: null,
        baseUpdate: null
        }
    }
}
```

<br>

- 리액트 훅은 파이버 객체의 링크드 리스트의 호출 순서에 따라 저장된다.
  - 각 훅이 파이버 객체 내에서 순서에 의존에 state나 effect의 결과에 대한 값을 저장하고 있기 때문
  - 고정된 순서에 의존해 훅과 관련된 정보를 저장함으로써 이전 값에 대한 비교와 실행이 가능해진다.
- 따라서 항상 훅은 실행 순서를 보장받을 수 있는 컴포넌트 최상단에 선언돼 있어야 한다.

<br>

## 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?

### 사용자 정의 훅

- 복잡하고 반복되는 로직을 사용자 정의 훅으로 간단하게 만들어 관리할 수 있다.
- 리액트 훅의 규칙을 따르고 react-hooks/rules-of-hooks 의 도움을 받기 위해서는 use로 시작하는 이름을 가져야 한다.

<br>

### 고차 컴포넌트

- 컴포넌트 자체의 로직을 재사용하기 위한 방법
- 대표적으로는 React.memo가 있다.
  - 부모 컴포넌트가 렌더링될 때 자식 컴포넌트의 props가 변경되지 않아도 자식 컴포넌트도 함께 렌더링된다.
  - 이렇게 props가 변경되지 않아도 렌더되는 것을 방지하기 위해 만들어졌다.
  - useMemo로도 구현이 가능하나, 목적과 용도가 뚜렷한 memo를 사용하는 편이 좋다.

<br>

예시

사용자 인증 정보에 따라서 인증된 사용자 / 그렇지 않은 사용자에게 다른 컴포넌트를 보여주는 시나리오

```jsx
interface LoginProps {
  loginRequired?: boolean
}

function withLoginComponent<T>(Component: ComponentType<T>){
  return function(props: T & LoginProps){
    const {loginRequired, ...restProps} = props

    if(loginRequired){
      return <>로그인이 필요합니다.</>
    }

    return <Component {...(restProps as T)} />
  }
}

// 원래 구현하고자 하는 컴포넌트를 만들고 withLoginComponent로 감싼다.
// 로그인 여부에 따라 다른 컴포넌트를 렌더링하는 책임을 모두 고차 컴포넌트에 맡긴다.
const Component = withLoginComponent((props : {value: string}) => {
  return <h3>{props.value}</h3>
})

export default function App(){
  const isLogin = true
  return <Component value="text" loginRequired={isLogin}/>
}
```

<br>

주의할 점

- with로 시작하는 이름을 사용해야 한다.
- 부수 효과를 최소화해야 한다.
  - props를 임의로 수정, 추가, 삭제하지 않는다.
- 여러 개의 고차 컴포넌트로 컴포넌트를 감쌀 경우 복잡성이 커진다. -> 최소한으로 사용해야 한다.

<br>

- 사용자 정의 훅

  - 리액트에서 제공하는 훅으로만 공통 로직을 격리할 수 있을 때 사용하는 것이 좋다.
  - 그 자체로는 렌더링에 영향을 미치지 못함 -> 컴포넌트 내부에 미치는 영향을 최소화할 수 있다.
  - 단순히 컴포넌트 전반에 걸쳐 동일한 로직으로 값을 제공하거나 특정한 훅의 작동을 취하게 하고 싶을 떄 사용한다.

<br>

- 고차 컴포넌트
  - 렌더링의 결과물에도 영향을 미치는 공통 로직이라면 고차 컴포넌트를 사용한다.
  - 공통화된 렌더링 로직을 처리하기에 좋다.
  - 복잡성에 주의하여 신중하게 사용해야 한다.
