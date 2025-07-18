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

## 팩토리 패턴

### 팩토리 패턴 (factory pattern) 이란?

- 객체를 사용하는 코드에서 객체 생성 부분을 떼어내 추상화한 패턴
- 상속 관계에서 상위 클래스가 중요한 뼈대를 결정, 하위 클래스에서 객체 생성에 관한 구체적인 내용 결정
  - 두 클래스 분리 -> 느슨한 결합
  - 상위 클래스 -> 더 많은 유연성 (생성 방식 몰라도 됨)
  - 유지 보수성 증가

## 자바스크립트 팩토리 패턴

- 자바스크립스의 new Object로 전달받은 값에 따라 다른 객체 생성

```js
const num = new Object(42);
const str = new Object("abc");
num.constructor.name; // Number
num.constructor.name; // String
```

### 팩토리 패턴 활용 자바스크립트 예시 코드

```js
class Latte {
  constructor() {
    this.name = "latte";
  }
}

class Espresso {
  constructor() {
    this.name = "Espresso";
  }
}

class LatteFactory {
  static createCoffee() {
    return new Latte();
  }
}

class EspressoFactory {
  static createCoffee() {
    return new Espresso();
  }
}

const factoryList = { LatteFactory, EspressoFactory };

class CoffeeFactory {
  static createCoffee(type) {
    const factory = factoryList[type];
    return factory.createCoffee();
  }
}

const main = () => {
  // 라떼 커피를 주문
  const coffee = CoffeeFactory.createCoffee("LatteFactory");
  // 커피 이름을 부름
  console.log(coffee.name); // latte
};
```

- LatteFactory에서 생성한 인스턴스를 CoffeeFactory에서 주입하기 때문에 의존성 주입으로도 볼 수가 있다.
- static 정적 메서드 -> 객체를 만들지 않고도 호출 가능, 메모리 할당 한 번만 실행

<br>

### 팩토리 패턴 활용 자바 예시 코드

```java
abstract class Coffee{
	public abstract int getPrice();

	@Override
	public String toString(){
		return "Hi this coffee is " + this.getPrice();
	}
}

class CoffeeFactory{
	public static Coffee getCoffee(String type, int price){
		if("Latte".equalsIgnore(type)) return new Latte(price);
		else if("Americano".equalsIgnore(type)) return new Americano(price);
		else return new DefaultCoffee();
	}
}
```

<br>

## 전략 패턴

### 전략 패턴 (strategy pattern) 이란?

- 객체의 행위를 바꾸고 싶은 경우 '직접' 수정하지 않고 전략이라고 부르는 '캡슐화한 알고리즘'을 컨텍스트 안에서 바꿔주면서 상호 교체가 가능하게 만드는 패턴

### 실 사용 예시

- Node.js의 인증 모듈 구현 : passport 라이브러리
  - LocalStrategy 전략 : 회원가입된 아이디와 비밀번호를 기반으로 인증
  - OAuth 전략 : 페이스북, 네이버 등 다른 서비스를 기반으로 인증

### 전략 패턴 자바 예시 코드

- 쇼핑 카트에 아이템을 담아 LUNACard 또는 KAKAOCard라는 두개의 전략으로 결제하는 코드

