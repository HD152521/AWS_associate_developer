# Day 48 - CloudTrail, EventBridge

📅 날짜: 2026년 7월 21일 (화요일)  
🎯 주제: AWS CloudTrail & EventBridge  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- CloudTrail로 AWS API 호출 감사 로그를 관리한다
- EventBridge로 이벤트 기반 아키텍처를 구현한다
- CloudTrail과 EventBridge를 통합한다

---

## 📖 이론 내용

### 1. AWS CloudTrail

AWS 계정의 모든 API 호출을 기록하는 감사(Audit) 서비스입니다.

**CloudTrail 이벤트 유형:**
- **Management Events**: AWS 리소스 변경 (EC2 생성, S3 버킷 정책 변경 등)
- **Data Events**: 데이터 접근 (S3 GetObject, Lambda 호출 등), 기본 비활성화
- **Insight Events**: 비정상적인 활동 감지

```bash
# CloudTrail 이벤트 조회
aws cloudtrail lookup-events \
    --lookup-attributes AttributeKey=Username,AttributeValue=admin \
    --start-time 2026-07-01T00:00:00Z \
    --end-time 2026-07-21T00:00:00Z

# 특정 서비스 이벤트 조회
aws cloudtrail lookup-events \
    --lookup-attributes AttributeKey=EventName,AttributeValue=DeleteBucket
```

**CloudTrail 보존 및 분석:**
```
CloudTrail → S3 버킷 (장기 보존)
          → CloudWatch Logs (실시간 모니터링)
          → Athena (SQL로 분석)
```

**⭐ 기본 이벤트 기록**: 90일 (이후 S3에 아카이빙 필요)

### 2. CloudTrail Insights

비정상적인 API 활동을 자동으로 감지합니다:

```json
{
  "예시": "IAM 사용자가 갑자기 100개의 S3 버킷을 30분 만에 삭제",
  "Insights 이벤트": {
    "EventName": "DeleteBucket",
    "InsightType": "ApiCallRateInsight",
    "비정상 감지": "평상시 대비 10배 이상의 호출"
  }
}
```

### 3. Amazon EventBridge

AWS 서비스, SaaS 앱, 사용자 정의 이벤트를 처리하는 서버리스 이벤트 버스입니다.

**이벤트 소스:**
- AWS 서비스 (EC2 상태 변경, S3 업로드 등)
- SaaS 파트너 (Salesforce, Zendesk 등)
- 사용자 정의 애플리케이션

**EventBridge 규칙 예시:**

```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["terminated"]
  }
}
```

```python
# 사용자 정의 이벤트 전송
import boto3

events = boto3.client('events')

events.put_events(
    Entries=[
        {
            'Source': 'myapp.orders',
            'DetailType': 'OrderCreated',
            'Detail': '{"orderId": "O001", "amount": 50000}',
            'EventBusName': 'default'
        }
    ]
)
```

### 4. EventBridge 대상

```
EventBridge 규칙 → 대상 (Target)
                    |
                    +-- Lambda 함수
                    +-- SNS 주제
                    +-- SQS 큐
                    +-- Step Functions
                    +-- ECS 작업
                    +-- API Gateway
                    +-- Kinesis Data Streams
                    +-- CodePipeline
```

### 5. CloudTrail + EventBridge 통합

```
[AWS API 호출] → [CloudTrail]
                      |
                      | Management Event
                      v
                [EventBridge]
                      |
                      | 조건 매칭
                      v
                [Lambda] → [SNS 알림]
                
예: root 계정 로그인 감지 → 즉시 알림
```

---

## 아키텍처 다이어그램

```
CloudTrail 감사 아키텍처
================================

모든 AWS API 호출
     |
     v
[CloudTrail]
     |
     +---> [S3] (90일 이후 장기 보존)
     |
     +---> [CloudWatch Logs] (실시간 모니터링)
                |
                v
          [CloudWatch 알람]
                |
                v
          [SNS 알림]

EventBridge 이벤트 버스
================================

[EC2] [S3] [RDS] [사용자 앱]
          |
          v
    [EventBridge]
          |
     조건 매칭
          |
    +-----+-----+
    |     |     |
[Lambda] [SQS] [SNS]
```

---

## ⭐ 핵심 포인트

1. ⭐ **CloudTrail**: 모든 AWS API 호출 기록, 기본 90일 보존
2. ⭐ **Data Events**: S3 Object 접근, Lambda 호출 기록, 기본 비활성화
3. ⭐ **CloudTrail Insights**: 비정상 API 활동 자동 감지
4. ⭐ **EventBridge**: 이벤트 기반 아키텍처, 18개 이상 대상
5. ⭐ **root 로그인 감지**: CloudTrail → EventBridge → SNS 알림 패턴

---

## 📝 연습 문제

**문제 1.** AWS 계정에서 누가 S3 버킷을 삭제했는지 확인하려면?

A) CloudWatch Logs  
B) CloudTrail  
C) X-Ray  
D) VPC Flow Logs  

**정답: B** - CloudTrail은 모든 AWS API 호출을 기록하므로 DeleteBucket 이벤트에서 누가 실행했는지 확인할 수 있습니다.

---

**문제 2.** CloudTrail Data Events의 특징은?

A) 기본 활성화, 추가 비용 없음  
B) 기본 비활성화, 활성화 시 추가 비용  
C) 관리형 이벤트보다 중요도 낮음  
D) S3에만 적용됨  

**정답: B** - Data Events(S3 GetObject, Lambda 호출 등)는 기본 비활성화 상태이며 활성화하면 추가 비용이 발생합니다.

---

**문제 3.** root 계정 로그인을 실시간으로 감지하여 알림을 보내려면?

A) CloudWatch Metrics 알람  
B) CloudTrail → EventBridge → SNS  
C) GuardDuty만으로 가능  
D) IAM 정책으로 방지  

**정답: B** - CloudTrail에서 root 로그인 이벤트를 EventBridge로 라우팅하고 SNS로 알림을 보낼 수 있습니다.

---

**문제 4.** EventBridge와 SNS의 차이점은?

A) 속도 차이  
B) EventBridge는 이벤트 기반 라우팅과 필터링, SNS는 단순 구독/알림  
C) 비용 차이  
D) 지역 제한  

**정답: B** - EventBridge는 세밀한 이벤트 필터링과 다양한 대상 라우팅을 지원하고, SNS는 주제 기반 단순 발행/구독 모델입니다.

---

**문제 5.** CloudTrail 이벤트의 기본 보존 기간은?

A) 7일  
B) 30일  
C) 90일  
D) 1년  

**정답: C** - CloudTrail 이벤트는 기본 90일 동안 보존됩니다. 장기 보존은 S3로 아카이빙해야 합니다.

---

## 📌 오늘의 요약

1. CloudTrail: 모든 AWS API 감사, 기본 90일, S3로 장기 보존
2. Data Events: S3/Lambda 데이터 접근 기록, 기본 비활성화
3. CloudTrail Insights: 비정상 API 활동 자동 감지
4. EventBridge: 이벤트 기반 아키텍처, AWS/SaaS/사용자 이벤트 처리
5. CloudTrail + EventBridge: root 로그인, 보안 이벤트 실시간 알림
