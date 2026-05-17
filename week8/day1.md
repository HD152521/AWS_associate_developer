# Day 36 - CI/CD: CodeCommit, CodeBuild

📅 날짜: 2026년 7월 5일 (일요일)  
🎯 주제: AWS CI/CD 파이프라인 - 소스 및 빌드  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- AWS CodeCommit의 특성과 보안을 이해한다
- CodeBuild의 빌드 프로세스와 buildspec.yml을 작성한다
- CI/CD 파이프라인의 전체 흐름을 파악한다

---

## 📖 이론 내용

### 1. AWS CodeCommit

AWS가 관리하는 Git 기반 소스 제어 서비스입니다.

**특징:**
- 완전 관리형 Git 저장소
- AWS 계정 내 프라이빗 저장소
- IAM 기반 접근 제어
- 전송 암호화 (HTTPS/SSH)
- 저장 암호화 (KMS)

**인증 방법:**
```bash
# SSH 인증
git remote add origin ssh://git-codecommit.ap-northeast-2.amazonaws.com/v1/repos/my-repo

# HTTPS (Git 자격 증명)
git remote add origin https://git-codecommit.ap-northeast-2.amazonaws.com/v1/repos/my-repo

# HTTPS (Git 자격 증명 도우미)
git config --global credential.helper '!aws codecommit credential-helper $@'
git config --global credential.UseHttpPath true
```

### 2. CodeCommit 알림 및 트리거

```json
{
  "규칙": "pull request 생성 시 Slack 알림",
  "이벤트": "pullRequestCreated",
  "대상": "SNS Topic → Lambda → Slack Webhook"
}
```

**트리거 vs 알림:**
- **트리거**: 코드 변경 시 Lambda 또는 SNS 직접 호출
- **알림**: CodeStar Notifications를 통해 더 세밀한 이벤트 필터링

### 3. AWS CodeBuild

소스 코드를 빌드, 테스트, 아티팩트 생성하는 완전 관리형 CI 서비스입니다.

**빌드 프로세스:**
```
소스 코드 (CodeCommit/S3/GitHub)
          |
          v
[CodeBuild 빌드 환경]
   buildspec.yml 실행
          |
          v
[빌드 아티팩트] → S3
```

**buildspec.yml:**
```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      python: 3.9
    commands:
      - pip install -r requirements.txt
  
  pre_build:
    commands:
      - echo "테스트 시작..."
      - python -m pytest tests/ -v
  
  build:
    commands:
      - echo "빌드 시작..."
      - zip -r function.zip .
  
  post_build:
    commands:
      - echo "빌드 완료"
      - aws s3 cp function.zip s3://my-bucket/function.zip

artifacts:
  files:
    - function.zip
  discard-paths: yes

cache:
  paths:
    - '/root/.cache/pip/**/*'
```

### 4. CodeBuild 환경 변수

```yaml
# 일반 환경 변수
env:
  variables:
    ENV: production
    REGION: ap-northeast-2
  
  # SSM Parameter Store에서 가져오기
  parameter-store:
    DB_PASSWORD: /myapp/db/password
  
  # Secrets Manager에서 가져오기
  secrets-manager:
    API_KEY: prod/myapp/api-key
```

---

## 아키텍처 다이어그램

```
CI/CD 소스 및 빌드 파이프라인
================================

[개발자]
    |
    | git push
    v
[CodeCommit]
    |
    | 코드 변경 감지
    v
[CodeBuild]
    |
    +-- install → pre_build → build → post_build
    |
    +-- 단위 테스트 실행
    |
    +-- 아티팩트 생성
    v
[S3 아티팩트 버킷]
    |
    v
[배포 단계 (CodeDeploy)]
```

---

## ⭐ 핵심 포인트

1. ⭐ **CodeCommit**: 완전 관리형 Git, IAM 기반 접근 제어
2. ⭐ **buildspec.yml**: 빌드 명령 정의, install/pre_build/build/post_build
3. ⭐ **CodeBuild 환경 변수**: 평문, SSM Parameter Store, Secrets Manager 지원
4. ⭐ **빌드 캐시**: S3 또는 로컬 캐시로 빌드 속도 향상
5. ⭐ **아티팩트**: S3에 저장, 다음 배포 단계에서 사용

---

## 📝 연습 문제

**문제 1.** CodeCommit의 인증에 사용할 수 없는 방법은?

A) SSH 키  
B) HTTPS Git 자격 증명  
C) IAM 사용자/역할  
D) 사용자 이름/비밀번호 직접 입력  

**정답: D** - CodeCommit은 SSH, HTTPS Git 자격 증명, AWS 자격 증명 도우미를 지원합니다. 단순 사용자명/비밀번호는 지원하지 않습니다.

---

**문제 2.** buildspec.yml에서 의존성 설치는 어느 단계에서?

A) pre_build  
B) install  
C) build  
D) post_build  

**정답: B** - 런타임 및 의존성 설치는 install 단계에서 수행합니다.

---

**문제 3.** CodeBuild에서 DB 비밀번호를 안전하게 주입하는 방법은?

A) buildspec.yml에 평문으로 저장  
B) 환경 변수로 직접 설정  
C) SSM Parameter Store 또는 Secrets Manager 참조  
D) S3에서 파일로 가져오기  

**정답: C** - 민감한 정보는 SSM Parameter Store나 Secrets Manager에 저장하고 참조합니다.

---

**문제 4.** CodeBuild 빌드 속도를 높이는 방법은?

A) 더 큰 인스턴스 사용  
B) 의존성 캐시 활용  
C) 멀티스레드 빌드  
D) 병렬 빌드만 가능  

**정답: B** - 의존성 캐시(S3 또는 로컬)를 활용하면 의존성 재설치를 건너뛰어 빌드 속도를 높일 수 있습니다.

---

**문제 5.** CodeCommit과 GitHub의 가장 큰 차이점은?

A) 성능 차이  
B) CodeCommit은 AWS 계정 내 완전 프라이빗, IAM 통합  
C) 비용 차이  
D) 지원 언어 차이  

**정답: B** - CodeCommit은 AWS 계정 내에서 완전히 격리된 프라이빗 저장소를 제공하고 IAM으로 세밀하게 접근을 제어할 수 있습니다.

---

## 📌 오늘의 요약

1. CodeCommit: 완전 관리형 Git, 프라이빗, IAM/SSH/HTTPS 인증
2. CodeBuild: 소스 빌드/테스트/아티팩트 생성, buildspec.yml로 정의
3. buildspec.yml: install → pre_build → build → post_build 단계
4. 환경 변수: 평문, SSM Parameter Store, Secrets Manager 지원
5. 빌드 캐시: S3/로컬 캐시로 빌드 속도 향상
