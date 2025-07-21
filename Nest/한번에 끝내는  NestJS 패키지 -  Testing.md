![alt text](/Nest/image/codeFactory.png)
출처 : https://fastcampus.co.kr/dev_online_nestjs

<br>

## 테스팅이란?

소프트웨어가 예상대로 실행되는지 검증하고 확인하는 과정.

<br>

### 주요 목적:

- **버그 발견**: 프로덕션 환경에서 문제를 예방.
- **코드 리팩터링 지원**: 변경된 코드가 기존 로직에 문제를 일으키는지 확인.
- **다큐멘테이션 역할**: 코드 작동 방식을 설명.
- **자동화된 로직 검증**: 반복적 테스트 작업을 줄임.

<br>

### Testing의 중요성

- 현재 우리의 지식으로 이 코드가 잘 작동하는지 테스트하
  려면 이 코드가 실행되는 Controller를 통해 API 콜을 직
  접 하는 방법밖에 없다.
- 너무 오래걸리고 일관성이 부족하다.
- 이 코드를 작성하지 않은 사람은 어떻게 테스트 해야할지
  알 수 없다. 어떤 값들까지 테스트 해야하는지 등
- 정확한 로직의 바운더리를 알 수 없으니 다른 사람이 이 코
  드를 변경 했을때 내가 망가뜨린 로직이 없는지 쉽게 확인
  이 불가능하다.
- 그래서 우리는 코드를 코드로 테스트하고, 테스트 코드라
  고 부른다.

  <br>

```tsx
// 기존 일반 코드

import { Injectable } from "@nestjs/common";

@Injectable()
export class MathService {
  add(a: number, b: number): number {
    return a + b;
  }
}
```

<br>

### Testing 예제

```tsx
import { Test, TestingModule } from "@nestjs/testing";
import { MathService } from "./math.service";

describe("MathService", () => {
  let service: MathService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [MathService],
    }).compile();

    service = module.get<MathService>(MathService);
  });

  it("should be defined", () => {
    expect(service).toBeDefined(); // 정의가 되었는지 확인한다.
  });

  it("should return correct sum of two numbers", () => {
    const result = service.add(2, 3);
    expect(result).toEqual(5); // 인풋에 대한 아웃풋을 테스트한다.
  });

  it("should handle negative numbers", () => {
    const result = service.add(-1, -3);
    expect(result).toEqual(-4); // 다양한 케이스를 테스트한다.
  });
});
```

- 테스트 코드는 코드의 작동 로직을 검증하는 코드다.
- 흔히 어떤 특정 값이 입력됐을때 어떤 값이 반환되는지 확인한다.
- 다양한 값들을 입력 해볼 수 있다. 예제에서 처럼 양수 뿐만 아니라 음수에 대한 덧셈도 테스트 해볼 수 있다.
- 특정 기대치에 대한 assert가 가능하다.
- expect()를 사용한 assert는 false를 반환할때 테스트가 실패한다.
- 억지로 이상한 값을 넣어서 기대하는 에러가 발생하는지 확인 가능하다.
- 가능하면 안되는 로직을 검증 할 수 있다.

### Testing 결과

![alt text](/Nest/image/testing.png)

- “pnpm test” 커맨드를 사용해서 테스트를 실행하고 결과 값을 받아 볼 수 있다.
- 어떤 테스트가 성공/실패 했는지와 함께 몇개의 테스트가 성공/실패 했는지, 실행하는데 얼마나 걸렸는지 등의 통계를 받을 수 있다.

### Matcher 함수

![alt text](/Nest/image/testing2.png)

### 기본 Matcher

- toBe(value) 값이 같은지 확인한다.
- toEqual(value) 객체의 모든 값이 같은지 재귀적으로 확인한다.
- toBeNull() : toBe(null)과 같다. Falsy일때 조금 더 명확한 에러 메세지가 발생한다.
- toBeUndefined() 값이 undefined인걸 확인한다. toBe(undefined)를 실행 할 수도 있지만 직접 코드에서undefined를 레퍼런스 하는건 지양해야 한다.
- toBeDefined() : toBeUndefined()의 반대다.
- toBeTruthy() JS에서 인지하는 true 값을 반환하는지 확인한다.
- toBeFalsy() : toBeTruthy()의 반대
- toBeNan() 숫자가 아님을 확인한다.

### toEqual() vs toBe()

