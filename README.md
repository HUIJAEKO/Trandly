# 실시간 인기 상품 기반 쇼핑 플랫폼 - Trendly

사용자 행동을 기반으로 실시간 인기 상품을 반영하는 쇼핑 플랫폼

---

## 프로젝트 설명

Trendly는 사용자의 클릭 데이터를 기반으로 실시간 인기 상품을 집계하고,  
이를 메인 화면 및 카테고리별 추천에 반영하는 데이터 중심형 이커머스 플랫폼

Redis의 SortedSet 구조를 이용해 상품의 클릭 수를 기록하고,  
정렬된 순위 데이터를 통해 실시간 인기 상품 리스트를 제공

---

## 사용 기술 스택

### Backend
- Java 17
- Spring Boot 3.4.3
- Spring Security (JWT 기반 인증)
- Spring Data JPA (Hibernate)

### Database
- PostgreSQL
  - 회원, 상품, 장바구니, 주문, 카테고리 등 핵심 데이터 저장

### Redis
- `SortedSet` 자료구조 활용
  - 실시간 인기 상품 클릭 수 집계 및 정렬

### AWS S3
- 상품 이미지 업로드/조회 처리
- URL 기반 접근 방식

### 기타
- Spring Mail (이메일 인증)
- Docker

---

## ERD
[erd](/images/trandly_erd.png)

---

## 주요 기능

### [회원]
- 회원가입
  - 이메일 중복 확인
  - 이메일 인증 (코드 발송 및 검증)
- 로그인
  - JWT 토큰 발급
- 비밀번호 재설정
  - 이메일 인증 후 비밀번호 변경

---

### [상품]
- 상품 등록 / 수정 / 삭제 (관리자 전용)
  - 이름, 설명, 가격, 재고, 이미지(S3), 카테고리
  - 카테고리는 서비스에서 정의한 기본값 제공 + 관리자 확장 가능
- 전체 상품 조회 및 카테고리별 조회
  - 정렬 옵션: 최신순 / 가격순 / 인기순
- 상품 상세 페이지
  - 클릭 시 Redis에 클릭 수 +1 반영
  - 인기 랭킹에 자동 반영 (실시간)

---

### [장바구니]
- 상품 장바구니에 담기
- 장바구니 목록 보기
- 장바구니 항목 삭제

> 장바구니는 결제 전까지 Redis 캐시 기반으로 관리, 결제 시점에 DB에 영구 저장

> 장바구니의 경우 지워져도 치명적이지 않은 데이터이기 때문에, 초기에는 Redis에만 저장하고 추후 DB 연동으로 확장 예정

---

### [주문]
- 장바구니 기반 결제
  - 결제 전 상품 및 재고 검증
  - 주문 생성 및 결제 요청
  - 결제 완료 후 → 주문 확정 및 DB 저장
- 결제 기능
  - 카카오페이 API 연동 예정
  - 초기 단계에서는 모의 결제 흐름으로 개발
- 주문 내역 조회

#### 재고 감소 및 동시성 처리
> 여러 사용자가 동시에 동일 상품을 결제하는 상황을 고려하여, 재고의 무결성과 동시성 보장

> 초반에는 DB 락 기반으로 개발한 뒤. 트래픽이 많아짐을 가정하여 추후에 Redis Lua Script 방식으로 확장 예정

---

### [실시간 인기 상품]

- 상품 상세 페이지 진입 시, Redis SortedSet에 클릭 수 +1 저장
- 인기순 정렬 기반으로 상품 리스트 제공 (예: 인기 상품 Top 10)

#### 백업 전략
- 1일 2회 스케줄러로 Redis 데이터를 정렬된 상태로 DB에 저장
- Redis 초기화 발생 시, DB 백업 데이터를 기반으로 초기 값 복구

> 클릭 수는 Redis를 통해 실시간 처리  