```java

interface PaymentStrategy{
	public void pay(int amount);
}

class KAKAOCardStrategy implements PaymentStrategy{
	private String name;
	private String cardNumber;
	private String cvv;
	private String dateOfExpiry;

	public KAKAOCardStrategy(String nm, String ccNum, String cvv, String expiryDate){
		this.name = nm;
		this.cardNumber = ccNum;
		this.cvv = cvv;
		this.dateOfExpiry = expiryDate;
	}

	@Override
	public void pay(int amount){
		System.out.println(amount + " paid using KAKAOCard.");
	}
}

class LUNACardStrategy implements PaymentStrategy{
	private String emailId;
	private String password;

	public LUNACardStrategy(String email, String pwd){
		this.emailId = email;
		this.password = pwd;
	}

	@Override
	public void pay(int amount){
		System.out.println(amount + " paid using LUNACard.")
	}
}

class Item{
	private String name;
	private int price;
	public Item(String name, int cost){
		this.name = name;
		this.price = cost;
	}

	public String getName(){
		return name;
	}

	public int getPrice(){
		return price;
	}
}

class ShoppingCart{
	List<Item> items;

	public ShoppingCart(){
		this.items = new ArrayList<Item>();
	}

	public void addItem(Item item){
		this.items.add(item);
	}

	public void removeItem(Item item){
		this.items.remove(item);
	}

	public int calculateTotal(){
		int sum = 0;
		for(Item item: items)
			sum += item.getPrice();
		return sum;
	}

	public void pay(PaymentStrategy paymentMethod){
		int amount = calculateTotal();
		paymentMethod.pay(amount);
	}
}

public class StrategyPattern{
	public static void main(String[] args){
		ShoppingCart cart = new ShoppingCart();

		Item A = new Item("kundolA",100);
		Item B = new Item("kundolB",200);

		cart.addItem(A);
		cart.addItem(B);

		// pay by LUNACard
		cart.pay(new LUNACardStrategy("kundol@example.com","temppwd"));

		// pay by KAKAOCard
		cart.pay(new KAKAOCardStrategy("Yijun","1234","123","8/13"));
	}
}

/*
300 paid using LUNACard.
400 paid using KAKAOCard.
*/
```

<br>

## 옵저버 패턴

### 옵저버 패턴 (observer pattern) 이란?

- 주체가 어떤 객체의 상태 변화를 관찰하다가 상태 변화가 있을 때마다 메서드 등을 통해 옵저버 목록에 있는 옵저버들에게 변화를 알려주는 디자인 패턴
  - 주체 : 객체의 상태 변화를 보고 있는 관찰자
  - 옵저버 : 객체의 상태 변화에 따라 전달되는 메서드 등을 기반으로 '추가 변화 사항'이 생기는 객체

### 실제 사용 예시

- 트위터
  - 옵저버들이 '팔로우'한 주체의 포스팅이 '팔로워'들에게 알림이 간다.
- Vue.js 3.0 프레임워크
  - ref나 reactive로 정의 -> 해당 값 변경 시 자동으로 DOM 반영
- 주로 이벤트 기반 시스템 & MVC 패턴에 사용
  - 주체(모델)에서 변경사항이 생겨 update() 메서드로 옵저버(뷰)에 알리고 이를 기반으로 컨트롤러 작동

<br>

## 옵저버 패턴 활용 예시 코드

### Java

- 해당 코드에서 topic은 주체이자 객체임

```java

interface Subject{
	public void register(Observer obj);
	public void unregister(Observer obj);
	public void notifyObservers();
	public Object getUpdate(Observer obj);
}

interface Observer{
	public void update();
}

class Topic implements Subject{
	private List<Observer> observers;
	private String message;

	public Topic(){
		this.observers = new ArrayList<>();
		this.message = "";
	}

	@Override
	public void register(Observer obj){
		if(!observers.contains(obj))
			observers.add(obj);
	}

	@Override
	public void unregister(Observer obj){
		observers.remove(obj);
	}

	@Override
	public void notifyObservers(){
		this.observers.forEach(Observer::update);
	}

	@Override
	public Object getUpdate(Observer obj){
		return this.message;
	}

	public void postMassage(String msg){
		System.out.println("Message sended to Topic: " + msg);
		this.message = msg;
		notifyObservers();
	}
}

class TopicSubscriber implements Observer{
	private String name;
	private Subject topic;

	public TopicSubscriber(String name, Subject topic){
		this.name = name;
		this.topic = topic;
	}

	@Override
	public void update(){
		String msg = (String) topic.getUpdate(this);
		System.out.println(name + ":: got message >> " + msg);
	}
}

public class ObserverPattern{
	public static void main(String[] args){
		Topic topic = new Topic();
		Observer a = new TopicSubscriber("a", topic);
		Observer b = new TopicSubscriber("b", topic);

		topic.register(a);
		topic.register(b);

		topic.postMassage("Happy new year!");
	}
}

/*
Message sended to Topic: Happy new year!
a:: got message >> Happy new year!
b:: got message >> Happy new year!
*/

```

<br>

### 상속과 구현

- 상속(extends) 이란?

  - 자식 클래스가 부모 클래스의 메서드 등을 상속받아 사용하며, 자식 클래스에서 추가 및 확징을 할 수 있는 것
    - 재사용성, 중복성의 최소화

