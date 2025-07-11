![alt text](/Nest/image/codeFactory.png)
출처 : https://fastcampus.co.kr/dev_online_nestjs

<br>

![alt text](/Nest/image/Hashing.png)

<br>

![alt text](/Nest/image/Hashing2.png)

<br>

![alt text](/Nest/image/Hashing3.png)

<br>

![alt text](/Nest/image/Hashing4.png)

<br>

![alt text](/Nest/image/Hashing5.png)

<br>

![alt text](/Nest/image/Hashing6.png)

<br>

![alt text](/Nest/image/Hashing7.png)

<br>

![alt text](/Nest/image/Hashing8.png)

<br>

![alt text](/Nest/image/Hashing9.png)

<br>

# 🔐 NestJS 기반 해싱 인증 흐름 및 비밀번호 해싱 이론 정리

## ✅ 개요

사용자 인증에서 비밀번호는 가장 민감한 정보입니다. 해커가 DB에 접근하더라도 **비밀번호 원문이 유출되지 않도록 안전하게 보호**하는 것이 핵심입니다.

이를 위해 NestJS에서는 `bcrypt` 해시 알고리즘을 이용하여 안전한 비밀번호 처리 및 인증 시스템을 구성할 수 있습니다.

---

<br>

## 🧭 전체 회원가입 & 로그인 프로세스

1. **회원가입 요청**

   - 클라이언트에서 이메일/비밀번호를 서버로 전달

2. **비밀번호 해싱 후 저장**

   - bcrypt로 비밀번호 해시화
   - 이메일과 함께 DB에 저장

3. **로그인 요청**

   - 클라이언트에서 이메일/비밀번호 전달

4. **비밀번호 비교 검증**

   - bcrypt의 `compare()`로 해시값 비교

5. **JWT 토큰 발급**

   - Access Token + Refresh Token 생성

6. **클라이언트 전달**

   - 토큰을 클라이언트에 응답

7. **인증이 필요한 요청**
   - 토큰을 Authorization Header에 담아 API 호출

---

<br>

## 🔐 해싱(Hasing) vs 암호화(Encryption)

| 구분           | 해싱 (Hashing)                 | 암호화 (Encryption)       |
| -------------- | ------------------------------ | ------------------------- |
| 목적           | 무결성 확인, 데이터 식별, 인증 | 기밀성 유지 (복호화 가능) |
| 복원 가능 여부 | ❌ 복원 불가                   | ✅ 복원 가능              |
| 사용 예시      | 비밀번호 저장, 디지털 서명     | 메신저 메시지, 금융 정보  |
| 대표 알고리즘  | SHA256, bcrypt, argon2 등      | AES, RSA 등               |

---

<br>

## 🔎 해시 저장 방식의 특징

- **원본 비밀번호 저장 금지**

  - 서버가 해킹되더라도 원본 비밀번호를 알 수 없음

- **항상 해시된 값만 DB에 저장**

  - 평문이 아닌 해시 문자열 저장

- **같은 입력 → 같은 출력**

  - 해시 알고리즘은 동일 입력에 대해 동일한 출력값을 반환

- **복호화 불가**
  - 해시값은 다시 원래 값으로 복원 불가능

---

<br>

## 왜 Salt를 사용해야 하나?

**Salt**란: 해시하기 전 추가되는 **무작위 문자열**로서, 동일한 비밀번호라도 다른 결과가 나오도록 함.

- 동일한 비밀번호라도 다른 사용자마다 다른 해시값 생성
- Rainbow Table 공격(사전 해시 테이블 공격) 방지
- bcrypt는 내부적으로 salt를 생성하고 함께 해시함

### 예시

입력: 123123 → 해싱 결과: apzxlcbkjo31248sasdf <br>
입력: 123123 + salt → 해싱 결과: completely different

---

<br>

## 평문 비밀번호 저장의 위험성

### 해킹된 경우

| id  | email                | password |
| --- | -------------------- | -------- |
| 1   | jc@codefactory.ai    | 123123   |
| 2   | admin@codefactory.ai | abcabc   |

- 해커가 이 정보를 탈취하면 **다른 사이트 로그인 시도 가능**
- 동일한 이메일/비밀번호 조합 사용 시 **2차 피해 발생**

---

<br>

## ⚙️ NestJS 해싱 인증 구현 흐름

### 1. bcrypt 설치

```bash
npm install bcrypt
npm install -D @types/bcrypt
```

<br>

### 2. 회원가입 - 비밀번호 해시화

```ts
import * as bcrypt from "bcrypt";

const saltRounds = 10;
const hashedPassword = await bcrypt.hash(plainPassword, saltRounds);

// hashedPassword와 email을 DB에 저장
```

<br>

### 3. 로그인 - 비밀번호 비교

```ts
const user = await usersService.findOneByEmail(email);
const isMatch = await bcrypt.compare(inputPassword, user.password);

if (!isMatch) {
  throw new UnauthorizedException("비밀번호 불일치");
}

// isMatch === true면 로그인 성공 → JWT 토큰 발급
```

<br>

## 해싱 충돌, 무차별 대입, 사전 공격 방지 방법

| 공격 유형          | 대응 방식                              |
| ------------------ | -------------------------------------- |
| 무차별 대입 공격   | bcrypt의 느린 연산, salt 사용으로 방어 |
| Rainbow Table 공격 | salt 추가로 무력화                     |
| 해시 충돌 유도     | 충돌 가능성 거의 없음 (bcrypt 기준)    |

---

<br>

## SHA256 vs bcrypt 비교

| 항목          | SHA256                  | bcrypt                  |
| ------------- | ----------------------- | ----------------------- |
| 속도          | 매우 빠름               | 느림 (일부러)           |
| salt 필요     | ❌ 수동으로 처리해야 함 | ✅ 내부적으로 자동 생성 |
| 보안성        | 낮음 (속도 때문에)      | 높음 (해킹 대비 우수)   |
| 비밀번호 용도 | ❌ 적합하지 않음        | ✅ 가장 많이 사용됨     |

<br>

## 정리

- bcrypt는 보안성이 높고, 비밀번호 저장에 가장 적합한 알고리즘입니다.

- NestJS에서는 회원가입 시 bcrypt.hash(), 로그인 시 bcrypt.compare()를 사용하여 안전한 인증을 구현합니다.

- Salt를 통해 무작위성을 부여하여 같은 비밀번호라도 서로 다른 해시값이 생성되도록 해야 합니다.

- 절대로 평문 비밀번호를 DB에 저장해서는 안 됩니다.
