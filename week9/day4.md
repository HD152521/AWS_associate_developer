# Day 44 - WAF, Shield, ACM

📅 날짜: 2026년 7월 15일 (수요일)  
🎯 주제: AWS 보안 서비스  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- WAF로 웹 애플리케이션 공격을 방어한다
- Shield Standard와 Advanced의 차이를 이해한다
- ACM으로 SSL/TLS 인증서를 관리한다

---

## 📖 이론 내용

### 1. AWS WAF (Web Application Firewall)

Layer 7(HTTP/HTTPS) 트래픽을 필터링하는 방화벽입니다.

**WAF 적용 가능 서비스:**
- Application Load Balancer
- Amazon CloudFront
- API Gateway
- AWS AppSync

**WAF 규칙 유형:**
```json
{
  "웹 ACL 규칙 예시": [
    {
      "이름": "IP 차단",
      "유형": "IP Set",
      "조건": "악성 IP 목록에 포함"
    },
    {
      "이름": "SQL 인젝션 방어",
      "유형": "SQL Injection Rule",
      "조건": "쿼리 파라미터에 SQL 패턴 감지"
    },
    {
      "이름": "요청 비율 제한",
      "유형": "Rate-based Rule",
      "조건": "5분간 IP당 2000 요청 초과"
    },
    {
      "이름": "특정 국가 차단",
      "유형": "Geo Match",
      "조건": "특정 국가에서 요청"
    }
  ]
}
```

```bash
# WAF 웹 ACL 생성 (CLI)
aws wafv2 create-web-acl \
    --name MyWebACL \
    --scope REGIONAL \
    --default-action Allow={} \
    --rules file://rules.json \
    --visibility-config SampledRequestsEnabled=true,...
```

### 2. AWS WAF 관리형 규칙

AWS와 파트너사가 미리 작성한 규칙 세트:
- **AWS Managed Rules**: OWASP Top 10, 알려진 악성 IP
- **AWS Marketplace Rules**: 서드파티 보안 업체 규칙

### 3. AWS Shield

**Shield Standard (무료):**
- 모든 AWS 고객에게 자동 적용
- L3/L4 DDoS 공격 방어 (SYN/UDP 플러드)
- EC2, ELB, CloudFront, Route 53에 적용

**Shield Advanced ($3,000/월):**
- 강화된 DDoS 방어
- DDoS 비용 보호 (DDoS 공격으로 인한 AWS 비용 크레딧)
- 24/7 AWS DDoS 대응 팀(DRT) 지원
- 실시간 공격 가시성
- CloudFront, ALB, Route 53, EC2 등 지원

| 특성 | Shield Standard | Shield Advanced |
|------|-----------------|-----------------|
| 비용 | 무료 | $3,000/월 |
| DDoS 보호 | L3/L4 기본 | L3/L4/L7 강화 |
| 비용 보호 | 없음 | DDoS 비용 크레딧 |
| DRT 지원 | 없음 | 24/7 지원 |

### 4. AWS Certificate Manager (ACM)

SSL/TLS 인증서를 무료로 생성하고 관리하는 서비스입니다.

```bash
# 퍼블릭 인증서 요청
aws acm request-certificate \
    --domain-name example.com \
    --subject-alternative-names www.example.com \
    --validation-method DNS

# 인증서 목록 조회
aws acm list-certificates

# 인증서 세부 정보
aws acm describe-certificate \
    --certificate-arn arn:aws:acm:...
```

**ACM 사용 가능 서비스:**
- CloudFront (us-east-1 리전 인증서만)
- ALB / NLB
- API Gateway
- Elastic Beanstalk

**⭐ 중요**: ACM 인증서는 AWS 서비스에만 사용 가능 (EC2에 직접 설치 불가)

---

## 아키텍처 다이어그램

```
WAF + Shield + ACM 아키텍처
================================

[인터넷]
     |
     | DDoS 공격 → [Shield Advanced] 방어
     |
     v
[CloudFront] → ACM 인증서 (HTTPS)
     |
     | SQL 인젝션, XSS, 악성 IP → [WAF] 차단
     v
[ALB] → ACM 인증서 (HTTPS)
     |
     v
[EC2 / Lambda / ECS]

WAF 요청 처리 흐름
================================

요청 수신
   |
   +-- IP 차단 규칙? → 차단
   |
   +-- 요청 비율 초과? → 차단
   |
   +-- SQL 인젝션 패턴? → 차단
   |
   +-- Geo 차단 국가? → 차단
   |
   모든 규칙 통과 → 정상 처리
```

---

## ⭐ 핵심 포인트

1. ⭐ **WAF**: Layer 7 방화벽, ALB/CloudFront/API GW에 적용
2. ⭐ **WAF 규칙**: IP 차단, SQL 인젝션, Rate Limiting, Geo 차단
3. ⭐ **Shield Standard**: 무료, L3/L4 DDoS 방어, 자동 적용
4. ⭐ **Shield Advanced**: $3,000/월, L7까지 방어, 비용 보호, DRT 지원
5. ⭐ **ACM**: 무료 SSL/TLS, EC2에 직접 설치 불가, CloudFront는 us-east-1만

---

## 📝 연습 문제

**문제 1.** SQL 인젝션 공격을 방어하려면?

A) Shield Advanced  
B) AWS WAF  
C) NACLs  
D) Security Group  

**정답: B** - WAF는 Layer 7(HTTP) 트래픽을 분석하여 SQL 인젝션 패턴을 감지하고 차단합니다.

---

**문제 2.** DDoS 공격으로 인한 AWS 비용이 급증했을 때 보호받으려면?

A) Shield Standard  
B) Shield Advanced  
C) WAF  
D) CloudFront  

**정답: B** - Shield Advanced는 DDoS 공격으로 인한 AWS 비용에 대해 크레딧을 제공합니다.

---

**문제 3.** ACM 인증서를 CloudFront에 사용하려면?

A) 아무 리전에서나 생성  
B) ap-northeast-2 리전에서 생성  
C) us-east-1 리전에서 생성  
D) CloudFront 자체에서 생성  

**정답: C** - CloudFront에 사용하는 ACM 인증서는 반드시 us-east-1 리전에서 생성해야 합니다.

---

**문제 4.** WAF Rate-based Rule의 용도는?

A) SQL 인젝션 방어  
B) IP 주소 차단  
C) 과도한 요청(브루트포스, DoS) 방어  
D) 크로스사이트 스크립팅 방어  

**정답: C** - Rate-based Rule은 특정 IP에서 일정 시간 내 요청 수를 제한하여 브루트포스나 DoS 공격을 방어합니다.

---

**문제 5.** EC2 인스턴스에 직접 SSL 인증서를 설치하려면?

A) ACM 사용  
B) ACM은 불가, 직접 인증서 구매 후 설치  
C) Shield로 SSL 설정  
D) IAM 인증서 사용  

**정답: B** - ACM 인증서는 AWS 관리형 서비스(ALB, CloudFront 등)에만 사용 가능합니다. EC2에는 직접 인증서를 구매하거나 Let's Encrypt 등을 사용해야 합니다.

---

## 📌 오늘의 요약

1. WAF: Layer 7 방화벽, SQL 인젝션/XSS/IP 차단/Rate Limiting/Geo 차단
2. WAF 적용: ALB, CloudFront, API Gateway, AppSync
3. Shield Standard: 무료, 기본 DDoS 방어, 자동 적용
4. Shield Advanced: $3,000/월, L7까지 방어, 비용 보호, 24/7 DRT
5. ACM: 무료 SSL/TLS, EC2 직접 설치 불가, CloudFront는 us-east-1 필수
