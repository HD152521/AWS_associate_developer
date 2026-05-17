# Day 17 - API Gateway: 통합 유형, 매핑 템플릿

📅 날짜: 2026년 6월 8일 (월요일)  
🎯 주제: API Gateway 통합 유형 및 변환  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- API Gateway 통합 유형(Lambda Proxy, Lambda, HTTP, AWS, Mock)을 구분한다
- 매핑 템플릿(VTL)으로 요청/응답을 변환한다
- 모델(Model)을 사용하여 요청 데이터를 검증한다

---

## 📖 이론 내용

### 1. 통합 유형 (Integration Types)

#### Lambda Proxy 통합 (AWS_PROXY)
- 요청 전체를 Lambda로 그대로 전달
- API Gateway가 요청/응답 변환을 하지 않음
- Lambda가 응답 형식을 직접 제어 (statusCode, headers, body 포함)
- **⭐ 가장 많이 사용하는 방식**

```python
# Lambda Proxy 응답 형식
def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*'  # CORS
        },
        'body': json.dumps({'message': 'Hello!'})
    }
```

#### Lambda 통합 (AWS)
- 매핑 템플릿으로 요청/응답 변환
- API Gateway와 Lambda 간 계약 설정 가능
- 더 세밀한 제어 가능 (복잡한 설정 필요)

#### HTTP 통합 (HTTP/HTTP_PROXY)
- 외부 HTTP 엔드포인트와 통합
- **HTTP Proxy**: 요청/응답 그대로 전달
- **HTTP**: 매핑 템플릿으로 변환 가능

#### AWS 서비스 통합
- Lambda 없이 직접 AWS 서비스 호출
- 예: DynamoDB, SQS, S3, Kinesis 등
- VTL 매핑 템플릿으로 요청 변환

#### Mock 통합
- 백엔드 없이 정적 응답 반환
- 개발/테스트 시 유용

### 2. 매핑 템플릿 (Mapping Templates)

Lambda Proxy 제외 통합에서 요청/응답을 변환하는 VTL(Velocity Template Language) 템플릿입니다.

**요청 매핑 템플릿 예시 (API → DynamoDB):**
```json
{
  "TableName": "Orders",
  "Key": {
    "orderId": {
      "S": "$input.params('orderId')"
    }
  }
}
```

**응답 매핑 템플릿 예시 (DynamoDB → API):**
```velocity
#set($item = $input.path('$.Item'))
{
  "orderId": "$item.orderId.S",
  "status": "$item.status.S",
  "total": $item.total.N
}
```

### 3. 요청 검증 (Request Validation)

- API Gateway가 백엔드 호출 전에 요청을 검증
- 검증 실패 시 400 오류 반환 (Lambda 호출 없음)
- **검증 가능 항목**: 쿼리 파라미터, 헤더, 요청 본문(Model 기반)

**JSON Schema를 이용한 모델 정의:**
```json
{
  "title": "CreateOrderRequest",
  "type": "object",
  "properties": {
    "productId": {"type": "string"},
    "quantity": {"type": "integer", "minimum": 1},
    "address": {"type": "string"}
  },
  "required": ["productId", "quantity"]
}
```

---

## 아키텍처 다이어그램

```
통합 유형 비교
================================

[Lambda Proxy 통합]
클라이언트 → API GW → Lambda (요청 전체)
                  ←      (statusCode+headers+body)
변환 없음, Lambda가 모든 응답 형식 제어

[AWS 직접 통합 (DynamoDB)]
클라이언트 → API GW → [매핑 템플릿] → DynamoDB
              ←        [매핑 템플릿] ←
중간에 Lambda 없음! API GW가 직접 DynamoDB 호출

[Mock 통합]
클라이언트 → API GW → [정적 응답]
개발 단계에서 백엔드 없이 테스트

API Gateway 요청 처리 흐름
================================

요청 수신
  |
  v
[메서드 요청 검증]
  파라미터, 헤더, 본문(Model)
  |  실패 → 400 반환
  |
  v
[통합 요청 매핑]
  VTL 템플릿으로 변환
  |
  v
[백엔드 호출]
  Lambda / HTTP / DynamoDB / Mock
  |
  v
[통합 응답 매핑]
  VTL 템플릿으로 변환
  |
  v
[메서드 응답]
  HTTP 상태 코드 + 헤더 + 바디
  |
  v
클라이언트에 응답
```

---

## ⭐ 핵심 포인트 (시험 출제 빈도 높음)

1. ⭐ **Lambda Proxy (AWS_PROXY)**: 가장 일반적, Lambda가 응답 형식 직접 제어
2. ⭐ **AWS 직접 통합**: Lambda 없이 DynamoDB/SQS 등 직접 호출 가능
3. ⭐ **요청 검증**: Lambda 호출 전 API GW에서 검증, 400 오류로 비용 절감
4. ⭐ **매핑 템플릿**: VTL 언어 사용, Lambda Proxy에서는 불필요
5. ⭐ **CORS**: Lambda Proxy에서는 Lambda가 Access-Control-Allow-Origin 헤더 반환

