# Day 37 - CodeDeploy: 배포 전략

📅 날짜: 2026년 7월 6일 (월요일)  
🎯 주제: AWS CodeDeploy  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- CodeDeploy의 배포 대상과 배포 전략을 이해한다
- appspec.yml 구성 방법을 학습한다
- 롤링/블루그린 배포를 구현한다

---

## 📖 이론 내용

### 1. AWS CodeDeploy란?

애플리케이션을 EC2, Lambda, ECS에 자동으로 배포하는 서비스입니다.

**배포 대상:**
- EC2 인스턴스 (CodeDeploy Agent 필요)
- Auto Scaling Group
- Lambda 함수
- ECS 서비스

### 2. EC2/온프레미스 배포 전략

**In-Place (In-Place Rolling):**
```
현재: [인스턴스1] [인스턴스2] [인스턴스3] (v1)
        |
        v
배포 중: [인스턴스1:v2] [인스턴스2:배포중] [인스턴스3:v1]
        |
        v
완료: [인스턴스1] [인스턴스2] [인스턴스3] (v2)
```

**배포 구성 옵션:**
- `AllAtOnce`: 모든 인스턴스 동시 배포 (중단 있음)
- `HalfAtATime`: 절반씩 배포
- `OneAtATime`: 하나씩 배포 (가장 안전)

### 3. appspec.yml (EC2 배포)

```yaml
version: 0.0
os: linux

files:
  - source: /
    destination: /home/ec2-user/myapp

permissions:
  - object: /home/ec2-user/myapp
    owner: ec2-user
    group: ec2-user

hooks:
  BeforeInstall:
    - location: scripts/stop_server.sh
      timeout: 60
      runas: ec2-user
  
  AfterInstall:
    - location: scripts/install_deps.sh
      timeout: 120
      runas: ec2-user
  
  ApplicationStart:
    - location: scripts/start_server.sh
      timeout: 60
      runas: ec2-user
  
  ValidateService:
    - location: scripts/validate.sh
      timeout: 60
      runas: ec2-user
```

**배포 라이프사이클 훅:**
```
ApplicationStop → BeforeInstall → Install → AfterInstall
      |
      v
ApplicationStart → ValidateService
```

### 4. Lambda 배포 (appspec.yml)

```yaml
version: 0.0
Resources:
  - MyLambdaFunction:
      Type: AWS::Lambda::Function
      Properties:
        Name: MyFunction
        Alias: live
        CurrentVersion: 1
        TargetVersion: 2

Hooks:
  - BeforeAllowTraffic: PreTrafficHook
  - AfterAllowTraffic: PostTrafficHook
```

**Lambda 배포 전략:**
- `Canary10Percent5Minutes`: 10%를 5분간 테스트 후 나머지 배포
- `Linear10PercentEvery1Minute`: 1분마다 10%씩 증가
- `AllAtOnce`: 즉시 전체 전환

### 5. 블루/그린 배포

```
[블루 환경 - 현재 운영]     [그린 환경 - 새 버전]
  인스턴스1:v1              인스턴스4:v2
  인스턴스2:v1              인스턴스5:v2
  인스턴스3:v1              인스턴스6:v2
        |                         |
        +-----[로드밸런서]----------+
        
검증 후 → 로드밸런서를 그린으로 전환
롤백 → 로드밸런서를 블루로 복귀
```

---

## 아키텍처 다이어그램

```
CodeDeploy 배포 흐름
================================

[S3 아티팩트] 또는 [GitHub]
          |
          v
[CodeDeploy 배포 그룹]
          |
          | appspec.yml 실행
          v
[배포 라이프사이클]
  1. ApplicationStop
  2. BeforeInstall
  3. Install
  4. AfterInstall
  5. ApplicationStart
  6. ValidateService

Lambda 배포 (Canary)
================================

[ALias:live → v1: 90%]
[Alias:live → v2: 10%]   ← Canary 10%
       |
  5분간 모니터링
       |
  오류 없음 → v2: 100%
  오류 발생 → 자동 롤백 v1: 100%
```

---

## ⭐ 핵심 포인트

1. ⭐ **EC2 배포**: CodeDeploy Agent 필수 설치
2. ⭐ **appspec.yml**: 파일 복사 + 훅 스크립트 정의
3. ⭐ **배포 전략**: AllAtOnce(빠름/위험), OneAtATime(느림/안전)
4. ⭐ **Lambda Canary**: 점진적 트래픽 전환, 자동 롤백
5. ⭐ **블루/그린**: 즉시 롤백 가능, 비용은 2배

---

## 📝 연습 문제

**문제 1.** EC2에 CodeDeploy를 사용하기 위한 전제 조건은?

A) CodePipeline 설정  
B) EC2에 CodeDeploy Agent 설치  
C) IAM 역할 생성  
D) S3 버킷 생성  

**정답: B** - EC2 인스턴스에 CodeDeploy Agent를 설치해야 CodeDeploy가 작동합니다.

---

**문제 2.** 가장 안전하지만 느린 EC2 배포 전략은?

A) AllAtOnce  
B) HalfAtATime  
C) OneAtATime  
D) RollingWithAdditionalBatch  

**정답: C** - OneAtATime은 하나씩 배포하여 가장 안전하지만 가장 느립니다.

---

**문제 3.** appspec.yml에서 배포 후 서비스 검증 훅은?

A) AfterInstall  
B) ApplicationStart  
C) ValidateService  
D) PostDeploy  

**정답: C** - ValidateService 훅에서 배포 후 서비스 정상 동작 여부를 검증합니다.

---

**문제 4.** Lambda Canary10Percent5Minutes의 의미는?

A) 10개의 Lambda에 5분씩 배포  
B) 5분 동안 10%의 트래픽만 새 버전으로 전송 후 나머지 배포  
C) 10분 동안 5%씩 증가  
D) 5분 간격으로 10번 배포  

**정답: B** - 처음 5분간 10%의 트래픽을 새 버전으로 보내고, 문제없으면 나머지 90%를 전환합니다.

---

**문제 5.** 블루/그린 배포의 장점은?

A) 비용 절감  
B) 즉각적인 롤백 가능  
C) 인스턴스 수 감소  
D) 자동 스케일링  

**정답: B** - 블루/그린 배포는 이전 환경(블루)을 유지하므로 문제 발생 시 로드밸런서를 즉시 원래 환경으로 전환할 수 있습니다.

---

## 📌 오늘의 요약

1. CodeDeploy: EC2/Lambda/ECS 자동 배포, appspec.yml로 정의
2. EC2 배포: Agent 필수, 라이프사이클 훅으로 스크립트 실행
3. 배포 전략: AllAtOnce(빠름), OneAtATime(안전), HalfAtATime
4. Lambda 배포: Canary/Linear/AllAtOnce, 자동 롤백 지원
5. 블루/그린: 2배 비용이지만 즉각적인 롤백 가능
