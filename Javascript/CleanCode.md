# 변수

## var 지양

- var : 함수 스코프
  아래 코드에서 에러 발생 X
  ```jsx
  var name = "이름";
  var name = "이름1";
  name = "이름2";
  ```
- let, const : 블록 스코프 → 재할당 불가 → 안전하게 코드 작성 가능
  아래 코드에서 에러 발생
  ```jsx
  let name = "이름";
  let naem = "이름1";
  ```
- const : 재할당 금지 But 객체 조작 가능
  ```jsx
  const person = { name: "kim" };
  person = { name: "lee" }; // Error
  person.name = "lee"; // Ok
  ```
- 권장
  var < let < const

## 전역 공간 사용 최소화

- 전역 공간

  - Window : **브라우저** 환경에서의 최상위 전역 공간
  - global : **Node.js** 환경에서의 최상위 전역 공간

  → 어디서나 접근 가능 (다른 파일에서도 해당 전역변수에 접근 가능)

- 사용 권장 목록
  - IIFE (즉시 실행 함수)
  - Module
  - Closeure
  - let & const

## 임시변수 제거

- 바로반환
  - 비권장 코드
    ```jsx
    function getElements() {
      const result = {};
      result.title = document.querySelector(".title");
      return result;
    }
    ```
  - 권장 코드
    ```jsx
    function getElements() {
    	return {
    		title = document.querySelector('.title')
    	}
    }
    ```
- 함수 나누기
  - 비권장 코드
    ```jsx
    function getDateTime(targetDate) {
    	const month = targetDate.getMonth();
    	const day =
    	const hour =

    	month =
    	day =
    	hour =

    	return { month, day, hour }
    }
    ```
  - 권장 코드
    ```jsx
    function getDateTime(targetDate) {
    	return {
    		month: ,
    		day: ,
    		hour:
    	}
    }

    // 후 가공 함수
    function getDateTime() {
    	const currentDateTime = getDateTime(new Date());

    	return {
    		month: ** + "분 전",
    		day: ,
    		hour:
    	}
    }
    ```
- 이유 (하지 않을 경우)
  - 명령형으로 가득찬 로직이 된다.
  - 디버깅이 힘들다.
  - 추가적인 코드의 유혹이 생긴다. → 유지보수가 힘들어 진다.
- 해결 방안
  - 함수 나누기
  - 바로 반환
  - 고차함수 사용 (map, filter, reduce 등)
  - 선언형 프로그래밍으로 작성

## 호이스팅 주의

- 호이스팅
  런타임시(동작할 때), 선언을 회상단으로 끌어올려 주는 것
  → 코드를 작성했을 때, 예측하지 못한 실행 결과가 노출
- 해결 방안
  - var 사용 지양
  - 선언과 동시에 할당
  - 함수 표현식 사용 : 함수 작성시 const로 선언 후 할당하는 방식
    ```jsx
    const sum = function() { ... }
    ```

---

# 타입

## 타입 검사

- javascript는 동적으로 변하는 언어이기 때문에 type도 동적이다. → 타입 검수가 힘들다.
- `typeof`
  ```jsx
  function myFunction() { }
  class myClass [ }
  cosnt str = new String('문자열')

  typeof '문자열'        // 'string'
  typeof true          // 'boolean'
  typeof undefined     // 'undefined'
  typeof 123           // 'number'
  typeof Symbol()      // 'symbol'
  typeof myFunction    // 'function'
  typeof myClass       // 'function
  typeof str           // 'object'
  typeof null          // 'object' : 언어적 오류 (javascript에서 인정)
  ```
  - Primitive 타입과 달리 Reference 타입은 typeof로 구별하기 힘들다.
- `instanceof`
  ```jsx
  function Person(name, age) {
  	this.name = name;
  	this.age = age;
  }

  const poco = new Person('poco', 100)
  poco instanceof Person   // true

  const p = {
  	name: 'poco',
  	age: 100
  }
  p instanceof Person.     // false
  ```
  - Reference 타입이기 때문에 최상위는 모두 Object 타입이다.
    ```jsx
    const arr = []
    const func = function() { }
    const date = new Date()

    arr instanceof Array       // true
    func instanceof function   // true
    date instanceof Date       // true

    arr instanceof Object      // true
    func instanceof Object     // true
    date instanceof Object     // true

    ```
