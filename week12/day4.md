# Day 59 - CDK, 아키텍처 패턴

📅 날짜: 2026년 8월 5일 (수요일)  
🎯 주제: AWS CDK 및 서버리스 아키텍처 패턴  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- AWS CDK의 기본 개념과 SAM/CloudFormation과의 차이를 이해한다
- 자주 출제되는 서버리스 아키텍처 패턴을 파악한다
- 실제 시험에 자주 나오는 아키텍처 설계 원칙을 학습한다

---

## 📖 이론 내용

### 1. AWS CDK (Cloud Development Kit)

프로그래밍 언어(TypeScript, Python, Java 등)로 인프라를 정의하는 IaC 프레임워크입니다.

**CDK vs CloudFormation vs SAM:**
- CloudFormation: YAML/JSON 템플릿
- SAM: 서버리스 특화 CloudFormation 확장
- CDK: 프로그래밍 언어로 인프라 정의 → CloudFormation으로 변환

```python
# CDK Python 예시
from aws_cdk import (
    Stack,
    aws_lambda as _lambda,
    aws_apigateway as apigw,
    aws_dynamodb as dynamodb
)
from constructs import Construct

class OrderStack(Stack):
    def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)
        
        # DynamoDB 테이블
        table = dynamodb.Table(
            self, "OrdersTable",
            partition_key=dynamodb.Attribute(
                name="orderId",
                type=dynamodb.AttributeType.STRING
            )
        )
        
        # Lambda 함수
        handler = _lambda.Function(
            self, "CreateOrderHandler",
            runtime=_lambda.Runtime.PYTHON_3_9,
            code=_lambda.Code.from_asset("src"),
            handler="create_order.handler",
            environment={
                "TABLE_NAME": table.table_name
            }
        )
        
        # 권한 부여
        table.grant_read_write_data(handler)
        
        # API Gateway
        api = apigw.RestApi(self, "OrdersApi")
        orders = api.root.add_resource("orders")
        orders.add_method("POST", apigw.LambdaIntegration(handler))
```

```bash
# CDK 명령어
cdk init --language python
cdk synth          # CloudFormation 템플릿 생성
cdk diff           # 변경 사항 확인
cdk deploy         # 배포
cdk destroy        # 삭제
```

### 2. 서버리스 아키텍처 패턴

**패턴 1: API Backend**
```
[클라이언트]
     |
     v
[API Gateway]
     |
     v
[Lambda]
     |
     +-- [DynamoDB] (데이터 저장)
     +-- [ElastiCache] (캐싱)
     +-- [Secrets Manager] (자격 증명)
```

**패턴 2: 이벤트 드리븐 처리**
```
[S3 파일 업로드]
     |
     v
[Lambda]
     |
     +-- 이미지 리사이즈 → S3
     +-- 메타데이터 저장 → DynamoDB
     +-- 알림 전송 → SNS
```

**패턴 3: 마이크로서비스 통신**
```
[Lambda A] → [SQS] → [Lambda B]
            (비동기, 느슨한 결합)

[Lambda A] → [SNS] → [Lambda B]
                   → [Lambda C]
            (팬아웃)
```

### 3. 시험에 자주 나오는 선택 기준

```
REST API: API Gateway REST API
WebSocket API: API Gateway WebSocket
간단한 API (저렴): HTTP API
GraphQL: AppSync
컨테이너: ECS Fargate
서버리스 함수: Lambda
작업 큐: SQS
알림: SNS
스트리밍: Kinesis
워크플로우: Step Functions
```

### 4. 잘못된 아키텍처 vs 올바른 아키텍처

**잘못된 패턴:**
```
[API Gateway] → [Lambda A] → [Lambda B] 동기 호출
(Lambda에서 Lambda를 직접 호출하면 비용이 2배, 오류 전파)
```

**올바른 패턴:**
```
[API Gateway] → [Lambda A] → [SQS] → [Lambda B]
(비동기, 느슨한 결합, 독립적 확장)
```

### 5. DVA-C02 시험 전략

**도메인별 출제 비중:**
```
Development (개발): 32%
  - Lambda, API Gateway, DynamoDB
  - SDK, CLI, CloudFormation

Security (보안): 26%
  - IAM, KMS, Cognito, Secrets Manager
  - 암호화, 인증/인가

Deployment (배포): 24%
  - CI/CD, CodePipeline, Beanstalk
  - ECS, SAM, CloudFormation

Troubleshooting (문제 해결): 18%
  - CloudWatch, X-Ray, CloudTrail
  - 오류 분석, 성능 최적화
```