```tsx
const can1 = {
  flavor: "grapefruit",
  ounces: 12,
};

const can2 = {
  flavor: "grapefruit",
  ounces: 12,
};

describe("Test", () => {
  test("have all the same properties", () => {
    expect(can1).toEqual(can2);
  });

  test("are not the exact same can", () => {
    expect(can1).not.toBe(can2);
  });
});
```

### 숫자 Matcher

- toBeGreaterThan(number) 값이 더 큰지 확인한다.
- toBeGreaterThanOrEqual(number) 값이 더 크거나 같은지 확인한다.
- toBeLessThan(number) 값이 더 작은지 확인한다.
- toBeLessThanOrEqual(number) 값이 더 작거나 같은지 확인한다.
- toBeCloseTo(number, numDigits?) 특정 소수점까지 같은 값인지 확인한다.

### toBeCloseTo()

![alt text](/Nest/image/testing3.png)

### 함수 Matcher

- toHaveBeenCalled() : mock function이 호출되었는지 확인합니다.
- toHaveBeenCalledTimes(number) : mock function이 지정된 횟수만큼 호출되었는지 확인한다.
- toHaveBeenCalledWith(arg1, arg2, …) : mock function이 특정 파라미터와 함께 호출되었는지 확인한다.
- toHaveBeenLastCalledWith(value) : mock function이 마지막으로 호출될 때 특정 파라미터와 함께 호출되었는지 확인한다.
- toHaveBeenNthCalledWith(nthCall, value) : mock function이 n번째로 호출될 때 특정 파라미터와 함께 호출되었는지 확인한다.
- toHaveReturned() : mock function이 값을 반환했는지 확인한다. (에러를 던지지 않음)
- toHaveReturnedTimes(number) : mock function이 값을 지정된 횟수만큼 반환했는지 확인한다.
- toHaveReturnedWith(value) : mock function이 특정 값을 반환했는지 확인한다.
- toHaveLastReturnedWith(value) : mock function이 마지막으로 특정 값을 반환했는지 확인한다.
- toHaveNthReturnedWith(nthCall, value) : mock function이 n번째로 특정 값을 반환했는지 확인한다

### toHaveBeenCalled()

```tsx
function drinkAll(callback, flavour) {
  if (flavour !== "octopus") {
    callback(flavour);
  }
}

describe("Test", () => {
  test("drinks something lemon-flavoured", () => {
    const drink = jest.fn();
    drinkAll(drink, "lemon");
    expect(drink).toHaveBeenCalled();
  });

  test("does not drink something octopus-flavoured", () => {
    const drink = jest.fn();
    drinkAll(drink, "octopus");
    expect(drink).not.toHaveBeenCalled();
  });
});
```

### toHaveNthReturnedWith()

```tsx
test("drink returns expected nth calls", () => {
  const beverage1 = { name: "Lemon" };
  const beverage2 = { name: "Orange" };
  const drink = jest.fn((beverage) => beverage.name);

  drink(beverage1);
  drink(beverage2);

  expect(drink).toHaveNthReturnedWith(1, "Lemon");
  expect(drink).toHaveNthReturnedWith(2, "Orange");
});
```

### 배열 및 객체 Matcher

- toContain(item) 배열 또는 문자열에 특정 항목이 포함되어 있는지 확인한다.
- toContainEqual(item) 배열에 구조적으로 같은 항목이 포함되어 있는지 확인한다.
- toHaveLength(number) 배열, 문자열 또는 객체의 길이/크기가 특정 값과 일치하는지 확인한다.
- toHaveProperty(keyPath, value?) 객체에 특정 경로의 속성이 존재하고, 선택적으로 해당 속성의 값이 특정 값과 일치하는지 확인한다.
- toMatchObject(object) 객체가 특정 객체와 부분적으로 일치하는지 확인한다.

### toContainEqual()

```tsx
describe("my beverage", () => {
  test("is delicious and not sour", () => {
    const myBeverage = { delicious: true, sour: false };
    expect([myBeverage, ...[]]).toContainEqual(myBeverage);
  });
});
```

### toHaveProperty()

```tsx
const houseForSale = {
  bath: true,
  bedrooms: 4,
  kitchen: {
    amenities: ["oven", "stove", "washer"],
    area: 20,
    wallColor: "white",
    "nice.oven": true,
  },
  livingroom: {
    amenities: [
      {
        couch: [
          ["large", { dimensions: [20, 20] }],
          ["small", { dimensions: [10, 10] }],
        ],
      },
    ],
  },
  "ceiling.height": 2,
};
```