- `Obejct.prototype`
  ```jsx
  Object.prototype.toString.call('')                // [object String]
  Object.prototype.toString.call(new String('')     // [object String]
  Object.prototype.toString.call(arr)               // [object Array]
  Object.prototype.toString.call(func)              // [object Function]
  Object.prototype.toString.call(date)              // [object Date]
  ```

## undefined vs. null

- null : 값이 없음을 명시적으로 표현
  ```jsx
  !null; // true
  !!null; // false

  null === false; // false

  !null === true; // true

  null + 123; // 123 숫자적으로는 0으로 본다.
  ```
- undefined : 값이 정의되지 않음
  ```jsx
  undefined + 10; // NaN
  !undefined; // true

  undefined == null; // true
  undefined === null; // fasle
  !undefined === !null; // true
  ```

## eqeq 줄이기

- == : 동등연산자
  ```jsx
  "1" == 1; // true
  1 == true; // true
  ```
- === : 엄격한 동등연산자
  ```jsx
  "1" === 1; // false
  1 === true; // false
  ```

## 형변환 주의

- 느슨한 검사 → 암묵적 형 번환
  ```jsx
  "1" == 1; // true
  1 == true; // true
  0 == false; // true

  11 + " 문자와 결합"; // '11 문자와 결합'
  !!"문자열"; // true
  !!""; // false
  ```
- 명시적 형 변환
  ```jsx
  String(11 + " 문자와 결합"); // '11 문자와 결합'
  Boolean("문자열"); // true
  Boolean(""); // false
  Number("11"); // 11
  parseInt("9.999", 10); // 9
  ```

## isNaN

```jsx
isNaN(123);
typeof 123 !== "number";
```

```jsx
isNaN(123 + "문자열"); // true  : 느슨한 검사
Number.isNaN(123 + "문자열"); // false : 엄격한 검사
```

---

# 경계

## min - max

- 최소값, 최대값을 상수값으로 지정하고 다루자.
- 포함여부를 결정 (이상-초과/이하-미만)
- 상수값의 네이밍에 포함 여부를 포함한다.

## begin - end

- 보통 begin은 포함, end는 불포함

## first - last

- 양 끝점을 포함하나, 연속성을 보장하지는 않을 때

## prefix - suffix

접두사 - 접미사

- 예시
  - setter, getter
  - React Hooks : use~~

## 매개변수의 순서

1. 매개변수의 변수가 되도록이면 2개를 넘지 않도록 설정
2. arguements, rest parameter 사용
3. 매개변수를 객체에담아서 전달

   ```jsx
   function func({ arg1, arg2 }) {}
   ```

4. 랩핑하는 함수

   ```jsx
   function func1(arg1, arg2, arg3, arg4) { }

   function func2(arg1, arg2) {
   	func1(arg1, arg2
   }
   ```

---

# 분기

## 단축평가

```jsx
function fetchData() {
	if(state.data) {
		return state.data;
	} else {
		return 'Fetching...';
	}
	->
	return state.data || 'Fetching...';
}
```

```jsx
function getActiveUserName(user, isLogin) {
	if(isLogin) {
		if(user) {
			if(user.name) {
				return user.name;
			} else {
				return '이름없음';
			}
		}
	}
	->
	if(isLogin && user) {
		return user.name || '이름없음';
	}
}
```

## else if 피하기

```jsx
if(x >= 0) {
	console.log('x는 0보다 크거나 같다.');
} else if(x > 0) {
	console.log('x는 0보다 크다');
}
==
if(x >= 0) {
	console.log('x는 0보다 크거나 같다.');
} else {
	if(x > 0) {
		console.log('x는 0보다 크다');
	}
}
-> 권장 코드 :
if(x >= 0) {
	console.log('x는 0보다 크거나 같다.');
}
if(x > 0) {
	console.log('x는 0보다 크다');
}
```

- else if 가 많아지면 차라리 switch-case 사용을 추천

## else 피하기

```jsx
function getActiveUserName(user) {
	if(user.name) {
		return user.name;
	} else {
		return '이름없음';
	}
}
-> 권장코드 :
function getActiveUserName(user) {
	if(user.name) {
		return user.name;
	}

	return '이름없음';
}
```

## Early Return

- 비권장 코드

  ```jsx
  function loginService(isLogin, user) {
    if (!isLogin) {
      if (checkToken()) {
        if (!user.nickName) {
          return registerUser(user);
        } else {
          refreshToken();
          return "로그인 성공";
        }
      } else {
        throw new Error("No Token");
      }
    }
  }
  ```