---

## 아키텍처 다이어그램

```
완전한 서버리스 주문 처리 시스템
================================

[모바일/웹 앱]
     |
     | JWT (Cognito)
     v
[API Gateway] → [Cognito Authorizer]
     |
     +-- POST /orders → [Lambda: CreateOrder]
     |                       |
     |               [DynamoDB: Orders]
     |               [SQS: 주문 처리 큐]
     |
     +-- GET /orders/{id} → [Lambda: GetOrder]
                                  |
                            [ElastiCache: 캐시]
                            [DynamoDB: 폴백]

SQS → [Lambda: ProcessOrder]
           |
           +-- [SES: 이메일]
           +-- [SNS: 푸시 알림]
           +-- [Step Functions: 복잡한 처리]

모니터링:
  [X-Ray] → 분산 추적
  [CloudWatch] → 지표/알람
  [CloudTrail] → 감사 로그
```

---

## ⭐ 핵심 포인트

1. ⭐ **CDK**: 프로그래밍 언어로 인프라, cdk synth로 CF 변환
2. ⭐ **Lambda 직접 호출**: 피해야 함, SQS/SNS로 느슨한 결합
3. ⭐ **서버리스 장점**: 자동 확장, 사용량 기반 과금, 관리 최소화
4. ⭐ **DVA-C02**: 개발(32%) > 보안(26%) > 배포(24%) > 문제해결(18%)
5. ⭐ **패턴 암기**: 시나리오별 최적 서비스 조합 선택이 핵심

---

## 📝 연습 문제

**문제 1.** AWS CDK의 특징은?

A) YAML 템플릿만 사용  
B) 프로그래밍 언어로 인프라 정의, CloudFormation으로 변환  
C) Terraform과 동일  
D) 서버리스만 지원  

**정답: B** - CDK는 TypeScript, Python 등 프로그래밍 언어로 인프라를 정의하고 CloudFormation 템플릿으로 합성(synth)됩니다.

---

**문제 2.** Lambda 함수에서 다른 Lambda를 동기로 직접 호출하는 것이 나쁜 이유는?

A) 성능 차이가 없음  
B) 비용이 2배, 오류 전파, 결합도 증가  
C) Lambda 권한 부족  
D) 리전 제한  

**정답: B** - 동기 호출은 두 Lambda 모두 실행 시간만큼 과금되고, 하나의 오류가 체인 전체에 영향을 줍니다.

---

**문제 3.** DVA-C02 시험에서 가장 높은 비중의 도메인은?

A) Security (보안)  
B) Deployment (배포)  
C) Development (개발)  
D) Troubleshooting (문제 해결)  

**정답: C** - Development (개발) 도메인이 32%로 가장 높은 비중입니다.

---

**문제 4.** cdk synth 명령의 역할은?

A) CDK 스택 배포  
B) CDK 앱을 CloudFormation 템플릿으로 변환  
C) 스택 삭제  
D) 변경 사항 확인  

**정답: B** - `cdk synth`는 CDK 코드를 CloudFormation 템플릿으로 합성하여 표준 출력으로 보여줍니다.

---

**문제 5.** 대용량 파일 처리를 위한 최적의 서버리스 아키텍처는?

A) API Gateway → Lambda → 직접 처리  
B) S3 → Lambda(이벤트 트리거) → 처리 → 결과 저장  
C) EC2 전용 서버  
D) RDS에 파일 저장  

**정답: B** - S3에 파일을 업로드하면 Lambda 이벤트 트리거로 비동기 처리하여 Lambda 15분 제한 내에서 효율적으로 처리할 수 있습니다.

---

## 📌 오늘의 요약

1. CDK: 프로그래밍 언어 IaC, cdk synth → CloudFormation, cdk deploy → 배포
2. 서버리스 패턴: API Backend, 이벤트 드리븐, 팬아웃 처리
3. 안티패턴: Lambda→Lambda 동기 호출, 대신 SQS/SNS 사용
4. DVA-C02 비중: 개발(32%), 보안(26%), 배포(24%), 문제해결(18%)
5. 시나리오별 서비스 선택: API GW, Lambda, DynamoDB, SQS/SNS, Kinesis
