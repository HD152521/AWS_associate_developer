# Day 41 - KMS: 키 관리 서비스

📅 날짜: 2026년 7월 12일 (일요일)  
🎯 주제: AWS Key Management Service  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- KMS의 키 유형과 키 관리 방법을 이해한다
- Envelope Encryption 개념을 파악한다
- KMS API 호출 패턴을 학습한다

---

## 📖 이론 내용

### 1. AWS KMS란?

AWS 서비스의 암호화 키를 생성하고 관리하는 완전 관리형 서비스입니다.

**KMS 키 유형:**
- **AWS Managed Key**: AWS 서비스가 자동 생성/관리, 무료 (예: aws/s3, aws/rds)
- **Customer Managed Key (CMK)**: 고객이 생성/관리, 월 $1
- **AWS Owned Key**: AWS가 소유, 고객에게 보이지 않음

**키 사양:**
- 대칭: AES-256 (가장 일반적)
- 비대칭: RSA, ECC (디지털 서명 등)

### 2. KMS 핵심 API

```python
import boto3

kms = boto3.client('kms')

# 데이터 암호화 (최대 4KB)
response = kms.encrypt(
    KeyId='arn:aws:kms:ap-northeast-2:123456789:key/key-id',
    Plaintext=b'Hello World'
)
ciphertext = response['CiphertextBlob']

# 데이터 복호화
response = kms.decrypt(
    CiphertextBlob=ciphertext
)
plaintext = response['Plaintext']

# 데이터 키 생성 (Envelope Encryption)
response = kms.generate_data_key(
    KeyId='arn:aws:kms:ap-northeast-2:123456789:key/key-id',
    KeySpec='AES_256'
)
plaintext_key = response['Plaintext']      # 암호화에 사용 후 메모리에서 삭제
encrypted_key = response['CiphertextBlob'] # 저장
```

### 3. Envelope Encryption (봉투 암호화)

KMS의 4KB 제한을 넘는 데이터 암호화 방법:

```
봉투 암호화 흐름
================================

[암호화]
1. KMS에서 데이터 키 생성 (GenerateDataKey)
   → 평문 데이터 키 + 암호화된 데이터 키 반환
2. 평문 데이터 키로 실제 데이터 암호화 (로컬)
3. 평문 데이터 키 메모리에서 삭제
4. 암호화된 데이터 + 암호화된 데이터 키 저장

[복호화]
1. 암호화된 데이터 키를 KMS로 복호화
2. 복호화된 데이터 키로 실제 데이터 복호화 (로컬)
3. 데이터 키 삭제
```

**장점:**
- 대용량 데이터 암호화 가능
- KMS API 호출 최소화 (비용 절감)
- 네트워크 트래픽 감소

### 4. KMS 키 정책

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow Lambda to use key",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789:role/LambdaRole"
      },
      "Action": [
        "kms:Decrypt",
        "kms:GenerateDataKey"
      ],
      "Resource": "*"
    }
  ]
}
```

### 5. KMS 로테이션

```bash
# 자동 키 로테이션 활성화 (1년마다)
aws kms enable-key-rotation --key-id key-id

# 수동 로테이션 확인
aws kms get-key-rotation-status --key-id key-id
```

**AWS Managed Key**: 자동 로테이션 (3년마다)  
**Customer Managed Key**: 수동 또는 자동 로테이션 설정 가능

---

## 아키텍처 다이어그램

```
KMS Envelope Encryption 흐름
================================

[애플리케이션]
     |
     | 1. GenerateDataKey 요청
     v
[KMS]
     |
     | 2. 평문 DEK + 암호화된 DEK 반환
     v
[애플리케이션]
     |
     | 3. 평문 DEK로 데이터 암호화 (로컬)
     |    평문 DEK 메모리에서 삭제
     |
     | 4. 암호화된 데이터 + 암호화된 DEK 저장
     v
[S3 / DynamoDB / 기타 스토리지]

복호화 시:
  암호화된 DEK → KMS Decrypt → 평문 DEK → 데이터 복호화
```

---

## ⭐ 핵심 포인트

1. ⭐ **KMS 직접 암호화**: 최대 4KB 제한
2. ⭐ **Envelope Encryption**: 4KB 초과 데이터, GenerateDataKey 사용
3. ⭐ **CMK**: 고객 관리 키, 키 정책으로 세밀한 제어
4. ⭐ **키 로테이션**: AWS Managed는 자동(3년), CMK는 설정 가능(1년)
5. ⭐ **키 정책 필수**: KMS 키에 IAM 정책만으로는 접근 불가, 키 정책 필요

---

## 📝 연습 문제

**문제 1.** 10MB 파일을 KMS로 암호화하는 올바른 방법은?

A) kms:Encrypt API 직접 사용  
B) Envelope Encryption (GenerateDataKey)  
C) S3 서버 측 암호화  
D) 불가능  

**정답: B** - KMS 직접 암호화는 최대 4KB까지만 가능합니다. 10MB 파일은 Envelope Encryption을 사용해야 합니다.

---

**문제 2.** Envelope Encryption에서 평문 데이터 키는 언제 삭제해야 하는가?

A) 데이터를 S3에 저장한 후  
B) 데이터 암호화 직후 메모리에서 삭제  
C) 애플리케이션 종료 시  
D) 삭제할 필요 없음  

**정답: B** - 평문 데이터 키는 데이터를 암호화한 후 즉시 메모리에서 삭제해야 합니다.

---

**문제 3.** Customer Managed Key의 특징은?

A) 무료, AWS가 자동 관리  
B) 월 $1, 고객이 생성/관리, 키 정책 제어  
C) 완전히 사용자 하드웨어에 저장  
D) 자동으로 매월 로테이션  

**정답: B** - CMK는 월 $1이며 고객이 생성/관리하고 키 정책으로 세밀하게 제어할 수 있습니다.

---

**문제 4.** KMS 키에 접근하기 위한 필수 조건은?

A) IAM 정책만으로 충분  
B) 키 정책 + IAM 정책 모두 필요  
C) 키 정책만으로 충분  
D) VPC 엔드포인트 필요  

**정답: B** - KMS 키에 접근하려면 키 정책에서 허용하고 IAM 정책에서도 허용해야 합니다.

---

**문제 5.** AWS Managed Key의 자동 로테이션 주기는?

A) 1년  
B) 2년  
C) 3년  
D) 5년  

**정답: C** - AWS Managed Key는 3년마다 자동으로 로테이션됩니다.

---

## 📌 오늘의 요약

1. KMS: 완전 관리형 키 관리, AWS Managed / Customer Managed 키
2. 직접 암호화: 최대 4KB, KMS API 직접 호출
3. Envelope Encryption: 4KB 초과 데이터, 데이터 키로 로컬 암호화
4. 키 정책: KMS 접근에 필수, IAM 정책과 함께 사용
5. 키 로테이션: AWS Managed 3년 자동, CMK 1년 자동 설정 가능
