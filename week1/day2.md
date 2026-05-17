# Day 2 - IAM: 사용자, 그룹, 역할, 정책

📅 날짜: 2026년 5월 18일 (월요일)  
🎯 주제: AWS Identity and Access Management (IAM)  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- IAM의 핵심 구성 요소(사용자, 그룹, 역할, 정책)를 이해한다
- IAM 정책의 구조와 작동 방식을 설명할 수 있다
- 최소 권한 원칙을 이해하고 적용할 수 있다

---

## 📖 이론 내용

### 1. IAM이란?

IAM(Identity and Access Management)은 AWS 리소스에 대한 접근을 안전하게 제어하는 서비스입니다. IAM을 통해 누가(인증, Authentication) AWS 리소스에 어떤 작업(인가, Authorization)을 할 수 있는지 관리합니다.

**IAM의 핵심 원칙: 최소 권한 원칙 (Principle of Least Privilege)**
필요한 작업을 수행하는 데 필요한 최소한의 권한만 부여합니다.

### 2. IAM 구성 요소

#### 루트 계정 (Root Account)
- AWS 계정 생성 시 자동으로 만들어지는 최고 관리자 계정
- **⭐ 일상적인 작업에는 절대 사용하지 말 것!**
- MFA(다중 인증) 반드시 활성화
- 루트 계정만 할 수 있는 작업: 계정 해지, 지원 플랜 변경, 빌링 정보 변경

#### IAM 사용자 (User)
- AWS 서비스와 상호 작용하는 개인 또는 애플리케이션
- 이름과 자격 증명(비밀번호/액세스 키)을 가짐
- 콘솔 접근: 사용자 이름 + 비밀번호
- 프로그래밍 방식 접근: 액세스 키 ID + 시크릿 액세스 키

#### IAM 그룹 (Group)
- IAM 사용자들의 컬렉션
- 그룹에 정책을 연결하면 그룹의 모든 사용자에게 적용
- 그룹 안에 그룹은 불가능 (중첩 그룹 미지원)
- 하나의 사용자는 여러 그룹에 속할 수 있음

#### IAM 역할 (Role)
- 특정 권한을 가진 임시 자격 증명
- 사용자, EC2 인스턴스, AWS 서비스 등이 역할을 "수임(assume)"
- 장기 자격 증명 없이 임시 보안 자격 증명 제공
- **사용 사례**: EC2가 S3에 접근, Lambda가 DynamoDB에 접근

#### IAM 정책 (Policy)
- JSON 형식으로 권한을 정의하는 문서
- 어떤 AWS 서비스의 어떤 리소스에 어떤 작업을 허용/거부할지 정의

### 3. IAM 정책 구조

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadAccess",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ],
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": "ap-northeast-2"
        }
      }
    }
  ]
}
```

**정책 요소 설명:**
- **Version**: 정책 언어 버전 ("2012-10-17" 사용)
- **Statement**: 권한 문장의 배열
- **Sid**: 문장 식별자 (선택 사항)
- **Effect**: Allow(허용) 또는 Deny(거부)
- **Action**: 허용/거부할 AWS 서비스 작업
- **Resource**: 작업이 적용되는 AWS 리소스 ARN
- **Condition**: 정책이 적용되는 조건 (선택 사항)

### 4. 정책 유형

| 유형 | 설명 |
|------|------|
| AWS 관리형 정책 | AWS가 생성하고 관리하는 정책 |
| 고객 관리형 정책 | 고객이 직접 생성하고 관리하는 정책 |
| 인라인 정책 | 사용자/그룹/역할에 직접 포함된 정책 |

### 5. 권한 평가 방법

**⭐ 시험 핵심**: 명시적 Deny > Deny(기본값) > Allow
1. 모든 요청은 기본적으로 거부 (암묵적 거부)
2. Allow 정책이 있으면 허용
3. 명시적 Deny가 있으면 무조건 거부 (Allow보다 우선)

---

## 아키텍처 다이어그램

```
IAM 구성 요소 관계도
================================

AWS 계정
+----------------------------------------------------------+
|                                                          |
|  루트 계정 (최고 관리자 - 일상 사용 금지!)              |
|                                                          |
|  IAM 사용자들          IAM 그룹들                       |
|  +----------+          +-----------+                    |
|  | Alice    |--------->| Admins    |<----[AdPolicy]     |
|  +----------+    |     +-----------+                    |
|  | Bob      |----+     +-----------+                    |
|  +----------+    +---->| Developers|<----[DevPolicy]    |
|  | Charlie  |          +-----------+                    |
|  +----------+                                           |
|                                                         |
|  IAM 역할                IAM 정책                      |
|  +----------+             +-------------+              |
|  | EC2Role  |<--[Assume]--| EC2Instance |              |
|  +----+-----+             +-------------+              |
|       |                                                 |
|       v                                                 |
|  [S3ReadPolicy]                                        |
|  Effect: Allow                                         |
|  Action: s3:GetObject                                  |
|  Resource: arn:aws:s3:::*                              |
|                                                         |
+----------------------------------------------------------+

권한 평가 흐름
================================

요청 수신
    |
    v
명시적 Deny 있음? --> YES --> 거부 (무조건)
    |
    NO
    v
