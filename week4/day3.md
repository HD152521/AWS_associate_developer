# Day 18 - API Gateway: 보안, 캐싱, 스로틀링

📅 날짜: 2026년 6월 9일 (화요일)  
🎯 주제: API Gateway 보안 및 성능  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- API Gateway의 인증/인가 방법을 이해한다
- 응답 캐싱으로 성능을 최적화한다
- 스로틀링과 사용량 플랜으로 API를 보호한다

---

## 📖 이론 내용

### 1. API Gateway 인증/인가

#### IAM 인증
- AWS 자격 증명으로 API 요청 서명 (Signature Version 4)
- AWS 내부 서비스 간 호출에 적합
- `aws:SourceIp` 조건으로 IP 제한 가능

#### Lambda 권한 부여자 (Authorizer)
- **토큰 기반**: JWT, OAuth 토큰 검증
- **요청 기반**: 헤더, 쿼리 파라미터, 경로 파라미터로 검증
- Lambda 함수가 IAM 정책 반환 (Allow/Deny)
- **⭐ 결과를 TTL 기간 동안 캐시 가능** (기본 300초)

```python
# Lambda 권한 부여자 예시
def lambda_handler(event, context):
    token = event.get('authorizationToken', '')
    
    if validate_token(token):
        return generate_policy('user123', 'Allow', event['methodArn'])
    else:
        return generate_policy('user123', 'Deny', event['methodArn'])

def generate_policy(principal_id, effect, resource):
    return {
        'principalId': principal_id,
        'policyDocument': {
            'Version': '2012-10-17',
            'Statement': [{
                'Action': 'execute-api:Invoke',
                'Effect': effect,
                'Resource': resource
            }]
        }
    }
```

#### Cognito 사용자 풀 권한 부여자
- Cognito 사용자 풀의 JWT 토큰 자동 검증
- Lambda 코드 없이 설정 가능
- **⭐ 소셜 로그인(Google, Facebook)과 통합 가능**

#### API 키
- `x-api-key` 헤더로 전달
- 사용량 플랜과 함께 사용하여 호출 제한
- **⭐ 인증 수단이 아닌 식별 및 사용량 추적 수단**

### 2. 응답 캐싱 (Caching)

- 스테이지 레벨에서 설정 (0.5GB ~ 237GB)
- TTL: 0~3600초 (기본 300초)
- **⭐ 클라이언트가 Cache-Control: max-age=0 헤더로 캐시 무효화 가능**
- 캐시 키: 메서드 파라미터, 헤더, 쿼리 파라미터

### 3. 스로틀링 (Throttling)

**계층적 스로틀링:**
1. **계정 수준**: 리전당 초당 10,000 요청 (기본)
2. **스테이지/메서드 수준**: 스테이지/메서드별 한도
3. **사용량 플랜**: API 키별 한도 및 할당량

**429 Too Many Requests**: 스로틀링 발생 시 오류 코드

**사용량 플랜 (Usage Plan):**
- 스로틀링: 초당 요청 수 (RPS) 및 버스트 한도
- 할당량: 일/주/월당 최대 요청 수
- **⭐ API 키와 결합하여 고객별 차등 제한 설정**

---

## 아키텍처 다이어그램

```
API Gateway 인증 흐름
================================

[클라이언트]
     |
     | 요청 + 인증 정보
     v
[API Gateway]
     |
     +--[IAM 인증]--> SigV4 서명 검증
     |
     +--[Cognito]--> Cognito 사용자 풀 토큰 검증
     |
     +--[Lambda 권한부여자]
     |       |
     |       v
     |  [Lambda 함수]
     |       |
     |       v
     |  [IAM 정책 반환] --> Allow/Deny
     |       |
     |  [캐시 (TTL 300s)]
     |
     v (Allow인 경우)
[백엔드 통합]

캐싱 동작
================================

첫 번째 요청 (캐시 없음):
클라이언트 → API GW → Lambda → 응답 → [캐시 저장] → 클라이언트

동일 요청 (TTL 이내):
클라이언트 → API GW → [캐시 히트] → 클라이언트 (Lambda 호출 없음!)

캐시 무효화:
클라이언트 → API GW (Cache-Control: max-age=0) → Lambda → 새 응답
```

---

## ⭐ 핵심 포인트 (시험 출제 빈도 높음)

