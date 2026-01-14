# Infra_Compute

## 1️⃣ AWS 글로벌 인프라

### Region

- AWS 서비스가 운영되는 지리적 영역
- 요금, 서비스 지원 여부, 법적 규제 차이 존재

### Availability Zone (AZ)

- 하나의 Region 내 물리적으로 분리된 데이터센터
- 고가용성(High Availability) 설계의 핵심

### Edge Location

- CloudFront, Route 53에서 사용
- 사용자와 가장 가까운 위치에서 콘텐츠 제공

### 출제

- 고가용성 → **Multi-AZ**
- 지연 시간 최소화 → **가까운 Region + CloudFront**
- 재해 복구 → **Multi-Region**

---

## 2️⃣ EC2 (Elastic Compute Cloud)

### EC2 기본 개념

- 가상 서버 서비스
- AMI + Instance Type + Storage + Network로 구성

### 인스턴스 타입

| 타입 | 용도 |
| --- | --- |
| General Purpose (t, m) | 범용 |
| Compute Optimized (c) | CPU 집약 |
| Memory Optimized (r, x) | 메모리 집약 |
| Storage Optimized (i, d) | 고속 I/O |

---

### EC2 구매 옵션 (★ 빈출)

| 옵션 | 특징 |
| --- | --- |
| On-Demand | 유연, 비용 높음 |
| Reserved | 장기 사용, 비용 절감 |
| Spot | 최대 90% 할인, 중단 가능 |
| Dedicated | 물리 서버 단독 사용 |

➡ **비용 절감 문제 = Reserved / Spot**

---

## 3️⃣ EC2 스토리지

### EBS (Elastic Block Store)

- EC2에 연결되는 블록 스토리지
- AZ 단위
- Snapshot은 S3에 저장

**EBS 타입**

- gp3/gp2: 범용
- io1/io2: 고성능 IOPS
- st1/sc1: 대용량, 저비용

### Instance Store

- EC2에 직접 연결된 임시 스토리지
- 인스턴스 종료 시 데이터 삭제

### 출제

- 영구 데이터 → **EBS**
- 초고속 임시 데이터 → **Instance Store**

---

## 4️⃣ ELB & Auto Scaling

### ELB (Elastic Load Balancer)

- 트래픽 분산
- 단일 장애 지점 제거

**종류**

- ALB: HTTP/HTTPS (웹 서비스)
- NLB: TCP, 초고성능
- CLB: 레거시

### Auto Scaling

- 트래픽에 따라 EC2 자동 확장/축소
- 비용 최적화 + 가용성 확보

### 출제

- 확장성 → **Auto Scaling**
- 고가용성 → **ELB + Multi-AZ**

---

## 5️⃣ VPC (Virtual Private Cloud)

### VPC 기본

- AWS 내 논리적으로 격리된 네트워크

### Subnet

| 유형 | 특징 |
| --- | --- |
| Public Subnet | Internet Gateway 연결 |
| Private Subnet | 외부 직접 접근 불가 |

### 인터넷 통신

- Internet Gateway: 퍼블릭 통신
- NAT Gateway: 프라이빗 → 외부 통신

### Route Table

- 서브넷의 트래픽 경로 제어

---

## 6️⃣ 보안: Security Group vs NACL

### Security Group

- 인스턴스 단위
- 허용 규칙만 가능
- Stateful

### NACL

- 서브넷 단위
- 허용 + 거부 가능
- Stateless

### 출제

- 대부분의 접근 제어 → **Security Group**
- 서브넷 전체 제어 → **NACL**