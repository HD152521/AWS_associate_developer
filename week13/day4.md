# Day 64 - 최종 복습 4: 메시징, 컨테이너, 아키텍처 패턴

📅 날짜: 2026년 8월 12일 (수요일)  
🎯 주제: 메시징/컨테이너 최종 복습 + 아키텍처 패턴  
⏱️ 학습 시간: 약 120분

---

## 🎯 학습 목표

- 메시징, 컨테이너, 아키텍처 패턴의 핵심을 최종 정리한다
- 시나리오 기반 아키텍처 선택 문제를 풀어본다

---

## 📖 최종 핵심 정리

### 메시징 서비스 선택 가이드
```
SQS 표준: 높은 처리량, 순서/중복 미보장
SQS FIFO: 순서 보장, 중복 제거, 초당 300개
SNS: Push, 여러 구독자 동시, 팬아웃 패턴
Kinesis Streams: 실시간 스트리밍, 여러 Consumer, 24시간 보존
Kinesis Firehose: ETL, S3/Redshift/OpenSearch, 1분 지연
Step Functions: 워크플로우 오케스트레이션, 재시도/병렬
AppSync: GraphQL, 실시간 구독, 오프라인 지원
```

### 핵심 숫자
```
SQS: 256KB, 14일, FIFO 300/s, VisibilityTimeout 30초
Kinesis: 샤드 1MB/s 쓰기, 2MB/s 읽기, 24시간 기본 보존
SNS: 여러 구독 대상, Push 방식
```

### 컨테이너/IaC 핵심 암기
```
ECS Fargate: 서버리스, awsvpc 모드
executionRole: ECR pull, CloudWatch Logs
taskRole: 컨테이너에서 AWS 서비스 접근
ECR: 취약점 스캔, 수명 주기 정책
CloudFormation: Change Set(검토), !ImportValue(교차 스택)
SAM: Transform 필수, sam local(Docker 필요)
CDK: 프로그래밍 언어 → CloudFormation 변환
```

---

## 아키텍처 패턴 총정리

```
패턴 1: 서버리스 REST API
================================
[클라이언트] → [API Gateway] → [Lambda] → [DynamoDB]
                                        ↘ [ElastiCache]
인증: Cognito Authorizer

패턴 2: 이벤트 기반 비동기 처리
================================
[서비스A] → [SQS] → [Lambda B]
                  ↘ (실패 시) [DLQ]

패턴 3: 팬아웃
================================
[S3 업로드] → [SNS] → [SQS1] → [Lambda: 리사이즈]
                    → [SQS2] → [Lambda: 분석]
                    → [Lambda: 알림]

패턴 4: 스트리밍 파이프라인
================================
[데이터 소스] → [Kinesis Streams] → [Lambda: 실시간]
                                  → [Firehose] → [S3]

패턴 5: 완전 CI/CD
================================
[Git Push] → [CodePipeline]
  → [CodeBuild: 테스트/빌드]
  → [Manual Approval]
  → [CodeDeploy: 배포]
     (Canary for Lambda, Blue/Green for EC2)
```

---

## 📝 최종 모의고사 - Part 4

**문제 1.** 수백만 사용자의 클릭 이벤트를 실시간으로 수집하고 여러 시스템에서 분석해야 할 때?

A) SQS  
B) SNS  
C) Kinesis Data Streams  
D) EventBridge  

**정답: C** - Kinesis Data Streams는 대용량 실시간 스트리밍을 지원하고 여러 Consumer가 동시에 동일 스트림을 읽을 수 있습니다.

---

**문제 2.** 마이크로서비스 A가 B의 응답을 기다리지 않고 비동기로 작업을 처리하려면?

A) Lambda A에서 Lambda B를 동기 호출  
B) Lambda A → SQS → Lambda B  
C) API Gateway 직접 연결  
D) DynamoDB를 통한 데이터 공유  

**정답: B** - SQS를 통한 비동기 통신은 두 서비스를 느슨하게 결합하고 Lambda A가 응답을 기다리지 않아도 됩니다.

---

**문제 3.** ECS 태스크가 실행 중 Secrets Manager에서 비밀번호를 가져오려면?

A) executionRoleArn에 권한 추가  
B) taskRoleArn에 권한 추가  
C) 환경 변수에 하드코딩  
D) CloudFormation 파라미터로 주입  

**정답: B** - taskRole은 컨테이너 내 애플리케이션 코드가 AWS 서비스에 접근하는 데 사용합니다.

---

**문제 4.** 결제 처리 → 이메일 발송 → 재고 업데이트 순서로 실행하되 각 단계 실패 시 재시도해야 할 때?

A) SQS 체인  
B) Lambda 체인 (Lambda에서 Lambda 직접 호출)  
C) Step Functions  
D) Kinesis  

**정답: C** - Step Functions는 순차 실행, 오류 처리, 자동 재시도를 시각적으로 관리할 수 있습니다.

---

**문제 5.** CloudFormation에서 VPC를 별도 스택으로 관리하고 다른 스택에서 참조하려면?

A) VPC ID를 수동으로 파라미터로 전달  
B) Outputs에서 Export 후 !ImportValue로 참조  
C) 동일 스택에 모두 포함  
D) 크로스 스택 참조 불가  

**정답: B** - Outputs 섹션에서 Export 이름을 지정하고 다른 스택에서 !ImportValue로 참조합니다.

---

**문제 6.** S3 업로드 이벤트로 이미지 리사이즈, 메타데이터 분석, 관리자 알림을 동시에 처리하려면?

A) S3 이벤트를 각 Lambda에 직접 설정  
B) S3 → SNS → 여러 SQS → 각 Lambda  
C) S3 → Kinesis → Lambda  
D) S3 → EventBridge → Lambda 3개  

**정답: B 또는 D** - 팬아웃 패턴입니다. SNS를 중간에 두고 각 SQS로 분기하거나(B), EventBridge로 여러 Lambda를 동시에 트리거합니다(D).

---

**문제 7.** Beanstalk에서 중단 없이 새 버전을 배포하는 가장 안전한 전략은?

A) All at once  
B) Rolling  
C) Immutable  
D) Blue/Green  

**정답: C** - Immutable 배포는 새 ASG에 새 버전을 배포하고 검증 후 교체하므로 가장 안전합니다.

---

**문제 8.** CDK 앱을 배포할 때 실제로 생성되는 것은?

A) Terraform 플랜  
B) CloudFormation 스택  
C) ECS 태스크  
D) Lambda 레이어  

**정답: B** - CDK는 cdk deploy 시 코드를 CloudFormation 템플릿으로 변환하고 CloudFormation 스택을 생성합니다.

---

## 📌 오늘의 요약

1. SQS(큐/비동기) vs SNS(팬아웃/Push) vs Kinesis(스트리밍/다중Consumer)
2. Step Functions: 복잡한 워크플로우, 재시도, 병렬, 오류 처리
3. ECS: taskRole(앱 권한), executionRole(인프라 권한)
4. CDK: 프로그래밍 언어 → CloudFormation, cdk synth/deploy
5. 핵심 패턴: 팬아웃(SNS→SQS), 비동기(SQS), 워크플로우(Step Functions)
