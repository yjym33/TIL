![alt text](/React/image/모던리액트DeepDive.png) <br>
출처 : https://www.yes24.com/product/goods/123161563

## 상태관리가 필요한 이유

### 리액트 상태 관리의 역사

### FLUX 패턴

> Action - Dispatcher - Model - View

- 단방향 데이터 흐름
  - action : 어떠한 작업을 처리할 액션과 그 액션 발생 시 함께 포함시킬 데이터. 액션 타입과 데이터를 각각 정의해 dispatcher로 보낸다.
  - dispatcher : 액션을 스토어에 보낸다.
  - store : 실제 상태에 따른 값과 상태를 변경할 수 있는 메서드를 가지고 있다.
  - view : 스토어에서 만들어진 데이터를 가져와 화면을 렌더링한다. 입력이나 행위에 따라 상태를 업데이트할 때는 액션을 호출한다.

<br>

### Redux

```elm
module Main exposing (...)

import Browser
import Html exposing (Html, button, div, text)
import Html.Events exposing (onClick)

-- MAIN
main =
Browser.sandbox {init = init, update = update, view = view}

-- MODEL
type alias Model = Int

init : Model
init =
0

-- UPDATE
type Msg
= Increment
| Decrement

update : Msg -> Model -> Model
update msg model =
case msg of
Increment ->
model + 1

    Decrement ->
      model - 1

-- VIEW

view : Model -> Html Msg
view model =
div []
[ button [onClick Decrement] [text "-"]]
, div[][text(String.fromInt model)]
, button [onClick Increment][text "+"]

<div>
  <button>-<button>
  <div>2</div>
  <button>+</button>
</div>
```

- Elm 아키텍처

  - Elm : 웹페이지를 선언적으로 작성하기 위한 언어
  - 리덕스는 이 Elm 아키텍처의 영향을 받아 작성되었다.
  - model : 애플리케이션의 상태. Model을 의미하고, 초깃값은 0
  - view : 모델을 표현하는 HTML. Model을 인수로 받아서 HTML을 표현한다.
  - update : 모델을 수정하는 방식. Increment, Decrement를 선언해 각각의 방식이 어떻게 모델을 수정하는지 나타냈다.

<br>

- 리덕스는 하나의 상태 객체를 스토어에 저장해 두고, 이 객체를 업데이트하는 작업을 디스패치해 업데이트를 수행한다. -> reducer 함수

- reducer 함수 실행으로 웹 애플리케이션 상태에 대한 완전히 새로운 복사본을 반환한 뒤, 애플리케이션에 이 새롭게 만들어진 상태를 전파한다.

<br>

### ContextAPI와 useContext

- Context로 상태를 주입하고, 주입된 상태는 props로 값을 넘겨받지 않아도 사용할 수 있다.

### React Query와 SWR

- HTTP 요청에 특화된 상태 관리 라이브러리
  이후 Recoil, Zustand, Jotai, Valtio 등 많은 라이브러리가 등장했다.

<br>

## 리액트 훅으로 시작하는 상태 관리

### useState와 useReducer

- 약간의 구현상의 차이만 있을 뿐, 두 훅 모두 지역 상태 관리를 위해 만들어졌다.
- 훅을 사용할 때마다 컴포넌트 별로 초기화되므로 컴포넌트에 따라 서로 다른 상태를 가질 수밖에 없다.(지역 상태)

<br>

### useState의 상태를 바깥으로 분리하기

```jsx
export type State = { counter: number }

// 상태를 아예 컴포넌트 밖에 선언
let state : State = {
  counter: 0,
}

// geter
export function get(): State {
  return state
}

// useState와 동일하게 구현하기 위해 게으른 초기화 함수나 값을 받을 수 있게 함
type initializer<T> = T extends any ? T | ((prev: T) => T) : never

// setter
export function set<T>(nextState: Initializer<T>){
  state = typeof nextState === 'function' ? nextState(state) : nextState
}

// Counter
function Counter(){
  const state = get()

  function handleClick(){
    set((prev : State) => ({ counter: prev.counter + 1 }))
  }

  return (
    <>
      <h3>{state.counter}</h3>
      <button onClick={handleClick}>+</button>
    </>
  )
}
```

<br>

- 위와 같은 방법으로는 컴포넌트가 리렌더링되지 않는다.
- 리렌더링을 일으키기 위한 방법
  - useState, useReducer의 반환값 중 두 번째 인수가 어떻게든 호출된다.
  - 부모 컴포넌트가 리렌더링되거나 해당 컴포넌트가 다시 실행되어야 한다.

<br>

```jsx
function Counter1() {
  const [count, setCount] = useState(state);

  function handleClick() {
    set((prev: State) => {
      const newState = { counter: prev.counter + 1 };
      setCount(newState);
      return newState;
    });
  }

  return (
    <>
      <h3>{state.counter}</h3>
      <button onClick={handleClick}>+</button>
    </>
  );
}

function Counter2() {
  const [count, setCount] = useState(state);

  function handleClick() {
    set((prev: State) => {
      const newState = { counter: prev.counter + 1 };
      setCount(newState);
      return newState;
    });
  }

  return (
    <>
      <h3>{state.counter}</h3>
      <button onClick={handleClick}>+</button>
    </>
  );
}
```

