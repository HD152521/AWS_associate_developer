# Day 19 - API Gateway: WebSocket API, HTTP API

📅 날짜: 2026년 6월 10일 (수요일)  
🎯 주제: WebSocket API 및 HTTP API  
⏱️ 학습 시간: 약 90분

---

## 🎯 학습 목표

- WebSocket API를 이용한 실시간 양방향 통신을 이해한다
- HTTP API와 REST API의 차이점을 비교한다
- WebSocket 연결 라우팅과 메시지 처리를 구현한다

---

## 📖 이론 내용

### 1. WebSocket API

WebSocket은 클라이언트와 서버 간의 지속적인 양방향 연결을 제공합니다.

**WebSocket vs REST:**
| 특성 | REST API | WebSocket API |
|------|----------|---------------|
| 연결 | 요청마다 새 연결 | 지속적 연결 |
| 방향 | 단방향 (요청-응답) | 양방향 |
| 사용 사례 | 일반 API | 실시간 채팅, 알림 |
| 효율성 | 낮음 (헤더 반복) | 높음 |

**WebSocket 라우트 키:**
- `$connect`: 연결 시 실행
- `$disconnect`: 연결 해제 시 실행
- `$default`: 매칭되지 않는 모든 메시지
- 사용자 정의: 특정 액션 처리

```python
# WebSocket Lambda 핸들러 예시
import json
import boto3

# API Gateway Management API 클라이언트
apigw = boto3.client('apigatewaymanagementapi',
    endpoint_url='https://abc123.execute-api.ap-northeast-2.amazonaws.com/prod')

def lambda_handler(event, context):
    route_key = event['requestContext']['routeKey']
    connection_id = event['requestContext']['connectionId']
    
    if route_key == '$connect':
        # 연결 ID를 DynamoDB에 저장
        save_connection(connection_id)
        return {'statusCode': 200}
    
    elif route_key == '$disconnect':
        # 연결 ID 삭제
        remove_connection(connection_id)
        return {'statusCode': 200}
    
    elif route_key == 'sendMessage':
        body = json.loads(event.get('body', '{}'))
        message = body.get('message', '')
        
        # 모든 연결된 클라이언트에게 브로드캐스트
        for conn_id in get_all_connections():
            try:
                apigw.post_to_connection(
                    ConnectionId=conn_id,
                    Data=json.dumps({'message': message}).encode()
                )
            except apigw.exceptions.GoneException:
                # 연결이 끊어진 경우
                remove_connection(conn_id)
        
        return {'statusCode': 200}
```

**서버에서 클라이언트로 메시지 전송:**
```python
# WebSocket 콜백 URL 형식:
# https://{api-id}.execute-api.{region}.amazonaws.com/{stage}/@connections/{connectionId}

endpoint = f"https://{api_id}.execute-api.{region}.amazonaws.com/{stage}"
client = boto3.client('apigatewaymanagementapi', endpoint_url=endpoint)

client.post_to_connection(
    ConnectionId=connection_id,
    Data=b'{"message": "서버에서 보낸 메시지"}'
)
```

### 2. HTTP API vs REST API

**HTTP API 장점:**
- REST API보다 **70% 저렴**
- 지연 시간 더 낮음
- CORS 자동 설정 지원
- JWT 자동 검증 지원 (Cognito, Auth0 등)

**HTTP API 제한:**
- API 키, 사용량 플랜 미지원
- X-Ray 트레이싱 미지원
- 리소스 정책 미지원
- 요청/응답 변환 제한적
- 캐싱 미지원

**언제 무엇을 사용할까:**
- **HTTP API**: 단순한 Lambda/HTTP 프록시, 저비용이 중요할 때
- **REST API**: 복잡한 기능(사용량 플랜, 캐싱, 변환) 필요 시

---

## 아키텍처 다이어그램

