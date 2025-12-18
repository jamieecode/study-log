# Next.js 16

## 컴포넌트 캐싱

“use cache” 로 페이지, 컴포넌트, 함수 단위의 명시적인 캐싱 지원

이전 버전의 App Router에서 제공하던 암묵적 캐싱 대신 옵트인 방식으로 변경. 기존값은 동적 실행이며 모든 동적 코드가 요청시 실행됨(부분 사전 렌더링 Partial Pre-Rendering, PPR)

```jsx
// next.config.ts
const nextConfig = {
  cacheComponents: true,
};
 
export default nextConfig;
```

“use client” “use server” “use memo” 등등 지시어의 역할을 명확하게 이해하고 사용해야

---

## Next.js Devtools MCP

Next.js의 라우팅, 캐싱, 렌더링 동작, 통합 로그, 스택 트레이스, 현재 라우트 등의 컨텍스트를 AI 에이전트에게 제공할 수 있다.

.mcp.json 파일을 루트에 추가하면 됨.

```jsx
{
  "mcpServers": {
    "next-devtools": {
      "command": "npx",
      "args": ["-y", "next-devtools-mcp@latest"]
    }
  }
}
```

---

## middleware.ts →proxy.ts

명확한 네트워크 경계를 위한 조치. 요청 인터셉트 로직은 동일하게 유지하면서 이름만 변경됨.

마이그레이션을 위해서는 기존 middleware.ts 파일명을 proxy.ts로 변경하고 export 하는 함수명을 아래와 같이 proxy로 변경

```jsx
// proxy.ts
export default function proxy(request: NextRequest) {
  return NextResponse.redirect(new URL('/home', request.url));
}
```

---

## 라우팅 및 내비게이션 개선

### 레이아웃 중복 제거(Layout deduplication)

- 동일 레이아웃을 공유하는 여러 링크를 프리페칭할 때 레이아웃이 한 번만 다운로드됨

### 증분 프리페칭(Incremental prefetching)

- 이미 캐시된 부분은 재요청하지 않고, 뷰포트를 떠나면 요청을 취소하거나 우선순위를 조정하는 등의 제어 가능

---

## 캐싱 API 개선

### revalidateTag() API

- revalidateTag(tag, profile) 형태로 변환. stale-while-revalidate(SWR) 동작 지원

### updateTag(tag) API 추가

- Server Actions 내에서 캐시를 바로 만료하고 최신 데이터를 즉시 반영 가능

### refresh() API 추가

- Server Action 수행 후 페이지 다른 위치에 표시된 캐시되지 않은 데이터를 새로고침해야할 때 사용
- 캐시된 내용에는 영향을 주지 않으면서 동적 데이터만 갱신 가능
- 클라이언트의 router.refresh() 보완

---

## React 19.2 와 Canary 기능 지원

### viewTransitions

- 네비게이션, 업데이트 시 UI 애니메이션 지원

### useEffectEvent()

### <Activity />

- UI를 숨기고 상태를 유지하며 백그라운 작업을 렌더링할 수 있는 컴포넌트