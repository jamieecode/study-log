# OTP 인증번호 자동완성 기기/브라우저별 동작 불일치 이슈

### **이슈**

휴대폰 인증 과정에서 SMS로 수신한 인증번호가 입력창에 자동완성되는 기능이 **기기 및 브라우저 조합에 따라 다르게 동작**하는 현상 확인. `autocomplete="one-time-code"` 속성을 적용했으나 일부 환경에서는 정상 동작하지 않음.

---

### **테스트**

**기대 동작**

- SMS 수신 후 인증번호 입력란에 자동완성 제안이 노출되거나, 자동으로 입력되어 사용자가 수동 입력을 하지 않아도 인증 가능해야 함.

**실제 동작**

- 일부 안드로이드 기기(특히 최신 기기 + 삼성 인터넷 브라우저)에서 SMS 수신 후에도 OTP 자동완성 UI가 나타나지 않음.

| 기기 | 브라우저 | 결과 |
| --- | --- | --- |
| 갤럭시 S21 | Chrome | ✅ 정상 동작 |
| 갤럭시 S21 | 삼성 인터넷 | ✅ 정상 동작 |
| 갤럭시 S22 | Chrome | ✅ 정상 동작 |
| 갤럭시 S22 | 삼성 인터넷 | ✅ 정상 동작 |
| 갤럭시 Fold7 | Chrome | ✅ 정상 동작 |
| 갤럭시 Fold7 | 삼성 인터넷 | ❌ 동작 안 함 |
| iPhone | Safari | ✅ 정상 동작 |
| iPhone | Chrome | ✅ 정상 동작 (Safari 엔진 사용) |
- iOS Safari 계열은 정상동작, 삼성 갤럭시의 경우 일부 동작 하지 않음.

---

### **원인 분석**

관련해서 찾아봤을 때 로직 문제라기보다 **OS 및 브라우저의 OTP 파싱/자동완성 정책 차이**에서 비롯된 것으로 보인다. 이 기능은 웹 표준으로 완전히 제어 가능한 영역이 아니라, 모바일 OS와 브라우저가 SMS 내용을 해석해 제공하는 플랫폼 의존 기능에 가깝다.

OTP 자동완성은 단순한 HTML 기능이 아니라

- 모바일 OS가 SMS 내용을 분석하여 OTP 여부 판단
- 브라우저 및 키보드가 해당 정보를 자동완성 UI로 제공

하는 구조이기 때문에 다음 요소에 영향을 받을 수 있다고 한다.

| 요소 | 영향 |
| --- | --- |
| 안드로이드 OS 버전 | SMS OTP 감지 로직 상이 |
| 브라우저 엔진 (Chrome vs Samsung Internet) | 자동완성 지원 방식 차이 |
| SMS 메시지 포맷 | 숫자 위치, 문장 구조에 따라 인식률 달라짐 |
| 키보드 앱 | OTP 제안 표시 여부에 영향 |

---

### **권장 사항**

자동완성은 기기 의존성이 높아 100% 보장 불가하므로, 다음 보조 수단 병행 권장한다고 한다.

1. **SMS 포맷 개선**
    - 인증번호를 문장 앞 또는 단독 줄에 배치
        - OTP 자동완성 인식률은 SMS 내에서 **인증번호의 위치(문장 앞/단독 줄 여부)** 및 **숫자 주변 텍스트 구조**에 따라 달라질 수 있음
        
        ```
        482193
        OOO 인증번호입니다.
        ```
        
2. **WebOTP API 적용 (Android Chrome 전용 자동입력)**
    
    → 지원 환경에서는 문자 수신 시 자동으로 코드 입력 가능
    
3. **붙여넣기 UX 개선**
    
    → 사용자가 문자에서 복사 후 붙여넣기 시 숫자만 자동 추출
    
4. **인증 링크 포함 방식 (Fallback)**
    
    ```
    https://example.com/auth?code=482193
    ```
    
    → 어떤 기기/브라우저에서도 동작하는 최종 백업 수단
    

---

### **Issue**

We observed that the SMS OTP (one-time password) autofill behavior during phone verification **works inconsistently depending on device and browser combinations**.

Although `autocomplete="one-time-code"` has been applied, it does not function as expected in some environments.

---

### **Testing**

**Expected Behavior**

- After receiving the SMS, the OTP input field should show an autofill suggestion or automatically populate the verification code so that the user does not need to type it manually.

**Actual Behavior**

- On some Android devices (especially newer models using Samsung Internet), the OTP autofill UI does not appear even after the SMS is received.

| Device | Browser | Result |
| --- | --- | --- |
| Galaxy S21 | Chrome | ✅ Works |
| Galaxy S21 | Samsung Internet | ✅ Works |
| Galaxy S22 | Chrome | ✅ Works |
| Galaxy S22 | Samsung Internet | ✅ Works |
| Galaxy Fold7 | Chrome | ✅ Works |
| Galaxy Fold7 | Samsung Internet | ❌ Does not work |
| iPhone | Safari | ✅ Works |
| iPhone | Chrome | ✅ Works (uses Safari engine) |
- iOS Safari-based browsers work consistently, while some Samsung Galaxy + Samsung Internet combinations do not.

---

### **Root Cause Analysis**

Based on research, this does not appear to be a logic issue in our implementation, but rather a result of **differences in OTP parsing and autofill policies between operating systems and browsers**.

This feature is not fully controllable via web standards. Instead, it behaves more like a **platform-dependent feature**, where the mobile OS and browser interpret SMS content and decide whether to offer OTP autofill.

OTP autofill is not just an HTML feature. The overall flow works as follows:

- The mobile OS analyzes the SMS content to determine whether it contains an OTP
- The browser and keyboard then surface that information as an autofill suggestion

Because of this, the behavior can be affected by the following factors:

| Factor | Impact |
| --- | --- |
| Android OS version | Different SMS OTP detection logic |
| Browser engine (Chrome vs Samsung Internet) | Different autofill support and integration |
| SMS message format | OTP recognition rate varies depending on number placement and sentence structure |
| Keyboard app | Influences whether OTP suggestions are displayed |

---

### **Recommendations**

Since OTP autofill is highly device-dependent and cannot be guaranteed 100%, it is recommended to use the following complementary approaches:

1. **Improve SMS Format**
    - Place the verification code at the beginning of the message or on a separate line
    - OTP recognition rates may vary depending on **where the code appears (start of message / standalone line)** and the **surrounding text structure**
    
    ```
    482193
    This is your OOO verification code.
    ```
    
2. **Adopt the WebOTP API (Android Chrome only)**
    
    → In supported environments, the verification code can be automatically retrieved and filled in when the SMS arrives
    
3. **Enhance Paste UX**
    
    → When users copy the SMS and paste it, automatically extract and insert only the numeric code
    
4. **Include a Verification Link in SMS (Fallback)**
    
    ```
    https://example.com/auth?code=482193
    ```
    
    → Works across all devices and browsers as a reliable fallback method