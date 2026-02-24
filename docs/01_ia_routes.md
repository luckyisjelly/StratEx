# Phase 0 - IA 및 라우팅 구조 정의

본 문서는 StratEx의 화면 구조(IA: Information Architecture) 및 라우팅 설계를 정의한다.

---

## 1. 전체 화면 구조
/
└── /markets/{symbol}
├── trade
├── info
└── community

/login
/signup
/me/assets
/me/strategies

---

## 2. 페이지별 상세 정의

### 2.1 로그인 페이지

경로:
/login

기능:
- 자체 로그인 (email/password)
- 카카오 OAuth 로그인

필요 API:
- POST /auth/login
- GET /auth/kakao/callback

---

### 2.2 메인 페이지 (코인 목록)

경로:
/

표시 정보:
- 코인 목록
- 현재가 (Internal Price)
- 해외 참고 시세 (Reference Price)
- 괴리율
- 24h 변동률 (선택)

필요 API:
- GET /markets
- GET /markets/{symbol}/price

WebSocket:
- PRICE_UPDATE

---

### 2.3 코인 상세 페이지

경로:
/markets/{symbol}

기본 구조:
- 상단: 현재가 / Reference Price / 괴리율
- 탭:
    - trade
    - info
    - community

---

## 3. 매매 탭 (trade)

경로:
/markets/{symbol}?tab=trade

표시 정보:
- 현재가 (Internal Price)
- 해외 참고 시세 (Reference Price)
- 허용 가격 범위 (Price Band)
- 호가창 (상위 N개)
- 최근 체결 내역
- 매수 / 매도 입력창
- 내 주문 목록

필요 API:
- GET /markets/{symbol}/orderbook
- GET /markets/{symbol}/trades
- POST /orders
- DELETE /orders/{id}
- GET /me/orders

WebSocket 이벤트:
- ORDERBOOK_UPDATE
- TRADE_EXECUTED
- USER_ORDER_UPDATE

---

## 4. 코인 정보 탭 (info)

경로:
/markets/{symbol}?tab=info

표시 정보:
- 시가총액
- 발행량
- 코인 설명
- 가격 요약 정보

필요 API:
- GET /markets/{symbol}/info

---

## 5. 커뮤니티 탭 (community)

경로:
/markets/{symbol}?tab=community

기능:
- 댓글 목록 조회
- 댓글 작성
- 댓글 삭제 (작성자 본인만)

필요 API:
- GET /markets/{symbol}/comments
- POST /markets/{symbol}/comments
- DELETE /comments/{id}

---

## 6. 마이 자산 페이지

경로:
/me/assets

표시 정보:
- 보유 자산 목록
- available / locked 구분
- 총 자산 평가 금액

필요 API:
- GET /me/accounts

---

## 7. 전략 관리 페이지 (조건식 자동매매)

경로:
/me/strategies

기능:
- 전략 생성
- 전략 활성/비활성
- 전략 삭제
- 전략 목록 조회

필요 API:
- GET /me/strategies
- POST /me/strategies
- PATCH /me/strategies/{id}
- DELETE /me/strategies/{id}

---

## 8. 접근 제어 정책

- 로그인 없이 접근 가능:
    - 메인 페이지
    - 코인 상세 페이지 (조회만)

- 로그인 필요:
    - 주문 생성
    - 댓글 작성
    - 전략 생성
    - 자산 조회

---

## 9. 설계 원칙

- 실시간 데이터는 WebSocket 기반
- 가격/체결/주문 상태는 서버가 단일 진실 소스
- 계산 로직은 서버에서 수행