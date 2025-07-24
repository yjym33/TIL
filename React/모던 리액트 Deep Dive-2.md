![alt text](/React/image/모던리액트DeepDive.png) <br>
출처 : https://www.yes24.com/product/goods/123161563

## JSX

- 트랜스파일러를 거쳐야 JS 런타임이 이해할 수 있는 코드로 변환된다.
- XML 스타일의 트리 구문을 작성하는 데 많은 도움을 주는 문법

### 정의

- JSXElement, JSXAttributes, JSXChildren, JSXString 라는 4가지 컴포넌트를 기반으로 구성되어 있다.

<br>

- JSXElement
  - HTML의 element와 비슷한 역할. 다음과 같은 형태 중 하나여야 한다.
  - JSXOpeningElement, JSXClosingElement : <JSXElement></JSXElement>
  - JSXSelfClosingElement : <JSXElement/>
  - JSXFragment : <>JSXChildren</>
  - JSXElementName : JSXElement의 요소 이름으로 가능한 것. 이름으로 가능한 것은 다음과 같다.
    - JSXIdentifier
      - JSX 내부에서 사용할 수 있는 식별자를 의미한다.
      - JS 식별자 규칙과 동일하다.(숫자로 시작하거나 $와 \_외의 다른 특수문자로 시작할 수 없다.)
    - JSXNamespacedName : :을 통해 서로 다른 식별자를 이을 수 있다.(한 개만) <foo:bar></foo:bar>
    - JSXMemberExpression : .을 통해 서로 다른 식별자를 이을 수 있다.(여러 개 가능) ```<foo.bar.baz></foo.bar.baz>
  - JSXAttributes : 부여할 수 있는 속성. 모든 경우에서 필수값이 아니다.
    - JSXSpreadAttributes : 전개 연산자와 동일한 역할
    - JSXAttribute : 속성을 나타내는 키와 값으로 짝을 이루어 표현
      - JSXAttributeName : 속성의 키 값. JSXIdentifier와 JSXNamespacedName 가능 <foo.bar foo:bar="baz"></foo.bar>
      - JSXAttributeValue : 속성의 키에 할당할 수 있는 값.
    - JSXChildren : 자식 값. 트리 구조 표현을 위해 부모 자식 관계를 나타낼 수 있다.
    - JSXStrings : JSXAttributeValue, JSXText는 HTML과 JSX 사이에 복사 붙여넣기를 쉽게 할 수 있도록 설계되어 있다.

<br>

### JSX는 JS에서 어떻게 변환되는가?

- @babel/plugin-transform-react-jsx 플러그인을 통해 변환된다.
- 변환결과
  - JSXElement를 첫 번째 인수를 선언해 요소를 정의한다.
  - 옵셔널인 JSXChildren, JSXAttributes, JSXStrings는 이후 인수로 넘겨주어 처리한다.

```jsx
// 변환 전
const ComponentA = <A required={true}>Hello World</A>;
const ComponentB = <>Hello World</>;
const ComponentC = (
  <div>
    <span>Hello World</span>
  </div>
);

// 변환 후
var ComponentA = React.createElement(
  A,
  {
    required: true,
  },
  "Hello World"
);

var ComponentB = React.createElement(React.Fragment, null, "Hello World");
var ComponentC = React.createElement(
  "div",
  null,
  React.createElement("span", null, "Hello World")
);
```

- 결과를 활용해 JSXElement만 다르고 JSXAttributes, JSXChildren이 완전 동일할 때 중복 코드를 최소화할 수 있다.

<br>

```jsx

// props 여부에 따라 children 요소가 달라지는 경우
// 1) 삼항연산자로 처리(bad)
import {createElement, propsWithChildren} from 'react'

function TextOrHeading({
  isHeading,
  children
}){
  return isHeading ? (
    <h1 classname="text">{children}</h1>
  ) : (
    <span classname="text">{children}</span>
  )
}

// 2) 간결하게 처리(good)
import {createElement} from 'react'

function TextOrHeading({
  isHeading,
  children
}) : {
  return createElement({isHeading}){
    isHeading ? 'h1' : 'span',
    {className: 'text'},
    children
  }
}

