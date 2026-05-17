# Day 53 - Kinesis: 실시간 스트리밍

📅 날짜: 2026년 7월 28일 (화요일)  
🎯 주제: Amazon Kinesis  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- Kinesis 4가지 서비스의 역할을 구분한다
- Kinesis Data Streams의 샤드 개념을 이해한다
- Kinesis와 SQS/SNS의 차이를 파악한다

---

## 📖 이론 내용

### 1. Amazon Kinesis란?

실시간 스트리밍 데이터를 수집, 처리, 분석하는 서비스 패밀리입니다.

**4가지 서비스:**
- **Kinesis Data Streams**: 실시간 데이터 스트리밍, 직접 처리
- **Kinesis Data Firehose**: 완전 관리형 데이터 전달, S3/Redshift/OpenSearch
- **Kinesis Data Analytics**: SQL로 스트리밍 데이터 분석
- **Kinesis Video Streams**: 비디오 스트리밍

### 2. Kinesis Data Streams

```
샤드 구조
================================

[Producer]
  |
  | 파티션 키 기반으로 샤드 결정
  v
[샤드 1] [샤드 2] [샤드 3]
  각 샤드:
    - 쓰기: 1MB/s 또는 1,000 레코드/s
    - 읽기: 2MB/s
    - 보존: 기본 24시간 (최대 7일 또는 365일)
  |
  v
[Consumer (Lambda, KCL, SDK)]
```

```python
import boto3
import json

kinesis = boto3.client('kinesis')

# 레코드 전송
kinesis.put_record(
    StreamName='order-stream',
    Data=json.dumps({'orderId': 'O001', 'amount': 50000}),
    PartitionKey='order-001'  # 같은 파티션 키 = 같은 샤드 (순서 보장)
)

# 배치 전송 (최대 500개, 5MB)
kinesis.put_records(
    StreamName='order-stream',
    Records=[
        {
            'Data': json.dumps({'orderId': f'O{i}'}),
            'PartitionKey': f'order-{i}'
        }
        for i in range(10)
    ]
)

# 레코드 읽기
response = kinesis.get_shard_iterator(
    StreamName='order-stream',
    ShardId='shardId-000000000000',
    ShardIteratorType='LATEST'
)
shard_iterator = response['ShardIterator']

records = kinesis.get_records(
    ShardIterator=shard_iterator,
    Limit=10
)
```

### 3. Lambda와 Kinesis 통합

```python
# Lambda가 Kinesis 스트림을 소비
def lambda_handler(event, context):
    for record in event['Records']:
        # Base64 디코딩
        import base64
        payload = base64.b64decode(record['kinesis']['data']).decode('utf-8')
        data = json.loads(payload)
        
        print(f"주문 처리: {data['orderId']}")
        
        # 비즈니스 로직 처리
        process_order(data)
```

### 4. Kinesis Data Firehose

```
완전 관리형 ETL 파이프라인
================================

[Producer]
     |
     v
[Kinesis Data Firehose]
     |
     | 변환 (Lambda, 선택)
     | 버퍼링 (1분 또는 1MB)
     v
[대상]
  - S3 (JSON, CSV, Parquet 변환)
  - Amazon Redshift
  - Amazon OpenSearch
  - Splunk
```

**특징:**
- 서버리스, 자동 확장
- 버퍼링 후 배치 전달
- 실시간보다는 Near Real-Time (1분 이상 지연)

### 5. Kinesis vs SQS

| 특성 | Kinesis Data Streams | SQS |
|------|---------------------|-----|
| 순서 | 샤드 내 순서 보장 | FIFO만 보장 |
| 보존 | 1~365일 | 최대 14일 |
| 소비 | 여러 Consumer 동시 가능 | 하나의 Consumer |
| 용도 | 실시간 분석, IoT | 작업 큐, 디커플링 |

---

## 아키텍처 다이어그램

```
실시간 데이터 파이프라인
================================

[웹 앱 클릭 이벤트]
[IoT 센서 데이터]
[앱 로그]
         |
         | Kinesis SDK/Agent로 전송
         v
[Kinesis Data Streams]
         |
         +-- Lambda (실시간 처리, 알림)
         |
         +-- KCL 앱 (복잡한 처리)
         |
         v
[Kinesis Data Firehose]
         |
         +-- S3 (데이터 레이크)
         |
         +-- Redshift (BI 분석)
         |
         +-- OpenSearch (검색)
```

---

## ⭐ 핵심 포인트

1. ⭐ **샤드**: Kinesis의 기본 단위, 1MB/s 쓰기, 2MB/s 읽기
2. ⭐ **파티션 키**: 같은 키 = 같은 샤드 = 순서 보장
3. ⭐ **Firehose**: 서버리스 ETL, S3/Redshift/OpenSearch 전달
4. ⭐ **데이터 보존**: Kinesis 24시간~365일, SQS 최대 14일
5. ⭐ **여러 Consumer**: Kinesis는 동시에 여러 Consumer 가능

---

## 📝 연습 문제

**문제 1.** Kinesis Data Streams의 각 샤드 쓰기 처리량은?

A) 10MB/s  
B) 1MB/s 또는 1,000 레코드/s  
C) 100MB/s  
D) 5MB/s  

**정답: B** - 각 Kinesis 샤드는 초당 1MB 또는 1,000개의 레코드를 쓸 수 있습니다.

---

**문제 2.** 실시간 클릭스트림 데이터를 S3에 저장하는 가장 쉬운 방법은?

A) Kinesis Data Streams + Lambda + S3 직접 저장  
B) Kinesis Data Firehose → S3  
C) SQS → Lambda → S3  
D) 직접 S3 API 호출  

**정답: B** - Kinesis Data Firehose는 서버리스로 데이터를 수집하고 자동으로 S3에 저장합니다.

---

**문제 3.** Kinesis에서 같은 파티션 키를 사용하는 레코드의 특성은?

A) 다른 샤드에 분산  
B) 같은 샤드에 저장, 순서 보장  
C) 우선순위 처리  
D) 중복 제거  

**정답: B** - 같은 파티션 키를 가진 레코드는 항상 같은 샤드에 저장되어 순서가 보장됩니다.

---

**문제 4.** Kinesis Data Streams와 SQS의 가장 큰 차이점은?

A) 비용 차이  
B) Kinesis는 여러 Consumer가 동시에 같은 데이터를 읽을 수 있음  
C) 처리 속도 차이  
D) 리전 제한  

**정답: B** - Kinesis는 데이터가 스트림에 보존되므로 여러 Consumer가 각자의 위치에서 독립적으로 데이터를 읽을 수 있습니다.

---

**문제 5.** Kinesis Data Firehose의 최소 지연 시간은?

A) 즉시  
B) 1분  
C) 5분  
D) 1시간  

**정답: B** - Firehose는 데이터를 버퍼링하여 배치로 전달하므로 최소 1분의 지연이 발생합니다 (Near Real-Time).

---

## 📌 오늘의 요약

1. Kinesis Data Streams: 실시간 스트리밍, 샤드 단위, 여러 Consumer
2. Kinesis Data Firehose: 서버리스 ETL, S3/Redshift/OpenSearch 전달
3. 샤드: 1MB/s 쓰기, 2MB/s 읽기, 파티션 키로 샤드 결정
4. 보존 기간: 기본 24시간, 최대 365일 (확장 보존)
5. Kinesis vs SQS: Kinesis는 스트리밍/다중Consumer, SQS는 작업큐/단일Consumer
