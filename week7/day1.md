# Day 31 - RDS: 개요, Multi-AZ, Read Replica

📅 날짜: 2026년 6월 28일 (일요일)  
🎯 주제: Amazon RDS 핵심 기능  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- RDS의 지원 엔진과 기본 특성을 이해한다
- Multi-AZ와 Read Replica의 차이를 명확히 구분한다
- 적절한 RDS 아키텍처를 선택한다

---

## 📖 이론 내용

### 1. Amazon RDS란?

RDS (Relational Database Service)는 완전 관리형 관계형 데이터베이스 서비스입니다.

**지원 엔진:**
- MySQL, PostgreSQL, MariaDB (오픈소스)
- Oracle, SQL Server (상용)
- Amazon Aurora (AWS 독자 개발)

**RDS vs EC2에 DB 직접 설치:**
- RDS: 자동 패치, 백업, 모니터링, Multi-AZ, 고가용성
- EC2: 직접 관리, 더 많은 제어권

### 2. Multi-AZ (고가용성)

**목적**: 가용성 향상 (Availability)

```
Primary DB (AZ-A)
      |
      | 동기식 복제 (Synchronous Replication)
      v
Standby DB (AZ-B)

- 자동 장애 조치 (Failover): 1-2분 내 자동 전환
- DNS 엔드포인트는 동일 (애플리케이션 변경 불필요)
- 읽기/쓰기 모두 Primary에서만
- Standby는 백업에만 활용 가능
```

**⭐ 중요**: Multi-AZ는 재해 복구(DR)용, 성능 향상 목적 아님!

### 3. Read Replica (읽기 확장)

**목적**: 읽기 성능 향상 (Scalability)

```python
# 읽기 쿼리는 Read Replica로
read_db = connect(read_replica_endpoint)
read_db.execute("SELECT * FROM products")

# 쓰기는 Primary로
write_db = connect(primary_endpoint)
write_db.execute("INSERT INTO orders ...")
```

**특징:**
- 최대 15개의 Read Replica 생성 가능
- 비동기식 복제 (Asynchronous Replication)
- 동일 리전 또는 다른 리전 (Cross-Region)
- Read Replica를 승격(Promote)하여 독립 DB로 활용 가능

### 4. Multi-AZ vs Read Replica 비교

| 특성 | Multi-AZ | Read Replica |
|------|----------|--------------|
| 목적 | 고가용성 | 읽기 확장 |
| 복제 방식 | 동기식 | 비동기식 |
| Standby 읽기 | 불가 | 가능 |
| 자동 Failover | 가능 | 불가 |
| 최대 개수 | 1개 | 15개 |
| 비용 | 2배 | 추가 |

---

## 아키텍처 다이어그램

```
RDS 고가용성 + 읽기 확장 아키텍처
================================

[애플리케이션]
      |
      |-- 쓰기 --> [Primary RDS] (AZ-A)
      |                 |
      |                 | 동기식 복제
      |                 v
      |           [Multi-AZ Standby] (AZ-B)
      |           (자동 Failover 대기)
      |
      |-- 읽기 --> [Read Replica 1] (AZ-A)
      |           [Read Replica 2] (AZ-B)
      |           [Read Replica 3] (Cross-Region)

장애 발생 시:
Primary (AZ-A) 장애 → DNS 자동 전환 → Standby (AZ-B) Primary로 승격
(1-2분 내 자동 처리)
```

---

## ⭐ 핵심 포인트

1. ⭐ **Multi-AZ**: 동기 복제, 자동 Failover, 고가용성 목적
2. ⭐ **Read Replica**: 비동기 복제, 읽기 성능 향상 목적
3. ⭐ **Multi-AZ Standby**: 읽기 불가, DR 목적만
4. ⭐ **Read Replica 최대**: 15개, Cross-Region 가능
5. ⭐ **RDS 관리**: AWS가 패치/백업/모니터링 자동 처리

---

## 📝 연습 문제

**문제 1.** RDS Multi-AZ의 주요 목적은?

A) 읽기 성능 향상  
B) 고가용성 및 자동 Failover  
C) 비용 절감  
D) 쓰기 성능 향상  

**정답: B** - Multi-AZ는 동기 복제와 자동 Failover로 고가용성을 제공합니다.

---

**문제 2.** Read Replica의 최대 개수는?

A) 5개  
B) 10개  
C) 15개  
D) 20개  

**정답: C** - RDS Read Replica는 최대 15개까지 생성 가능합니다.

---

**문제 3.** Multi-AZ의 복제 방식은?

A) 비동기식  
B) 동기식  
C) 반동기식  
D) 배치 복제  

**정답: B** - Multi-AZ는 동기식 복제(Synchronous Replication)를 사용합니다.

---

**문제 4.** 읽기 쿼리 부하가 높을 때 가장 적합한 해결책은?

A) Multi-AZ 활성화  
B) 인스턴스 크기 증가  
C) Read Replica 추가  
D) 백업 빈도 낮추기  

**정답: C** - Read Replica를 추가하면 읽기 트래픽을 분산하여 성능을 향상할 수 있습니다.

---

**문제 5.** RDS Multi-AZ Standby 인스턴스의 특징은?

A) 읽기 쿼리 처리 가능  
B) 장애 시 자동으로 Primary 역할 수행  
C) 별도의 DNS 엔드포인트 제공  
D) 수동으로 Failover 수행 필요  

**정답: B** - Multi-AZ Standby는 Primary 장애 시 자동으로 Primary 역할을 수행합니다.

---

## 📌 오늘의 요약

1. RDS: 완전 관리형 관계형 DB, MySQL/PostgreSQL/MariaDB/Oracle/SQL Server/Aurora 지원
2. Multi-AZ: 동기 복제, 자동 Failover, 고가용성 목적 (Standby는 읽기 불가)
3. Read Replica: 비동기 복제, 읽기 확장, 최대 15개, Cross-Region 가능
4. Multi-AZ와 Read Replica는 함께 사용 가능 (고가용성 + 읽기 확장)
5. Failover 시 DNS 엔드포인트는 동일 → 애플리케이션 변경 불필요