- 구현(implements) 이란?
  - 부모 인터페이스를 자식 클래스에서 '재정의'하여 구현하는 것으로 반드시 부모 클래스의 메서드를 재정의해야 함
    - 인터페이스(interface)를 기반으로 구현

<br>

### Javascript

- 자바스크립트에서 옵저버 패턴은 프록시 객체를 사용하는것을 의미함
  - 프록시(proxy) 객체 : 어떠한 대상의 기본적인 동작의 작업을 가로챌 수 있는 객체로, 두 매개변수를 가짐
    - target : 프록시할 대상
    - handler : target 동작을 가로채고 어떠한 동작을 할 것인지의 함수

```javascript
function createReactiveObject(target, callback) {
  const proxy = new Proxy(target, {
    set(obj, prop, value) {
      if (value !== obj[prop]) {
        const prev = obj[prop];
        obj[prop] = value;
        callback(`${prop}가 [${prev}] >> [${value}] 로 변경되었습니다.`);
      }
      return true;
    },
  });
  return proxy;
}

const a = { 형규: "솔로" };
const b = createReactiveObject(a.console.log);
b.형규 = "솔로";
b.형규 = "커플";
// 형규가 [솔로] >> [커플] 로 변경되었습니다.
```

<br>

## 프록시 패턴

### 프록시 패턴 (proxy pattern) 이란?

- 대상 객체에 접근하기 전 그 접근에 대한 흐름을 가로채 대상 객체 앞단의 인터페이스 역할을 하는 디자인 패턴
- 객체의 속성, 변환등을 보완
- 보안, 데이터 검증, 캐싱, 로깅에 사용

### 프록시 서버 (proxy server) 란?

- 서버와 클라 사이에서 클라가 자신을 통해 다른 네트워크 서비스에 간접적으로 접속할 수 있게 해주는 시스템이나 응용 프로그램

## Nginx

### nginx란?

- nginx는 비동기 이벤트 기반의 구조와 다수의 연결을 효과적으로 처리 가능한 웹 서버
- 주로 Node.js 서버 앞단의 프록시 서버로 활용
  - 실제 서버의 포트를 숨기고, 정적 자원을 gzip 압축, 메인 서버 앞단에서의 로깅 가능
    -> 익명의 사용자가 서버의 직접 접근 차단, 보안 강화

## CloudFlare

### CloudFlare란?

- CloudFlare는 웹 서버 앞단에 프록시 서버로 두어 DDOS 공격 방어나 HTTPS 구축에 쓰임
- 의심스러운 트래픽인지 먼저 판단해 CAPTCHA 등을 기반으로 막아주기도 함

<br>

`DDOS 공격` : DDOS는 짧은 시간 동안 네트워크에 많은 요청을 보내 네트워크를 마비시켜 웹사이트의 가용성을 방해하는 사이버 공격 유형  
`HTTPS 구축` : 서버에서 HTTPS를 구축할 때, CloudFlare는 별도의 인증서 없이 손쉽게 가능

<br>

## CORS

참고 : https://www.youtube.com/watch?v=j2Q2Ev6CZzQ

### CORS (Cross-Origin Resource Sharing) 란?

- CORS는 서버가 웹 브라우저 리소스를 로드할 때 다른 오리진을 통해 로드하지 못하게 하는 HTTP 헤더 기반 메커니즘 -> 프록시 서버를 두어 프론트엔트 서버에서 요청되는 오리진을 서버와 맞게 바꿈
- SOP : Same Origin Policy 를 지키기 위함

<br>

`오리진` : 프로토콜과 호스트 이름, 포트의 조합 <br>
ex) https://yijun.com:8080

<br>

## Preflight request

### Preflight request란?

- GET, HEAD, POST, [Content-Type 헤더]... 등과 같은 Simple request와는 달리,
  PUT, DELETE와 같은 request는 OPTION 메서드로 CORS인지 한 번 더 요청하는 과정을 거침 - 물론 캐싱 가능

### 프록시 객체 활용 자바스크립트 예시 코드

```javascript
const handler = {
  get: function(target, name){
    return name === 'name' ? `${} ${}` : target[name]
  }
}

const p = new Proxy({a: 'I', b: 'AM GANGPLANK ZANGIN'}, handler)
console.log(p.name) // I AM GANGPLANK ZANGIN
```
