# ts-pattern

TypeScript에서 `switch / if-else`를 더 **선언적 + 타입 안전하게** 사용할 수 있게 해주는 패턴 매칭 라이브러리

- 분기 로직을 “값 비교”가 아니라 “패턴”으로 표현
- 모든 케이스 처리 여부를 컴파일 타임에 체크
- 복잡한 객체 조건까지 처리 가능

---

## 기본 사용법 (switch 대체)

```tsx
import { match }from "ts-pattern";

const message = match(status)
	.with("loading", () =>"로딩중")
	.with("success", () =>"성공")
	.with("error", () =>"실패")
	.otherwise(() =>"알 수 없음");
```

---

## 완전성 검사(Exhaustive Check)

모든 케이스를 처리하지 않으면 컴파일 에러 발생

```tsx
type Status = "loading"|"success"|"error";

const message = match<Status>(status)
	.with("loading", () =>"로딩중")
	.with("success", () =>"성공")
	.with("error", () =>"실패")
	.exhaustive();
```

→ 새로운 상태가 추가되면 반드시 처리하도록 강제됨

---

## 객체 패턴 매칭

```tsx
import { match, P }from "ts-pattern";

type User = {
  role: "admin" | "user";
  age: number;
};

const result = match(user)
	.with({ role:"admin" }, () =>"관리자")
	.with({ role:"user", age:P.when((age) =>age>=20) }, () =>"성인 유저")
	.otherwise(() =>"미성년자 또는 기타");
```

→ 단순 값 비교를 넘어서 조건까지 표현 가능

---

## API 응답 처리

### 기존 방식

```tsx
if (res.status === 200) {
	return res.data;
} else if (res.status === 401) {
	throw new Error("Unauthorized");
} else {
	throw new Error("Unknown error");
}
```

---

### ts-pattern 방식

```tsx
const result = match(res)
	.with({ status:200 }, (r) =>r.data)
	.with({ status:401 }, () => {
		throw new Error("Unauthorized");
  })
	.otherwise(() => {
		throw new Error("Unknown error");
  });
```

→ 상태 코드 기반 분기 로직이 명확해짐

---

## Zustand에서 사용

```tsx
import { match } from "ts-pattern";

type StoreState=
	| { status:"idle" }
	| { status:"loading" }
	| { status:"success"; data:string }
	| { status:"error"; error:string };

const view = match(store.getState())
	.with({ status:"idle" }, () => "대기")
	.with({ status:"loading" }, () => "로딩중")
	.with({ status:"success" }, (state) =>state.data)
	.with({ status:"error" }, (state) =>state.error)
	.exhaustive();
```