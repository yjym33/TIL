![alt text](/Nest/image/codeFactory.png)
출처 : https://fastcampus.co.kr/dev_online_nestjs

<br>

![alt text](/Nest/image/MappedTypes.png)

<br>

## **Partial**

```tsx
export class CreateUserDto {
  @IsString()
  readonly name: string;
  @IsEmail()
  readonly email: string;
  @IsString()
  readonly password: string;
}
```

```tsx
export class UpdateUserDto extends PartialType(createUserDto) {}
/*
{
	name?: string;
	email?: string;
	password?: string;
}
*/
```

<br>

## Pick

```tsx
export class CreateUserDto {
  @IsString()
  readonly name: string;
  @IsEmail()
  readonly email: string;
  @IsString()
  readonly password: string;
}
```

```tsx
export class LoginUserDto extends PickType(createUserDto, ['email', 'password') as const {}
/*
{
	email?: string;
	password?: string;
}
*/
```

<br>

## Omit

```tsx
export class CreateUserDto {
  @IsString()
  readonly name: string;
  @IsEmail()
  readonly email: string;
  @IsString()
  readonly password: string;
}
```

```tsx
export class PublicDto extends OmitType(createUserDto, ['password') as const {}
/*
{
	name: string;
	email: string;
}
*/
```

<br>

## Intersection

```tsx
export class UserDetailsDto {
  @IsString()
  readonly name: string;
  @IsEmail()
  readonly email: string;
}
```

```tsx
export class AddressDto {
  @IsString()
  readonly street: string;

  @IsString()
  readonly city: string;

  @IsString()
  readonly country: string;
}
```

```tsx
export class UserWithAddressDto extends IntersectionType(
  UserDetailsDto,
  AddressDto
) {}
/*
{
	name: string;
	email: string;
	street: string;
	city: string;
	country: string;
}
*/
```

<br>

## Composition

```tsx
export class UpdateCatDto extends PartialType(
  OmitType(CreateCatDto, ["name"] as const)
) {}
```

<br>

## 추가 개념정리 (GPT 기반)

<br>

## ✅ 1. Mapped Type이란?

### 📌 정의

Mapped Type은 기존 타입을 **재활용하여 새로운 타입을 생성**할 수 있도록 도와주는 기능입니다.

NestJS에서는 다음과 같은 상황에서 주로 사용됩니다:

- **`CreateUserDto`를 기반으로 `UpdateUserDto`를 만들고 싶은 경우**
- **일부 속성을 선택하거나 필수 → 선택(optional)으로 바꾸고 싶은 경우**

---

<br>

## ✅ 2. NestJS에서 제공하는 주요 MappedType 유틸

NestJS는 `@nestjs/mapped-types` 패키지를 통해 다음 유틸리티 타입을 제공합니다:

| 헬퍼 함수            | 설명                                                |
| -------------------- | --------------------------------------------------- |
| `PartialType()`      | 모든 속성을 선택적으로 바꿈 (`required → optional`) |
| `PickType()`         | 특정 속성만 골라 새 타입 생성                       |
| `OmitType()`         | 특정 속성을 제외하고 새 타입 생성                   |
| `IntersectionType()` | 여러 DTO를 결합하여 새 타입 생성                    |

> ✅ 이 유틸들은 모두 클래스를 입력으로 받고 클래스 반환 (=> DTO 재사용 시 매우 편리)

---

<br>

## ✅ 3. 예제: `CreateUserDto` → `UpdateUserDto`

### 📌 원본: `CreateUserDto`

```tsx
import { IsString, IsInt } from "class-validator";

export class CreateUserDto {
  @IsString()
  name: string;

  @IsInt()
  age: number;
}
```

---

<br>

### 📌 1) `PartialType` 사용 – 모든 필드를 optional로

```tsx
import { PartialType } from "@nestjs/mapped-types";
import { CreateUserDto } from "./create-user.dto";

export class UpdateUserDto extends PartialType(CreateUserDto) {}
```

> 👆 UpdateUserDto는 name?: string, age?: number 가 됩니다.

---

<br>

### 📌 2) `PickType` 사용 – 특정 필드만 선택

```tsx
import { PickType } from "@nestjs/mapped-types";

export class LoginDto extends PickType(CreateUserDto, ["name"] as const) {}
```

> 👆 LoginDto는 { name: string } 형태만 가짐

---

<br>

### 📌 3) `OmitType` 사용 – 특정 필드 제외

```tsx
import { OmitType } from "@nestjs/mapped-types";

export class UserWithoutAgeDto extends OmitType(CreateUserDto, [
  "age",
] as const) {}
```

> 👆 name만 포함된 DTO

---

<br>

### 📌 4) `IntersectionType` 사용 – 둘 이상의 DTO를 결합

```tsx
import { IntersectionType } from "@nestjs/mapped-types";

class AuthDto {
  @IsString()
  token: string;
}

export class UserWithAuthDto extends IntersectionType(CreateUserDto, AuthDto) {}
```

---

<br>

## ✅ 4. MappedType을 사용하는 이유

| 이유                            | 설명                                                          |
| ------------------------------- | ------------------------------------------------------------- |
| ✅ **DRY 원칙 준수**            | 중복 없이 기존 DTO 재활용 가능                                |
| ✅ **일관된 유효성 검사 유지**  | `class-validator` 데코레이터 유지됨                           |
| ✅ **자동 Swagger 문서화 연계** | `@nestjs/swagger`와 함께 사용할 때 API 문서도 자동으로 반영됨 |
| ✅ **타입 안정성 보장**         | TypeScript 타입 안전 보장                                     |

---

<br>

## ✅ 5. 주의사항

- `PartialType` 등은 **클래스를 인자로 받으므로**, DTO는 반드시 `class`로 정의되어야 합니다 (`interface`는 안 됨).
- NestJS의 `ValidationPipe`와 함께 쓸 경우, `@IsOptional()`을 명시하지 않아도 `PartialType`이 알아서 optional로 처리해줍니다.

---

<br>

## ✅ 요약

| 유틸리티           | 역할                        |
| ------------------ | --------------------------- |
| `PartialType`      | 모든 필드를 optional로 변환 |
| `PickType`         | 일부 필드만 선택            |
| `OmitType`         | 일부 필드를 제외            |
| `IntersectionType` | 여러 DTO를 합침             |
