# 🤖 AI 활용 프로젝트 포트폴리오

이세령 | Backend / Full-stack Engineer  
GitHub: https://github.com/Hediar  
Blog: https://velog.io/@hediar


---

## 0. AI 활용에 대한 기본 철학

저는 AI를 “결과를 대신 만들어주는 도구”가 아닌, 개발 생산성과 설계 정확도를 높이는 협업 도구로 활용합니다.

### 공통적인 AI 활용 방식

- **요구사항 정리 (Human)**
  - 기획자 / 팀원과 기능 범위·애매한 지점 먼저 합의
- **설계 정리 (Human + Tool)**
  - Text 명세 + diagram 도구(dbdiagram.io, FigJam 등)
- **AI 지시 (Prompt Engineering)**
  - 최대한 명확하고 제약이 많은 요구
  - 기능 단위를 잘게 쪼갠 후, 중요도가 높은 요구사항부터 개발 진행
- **결과 검증 (Human)**
  - 코드 리뷰, 로그 포인트 삽입, 예상 플로우 검증
- **수정 및 고도화**
  - AI 초안 → 직접 리팩토링
- **Git 단위 관리**
  - 기능 단위 커밋으로 언제든 롤백 가능하게 유지

### 사용 도구 선택 기준

- **Codex (OpenAI 계열)**
  - 코드 맥락 이해, 텍스트 기반 요구사항 해석에 강점
- **Claude**
  - Codex와 같은 요구를 했을 때, 다른점을 비교하기 위해 사용

> “명확하게 요구할수록, 결과는 예상에서 벗어나지 않는다” 라고 생각합니다.

---

## 1. AI를 활용한 문제 해결 / 자동화 사례

### 📌 경량 로그 수집 & 모니터링 시스템

#### 문제 상황
- 여러 서버를 운영하다 보니 로그 확인이 잦음
- 매번 서버에 접속해 파일을 열어봐야 했고, 로그에 익숙하지 않은 사람에게는 부담이 큼
- 결국 일부 인원만 로그를 확인하는 구조
- 누구나 접근 가능한 경량 로그 모니터링 도구가 필요함

#### AI 활용 방법 (How)

1. **요구사항을 텍스트로 명확히 정의**
   - 로그 양은 많지 않음
   - 운영 환경 안정성이 최우선(테스트 서버에서 먼저 확인)
   - 배포/파일 이동 시 단일 전송 방식 선호
   - 중복 데이터 유입 방지 필요

2. **AI에게 “설계 초안” 요청**
   - 기존 구조(cron + scp + Opensearch)를 설명
   - 대체 가능한 방식 제안 요청
   - AI가 SFTP 단일 방식을 제안

3. **AI 제안 검증 & 채택**
   - scp / rsync 대비
   - 포트 관리 명확
   - 인증 구조 단순
   - 운영 환경 일관성 확보

#### 구현 상세

- **수집 구조**
  - cron ingest job (쓰기)
  - API 서버 (읽기) 분리

- **중복 방지**
  - 로그 _id를 hash 기반으로 생성
  - 동일 로그 재수집 방지

- **저장소**
  - OpenSearch

#### 개선 결과

| 항목 | 개선 전 | 개선 후 |
|------|---------|---------|
| 로그 전송 방식 | scp / rsync 실패 시 수동 처리 | scp / rsync 실패 시 SFTP fallback |
| 수집 방식 | cron 단독 | cron + ingest job 분리 |
| 중복 데이터 | 수동 관리 | Hash 기반 차단 |

#### 기여도
- 아키텍처 설계: 100%
- AI 활용: js 스크립트 구현

### 📌 대용량 데이터 수정 자동화

#### 문제 상황
- 고객 요청으로 대용량 데이터 수정이 필요
- 특정 조건으로 파일을 분할해야 함
- 사진들이 규칙적으로 들어가도록 형식을 정의해야 함

#### AI 활용 방법 (How)
1. **요구사항을 구조화**
   - 분할 기준(조건), 파일명 규칙, 사진 배치 규칙을 문서화
   - 입력/출력 포맷을 명확히 정의

2. **AI에게 자동화 설계/초안 요청**
   - 조건 분기 로직과 파일 분할 전략 제안 요청
   - JS로 재현 가능한 처리 파이프라인 설계

3. **검증 및 수정**
   - 샘플 데이터로 결과 검증
   - 예외 케이스(누락/중복/순서 오류) 기준 보완

#### 구현 상세
- **처리 흐름**
  - 입력 데이터 파싱 → 조건 분류 → 파일 분할 → 사진 규칙 적용 → 결과 출력
- **규칙 적용**
  - 사진 순서/위치 규칙을 명확히 정의하고 자동 삽입

#### 결과
- 대용량 수정 요청을 반복 가능한 자동화 파이프라인으로 전환
- 파일 분할 기준과 산출물 형식이 표준화되어 재작업 감소

#### 기여도
- 요구사항 정의: 100%
- 자동화 설계: Human 80% + AI 20%
- 구현: AI 초안 80% -> Human 리팩토링 20% 
- 검증: 100%


## 2. AI로 생성한 초안 vs 최종 산출물

### 📌 불도저(BDZ) 프로젝트 – ERD 설계 자동화

#### 배경
- 팀원/기획자와 기능 정의 문서 먼저 정리
- 애매한 정책은 사전 조율 후 확정
- 이후 AI를 활용해 dbdiagram.io ERD 문서 생성

#### 사전 정리한 요구사항 (축약된 내용 예시)