```

<br>

## 가상 DOM과 리액트 파이버

### DOM과 브라우저 렌더링 과정

<br>

#### 브라우저 렌더링

- 주소를 방문해 HTML 파일을 다운로드 한다.
- HTML을 파싱해 DOM을 만들고, CSS를 파싱해 CSSOM을 만든다.
- DOM 노드를 순회하며 분석한다.
- 분석한 DOM 노드를 대상으로 해당 노드에 대한 CSSOM 정보를 찾고 발견한 CSS 정보를 노드에 적용한다.
  - 레이아웃 : 화면 어느 좌표에 나타나여 하는지 계산하는 과정
  - 페인팅 : 레이아웃 단계를 거친 노드에 색과 같은 실제 유효한 모습을 그리는 과정

<br>

#### 가상 DOM의 탄생 배경

- SPA : 하나의 페이지에서 모든 작업이 일어난다. -> 계속해서 요소의 위치를 재계산하기 때문에 비용이 커진다.
- 리액트가 관리하는 가상의 DOM
- 웹페이지가 표시해야 할 DOM을 메모리에 일단 저장 -> 준비 완료됐을 떄 실제 브라우저 DOM에 반영한다.
- 메모리에서 계산하는 과정을 거쳐 렌더링 과정을 최소화 한다.

<br>

#### 리액트 파이버

- 개념

  - 리액트에서 관리하는 JS 객체, 파이버 재조정자(fiber reconciler)가 관리한다.
  - 가상 DOM과 실제 DOM을 비교해 변경 사항을 수집하고, 차이가 있으면 변경과 관련된 정보를 가지고 있는 파이버를 기준으로 화면에 렌더링을 요청한다.
  - 재조정(reconciliation) : 리액트에서 어떤 부분을 새롭게 렌더링 해야하는 지 두 DOM 을 비교하는 작업(알고리즘)

- 역할

  - 작업을 작은 단위로 분할하고 쪼갠 뒤, 우선순위를 매긴다.
  - 이러한 작업을 일시 중지하고 나중에 다시 시작할 수 있다.
  - 이전에 했던 작업을 다시 재사용하거나 필요하지 않은 경우에 폐기할 수 있다.
  - 이 모든 과정은 비동기로 일어난다.

- 구성

  - 하나의 작업단위로 구성된다.
  - 리액트는 작업 단위를 하나씩 처리하고 finishedWork()라는 작업으로 마무리한다.
  - 렌더 단계 : 모든 비동기 작업을 수행 + 파이버의 작업, 우선순위를 지정하거나 중지하거나 버린다.
  - 커밋 단계 : DOM에 실제 변경 사항을 반영하는 commitWork()가 실행된다. (동기식, 중단될수 없다.)

- 특징

  - 컴포넌트가 최초로 마운트되는 시점에 생성되어 가급적 재사용된다.
  - 하나의 element에 하나가 생성되는 1:1의 관계 -> 1:1로 매칭된 정보를 가지고 있는 것이 Tag

- 속성

  - stateNode : 파이버 자체에 대한 참조, 참조를 바탕으로 리액트는 파이버와 관련된 상태에 접근한다.
  - child, sibling, return

        - 파이버 간의 관계 개념을 나타낸다.
        - 리액트 컴포넌트 트리처럼 파이버도 트리 형식을 가진다. -> 트리 형식을 구성하는데 필요한 정보가 이 속성 내부에 정의된다.
        - 하나의 child만 존재한다. -> 여러 개면 첫번째 자식의 참조로 구성된다.

```jsx
// 구조
<ul>
  <li>하나</li>
  <li>둘</li>
  <li>셋</li>
</ul>;

// 파이버
const l3 = {
  return: ul, // 부모
  index: 2, // 형제 중 순서
};

const l2 = {
  sibling: 13,
  return: ul,
  index: 1,
};

const l1 = {
  sibling: 12,
  return: ul,
  index: 0,
};

const ul = {
  //...
  child: l1,
};
```

<br>

- index : 여러 sibling 사이에서 자신의 위치가 몇 번째인지

- pendingProps : 아직 작업을 미처 처리하지 못한 props

- memoizeProps : pendingProps를 기준으로 렌더링 완료 이후 pendingProps를 memoizedProps로 저장해 관리

- updateQueue : 상태 업데이트, 콜백 함수, DOM 업데이트 등 필요한 작업을 담아두는 큐

<br>

```jsx

