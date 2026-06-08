# OWASP Top 10 (2025)

OWASP(Open Worldwide Application Security Project)에서 발표하는 웹 애플리케이션에서 가장 위험하고 빈번하게 발생하는 보안 취약점 정리 가이드

## A01: Broken Access Control(접근 제어 취약점)

- 사용자가 접근할 수 없는 데이터나 기능에 접근하는 취약점
- url만 변경해서 다른 사용자 정보 조회
    
    ```jsx
    /users/123
    /users/1234
    ```
    
- 대응방법
    - 서버: 권한 검증 수행
    - 최소 권한 원칙 적용
    - 관리자 기능 별도 검증

---

## A02: Security Misconfiguration(보안 구성 오류)

- 기본 계정 미변경, 불필요한 서비스 활성화, 잘못된 설정으로 발생하는 보안문제
- 현대의 서비스는 코드 보다 설정에 의존하는 부분이 많고, 클라우드와 컨테이너 환경으로 전환되면서 상승
- 대응방법
    - 기본 계정 제거
    - 디버그 기능 비활성화
    - 보안 헤더 설정
    - 환경변수 관리

---

## A03: Software Supply Chain Failures(소프트웨어 공급망 실패)

- 신규 핵심 카테고리. 기본 Vulnerable and Outdated Components 확장한 개념.
- 단순히 라이브러리 버전 문제가 아니라 소프트웨어를 만드는 전체 과정이 공격 대상이 됨
- 대응 방법
    - 의존성 정기 점검
    - npm audit 활용
    - 신뢰 가능한 패키지만 사용
    - SBOM 관리

---

## A04: Cryptographic Failures(암호화 실패)

- 암호화 사용 오류
- 비밀번호 평문 저장, 민감 정보 HTTPS 미사용
- 대응방법
    - bcrypt
    - Argon2
    - TLS 사용
    - 개인정보 암호화

---

## A05: Injection(인젝션)

- 사용자 입력이 시스템 명령으로 실행되는 취약점
- SQL Injection, 사용자 입력, XSS…
- 대응방법
    - Parameterized Query
    - ORM 사용
    - 입력값 검증

---

## A06: Insecure Design(안전하지 않은 설계)

- 소스코드를 안전하게 짜더라도 기획이나 아키텍처 설계 자체가 안전하지 않은 경우
- 로그인 시도 횟수 제한 없음, 비번 재설정 검증 없음, 중요 API Rate Limit 없음
- 대응방법
    - Threat Modeling
    - Secure Design Review
    - Abuse Case 검토

---

## A07: Identification and Authentication Failures(식별 및 인증 실패)

- 인증 관련 취약점
- 약한 비밀번호 정책, 다중 인증(MFA) 미적용, 세션 탈취
- 대응방법
    - MFA 적용
    - 강력한 비번 정책
    - 세션 만료 관리
    - JWT 저장 위치: localStorage vs. httpOnly Cookie

---

## A08: Software or Data Integrity Failures(소프트웨어 및 데이터 무결성 실패)

- 플러그인, 라이브러리, 업데이트 파일이나 코드와 데이터의 무결성을 검증하지 않아 악성 코드나 변조된 데이터가 유입
- 신뢰할 수 없는 스크립트 실행
- 대응방법
    - Subresource Integrity(SRI)
    - 코드 서명
    - 배포 검증

---

## A09: Security Logging & Alerting Failures(보안 로그 및 모니터링 실패)

- 로그는 있는데 알림이 없거나, 로그 자체가 부족한 경우
- 대응방법
    - 이상 행동 탐지
    - 실시간 알림
    - 감사 로그 구축

---

## A10: Mishandling of Exceptional Conditions(예외/오류 처리 미흡)

- 예외 상황 처리 실패로 인해 발생하는 취약점
- Null 처리 실패, Fail Open, 예외 누락, 에러 메시지 정보 노출
- 대응방법
    - Fail Secure 원칙
    - 예외 처리 표준화
    - 민감정보 제거