```
공통
- id (PK)
- 생성일, 수정일 (Nullable)

User
- 이메일, 비밀번호, 이름, 전화번호
- 역할, 차번호, 프로필 이미지
- 활성화 여부, 이메일 인증 여부

알림
- 사용자 ID(FK)
- 알림 종류, 메시지, 상태, 발송 시간

예약
- 시작/종료 시간
- 차량 번호
- 예약 상태

결제
- 결제 수단, 상태, PG 트랜잭션
- 결제 금액, 실패 사유
```


#### AI 활용 방법

1. **명확한 프롬프트**
   - 위 요구사항을 기준으로
   - dbdiagram.io 문법 사용
   - 테이블 간 관계 명확히
   - 공통 컬럼 재사용
   - 로그 수집 DB 고려

2. **AI가 생성한 ERD 초안**
   - 전체 테이블 구조 자동 생성
   - FK 관계 자동 연결

#### 문제점 (AI 초안)
- 일부 관계 누락
- 예약/결제 흐름이 실제 비즈니스와 불일치
- Nullable 정책 미흡
- 요구하지 않은 범위까지 포함하는 과설계 발생

#### 최종 수정 (Human)
- 관계 재정의
- 정책 컬럼 및 추가 기능에 따른 테이블 추가
- 실제 API 흐름 기준으로 테이블 분리
- 설계 최적화

#### 최종 결과물 (DBML 축약)

```dbml
// BDZ_ERD (DBML, shortened)
Table users {
  id bigint [pk, increment]
  email varchar(255) [not null, unique]
  password varchar(255) [not null]
  name varchar(100)
  phone varchar(30)
  role varchar(50)
  is_active boolean [default: true]
  is_email_verified boolean [default: false]
  created_at timestamp
  updated_at timestamp
}


Table notifications {
  id bigint [pk, increment]
  receiver_user_id bigint [not null, ref: > users.id]
  type varchar(50)
  status varchar(20)
  sent_at timestamp
  target_type varchar(50)
  target_id bigint
  created_at timestamp
  updated_at timestamp
}

Table reservations {
  id bigint [pk, increment]
  user_id bigint [not null, ref: > users.id]
  parking_lot_id varchar(20) [not null, ref: > parking_lot.id]
  car_id bigint [ref: > cars.id]
  start_time timestamp [not null]
  end_time timestamp [not null]
  status varchar(30)
  created_at timestamp
  updated_at timestamp
}

Table payments {
  id bigint [pk, increment]
  user_id bigint [not null, ref: > users.id]
  reservation_id bigint [not null, ref: > reservations.id]
  amount int [not null]
  status varchar(30)
  pg_transaction_id varchar(200)
  created_at timestamp
  updated_at timestamp
}

Table payment_events {
  id bigint [pk, increment]
  payment_id bigint [not null, ref: > payments.id]
  event_type varchar(50)
  occurred_at timestamp [not null]
  created_at timestamp
  updated_at timestamp
}


Table parking_fee_policy {
  parking_lot_id varchar(20) [pk, ref: > parking_lot.id]
  date_type char(3) [not null]
  base_fee int [not null]
  add_fee int [not null]
  created_at timestamp
  updated_at timestamp
}


Table recommendation_results {
  id bigint [pk, increment]
  request_id bigint [not null, ref: > recommendation_requests.id]
  parking_lot_id varchar(20) [not null, ref: > parking_lot.id]
  rank int [not null]
  created_at timestamp
  updated_at timestamp
}

Table recommendation_events {
  id bigint [pk, increment]
  user_id bigint [not null, ref: > users.id]
  result_id bigint [not null, ref: > recommendation_results.id]
  event_type varchar(30) [not null]
  reservation_id bigint [ref: > reservations.id]
  occurred_at timestamp [not null]
  created_at timestamp
  updated_at timestamp
}

Table conversations {
  conversation_id varchar(64) [pk]
  user_id bigint [not null, ref: > users.id]
  created_at timestamp
  ended_at timestamp
}

```

#### 기여도
- 요구사항 정의: 100%
- ERD 구조 설계: Human 80% + AI 20%
- 최종 검증 및 수정: 100%


## 3. AI의 한계를 겪고 해결한 경험

### 📌 Viewtrack 백엔드 NestJS 마이그레이션

#### 문제 상황
- 기존 프로젝트 구조 유지 필요
- JS → TS 마이그레이션
- 타입 관리가 핵심(타입 에러 방지)
- OpenSearch + Redis(시계열) 기반 데이터 조회 기능 필요

#### AI 활용 방식

1. **기존 소스코드를 제공**
   - “전체 재작성 금지”
   - “기존 구조 유지”
   - “기능별 폴더 분리”

2. **AI가 제안한 초기 구조**
   - NestJS 모듈 분리
   - Service / Controller / Module 구조

#### AI 한계
- 실제 런타임 의존성 고려 부족

#### 해결 과정
- 실제 서비스 구조 연결 초안 작성
- OpenSearch 쿼리는 공식 문서와 기존 결과와 동일하도록 재작성
- 핵심 기능 조회 API는 AI 활용

#### 결과
- 점진적 마이그레이션 성공
- 가장 중요한 장치 데이터 조회 기능 안정화

#### 기여도
- AI 활용: 구조 초안 & 코드 참고
- 최종 구현 및 안정화: 100%


## 마무리

- AI는 설계 보조 및 구현
- 최종 책임은 사람
- 검증 가능한 구조와 로그, git 형상관리 중요
