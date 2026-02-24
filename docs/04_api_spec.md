# Phase 0 - API 명세서

본 문서는 StratEx의 REST API 명세를 정의한다.  
모든 응답은 JSON 형식이며, 공통 응답 구조를 따른다.

------------------------------------------------------------

1. 공통 응답 형식

성공 응답

{
"success": true,
"data": {},
"error": null
}

실패 응답

{
"success": false,
"data": null,
"error": {
"code": "ERROR_CODE",
"message": "에러 메시지"
}
}

------------------------------------------------------------

2. 인증 API

2.1 로그인

POST /auth/login

Request
{
"email": "user@example.com",
"password": "password"
}

Response
{
"accessToken": "jwt-token",
"refreshToken": "refresh-token"
}

2.2 토큰 재발급

POST /auth/refresh

Request
{
"refreshToken": "refresh-token"
}

2.3 로그아웃

POST /auth/logout

------------------------------------------------------------

3. 마켓 API

3.1 마켓 목록 조회

GET /markets

Response
[
{
"symbol": "BTC-KRW",
"internalPrice": 70000,
"referencePrice": 70200,
"premiumRate": -0.28
}
]

3.2 오더북 조회

GET /markets/{symbol}/orderbook

Response
{
"bids": [
{ "price": 69900, "quantity": 0.5 }
],
"asks": [
{ "price": 70100, "quantity": 0.3 }
]
}

3.3 최근 체결 내역 조회

GET /markets/{symbol}/trades

------------------------------------------------------------

4. 주문 API

4.1 주문 생성

POST /orders

Request
{
"symbol": "BTC-KRW",
"side": "BUY",
"price": 70000,
"quantity": 0.01
}

가능한 에러 코드
- INSUFFICIENT_BALANCE
- PRICE_BAND_EXCEEDED
- INVALID_TICK_SIZE

4.2 주문 취소

DELETE /orders/{orderId}

4.3 내 주문 조회

GET /me/orders

------------------------------------------------------------

5. 자산 API

5.1 자산 조회

GET /me/accounts

Response
[
{
"asset": "KRW",
"available": 300000,
"locked": 700000
}
]

------------------------------------------------------------

6. 전략 API

6.1 전략 생성

POST /me/strategies

Request
{
"symbol": "BTC-KRW",
"type": "RSI",
"parameters": {
"threshold": 60
}
}

6.2 전략 목록 조회

GET /me/strategies

6.3 전략 수정

PATCH /me/strategies/{id}

6.4 전략 삭제

DELETE /me/strategies/{id}

------------------------------------------------------------

7. 댓글 API

7.1 댓글 조회

GET /markets/{symbol}/comments

7.2 댓글 작성

POST /markets/{symbol}/comments

Request
{
"content": "댓글 내용"
}

7.3 댓글 삭제

DELETE /comments/{id}