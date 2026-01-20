# Cookie

## 트러블 슈팅: 개발 서버에서 쿠키가 정상적으로 저장되지 않는 이슈

로컬 환경에서는 정상적으로 동작하던 쿠키 설정 로직이 개발 서버에 배포된 이후 제대로 동작하지 않는 이슈가 발생했다.

초기에는 `httpOnly` 옵션이 `true`로 설정되어 있어 클라이언트에서 쿠키 접근이 제한되는 문제가 원인이라고 판단했다. 이에 따라 `NODE_ENV`를 기준으로 프로덕션 환경에서만 `httpOnly`를 활성화하도록 분기 처리했지만, 해당 수정만으로는 문제가 해결되지 않았다.

조사 결과, 문제의 핵심은 `httpOnly` 옵션이 아니라 **브라우저의 쿠키 정책**, 특히 `SameSite`와 `Secure` 옵션 조합에 있었다.

개발 서버 환경에서는 프론트엔드와 백엔드가 서로 다른 도메인(또는 서브도메인)을 사용하고 있었고, 이로 인해 `SameSite` 설정이 적절하지 않으면 쿠키가 브라우저에 저장되지 않았다. 특히 `SameSite=None`을 사용하는 경우 `Secure=true`가 반드시 함께 설정되어야 하며, 이 조건을 만족하지 않으면 브라우저에서 쿠키를 자동으로 차단한다는 점을 확인했다.

결과적으로 환경(local / dev / prod)에 따라 쿠키 옵션(`httpOnly`, `secure`, `sameSite`)을 명확히 분리하여 설정함으로써 개발 서버에서도 쿠키가 정상적으로 저장되고 전송되는 것을 확인할 수 있었다.

---

## Troubleshooting: Cookie not working properly on the development server

A cookie-setting logic that worked correctly in the local environment failed after being deployed to the development server.

At first, the issue was assumed to be caused by the `httpOnly` option being set to `true`, which prevents client-side JavaScript from accessing cookies. Based on this assumption, the logic was modified to enable `httpOnly` only in the production environment using `NODE_ENV`.

However, this change alone did not resolve the issue.

Further investigation revealed that the root cause was not `httpOnly`, but the **browser’s cookie policy**, particularly the interaction between `SameSite` and `Secure`.

In the development environment, the frontend and backend were running on different domains (or subdomains). Under this setup, cookies are not stored unless the `SameSite` option is configured correctly. It was confirmed that when `SameSite=None` is used, the `Secure=true` option must be set as well. If this condition is not met, modern browsers automatically block the cookie.

By explicitly configuring cookie options (`httpOnly`, `secure`, `sameSite`) based on the environment

(local / development / production), the issue was resolved and cookies were successfully stored and sent on the development server.

---

## 쿠키 옵션 정리 (httpOnly / Secure / SameSite)

### 1. `httpOnly`

- **설명**
    
    JavaScript(`document.cookie`)로 쿠키에 접근할 수 없도록 하는 옵션
    
- **목적**
    
    XSS 공격으로부터 쿠키 탈취를 방지
    
- **특징**
    - 서버 → 브라우저 전송은 가능
    - 클라이언트 JS에서는 접근 불가
- **주의사항**
    - 프론트엔드에서 쿠키 값을 직접 읽어야 하는 경우 사용 불가

```tsx
httpOnly:true
```

---

### 2. `Secure`

- **설명**
    
    HTTPS 연결에서만 쿠키를 전송하도록 제한하는 옵션
    
- **목적**
    
    중간자 공격(MITM) 방지
    
- **특징**
    - `Secure=true`인 쿠키는 **HTTPS 환경에서만 동작**
- **주의사항**
    - `SameSite=None`을 사용할 경우 **반드시 `Secure=true` 필요**
    - 로컬(http) 환경에서는 동작하지 않음

```tsx
secure:true
```

---

### 3. `SameSite`

- **설명**
    
    다른 사이트에서 요청이 발생했을 때 쿠키를 함께 전송할지 결정하는 옵션
    
- **종류**

| 값 | 설명 |
| --- | --- |
| `Strict` | 동일 사이트 요청에서만 쿠키 전송 |
| `Lax` | 일부 GET 요청(링크 이동 등)에만 쿠키 전송 |
| `None` | 모든 요청에 쿠키 전송 (크로스 사이트 허용) |
- **중요 포인트**
    - 프론트엔드와 백엔드 도메인이 다른 경우 (`SameSite=None` 필요)
    - `SameSite=None` 사용 시 **반드시 `Secure=true` 설정**

```tsx
sameSite:'none'
```

---

## 환경별 쿠키 설정 예시

```tsx
const isProd = process.env.NODE_ENV ==='production';

res.cookie('token', token, {
httpOnly:true,
secure: isProd,// prod에서만 true
sameSite: isProd ?'none' :'lax',
path:'/',
});
```