# Day 12 - Lambda 트리거 및 이벤트 소스

📅 날짜: 2026년 6월 1일 (월요일)  
🎯 주제: Lambda 이벤트 소스와 트리거  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- Lambda를 트리거하는 다양한 이벤트 소스를 이해한다
- 동기 호출과 비동기 호출의 차이를 구분한다
- 이벤트 소스 매핑(Event Source Mapping)의 동작 방식을 이해한다

---

## 📖 이론 내용

### 1. Lambda 호출 방식

#### 동기 호출 (Synchronous)
- 결과를 기다림
- 오류 발생 시 재시도는 호출자 책임
- 예: API Gateway, Cognito, ALB

#### 비동기 호출 (Asynchronous)
- 결과를 기다리지 않음
- Lambda가 내부적으로 재시도 (최대 2회)
- 실패 시 DLQ(Dead Letter Queue) 또는 Destination으로 이벤트 전송
- 예: S3, SNS, EventBridge

#### 이벤트 소스 매핑 (Event Source Mapping)
- Lambda가 폴링하여 이벤트를 가져옴
- 예: SQS, Kinesis, DynamoDB Streams, Kafka

### 2. 주요 이벤트 소스

#### API Gateway (동기)
- REST API, HTTP API, WebSocket API
- HTTP 요청 → Lambda 호출 → HTTP 응답
- 가장 일반적인 서버리스 패턴

#### S3 (비동기)
- 객체 생성, 삭제, 복원 등 이벤트
- **⭐ S3 Event Notification**: S3 → Lambda, SQS, SNS 직접 발송
- 주의: 동일 버킷에서 무한 루프 방지 (읽기/쓰기 버킷 분리)

#### SQS (이벤트 소스 매핑)
- Lambda가 SQS 큐를 폴링
- 배치 크기: 1~10,000 메시지
- **⭐ 처리 실패 시 메시지 큐로 반환** (가시성 타임아웃 후)
- FIFO SQS: Lambda 함수도 FIFO 보장

#### DynamoDB Streams (이벤트 소스 매핑)
- 테이블 변경 사항(INSERT, MODIFY, REMOVE) 스트림
- 배치 크기: 1~10,000 레코드
- 샤드당 하나의 Lambda 함수

#### SNS (비동기)
- SNS 주제 → Lambda 구독
- 팬아웃 패턴 가능

#### EventBridge (비동기)
- 스케줄 기반 실행 (Cron)
- 이벤트 패턴 매칭

#### Kinesis Data Streams (이벤트 소스 매핑)
- 샤드당 하나의 Lambda 함수
- 배치 크기: 1~10,000 레코드

### 3. 이벤트 소스 매핑 상세

```
[SQS/Kinesis/DynamoDB Streams]
        |
        | (Lambda가 폴링)
        v
[Lambda 서비스]
        |
        | (배치로 레코드 가져옴)
        v
[Lambda 함수 실행]
        |
 성공?  +-- YES --> 레코드 삭제 (SQS) / 위치 전진 (Kinesis)
        |
        NO
        |
        v
 [재시도 or DLQ 전송]
```

---

## 아키텍처 다이어그램

```
Lambda 이벤트 소스 전체 구성
================================

[동기 호출]
API Gateway --> Lambda --> 응답 반환
ALB         --> Lambda --> 응답 반환
Cognito     --> Lambda --> 응답 반환

[비동기 호출]
S3          --> Lambda (성공/실패 이벤트 → Destination)
SNS         --> Lambda
EventBridge --> Lambda

[이벤트 소스 매핑 - 폴링]
SQS Queue   <-- Lambda (폴링) --> Lambda 실행
Kinesis     <-- Lambda (폴링) --> Lambda 실행
DynamoDB    <-- Lambda (폴링) --> Lambda 실행
Streams

서버리스 이미지 처리 파이프라인
================================

사용자
  |
  | 이미지 업로드
  v
[S3 원본 버킷]
  |
  | S3 이벤트 알림 (비동기)
  v
[Lambda - 이미지 리사이즈]
  |
  | 처리된 이미지 저장
  v
[S3 처리 버킷]
  |
  | SNS 알림 발송
  v
[SNS Topic]
  |
  +------> [이메일 알림]
  |
  +------> [SQS Queue]
                |
                v
          [Lambda - DB 업데이트]
                |
                v
          [DynamoDB]
```

---

## ⭐ 핵심 포인트 (시험 출제 빈도 높음)

1. ⭐ **동기 vs 비동기**: API GW=동기, S3/SNS=비동기, SQS=이벤트 소스 매핑
2. ⭐ **SQS + Lambda**: Lambda가 폴링, 실패 시 메시지 가시성 타임아웃 후 재시도
3. ⭐ **S3 → Lambda 무한 루프 주의**: 처리된 파일을 같은 버킷에 저장하면 재귀 호출
4. ⭐ **비동기 재시도**: 최대 2회, 이후 DLQ 또는 Destination으로 이벤트 전송
5. ⭐ **이벤트 소스 매핑**: Lambda가 능동적으로 폴링 (push 방식 아님)

---

## 💻 실제 예시

