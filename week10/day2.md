# Day 47 - X-Ray: 분산 추적

📅 날짜: 2026년 7월 20일 (월요일)  
🎯 주제: AWS X-Ray  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- X-Ray 분산 추적의 개념을 이해한다
- X-Ray SDK를 Lambda와 EC2에 통합한다
- X-Ray 서비스 맵과 트레이스를 분석한다

---

## 📖 이론 내용

### 1. AWS X-Ray란?

마이크로서비스 아키텍처에서 요청의 흐름을 추적하는 분산 추적 서비스입니다.

**X-Ray가 해결하는 문제:**
```
API Gateway → Lambda → DynamoDB → ElastiCache
    |
    어디서 지연이 발생하는가?
    어디서 오류가 발생하는가?
    어떤 서비스가 병목인가?
```

**핵심 개념:**
- **Trace**: 하나의 요청 전체 흐름
- **Segment**: 각 서비스에서의 처리 정보
- **Subsegment**: 세부 작업 (DB 쿼리, HTTP 호출 등)
- **Annotation**: 인덱싱 가능한 키-값 (필터링 가능)
- **Metadata**: 인덱싱 불가 추가 정보

### 2. Lambda에서 X-Ray 활성화

```python
# Lambda에서 X-Ray SDK 사용
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.core import patch_all

# AWS SDK 자동 패치 (boto3 등)
patch_all()

@xray_recorder.capture('process_order')
def process_order(order_id):
    # 이 함수의 실행 시간이 자동으로 추적됨
    with xray_recorder.in_subsegment('validate_order') as subsegment:
        # 주석 추가 (필터링 가능)
        subsegment.put_annotation('orderId', order_id)
        subsegment.put_annotation('environment', 'production')
        
        # 메타데이터 추가 (필터링 불가)
        subsegment.put_metadata('orderDetails', {'amount': 50000})
        
        result = validate(order_id)
    
    return result

def lambda_handler(event, context):
    order_id = event.get('orderId')
    return process_order(order_id)
```

**Lambda X-Ray 활성화:**
```bash
# Lambda 함수에서 X-Ray 활성화
aws lambda update-function-configuration \
    --function-name my-function \
    --tracing-config Mode=Active
```

### 3. EC2/ECS에서 X-Ray 데몬

EC2/ECS에서는 X-Ray 데몬이 필요합니다:

```bash
# X-Ray 데몬 설치 및 실행
curl https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-3.x.rpm -o xray.rpm
sudo yum install -y xray.rpm
sudo systemctl start xray
```

**X-Ray 데몬 동작:**
```
[애플리케이션 SDK]
     |
     | UDP 포트 2000으로 세그먼트 전송
     v
[X-Ray 데몬]
     |
     | HTTPS로 X-Ray 서비스 전송
     v
[AWS X-Ray 서비스]
```

### 4. X-Ray 서비스 맵

```
시각적으로 표현:

[사용자] → [API Gateway] → [Lambda] → [DynamoDB]
                                 |
                                 v
                         [ElastiCache]
                         
각 연결선에:
- 평균 응답 시간
- 요청 수 (RPM)
- 오류율 (%)
표시됨
```

### 5. X-Ray 샘플링

모든 요청을 추적하면 비용이 많이 들 수 있습니다:

```json
{
  "version": 2,
  "rules": [
    {
      "description": "상태 확인 제외",
      "host": "*",
      "http_method": "GET",
      "url_path": "/health",
      "fixed_target": 0,
      "rate": 0.00
    }
  ],
  "default": {
    "fixed_target": 1,
    "rate": 0.05
  }
}
```

**기본 샘플링**: 처음 1개 요청 + 이후 초당 5%

---

## 아키텍처 다이어그램

```
X-Ray 분산 추적 흐름
================================

[클라이언트 요청]
      |
      v
[API Gateway] → Segment (100ms)
      |
      v
[Lambda 함수] → Segment (250ms)
      |
      +-- DynamoDB GetItem → Subsegment (30ms)
      |
      +-- ElastiCache Get → Subsegment (5ms)
      |
      +-- 외부 API 호출 → Subsegment (200ms)

전체 트레이스 시간: ~585ms
병목 위치: 외부 API (200ms)

X-Ray 콘솔에서:
  서비스 맵: 각 서비스 연결과 응답 시간 시각화
  트레이스: 개별 요청의 세부 타임라인
```

---

## ⭐ 핵심 포인트

1. ⭐ **Trace**: 하나의 요청 전체 흐름
2. ⭐ **Annotation**: 인덱싱 가능, 필터링/검색에 사용
3. ⭐ **EC2/ECS**: X-Ray 데몬 별도 설치 필요
4. ⭐ **Lambda**: 콘솔에서 X-Ray 활성화만 하면 됨 (데몬 불필요)
5. ⭐ **샘플링**: 기본 처음 1개 + 초당 5%, 비용 최적화

---

## 📝 연습 문제

**문제 1.** X-Ray에서 필터링 가능한 데이터를 추가하려면?

A) Metadata  
B) Annotation  
C) Comment  
D) Tag  

**정답: B** - Annotation은 키-값 쌍으로 X-Ray 콘솔에서 필터링 및 검색이 가능합니다.

---

**문제 2.** EC2에서 X-Ray를 사용하기 위한 전제 조건은?

A) CloudWatch Agent 설치  
B) X-Ray 데몬 설치 및 실행  
C) VPC Flow Logs 활성화  
D) CloudTrail 활성화  

**정답: B** - EC2에서 X-Ray를 사용하려면 X-Ray 데몬을 설치하고 실행해야 합니다.

---

**문제 3.** X-Ray 서비스 맵에서 알 수 있는 정보가 아닌 것은?

A) 서비스 간 연결 관계  
B) 평균 응답 시간  
C) 서비스 비용  
D) 오류율  

**정답: C** - X-Ray 서비스 맵은 연결 관계, 응답 시간, 요청 수, 오류율을 보여주지만 비용 정보는 포함되지 않습니다.

---

**문제 4.** X-Ray 기본 샘플링 규칙은?

A) 모든 요청 추적  
B) 처음 1개 + 초당 5%  
C) 처음 10개 + 초당 1%  
D) 1분당 1개  

**정답: B** - X-Ray 기본 샘플링은 매 초 처음 1개 요청과 이후 5%의 요청을 추적합니다.

---

**문제 5.** Lambda에서 X-Ray를 활성화하는 방법은?

A) X-Ray 데몬 Lambda Layer 추가  
B) Lambda 설정에서 X-Ray Active Tracing 활성화  
C) CloudWatch와 연동  
D) VPC 설정 변경  

**정답: B** - Lambda 콘솔이나 CLI에서 X-Ray Active Tracing을 활성화하면 됩니다. 데몬 설치가 필요 없습니다.

---

## 📌 오늘의 요약

1. X-Ray: 분산 추적, 마이크로서비스 디버깅 및 성능 분석
2. Trace/Segment/Subsegment로 계층적 요청 추적
3. Annotation: 필터 가능, Metadata: 필터 불가 추가 정보
4. Lambda: Active Tracing 활성화만으로 사용 가능
5. EC2/ECS: X-Ray 데몬 설치 필요, UDP 2000 포트로 통신
