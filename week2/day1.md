# Day 6 - EC2 기본: 인스턴스 유형, AMI

📅 날짜: 2026년 5월 24일 (일요일)  
🎯 주제: Amazon EC2 기초  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- EC2(Elastic Compute Cloud)의 개념과 주요 기능을 이해한다
- EC2 인스턴스 유형과 각 유형의 사용 사례를 구분한다
- AMI(Amazon Machine Image)의 개념과 활용 방법을 이해한다
- EC2 구매 옵션(온디맨드, 예약, 스팟)을 비교할 수 있다

---

## 📖 이론 내용

### 1. Amazon EC2란?

EC2(Elastic Compute Cloud)는 AWS의 핵심 컴퓨팅 서비스로, 클라우드에서 가상 서버(인스턴스)를 제공합니다. "Elastic"은 필요에 따라 용량을 늘리고 줄일 수 있다는 의미입니다.

**EC2의 주요 기능:**
- 가상 서버(인스턴스) 임대
- 다양한 운영체제 지원 (Amazon Linux, Ubuntu, Windows 등)
- 스토리지 연결 (EBS, 인스턴스 스토어)
- 네트워크 구성 (VPC, 보안 그룹)
- 탄력적 IP(Elastic IP) 할당

### 2. EC2 인스턴스 유형

EC2 인스턴스 유형은 CPU, 메모리, 스토리지, 네트워크 특성의 조합입니다.

#### 인스턴스 유형 명명 규칙
```
m5.xlarge
|  |  +----> 크기 (nano, micro, small, medium, large, xlarge, 2xlarge, ...)
|  +-------> 세대 (숫자가 높을수록 최신)
+----------> 패밀리 (특성 구분)
```

#### 주요 인스턴스 패밀리

| 패밀리 | 유형 | 사용 사례 |
|--------|------|-----------|
| **범용** | t3, t4g, m5, m6i | 웹 서버, 소규모 DB, 개발 환경 |
| **컴퓨팅 최적화** | c5, c6i | CPU 집약적 작업, 고성능 웹 서버, 게임 서버 |
| **메모리 최적화** | r5, r6i, x1 | 대용량 메모리 DB, 실시간 빅데이터 |
| **스토리지 최적화** | i3, d2 | 고성능 로컬 스토리지, OLTP |
| **가속 컴퓨팅** | p3, g4 | ML/AI, GPU 처리, 그래픽 |

**⭐ 각 패밀리 첫 글자 암기:**
- **M**: Most scenarios (범용)
- **C**: Compute (컴퓨팅 최적화)
- **R**: RAM (메모리 최적화)
- **I/D**: I/O, Disk (스토리지 최적화)
- **G/P**: GPU (가속 컴퓨팅)

### 3. AMI (Amazon Machine Image)

AMI는 EC2 인스턴스를 시작하는 데 필요한 정보를 포함하는 템플릿입니다.

**AMI 구성 요소:**
- 루트 볼륨 템플릿 (OS, 애플리케이션)
- 시작 권한 (어떤 계정이 이 AMI를 사용할 수 있는지)
- 블록 디바이스 매핑 (연결할 EBS 볼륨)

**AMI 유형:**
- **AWS에서 제공**: Amazon Linux 2023, Ubuntu, Windows Server 등
- **AWS Marketplace**: 써드파티 소프트웨어 포함 AMI
- **커뮤니티 AMI**: 다른 AWS 사용자가 공유한 AMI
- **나만의 AMI**: 기존 인스턴스로부터 생성한 커스텀 AMI

**⭐ AMI는 특정 리전에 종속적** - 다른 리전 사용 시 AMI를 복사해야 함

### 4. EC2 구매 옵션

| 옵션 | 설명 | 할인율 | 사용 사례 |
|------|------|--------|-----------|
| **온디맨드** | 사용한 만큼 시간/초 단위 과금 | - | 단기, 불규칙 워크로드 |
| **예약 인스턴스** | 1년 또는 3년 약정 | 최대 72% | 안정적, 예측 가능한 사용 |
| **절약 플랜** | 사용량 약정 ($/시간) | 최대 72% | 유연한 예약 |
| **스팟 인스턴스** | 미사용 용량 경매 | 최대 90% | 내결함성, 배치 처리 |
| **전용 호스트** | 물리적 서버 전용 | - | 라이선스 규정 준수 |
| **전용 인스턴스** | 전용 하드웨어 | - | 규정 준수 요건 |

---

## 아키텍처 다이어그램

```
EC2 인스턴스 유형 구분
================================

              CPU   메모리  I/O
범용 (M, T)   중간   중간   중간  --> 웹서버, 앱서버
                                    
컴퓨팅 (C)    높음   보통   보통  --> 게임서버, HPC
                                    
메모리 (R, X) 보통   높음   보통  --> 대용량 DB, Redis

스토리지 (I)  보통   보통   높음  --> NoSQL DB, 데이터웨어하우스

GPU (G, P)    보통   보통   보통  --> ML/AI, 그래픽 렌더링
              +GPU


AMI -> EC2 인스턴스 생성 과정
================================

[AMI 선택]
AWS 제공 AMI | 마켓플레이스 AMI | 커스텀 AMI
     |
     v
[인스턴스 유형 선택]
t3.micro | m5.large | c5.xlarge | ...
     |
     v
[구성 설정]
VPC, 서브넷, IAM 역할, 사용자 데이터
     |
     v
[스토리지 설정]
EBS 볼륨 크기/유형
     |
     v
[보안 그룹, 키 페어 설정]
     |
     v
[EC2 인스턴스 실행]


구매 옵션 비교 (비용 절감률)
================================

온디맨드:    |||||||||||||||||||||  100% (기준)
예약 (1년):  ||||||||||||          ~60% (40% 절감)
예약 (3년):  ||||||||              ~40% (60% 절감)
스팟:        ||                    ~10% (최대 90% 절감!)
```

