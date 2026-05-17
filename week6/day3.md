# Day 28 - DynamoDB: 읽기/쓰기 용량, Streams

📅 날짜: 2026년 6월 23일 (화요일)
🎯 주제: DynamoDB 용량 모드 및 Streams
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- DynamoDB 읽기/쓰기 용량 단위(RCU/WCU)를 계산한다
- 프로비저닝 모드와 온디맨드 모드를 비교한다
- DynamoDB Streams의 동작 방식을 이해한다

---

## 📖 이론 내용

### 1. 용량 단위 (Capacity Units)

**읽기 용량 단위 (RCU):**
- 강력한 일관성 읽기: 4KB당 1 RCU
- 최종적 일관성 읽기: 4KB당 0.5 RCU
- 트랜잭션 읽기: 4KB당 2 RCU

**RCU 계산 예시:**
`
항목 크기 10KB, 강력한 일관성 10회 읽기
RCU = ceil(10KB/4KB) x 1 x 10 = 3 x 10 = 30 RCU
`

**쓰기 용량 단위 (WCU):**
- 1KB당 1 WCU (항상 동일)
- 트랜잭션 쓰기: 1KB당 2 WCU

**WCU 계산 예시:**
`
항목 크기 2.5KB, 분당 10회 쓰기 (초당 약 0.17회)
WCU = ceil(2.5KB/1KB) x 1 = 3 WCU
`

### 2. 용량 모드

**프로비저닝 모드:**
- 미리 RCU/WCU 지정
- Auto Scaling으로 자동 조정 가능
- 예측 가능한 워크로드에 비용 효율적

**온디맨드 모드:**
- 사용량에 따라 자동 확장
- 예측 불가능한 워크로드에 적합
- 프로비저닝보다 약 2~3배 비용

**⭐ 모드 전환**: 24시간에 한 번만 전환 가능

### 3. DynamoDB Streams

테이블의 모든 변경 사항(INSERT, MODIFY, REMOVE)을 스트림으로 기록합니다.

**스트림 뷰 유형:**
- KEYS_ONLY: 기본 키만
- NEW_IMAGE: 변경 후 전체 항목
- OLD_IMAGE: 변경 전 전체 항목
- NEW_AND_OLD_IMAGES: 변경 전/후 모두

**사용 사례:**
- 실시간 데이터 처리 (Lambda 트리거)
- 교차 리전 복제
- ElastiCache 캐시 무효화
- 감사 로그

**⭐ Lambda + DynamoDB Streams:**
- Lambda가 Streams를 폴링
- 샤드당 하나의 Lambda 동시 실행
- 최대 1년 데이터 보존

---

## 아키텍처 다이어그램

`
DynamoDB 용량 계산
================================

RCU 계산:
항목 크기 6KB, 강력한 일관성 읽기:
ceil(6/4) = 2 RCU/읽기

항목 크기 6KB, 최종적 일관성 읽기:
ceil(6/4) x 0.5 = 1 RCU/읽기

WCU 계산:
항목 크기 3.5KB 쓰기:
ceil(3.5/1) = 4 WCU/쓰기

DynamoDB Streams 아키텍처
================================

사용자 주문 생성
      |
      v
[DynamoDB Table]
  PUT Item (신규 주문)
      |
      | Streams 이벤트 발생
      v
[DynamoDB Streams]
  이벤트: INSERT
  NewImage: {orderId: "O001", status: "PENDING"}
      |
      | Lambda 폴링
      v
[Lambda 함수]
      |
      +---> [SNS] 알림 발송
      |
      +---> [ElastiCache] 캐시 업데이트
      |
      +---> [OpenSearch] 검색 인덱스 업데이트
`

---

## ⭐ 핵심 포인트

1. ⭐ **RCU 계산**: ceil(항목크기/4KB) x 읽기 일관성 계수
2. ⭐ **WCU 계산**: ceil(항목크기/1KB) x 1
3. ⭐ **용량 모드 전환**: 24시간에 한 번만 가능
4. ⭐ **Streams 보존**: 최대 24시간 (Lambda로 처리)
5. ⭐ **NEW_AND_OLD_IMAGES**: 변경 전/후 모두 기록

---

## 📝 연습 문제

**문제 1.** 6KB 항목을 강력한 일관성으로 읽을 때 소비 RCU는?

A) 1 RCU  
B) 2 RCU  
C) 3 RCU  
D) 6 RCU  

**정답: B** - ceil(6/4) = 2 RCU (강력한 일관성 1 RCU/4KB)

---

**문제 2.** DynamoDB Streams의 데이터 보존 기간은?

A) 1시간  
B) 24시간  
C) 7일  
D) 30일  

**정답: B** - DynamoDB Streams의 데이터는 24시간 동안 보존됩니다.

---

**문제 3.** 프로비저닝 모드에서 온디맨드 모드로 전환 제한은?

A) 언제든 가능  
B) 24시간에 한 번  
C) 1주일에 한 번  
D) 1달에 한 번  

**정답: B** - DynamoDB 용량 모드는 24시간에 한 번만 전환 가능합니다.

---

**문제 4.** 변경 전후 모두 기록하는 Streams 뷰 유형은?

A) KEYS_ONLY  
B) NEW_IMAGE  
C) OLD_IMAGE  
D) NEW_AND_OLD_IMAGES  

**정답: D** - NEW_AND_OLD_IMAGES는 변경 전 항목과 변경 후 항목을 모두 스트림에 기록합니다.

---

**문제 5.** 트랜잭션 쓰기의 WCU 소비는?

A) 0.5 WCU/1KB  
B) 1 WCU/1KB  
C) 2 WCU/1KB  
D) 4 WCU/1KB  

**정답: C** - 트랜잭션 쓰기는 일반 쓰기(1 WCU/1KB)보다 2배인 2 WCU/1KB를 소비합니다.

---

## 📌 오늘의 요약

1. RCU: 강력한 일관성 1 RCU/4KB, 최종적 일관성 0.5 RCU/4KB, 트랜잭션 2 RCU/4KB
2. WCU: 1 WCU/1KB, 트랜잭션 2 WCU/1KB
3. 프로비저닝(예측 가능, 저렴) vs 온디맨드(유연, 비쌈) - 24시간에 한 번만 전환
4. DynamoDB Streams: 변경 사항 스트림, 24시간 보존, Lambda로 실시간 처리
5. Streams 뷰: KEYS_ONLY, NEW_IMAGE, OLD_IMAGE, NEW_AND_OLD_IMAGES