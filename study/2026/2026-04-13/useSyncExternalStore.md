# useSyncExternalStore

**React 외부에 존재하는 상태(external store)**를 React 컴포넌트와 **안전하게 동기화하기 위한 Hook**

React가 직접 관리하지 않는 데이터 소스를 사용할 때, **동시성 렌더링 환경에서도 일관된 상태를 보장**하기 위해 사용된다.

---

컴포넌트가 React 외부 데이터 소스를 읽어야 할 때 사용한다.

- **브라우저 API**
    - `navigator.onLine` (온라인 상태)
    - `document.visibilityState` (페이지 가시성)
    - `window.matchMedia` (미디어 쿼리)
- **외부 상태 관리 라이브러리**
    - 구버전 Redux, MobX
    → 최신 Redux, Zustand, Jotai 등은 내부적으로 `useSyncExternalStore` 기반으로 동작
- **기타 외부 상태**
    - 전역 변수
    - 커스텀 이벤트 시스템
    - WebSocket 상태

---

### 기존 방식의 문제점

과거에는 `useEffect + useState`로 외부 상태를 구독했다.

```
useEffect(() => {
constunsubscribe=store.subscribe(() => {
setState(store.getState());
  });
returnunsubscribe;
}, []);
```

이 방식은 단순한 경우에는 문제가 없지만, **React 18의 동시성 렌더링(Concurrent Rendering)** 환경에서는 문제가 발생할 수 있다.

---

### 동시성 렌더링

React가 UI 업데이트를 **중단, 재개, 취소**하면서 여러 작업을 동시에 처리하는 방식 → 사용자 경험(UX) 향상

### 문제: 테어링(Tearing)

렌더링 도중 외부 상태가 변경되면서 **컴포넌트마다 서로 다른 상태 값을 읽는 현상**

- A 컴포넌트 → 이전 값
- B 컴포넌트 → 최신 값
→ UI가 "찢어진 것처럼" 불일치 발생(useSyncExternalStore로 해결)

---

- 모든 컴포넌트가 **동일한 시점의 스냅샷**을 읽음
- 렌더링 도중 상태가 바뀌어도 **일관성 유지**
- 테어링 방지

---

## API 구조

```tsx
useSyncExternalStore(
  subscribe,
  getSnapshot,
  getServerSnapshot?
)
```

---

### 1. subscribe(callback)

외부 스토어를 구독하는 함수

- React가 전달하는 `callback`을 반드시 호출해야 함
- 상태가 변경될 때마다 `callback()` 실행
- **반드시 unsubscribe 함수 반환**

```tsx
const unsubscribe=subscribe(callback);
```

React는 다음 시점에 cleanup 실행:

- 컴포넌트 언마운트
- subscribe 함수가 변경될 때

---

### 2. getSnapshot()

현재 스토어의 값을 반환하는 함수

- 반드시 **동기적(synchronous)** 이어야 함
- 반드시 **순수 함수(pure)** 이어야 함 (side effect 없음)
- React가 여러 번 호출할 수 있음 → **빠르게 실행되어야 함**

> 값이 변경되지 않았다면 **같은 참조(reference)**를 반환해야 한다
> 

```tsx
// 잘못된 예
return { ...state };// 매번 새로운 객체

// 올바른 예
return state;
```

React는 `Object.is` 비교로 리렌더링 여부 판단

---

### 3. getServerSnapshot() (선택)

SSR(Server-Side Rendering)에서 사용

- 서버에서 사용할 초기 스냅샷 반환
- hydration 시 클라이언트 값과 일치해야 함

### 주의사항

- 브라우저 API 기반이면 서버에서는 값이 없음 → fallback 값 필요
- 제공하지 않으면 hydration mismatch 또는 Suspense 상태 진입 가능