---

## 💻 실제 예시

```python
# Lambda Proxy 통합 - 완전한 응답 구조
import json

def lambda_handler(event, context):
    # 이벤트에서 데이터 추출
    http_method = event['httpMethod']
    path = event['path']
    query_params = event.get('queryStringParameters', {})
    path_params = event.get('pathParameters', {})
    body = json.loads(event.get('body', '{}'))
    headers = event.get('headers', {})
    
    # 요청 처리
    if http_method == 'GET':
        result = {'items': ['item1', 'item2']}
        status_code = 200
    elif http_method == 'POST':
        # body 처리
        result = {'created': True, 'id': '123'}
        status_code = 201
    else:
        result = {'error': 'Method not allowed'}
        status_code = 405
    
    return {
        'statusCode': status_code,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*',
            'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
            'Access-Control-Allow-Headers': 'Content-Type, Authorization'
        },
        'body': json.dumps(result, ensure_ascii=False)
    }
```

```bash
# 요청 검증 설정
aws apigateway create-model \
  --rest-api-id abc123 \
  --name CreateOrderRequest \
  --content-type application/json \
  --schema '{
    "title": "CreateOrderRequest",
    "type": "object",
    "properties": {
      "productId": {"type": "string"},
      "quantity": {"type": "integer", "minimum": 1}
    },
    "required": ["productId", "quantity"]
  }'

# 메서드에 검증 설정
aws apigateway create-request-validator \
  --rest-api-id abc123 \
  --name ValidateBody \
  --validate-request-body true \
  --validate-request-parameters false
```

---

## 📝 연습 문제

**문제 1.** Lambda Proxy 통합(AWS_PROXY)의 특징은?

A) API Gateway가 요청과 응답을 변환한다  
B) Lambda 함수가 응답 형식을 직접 제어한다  
C) 매핑 템플릿이 필요하다  
D) Lambda 없이 AWS 서비스를 직접 호출한다  

**정답: B** - Lambda Proxy 통합에서 API Gateway는 요청을 그대로 Lambda에 전달하고, Lambda가 statusCode, headers, body를 포함한 전체 HTTP 응답을 구성합니다.

---

**문제 2.** API Gateway에서 Lambda 없이 DynamoDB를 직접 호출하려면?

A) Lambda Proxy 통합 사용  
B) HTTP 통합 사용  
C) AWS 서비스 통합 사용  
D) Mock 통합 사용  

**정답: C** - AWS 서비스 통합을 사용하면 Lambda 없이 API Gateway가 DynamoDB, SQS 등 AWS 서비스를 직접 호출할 수 있습니다.

---

**문제 3.** API Gateway 요청 검증의 장점은?

A) 더 빠른 Lambda 실행  
B) Lambda 호출 전 잘못된 요청을 차단하여 비용 절감  
C) 자동 인증 처리  
D) 응답 캐싱 개선  

**정답: B** - 요청 검증으로 파라미터나 본문이 잘못된 요청을 Lambda 호출 전에 차단하여 Lambda 실행 비용을 절감하고 백엔드를 보호할 수 있습니다.

---

**문제 4.** CORS를 Lambda Proxy 통합으로 처리할 때 필요한 것은?

A) API Gateway에서만 CORS 헤더 설정  
B) Lambda 함수가 응답에 Access-Control-Allow-Origin 헤더 포함  
C) CloudFront에서 CORS 설정  
D) IAM 역할에 CORS 권한 추가  

**정답: B** - Lambda Proxy 통합에서 Lambda 함수가 응답에 CORS 헤더(Access-Control-Allow-Origin 등)를 직접 포함해야 합니다.

---

**문제 5.** VTL(Velocity Template Language) 매핑 템플릿은 어떤 통합에서 사용되는가?

A) Lambda Proxy (AWS_PROXY) 통합  
B) Lambda (AWS), HTTP, AWS 서비스 통합  
C) Mock 통합에서만 사용  
D) WebSocket 통합에서만 사용  

**정답: B** - VTL 매핑 템플릿은 Lambda Proxy(AWS_PROXY)를 제외한 Lambda(AWS), HTTP, AWS 서비스 통합에서 요청/응답 변환에 사용됩니다.

---

## 📌 오늘의 요약

1. 통합 유형: Lambda Proxy(요청 그대로), AWS(매핑 템플릿), HTTP(외부), AWS 서비스(직접), Mock(정적)
2. Lambda Proxy가 가장 일반적이며 Lambda가 응답 형식(statusCode+headers+body) 전체를 제어한다
3. AWS 서비스 통합으로 Lambda 없이 DynamoDB, SQS 등을 직접 호출 가능하다
4. 매핑 템플릿(VTL)은 Lambda Proxy 제외 통합에서 요청/응답 변환에 사용된다
5. 요청 검증으로 잘못된 요청을 Lambda 호출 전에 차단하여 비용을 절감한다
