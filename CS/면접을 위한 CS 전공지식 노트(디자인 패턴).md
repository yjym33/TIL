![alt text](/CS/image/면접을위한CS전공지식노트.png)

## 📚 목차

- [1장 디자인 패턴과 프로그래밍 패러다임](#1장-디자인-패턴과-프로그래밍-패러다임)
  - [1.1 디자인 패턴](#11-디자인-패턴)
    - [1.1.1 싱글톤 패턴](#111-싱글톤-패턴)
    - [1.1.2 팩토리 패턴](#112-팩토리-패턴)
    - [1.1.3 전략 패턴](#113-전략-패턴)
    - [1.1.4 옵저버 패턴](#114-옵저버-패턴)
    - [1.1.5 프록시 패턴과 프록시 서버](#115-프록시-패턴과-프록시-서버)
    - [1.1.6 이터레이터 패턴](#116-이터레이터-패턴)
    - [1.1.7 노출모듈 패턴](#117-노출모듈-패턴)
    - [1.1.8 MVC 패턴](#118-mvc-패턴)
    - [1.1.9 MVP 패턴](#119-mvp-패턴)
    - [1.1.10 MVVM 패턴](#1110-mvvm-패턴)

<br>

## 디자인 패턴

### 디자인 패턴이란?

- 프로그램을 설계할 때 발생했던 문제점들을 객체 간의 상호 관계 등을 이용하여 해결할 수 있도록 하나의 '규약' 형태로 만들어 놓은 것

<br>

## 싱글톤 패턴

### 싱글톤 패턴 (singleton pattern) 이란?

- 싱글톤 패턴 (singleton pattern) 이란?
- 보통 데이터베이스 연결 모듈에 많이 사용

## 실제 사용 예시

Node.js - MongoDB - 데이터베이스 연결 mongoose 모듈 <br>
Node.js - MySQL - 데이터베이스 연결

## 장점

인스턴스를 생성할 때 드는 비용이 줄어듦
사용하기 쉽고 실용적임

## 단점

TDD(Test Driven Development) 시 걸림돌

단위 테스트는 테스트가 서로 독립적이어야 하고 테스트를 어떤 순서로든 실행할 수 있어야 함
의존성이 높아짐

<br>

## 자바스크립트 싱글톤 패턴

👉 자바스크립스의 리터럴 {} 또는 new Object로 객체 생성시 다른 어떤 객체와도 같지 않으므로 이 자체만으로도 싱글톤 구현

```bash
const obj1 = {
a: 27
}

const obj2 = {
a: 27
}

console.log(obj1 === obj2) // false
```

<br>

## 실제 싱글톤 구현 코드

```js
class Singleton {
  constructor() {
    if (!Singleton.instance) {
      Singleton.instance = this;
    }
    return Singleton.instance;
  }
  getInstance() {
    return this.instance;
  }
}

const a = new Singleton();
const b = new Singleton();
console.log(a === b); // true
```

<br>

## 자바 싱글톤 패턴

```java
class Singleton{
	private static class singleInstanceHolder{
    	private static final Singleton INSTANCE = new Singleton();
    }
	public static Singleton getInstance(){
    	return singleInstanceHolder.INSTANCE;
    }
}

Singleton a = Singleton.getInstance();
Singleton b = Singleton.getInstance();
// a.hashCode == b.hashCode
```

<br>

## 의존성 주입

### 의존성 주입(Dependency Injection)이란?

싱글톤 패턴의 모듈 간의 강한 결합을 느슨하게 만드는 방법
모듈 계층의 중간에 의존성 주입자 활용하여, 메인 모듈이 '간접'적으로 하위 모듈에 의존성을 주입하는 방식 이를 '디커플링 된다' 라고도 함

### 의존성 주입 원칙

- 상위 모듈은 하위 모듈에서 어떠한 것도 가져오지 않아야 합니다. 또한, 둘 다 추상화에 의존해야 하며, 이때 추상화는 세부 사항에 의존하지 말아야 합니다."

## 장점

- 모듈들을 쉽게 교체할 수 있는 구조가 됨
- 테스팅하기 쉬움
- 마이그레이션 수월
- 추상화 레이어를 넣어, 이를 기반으로 구현체를 넣음
- 애플리케이션 의존성 방향이 일관됨
- 애플리케이션 쉽게 추론
- 모듈 간의 관계들이 명확해짐

## 단점

- 모듈들이 더욱더 분리돼 클래스 수 증가
- 복잡성 증가
- 약간의 런타임 패널티
