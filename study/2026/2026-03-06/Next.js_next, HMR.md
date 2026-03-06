# Next.js _.next, HMR

## Next.js .next cache

### `.next` directory

Next.js가 **빌드 결과물과 캐시 데이터를 저장하는 디렉토리**

포함되는 주요 내용:

- 빌드 결과물 (server / client bundle)
- 페이지별 컴파일 결과
- SWC 트랜스파일 캐시
- 이미지 최적화 캐시
- App Router의 RSC(Server Components) 관련 캐시
- HMR 관련 메타데이터

### `.next` 를 지워야 하는 경우

- next.config.ts 변경 후 반영이 안 될 때
    - `basePath`
    - `rewrites`
    - `redirects`
    - `env`
    - `images`
    - `experimental` 옵션 등
- SCSS/CSS 전역 스타일 변경이 반영 안 될 때
- 새 라우트 디렉토리 추가 후 404가 날 때
- 패키지 업그레이드 후 빌드 오류가 날 때
- 이상한 캐시 관련 오류가 날 때
    - ENOENT, Cannot find module, Hydration mismatch 등
- 타입 오류가 사라졌는데 빌드는 실패할 때

→  삭제 후 실

```tsx
rm -rf .next
npm run dev
```

---

### 지우지 않아도 되는 경우

- 일반 컴포넌트/페이지 수정 → Turbopack/HMR이 자동으로 처리
- 새 컴포넌트 파일 추가 (디렉토리 신규 생성이 아닌 경우)
- API 코드 수정

---

## HMR(Hot Module Replacement)

코드를 수정했을 때 전체 페이지를 새로고침하지 않고 변경된 모듈만 교체하는 기술(Fast Refresh)

- 컴포넌트 수정 시 즉시 반영
- React state 유지
- 문법 오류 발생 시 화면 유지
- 에러 수정 후 자동 복구

상태 유지 조건

- 함수 컴포넌트
- export 형태가 변경되지 않을 것
- Hook 호출 순서가 변경되지 않을 것

→ 위 조건이 깨지면 전체 리렌더링이 발생한다.

---

### 동작 원리

1. 파일 변경 감지
- Turbopack 또는 Webpack이 파일 변경 감지
- 변경된 모듈만 다시 번들링
1. WebSocket 통신
- 개발 서버 ↔ 브라우저 간 WebSocket 연결 유지
- 변경 사항을 클라이언트에 알림
1. 모듈 교체
- 변경된 모듈만 런타임에 교체
- 전체 페이지 reload 없이 반영

### serverComponentsHmrCache

App Router에서 Server Components를 사용할 때

- HMR 요청 간 fetch 응답을 캐싱
- 동일한 요청에 대해 불필요한 재실행 방지
- 개발 중 불필요한 네트워크 요청 감소

→ 개발 환경에서 RSC 재실행 비용을 줄이기 위한 캐시 전략