- 외부에 선언한 set을 실행해 외부 상태값 업데이트 + useState의 두번째 인수로 렌더
- 외부에 상태가 있음에도 불구하고 함수 컴포넌트의 렌더링을 위해 useState가 또 존재하는 비효율적인 구조
- 같은 상태를 바라보지만 반대쪽 컴포넌트에는 렌더링되지 않는다.
  - 다른쪽 컴포넌트의 리렌더링을 일으키기 위해서는 클릭 이벤트가 발생해야 한다.
- 외부 상태를 참조하고 렌더링까지 자연스럽게 일어나게 하는 방법
  - 컴포넌트 외부 어딘가에 상태를 두고 여러 컴포넌트가 같이 쓸 수 있어야 한다.
  - 컴포넌트가 상태 변화를 알아챌 수 있어야 하고, 상태가 변화될 때마다 리렌더링이 일어나야 한다.(상태를 참조하는 모든 컴포넌트에서 동일해야 한다.)
  - 상태가 객체인 경우, 감지하지 않는 값이 변하면 리렌더링이 발생하면 안된다.

<br>

```jsx
type Initializer<T> = T extends any ? T | ((prev: T) => T) : never

type Store<State> = {
  get: () => State // 항상 새롭게 값을 가져오기 위해 시도한다.
  set: (action : Initializer<State>) => State
  subscribe: (callback: () => void) => () => void // store의 변경을 감지하고 싶은 컴포넌트들이 callback을 등록. store는 값이 변경될 때마다 모든 callback을 실행한다.
}

export const createStore = <State extends unknown>(
  initialState: Initializer<State>,
): Store<State> => {
  let state = typeof initialState !== 'function' ? initialState : initialState()

  // callbacks는 자료형에 관계없이 유일한 값을 저장할 수 있는 Set 사용
  const callbacks = new Set<() => void>()
  const get = () => state
  const set = (nextState: State | ((prev: State) => State)) => {
    state =
      typeof nextState === 'function'
        ? (nextState as (prev: State) => State)(state)
        : nextState
  callbacks.forEach((callback) => callback())
  return state
  }

  const subscribe = (callback: () => void) => {
    callbacks.add(callback)
    // 클린업 실행 시 이를 삭제해서 반복적으로 추가되는 것을 막는다.
    return () => {
      callbacks.delete(callback)
    }
  }

  return { get, set, subscribe }
}

// store의 값을 참조하고, 이 값에 변화에 따라 컴포넌트 렌더링을 유도할 사용자 정의 훅
export const useStore = <State extends unknown>(store: Store<State>) => {
  const [state, setState] = useStaet<State>(() => store.get()) // 렌더링 유도

  useEffect(() => {
    const unsubscribe = store.subscribe(() => {
      setState(store.get())
    })
    return unsubscribe
  },[store]) // store 값이 변경될 때마다 state의 값이 변경되는 것을 보장받을 수 있다.

  return [state, store.get] as const
}

```

- 스토어가 객체라면 어떤 값이 바뀌든지 간에 리렌더링이 일어난다. 아래와 같이 수정할 수 있다.

```jsx
export const useStoreSelector = <State extends unknown, Value extends unknown>(
  store: Store<State>,
  selector: (state: State) => Value // store 상태에서 어떤 값을 가져올 지 정의하는 함수
) => {
  const [state, setState] = useState(() => selector(store.get()))

  useEffect(() => {
    const unsubscribe = store.subscribe(() => {
      const value = selector(store.get())
    })

    return unsubscribe
  }, [store, selector])

  return state
}
```

- 이제 store가 객체로 구성되어 있어도 필요한 값만 select해서 사용할 수 있다.
- 두 번째 인수인 selector를 컴포넌트 밖에 선언하거나, useCallback을 사용해 참조를 고정시켜야 한다.
  - 컴포넌트 내에 selector 함수를 생성하고 useCallback으로 감싸두지 않으면 컴포넌트가 리렌더링될 때마다 함수가 계속 재생성된다.

<br>

### useState와 useContext 동시에 사용하기

- useStore, useStoreSelector 훅을 활용하는 방법 -> 반드시 하나의 스토어만 가지게 된다.
- 스토어의 구조는 동일하되 여러 개의 서로 다른 데이터를 공유해 사용하고 싶을 때 ?
  - Context를 활용해 해당 스토어를 하위 컴포넌트에 주입한다.
  - 자신이 주입한 스토어에 대해서만 접근할 수 있게 된다.

