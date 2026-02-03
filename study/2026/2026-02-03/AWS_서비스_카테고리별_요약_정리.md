# AWS 주요 서비스 카테고리별 요약 정리

## 메시징 / 이벤트 / 데이터 스트리밍

### AMQP / Amazon MQ

- **AMQP**: 메시지 지향 미들웨어용 오픈 표준 프로토콜
- **Amazon MQ**
    - RabbitMQ, ActiveMQ 지원
    - AMQP 네이티브 지원 → 기존 앱 코드 변경 최소화 마이그레이션 가능

### Amazon Kinesis

- **Kinesis Data Streams**: 실시간 데이터 수집 및 처리에 최적화
- **Kinesis Data Firehose**: 수집 데이터를 S3 등으로 자동 적재

### Amazon EventBridge

- 이벤트 기반 아키텍처 구성
- 이벤트 규칙을 통해 여러 단계의 Lambda 등을 느슨하게 연결

---

## 네트워킹 / 글로벌 트래픽 / 연결

### Amazon Global Accelerator

- AWS 글로벌 네트워크 통해 지연시간 감소
- 사용자 트래픽을 가장 가까운 NLB로 라우팅

### AWS PrivateLink

- 인터넷을 거치지 않는 프라이빗 연결
- AWS 서비스 ↔ 온프레미스 / 외부 애플리케이션 보안 통신

### S3 인터페이스 VPC 엔드포인트

- PrivateLink 기반 S3 사설 접근
- NAT 게이트웨이 비용 절감 가능

### VPC Lattice

- 여러 VPC/계정 간 서비스 간 통신 단순화
- 서비스 네트워크 기반 마이크로서비스 연결 관리

### Route 53 다중값 응답 라우팅

- 여러 IP 중 정상 상태 리소스만 응답

---

## 파일 전송 / 스토리지 연동

### AWS Transfer Family (SFTP)

- 관리형 SFTP 서비스
- 업로드 시 자동으로 S3 저장
- 레거시 SFTP 시스템 연동에 적합

### AWS Storage Gateway (S3 File Gateway)

- 온프레미스에서 NFS/SMB 프로토콜로 S3 사용

---

## 스토리지 / 파일 시스템 / 캐시

### Amazon FSx for Lustre

- HPC용 고성능 파일 시스템
- 대규모 분석, 스크래치 스토리지에 적합

### Amazon EFS

- 다중 AZ 공유 파일 시스템
- 여러 EC2에서 동시에 마운트 가능
- 자동 확장

### ElastiCache for Redis

- 밀리초 미만 응답 속도
- DB 부하 감소용 캐시
- 스냅샷 및 복제 지원

### S3 Storage Lens

- S3 사용량 및 활동 가시화
- 보안 설정 및 객체 통계 분석

---

## 데이터 분석 / ETL / BI

### AWS Glue DataBrew

- 코드 없이 데이터 정리 및 전처리
- 데이터 프로파일링 및 레시피 재사용 가능

### Amazon Athena

- S3 데이터에 대해 SQL 쿼리 수행 (서버리스)

### Amazon QuickSight

- 관리형 BI 서비스
- 대시보드 및 데이터 시각화

---

## 워크플로 / 자동화 / 운영

### AWS Step Functions

- 서버리스 워크플로
- 상태 관리, 재시도, 오류 처리, 병렬 처리 지원

### Systems Manager Automation

- 예: DeleteKey 발생 시 자동 복구 실행 가능

---

## 컴퓨팅 / 컨테이너 / HPC

### AWS Fargate

- 서버 관리 없이 컨테이너 실행

### Amazon ECS / EKS

- ECS: AWS 자체 컨테이너 오케스트레이션
- EKS: 관리형 Kubernetes 서비스

### Elastic Fabric Adapter (EFA)

- HPC용 초저지연 네트워크
- EC2 인스턴스 간 고성능 통신 지원

---

## 데이터베이스 / 마이그레이션

### AWS DMS

- 전체 로드 + CDC(Change Data Capture) 지원
- 지속적 변경 데이터 복제 가능

### Amazon Aurora Read Replica

- 읽기 트래픽 분산
- 쓰기(클러스터 엔드포인트) / 읽기(리더 엔드포인트) 분리

### RDS Custom for Oracle

- BYOL(Bring Your Own License) 지원

---

## 보안 / 거버넌스 / 규정 준수

### AWS Control Tower (사전 예방 제어)

- 규정 위반 리소스 생성 사전 차단

### Service Control Policy (SCP)

- 조직 단위 API 동작 제한
- IAM 정책보다 상위 레벨에서 제어

### AWS Secrets Manager

- 비밀번호, API 키 등 비밀 정보 저장
- 자동 로테이션 지원

### AWS Security Hub

- 보안 상태 중앙 모니터링
- 산업 표준 프레임워크 기반 평가

---

## 인증 / 엣지 / CDN

### Amazon Cognito

- 서버리스 인증 서비스
- 사용자 풀 및 권한 풀 기반 인증 관리

### CloudFront

- 글로벌 CDN 서비스

### Lambda@Edge

- CloudFront 엣지 로케이션에서 코드 실행
- 접근 제어 및 요청/응답 처리 로직 구현

---

## SaaS 연동 / 데이터 통합

### Amazon AppFlow

- SaaS ↔ AWS 간 데이터 자동 전송
- 변환, 필터링, 매핑 기능 제공
- 예: Salesforce → Redshift

---

## 비용 최적화 / 리소스 분석

### AWS Compute Optimizer

- EC2, EBS, Lambda 사용 패턴 분석
- 비용 절감 권장 사항 제공

---

## 하이브리드 / 온프레미스 확장

### AWS Outposts

- 온프레미스에 AWS 환경 구축
- 로컬 환경에서 EKS 등 AWS 관리 서비스 사용 가능

---

## 컨테이너 보안

### Amazon ECR

- 컨테이너 이미지 저장소
- 기본 CVE 취약점 스캔 제공