```tsx
test("this house has my desired features", () => {
  expect(houseForSale).toHaveProperty("bath");
  expect(houseForSale).toHaveProperty("bedrooms", 4);

  expect(houseForSale).not.toHaveProperty("pool");

  // '.'을 사용해서 깊게 레퍼런싱하기
  expect(houseForSale).toHaveProperty("kitchen.area", 20);
  expect(houseForSale).toHaveProperty("kitchen.amenities", [
    "oven",
    "stove",
    "washer",
  ]);

  expect(houseForSale).not.toHaveProperty("kitchen.open");

  // Array를 사용해서 깊게 레퍼런싱하기
  expect(houseForSale).toHaveProperty(["kitchen", "area"], 20);
  expect(houseForSale).toHaveProperty(
    ["kitchen", "amenities"],
    ["oven", "stove", "washer"]
  );
  expect(houseForSale).toHaveProperty(["kitchen", "amenities", 0], "oven");
  expect(houseForSale).toHaveProperty(
    ["livingroom", "amenities", 0, "couch", 0, 1, "dimensions", 0],
    20
  );
  expect(houseForSale).toHaveProperty(["kitchen", "nice.oven"]);
  expect(houseForSale).not.toHaveProperty(["kitchen", "open"]);

  // 키값 자체에 '.'이 있는 경우 Array 사용
  expect(houseForSale).toHaveProperty(["ceiling.height"], "tall");
});
```

### 에러 Matcher

- toThrow(error?) 함수가 호출될 때 특정 오류를 던지는지 확인한다.

### toThrow()

```tsx
function drinkFlavor(flavor) {
  if (flavor == "octopus") {
    throw new DisgustingFlavorError("으악 문어 노맛이야!");
  }
}
```

```tsx
test("throws on octopus", () => {
  function drinkOctopus() {
    drinkFlavor("octopus");
  }

  // 에러메시지 어디엔가 '노맛'이라고 쓰여있는지 확인한다.
  expect(drinkOctopus).toThrow(/노맛/);
  expect(drinkOctopus).toThrow("노맛");

  // 정확한 문장을 테스트한다.
  expect(drinkOctopus).toThrow(/^으악 문어 노맛이야!$/);
  expect(drinkOctopus).toThrow(new Error("으악 문어 노맛이야!"));

  // DisgustingFlavorError 타입의 에러가 던져지는걸 확인한다.
  expect(drinkOctopus).toThrow(DisgustingFlavorError);
});
```

### 기타 Matcher

- toStrictEqual(value) 객체가 구조적으로 완벽히 동일한지 확인한다 (프로토타입 및 비열거형 속성 포함).
- toBeInstanceOf(Class) 값이 특정 클래스의 인스턴스인지 확인한다.
- toMatch(regexp | string) 문자열이 정규 표현식 또는 문자열과 일치하는지 확인한다.
- expect.anything() 아무 값이나 허용하지만 null이나 undefined는 제외한다.
- expect.any(constructor) 특정 생성자의 인스턴스인지 확인한다.
- expect.arrayContaining(array) 입력된 array가 비교 대상 array의 subset인지 확인한다. (전부 포함하는지)
- expect.objectContaining(object) 입력된 객체가 비교 대상 객체의 subset인지 확인한다. (전부 포함하는지)
- expect.stringContaining(string) 특정 문자열이 포함 돼있는지 확인한다.

### arrayContaining()

```tsx
describe("arrayContaining", () => {
  const expected = ["Alice", "Bob"];

  it("matches even if received contains additional elements", () => {
    expect(["Alice", "Bob", "Eve"]).toEqual(expect.arrayContaining(expected));
  });
});
```

### Modifiers

- not : 반대 테스트로 전환
- resolves : Promise 정상 반환으로 전환
- rejects : Promise 던지는 상황으로 전환

### not

![alt text](/Nest/image/testing4.png)

### Resolve

![alt text](/Nest/image/testing5.png)

### Reject

![alt text](/Nest/image/testing6.png)

### Mock/ Stub / Fake

