# Day 29 - DynamoDB: 트랜잭션, 조건부 쓰기

📅 날짜: 2026년 6월 24일 (수요일)  
🎯 주제: DynamoDB 고급 쓰기 기능  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- DynamoDB 트랜잭션의 동작 방식을 이해한다
- 조건부 쓰기로 경쟁 조건을 방지한다
- Optimistic Locking으로 동시성 문제를 해결한다

---

## 📖 이론 내용

### 1. DynamoDB 트랜잭션

ACID 트랜잭션을 지원하는 두 가지 API:

**TransactWriteItems:**
- 최대 25개 항목, 4MB 이하
- 여러 테이블의 쓰기를 원자적으로 실행
- 전부 성공하거나 전부 실패 (All-or-Nothing)

**TransactGetItems:**
- 최대 25개 항목, 4MB 이하
- 여러 테이블의 읽기를 원자적으로 실행

**비용:**
- 쓰기: 2 WCU/1KB (일반의 2배)
- 읽기: 2 RCU/4KB (일반의 2배)

```python
import boto3

dynamodb = boto3.client('dynamodb')

# 트랜잭션 쓰기 예시 (주문 생성 + 재고 감소를 원자적으로)
response = dynamodb.transact_write_items(
    TransactItems=[
        {
            'Put': {
                'TableName': 'Orders',
                'Item': {
                    'orderId': {'S': 'O001'},
                    'userId': {'S': 'U001'},
                    'total': {'N': '50000'}
                }
            }
        },
        {
            'Update': {
                'TableName': 'Inventory',
                'Key': {'productId': {'S': 'P001'}},
                'UpdateExpression': 'SET quantity = quantity - :qty',
                'ConditionExpression': 'quantity >= :qty',
                'ExpressionAttributeValues': {
                    ':qty': {'N': '1'}
                }
            }
        }
    ]
)
```

### 2. 조건부 쓰기 (Conditional Writes)

조건이 만족될 때만 쓰기 수행:

```python
# 조건부 업데이트 (재고가 있는 경우만 감소)
dynamodb.update_item(
    TableName='Inventory',
    Key={'productId': {'S': 'P001'}},
    UpdateExpression='SET quantity = quantity - :dec',
    ConditionExpression='quantity > :zero',
    ExpressionAttributeValues={
        ':dec': {'N': '1'},
        ':zero': {'N': '0'}
    }
)

# 조건부 삽입 (항목이 없는 경우만)
dynamodb.put_item(
    TableName='Users',
    Item={'userId': {'S': 'U001'}, 'name': {'S': 'Kim'}},
    ConditionExpression='attribute_not_exists(userId)'
)
```

### 3. Optimistic Locking (낙관적 잠금)

버전 번호를 사용하여 동시 수정 방지:

```python
# 버전 번호 기반 조건부 업데이트
dynamodb.update_item(
    TableName='Products',
    Key={'productId': {'S': 'P001'}},
    UpdateExpression='SET price = :newPrice, version = version + :inc',
    ConditionExpression='version = :currentVersion',
    ExpressionAttributeValues={
        ':newPrice': {'N': '29900'},
        ':inc': {'N': '1'},
        ':currentVersion': {'N': '3'}
    }
)
# version이 3이 아니면 ConditionalCheckFailedException 발생
```

### 4. DynamoDB TTL (Time-To-Live)

항목 자동 만료 기능:
- 지정된 타임스탬프가 지나면 자동 삭제
- 추가 비용 없음
- 만료 후 48시간 내 삭제 (비동기)
- **⭐ 세션, 캐시, 임시 데이터 관리에 유용**

---

## 아키텍처 다이어그램

```
DynamoDB 트랜잭션 동작
================================

주문 처리 트랜잭션:

BEGIN TRANSACTION
  PUT Orders(orderId=O001, total=50000)
  UPDATE Inventory(productId=P001, quantity -= 1)
    WHERE quantity >= 1
END TRANSACTION

성공: 주문 생성 + 재고 감소 동시 반영
실패: 재고 없음 → 주문도 취소 (All-or-Nothing)

Optimistic Locking 흐름
================================

사용자 A가 가격 읽기 (version=3)
사용자 B가 가격 읽기 (version=3)

사용자 A가 먼저 업데이트:
  SET price=29900, version=4
  WHERE version=3 → 성공!

사용자 B가 업데이트 시도:
  SET price=35000, version=4
  WHERE version=3 → 실패! (이미 4로 변경됨)
  ConditionalCheckFailedException!

B는 최신 데이터를 다시 읽고 재시도
```

---

## ⭐ 핵심 포인트

1. ⭐ **트랜잭션 비용**: 쓰기 2 WCU/KB, 읽기 2 RCU/4KB (일반의 2배)
2. ⭐ **트랜잭션 제한**: 최대 25개 항목, 4MB
3. ⭐ **attribute_not_exists**: 항목이 없을 때만 삽입 조건
4. ⭐ **TTL**: 만료 타임스탬프로 자동 삭제, 추가 비용 없음
5. ⭐ **Optimistic Locking**: 버전 번호로 경쟁 조건 방지

---

## 📝 연습 문제

**문제 1.** DynamoDB 트랜잭션의 최대 항목 수는?

A) 10개  
B) 25개  
C) 50개  
D) 100개  

**정답: B** - DynamoDB 트랜잭션은 최대 25개 항목까지 지원합니다.

---

**문제 2.** 항목이 없을 때만 삽입하는 조건 표현식은?

A) attribute_exists(pk)  
B) attribute_not_exists(pk)  
C) not_exists(pk)  
D) if_not_exists(pk)  

**정답: B** - attribute_not_exists(pk) 조건을 사용하면 파티션 키가 없는 경우(즉, 새 항목)만 삽입됩니다.

---

**문제 3.** Optimistic Locking의 장점은?

A) 쓰기 비용 절감  
B) 동시 수정 시 데이터 충돌 방지  
C) 읽기 성능 향상  
D) 자동 백업  

**정답: B** - Optimistic Locking은 버전 번호를 통해 동시 수정으로 인한 데이터 충돌을 감지하고 방지합니다.

---

**문제 4.** DynamoDB TTL에 대한 올바른 설명은?

A) 추가 삭제 비용이 발생한다  
B) 만료 즉시 삭제된다  
C) 세션이나 임시 데이터 자동 정리에 유용하다  
D) 모든 테이블에 반드시 설정해야 한다  

**정답: C** - DynamoDB TTL은 추가 비용 없이 만료된 데이터를 자동으로 삭제하여 세션, 캐시, 임시 데이터 관리에 유용합니다.

---

**문제 5.** 트랜잭션 쓰기의 WCU 소비는 일반 쓰기의 몇 배인가?

A) 0.5배  
B) 1배  
C) 2배  
D) 4배  

**정답: C** - 트랜잭션 쓰기는 일반 쓰기의 2배 WCU를 소비합니다 (2 WCU/1KB vs 1 WCU/1KB).

---

## 📌 오늘의 요약

1. DynamoDB 트랜잭션: 여러 테이블의 쓰기/읽기를 원자적으로 실행, 최대 25개 항목
2. 트랜잭션 비용: 쓰기 2 WCU/KB, 읽기 2 RCU/4KB (일반의 2배)
3. 조건부 쓰기: 조건 만족 시만 실행, attribute_not_exists로 중복 방지
4. Optimistic Locking: 버전 번호로 동시 수정 충돌 감지 및 방지
5. TTL: 무료 자동 삭제, 만료 후 48시간 내 처리
