# Day 35 - Week 7 복습 + 연습문제 (RDS/ElastiCache/Aurora)

📅 날짜: 2026년 7월 2일 (목요일)  
🎯 주제: RDS, ElastiCache, Aurora 종합 복습  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- RDS, ElastiCache, Aurora의 핵심 개념을 종합 정리한다
- 실전 시험 유형의 문제를 풀어 실력을 점검한다

---

## 📖 Week 7 핵심 정리

### RDS 핵심 암기
```
Multi-AZ: 동기 복제, 자동 Failover, 고가용성 목적
Read Replica: 비동기 복제, 읽기 확장, 최대 15개
암호화 변경: 스냅샷 → 암호화 복사 → 새 DB
IAM 인증 토큰: 15분 유효, SSL 필수
자동 백업: 최대 35일, DB 삭제 시 함께 삭제
수동 스냅샷: 무기한, DB 삭제 후에도 유지
```

### ElastiCache 핵심 암기
```
Redis: 영속성, 백업, Multi-AZ, 복잡한 자료구조, Read Replica
Memcached: 단순 캐시, 멀티스레드, 영속성 없음
Lazy Loading: Cache Miss 시 DB 조회 후 캐시
Write-Through: 쓰기 시 캐시도 동시 업데이트
```

### Aurora 핵심 암기
```
성능: MySQL 5배, PostgreSQL 3배
스토리지: 3개 AZ, 6개 사본, 10GB~128TB 자동
Replica: 최대 15개
Serverless: 간헐적 트래픽, 자동 확장
글로벌 DB: 1초 미만 복제, 1분 내 Failover
```

---

## 아키텍처 다이어그램

```
Week 7 전체 아키텍처 패턴
================================

[클라이언트]
     |
     v
[API Gateway + Lambda]
     |
     +--[읽기 빈번]--> [ElastiCache Redis]
     |                      |
     |              Cache Miss |
     |                      v
     +--[쓰기/복잡쿼리]--> [Aurora Primary]
                              |
                         +----+----+
                         |         |
                    [Aurora    [Aurora
                   Replica1]  Replica2]
```

---

## 📝 Week 7 종합 연습문제

**문제 1.** RDS Multi-AZ와 Read Replica의 차이점은?

A) Multi-AZ는 비동기, Read Replica는 동기 복제  
B) Multi-AZ는 고가용성, Read Replica는 읽기 확장 목적  
C) Multi-AZ는 여러 리전, Read Replica는 단일 리전만  
D) 비용 차이만 있음  

**정답: B** - Multi-AZ는 동기 복제로 고가용성을 제공하고, Read Replica는 비동기 복제로 읽기 성능을 향상합니다.

---

**문제 2.** 세션 데이터를 여러 서버 간 공유하고 영속성도 필요한 경우?

A) Memcached  
B) Redis  
C) RDS  
D) S3  

**정답: B** - Redis는 영속성과 다중 서버 간 세션 공유를 모두 지원합니다.

---

**문제 3.** Aurora의 스토리지 사본 수와 AZ 분산은?

A) 2개 AZ에 4개 사본  
B) 3개 AZ에 6개 사본  
C) 3개 AZ에 3개 사본  
D) 2개 AZ에 2개 사본  

**정답: B** - Aurora는 3개 AZ에 자동으로 6개의 스토리지 사본을 유지합니다.

---

**문제 4.** RDS 비암호화 DB를 암호화하는 절차는?

A) 설정에서 암호화 활성화  
B) 스냅샷 → 암호화 스냅샷 복사 → 새 DB 생성  
C) Multi-AZ 활성화  
D) 불가능  

**정답: B** - 기존 DB를 직접 암호화할 수 없습니다. 스냅샷 생성 후 암호화된 스냅샷으로 복사하고 새 DB를 생성해야 합니다.

---

**문제 5.** 간헐적으로 사용하는 개발 환경 DB에 적합한 Aurora 구성은?

A) Aurora 표준  
B) Aurora Multi-AZ  
C) Aurora Serverless  
D) Aurora 글로벌 DB  

**정답: C** - Aurora Serverless는 사용하지 않을 때 자동으로 스케일 다운하여 간헐적 사용에 비용 효율적입니다.

---

**문제 6.** ElastiCache Lazy Loading의 특징은?

A) 모든 데이터를 미리 캐시  
B) Cache Miss 시 DB를 조회하고 캐시에 저장  
C) 쓰기 시 항상 캐시 업데이트  
D) 캐시 히트율 보장  

**정답: B** - Lazy Loading은 Cache Miss 시 DB에서 데이터를 가져와 캐시에 저장하는 방식입니다.

---

**문제 7.** RDS 자동 백업의 특징은?

A) 무기한 보존  
B) 최대 35일 보존, DB 삭제 시 함께 삭제  
C) S3에 직접 접근 가능  
D) 수동으로만 트리거 가능  

**정답: B** - RDS 자동 백업은 최대 35일 보존되며, DB 삭제 시 자동 백업도 함께 삭제됩니다.

---

**문제 8.** Aurora 글로벌 데이터베이스의 복제 지연과 Failover 시간은?

A) 복제 지연 수초, Failover 수분  
B) 복제 지연 1초 미만, Failover 1분 미만  
C) 복제 지연 없음, Failover 즉시  
D) 복제 지연 1분, Failover 5분  

**정답: B** - Aurora 글로벌 DB는 1초 미만의 복제 지연과 1분 미만의 Failover를 제공합니다.

---

## 📌 오늘의 요약

1. RDS: Multi-AZ(고가용성/동기), Read Replica(읽기확장/비동기), 최대 15개
2. RDS 보안: KMS 암호화, IAM 토큰(15분), SSL 필수
3. 백업: 자동(35일, DB삭제시 함께삭제), 수동(무기한, 유지됨)
4. ElastiCache: Redis(영속성/복잡구조), Memcached(단순/멀티스레드)
5. Aurora: MySQL 5배 성능, 3AZ 6사본, Serverless, 글로벌 DB
