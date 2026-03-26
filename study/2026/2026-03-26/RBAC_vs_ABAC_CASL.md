# RBAC vs ABAC/CASL

# RBAC vs ABAC

## RBAC (Role-Based Access Control)

- 역할(Role) 기반 접근 제어 → 사용자에게 역할을 부여하고, 해당 역할에 따라 권한을 부여하는 방식

### 예시

- admin → 모든 권한
- user → 읽기만 가능
- manager → 삭제 가능

### 구조

```
User → Role → Permission
```

### 코드 예시

```tsx
if (user.role==='admin') {
	return true;
}
```

### 장점

- 구조가 단순함
- 관리가 쉬움
- 소규모 프로젝트에 적합

### 단점

- 세밀한 제어 어려움
- 역할이 많아질수록 복잡도 증가
- "작성자만 수정 가능" 같은 조건 처리 어려움

---

## ABAC (Attribute-Based Access Control)

- 속성(Attribute) 기반 접근 제어 → 사용자, 리소스, 환경 등의 속성을 기준으로 권한을 판단하는 방식

### 예시

- 작성자만 수정 가능
- 같은 부서만 조회 가능
- 게시글 상태가 published일 때만 읽기 가능

### 구조

```
User 속성 + Resource 속성 + 조건 → 허용 여부 결정
```

### 코드 예시

```tsx
if (user.id===post.authorId) {
  return true;
}
```

### 장점

- 매우 정교한 권한 제어 가능
- SaaS / 협업툴에 적합
- 데이터 중심 설계 가능

### 단점

- 설계 복잡
- 테스트 비용 증가
- 성능 고려 필요

---

# 실무에서의 권한 설계

실무에서는 대부분 **RBAC + ABAC 혼합 전략**을 사용한다고 한다

### 1차 필터링 → RBAC

- admin
- user
- manager

### 2차 세밀 제어 → ABAC

- 작성자만 수정
- 부서 동일한 경우만 접근
- 상태 조건 검사

---

# CASL?

JavaScript/TypeScript 환경에서 ABAC 기반 권한 제어를 쉽게 구현할 수 있는 라이브러리

- 조건 기반 권한 정의 가능
- NestJS / React에서 함께 사용 가능
- Mongo / Prisma 등과 연동 가능

---

# NestJS에서 CASL 적용 방법

## 1. 설치

```
npm install @casl/ability
```

---

## 2. Ability 정의

```tsx
// ability.factory.ts
import { AbilityBuilder, createMongoAbility }from'@casl/ability';

export function defineAbility(user:User) {
  const { can, cannot, build }=newAbilityBuilder(createMongoAbility);

  if (user.role==='admin') {
    can('manage','all');
  } else {
    can('read','Post');
    can('update','Post', { authorId:user.id });
  }

  return build();
}
```

---

## 3. Service 레벨에서 권한 검사 (권장)

```tsx
if (ability.cannot('delete',post)) {
  throw newForbiddenException();
}
```

→ 실제 보안은 반드시 서버에서 처리해야 함

---

# React에서 CASL 적용 방법

프론트에서 CASL의 역할은 **보안이 아니라 UI 제어**

---

## 1. 설치

```
npm install @casl/ability @casl/react
```

---

## 2. Ability 생성

```tsx
export function defineAbilityFor(user:User) {
  const { can, build }=newAbilityBuilder(createMongoAbility);

  if (user.role==='admin') {
    can('manage','all');
  } else {
    can('read','Post');
    can('update','Post', { authorId:user.id });
  }

  return build();
}
```

---

## 3. Provider로 감싸기

```tsx
<AbilityProvider user={user}>
  <App/>
</AbilityProvider>
```

---

## 4. 버튼 제어 예시

```tsx
<Can I="update" this={post}>
  <button>수정</button>
</Can>
```

또는

```tsx
if (!ability.can('delete',post)) return null;
```

---

# 전체 아키텍처 흐름

```
React (UI 제어)
   ↓
NestJS (실제 권한 검증)
   ↓
DB
```

- React → 버튼 숨김 (UX)
- NestJS → 실제 접근 차단 (보안)