- 권장코드
  ```jsx
  function loginService(isLogin, user) {
    if (isLogin) {
      return;
    }

    if (!checkToken()) {
      throw new Error("No Token");
    }

    if (!user.nickName) {
      return registerUser(user);
    }

    refreshToken();

    return "로그인 성공";
  }
  ```

## 부정 조건문 지양

- 부정 조건 예외
  - Early Return
  - Form Validation
  - 보안 또는 검사하는 로직

## Default Case 고려

```jsx
function sum(x, y) {
  x = x || 1;
  y = y || 1;

  return x + y;
}
```

## nullish coalescing operator

null 병합 연산자

피 연산자가 null 이거나 undefined일 때만, 작동

```jsx
value ?? 10;
```

- early return 에서 주의
  ```jsx
  function helloWorld(message) {
  	if(!message) {
  		return 'Hello! World';
  	}

  	return 'Hello! ' + (message || 'World');
  }

  console.log(helloWorld(0)); // Hello! World

  -> 권장 코드
  function helloWorld(message) {
  	return 'Hello! ' + (message ?? 'World');
  }
  ```

---

# 배열

## 배열 ≃ 객체

javascript의 배열은 객체처럼 취급된다.

- 배열

  ```jsx
  const arr = [1, 2, 3];
  arr[3] = "test";
  arr["property"] = "string value";
  arr["obj"] = {};
  arr["{}"] = [1, 2, 3];
  arr["func"] = function () {
    return "hello";
  };
  ```

- arr 출력

  ```jsx
  [
  	1,
  	2,
  	3,
  	'test',
  	property: 'string value',
  	obj: {},
  	'[object Object]': [1, 2, 3],
  	func: [λ]
  ]
  ```

- 객체

  ```jsx
  const obj = {
    arr: [1, 2, 3],
    3: "test",
    property: "string value",
    obj: {},
    "{}": [1, 2, 3],
    func: function () {
      return "hello";
    },
  };
  ```

- obj 출력
  ```jsx
  {
  	arr: [1, 2, 3]
  	3: 'test',
  	property: 'string value',
  	obj: {},
  	'{}': [1, 2, 3],
  	func: [λ: func]
  }
  ```

## Array.length

- 배열의 길이를 보장하지 않는다. 배열의 마지막 인덱스에 가깝다.

```jsx
const arr = [1, 2, 3];
console.log(arr.length); // 3

arr[3] = 4;
console.log(arr.length); // 4

arr[5] = 6;
console.log(arr.length); // 6

arr.length = 10;
console.log(arr.length); // 10
```

→ 배열의 길이에 0을 할당하면 배열이 초기화 된다.

```jsx
Array.prototype.clear = function () {
  this.length = 0;
};

function clearArray(array) {
  array.length = 0;
  return array;
}

const arr = [1, 2, 3];
console.log(clearArray(arr)); // []

arr = [1, 2, 3];
arr.clear(); // []
```

## 배열 요소에 접근

```jsx
const [firstInput, secondInput] = input;
```

```jsx
function func([firstInput, secondInput]) { ... }
func([1, 2]);
```

```jsx
const [year, month, day] = date.split("-");
```

## 유사 배열 객체

```jsx
const arrayLikeObject = {
  0: "Hello",
  1: "World",
  length: 2,
};
const arr = Array.from(arrayLikeObject);

console.log(Array.isArray(arrayLikeObject)); // false
console.log(Array.isArray(arr)); // true
```

- arguments
  ```jsx
  function generatePriceList() {
    console.log(Array.isArray(arguments)); // false

    for (let i; i < arguments.length; i++) {
      const element = arguments[index];
      console.log(element); // 100 200 300
    }
  }

  generatePriceList(100, 200, 300);
  ```

⇒ 유사 배열 객체는 map, forEach, reduce, filter, every, some 등은 사용할 수 없다.

## 불변성

immutable

1. 배열을 복사

   ```jsx
   const originArray = [1, 2, 3];
   const newArray = [...originArray];

   originArray.push(4);
   newArray.push(5);
   ```

   ⇒ originArray ≠ newArray

```jsx
const originArray = [1, 2, 3];
const newArray = [originArray];

originArray.push(4);
newArray.push(5);
```

⇒ originArray == newArray

1. 새로운 배열을 반환하는 메서드를 활용

   예시) map, filter, slice, . . .

## for문 → 고차함수

고차함수 : map, filter, sort, . . .

```jsx
function getWonPrice(priceList) {
	let temp[];

	for(let i=0; i<priceList.length; i++) {
		temp.push(priceList[i] + '원');
	}

	return temp;
}
```

