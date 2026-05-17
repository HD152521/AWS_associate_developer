# Day 57 - CloudFormation: 인프라 as 코드

📅 날짜: 2026년 8월 3일 (월요일)  
🎯 주제: AWS CloudFormation  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- CloudFormation 템플릿 구조를 이해한다
- 스택 생성과 업데이트 전략을 학습한다
- CloudFormation 핵심 기능을 파악한다

---

## 📖 이론 내용

### 1. AWS CloudFormation이란?

인프라를 코드로 정의하고 자동으로 프로비저닝하는 IaC(Infrastructure as Code) 서비스입니다.

**장점:**
- 인프라 버전 관리 가능
- 동일한 환경 반복 생성
- 변경 사항 추적
- 드리프트 감지

### 2. CloudFormation 템플릿 구조

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: My Application Stack

# 입력 파라미터
Parameters:
  EnvironmentName:
    Type: String
    Default: production
    AllowedValues: [production, staging, development]
  
  InstanceType:
    Type: String
    Default: t3.micro

# 조건
Conditions:
  IsProduction: !Equals [!Ref EnvironmentName, production]

# 매핑 (조회 테이블)
Mappings:
  RegionAMI:
    ap-northeast-2:
      AMI: ami-0c9c942bd7bf113a2
    us-east-1:
      AMI: ami-0abcdef1234567890

# 리소스 (필수)
Resources:
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-vpc"
  
  MyEC2:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: !FindInMap [RegionAMI, !Ref AWS::Region, AMI]
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-server"
  
  MyLambda:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: !Sub "${EnvironmentName}-function"
      Runtime: python3.9
      Handler: index.handler
      Role: !GetAtt LambdaRole.Arn
      Code:
        ZipFile: |
          def handler(event, context):
              return {'statusCode': 200}

# 출력값
Outputs:
  VPCId:
    Value: !Ref MyVPC
    Export:
      Name: !Sub "${EnvironmentName}-vpc-id"
  
  LambdaArn:
    Value: !GetAtt MyLambda.Arn
    Description: Lambda 함수 ARN
```

### 3. CloudFormation 주요 함수

```yaml
# !Ref - 파라미터 또는 리소스 참조
InstanceType: !Ref InstanceTypeParam

# !GetAtt - 리소스 속성 참조
RoleArn: !GetAtt MyRole.Arn

# !Sub - 문자열 치환
Name: !Sub "${EnvironmentName}-${AWS::Region}-stack"

# !FindInMap - 매핑 조회
AMI: !FindInMap [RegionMap, !Ref AWS::Region, AMI]

# !If - 조건 분기
InstanceType: !If [IsProduction, m5.large, t3.micro]

# !Join - 문자열 연결
CIDR: !Join ['', ['10.', !Ref ThirdOctet, '.0.0/16']]

# !Select - 목록에서 선택
SubnetId: !Select [0, !GetAZs '']

# !ImportValue - 다른 스택 출력 참조
VPCId: !ImportValue prod-vpc-id
```

### 4. Change Sets (변경 세트)

스택 업데이트 전 변경 사항을 미리 검토합니다:

```bash
# 변경 세트 생성
aws cloudformation create-change-set \
    --stack-name my-stack \
    --template-body file://template.yaml \
    --change-set-name my-changes \
    --parameters ParameterKey=EnvironmentName,ParameterValue=production

# 변경 세트 검토
aws cloudformation describe-change-set \
    --stack-name my-stack \
    --change-set-name my-changes

# 변경 세트 실행 (스택 업데이트)
aws cloudformation execute-change-set \
    --stack-name my-stack \
    --change-set-name my-changes
```

### 5. 스택 정책 (Stack Policy)

중요한 리소스의 삭제/교체를 방지합니다:

```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "Update:*",
      "Principal": "*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": ["Update:Replace", "Update:Delete"],
      "Principal": "*",
      "Resource": "LogicalResourceId/ProductionDB"
    }
  ]
}
```

---

## 아키텍처 다이어그램

```
CloudFormation IaC 워크플로우
================================