테스트할때 의존성 (Dependency)를 해결하는 방법이 다양하게 존재한다. 모든 의존성 (데이터베이스등)을 그대로 사용하는 테스트도 존재하지만 그런 테스트는 너무 무겁고 오래걸린다. 일반적으로 디펜던시를 각자 객체로 스왑 후 사용한다.

Mock

- Mock은 상호작용을 검증하는 객체이다.

Stub

- Stub은 함수나 객체의 간소화된 버전으로 미리 정의된 값을 반환한다.

Fake

- Fake는 실제 객체를 간소하게 구현한 형태이다. 복잡한 실제 객체의 작동 방식을 최소화하여 구현한 형태이다. 실제 객체는 너무 헤비하지만 Stub 보다는 현실적인 작동이 필요할때 많이 사용된다.

의존성 해결을 해주는 객체가 셋중 꼭 어느 하나에 속한다고 생각할 필요는 없다. Mock이면서 Stub일 수 있다. 명칭은 위와 같이 정의하지만 일반적으론 일괄적으로 Mock이라고 부른다.

### Mock

```tsx
import { Test, TestingModule } from "@nestjs/testing";
import { UserService } from "./user.service";
import { UserRepository } from "./user.repository";

describe("UserService with Mock", () => {
  let userService: UserService;
  let userRepositoryMock: { findById: jest.Mock };

  beforeEach(async () => {
    userRepositoryMock = { findById: jest.fn() }; // Mock 생성하기

    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UserService,
        { provide: UserRepository, useValue: userRepositoryMock },
      ],
    }).compile();

    userService = module.get<UserService>(UserService);
  });

  it("should call findById on UserRepository", () => {
    const userId = "1";
    userService.findUserById(userId);

    // 실행된 값 확인
    expect(userRepositoryMock.findById).toHaveBeenCalledWith(userId);

    // 한 번만 호출된 확인
    expect(userRepositoryMock.findById).toHaveBeenCalledTimes(1);
  });
});
```

### Stub

```tsx
import { Test, TestingModule } from "@nestjs/testing";
import { UserService } from "./user.service";
import { UserRepository } from "./user.repository";

describe("UserService with Stub", () => {
  let userService: UserService;

  beforeEach(async () => {
    const userRepositoryStub = {
      findById: (id: string) => ({ id, name: "Stubbed User" }), // Stub 생성하기
    };

    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UserService,
        { provide: UserRepository, useValue: userRepositoryStub },
      ],
    }).compile();

    userService = module.get<UserService>(UserService);
  });

  it("should return the stubbed user", () => {
    const userId = "1";
    const result = userService.findUserById(userId);

    // 반환값 검증
    expect(result).toEqual({ id: userId, name: "Stubbed User" });
  });
});
```

### Fake

```tsx
import { Test, TestingModule } from "@nestjs/testing";
import { UserService } from "./user.service";
import { UserRepository } from "./user.repository";

// Fake 생성
class FakeUserRepository {
  private users = [{ id: "1", name: "Fake User" }];

  findById(id: string) {
    return this.users.find((user) => user.id === id) || null;
  }
}

describe("UserService with Fake", () => {
  let userService: UserService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UserService,
        { provide: UserRepository, useClass: FakeUserRepository },
      ],
    }).compile();

    userService = module.get<UserService>(UserService);
  });

  it("should return the fake user", () => {
    const userId = "1";
    const result = userService.findUserById(userId);

    // 결과 값 검증
    expect(result).toEqual({ id: userId, name: "Fake User" });
  });

  it("should return null if user is not found", () => {
    const userId = "2";
    const result = userService.findUserById(userId);

    // 결과 값 검증
    expect(result).toBeNull();
  });
});
```

![alt text](/Nest/image/testing7.png)

### Mock Function

```tsx
import { Test, TestingModule } from "@nestjs/testing";
import { UserService } from "./user.service";
import { UserRepository } from "./user.repository";

describe("UserService with Mock", () => {
  let userService: UserService;
  let userRepositoryMock: { findById: jest.Mock };

  beforeEach(async () => {
    userRepositoryMock = { findById: jest.fn() }; // Mock 생성하기

    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UserService,
        { provide: UserRepository, useValue: userRepositoryMock },
      ],
    }).compile();

    userService = module.get<UserService>(UserService);
  });

  it("should call findById on UserRepository", () => {
    const userId = "1";
    userService.findUserById(userId);

    // 실행된 값 확인
    expect(userRepositoryMock.findById).toHaveBeenCalledWith(userId);

    // 한 번만 호출된 확인
    expect(userRepositoryMock.findById).toHaveBeenCalledTimes(1);
  });
});
```