Allow 정책 있음? --> NO --> 거부 (암묵적)
    |
    YES
    v
허용
```

---

## ⭐ 핵심 포인트 (시험 출제 빈도 높음)

1. ⭐ **루트 계정**: 일상 작업에 절대 사용 금지, MFA 필수 활성화
2. ⭐ **IAM 역할**: 임시 자격 증명 제공, EC2/Lambda 등 서비스가 다른 서비스에 접근할 때 사용
3. ⭐ **그룹 중첩 불가**: IAM 그룹 안에 다른 그룹을 넣을 수 없음
4. ⭐ **명시적 Deny 우선**: Allow보다 Deny가 항상 우선 적용
5. ⭐ **정책 Version**: "2012-10-17" 사용 (최신 버전이며 조건 키 지원)

---

## 💻 실제 예시 - IAM 관련 CLI 명령어

```bash
# IAM 사용자 생성
aws iam create-user --user-name developer01

# IAM 그룹 생성
aws iam create-group --group-name Developers

# 사용자를 그룹에 추가
aws iam add-user-to-group \
  --user-name developer01 \
  --group-name Developers

# 정책을 그룹에 연결
aws iam attach-group-policy \
  --group-name Developers \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# IAM 역할 생성 (EC2 용)
aws iam create-role \
  --role-name EC2S3AccessRole \
  --assume-role-policy-document file://trust-policy.json

# MFA 디바이스 목록 확인
aws iam list-mfa-devices

# 현재 사용자 정보 확인
aws sts get-caller-identity
```

**trust-policy.json (EC2 역할의 신뢰 정책):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

---

## 📝 연습 문제

**문제 1.** IAM 그룹에 대한 설명으로 올바른 것은?

A) 그룹 안에 다른 그룹을 포함시킬 수 있다  
B) 그룹에는 IAM 사용자만 포함될 수 있다  
C) 하나의 사용자는 하나의 그룹에만 속할 수 있다  
D) 그룹은 AWS 서비스에 직접 로그인할 수 있다  

**정답: B**  
해설: IAM 그룹에는 IAM 사용자만 포함될 수 있으며, 그룹 중첩(그룹 안에 그룹)은 지원되지 않습니다. 하나의 사용자는 여러 그룹에 속할 수 있습니다.

---

**문제 2.** 다음 IAM 정책에서 S3 버킷의 객체를 삭제하는 요청이 들어왔을 때 결과는?

```json
{
  "Statement": [
    {"Effect": "Allow", "Action": "s3:*", "Resource": "*"},
    {"Effect": "Deny", "Action": "s3:DeleteObject", "Resource": "*"}
  ]
}
```

A) 허용된다 (Allow가 Deny보다 먼저 정의되어 있으므로)  
B) 거부된다 (명시적 Deny가 Allow보다 우선)  
C) 오류가 발생한다  
D) 관리자에게 승인을 요청한다  

**정답: B**  
해설: IAM 정책 평가에서 명시적 Deny는 항상 Allow보다 우선합니다. 정책 순서와 관계없이 Deny가 적용됩니다.

---

**문제 3.** EC2 인스턴스에서 S3 버킷에 접근하는 가장 안전한 방법은?

A) EC2 인스턴스에 AWS 자격 증명을 직접 저장  
B) 환경 변수에 액세스 키 설정  
C) IAM 역할을 EC2 인스턴스에 연결  
D) 모든 EC2 인스턴스에 동일한 IAM 사용자 자격 증명 공유  

**정답: C**  
해설: IAM 역할을 EC2에 연결하면 임시 자격 증명이 자동으로 제공되며, 별도로 자격 증명을 저장하거나 관리할 필요가 없습니다.

---

**문제 4.** 루트 계정 보안 강화를 위해 해야 할 첫 번째 조치는?

A) 루트 계정의 비밀번호를 주기적으로 변경  
B) 루트 계정에 MFA(다중 인증) 활성화  
C) 루트 계정을 여러 관리자와 공유  
D) 루트 계정으로 모든 일상적인 작업 수행  

**정답: B**  
해설: 루트 계정에는 반드시 MFA를 활성화해야 합니다. 루트 계정은 일상적인 작업에 사용해서는 안 되며, 반드시 필요한 작업에만 사용해야 합니다.

---

**문제 5.** IAM 정책의 JSON 구조에서 "Effect" 필드의 올바른 값은?

A) True 또는 False  
B) Grant 또는 Revoke  
C) Allow 또는 Deny  
D) Permit 또는 Block  

**정답: C**  
해설: IAM 정책의 Effect 필드는 "Allow"(허용) 또는 "Deny"(거부) 두 가지 값만 사용할 수 있습니다.

---

## 📌 오늘의 요약

1. IAM은 AWS 리소스에 대한 인증(누구인가)과 인가(무엇을 할 수 있는가)를 관리한다
2. IAM 구성 요소: 루트 계정, 사용자, 그룹(중첩 불가), 역할, 정책
3. IAM 정책은 JSON 형식으로 Effect, Action, Resource, Condition을 정의한다
4. 권한 평가 순서: 명시적 Deny > 암묵적 Deny > Allow (Deny가 항상 우선)
5. EC2 등 서비스가 다른 AWS 서비스에 접근할 때는 IAM 역할(임시 자격 증명)을 사용한다
