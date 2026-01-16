# Storage & Database

## 1️⃣ Amazon S3 (Simple Storage Service)

### S3 기본 개념

- 객체 스토리지(Simple Storage Service)
- 무제한 확장
- 정적 웹사이트 호스팅 가능
- 99.999999999% 내구성 (11 9’s)

### S3 주요 특징

- 객체 = Key + Value + Metadata
- 파일 최대 크기: 5TB
- 폴더 개념 ❌ (Prefix)

---

### S3 Storage Class

| 클래스 | 용도 |
| --- | --- |
| Standard | 자주 접근하는 데이터 |
| IA | 가끔 접근 |
| One Zone-IA | 단일 AZ |
| Glacier | 장기 보관(저렴하지만 꺼낼 때 오래 걸림) |
| Glacier Deep Archive | 초저비용 |

### 📝 출제

- 비용 절감 → **Lifecycle Policy**
- 백업/아카이빙 → **Glacier**

---

## 2️⃣ EBS vs EFS vs FSx

### EFS (Elastic File System)

- 여러 EC2에서 동시 접근
- Linux 기반
- 자동 확장

### FSx

- Windows 또는 고성능 파일 시스템
- FSx for Windows / FSx for Lustre

| 비교 | EBS | EFS |
| --- | --- | --- |
| 연결 | 단일 EC2 | 다중 EC2 |
| AZ | 단일 | Multi-AZ |
| 사용 | OS, DB | 공유 파일 |

---

## 3️⃣ 데이터베이스 개요

### RDS (관계형 DB)

- MySQL, PostgreSQL, Oracle 등
- 자동 백업
- Multi-AZ(다른 데이터 센터에 복제본, 장애 대비용) 지원

### Aurora

- AWS 자체 DB
- RDS 대비 빠름
- 고가용성 기본 제공(3개 가용 영역에 6개 복제본 생성)

---

### DynamoDB (NoSQL)

- Key-Value
- 서버리스
- 초고속, 자동 확장

### 📝 출제

- 트랜잭션 / JOIN → **RDS**
- 대규모 트래픽 / 낮은 지연 → **DynamoDB**

---

## 4️⃣ 백업 & 복구

- RDS 자동 백업
- EBS Snapshot → S3
- DynamoDB Point-in-Time Recovery