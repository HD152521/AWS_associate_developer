# Day 33 - ElastiCache: Redis, Memcached, 캐싱 전략

📅 날짜: 2026년 6월 30일 (화요일)  
🎯 주제: Amazon ElastiCache  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- ElastiCache Redis와 Memcached의 차이를 이해한다
- 적합한 캐싱 전략을 선택한다
- ElastiCache 사용 패턴을 파악한다

---

## 📖 이론 내용

### 1. Amazon ElastiCache란?

완전 관리형 인메모리 캐시 서비스로, 데이터베이스 부하를 줄이고 응답 속도를 높입니다.

**두 가지 엔진:**
- **Redis**: 고급 데이터 구조, 영속성, 클러스터
- **Memcached**: 단순한 캐시, 멀티스레드

### 2. Redis vs Memcached

| 특성 | Redis | Memcached |
|------|-------|-----------|
| 데이터 구조 | String, Hash, List, Set, ZSet | String만 |
| 영속성 | 지원 (AOF, RDB) | 미지원 |
| Multi-AZ | 지원 | 미지원 |
| Read Replica | 지원 | 미지원 |
| 백업/복원 | 지원 | 미지원 |
| 클러스터 | 지원 | 지원 |
| 멀티스레드 | 단일 스레드 | 멀티스레드 |
| 용도 | 세션, 순위표, 복잡한 캐싱 | 단순 캐시, 수평 확장 |

**⭐ 시험에서 "영속성, 백업, Multi-AZ" → Redis 선택**

### 3. 캐싱 전략

**Lazy Loading (Cache-Aside, 조회 시 채우기):**

```python
import redis
import boto3

cache = redis.Redis()
db = boto3.resource('dynamodb')

def get_user(user_id):
    # 1. 캐시 확인
    cached = cache.get(f'user:{user_id}')
    if cached:
        return json.loads(cached)  # Cache Hit!
    
    # 2. Cache Miss - DB 조회
    table = db.Table('Users')
    user = table.get_item(Key={'userId': user_id})['Item']
    
    # 3. 캐시에 저장 (TTL: 1시간)
    cache.setex(f'user:{user_id}', 3600, json.dumps(user))
    
    return user
```

**Write-Through (쓸 때 캐시 업데이트):**

```python
def update_user(user_id, data):
    # DB에 쓰기
    table.put_item(Item=data)
    
    # 캐시도 동시에 업데이트
    cache.setex(f'user:{user_id}', 3600, json.dumps(data))
```

**전략 비교:**

| 전략 | 장점 | 단점 |
|------|------|------|
| Lazy Loading | 필요한 데이터만 캐시, 간단 | 첫 요청 느림 (Cache Miss) |
| Write-Through | 항상 최신 데이터 | 모든 쓰기에 캐시 업데이트 비용 |

### 4. ElastiCache 세션 관리

```python
# Lambda에서 세션 데이터 저장
import redis

cache = redis.Redis(host='my-elasticache.cache.amazonaws.com')

def handle_login(user_id):
    session_id = generate_session_id()
    session_data = {'userId': user_id, 'loginTime': time.time()}
    
    # 세션 저장 (30분 TTL)
    cache.setex(f'session:{session_id}', 1800, json.dumps(session_data))
    
    return session_id
```

---

## 아키텍처 다이어그램

```
ElastiCache 캐싱 아키텍처
================================

[클라이언트]
      |
      v
[애플리케이션 서버]
      |
      |-- 1. 캐시 조회 --> [ElastiCache Redis]
      |         |
      |    Cache Hit  <-- 캐시된 데이터 반환
      |    (빠른 응답)
      |
      |-- Cache Miss --> [RDS Database]
                  |
                  | 데이터 조회 후 캐시 저장
                  v
          [ElastiCache Redis] (TTL 설정)

Redis 클러스터 구성
================================

[Primary Node] <-- 쓰기
      |
      | 복제
      v
[Read Replica] <-- 읽기
[Read Replica] <-- 읽기

Multi-AZ 활성화 시 자동 Failover
```

---

## ⭐ 핵심 포인트

1. ⭐ **Redis vs Memcached**: Redis는 영속성/백업/Multi-AZ 지원, Memcached는 멀티스레드
2. ⭐ **Lazy Loading**: Cache Miss 시 DB 조회 후 캐시 저장
3. ⭐ **Write-Through**: DB 쓰기와 동시에 캐시 업데이트
4. ⭐ **세션 관리**: ElastiCache로 서버 간 세션 공유
5. ⭐ **TTL**: 캐시 만료 시간 설정으로 오래된 데이터 자동 삭제

---

## 📝 연습 문제

**문제 1.** Redis와 Memcached의 가장 큰 차이점은?

A) 속도 차이  
B) Redis는 영속성과 백업 지원, Memcached는 미지원  
C) 비용 차이  
D) 최대 메모리 크기  

**정답: B** - Redis는 영속성, 백업, Multi-AZ를 지원하지만 Memcached는 단순 캐시 기능만 제공합니다.

---

**문제 2.** Lazy Loading의 단점은?

A) 모든 데이터가 캐시에 저장됨  
B) 첫 번째 요청 시 캐시 미스로 느린 응답  
C) 데이터 일관성 문제  
D) 구현 복잡도  

**정답: B** - Lazy Loading은 Cache Miss 시 DB를 조회해야 하므로 첫 번째 요청이 느릴 수 있습니다.

---

**문제 3.** 순위표(Leaderboard) 구현에 적합한 ElastiCache 엔진은?

A) Memcached  
B) Redis  
C) 둘 다 가능  
D) 둘 다 불가  

**정답: B** - Redis의 Sorted Set(ZSet)은 순위표 구현에 최적화되어 있습니다.

---

**문제 4.** Write-Through 전략의 장점은?

A) 쓰기 성능 향상  
B) 항상 최신 데이터가 캐시에 존재  
C) 캐시 용량 절감  
D) 구현 단순성  

**정답: B** - Write-Through는 쓰기 시 캐시도 업데이트하므로 캐시에 항상 최신 데이터가 유지됩니다.

---

**문제 5.** 여러 서버 간 세션 공유에 가장 적합한 서비스는?

A) RDS  
B) S3  
C) ElastiCache  
D) DynamoDB  

**정답: C** - ElastiCache는 인메모리 기반으로 빠른 세션 저장/조회가 가능하여 서버 간 세션 공유에 적합합니다.

---

## 📌 오늘의 요약

1. ElastiCache: 완전 관리형 인메모리 캐시, Redis와 Memcached 지원
2. Redis: 영속성/백업/Multi-AZ/복잡한 데이터 구조, 세션/순위표에 적합
3. Memcached: 단순 캐시, 멀티스레드, 영속성 없음
4. Lazy Loading: Cache Miss 시 DB 조회 후 캐시 저장 (가장 일반적)
5. Write-Through: DB 쓰기와 동시에 캐시 업데이트 (항상 최신 데이터)