---

## ⭐ 핵심 포인트 (시험 출제 빈도 높음)

1. ⭐ **스팟 인스턴스**: 최대 90% 할인, 단 AWS가 2분 예고 후 회수 가능
2. ⭐ **예약 인스턴스 vs 절약 플랜**: 예약=특정 인스턴스 유형, 절약=사용량 약정
3. ⭐ **AMI는 리전 종속적**: 다른 리전에서 사용 시 AMI 복사 필요
4. ⭐ **T 계열**: 버스트 가능 인스턴스 (크레딧 기반 CPU 버스팅)
5. ⭐ **전용 호스트 vs 전용 인스턴스**: 호스트=물리 서버 전용, 인스턴스=하드웨어 전용

---

## 💻 실제 예시 - EC2 CLI 명령어

```bash
# 인스턴스 유형 목록 조회 (특정 리전)
aws ec2 describe-instance-types \
  --filters "Name=instance-type,Values=t3.*" \
  --query 'InstanceTypes[*].[InstanceType,VCpuInfo.DefaultVCpus,MemoryInfo.SizeInMiB]' \
  --output table

# AMI 검색 (Amazon Linux 2023 최신 버전)
aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=al2023-ami-*" \
  --query 'sort_by(Images, &CreationDate)[-1].[ImageId,Name]' \
  --output table

# EC2 인스턴스 시작
aws ec2 run-instances \
  --image-id ami-0c9c942bd7bf113a2 \
  --instance-type t3.micro \
  --key-name my-key-pair \
  --security-group-ids sg-12345678 \
  --subnet-id subnet-12345678 \
  --count 1 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=MyWebServer}]'

# 실행 중인 인스턴스 목록
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query 'Reservations[*].Instances[*].[InstanceId,InstanceType,PublicIpAddress,Tags[?Key==`Name`].Value|[0]]' \
  --output table

# 커스텀 AMI 생성
aws ec2 create-image \
  --instance-id i-1234567890abcdef0 \
  --name "MyWebServer-AMI-v1.0" \
  --description "Web server with application installed" \
  --no-reboot
```

---

## 📝 연습 문제

**문제 1.** 메모리 집약적 데이터베이스 워크로드에 가장 적합한 EC2 인스턴스 패밀리는?

A) C 계열 (컴퓨팅 최적화)  
B) R 계열 (메모리 최적화)  
C) T 계열 (범용 버스트)  
D) I 계열 (스토리지 최적화)  

**정답: B**  
해설: R 계열은 메모리 최적화 인스턴스로, 대용량 메모리가 필요한 인메모리 데이터베이스, 빅데이터 처리, 실시간 분석에 적합합니다.

---

**문제 2.** 스팟 인스턴스에 대한 올바른 설명은?

A) 항상 사용 가능하며 AWS가 중단할 수 없다  
B) 온디맨드보다 비용이 높다  
C) AWS가 2분 전 경고 후 회수할 수 있으며 최대 90% 할인  
D) 1년 또는 3년 약정이 필요하다  

**정답: C**  
해설: 스팟 인스턴스는 AWS의 미사용 용량을 경매 방식으로 제공하며, 최대 90% 저렴하지만 AWS가 필요 시 2분 경고 후 회수할 수 있습니다.

---

**문제 3.** AMI(Amazon Machine Image)에 대한 올바른 설명은?

A) AMI는 모든 리전에서 자동으로 사용 가능하다  
B) AMI는 특정 리전에 종속적이다  
C) AMI는 반드시 AWS에서 제공하는 것만 사용해야 한다  
D) AMI를 사용하면 항상 동일한 인스턴스 유형을 사용해야 한다  

**정답: B**  
해설: AMI는 특정 리전에 종속됩니다. 다른 리전에서 같은 AMI를 사용하려면 해당 리전으로 AMI를 복사해야 합니다.

---

**문제 4.** 예측 가능하고 안정적인 워크로드에 비용을 절감하려면 어떤 구매 옵션이 가장 적합한가?

A) 온디맨드 인스턴스  
B) 스팟 인스턴스  
C) 예약 인스턴스 (1년 또는 3년)  
D) 전용 호스트  

**정답: C**  
해설: 예약 인스턴스는 1년 또는 3년 약정으로 최대 72% 비용을 절감할 수 있으며, 예측 가능하고 지속적인 워크로드에 최적입니다.

---

**문제 5.** 인스턴스 유형 "c5.2xlarge"에서 "c"가 나타내는 것은?

A) 비용(Cost) 최적화 인스턴스  
B) 컨테이너(Container) 최적화 인스턴스  
C) 컴퓨팅(Compute) 최적화 인스턴스  
D) 커뮤니티(Community) 인스턴스  

**정답: C**  
해설: EC2 인스턴스 유형에서 첫 글자는 인스턴스 패밀리를 나타냅니다. "c"는 컴퓨팅 최적화(Compute Optimized) 인스턴스를 의미합니다.

---

## 📌 오늘의 요약

1. EC2는 AWS의 가상 서버 서비스로 다양한 인스턴스 유형(범용, 컴퓨팅, 메모리, 스토리지, GPU)을 제공한다
2. 인스턴스 유형은 패밀리-세대-크기 형식으로 명명된다 (예: m5.xlarge)
3. AMI는 인스턴스 시작 템플릿이며 특정 리전에 종속적이다
4. 구매 옵션: 온디맨드(유연), 예약(72% 절감), 스팟(90% 절감), 전용 호스트(라이선스)
5. 스팟 인스턴스는 가장 저렴하지만 AWS가 2분 경고 후 회수할 수 있어 내결함성 워크로드에 적합하다
