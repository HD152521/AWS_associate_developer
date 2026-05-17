# Day 23 - S3 보안: 버킷 정책, ACL, 암호화

📅 날짜: 2026년 6월 16일 (화요일)  
🎯 주제: S3 보안  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- S3 버킷 정책, ACL, 퍼블릭 액세스 차단을 이해한다
- S3 암호화 방식(SSE-S3, SSE-KMS, SSE-C)을 구분한다
- HTTP 강제 차단 정책을 작성한다

---

## 📖 이론 내용

### 1. S3 접근 제어

#### 버킷 정책 (Bucket Policy)
JSON 형식 리소스 기반 정책, 교차 계정 접근 허용 가능:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "StringEquals": {"aws:PrincipalOrgID": "o-xxxx"}
      }
    }
  ]
}
```

#### ACL (Access Control List)
- 레거시 방식, 현재 권장하지 않음
- 객체 수준 적용 가능

#### ⭐ 퍼블릭 액세스 차단 (Block Public Access)
- 계정/버킷 수준 설정
- **버킷 정책의 퍼블릭 허용도 무시 (최우선 적용)**

### 2. S3 암호화 방식

#### SSE-S3 (S3 관리형 키)
- AWS가 키를 완전 관리
- 헤더: `x-amz-server-side-encryption: AES256`
- **가장 간단, 기본값, 추가 비용 없음**

#### SSE-KMS (KMS 관리형 키)
- AWS KMS로 키 관리
- 키 사용 **감사 로그** (CloudTrail)
- 헤더: `x-amz-server-side-encryption: aws:kms`
- **⭐ KMS 호출 한도**: 초당 5,500~30,000 요청 (병목 가능)

#### SSE-C (고객 제공 키)
- 고객이 키 제공, AWS가 암호화
- 키는 AWS에 저장되지 않음
- **HTTPS 필수** (키 전송 보안)
- 매 요청마다 키 제공 필요

#### 클라이언트 사이드 암호화
- 클라이언트가 암호화 후 업로드
- AWS는 암호화된 데이터만 저장

#### 전송 중 암호화 강제
```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": ["arn:aws:s3:::bucket", "arn:aws:s3:::bucket/*"],
  "Condition": {"Bool": {"aws:SecureTransport": "false"}}
}
```

---

## 아키텍처 다이어그램

```
S3 암호화 비교
================================

[SSE-S3]
데이터 --> S3 서비스 --> AES256 암호화 --> 저장
              AWS가 키 완전 관리

[SSE-KMS]
데이터 --> S3 --> KMS --> 데이터 키 생성 --> 암호화 --> 저장
                  |
                  CloudTrail 감사 로그!

[SSE-C]
데이터+키 --> S3 암호화 --> 저장 (키 저장 안 함)
HTTPS 필수!

퍼블릭 액세스 제어 계층
================================

계정 수준 퍼블릭 액세스 차단 (최우선!)
        |
버킷 수준 퍼블릭 액세스 차단
        |
버킷 정책 / ACL / IAM 정책
```

---

## ⭐ 핵심 포인트

1. ⭐ **SSE-KMS KMS 한도**: 고성능 환경에서 병목 가능
2. ⭐ **SSE-C**: HTTPS 필수, 매 요청마다 키 제공
3. ⭐ **퍼블릭 액세스 차단 우선**: 버킷 정책 허용도 무시
4. ⭐ **기본 암호화**: 버킷 기본 암호화로 새 객체 자동 암호화
5. ⭐ **HTTPS 강제**: aws:SecureTransport false 조건에 Deny

---

## 📝 연습 문제

**문제 1.** SSE-KMS 성능 문제의 원인은?

A) 데이터 크기 제한  
B) KMS API 호출 한도 초과  
C) 리전 제한  
D) 네트워크 대역폭  

**정답: B** - SSE-KMS는 암호화/복호화마다 KMS API를 호출하며 초당 한도가 있어 고성능 환경에서 병목이 될 수 있습니다.

---

**문제 2.** SSE-C의 필수 요구 사항은?

A) KMS 키 생성  
B) HTTPS 사용  
C) 버킷 버전 관리  
D) CloudTrail 활성화  

**정답: B** - SSE-C에서 고객이 매 요청 키를 제공하므로 키를 안전하게 전송하기 위해 HTTPS가 필수입니다.

---

**문제 3.** 퍼블릭 액세스 차단 활성화 시 버킷 정책의 퍼블릭 허용은?

A) 버킷 정책이 우선  
B) 퍼블릭 액세스 차단이 우선, 버킷 정책 허용 무시  
C) 두 설정 병합  
D) 오류 발생  

**정답: B** - 퍼블릭 액세스 차단이 버킷 정책보다 우선 적용됩니다.

---

**문제 4.** 키 사용 감사 로그가 필요할 때 사용하는 암호화 방식은?

A) SSE-S3  
B) SSE-KMS  
C) SSE-C  
D) 클라이언트 사이드  

**정답: B** - SSE-KMS는 KMS 키 사용 내역이 CloudTrail에 자동 기록됩니다.

---

**문제 5.** HTTP S3 요청을 차단하는 조건은?

A) aws:SecureTransport: true 에서 Allow  
B) aws:SecureTransport: false 에서 Deny  
C) s3:HttpOnly: true 에서 Allow  
D) aws:Protocol: https 에서 Allow  

**정답: B** - aws:SecureTransport가 false(HTTP)인 요청에 Deny를 설정하면 HTTP를 차단합니다.

---

## 📌 오늘의 요약

1. 버킷 정책: JSON 리소스 기반 정책, 교차 계정 지원, ACL은 레거시
2. 퍼블릭 액세스 차단이 버킷 정책보다 우선 적용됨
3. SSE-S3(간단), SSE-KMS(감사 로그, KMS 한도), SSE-C(고객 키, HTTPS 필수)
4. SSE-KMS의 KMS API 호출 한도가 고성능 환경에서 병목 가능
5. HTTPS 강제: aws:SecureTransport false 조건에 Deny 적용
