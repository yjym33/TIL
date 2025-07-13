![alt text](/Nest/image/codeFactory.png)
출처 : https://fastcampus.co.kr/dev_online_nestjs

<br>

## RBAC 이란?

- **RBAC는 Role Based Access Control의 약자다. 역할(Role) 기반으로 권한 (Permission)을 나누어서 특정 리소스에 CRUD 작업을 할 수 있는지 여부를 결정한다.**

<br>

## 구현방식

```tsx
// Role Enum
export enum Role {
  Admin = "admin",
  User = "user",
  Guest = "guest",
}
```

<br>

```tsx
// rbac.guard.ts 클래스

import { Injectable, CanActivate, ExecutionContext } from "@nestjs/common";
import { Reflector } from "@nestjs/core";
import { Role } from "./role.enum";

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get<Role[]>("roles", context.getHandler());
    if (!roles) {
      return true;
    }
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    return roles.some((role) => user?.roles?.includes(role));
  }
}
```

<br>

### 추가 설명

<br>

## RBAC (Role-Based Access Control) 이란?

RBAC은 시스템 내 자원(Resource)에 대한 접근 권한을 사용자(User)가 아닌 역할(Role)에 기반하여 제어하는 권한 관리 방식입니다.
사용자는 직접 권한을 부여받는 대신, **하나 이상의 역할(Role)**에 할당되고, 해당 역할이 갖는 권한을 통해 자원에 접근하게 됩니다.

<br>

## ✅ 핵심 개념

| 요소           | 설명                                                                           |
| -------------- | ------------------------------------------------------------------------------ |
| **User**       | 시스템을 사용하는 주체 (예: 회원, 관리자 등)                                   |
| **Role**       | 권한의 집합. 유사한 권한을 가진 사용자들을 그룹핑한 추상 개념                  |
| **Permission** | 특정 리소스에 대한 수행 가능한 작업 (예: `create`, `read`, `update`, `delete`) |
| **Resource**   | 보호 대상인 데이터나 기능 (예: 게시글, 사용자 정보, 주문 데이터 등)            |

---

<br>

## 🔁 동작 흐름

1. **관리자(시스템 관리자)**는 리소스와 작업에 기반하여 `권한(Permission)`을 정의합니다.
2. 권한들을 묶어 **역할(Role)** 을 생성합니다.
3. 사용자는 **하나 이상의 역할**에 할당됩니다.
4. 사용자는 자신의 역할을 통해 **권한이 허용된 자원**만 접근 할 수 있습니다.

예시:

| 사용자 | 역할(Role) | 권한(Permission)                            |
| ------ | ---------- | ------------------------------------------- |
| alice  | Admin      | `user:create`, `user:delete`, `post:delete` |
| bob    | Editor     | `post:create`, `post:update`                |
| eve    | Viewer     | `post:read`                                 |

<br>

## RBAC 구성 모델

RBAC는 기본적으로 다음 세 가지 모델로 나뉘며, 조합하여 사용할 수 있습니다:

### 1. Flat RBAC (기본 모델)

- 사용자 → 역할 → 권한

- 단순하고 직관적

- 소규모 서비스에 적합

### 2. Hierarchical RBAC (계층형)

- 역할 간 상속 구조를 도입

- 예: Admin > Manager > User

- 상위 역할은 하위 역할의 권한을 모두 포함

### 3. Constrained RBAC (제약형)

- 분리된 역할 간 동시 할당 금지(예: 승인자와 요청자 역할은 동시에 할당 불가)

- 조직 내부 보안 규정에 맞춘 접근 제어 적용 가능

<br>

## 🧩 RBAC vs ABAC vs ACL 비교

| 항목            | RBAC             | ABAC (Attribute-Based)      | ACL (Access Control List) |
| --------------- | ---------------- | --------------------------- | ------------------------- |
| **기반**        | 역할             | 속성 (사용자, 환경, 리소스) | 자원별 사용자 권한 목록   |
| **유연성**      | 보통             | 높음                        | 낮음                      |
| **관리 편의성** | 높음             | 복잡함                      | 낮음                      |
| **사용 예**     | 기업 내부 시스템 | 정부기관, 은행 등           | 간단한 파일 시스템        |

<br>

## 장점

- 관리 용이성: 사용자 수가 많아도 역할 기반으로 권한 관리가 쉬움

- 보안 강화: 최소 권한 원칙(Least Privilege Principle)을 실현 가능

- 확장성: 역할 추가만으로 손쉽게 권한 확장 가능

- 유지보수 편리: 신규 인력이나 팀 변경 시 역할만 재지정하면 됨

<br>

## 단점 / 한계

- 세밀한 정책 표현 어려움: ABAC보다 조건 기반 제어가 약함

- 역할이 많아질수록 복잡도 증가: 역할 수가 늘면 관리 부담 발생

- 직무 변경에 민감: 역할과 실제 업무 간 괴리가 커질 수 있음