```python
# S3 이벤트 핸들러 (이미지 리사이즈)
import json
import boto3
from PIL import Image
import io

s3 = boto3.client('s3')

def lambda_handler(event, context):
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        
        # 원본 이미지 다운로드
        response = s3.get_object(Bucket=bucket, Key=key)
        image_data = response['Body'].read()
        
        # 이미지 리사이즈
        image = Image.open(io.BytesIO(image_data))
        image.thumbnail((800, 600))
        
        # 처리된 이미지를 다른 버킷에 저장
        output = io.BytesIO()
        image.save(output, format='JPEG')
        output.seek(0)
        
        dest_bucket = 'processed-images-bucket'
        dest_key = f'resized/{key}'
        
        s3.put_object(
            Bucket=dest_bucket,
            Key=dest_key,
            Body=output.getvalue()
        )
        
        print(f"이미지 처리 완료: {bucket}/{key} -> {dest_bucket}/{dest_key}")
    
    return {'statusCode': 200}
```

```bash
# S3 이벤트 알림 설정 (Lambda 트리거)
aws s3api put-bucket-notification-configuration \
  --bucket source-images \
  --notification-configuration '{
    "LambdaFunctionConfigurations": [
      {
        "LambdaFunctionArn": "arn:aws:lambda:ap-northeast-2:123:function:ImageResize",
        "Events": ["s3:ObjectCreated:*"],
        "Filter": {
          "Key": {
            "FilterRules": [
              {"Name": "suffix", "Value": ".jpg"}
            ]
          }
        }
      }
    ]
  }'

# SQS 이벤트 소스 매핑 생성
aws lambda create-event-source-mapping \
  --function-name ProcessOrders \
  --event-source-arn arn:aws:sqs:ap-northeast-2:123:orders-queue \
  --batch-size 10 \
  --maximum-batching-window-in-seconds 5

# Lambda 비동기 호출 Destination 설정
aws lambda put-function-event-invoke-config \
  --function-name my-function \
  --maximum-retry-attempts 2 \
  --destination-config '{
    "OnSuccess": {
      "Destination": "arn:aws:sqs:ap-northeast-2:123:success-queue"
    },
    "OnFailure": {
      "Destination": "arn:aws:sqs:ap-northeast-2:123:failure-queue"
    }
  }'
```

---

## 📝 연습 문제

**문제 1.** S3에 파일이 업로드될 때 Lambda 함수를 자동으로 실행하는 것은 어떤 호출 유형인가?

A) 동기 호출  
B) 비동기 호출  
C) 이벤트 소스 매핑  
D) 직접 호출  

**정답: B**  
해설: S3 이벤트 알림에 의한 Lambda 호출은 비동기 방식입니다. Lambda는 결과를 S3에 반환하지 않으며, 실패 시 자동으로 최대 2회 재시도합니다.

---

**문제 2.** SQS와 Lambda 통합에 대한 올바른 설명은?

A) SQS가 Lambda를 직접 호출한다  
B) Lambda가 SQS를 폴링하여 메시지를 가져온다  
C) 메시지 처리 실패 시 즉시 삭제된다  
D) 배치 크기는 최대 100개로 제한된다  

**정답: B**  
해설: SQS-Lambda 통합에서 Lambda 서비스가 SQS 큐를 폴링합니다. 처리 실패 시 메시지는 가시성 타임아웃 후 다시 큐에 나타납니다. 배치 크기는 최대 10,000개입니다.

---

**문제 3.** S3 버킷에 파일이 업로드되면 Lambda가 처리 후 같은 버킷에 저장할 때 발생하는 문제는?

A) IAM 권한 오류  
B) 무한 재귀 호출  
C) 파일 크기 제한 초과  
D) 타임아웃 오류  

**정답: B**  
해설: 처리된 파일을 동일한 버킷에 저장하면 새 파일 업로드 이벤트가 발생하여 Lambda가 다시 호출되는 무한 루프가 생깁니다. 입력/출력 버킷을 분리하거나 파일명 필터를 사용해야 합니다.

---

**문제 4.** Lambda 비동기 호출에서 실패한 이벤트를 처리하는 방법은?

A) 동기 재시도 설정  
B) DLQ(Dead Letter Queue) 또는 Lambda Destination 설정  
C) CloudWatch 경보 설정  
D) API Gateway 재시도 설정  

**정답: B**  
해설: Lambda 비동기 호출 실패 시 DLQ(SQS 또는 SNS) 또는 Lambda Destinations를 설정하여 실패한 이벤트를 별도로 처리할 수 있습니다.

---

**문제 5.** API Gateway를 통한 Lambda 호출 방식은?

A) 비동기 호출  
B) 이벤트 소스 매핑  
C) 동기 호출  
D) 스케줄 호출  

**정답: C**  
해설: API Gateway는 Lambda를 동기 방식으로 호출합니다. 클라이언트가 HTTP 요청을 보내면 Lambda 함수가 실행되고 결과가 API Gateway를 통해 클라이언트에게 반환됩니다.

---

## 📌 오늘의 요약

1. Lambda 호출 유형: 동기(API GW), 비동기(S3/SNS), 이벤트 소스 매핑(SQS/Kinesis)
2. 비동기 호출 실패: 최대 2회 재시도, 이후 DLQ 또는 Destination으로 이벤트 전송
3. SQS+Lambda: Lambda가 폴링하는 이벤트 소스 매핑, 배치 크기 최대 10,000
4. S3 이벤트 처리: 입력/출력 버킷 분리 또는 파일명 필터로 무한 루프 방지
5. Kinesis/DynamoDB Streams: 샤드당 하나의 Lambda, 순서 보장 처리