1. ⭐ **Lambda 권한 부여자**: 사용자 정의 인증 로직, 결과 캐시 가능
2. ⭐ **Cognito 권한 부여자**: JWT 자동 검증, Lambda 코드 불필요
3. ⭐ **API 키 != 인증**: 사용량 추적/제한 목적, 보안 수단이 아님
4. ⭐ **캐시 무효화**: Cache-Control: max-age=0 헤더 사용
5. ⭐ **429 오류**: 스로틀링 발생 시 클라이언트가 받는 오류 코드

---

## 💻 실제 예시

```bash
# API 키 생성
aws apigateway create-api-key \
  --name "PremiumCustomer-Key" \
  --enabled

# 사용량 플랜 생성
aws apigateway create-usage-plan \
  --name "PremiumPlan" \
  --throttle burstLimit=100,rateLimit=50 \
  --quota limit=10000,period=MONTH \
  --api-stages apiId=abc123,stage=prod

# API 키를 사용량 플랜에 연결
aws apigateway create-usage-plan-key \
  --usage-plan-id plan123 \
  --key-id key123 \
  --key-type API_KEY

# 스테이지 캐시 설정
aws apigateway update-stage \
  --rest-api-id abc123 \
  --stage-name prod \
  --patch-operations \
    op=replace,path=/cacheClusterEnabled,value=true \
    op=replace,path=/cacheClusterSize,value=0.5 \
    op=replace,path=/*/*/caching/enabled,value=true \
    op=replace,path=/*/*/caching/ttlInSeconds,value=300
```

---

## 📝 연습 문제

**문제 1.** API Gateway에서 외부 JWT 토큰을 검증하는 가장 적합한 방법은?

A) IAM 인증  
B) Lambda 권한 부여자  
C) API 키  
D) 리소스 정책  

**정답: B** - Lambda 권한 부여자를 사용하면 사용자 정의 인증 로직으로 JWT 토큰을 검증하고 IAM 정책을 반환할 수 있습니다.

---

**문제 2.** API 키의 주요 목적은?

A) 강력한 인증 보안 제공  
B) 클라이언트 식별 및 사용량 추적/제한  
C) 데이터 암호화  
D) IAM 역할 대체  

**정답: B** - API 키는 인증 수단이 아니라 클라이언트를 식별하고 사용량을 추적하며 사용량 플랜을 통해 요청을 제한하는 목적으로 사용됩니다.

---

**문제 3.** API Gateway 캐시에서 클라이언트가 최신 데이터를 강제로 받으려면?

A) 요청에 Refresh: true 헤더 추가  
B) 요청에 Cache-Control: max-age=0 헤더 추가  
C) 스테이지를 재배포  
D) API를 재시작  

**정답: B** - Cache-Control: max-age=0 헤더를 요청에 포함하면 API Gateway가 캐시를 무시하고 백엔드를 직접 호출합니다.

---

**문제 4.** API Gateway에서 초당 요청 수가 한도를 초과할 때 반환되는 HTTP 상태 코드는?

A) 400 Bad Request  
B) 401 Unauthorized  
C) 403 Forbidden  
D) 429 Too Many Requests  

**정답: D** - API Gateway 스로틀링 한도 초과 시 HTTP 429 (Too Many Requests) 오류가 반환됩니다.

---

**문제 5.** Lambda 권한 부여자 결과를 캐싱하면 어떤 이점이 있는가?

A) Lambda 실행 시간 감소  
B) 권한 부여자 Lambda 호출 감소로 비용 절감  
C) API 응답 시간 증가  
D) 더 많은 동시 연결 지원  

**정답: B** - Lambda 권한 부여자 결과를 캐싱(기본 300초)하면 동일한 토큰으로 반복 요청 시 Lambda를 다시 호출하지 않아 지연 시간과 비용이 줄어듭니다.

---

## 📌 오늘의 요약

1. 인증 방법: IAM(AWS 서비스), Lambda 권한 부여자(사용자 정의), Cognito(소셜/사용자 풀)
2. API 키는 인증이 아닌 사용량 추적/제한 목적으로 사용됨
3. 응답 캐싱으로 Lambda 호출을 줄여 비용 절감, Cache-Control: max-age=0으로 무효화 가능
4. 스로틀링 초과 시 429 오류, 사용량 플랜으로 API 키별 차등 제한 설정 가능
5. Lambda 권한 부여자 캐시(기본 300초)로 인증 Lambda 반복 호출 방지
