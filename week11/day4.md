# Day 54 - Step Functions, AppSync

📅 날짜: 2026년 7월 29일 (수요일)  
🎯 주제: AWS Step Functions & AppSync  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- Step Functions로 복잡한 워크플로우를 오케스트레이션한다
- AppSync로 GraphQL API를 구현한다
- 서버리스 아키텍처에서의 적용 패턴을 이해한다

---

## 📖 이론 내용

### 1. AWS Step Functions란?

Lambda 함수와 AWS 서비스를 시각적으로 오케스트레이션하는 서버리스 워크플로우 서비스입니다.

**해결하는 문제:**
- Lambda 함수를 체인으로 연결할 때 코드 복잡도 감소
- 병렬 처리, 조건부 처리, 재시도 로직
- 워크플로우 상태 시각화

### 2. Step Functions 상태 유형

```json
{
  "Comment": "주문 처리 워크플로우",
  "StartAt": "ValidateOrder",
  "States": {
    "ValidateOrder": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:ValidateOrder",
      "Next": "CheckInventory",
      "Catch": [
        {
          "ErrorEquals": ["ValidationError"],
          "Next": "OrderFailed"
        }
      ]
    },
    "CheckInventory": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:CheckInventory",
      "Next": "ProcessPayment"
    },
    "ProcessPaymentAndNotify": {
      "Type": "Parallel",
      "Branches": [
        {
          "StartAt": "ProcessPayment",
          "States": {
            "ProcessPayment": {
              "Type": "Task",
              "Resource": "arn:aws:lambda:...:ProcessPayment",
              "End": true
            }
          }
        },
        {
          "StartAt": "SendNotification",
          "States": {
            "SendNotification": {
              "Type": "Task",
              "Resource": "arn:aws:lambda:...:SendNotification",
              "End": true
            }
          }
        }
      ],
      "Next": "OrderSuccess"
    },
    "OrderSuccess": {
      "Type": "Succeed"
    },
    "OrderFailed": {
      "Type": "Fail",
      "Error": "OrderError",
      "Cause": "주문 처리 실패"
    }
  }
}
```

**상태 유형:**
- `Task`: Lambda/서비스 호출
- `Choice`: 조건 분기
- `Parallel`: 병렬 실행
- `Wait`: 대기
- `Succeed/Fail`: 종료

### 3. Step Functions 재시도 및 오류 처리

```json
{
  "ProcessPayment": {
    "Type": "Task",
    "Resource": "arn:aws:lambda:...",
    "Retry": [
      {
        "ErrorEquals": ["Lambda.ServiceException"],
        "IntervalSeconds": 2,
        "MaxAttempts": 3,
        "BackoffRate": 2.0
      }
    ],
    "Catch": [
      {
        "ErrorEquals": ["States.ALL"],
        "Next": "HandleError"
      }
    ]
  }
}
```

### 4. AWS AppSync

GraphQL API를 위한 완전 관리형 서비스입니다.

```graphql
# GraphQL 스키마 정의
type Order {
  orderId: ID!
  userId: String!
  amount: Float!
  status: String!
  createdAt: String!
}

type Query {
  getOrder(orderId: ID!): Order
  listOrders(userId: String!): [Order]
}

type Mutation {
  createOrder(userId: String!, amount: Float!): Order
  updateOrderStatus(orderId: ID!, status: String!): Order
}

type Subscription {
  onOrderStatusUpdate(orderId: ID!): Order
  @aws_subscribe(mutations: ["updateOrderStatus"])
}
```

**AppSync 데이터 소스:**
- DynamoDB
- Lambda
- RDS (RDS Proxy)
- HTTP 엔드포인트
- Elasticsearch/OpenSearch

### 5. AppSync 실시간 구독

```javascript
// 클라이언트에서 실시간 구독
import { API, graphqlOperation } from 'aws-amplify';

const subscription = API.graphql(
    graphqlOperation(`
        subscription OnOrderUpdate($orderId: ID!) {
            onOrderStatusUpdate(orderId: $orderId) {
                orderId
                status
                updatedAt
            }
        }
    `, { orderId: 'O001' })
).subscribe({
    next: (event) => {
        console.log('주문 상태 업데이트:', event.value.data.onOrderStatusUpdate);
    }
});
```