type UpdateQueue = {
    first : Update | null
    last : Update | null
    hasForceUpdate : boolean
    callbackList : null | Array<Callback> //setState로 넘긴 콜백 목록
}

```

- memoizedState : 함수 컴포넌트의 훅 목록 저장

- alternate : 반대편 트리 파이버

<br>

> 이렇게 생성된 파이버는 state 변경, 생명주기 메서드 실행, DOM 변경 필요한 시점 등에 실행된다.

> 리액트의 핵심 원칙은 UI를 문자열, 숫자, 배열과 같은 값으로 관리한다는 것이다.

<br>

### 리액트 파이버 트리

- 현재 모습을 담은 파이버 트리와 작업중인 상태를 나타내는 worklnProgress 트리 두개가 존재한다.
- 더블 버퍼링 : 리액트 파이버 작업이 끝나면 단순히 포인터만 변경해 worklnProgress 트리를 현재 트리로 바꾼다. (커밋 단계에서 수행)
  1. current tree를 기준으로 작업을 시작
  2. 업데이트가 발생하면 파이버는 새로운 worklnProgress 트리를 빌드한다.
  3. 빌드가 끝나면 다음 렌더링에 이 트리를 사용한다.
  4. worklnProgess 트리가 UI에 렌더링되어 반영 완료되면 current가 worklnProgess로 변경된다.

<br>

### 파이버의 작업 순서

- 파이버 노드 생성

  1. beginWork() 실행 : 더이상 자식이 없는 파이버를 만날 때 까지 트리 형식으로 시작
  2. completeWork() 실행 : 파이버 작업 완료
  3. 형제가 있다면 형제로 넘어감
  4. 2, 3 이 모두 끝나면 return으로 돌아가 완료

- 업데이트가 필요하면 worklnProgress 트리를 빌드, 되도록 기존 파이버 업데이트

<br>

## 클래스 컴포넌트와 함수 컴포넌트

### 클래스 컴포넌트

```jsx
import React from 'react'

interface SampleProps {
  required?: boolean
  text: string
}

interface SampleState {
  count : number
  isLimited?: boolean
}

class SampleComponent extends React.Component<SampleProps, SampleState>{
  private constructor(props: SampleProps){
    super(props)
    this.state = {
      count : 0
      isLimited: false,
    }
  }

  private handleClick = () => {
    const newValue = this.state.count + 1
    this.setState({count : newValue, isLimited: newValue >= 10})
  }

  public render(){
    const {
      props : {required, text},
      state : {count, isLimited}
    } = this

    return (
      <h2>
        Sample Component
        <div>{required ? '필수' : '필수 아님'}</div>
        <div>문자: {text}</div>
        <div>count : {count}</div>
        <button onClick={this.handleClick} disabled={isLimited}>증가</button>
      </h2>
    )
  }
}
```

<br>

### 클래스 컴포넌트의 생명주기 메서드

- 생명주기 메서드가 실행되는 시점은 크게 3가지로 나뉜다.

  - mount : 컴포넌트가 마운팅 되는 시점
  - update : 이미 생성된 컴포넌트의 내용이 변경되는 시점
  - unmount : 컴포넌트가 더 이상 존재하지 않는 시점

- render()

  - 클래스 컴포넌트의 유일한 필수값
  - 컴포넌트가 UI를 렌더링하기 위해 쓰인다. mount, update에서 일어난다.

  - 항상 순수해야 하며 부수 효과가 없어야 한다.
    - 내부에서 this.setState를 호출해서는 안 된다.

- componentDidMount()

  - 클래스 컴포넌트가 마운트되고 준비되는 즉시 실행된다.
  - this.setState로 state 값 변경이 가능하다. 하지만 성능 문제를 일으킬 수 있으니 되도록 setState는 생성자에서 하는 것이 좋다.

- componentDidUpdate()

  - 컴포넌트 업데이트가 일어난 후 바로 실행된다.
  - 일반적으로 state, props의 변화에 따라 DOM을 업데이트 하는 등에 쓰인다.

- componentWillUnmount()

  - 컴포넌트가 언마운트되거나 더 이상 사용되지 않기 직전에 호출된다.
  - this.setState를 호출할 수 없다.
  - 이벤트를 지우거나, API 호출을 취소하거나, 타이머를 지우는 등 작업에 유용하다.

- shouldComponentUpdate()

  - this.setState()가 호출되면 리렌더링 -> 이 메서드를 활용하면 컴포넌트에 영향 받지 않는 변화에 대해 정의할 수 있다.

<br>

```jsx

