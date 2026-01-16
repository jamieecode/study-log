# Security & Networking

## 1️⃣ IAM (Identity and Access Management)

### IAM 구성요소

- User
- Group
- Role
- Policy (JSON)

### Role 특징

- 임시 자격 증명(EC2나 Lambda같은 서비스가 다른 AWS 자원 접근할 때)
- EC2 / Lambda / ECS에서 사용

### 📝 출제

- EC2 권한 부여 → **IAM Role**
- 하드코딩 ❌

---

## 2️⃣ KMS & 암호화

### KMS

- 암호화 키 관리
- S3, EBS, RDS와 연동

### 암호화 방식

- At-rest
- In-transit

---

## 3️⃣ VPC 네트워크 심화

### VPC Peering

- VPC(Virtual Private Cloud) 간 연결
- Transitive ❌

### VPC Endpoint

- AWS 서비스 사설 접근
- S3 / DynamoDB 빈출

---

## 4️⃣ 하이브리드 연결

| 방식 | 용도 |
| --- | --- |
| VPN | 빠른 구축 |
| Direct Connect | 안정적, 고속 |

---

## 5️⃣ 모니터링 & 보안 서비스

- CloudWatch: 로그/지표, 알람설정(CPU 80%시 알림)
- CloudTrail: API 기록(누가 언제 어떤 api를 호출했는지, 보안감사용)
- GuardDuty: 위협 탐지
- WAF: 웹 공격 방어
- EventBridge: 시스템 상태 변화를 감지해 특정 작업실행(ex. EC2 중지시 Lambda 실행)