```
WebSocket 실시간 채팅 아키텍처
================================

[클라이언트 A]         [클라이언트 B]
    |                       |
    | ws://                 | ws://
    v                       v
[WebSocket API Gateway]
    |
    +--[$connect]--> [Lambda - 연결 관리]
    |                        |
    |               [DynamoDB - 연결 ID 저장]
    |                ConnectionId: conn_A
    |                ConnectionId: conn_B
    |
    +--[sendMessage]--> [Lambda - 메시지 처리]
    |                        |
    |               연결된 모든 ID 조회
    |                        |
    |               [API GW Management API]
    |               post_to_connection(conn_A)
    |               post_to_connection(conn_B)
    |
    +--[$disconnect]--> [Lambda - 연결 해제]

HTTP API vs REST API 비교
================================

                 HTTP API    REST API
비용:            더 낮음     높음
성능:            더 빠름     보통
API 키:          X           O
사용량 플랜:     X           O
캐싱:            X           O
요청 검증:       X           O
VTL 변환:        X           O
JWT 자동 검증:   O           X (Lambda 필요)
CORS 자동:       O           수동 설정
X-Ray:           X           O
```

---

## ⭐ 핵심 포인트 (시험 출제 빈도 높음)

1. ⭐ **WebSocket 라우트**: $connect, $disconnect, $default, 사용자 정의
2. ⭐ **서버→클라이언트 전송**: @connections/{connectionId} API 사용
3. ⭐ **HTTP API**: REST API보다 70% 저렴, 사용량 플랜/캐싱 미지원
4. ⭐ **연결 ID 저장**: $connect 시 DynamoDB에 저장, $disconnect 시 삭제
5. ⭐ **GoneException**: 이미 끊어진 연결에 메시지 전송 시 발생

---

## 📝 연습 문제

**문제 1.** WebSocket API에서 클라이언트가 처음 연결할 때 실행되는 라우트 키는?

A) $default  
B) $connect  
C) $disconnect  
D) $open  

**정답: B** - $connect 라우트는 클라이언트가 WebSocket 연결을 처음 수립할 때 Lambda 함수를 호출합니다.

---

**문제 2.** API Gateway WebSocket에서 서버가 클라이언트에게 메시지를 보내려면?

A) 클라이언트가 먼저 요청을 보내야 한다  
B) @connections/{connectionId} API를 사용하여 push 가능  
C) SNS 토픽을 통해 전달해야 한다  
D) WebSocket에서는 서버→클라이언트 전송이 불가능하다  

**정답: B** - API Gateway Management API의 @connections 엔드포인트를 사용하여 서버에서 특정 연결 ID의 클라이언트에게 메시지를 푸시할 수 있습니다.

---

**문제 3.** HTTP API를 사용해야 하는 적절한 상황은?

A) API 키와 사용량 플랜이 필요한 경우  
B) 요청/응답 변환이 필요한 경우  
C) 단순 Lambda 프록시가 필요하고 비용을 최소화해야 하는 경우  
D) X-Ray 트레이싱이 필요한 경우  

**정답: C** - HTTP API는 단순한 Lambda/HTTP 프록시에 적합하며 REST API보다 70% 저렴합니다.

---

**문제 4.** WebSocket 연결이 이미 끊어진 클라이언트에게 메시지 전송 시 발생하는 예외는?

A) ConnectionClosedException  
B) GoneException  
C) NotFoundException  
D) DisconnectedException  

**정답: B** - 이미 연결이 끊어진 ConnectionId로 메시지를 전송하면 GoneException이 발생합니다. 이 경우 DynamoDB에서 해당 연결 ID를 삭제해야 합니다.

---

**문제 5.** HTTP API와 REST API 모두 지원하는 기능은?

A) API 키  
B) 사용량 플랜  
C) Lambda 프록시 통합  
D) 응답 캐싱  

**정답: C** - Lambda 프록시 통합(AWS_PROXY)은 HTTP API와 REST API 모두 지원합니다.

---

## 📌 오늘의 요약

1. WebSocket API: 지속적 양방향 연결, $connect/$disconnect/$default 라우트로 관리
2. 연결 관리: $connect 시 DynamoDB에 연결 ID 저장, $disconnect 시 삭제
3. 서버→클라이언트: Management API의 @connections/{connectionId}로 푸시
4. HTTP API: REST API보다 70% 저렴, 단순 프록시에 적합 (사용량 플랜/캐싱 미지원)
5. 실시간 채팅, 게임, 협업 도구 등에 WebSocket API 활용