- 권장코드
  ```jsx
  function getWonPrice(priceList) {
  	return priceList.map((price) -> price + '원');
  }
  ```
  ```jsx
  const suffixWon = (price) -> price + '원';

  function getWonPrice(priceList) {
  	return priceList.map(suffixWon);
  }
  ```
- 고차함수에서는 continue, break 사용 불가
  → for문으로 제어하거나 every, some, find, findIndex같은 고차함수 사용하여 제어

## map vs forEach

```jsx
priceList.forEach((price) -> price + '원');
priceList.map((price) -> price + '원')
```

- forEach : 배열을 순회하며 각각의 요소에 대해 함수를 실행
- map : 새로운 배열을 반환

---

# 객체

## Shorthand Properties

- 기본코드
  ```jsx
  const firstName = "poco";
  const secondName = "jang";

  const person = {
    firstName: "poco",
    secondName: "jang",
    getFullName: function () {
      return this.firstName + " " + this.secondName;
    },
  };
  ```
- 적용 코드
  ```jsx
  const firstName = "poco";
  const secondName = "jang";

  const person = {
    firstName,
    secondName,
    getFullName() {
      return this.firstName + " " + this.secondName;
    },
  };
  ```

## Computed Property Name

```jsx
const handleChange = (e) => {
	setState({
		**[**e.target.name**]**: e.target.value
	});
}

return (
	<React.Fragment>
		<input value={state.id} onChange={handleChange} name="name" />
		<input value={state.password} onChange={handleChange} name="password" />
	</React.Fragment>
);
```

## Lookup Table

- 배열 데이터 구조에서 key와 value로 관리된 배열이 나열된 형태
- else if문이나 switch-case에서 경우가 많아질 경우 사용 권장

```jsx
function getUserType(type) {
  return (
    {
      ADMIN: "관리자",
      INSTRUCTOR: "강사",
      STUDENT: "수강생",
    }[type] ?? "해당 없음"
  );
}
```

- 권장코드 (상수를 다른 파일에서 관리 가능)
  ```jsx
  function getUserType(type) {
    const USER_TYPE = {
      ADMIN: "관리자",
      INSTRUCTOR: "강사",
      STUDENT: "수강생",
      UNDEFINED: "해당 없음",
    };

    return USER_TYPE[type] || USER_TYPE.UNDEFINED;
  }
  ```

## Objecy Destructuring

객체 구조 할당

- 기존 코드

  ```jsx
  function Person(name, age, location) {
    this.name = name;
    this.age = age;
    this.location = location;
  }

  const poco = new Person("poco", 30, "korea");
  ```

- 권장 코드
  ```jsx
  function Person({ name, age, location }) {
    this.name = name;
    this.age = age ?? 30;
    this.location = location ?? "korea";
  }

  const poco = new Person({
    age: 30,
    location: "korea",
    name: "poco",
  });
  ```

```jsx
function Person(name, { age, location }) {
  this.name = name;
  this.age = age ?? 30;
  this.location = location ?? "korea";
}

const poco = new Person("poco", {
  age: 30,
  location: "korea",
});
```

## Prototype 조작 지양

- javascript는 이미 많이 발전했다.
- js 빌트인 객체를 건들지 말자

## hasOwnProperty

```jsx
const person = { name: "poco" };

person.hasOwnProperty("name"); // true
person.hasOwnProperty("age"); // false
```

- 다른 키워드에 있는 hasOwnProperty를 호출할 수 있으므로,
  객체의 prototype에 직접적으로 붙어서 call 메서드를 활용해야 안전하게 객체에 사용 가능하다.
      ```jsx
      const foo = {
      	hasOwnproperty: function() { return 'hasOwnProperty'; },
      	bar: 'string'
      }

      foo.hasOwnProperty('bar')                           // hasOwnProperty
      Object.prototype.hasOwnProperty.call(foo, 'bar')    // true
      ```

## 직접 접근 지양

- 지양할 코드
  ```jsx
  const model = { isLogin: false, isValidation: false };

  function login() {
    model.isLogin = true;
    model.isValidation = true;
  }
  function logout() {
    model.isLogin = false;
    model.isValidation = false;
  }
  ```
- 권장 코드
  ```jsx
  const model = { isLogin: false, isValidation: false };

  // model에 대신 접근
  function setLogin(bool) {
    model.isLogin = bool;
  }
  function setValidation(bool) {
    model.isValidation = bool;
  }

  // model에 직접 접근 X
  function login() {
    setLogin(true);
    setValidation(true);
  }
  function logout() {
    setLogin(false);
    setValidation(false);
  }
  ```

