# Day 55 - Week 11 복습 + 연습문제 (메시징)

📅 날짜: 2026년 7월 30일 (목요일)  
🎯 주제: 메시징 서비스 종합 복습  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- SQS, SNS, Kinesis, Step Functions의 핵심을 종합 정리한다
- 시험에서 자주 나오는 메시징 시나리오 문제를 풀어본다

---

## 📖 Week 11 핵심 정리

### 메시징 서비스 선택 가이드
```
순서 보장 + 정확히 1회: SQS FIFO
높은 처리량 + 순서 불필요: SQS 표준
하나의 메시지를 여러 Consumer: SNS (팬아웃)
실시간 데이터 스트리밍: Kinesis Data Streams
ETL → S3/Redshift: Kinesis Firehose
복잡한 워크플로우: Step Functions
GraphQL + 실시간: AppSync
```

### 핵심 숫자 암기
```
SQS 메시지 최대 크기: 256KB
SQS 최대 보존 기간: 14일
SQS FIFO 처리량: 초당 300개 (배치 3000개)
SQS VisibilityTimeout 기본: 30초
Kinesis 샤드 쓰기: 1MB/s 또는 1000 RPS
Kinesis 샤드 읽기: 2MB/s
Kinesis 기본 보존: 24시간 (최대 365일)
Firehose 최소 지연: 1분
```

---

## 아키텍처 다이어그램

```
메시징 서비스 선택 아키텍처
================================

[이벤트 소스]
     |
     +-- 순서/중복 중요? → SQS FIFO
     |
     +-- 높은 처리량? → SQS 표준
     |
     +-- 여러 Consumer 동시? → SNS → 여러 SQS
     |
     +-- 실시간 스트리밍? → Kinesis Data Streams
     |
     +-- ETL 파이프라인? → Kinesis Firehose → S3
     |
     +-- 복잡한 워크플로우? → Step Functions
     |
     +-- GraphQL API? → AppSync
```

---

## 📝 Week 11 종합 연습문제

**문제 1.** 결제 트랜잭션을 정확히 1회, 순서대로 처리해야 할 때?

A) SQS 표준 큐  
B) SQS FIFO 큐  
C) SNS  
D) Kinesis  

**정답: B** - FIFO 큐는 순서 보장과 정확히 1회 처리(중복 제거)를 지원합니다.

---

**문제 2.** 수백만 명의 사용자 클릭 이벤트를 실시간으로 분석하고 S3에 저장하려면?

A) SQS → Lambda → S3  
B) Kinesis Data Streams → 분석 → Firehose → S3  
C) SNS → SQS → S3  
D) DynamoDB Streams → S3  

**정답: B** - Kinesis Data Streams는 대용량 실시간 스트리밍에 최적화되어 있고, Firehose는 S3로 쉽게 전달합니다.

---

**문제 3.** S3 업로드 이벤트 하나에 Lambda 3개가 동시에 반응해야 할 때?

A) S3 이벤트를 각 Lambda에 직접 설정  
B) S3 → SNS → 3개의 Lambda 구독  
C) S3 → SQS → Lambda  
D) S3 → EventBridge → 3개의 Lambda  

**정답: B 또는 D** - SNS 팬아웃 또는 EventBridge를 사용하여 하나의 이벤트를 여러 Lambda에 동시에 전달합니다. 시험에서는 B가 정석적인 팬아웃 패턴입니다.

---

**문제 4.** SQS에서 처리에 3번 실패한 메시지를 격리하려면?

A) SQS 필터 정책 설정  
B) DLQ 설정 (maxReceiveCount=3)  
C) VisibilityTimeout 증가  
D) 롱 폴링 비활성화  

**정답: B** - DLQ의 maxReceiveCount를 3으로 설정하면 3번 실패한 메시지가 DLQ로 이동됩니다.

---

**문제 5.** 여러 Lambda를 순서대로 호출하고 중간에 실패하면 이전 단계로 롤백해야 할 때?

A) Lambda 체인 (Lambda에서 Lambda 직접 호출)  
B) Step Functions  
C) SQS 워크플로우  
D) EventBridge 규칙 체인  

**정답: B** - Step Functions는 워크플로우 상태를 관리하고 오류 처리, 재시도, 보상 트랜잭션을 지원합니다.

---

**문제 6.** Kinesis Data Streams와 SQS의 가장 큰 차이점은?

A) 처리 속도 차이  
B) Kinesis는 여러 Consumer가 동시에 데이터를 읽을 수 있고 데이터가 보존됨  
C) 비용 차이  
D) 리전 제한  

**정답: B** - Kinesis는 데이터가 보존되어 여러 Consumer가 독립적으로 읽을 수 있지만, SQS는 Consumer가 읽으면 메시지가 큐에서 사라집니다.

---

**문제 7.** 모바일 앱에서 GraphQL을 사용하고 실시간 업데이트가 필요할 때?

A) API Gateway REST API  
B) API Gateway WebSocket  
C) AWS AppSync  
D) Kinesis Firehose  

**정답: C** - AppSync는 GraphQL API와 실시간 구독(WebSocket)을 완전 관리형으로 제공합니다.

---

**문제 8.** SNS 메시지에서 프리미엄 고객 관련 메시지만 특정 SQS로 전달하려면?

A) SQS 메시지 필터  
B) SNS 필터 정책  
C) Lambda 중간 처리  
D) IAM 정책  

**정답: B** - SNS 구독에 필터 정책을 설정하면 메시지 속성에 따라 특정 구독자에게만 전달됩니다.

---

## 📌 오늘의 요약

1. SQS: 표준(무제한/비순서) vs FIFO(300/s, 순서/중복제거)
2. SNS: Push, 팬아웃, 필터 정책으로 선택적 전달
3. Kinesis: 대용량 스트리밍, 여러 Consumer, 샤드 단위 처리
4. Firehose: 서버리스 ETL, 1분 지연, S3/Redshift/OpenSearch
5. Step Functions: 워크플로우 오케스트레이션, 재시도, 병렬 실행
