# Day 52 - SNS: 발행/구독 패턴

📅 날짜: 2026년 7월 27일 (월요일)  
🎯 주제: Amazon SNS  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- SNS의 발행/구독 패턴을 이해한다
- SNS와 SQS를 팬아웃 패턴으로 통합한다
- SNS 필터 정책으로 선택적 메시지 전달을 구현한다

---

## 📖 이론 내용

### 1. Amazon SNS란?

발행자(Publisher)가 메시지를 보내면 모든 구독자(Subscriber)에게 즉시 전달하는 완전 관리형 알림 서비스입니다.

**SNS vs SQS:**
- **SNS**: Push 방식, 모든 구독자에게 동시 전달
- **SQS**: Pull 방식, 하나의 Consumer가 수신

**SNS 구독 대상:**
- 이메일/SMS
- HTTP/HTTPS 엔드포인트
- Lambda 함수
- SQS 큐
- Kinesis Data Firehose
- AWS 모바일 앱 (Mobile Push)

### 2. SNS 기본 사용

```python
import boto3

sns = boto3.client('sns')

# 주제 생성
response = sns.create_topic(Name='OrderNotifications')
topic_arn = response['TopicArn']

# 이메일 구독 추가
sns.subscribe(
    TopicArn=topic_arn,
    Protocol='email',
    Endpoint='admin@example.com'
)

# SQS 큐 구독 추가
sns.subscribe(
    TopicArn=topic_arn,
    Protocol='sqs',
    Endpoint='arn:aws:sqs:ap-northeast-2:123456789:order-queue'
)

# Lambda 구독 추가
sns.subscribe(
    TopicArn=topic_arn,
    Protocol='lambda',
    Endpoint='arn:aws:lambda:ap-northeast-2:123456789:function:process-order'
)

# 메시지 발행
sns.publish(
    TopicArn=topic_arn,
    Message='{"orderId": "O001", "status": "created"}',
    Subject='새 주문 생성',
    MessageAttributes={
        'orderType': {
            'DataType': 'String',
            'StringValue': 'premium'
        }
    }
)
```

### 3. SNS 팬아웃 패턴 (Fanout)

```
팬아웃 패턴 (SNS + SQS)
================================

[S3 이벤트] → [SNS 주제]
                  |
          +-------+-------+
          |       |       |
       [SQS1]  [SQS2]  [SQS3]
     (이미지  (데이터  (알림
      리사이즈) 분석)   발송)
          |
          v
      [Lambda]

S3 버킷은 단 하나의 이벤트 대상만 직접 설정 가능
→ SNS를 통해 여러 SQS에 동시 전달 (팬아웃)
```

### 4. SNS 필터 정책

특정 구독자에게만 메시지를 전달합니다:

```python
# 프리미엄 주문만 특정 SQS로 전달
sns.set_subscription_attributes(
    SubscriptionArn='arn:aws:sns:...',
    AttributeName='FilterPolicy',
    AttributeValue='{"orderType": ["premium"]}'
)

# 일반 주문용 SQS에는 standard 타입만
sns.set_subscription_attributes(
    SubscriptionArn='arn:aws:sns:...:standard-queue',
    AttributeName='FilterPolicy',
    AttributeValue='{"orderType": ["standard"]}'
)
```

### 5. SNS FIFO 주제

```python
# SNS FIFO 주제 생성 (SQS FIFO 큐와 사용)
response = sns.create_topic(
    Name='orders.fifo',
    Attributes={
        'FifoTopic': 'true',
        'ContentBasedDeduplication': 'true'
    }
)
```

**FIFO 주제 특성:**
- 순서 보장
- 중복 제거
- SQS FIFO 큐에만 구독 가능

---

## 아키텍처 다이어그램

```
SNS 팬아웃 아키텍처
================================

[주문 서비스]
     |
     | 주문 생성 이벤트
     v
[SNS 주제: OrderEvents]
     |
     +---[FilterPolicy: premium]---> [SQS: premium-orders]
     |                                      |
     |                                      v
     |                               [Lambda: VIP 처리]
     |
     +---[FilterPolicy: standard]--> [SQS: standard-orders]
     |                                      |
     |                                      v
     |                               [Lambda: 일반 처리]
     |
     +---> [이메일 구독] (관리자 알림)
     |
     +---> [Lambda: 재고 업데이트]
```

---

## ⭐ 핵심 포인트

1. ⭐ **SNS**: Push, 모든 구독자 동시 전달
2. ⭐ **팬아웃**: S3→SNS→여러SQS, 하나의 이벤트를 다중 처리
3. ⭐ **필터 정책**: 특정 속성 기반으로 선택적 메시지 전달
4. ⭐ **SNS+SQS**: SNS는 즉시 전달, SQS는 안정적 비동기 처리
5. ⭐ **메시지 보존**: SNS는 비보존 (전달 실패 시 소실), SQS는 보존

---

## 📝 연습 문제

**문제 1.** 하나의 이벤트를 여러 Lambda 함수가 동시에 처리해야 할 때 사용하는 패턴은?

A) SQS 단독 사용  
B) SNS 팬아웃 (SNS → 여러 SQS → 여러 Lambda)  
C) EventBridge 단독  
D) Kinesis 사용  

**정답: B** - 팬아웃 패턴은 SNS에서 여러 SQS로 메시지를 동시에 전달하여 각각 독립적으로 처리할 수 있습니다.

---

**문제 2.** SNS에서 특정 속성 값을 가진 메시지만 특정 구독자에게 전달하려면?

A) SNS 액세스 정책  
B) SNS 필터 정책  
C) SQS 큐 정책  
D) Lambda 환경 변수  

**정답: B** - SNS 필터 정책을 사용하면 메시지 속성에 따라 특정 구독자에게만 메시지를 전달합니다.

---

**문제 3.** SNS와 SQS의 가장 큰 차이점은?

A) 가격 차이  
B) SNS는 Push(즉시 전달), SQS는 Pull(Consumer가 수신)  
C) 보안 차이  
D) 리전 제한  

**정답: B** - SNS는 발행 즉시 모든 구독자에게 Push하고, SQS는 Consumer가 직접 Pull하는 방식입니다.

---

**문제 4.** S3 이벤트를 여러 Lambda에 동시에 전달해야 할 때?

A) S3 이벤트 → Lambda 직접 (여러 설정)  
B) S3 이벤트 → SNS → 여러 Lambda/SQS  
C) CloudWatch Events 사용  
D) 불가능  

**정답: B** - S3는 하나의 이벤트 대상만 직접 설정 가능하므로, SNS를 중간에 두어 팬아웃합니다.

---

**문제 5.** SNS 메시지가 Lambda 전달에 실패했을 때 기본 동작은?

A) SQS로 자동 이동  
B) 재시도 후 메시지 소실  
C) DLQ로 자동 이동  
D) 이메일 알림  

**정답: B** - SNS는 기본적으로 재시도 후 전달에 실패하면 메시지가 소실됩니다. Lambda 함수에 DLQ를 설정하거나 Destinations를 사용해야 합니다.

---

## 📌 오늘의 요약

1. SNS: 완전 관리형 발행/구독, Push 방식, 즉시 전달
2. 팬아웃: SNS → 여러 SQS, 하나의 이벤트를 다중 병렬 처리
3. 필터 정책: 메시지 속성 기반으로 선택적 전달
4. SNS FIFO: 순서 보장, 중복 제거, SQS FIFO만 구독 가능
5. SNS vs SQS: SNS는 Push/즉시/비보존, SQS는 Pull/보존
