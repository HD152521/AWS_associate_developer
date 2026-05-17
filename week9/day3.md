# Day 43 - Cognito: 사용자 인증 및 권한

📅 날짜: 2026년 7월 14일 (화요일)  
🎯 주제: Amazon Cognito  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- Cognito User Pool과 Identity Pool의 차이를 이해한다
- API Gateway와 Cognito를 통합하는 방법을 학습한다
- 소셜 로그인을 구현한다

---

## 📖 이론 내용

### 1. Amazon Cognito란?

웹/모바일 앱의 사용자 인증(Authentication)과 권한(Authorization)을 제공하는 서비스입니다.

**두 가지 구성 요소:**
- **User Pool**: 사용자 디렉터리, 인증(로그인/회원가입)
- **Identity Pool**: AWS 리소스 접근 권한 부여 (IAM 역할)

### 2. Cognito User Pool

```
User Pool 인증 흐름
================================

[사용자]
    |
    | 1. 로그인 요청 (username/password)
    v
[Cognito User Pool]
    |
    | 2. 인증 성공
    v
[JWT 토큰 발급]
    - ID Token (사용자 정보)
    - Access Token (API 접근)
    - Refresh Token (토큰 갱신)
    |
    v
[API Gateway]
    |
    | 3. JWT 토큰 검증 (Cognito Authorizer)
    v
[Lambda 함수]
```

**User Pool 기능:**
- 회원가입/로그인
- 이메일/SMS 검증
- MFA (다중 인증)
- 소셜 로그인 (Google, Facebook, Apple)
- 커스텀 Lambda 트리거 (사전/이후 인증 처리)

### 3. Cognito User Pool Lambda 트리거

```python
# Pre Authentication 트리거 예시
def lambda_handler(event, context):
    # 사용자 로그인 전 커스텀 검증
    if event['request']['userAttributes']['email'].endswith('@blocked.com'):
        raise Exception("차단된 도메인입니다.")
    
    # 이벤트 반환
    return event
```

**트리거 종류:**
- `PreSignUp`: 회원가입 전 검증
- `PostConfirmation`: 이메일 확인 후 처리
- `PreAuthentication`: 로그인 전 검증
- `PostAuthentication`: 로그인 후 처리
- `CustomMessage`: 이메일/SMS 메시지 커스터마이징

### 4. Cognito Identity Pool

User Pool의 토큰을 AWS IAM 임시 자격 증명으로 교환합니다.

```python
import boto3

# Identity Pool에서 임시 자격 증명 요청
cognito_identity = boto3.client('cognito-identity')

# Identity ID 가져오기
identity_id = cognito_identity.get_id(
    IdentityPoolId='ap-northeast-2:pool-id',
    Logins={
        'cognito-idp.ap-northeast-2.amazonaws.com/pool-id': id_token
    }
)['IdentityId']

# 임시 자격 증명 가져오기
credentials = cognito_identity.get_credentials_for_identity(
    IdentityId=identity_id,
    Logins={
        'cognito-idp.ap-northeast-2.amazonaws.com/pool-id': id_token
    }
)['Credentials']

# S3에 직접 접근 (임시 자격 증명 사용)
s3 = boto3.client(
    's3',
    aws_access_key_id=credentials['AccessKeyId'],
    aws_secret_access_key=credentials['SecretKey'],
    aws_session_token=credentials['SessionToken']
)
```

### 5. User Pool vs Identity Pool

| 특성 | User Pool | Identity Pool |
|------|-----------|---------------|
| 목적 | 사용자 인증 | AWS 리소스 접근 |
| 반환 | JWT 토큰 | IAM 임시 자격 증명 |
| 사용 | API Gateway 인증 | S3, DynamoDB 직접 접근 |
| 게스트 지원 | 불가 | 가능 (인증되지 않은 역할) |

---

## 아키텍처 다이어그램

```
Cognito 전체 인증 흐름
================================

[모바일/웹 앱]
     |
     | 1. 로그인
     v
[User Pool]
     |
     | 2. JWT 토큰 반환
     v
[앱이 JWT 보유]
     |
     +-- API 호출 시: JWT → API Gateway Authorizer
     |
     +-- AWS 리소스 직접 접근 시:
         JWT → [Identity Pool] → IAM 임시 자격 증명
                                       |
                      +----------------+----------------+
                      |                |                |
                   [S3]           [DynamoDB]        [SQS]
```

---

## ⭐ 핵심 포인트

1. ⭐ **User Pool**: 사용자 인증, JWT 토큰 발급, API Gateway 통합
2. ⭐ **Identity Pool**: JWT → IAM 자격 증명 교환, AWS 리소스 직접 접근
3. ⭐ **Lambda 트리거**: 인증 전후 커스텀 로직 실행
4. ⭐ **소셜 로그인**: Google, Facebook, Apple 통합 지원
5. ⭐ **게스트 접근**: Identity Pool에서 인증되지 않은 역할 지원

---

## 📝 연습 문제

**문제 1.** Cognito User Pool의 역할은?

A) AWS 리소스에 IAM 권한 부여  
B) 사용자 회원가입/로그인 처리, JWT 토큰 발급  
C) VPC 네트워크 관리  
D) 데이터베이스 인증  

**정답: B** - User Pool은 사용자 디렉터리로 회원가입/로그인을 처리하고 JWT 토큰을 발급합니다.

---

**문제 2.** Cognito Identity Pool의 역할은?

A) 사용자 회원가입  
B) JWT 토큰을 AWS IAM 임시 자격 증명으로 교환  
C) API Gateway 인증  
D) 소셜 로그인 관리  

**정답: B** - Identity Pool은 토큰을 AWS IAM 임시 자격 증명으로 교환하여 AWS 리소스에 직접 접근할 수 있게 합니다.

---

**문제 3.** 회원가입 전 커스텀 검증 로직을 추가하려면?

A) User Pool 설정 변경  
B) PreSignUp Lambda 트리거 사용  
C) API Gateway Authorizer  
D) IAM 정책 수정  

**정답: B** - PreSignUp Lambda 트리거를 사용하면 회원가입 전에 커스텀 검증 로직을 실행할 수 있습니다.

---

**문제 4.** 모바일 앱에서 S3에 직접 접근하려면 어떤 Cognito 기능을 사용해야 하는가?

A) User Pool  
B) Identity Pool  
C) Lambda 트리거  
D) MFA  

**정답: B** - Identity Pool을 통해 IAM 임시 자격 증명을 받아 S3에 직접 접근할 수 있습니다.

---

**문제 5.** Cognito User Pool이 API Gateway와 통합되는 방식은?

A) API Gateway가 User Pool에 직접 쿼리  
B) Cognito Authorizer를 사용하여 JWT 토큰 자동 검증  
C) Lambda가 중간에서 토큰 검증  
D) IAM 역할로 인증  

**정답: B** - API Gateway에 Cognito Authorizer를 설정하면 JWT 토큰이 자동으로 검증됩니다.

---

## 📌 오늘의 요약

1. User Pool: 사용자 인증 디렉터리, JWT 발급, MFA, 소셜 로그인
2. Identity Pool: JWT → IAM 임시 자격 증명, AWS 리소스 직접 접근
3. Lambda 트리거: 인증 전후 커스텀 로직, 이메일 커스터마이징
4. API Gateway 통합: Cognito Authorizer로 JWT 자동 검증
5. 게스트 접근: Identity Pool에서 인증되지 않은 사용자 역할 설정 가능
