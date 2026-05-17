# Day 21 - S3: 버킷, 객체, 스토리지 클래스

📅 날짜: 2026년 6월 14일 (일요일)  
🎯 주제: Amazon S3 기초  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- S3 버킷과 객체의 기본 개념을 이해한다
- S3 스토리지 클래스별 특성과 사용 사례를 구분한다
- S3 내구성, 가용성, 비용 구조를 이해한다

---

## 📖 이론 내용

### 1. Amazon S3란?

Amazon S3(Simple Storage Service)는 무한히 확장 가능한 객체 스토리지 서비스입니다.

**S3 핵심 특성:**
- 객체 단위 저장 (파일 시스템이 아님)
- 최대 객체 크기: **5TB** (5GB 이상은 멀티파트 업로드)
- 버킷은 글로벌 이름 공간 (전 세계 유일한 이름)
- 버킷은 특정 리전에 생성
- **99.999999999% (11 nines) 내구성**

### 2. 버킷 (Bucket)

**버킷 명명 규칙:**
- 소문자, 숫자, 하이픈(-), 점(.) 사용 가능
- 3~63자, IP 주소 형식 불가
- 글로벌로 유일해야 함

**버킷 URL 형식:**
```
가상 호스팅: https://bucket-name.s3.ap-northeast-2.amazonaws.com/key
```

### 3. 객체 (Object)

**객체 구성:**
- **키(Key)**: 전체 경로 (예: documents/2026/report.pdf)
- **값(Value)**: 실제 파일 내용
- **메타데이터**: 시스템 + 사용자 정의
- **버전 ID**: 버전 관리 활성화 시
- **태그**: 최대 10개

**⭐ S3는 디렉토리가 없음**: "/" 구분자로 폴더처럼 보이지만 실제로는 키의 일부

### 4. S3 스토리지 클래스

| 스토리지 클래스 | 가용성 | 최소 저장 | 사용 사례 |
|----------------|--------|-----------|-----------|
| S3 Standard | 99.99% | 없음 | 자주 접근 |
| S3 Standard-IA | 99.9% | 30일 | 가끔 접근, 빠른 조회 |
| S3 One Zone-IA | 99.5% | 30일 | 재생성 가능 데이터 |
| S3 Intelligent-Tiering | 99.9% | 없음 | 접근 패턴 불규칙 |
| S3 Glacier Instant | 99.9% | 90일 | 분기별 조회 |
| S3 Glacier Flexible | 99.99% | 90일 | 1~12시간 복구 |
| S3 Glacier Deep Archive | 99.99% | 180일 | 12~48시간 복구 |

**핵심 암기법:**
- Standard: 핫 데이터, 최고 가용성
- IA(Infrequent Access): 저렴, 조회 비용 추가, 30일 최소
- Glacier: 저비용 아카이브, 복구 시간 있음
- Intelligent-Tiering: 자동 티어 이동, 모니터링 비용 추가

---

## 아키텍처 다이어그램

```
S3 버킷 구조
================================

S3 버킷: my-company-docs (전 세계 유일 이름)
  |
  +-- documents/          (접두사, 실제 디렉토리 아님)
  |     +-- 2026/
  |           +-- report.pdf
  |           +-- presentation.pptx
  |
  +-- images/
  |     +-- logo.png
  |
  +-- backups/
        +-- 2026-05-17.tar.gz

스토리지 클래스 비용 vs 접근 빈도
================================

접근 빈도  높음 <----------------------> 낮음
           Standard  IA    Glacier  Deep Archive
저장 비용  높음      중    낮음     최저
조회 비용  낮음      중    높음     최고
복구 시간  즉시      즉시  분~시간  12~48시간
```

---

## ⭐ 핵심 포인트 (시험 출제 빈도 높음)

1. ⭐ **버킷 이름**: 전 세계 유일, 소문자만
2. ⭐ **최대 객체 크기**: 5TB (5GB 이상은 멀티파트 업로드 필수)
3. ⭐ **Intelligent-Tiering**: 모니터링 비용 추가, 자동 티어 이동
4. ⭐ **IA 최소 저장 30일**: 조기 삭제 시에도 30일치 과금
5. ⭐ **내구성 11 nines**: 99.999999999%, 모든 스토리지 클래스 동일

---

## 💻 실제 예시

```bash
# 버킷 생성
aws s3api create-bucket \
  --bucket my-unique-bucket-12345 \
  --region ap-northeast-2 \
  --create-bucket-configuration LocationConstraint=ap-northeast-2

# 객체 업로드 (스토리지 클래스 지정)
aws s3 cp archive.zip s3://my-bucket/archives/ \
  --storage-class GLACIER

# 버킷 내 객체 목록
aws s3 ls s3://my-bucket/documents/ --recursive

# 객체 스토리지 클래스 변경
aws s3 cp s3://my-bucket/old-file.txt s3://my-bucket/old-file.txt \
  --storage-class STANDARD_IA
```

---

## 📝 연습 문제

**문제 1.** S3 버킷 이름에 대한 올바른 설명은?

A) 계정 내에서만 유일하면 된다  
B) 대문자를 포함할 수 있다  
C) 전 세계적으로 유일해야 한다  
D) 숫자로 시작해야 한다  

**정답: C** - S3 버킷 이름은 전 세계 모든 AWS 계정에서 유일해야 합니다.

---

**문제 2.** 분기에 한 번 접근하는 데이터를 즉시 조회 가능하게 저장하려면?

A) S3 Standard  
B) S3 Standard-IA  
C) S3 Glacier Instant Retrieval  
D) S3 Glacier Deep Archive  

**정답: C** - Glacier Instant Retrieval은 분기별 접근에 최적화, 밀리초 이내 즉시 조회 가능합니다.

---

**문제 3.** S3 객체의 최대 크기는?

A) 5GB  
B) 100GB  
C) 1TB  
D) 5TB  

**정답: D** - S3 객체의 최대 크기는 5TB입니다.

---

**문제 4.** S3 Intelligent-Tiering의 특징은?

A) 수동으로 스토리지 클래스를 변경해야 한다  
B) 접근 패턴에 따라 자동으로 스토리지 클래스를 조정한다  
C) 조회 시간이 12시간이다  
D) 최소 저장 기간이 180일이다  

**정답: B** - S3 Intelligent-Tiering은 접근 패턴을 모니터링하여 자동으로 적절한 티어로 이동합니다.

---

**문제 5.** 30일 미만 저장 후 삭제 시 30일치 비용이 청구되는 클래스는?

A) S3 Standard  
B) S3 Standard-IA  
C) S3 Glacier Deep Archive  
D) S3 One Zone-IA  

**정답: B** - Standard-IA는 최소 저장 기간 30일, 30일 미만 삭제 시에도 30일치 과금됩니다.

---

## 📌 오늘의 요약

1. S3는 무한 확장 객체 스토리지, 버킷 이름은 글로벌 유일, 객체 최대 크기 5TB
2. 스토리지 클래스: Standard(핫), IA(저빈도), Glacier(아카이브), Deep Archive(최저가)
3. Intelligent-Tiering: 접근 패턴에 따라 자동 티어 이동, 모니터링 비용 추가
4. IA 최소 30일, Glacier 90일, Deep Archive 180일 (조기 삭제 시에도 과금)
5. S3는 디렉토리 없이 "/" 구분자로 폴더처럼 보이는 것은 키의 일부