### Mock Function 속성 접근자

- mockFn.mock.calls: mock function이 호출된 모든 파라미터들의 배열을 포함한다.
- mockFn.mock.results: mock function의 각 호출이 반환한 값 또는 예외를 포함하는 객체의 배열이다.
- mockFn.mock.instances: mock function이 호출될 때마다 생성된 this 인스턴스를 포함하는 배열이다.

### mock.instances

```tsx
const mockFn = jest.fn();

const a = new mockFn();
const b = new mockFn();

mockFn.mock.instances[0] === a; // true
mockFn.mock.instances[1] === b; // true
```

### Mock Function 구현

- mockFn.mockImplementation(fn) : mock function의 구현체를 변경한다. (실행할 함수
- mockFn.mockImplementationOnce(fn) : mockImplementation을 단 한번만 실행한다. 여러번 chaining 가능하다.
- mockFn.mockReturnThis() : mock function이 호출될 때마다 this를 반환하도록 설정한다.
- mockFn.mockReturnValue(value) : mock function이 호출될 때마다 특정 값을 반환하도록 설정한다.
- mockFn.mockReturnValueOnce(value) : mockReturnValue를 단 한번만 실행한다. 여러번 chaining 가능하다.
- mockFn.mockResolvedValue(value) : mock function이 호출될 때 Promise가 특정 값으로 Resolve 되도록 한다.
- mockFn.mockResolvedValueOnce(value) : mockResolvedValue를 단 한번만 실행한다. 여러번 chaining 가능하다.
- mockFn.mockRejectedValue(value) : mock function이 호출될 때 Promise가 특정 값으로 Reject 되도록 설정한다.
- mockFn.mockRejectedValueOnce(value) : mock function의 다음 한 번의 호출에 대해 프로미스가 특정 값으로 거부되도록 설정한다.

### mockImplementation()

```tsx
const mockFn = jest.fn((scalar) => 42 + scalar);

mockFn(0); // 42
mockFn(1); // 43

mockFn.mockImplementation((scalar) => 36 + scalar);

mockFn(2); // 38
mockFn(3); // 39
```

### mockReturnThis()

```tsx
jest.fn(function () {
  return this;
});
```

### mockReturnValue()

```tsx
jest.fn().mockImplementation(() => value);
```

### mockResolvedValue()

```tsx
jest.fn().mockImplementation(() => Promise.resolve(value));
```

### mockRejectedValue()

```tsx
jest.fn().mockImplementation(() => Promise.reject(value));
```

### Mock Function 구현

- mockFn.mockClear(): mock function의 호출 기록과 반환 값들을 지운다 (상태 초기화).
- mockFn.mockReset(): mockClear()의 기능을 모두 실행하고 mock 함수를 빈 함수로 대체한다.
- mockFn.mockRestore(): mockReset()의 작업을 모두 진행하고 mock 함수를 원래 구현체로 복원한다.

![alt text](/Nest/image/testing8.png)

<br>

![alt text](/Nest/image/testing9.png)

<br>

![alt text](/Nest/image/testing10.png)

<br>

![alt text](/Nest/image/testing11.png)

<br>

![alt text](/Nest/image/testing12.png)

## Unit Testing 예제

```tsx
// UserController

@Controller("users")
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get(":id")
  async getUserById(@Param("id", ParseIntPipe) id: number): Promise<User> {
    return this.userService.findUserById(id);
  }
}
```

```tsx
// UserService

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>
  ) {}

  async findUserById(id: number): Promise<User> {
    if (id === 2) return null;

    return await this.userRepository.findOne({
      where: {
        id,
      },
    });
  }
}
```

```tsx
describe("UserController", () => {
  let userController: UserController;
  let userService: UserService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      controllers: [UserController],
      providers: [
        {
          provide: UserService,
          useValue: {
            findUserById: jest.fn(),
          },
        },
      ],
    }).compile();

    userController = module.get<UserController>(UserController);
    userService = module.get<UserService>(UserService);
  });

  it("should call UserService.findUserById with the correct parameter", async () => {
    const id = "1";
    const userServiceSpy = jest
      .spyOn(userService, "findUserById")
      .mockReturnValue({ id, name: "John Doe" });

    const result = await userController.getUserById(id);

    expect(userServiceSpy).toHaveBeenCalledWith(id);
    expect(result).toEqual({ id, name: "John Doe" });
  });
});
```

## Integration Testing 예제

```tsx
//UserController

@Controller("users")
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get(":id")
  async getUserById(@Param("id", ParseIntPipe) id: number): Promise<User> {
    return this.userService.findUserById(id);
  }
}
```

```tsx
// UserService

@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>
  ) {}

  async findUserById(id: number): Promise<User> {
    if (id === 2) return null;

    return await this.userRepository.findOne({
      where: {
        id,
      },
    });
  }
}
```

```tsx
describe("UserController (Integration)", () => {
  let userController: UserController;
  let userService: UserService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      controllers: [UserController],
      providers: [UserService],
    }).compile();

    userController = module.get<UserController>(UserController);
    userService = module.get<UserService>(UserService);
  });

  it("should return user data for a valid ID", () => {
    const id = "1";

    const result = userController.getUserById(id);

    expect(result).toEqual({ id: "1", name: "John Doe" });
  });

  it("should return null for an invalid ID", () => {
    const id = "2";

    const result = userController.getUserById(id);

    expect(result).toBeNull();
  });
});
```

## End to End Testing 예제

```tsx
describe("UserController (E2E)", () => {
  let app: INestApplication;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  it("/users/:id (GET) - should return user data for a valid ID", async () => {
    const id = "1";
    const response = await request(app.getHttpServer())
      .get(`/users/${id}`)
      .expect(200);

    expect(response.body).toEqual({ id: "1", name: "John Doe" });
  });

  it("/users/:id (GET) - should return 404 for an invalid ID", async () => {
    const id = "2";
    await request(app.getHttpServer()).get(`/users/${id}`).expect(404);
  });

  afterAll(async () => {
    await app.close();
  });
});
```

## Coverage

![alt text](/Nest/image/testing13.png)

## Coverage(Statement)

```tsx
// Statement
const sum = (a, b) => a + b;
// Statement
console.log(sum(1, 2));
```

## Coverage(Branch)

```tsx
if (x > 10) {
  // 하나의 분기
  console.log("x는 10보다 큽니다");
} else {
  // 하나의 분기
  console.log("x는 10 이하입니다.");
}
```

## Coverage(Function)

```tsx
function add(a, b) {
  // 함수입니다.
  return a + b;
}

function subtract(a, b) {
  // 함수입니다.
  return a - b;
}
```

## Coverage(Line)

```tsx
const mulitiply = (a, b) => {
  // 라인
  return a * b;
};

// 라인
console.log(multiply(2, 3));
```

## Coverage Report

![alt text](/Nest/image/testing14.png)

## Coverage Report

![alt text](/Nest/image/testing15.png)

## Coverage Report

![alt text](/Nest/image/testing16.png)

## 테스트 하지 않은것들

- 프레임워크 기능

  - 근본적으로 프레임워크 자체적으로 유닛 테스트가 있을거란 가정을 한다. 예를들어 NestJS의 UseGuard Annotation이 잘 작동하는지 테스트 하지 않는다. NestJS 프레임워크에서 테스트가 잘 됐을거라고 가정한다. 그럼에도 정말 하고싶다면 절대로 하면 안되는건 아니다.

- 외부 디펜던시

  - 낮은 수준의 테스트일수록 (Unit Test, Integration Test) TypeORM, Logger등 외부 디펜던시가 잘 작동하는지 테스트 하지 않는다. 대신 Mock, Stub, Fake를 사용해서 기능을 모방하고 실제 내 코드상의 중요한 로직을 테스트한다. 근본적으로 내 코드가 아니면 테스트 하지 않는다.

- 퍼포먼스

  - 퍼포먼스 테스트는 보통 다른 로드테스트 툴을 사용해서 진행한다. Unit Test, Integration Test, End to End Test 등은 근본적으로
    로직의 정상 작동 여부를 테스트한다. 퍼포먼스와 로드 테스트는 따로 진행하도록 한다

- 로직이 없는 코드
  - 초보자들이 coverage를 올리기 위해서 흔히 하는 실수다. NestJS를 예를들면 Dto나 Entity를 테스트 할 필요 없다. 그냥 ignore 리스트에 넣어버리자.