```jsx
// Context를 생성하면 자동으로 스토어도 함께 생성한다
export const CounterStoreContext = createContext<Store<CounterStore>>(
  createStore<CounterStore>({ count: 0, text: 'hello' }),
)

export const CounterStoreProvider = ({
  initialState,
  children
} : PropsWithChildren<{
  initialState: CounterStore
}>) => {
  const storeRef = useRef<Store<CounterStore>>() // Provider로 넘기는 props가 불필요하게 변경돼서 리렌더링되는 것을 막기 위해

  // 스토어를 생성한 적이 없다면 최초 한 번 생성한다
  if(!storeRef.current){
    storeRef.current = createStore(initialState)
  }

  return (
    <CounterStoreContext.Provider value={storeRef.current}>
      {children}
    </CounterStoreContext.Provider>
  )
}

export const useCounterContextSelector = <State extends unknown>(
  selector: (state: CounterState) => State,
) => {
  const store = useContext(CounterStoreContext) // Context.Provider에서 제공된 스토어를 찾게 만든다.

  const subscription = useSubscription(
    useMemo(
      () => ({
        getCurrentValue: () => selector(store.get()),
        subscribe: store.subscribe
      })
    ,[store, selector])
  )

  return [subscription, store.set] as const
}

// 사용
const ContextCounter = () => {
  const id = useId()
  const [counter, setCounter] = useCounterContextSelector(
    useCallback((state: CounterStore) => state.count, []),
  )

  function handleClick(){
    setStore((prev) => ({...prev, count: prev.count + 1}))
  }

  useEffect(() => {
    console.log(`${id} Counter Rendered`)
  })

  return (
    <div>
      {counter} <button onClick={handleClick}>+</button>
    </div>
  )
}
```

- 서로 다른 context를 바라보게 할 수 있다.

```jsx
export default function App() {
  return (
    <>
      <ContextCounter /> // 0
      <CounterStoreProvider initialState={{ count: 10, text: "hello" }}>
        <ContextProvider /> // 10
        <CounterStoreProvider initialState={{ count: 20, text: "welcome" }}>
          <ContextProvider /> // 20
        </CounterStoreProvider>
      </CounterStoreProvider>
    </>
  );
}
```

<br>

## 상태 관리 라이브러리 Recoil, Jotai, Zustand 살펴보기

### Recoil, Zustand, Jotai, Valtio에 이르기까지

- 훅이라는 새로운 패러다임의 등장에 따라 훅을 활용해 상태를 가져오거나 관리할 수 있는 다양한 라이브러리가 등장했다.

- 기존의 리덕스 같은 라이브러리와 다르게 훅을 활용해 작은 크기의 상태를 효율적으로 관리한다.

- 페이스북이 만든 상태 관리 라이브러리 Recoil

Facebook에서 개발한 상태 관리 라이브러리
리액트에서 훅의 개념으로 상태 관리를 시작한 최초의 라이브러리

RecoilRoot

Recoil 애플리케이션을 감싸는 최상위 컴포넌트
Recoil 에서 생성되는 상태값을 저장하기 위한 스토어를 생성함.

import { RecoilRoot } from 'recoil';

```jsx
import { RecoilRoot } from "recoil";

function App() {
  return <RecoilRoot>{/* 하위 컴포넌트들 */}</RecoilRoot>;
}
```

- Recoil의 상태값은 RecoilRoot로 생성된 Context의 스토어에 저장된다.
- 스토어의 상태값에 접근할 수 있는 함수들이 있고, 이 함수를 활용해 상태값에 접근하거나 상태값을 변경할 수 있다.
- 값의 변경이 발생하면 이를 참조하고 있는 하위 컴포넌트에 모두 알린다.

### atom

- 상태를 나타내는 Recoil의 최소 상태 단위
- atom은 key값을 필수로 가지며, 이 키는 다른 atom과 구별하는 식별자가 되는 필수 값이다. (유일한 값)

```jsx
import { atom } from "recoil";

const countState = atom({
  key: "countState", // 고유 식별자
  default: 0, // 초기값
});
```

<br>

### useRecoilValue

- atom의 값을 읽어오는 훅, 값이 변경될 때마다 리렌더링 한다.

```jsx
import { useRecoilValue } from "recoil";

function Counter() {
  const count = useRecoilValue(countState);

  return <div>Count: {count}</div>;
}
```

<br>

### useRecoilState

- useState와 유사하게 값을 가져오고, 이 값을 변경할 수 있는 훅

```jsx
import { useRecoilState } from "recoil";

function Counter() {
  const [count, setCount] = useRecoilState(countState);

  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
    </div>
  );
}
```

- RecoilRoot를 선언해 하나의 스토어를 만들고, atom이라는 상태 단위를 RecoilRoot 에서 만든 스토어에 등록한다. atom은 Recoil에서 관리하는 작은 상태 관리이며, 각 값은 고유한 key를 바탕으로 구별된다. 그리고 컴포넌트는 Recoil에서 제공하는 훅을 통해 atom의 상태 변화를 구독하고, 값이 변경되면 forceUpdate같은 기법을 통해 리렌더링을 실행해 최신 atom값을 가져오게 된다.
