# 인증서(Certificate)

# 1. 인증서(Certificate)

인증서는 **인터넷에서 특정 서버나 서비스의 신원을 증명하기 위한 디지털 문서**

TLS/HTTPS 통신에서 인증서의 역할

- 서버의 **신원 인증**
- **공개키의 신뢰성 보장**
- **암호화 통신을 위한 기반 제공**

“이 공개키는 실제로 이 도메인의 서버가 맞다”는 것을 보증하는 문서

---

# 2. HTTPS와 TLS

- HTTPS는 HTTP 위에 TLS 보안 계층이 추가된 프로토콜
    - HTTPS = HTTP + TLS
- TLS의 보안 특성
    - Confidentiality: 통신 내용을 암호화
    - Integrity: 데이터 변조 방지
    - Authentication: 서버 신원 확인

---

# 3. TLS에서 사용하는 암호 방식

**비대칭키 + 대칭키** 함께 사용

## 1) 비대칭키 암호화 (Asymmetric Encryption)

**2개의 키**

- Public Key (공개키)
- Private Key (개인키)

**특징**

- 공개키로 암호화 → 개인키로 복호화
- 개인키로 서명 → 공개키로 검증

**사용 목적**

- 인증
- 키 교환

**대표 알고리즘**

- RSA
- ECDSA

---

## 2) 대칭키 암호화 (Symmetric Encryption)

**하나의 동일한 키로 암호화와 복호화 수행**

**특징**

- 속도가 매우 빠름
- 대용량 데이터 암호화에 적합

**대표 알고리즘**

- AES
- ChaCha20

---

# 4. TLS Handshake 과정

```
Client Hello
   ↓
Server Hello
   ↓
Certificate 전달
   ↓
Certificate 검증
   ↓
Key Exchange
   ↓
Session Key 생성
   ↓
Secure Communication 시작
```

## 1) Client Hello

클라이언트가 서버에게 전달하는 정보

- 지원 TLS 버전
- 지원 암호화 알고리즘
- 랜덤 값

---

## 2) Server Hello

서버가 응답하는 정보

- 선택한 TLS 버전
- 선택한 암호화 알고리즘
- 서버 랜덤 값

**+ 서버 인증서**를 전달한다.

---

## 3) Certificate 검증

브라우저는 인증서를 검증한다.

검증 항목

- CA 서명 확인
- 인증서 체인 검증
- 도메인 일치 여부
- 유효 기간 확인
- 폐기 여부 확인

---

## 4) Key Exchange

클라이언트와 서버가 **세션 키(Session Key) 생성.** 이 키는 이후 통신에서 사용됨

---

## 5) Secure Communication

이후 모든 데이터는 **대칭키로 암호화되어 전송됨**

---

# 5. PKI (Public Key Infrastructure)

인터넷에서는 공개키만으로는 신뢰를 보장할 수 없어 **PKI 구조 필요**

### **PKI 구성요소**

```
Root CA
   ↓
Intermediate CA
   ↓
Server Certificate
```

### Root CA

최상위 인증 기관

예) DigiCert, GlobalSign, ISRG Root (Let's Encrypt)

→ 브라우저와 OS에는 Root CA 목록이 기본적으로 포함되어 있음

### Intermediate CA

Root CA가 직접 인증서를 발급하지 않고 중간 CA를 통해 발급함

**이유**

- 보안 강화
- Root Key 보호

### Server Certificate

실제 서버가 사용하는 인증서

---

# 6. 인증서 체인 (Certificate Chain)

인증서는 다음 구조로 신뢰가 검증된다.

```
Server Certificate
   ↓
Intermediate CA
   ↓
Root CA
```

브라우저는 이 체인을 통해 인증서를 검증한다.

---

# 7. 인증서 주요 필드

인증서에는 다음 정보가 포함된다.

| 필드 | 설명 |
| --- | --- |
| Subject | 인증 대상 |
| Issuer | 인증서 발급자 |
| Public Key | 공개키 |
| Signature | CA 서명 |
| Validity | 유효 기간 |
| SAN | 인증 가능한 도메인 |

---

# 8. CN vs SAN

과거에는 인증서의 CN(Common Name)에 도메인을 저장했다.

```
CN = example.com
```

하지만 현재는 **SAN 필드가 표준이다.**

```
SAN:
- example.com
- www.example.com
- api.example.com
```

현대 브라우저는 **SAN을 기준으로 도메인을 검증한다.**

---

# 9. 인증서 폐기 (Revocation)

인증서는 만료되기 전에 폐기될 수 있다.

예

- 개인키 유출
- 잘못 발급된 인증서

폐기 여부는 다음 방식으로 확인한다.

## CRL

Certificate Revocation List

- 폐기된 인증서 목록을 제공

단점

- 목록 크기가 커질 수 있음

---

## OCSP

Online Certificate Status Protocol

- 인증서 상태를 실시간 조회

요즘은 **OCSP Stapling** 방식이 많이 사용된다.

---

# 10. 인증서 유형

인증서는 검증 수준에 따라 나뉜다.

| 유형 | 설명 |
| --- | --- |
| DV | 도메인 소유만 검증 |
| OV | 조직 검증 |
| EV | 기업 실사 후 발급 |

---

# 11. 인증서 자동화

최근에는 인증서 발급과 갱신을 자동화하는 것이 일반적이다.

대표 서비스

- Let's Encrypt
- AWS ACM
- Cloudflare SSL

자동화 방식

```
ACME protocol
```

예

```
certbot
```