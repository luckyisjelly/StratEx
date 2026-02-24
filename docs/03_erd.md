# Phase 0 - ERD 설계

본 문서는 StratEx 거래소의 데이터베이스 구조를 정의한다.  
거래소 핵심 도메인(사용자, 계좌, 주문, 체결, 전략 등)을 기준으로 설계하였다.

---

## 1. users

사용자 기본 정보

- id (PK, bigint)
- email (varchar, unique)
- password (varchar, nullable)
- role (varchar)
- created_at (timestamp)
- updated_at (timestamp)

---

## 2. auth_identities

OAuth 계정 연동 정보

- id (PK)
- user_id (FK → users.id)
- provider (KAKAO / LOCAL)
- provider_user_id (varchar)
- created_at

---

## 3. accounts

사용자 자산 관리 테이블

- id (PK)
- user_id (FK → users.id)
- asset (varchar)  // KRW, BTC 등
- available (DECIMAL(19, 8))
- locked (DECIMAL(19, 8))
- created_at
- updated_at

### 제약 조건

- (user_id, asset) unique
- available + locked = 총 자산
- 음수 값 허용 금지

---

## 4. markets

거래 마켓 정보

- id (PK)
- base_asset (varchar)
- quote_asset (varchar)
- created_at

예:
BTC / KRW

---

## 5. orders

주문 정보

- id (PK)
- user_id (FK → users.id)
- market_id (FK → markets.id)
- side (BUY / SELL)
- price (DECIMAL(19, 3))
- quantity (DECIMAL(19, 8))
- filled_quantity (DECIMAL(19, 8))
- status (OPEN / PARTIALLY_FILLED / FILLED / CANCELED / REJECTED)
- created_at
- updated_at

### 인덱스

- market_id + price (정렬용)
- user_id (사용자 주문 조회용)
- status (OPEN 필터링용)

---

## 6. trades

체결 기록

- id (PK)
- market_id (FK → markets.id)
- buy_order_id (FK → orders.id)
- sell_order_id (FK → orders.id)
- price (DECIMAL(19, 3))
- quantity (DECIMAL(19, 8))
- executed_at (timestamp)

### 인덱스

- market_id + executed_at
- buy_order_id
- sell_order_id

---

## 7. reference_prices

외부 참고 시세 캐시

- id (PK)
- market_id (FK → markets.id)
- price (DECIMAL(19, 3))
- updated_at

(운영 시 Redis 캐시 우선 사용 가능)

---

## 8. strategies

조건식 자동 매매 전략

- id (PK)
- user_id (FK → users.id)
- market_id (FK → markets.id)
- type (RSI / VOLUME / BREAKOUT)
- parameters (JSON)
- is_active (boolean)
- created_at
- updated_at

---

## 9. comments

코인 커뮤니티 댓글

- id (PK)
- user_id (FK → users.id)
- market_id (FK → markets.id)
- content (text)
- created_at
- updated_at

---

## 설계 원칙

- 금액 및 수량은 BigDecimal 기반
- float / double 사용 금지
- 모든 체결 및 잔고 변경은 트랜잭션 처리
- Price Band 및 Tick Size는 서비스 레벨에서 검증