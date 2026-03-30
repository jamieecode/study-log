# Cross Platform vs Native App

### 크로스 플랫폼 앱 (Cross Platform App)

하나의 소스 코드로 여러 운영체제(iOS, Android 등)에서 동작하도록 개발하는 방식

대표적인 프레임워크: React Native, Flutter

### 사용 목적

- 개발 비용 절감
- 개발 리소스 통합
- 빠른 MVP 제작
- 유지보수 효율성 증가

---

### 네이티브 앱 (Native App)

각 운영체제에 맞는 언어와 SDK를 사용하여 별도로 개발하는 방식

- iOS → Swift / Objective-C
- Android → Kotlin / Java

### 특징

- 플랫폼 최적화 성능
- OS 최신 기능 대응 빠름
- 플랫폼별 UI/UX 완벽 반영 가능

---

## React Native

React Native는 JavaScript(또는 TypeScript) 기반으로 개발하는 프레임워크

### 동작 방식 (기존 아키텍처)

```
JavaScript Code
        ↓
     Bridge
        ↓
Native Modules (iOS / Android)
```

- 런타임 중 JavaScript 브릿지를 생성
- JS 코드와 네이티브 코드 간 통신을 담당
- iOS, Android의 **표준 네이티브 UI 컴포넌트** 사용
- 브릿지 통신은 기본적으로 비동기 방식

### 단점

- 브릿지를 통한 통신 과정에서 성능 병목 발생 가능

---

### New Architecture

최근 React Native는 브릿지 의존도를 줄이기 위한 구조 도입

- JSI (JavaScript Interface)
- TurboModules
- Fabric Renderer

이를 통해:

- 브릿지 없는 동기 통신 지원
- 성능 개선
- 초기 로딩 속도 개선

---

### Metro

- JavaScript 번들러
- 빠른 Hot Reload 지원
- 대규모 모듈 처리 최적화

---

### 장점

- React 기반 → 웹 개발자 진입 장벽 낮음
- 네이티브 UI 컴포넌트 사용
- JavaScript 생태계 활용 가능
- 코드 재사용성 높음

### 단점

- 브릿지 기반 구조의 성능 이슈 가능성
- 네이티브 모듈 작성 시 러닝커브 존재
- 플랫폼별 UI 차이 대응 필요

---

## Flutter

Flutter는 Google이 개발한 크로스 플랫폼 UI 프레임워크

- 사용 언어: Dart
- 지원 플랫폼: iOS, Android, Web, Windows, macOS, Linux

---

### 동작 방식

Flutter는 네이티브 UI 컴포넌트를 사용하지 않음

```
Dart Code
    ↓
Flutter Engine (Skia)
    ↓
Canvas Rendering
```

- Skia 그래픽 엔진 기반 렌더링
- 모든 UI를 자체 위젯 트리로 직접 그림
- 브릿지 없이 플랫폼과 직접 통신

---

### 컴파일 전략

Flutter는 두 가지 컴파일 전략을 사용한다고 한다.

### 개발 중 → JIT (Just-In-Time)

- 빠른 Hot Reload
- 생산성 향상

### 배포 시 → AOT (Ahead-Of-Time)

- Dart 코드를 네이티브 코드로 사전 컴파일
- 빠른 실행 속도
- 성능 최적화

---

### 장점

- 브릿지 없음 → 높은 성능
- UI 일관성 유지 용이
- Hot Reload 빠름
- 멀티 플랫폼 지원 범위 넓음

### 단점

- 앱 용량이 비교적 큼
- 플랫폼 고유 UI와 차이 존재
- Dart 생태계가 JavaScript보다 작음

---

## WebView 기반 하이브리드 앱과의 차이

### WebView 기반

- Cordova
- Ionic
- PWA

HTML/CSS/JavaScript로 작성된 웹 앱을 WebView에 띄우는 방식.

### 특징

- 개발이 쉬움
- 성능은 상대적으로 낮음
- 네이티브 API 접근 제한적

---

### 네이티브 렌더링 기반

- React Native
- Flutter

네이티브 또는 엔진 기반으로 직접 렌더링하기 때문에 WebView 방식보다 성능이 우수