shouldComponentUpdate(nextProps: Props, nextState: State){
// true인 경우, 즉 props의 title이 같지 않거나 state의 input이 같지 않은 경우에는
// 컴포넌트 업데이트. 이외의 경우에는 업데이트하지 않는다.
return this.props.title !== nextProps.title || this.state.input !== nextState.input
}

```

<br>

- static getDerivedStateFromProps()
  - componentWillReceiveProps를 대체할 수 있는 메서드
  - render()를 호출하기 직전에 호출된다.
  - static으로 선언되어 this에 접근할 수 없고, 반환되는 객체는 내용이 모두 state로 들어간다.
  - 모든 render() 실행 시 호출된다.

<br>

```jsx
static getDerivedStaetFromProps(nextProps: Props, prevState: State){
  // 다음에 올 props를 바탕으로 현재의 state를 변경하고 싶을 때 사용한다.
  if(props.name !== state.name){
    // state가 이렇게 변경된다.
    return {
      name: props.name
    }

    // state에 영향을 미치지 않는다
    return null
  }
}
```

<br>

- getSnapShotBeforeUpdate()
  - componentWillUpdate를 대체할 수 있는 메서드
  - 클래스 컴포넌트에서만 사용이 가능하다.
  - DOM이 업데이트되기 직전에 호출된다.
  - 반환되는 값은 componentDidUpdate로 전달된다.
  - DOM이 렌더링 되기 전 윈도우 크기를 조절하거나 스크롤 위치를 조정하는 등 작업 처리에 유용하다.

<br>

```jsx
getSnapShotBeforeUpdate(prevProps: Props, prevState: State){
  // props로 넘겨받은 배열의 길이가 이전보다 길어지면 현재 스크롤 높이값을 반환
  if(prevProps.list.length < this.props.list.length){
    const list = this.listRef.current;
    return list.scrollHeight - list.scrollTop;
  }
  return null;
}