## Optional Chaining ?:

---

# 함수

## 함수, 메서드, 생성자

- 함수
  - 1급 객체
    - 변수나 데이터에 담길 수 있다.
    - 매개변수로 전달 가능 (콜백 함수)
    - 함수가 함수를 반환 (고차 함수)
  - this : 전역 객체(global)
  ```jsx
  function func() {
    return this;
  }

  func();
  ```
- 메서드 : 객체에 의존성이 있는 함수. (OOP 행동을 의미)
  - this : 호출된 객체
  ```jsx
  const obj = {
    method() {
      // == method: function() {
      return this;
    },
  };

  obj.method();
  ```
- 생성자 함수 : 인스턴스를 생성하는 역할 ⇒ 요즘은 Class 사용
  - this : 생성될 instance
  ```jsx
  function Func() {
    return this;
  }

  const instance = new Func();
  ```

## argument vs parameter

```jsx
function example(parameter) {
  console.log(parameter);
}

const argument = "foo";
example(argument);
```

## Rest Parameters

```jsx
function sumTotal(initValue, ...args) {
  Array.isArray(args); // true
  console.log(initValue); // 100
  return args.reduce((acc, curr) => acc + curr);
}

sumTotal(100, 1, 2, 3, 4, 5);
```

- rest parameters는 인자의 마지막에 위치해야 한다.

## 화살표 함수

- 함수
  ```jsx
  const user = {
    name: "poco",
    getName() {
      return this.name;
    },
  };

  user.getName(); // poco
  ```
- 화살표 함수
  - this : 상위 객체
  - arguments, call, apply, bind 등 사용 불가 (arguments는 rest parameters로 대체 가능)
  - 생성자로 사용 불가
  - 부모클래스에서 화살표 함수로 생성한 메서드는 super로 호출 불가
  - 부모클래스에서 화살표 함수로 생성한 메서드는 오버라이딩 불가 (오버라이딩 해도 부모 함수로 호출됨)
  - `function*`에서 사용 불가
  ```jsx
  const user = {
  	name: 'poco',
  	getName: () => {
  		return this.name;
  	}
  }

  user.getName();      // undefined

  const Person = (name, city) => {
  	this.name = name,
  	this.city = city
  }

  const person new Person('poco', 'korea');   // Error
  ```

## 콜백 함수

- 이전 코드
  ```jsx
  function register() {
    const isConfirm = confirm("회원가입에 성공했습니다.");

    if (isConfirm) {
      redirectUserInfoPage();
    }
  }

  function login() {
    const isConfirm = confirm("로그인에 성공했습니다.");

    if (isConfirm) {
      redirectIndexPage();
    }
  }
  ```
- 리팩토링
  ```jsx
  function confirmModel(message, cbFunc) {
    const isConfirm = confirm(message);

    if (isConfirm && cbFunc) {
      cbFunc();
    }
  }

  function register() {
    confirmModel("회원가입에 성공했습니다.", redirectUserInfoPage);
  }

  function login() {
    confirmModel("로그인에 성공했습니다.", redirectIndexPage);
  }
  ```

## 순수 함수 지향

- 이전 코드
  ```jsx
  const obj = { one: 1 };

  function changeObj(targetObj) {
    targetObj.one = 100;

    return targetObj;
  }

  changeObj(obj); // { one: 100 }
  obj; // { one: 100 }
  ```
- 리팩토링 : 객체, 배열은 새롭게 만들어서 반환
  ```jsx
  const obj = { one: 1 };

  function changeObj(targetObj) {
    return { ...targetObj, one: 100 };
  }

  changeObj(obj); // { one: 100 }
  obj; // { one: 1 }
  ```

## Closure

```jsx
function add(num1) {
  return function (num2) {
    return num1 + num2;
  };
}

const addOne = add(1);
const addTwo = addOne(2); // 3
const sum = add(1)(2); // 3
```

---

# 추상화

## Magin Number

```jsx
const DELAY_MS = 3 * 60 * 1000;
setTimeout(() => {
  scrollToTop();
}, DELAY_MS);

// Numeric Operator
const PRICE = {
  MIN: 1_000_000,
  MAX: 100_000_000,
};
getRandomPrice(PRICE.MIN, PRICE.MAX);
```

## 네이밍 컨벤션

- javascript의 예약어와 겹치지 않도록 주의

---
