# Phase 0 - WebSocket 이벤트 명세

본 문서는 StratEx의 실시간 통신(WebSocket) 이벤트 구조를 정의한다.  
모든 실시간 데이터는 서버를 단일 진실 소스(Single Source of Truth)로 하여 전송된다.

------------------------------------------------------------

1. 연결 정보

Endpoint:
- ws://{domain}/ws

인증 방식:
- JWT Access Token을 헤더 또는 쿼리 파라미터로 전달

------------------------------------------------------------

2. 공통 이벤트 형식

{
"event": "EVENT_NAME",
"data": {}
}

------------------------------------------------------------

3. 마켓 실시간 이벤트

3.1 PRICE_UPDATE

설명:
- 현재가 및 Reference Price 변경 시 전송

Payload 예시:

{
"event": "PRICE_UPDATE",
"data": {
"symbol": "BTC-KRW",
"internalPrice": 70000,
"referencePrice": 70200,
"premiumRate": -0.28
}
}

------------------------------------------------------------

3.2 ORDERBOOK_UPDATE

설명:
- 호가 변경 시 전송

Payload 예시:

{
"event": "ORDERBOOK_UPDATE",
"data": {
"symbol": "BTC-KRW",
"bids": [
{ "price": 69900, "quantity": 0.5 }
],
"asks": [
{ "price": 70100, "quantity": 0.3 }
]
}
}

------------------------------------------------------------

3.3 TRADE_EXECUTED

설명:
- 체결 발생 시 전송

Payload 예시:

{
"event": "TRADE_EXECUTED",
"data": {
"symbol": "BTC-KRW",
"price": 70000,
"quantity": 0.01,
"side": "BUY",
"executedAt": "2024-01-01T12:00:00"
}
}

------------------------------------------------------------

4. 사용자 전용 이벤트

4.1 USER_ORDER_UPDATE

설명:
- 사용자의 주문 상태 변경 시 전송

Payload 예시:

{
"event": "USER_ORDER_UPDATE",
"data": {
"orderId": 123,
"status": "PARTIALLY_FILLED",
"filledQuantity": 0.005
}
}

------------------------------------------------------------

4.2 USER_BALANCE_UPDATE

설명:
- 체결 또는 취소로 인해 잔고 변경 시 전송

Payload 예시:

{
"event": "USER_BALANCE_UPDATE",
"data": [
{
"asset": "KRW",
"available": 300000,
"locked": 0
},
{
"asset": "BTC",
"available": 0.01,
"locked": 0
}
]
}

------------------------------------------------------------

5. 전략 관련 이벤트

5.1 STRATEGY_TRIGGERED

설명:
- 조건식 자동 매매 전략이 발동된 경우 전송

Payload 예시:

{
"event": "STRATEGY_TRIGGERED",
"data": {
"strategyId": 10,
"symbol": "BTC-KRW",
"triggerType": "RSI",
"orderId": 456
}
}

------------------------------------------------------------

6. 시스템 이벤트

6.1 SYSTEM_NOTICE

설명:
- Reference Price 수집 실패 등 시스템 상태 알림

Payload 예시:

{
"event": "SYSTEM_NOTICE",
"data": {
"message": "Reference price temporarily unavailable."
}
}

------------------------------------------------------------

7. 설계 원칙

- 모든 가격 계산은 서버에서 수행
- 클라이언트는 전달받은 데이터만 표시
- 실시간 데이터는 DB 또는 캐시 상태와 일관성을 유지
- 사용자 전용 이벤트는 인증된 연결에서만 수신 가능