---

## 아키텍처 다이어그램

```
Step Functions 주문 워크플로우
================================

[주문 생성]
     |
     v
[ValidateOrder] → 실패 → [OrderFailed]
     |
     v
[CheckInventory] → 재고 없음 → [RefundOrder]
     |
     v
[Parallel]
  +-- [ProcessPayment]
  +-- [SendConfirmEmail]
  +-- [UpdateInventory]
     |
     v (모두 성공 시)
[OrderSuccess]

AppSync GraphQL 아키텍처
================================

[모바일/웹 클라이언트]
     |
     | GraphQL Query/Mutation
     v
[AWS AppSync]
     |
     +-- Query → [DynamoDB] (데이터 조회)
     |
     +-- Mutation → [Lambda] (비즈니스 로직)
     |
     +-- Subscription → [실시간 업데이트]
                        (WebSocket)
```

---

## ⭐ 핵심 포인트

1. ⭐ **Step Functions**: 워크플로우 오케스트레이션, 시각적 상태 기계
2. ⭐ **Parallel 상태**: 동시 실행, 모두 완료 후 다음 단계
3. ⭐ **Retry/Catch**: 자동 재시도, 오류별 처리 분기
4. ⭐ **AppSync**: GraphQL, 실시간 구독(WebSocket), 오프라인 지원
5. ⭐ **AppSync 데이터 소스**: DynamoDB, Lambda, RDS, HTTP

---

## 📝 연습 문제

**문제 1.** 여러 Lambda 함수를 순차적으로 실행하고 오류를 처리하는 가장 좋은 방법은?

A) Lambda 함수에서 다른 Lambda를 직접 호출  
B) Step Functions 워크플로우  
C) SQS 체인  
D) EventBridge 규칙  

**정답: B** - Step Functions는 Lambda 함수를 오케스트레이션하고 오류 처리, 재시도, 분기 등을 시각적으로 관리합니다.

---

**문제 2.** Step Functions에서 여러 작업을 동시에 실행하는 상태 유형은?

A) Choice  
B) Task  
C) Parallel  
D) Map  

**정답: C** - Parallel 상태는 여러 브랜치를 동시에 실행하고 모든 브랜치가 완료되면 다음 단계로 진행합니다.

---

**문제 3.** AppSync의 장점이 아닌 것은?

A) GraphQL API 지원  
B) 실시간 구독 (WebSocket)  
C) 관계형 쿼리만 지원  
D) 오프라인 데이터 동기화  

**정답: C** - AppSync는 GraphQL을 통해 DynamoDB, Lambda, RDS 등 다양한 데이터 소스를 지원합니다.

---

**문제 4.** Step Functions에서 API 비율 제한(ThrottlingException)이 발생할 때 자동 재시도하려면?

A) Lambda 함수 내부에서 재시도 로직 작성  
B) Retry 설정에 BackoffRate와 MaxAttempts 지정  
C) DLQ 설정  
D) 알람 설정  

**정답: B** - Step Functions의 Retry 설정으로 지수 백오프와 최대 재시도 횟수를 설정할 수 있습니다.

---

**문제 5.** 모바일 앱에서 주문 상태를 실시간으로 업데이트 받으려면?

A) 주기적으로 API 폴링  
B) AppSync GraphQL 구독  
C) SNS Push 알림  
D) SQS Long Polling  

**정답: B** - AppSync의 GraphQL 구독은 WebSocket을 통해 서버에서 클라이언트로 실시간 업데이트를 push합니다.

---

## 📌 오늘의 요약

1. Step Functions: 시각적 워크플로우, Lambda 오케스트레이션
2. 상태 유형: Task, Choice, Parallel, Wait, Succeed, Fail
3. 오류 처리: Retry(자동 재시도), Catch(오류별 분기)
4. AppSync: 완전 관리형 GraphQL, 실시간 구독, 오프라인 동기화
5. AppSync 데이터 소스: DynamoDB, Lambda, RDS, HTTP, OpenSearch