componentDidUpdate(prevProps: Props, prevState: State, snapshot: Snapshot){
  // getSnapshotBeforeUpdate로 넘겨받은 값은 snapshot으로 접근 가능
  // 값이 있다면 스크롤 위치 재조정
  if(snapshot !== null){
    const list = this.listRef.current
    list.scrollTop = list.scrollHeight - snapshot;
  }
}
```

<br>

- static getDerivedStateFromError()

  - 에러 상황에서 실행되는 메서드. 자식 컴포넌트에서 에러 발생했을 때 호출된다.
  - 클래스 컴포넌트에서만 사용 가능하다.
  - 반드시 state 값을 반환해야 하고, 렌더링 과정에서 호출되기 때문에 부수 효과가 발생해서는 안된다.

- componentDidCatch
  - 자식 컴포넌트에서 에러 발생했을 때 호출된다.
  - 클래스 컴포넌트에서만 사용 가능하다.
  - getDerivedStateFromError에서 에러를 잡고 state를 결정한 이후에 실행된다.
  - getDerivedStateFromError에서 하지 못했던 부수 효과를 수행할 수 있다.(로깅 등)
  - 개발 모드 : 에러 발생하면 window까지 전파 / 프로덕션 모드 : componentDidCatch로 잡히지 않은 에러만 window까지 전파

<br>

#### 한계

- 데이터 흐름 추적이 어렵다 : 여러 메서드에서 state 업데이트가 가능함
- 내부 로직 재사용이 어렵다 : 공통 로직은 고차 컴포넌트나 상속으로 관리 -> 흐름 파악이 어렵다.
- 기능이 많아질수록 컴포넌트의 크기가 커진다.
- 함수에 비해 어렵다.
- 코드 크기 최적화가 어렵다. : 번들 크기를 줄이는것이 어렵다.
- 핫 리로딩에 불리하다 : 최초 렌더링 시에 인스턴스 생성하고 내부에서 state값 관리 -> 인스턴스 내부 render를 수정하려면 인스턴스를 새로 만들어야 한다.

<br>

## 함수 컴포넌트

### 클래스 컴포넌트 vs 함수 컴포넌트

- 생명주기 메서드의 부재

  - 클래스 컴포넌트의 생명주기 메서드가 함수 컴포넌트에는 존재하지 않는다
    - 클래스 컴포넌트 : render 메서드가 있는 React.Component를 상속받아 구현하는 클래스
    - 함수 컴포넌트 : props를 받아 단순히 리액트 요소를 반환하는 함수
  - 함수 컴포넌트는 useEffect를 사용해 componentDidMount, componentDidUpdate, componentWillUnmount를 비슷하게 구현할수 있다.

- 렌더링된 값
  - 함수 : 렌더링된 값을 고정한다. / 클래스 컴포넌트 : 렌더링된 값을 고정하지 못한다.
  - 클래스 : props의 값을 항상 this로 부터 가져온다. -> this가 가리키는 인스턴스 멤버는 변경 가능하기 떄문에 변경된 값을 읽을 수 있다.
  - 함수 : props를 인수로 받는다. -> props는 변경할 수 없기 때문에 그대로 사용한다.

<br>

## 리액트의 렌더링이란?

- 리액트 애플리케이션 트리 안에 있는 모든 컴포넌트들이 현재 자신들이 가지고 있는 props와 state 값을 기반으로 어떻게 UI를 구성하고 이를 바탕으로 어떤 DOM 결과를 브라우저에 제공할 것인지 계산하는 일련의 과정

<br>

### 렌더링이 일어나는 이유

1. 최초 렌더링

   - 사용자가 처음 애플리케이션 진입 후 정보를 제공하기 위해 최초 렌더링 수행

2. 리렌더링
   - 최초 렌더링이 발생한 후 발생하는 모든 렌더링
   - 발생 조건
     - 클래스 컴포넌트의 setState가 실행되는 경우
     - 클래스 컴포넌트의 forceUpdate가 실행되는 경우
     - 함수 컴포넌트의 useState()의 두 번째 배열 요소인 setter가 실행되는 경우
     - 함수 컴포넌트의 useReducer()의 두 번째 배열 요소인 dispatch가 실행되는 경우
     - 컴포넌트의 key props가 변경되는 경우
     - props가 변경되는 경우
     - 부모 컴포넌트가 렌더링될 경우

<br>

### 렌더링 프로세스

- 컴포넌트 루트에서부터 아래쪽으로 내려가며 업데이트가 필요하다고 지정되어 있는 모든 컴포넌트를 찾는다.
- 업데이트가 필요한 컴포넌트를 발견하면 클래스는 render(), 함수 컴포넌트는 FunctionComponent() 그 자체를 호출한 뒤 결과물을 저장한다.
- 렌더링 결과물 JSX -> 컴파일되면서 React.createElement() 호출 -> UI 구조를 설명하는 일반 JS 객체 반환
- 렌더링 결과물 수집 후 가상 DOM과 비교해 실제 DOM에 반영하기 위한 모든 변경 사항 수집
- 모든 변경사항을 DOM에 제공해 변경된 결과물이 보인다.

<br>

### 렌더와 커밋

- 렌더 단계

  - 컴포넌트를 렌덜이하고 변경 사항을 개선하는 모든 작업
  - 컴포넌트를 실행해 결과와 이전 가상 DOM을 비교하는 과정을 거쳐 변경이 필요한 컴포넌트를 체크하는 단계
  - 비교하는 것 : type, props, key -> 하나라도 변경되면 변경이 필요한 것으로 체크된다.

- 커밋 단계
  - 렌더 단계의 변경 사항을 실제 DOM에 적용해 사용자에게 보여주는 과정
  - 이 단계가 끝나야 브라우저 렌더링이 발생한다.
  - DOM이 업데이트 된 후 만들어진 DOM 노드와 인스턴스를 가리키도록 리액트 내부 참조를 업데이트 한다.
  - 이후 componentDidMount, componentDidUpdate, / useLayoutEffect 훅을 호출한다.

<br>

> 리액트의 렌더링이 일어난다고 해서 무조건 DOM 업데이트가 일어나는 것은 아니다.
> 렌더 단계에서 계산 후 변경 사항이 없다면 커밋 단계는 생략될수 있다.

<br>

- 렌더링은 항상 동시적으로 작동했으나, 18버전부터 동시성 렌더링이 도입되었다.
