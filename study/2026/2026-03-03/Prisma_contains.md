# Prisma_contains

## 1. `contains`

Prisma에서 **문자열 필드에 특정 문자열이 포함되어 있는지**를 조건으로 조회할 때 사용하는 필터

SQL의 `LIKE '%keyword%'`와 동일한 역할

```tsx
await prisma.user.findMany({
  where: {
    name: {
      contains:'kim',
    },
  },
});
```

SQL로 보면

```sql
WHERE nameLIKE'%kim%'
```

---

## 2. 문자열 필터 종류 비교

| 필터 | 설명 | SQL 변환 |
| --- | --- | --- |
| `equals` | 정확히 일치 | `=` |
| `contains` | 포함 | `LIKE '%value%'` |
| `startsWith` | 시작 | `LIKE 'value%'` |
| `endsWith` | 끝 | `LIKE '%value'` |

---

## 3. 대소문자 무시 옵션 (`mode`)

PostgreSQL 기준으로 `mode: 'insensitive'` 옵션을 사용하면 대소문자 구분 없이 검색 가능

```tsx
await prisma.user.findMany({
  where: {
    name: {
      contains:'kim',
      mode:'insensitive',
    },
  },
});
```

---

## 4. AND / OR 조건과 함께 사용

```tsx
await prisma.user.findMany({
  where: {
    OR: [
      { name: { contains:'kim' } },
      { email: { contains:'naver.com' } },
    ],
  },
});
```

---

## 5. 타입별 주의사항

### 숫자 타입에는 사용 불가

```tsx
where: {
	age: {
		contains:'2', // 오류 - ❌
  },
}
```

---

### 배열 타입(String[])에는 `has` 사용

```tsx
where: {
	tags: {
		has:'backend',
  },
}
```

관련 연산자

| 연산자 | 설명 |
| --- | --- |
| `has` | 특정 값 포함 |
| `hasSome` | 일부 포함 |
| `hasEvery` | 모두 포함 |

---

## 6. 성능 이슈

`contains`는 `%keyword%` 형태이므로 인덱스를 제대로 타지 못하는 경우가 많다.

대용량 데이터에서 고려해야 할 점

- `startsWith`가 더 빠를 수 있음
- Full Text Search 도입 고려
- 검색 전용 컬럼 분리
- 검색 조건이 많은 경우 ElasticSearch 등 도입 검토