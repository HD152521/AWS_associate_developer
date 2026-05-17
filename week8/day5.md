# Day 40 - Week 8 복습 + 연습문제 (CI/CD)

📅 날짜: 2026년 7월 9일 (목요일)  
🎯 주제: CI/CD 종합 복습  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- CI/CD 파이프라인의 각 단계를 종합 정리한다
- 실전 시험 유형의 CI/CD 문제를 풀어 실력을 점검한다

---

## 📖 Week 8 핵심 정리

### CI/CD 서비스 역할
```
CodeCommit: 소스 코드 버전 관리 (Git)
CodeBuild: 빌드, 테스트, 아티팩트 생성
CodeDeploy: EC2/Lambda/ECS 배포 자동화
CodePipeline: 전체 파이프라인 오케스트레이션
Elastic Beanstalk: PaaS, 인프라 자동 관리
```

### CodeDeploy 배포 전략
```
EC2:
  AllAtOnce: 동시 배포, 다운타임 있음
  Rolling: 순차 배포, 용량 감소
  OneAtATime: 하나씩, 가장 안전
  
Lambda:
  Canary: 10% 테스트 후 100% 전환
  Linear: 점진적 증가
  AllAtOnce: 즉시 전환

Beanstalk:
  Immutable: 새 ASG 생성, 가장 안전
  Blue/Green: 새 환경 생성, URL 스왑
```

---

## 아키텍처 다이어그램

```
완전한 AWS CI/CD 파이프라인
================================

[개발자] git push
     |
     v
[CodeCommit] → EventBridge 트리거
     |
     v
[CodePipeline]
  Stage 1: Source (CodeCommit)
  Stage 2: Build (CodeBuild)
     - buildspec.yml 실행
     - 단위 테스트
     - 아티팩트 → S3
  Stage 3: Staging Deploy (CodeDeploy)
     - 스테이징 환경 배포
  Stage 4: Manual Approval
     - 관리자 검토/승인
  Stage 5: Prod Deploy (CodeDeploy)
     - 프로덕션 환경 배포
     - Canary 배포 전략
```

---

## 📝 Week 8 종합 연습문제

**문제 1.** buildspec.yml의 단계 순서는?

A) pre_build → install → build → post_build  
B) install → pre_build → build → post_build  
C) build → install → pre_build → post_build  
D) install → build → pre_build → post_build  

**정답: B** - buildspec.yml의 phases 순서는 install → pre_build → build → post_build입니다.

---

**문제 2.** Lambda 배포에서 점진적으로 트래픽을 전환하는 전략은?

A) AllAtOnce  
B) Blue/Green  
C) Canary 또는 Linear  
D) Rolling  

**정답: C** - Lambda Canary나 Linear 배포 전략은 점진적으로 새 버전으로 트래픽을 전환합니다.

---

**문제 3.** CodeDeploy에서 EC2 배포에 필요한 것은?

A) IAM 역할만  
B) CodeDeploy Agent 설치  
C) VPC 구성  
D) S3 버킷  

**정답: B** - EC2에 CodeDeploy를 사용하려면 각 인스턴스에 CodeDeploy Agent를 설치해야 합니다.

---

**문제 4.** Beanstalk에서 환경 설정 커스터마이징에 사용하는 파일은?

A) Dockerfile  
B) buildspec.yml  
C) appspec.yml  
D) .ebextensions/*.config  

**정답: D** - Elastic Beanstalk는 .ebextensions 폴더의 .config 파일로 환경을 커스터마이징합니다.

---

**문제 5.** CodePipeline에서 단계 간 데이터 전달 방법은?

A) 메모리 공유  
B) S3 아티팩트  
C) DynamoDB  
D) SQS 메시지  

**정답: B** - CodePipeline은 단계 간에 S3 아티팩트를 통해 데이터를 전달합니다.

---

**문제 6.** appspec.yml의 ValidateService 훅은 언제 실행되는가?

A) 배포 시작 전  
B) 파일 설치 중  
C) 애플리케이션 시작 후 검증  
D) 배포 취소 시  

**정답: C** - ValidateService는 애플리케이션이 시작된 후 서비스가 정상 동작하는지 검증하는 단계입니다.

---

**문제 7.** Beanstalk의 가장 안전한 배포 전략은?

A) All at once  
B) Rolling  
C) Rolling with Additional Batch  
D) Immutable  

**정답: D** - Immutable 배포는 새 ASG에 새 버전을 배포하고 검증 후 전환하므로 가장 안전합니다.

---

**문제 8.** CI/CD에서 "지속적 전달(Continuous Delivery)"의 의미는?

A) 매 커밋마다 자동으로 프로덕션 배포  
B) 자동으로 스테이징 배포, 프로덕션은 수동 승인  
C) 수동 빌드, 자동 배포  
D) 자동 빌드, 수동 배포  

**정답: B** - 지속적 전달은 자동으로 스테이징까지 배포하고, 프로덕션 배포는 수동 승인(Manual Approval) 후 진행합니다.

---

## 📌 오늘의 요약

1. CodeCommit: Git 소스 저장, IAM/SSH/HTTPS 인증
2. CodeBuild: buildspec.yml로 빌드/테스트/아티팩트 생성
3. CodeDeploy: appspec.yml로 EC2/Lambda/ECS 배포, 다양한 배포 전략
4. CodePipeline: 전체 파이프라인 오케스트레이션, S3로 아티팩트 전달
5. Beanstalk: PaaS, .ebextensions 커스터마이징, Immutable이 가장 안전