[개발자]
  template.yaml 작성
     |
     v
[CloudFormation]
     |
     | 스택 생성/업데이트
     v
[AWS 리소스 자동 프로비저닝]
  VPC, EC2, RDS, Lambda,
  ALB, Security Group...
  
변경 사항 관리:
[현재 스택] + [새 템플릿]
     |
     v
[Change Set]
  - 추가될 리소스
  - 삭제될 리소스
  - 수정될 리소스 (교체/수정)
  (검토 후 실행)

크로스 스택 참조
================================

[네트워크 스택]
  Outputs:
    VPCId (Export)
    SubnetId (Export)
        |
        | !ImportValue
        v
[애플리케이션 스택]
  EC2, ALB, RDS...
```

---

## ⭐ 핵심 포인트

1. ⭐ **Change Set**: 업데이트 전 변경 내용 검토, 프로덕션 배포 안전
2. ⭐ **!Sub**: 문자열 치환, 환경별 동적 이름 생성
3. ⭐ **!ImportValue**: 다른 스택의 출력값 참조 (크로스 스택)
4. ⭐ **스택 정책**: 중요 리소스 삭제/교체 방지
5. ⭐ **드리프트 감지**: 수동 변경 후 템플릿과 차이 감지

---

## 📝 연습 문제

**문제 1.** CloudFormation에서 환경에 따라 인스턴스 타입을 다르게 설정하려면?

A) 템플릿을 환경별로 복사  
B) Parameters + Conditions + !If 사용  
C) 하드코딩  
D) 환경 변수 사용  

**정답: B** - Parameters로 환경을 입력받고, Conditions로 조건을 정의하고, !If 함수로 값을 분기합니다.

---

**문제 2.** 프로덕션 스택 업데이트 전에 변경 사항을 안전하게 검토하는 방법은?

A) 바로 스택 업데이트  
B) Change Set 생성 후 검토  
C) 스냅샷 생성  
D) 스택 복사  

**정답: B** - Change Set을 생성하면 실제 업데이트 전에 추가/삭제/수정될 리소스를 확인할 수 있습니다.

---

**문제 3.** CloudFormation에서 다른 스택의 출력값을 참조하는 함수는?

A) !Ref  
B) !GetAtt  
C) !ImportValue  
D) !FindInMap  

**정답: C** - !ImportValue 함수로 다른 스택에서 Export한 출력값을 참조합니다.

---

**문제 4.** CloudFormation에서 RDS 인스턴스의 엔드포인트 주소를 가져오는 방법은?

A) !Ref MyRDS  
B) !GetAtt MyRDS.Endpoint.Address  
C) !Sub "${MyRDS}"  
D) !ImportValue MyRDS  

**정답: B** - !GetAtt 함수로 리소스의 특정 속성(Endpoint.Address)을 가져올 수 있습니다.

---

**문제 5.** CloudFormation 스택에서 중요한 RDS 리소스를 실수로 삭제하는 것을 방지하려면?

A) S3 버전 관리 활성화  
B) 스택 정책(Stack Policy) 설정  
C) IAM 권한 제한  
D) 스택 종료 방지만으로 충분  

**정답: B** - 스택 정책을 설정하면 특정 리소스의 삭제나 교체를 명시적으로 거부할 수 있습니다.

---

## 📌 오늘의 요약

1. CloudFormation: IaC, 템플릿으로 인프라 정의 및 자동 프로비저닝
2. 템플릿 구조: Parameters, Conditions, Mappings, Resources, Outputs
3. 주요 함수: !Ref, !GetAtt, !Sub, !FindInMap, !If, !ImportValue
4. Change Set: 업데이트 전 변경 내용 검토, 안전한 프로덕션 배포
5. 스택 정책: 중요 리소스 삭제/교체 방지, 드리프트 감지
