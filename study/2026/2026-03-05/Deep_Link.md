# Deep Link

## 1. 딥링크(Deep Link)

- **웹 링크(Web Link)**
    
    → 사용자를 특정 웹사이트 URL로 이동시킴
    
    → 예: `https://www.example.com`
    
- **딥링크(Deep Link)**
    
    → 사용자를 특정 **앱(App)** 으로 이동시켜
    
    - 원하는 화면을 바로 열거나
    - 특정 액션을 유도하는 링크

즉, 단순히 앱을 여는 것이 아니라 **앱 내부의 특정 화면까지 직접 연결하는 기술**이다.

---

# 2. 딥링크의 종류

## 2-1. Custom Scheme (URI Scheme)

```
myapp://product/123?ref=home
```

### 구조

- **App Scheme**: `myapp://`
- **Host**
- **Path**: 앱 내부 이동할 페이지
- **Query Parameters**: 추가 데이터 전달

### 특징

- 가장 오래되고 널리 사용되는 방식
- Android, iOS 모두 지원
- 앱 스킴(App Scheme), URI Scheme이라고도 불림

---

### Custom Scheme의 문제점

가장 큰 문제는 **스킴 충돌(Scheme Collision)** 이다.

- 서로 다른 앱이 동일한 스킴을 사용할 수 있음
- 앱 등록 시 기존 스킴 존재 여부 확인 불가

### 플랫폼별 차이

- **Android**
    - 같은 스킴을 가진 앱이 여러 개 있으면
    - "어떤 앱으로 열까요?" 선택 UI 표시
- **iOS**
    - 동일 스킴 충돌 시 명확한 해결 방법 없음
    - 의도하지 않은 앱이 실행될 가능성 존재

---

## 2-2. Universal Link (iOS) / App Link (Android)

2015년 Custom Scheme의 한계를 보완하기 위해 등장했으며 현재 가장 권장되는 방식

- iOS: **Universal Link**
- Android: **App Link**

---

### 동작 방식

표준 웹 URL을 사용

```
https://www.my-service.com/product/123
```

### 동작 시나리오

| 상황 | 동작 |
| --- | --- |
| 앱 설치 O | 앱 실행 + 해당 화면 이동 |
| 앱 설치 X | 웹 브라우저에서 해당 페이지 오픈 |

→ 사용자 경험(UX)이 매우 자연스러움

---

### 특징

- 표준 HTTPS 링크 사용
- 도메인 소유권 검증 필요
    - iOS: apple-app-site-association 파일
    - Android: assetlinks.json 파일
- 보안성 높음
- 스킴 충돌 문제 없음

---

### 주의점

Universal/App Link는 **사용자의 직접적인 액션(클릭)** 으로만 동작한다.

- JS로 자동 클릭 유도 시
    - 앱으로 이동하지 않고 웹 브라우저에서 열리는 경우가 많음

그래서 실무에서는

```
1. Universal/App Link 시도
2. 실패 시 Custom Scheme fallback
```

같은 방식으로 함께 사용하는 경우가 많다고 함

---

## 2-3. Intent Scheme (Android 전용)

```
intent://product/123#Intent;
scheme=myapp;
package=com.myapp.android;
S.browser_fallback_url=https://www.my-service.com;
end
```

### 특징

- Android 전용
- App Link 등장 이전에 WebView 환경에서 많이 사용
- 앱 패키지 정보 포함 가능
- fallback URL 지정 가능

### 장점

- 앱 미설치 시:
    - Play Store 이동
    - 특정 URL로 fallback
- 개발자가 세밀한 제어 가능

### 단점

- 구조가 복잡
- Android 전용

---

# 3. 실무에서 사용하는 구조

1. Universal/App Link 시도 → 실패 시 Custom Scheme → Android의 경우 Intent fallback
2. 웹 페이지 진입 → 서버에서 User-Agent 분석 → 플랫폼별 딥링크 분기 처리