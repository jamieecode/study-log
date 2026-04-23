# WebView Bridge Pattern

모바일 앱에서 일부 화면을 웹으로 구성하면 앱스토어 재배포 없이 빠르게 UI를 수정·배포할 수 있다는 장점이 있다. 그래서 공지, 이벤트 페이지, 프로모션 화면, 설정 일부 등 변경이 잦은 영역을 WebView로 구현하는 경우가 많다.

다만 카메라 촬영, 연락처 접근, 파일 선택, 푸시 토큰 조회, 생체 인증처럼 OS 네이티브 기능이 필요한 작업은 웹뷰 내부의 자바스크립트가 직접 접근할 수 없다. 웹 코드는 브라우저 런타임에서 실행되고, 네이티브 코드는 iOS/Android 런타임에서 실행되기 때문이다.

이때 웹과 네이티브가 서로 메시지를 주고받도록 만드는 통신 계층을 WebView Bridge 패턴이라고 한다.

---

## WebView Bridge 패턴이 필요한 이유

웹뷰 안의 자바스크립트와 네이티브 코드는 서로 다른 실행 환경에서 동작하므로, 일반 함수 호출처럼 **동기적으로 값을 즉시 반환받을 수 없다.** 대신 다음과 같은 **비동기 메시지 기반 구조**를 사용한다.

1. 웹 → 네이티브로 요청 전송
2. 네이티브에서 작업 수행
3. 완료 후 네이티브 → 웹으로 결과 전달

---

## React Native WebView 기준 브릿지 통신 방식

### 1. 웹에서 네이티브로 메시지 보내기

웹 코드에서는 `window.ReactNativeWebView.postMessage()` 사용

```tsx
window.ReactNativeWebView.postMessage(
	JSON.stringify({
    type:"GET_PUSH_TOKEN",
    requestId:"1234"
  })
);
```

- 문자열 형태로 메시지를 전달한다.
- 일반적으로 JSON 직렬화 후 사용한다.
- `requestId`를 함께 보내 응답과 매칭한다.

---

### 2. 네이티브에서 웹으로 응답 보내기

React Native 측에서는 `WebView` ref를 통해 `injectJavaScript()`로 웹뷰 내부 자바스크립트를 실행할 수 있다.

```tsx
webViewRef.current.injectJavaScript(`
  window.onBridgeMessage({
    requestId: "1234",
    success: true,
    data: "token-value"
  });
`);
```

이를 통해 웹 쪽 콜백 함수를 호출하거나 상태를 전달한다.

---

## Promise를 사용하는 이유

브릿지 호출은 결과가 즉시 오지 않기 때문에 `Promise` 패턴으로 감싸는 것이 일반적이다.

```tsx
function getPushToken() {
	return new Promise((resolve,reject) => {
		const requestId = createId();

		pendingMap[requestId]= { resolve, reject };

		window.ReactNativeWebView.postMessage(
		JSON.stringify({
        type:"GET_PUSH_TOKEN",
        requestId
      })
    );
  });
}
```

이후 네이티브가 응답을 보내면:

```tsx
functiononBridgeMessage(message) {
	const { requestId, success, data, error }=message;

	constpending=pendingMap[requestId];
	if (!pending) return;

	if (success) pending.resolve(data);
	else pending.reject(error);

	delete pendingMap[requestId];
}
```

---

### 요청과 응답은 고유 ID로 매칭한다

브릿지 요청이 여러 개 동시에 발생할 수 있으므로 `requestId` 같은 고유 식별자가 필요하다.

### 비동기 결과는 Promise로 전달한다

웹에서는 일반 API 호출처럼 아래와 같이 사용할 수 있다.

```
consttoken=awaitgetPushToken();
```

### 브릿지는 메시지 프로토콜 설계가 중요하다

아래 항목을 미리 정하면 유지보수가 쉬워진다.

- `type` (요청 종류)
- `requestId`
- `success`
- `data`
- `error`

---

## 실무에서 추가로 고려할 점

**1. 타임아웃 처리**

네이티브 응답이 오지 않는 경우를 대비해 일정 시간 후 reject 처리 필요.

**2. 버전 관리**

앱 버전마다 지원하는 브릿지 API가 다를 수 있으므로 버전 체크 필요.

**3. 보안**

민감 정보 전달 시 평문 노출, 스크립트 주입 위험 등을 고려해야 한다.

**4. 플랫폼 차이**

iOS / Android 동작 방식이나 권한 처리 흐름이 다를 수 있다.