# SPOTL

> 소규모 공연 주최자를 위한 공연 등록·예약·관객 관리 백엔드

SPOTL은 대학 동아리, 독립 밴드, 연극팀 등 소규모 공연 주최자가 공연 정보를 등록하고 관객의 예약 현황을 관리할 수 있도록 개발한 서비스입니다.

기존에 구글 폼, 계좌이체 내역, 수기 명단 등으로 분산되어 있던 공연 운영 과정을 하나의 시스템으로 관리하는 것을 목표로 했습니다.

> 이 README는 현재 저장소의 백엔드 구현을 기준으로 작성했습니다. QR 티켓 발행·스캔, 자동 입금 확인 등 아직 코드로 구현되지 않은 기능은 구현 기능 목록에서 제외했습니다.

---

## 프로젝트 개요

| 항목 | 내용 |
| --- | --- |
| 프로젝트명 | SPOTL |
| 개발 기간 | 2024.09.09 ~ 2024.12.10 |
| 주요 사용자 | 대학 동아리, 독립 밴드, 연극팀 등 소규모 공연 주최 단체 |
| 백엔드 환경 | Java 17, Spring Boot 3.3.3, MySQL |
| 배포 환경 | CloudType |
| 외부 서비스 | AWS S3, CoolSMS, Naver Map API |

---

## 문제 정의

소규모 공연은 전문 예매 시스템 대신 구글 폼과 수기 명단에 의존하는 경우가 많습니다.

이 과정에서 다음과 같은 문제가 발생할 수 있습니다.

- 관객 정보와 예약 현황이 여러 문서에 분산됨
- 동일 사용자의 중복 예약을 수동으로 확인해야 함
- 공연 포스터와 상세 정보를 일관된 형식으로 관리하기 어려움
- 공연 공개 전 운영자의 검토 절차가 없음
- 회원가입 과정에서 휴대전화 본인 확인이 어려움

SPOTL 백엔드는 사용자 인증, 공연 게시물, 관리자 승인, 공연 예약 기능을 API로 제공하여 이러한 운영 과정을 구조화하는 것을 목표로 합니다.

---

## 현재 구현된 백엔드 기능

### 1. 사용자 인증

- 휴대전화 번호 기반 회원가입
- BCrypt를 이용한 비밀번호 암호화
- 휴대전화 번호와 비밀번호를 이용한 로그인
- Spring Security와 JWT를 이용한 Access Token 발급
- JWT에 사용자 ID와 권한 정보 저장

### 2. SMS 본인 인증

- CoolSMS API를 이용한 인증번호 발송
- 전화번호별 인증번호 저장 및 검증
- 가장 최근에 발급된 인증번호를 기준으로 검증
- 인증번호 발송 후 3분 동안 재발송 제한
- 인증 성공 시 기존 사용자 정보 반환

### 3. 공연 게시물 관리

- 공연 제목, 장소, 주소, 날짜, 설명, 유의사항 및 정보 링크 등록
- 공연 포스터 이미지 선택 업로드
- 공연 게시물 상세 조회
- 승인된 공연 게시물 전체 조회
- 공연 게시물 수정 및 삭제
- 관리자의 공연 게시물 승인 처리

### 4. 공연 예약 관리

- 공연 ID와 사용자 ID를 이용한 예약 생성
- 사용자와 공연 존재 여부 검증
- 동일 사용자의 취소되지 않은 동일 공연 예약 검사
- 사용자별 활성 예약 목록 조회
- 예약 시간, 입금 여부, 확정 여부 및 취소 여부 저장

### 5. 파일 및 외부 API 연동

- AWS S3에 공연 포스터 이미지 업로드
- CoolSMS API를 통한 문자 인증
- Naver Map API 기반 장소 검색 기능 구현 이력
- Swagger/OpenAPI 기반 API 명세 작성

---

## 서비스 흐름

![서비스 플로우](https://github.com/user-attachments/assets/72bf1865-5b92-4175-a31c-75de5f27ae59)

1. 사용자는 휴대전화 인증 후 회원가입을 진행합니다.
2. 로그인에 성공하면 서버가 JWT Access Token을 발급합니다.
3. 공연 주최자는 공연 정보와 포스터 이미지를 등록합니다.
4. 등록된 공연은 관리자의 승인을 받은 후 조회 대상에 포함됩니다.
5. 사용자는 공연을 예약할 수 있습니다.
6. 서버는 사용자·공연 존재 여부와 기존 예약을 확인한 후 예약을 저장합니다.

---

## 백엔드 구조

```text
Client
  │
  ▼
Controller
  │  HTTP 요청 수신 및 API 응답 생성
  ▼
Service
  │  인증, 예약 검증, 파일 업로드 등 비즈니스 로직
  ▼
Repository
  │  Spring Data JPA
  ▼
MySQL
```

### 패키지 역할

| 패키지 | 역할 |
| --- | --- |
| `Controller` | 인증, 공연 게시물, 공연 예약 API 제공 |
| `Service` | 로그인, SMS 인증, 게시물 승인, 예약 검증, S3 업로드 처리 |
| `Repository` | 사용자, 공연, 예약, SMS 인증 데이터 접근 |
| `Entity` | 사용자, 공연 게시물, 공연 예약 등 도메인 모델 정의 |
| `Dto` | 공연 예약 요청 데이터 전달 |
| `Jwt` | JWT 생성·검증 및 인증 객체 구성 |
| `Response` | 공통 성공·오류 응답과 서비스 응답 코드 정의 |
| `Config` | Spring Security 및 외부 서비스 설정 |

---

## 주요 구현 내용

### JWT 로그인

로그인 시 휴대전화 번호로 사용자를 조회하고 BCrypt로 비밀번호를 비교합니다.

인증에 성공하면 `CustomUser`와 `Authentication` 객체를 생성하고, 다음 정보를 포함한 JWT를 발급합니다.

- Subject: 사용자 휴대전화 번호
- `userId`: 사용자 식별자
- `auth`: 사용자 권한
- 만료 시간: 24시간

현재 구현은 Access Token만 발급하며 Refresh Token 재발급 구조는 포함하지 않습니다.

### SMS 재발송 제한

SMS 인증번호 발송 전 마지막 발송 시간을 확인하여 3분 이내의 재요청을 제한합니다.

이를 통해 반복 요청으로 인한 불필요한 문자 발송과 비용 발생을 줄이도록 했습니다.

### 관리자 승인 기반 공연 공개

공연 게시물은 등록 직후 전체 목록에 노출되지 않습니다.

관리자가 승인 API를 호출하면 `approvalStatus`가 `true`로 변경되고, 상세 및 전체 조회 API는 승인된 게시물만 반환합니다.

### 예약 중복 검사

예약 생성 전에 다음 순서로 데이터를 검증합니다.

1. 공연 ID와 사용자 ID의 null 여부 확인
2. 공연 존재 여부 확인
3. 사용자 존재 여부 확인
4. 사용자의 기존 예약 목록 조회
5. 동일 공연에 대해 취소되지 않은 예약이 있는지 확인
6. 예약 상태 초기화 후 저장

---

## API 요약

### 인증

| Method | Endpoint | 설명 |
| --- | --- | --- |
| `GET` | `/auth/{phoneNumber}/send-sms` | SMS 인증번호 발송 |
| `POST` | `/auth/verify-sms` | SMS 인증번호 검증 |
| `POST` | `/auth/signup` | 회원가입 |
| `POST` | `/auth/signin` | 로그인 및 JWT 발급 |

### 공연 게시물

| Method | Endpoint | 설명 |
| --- | --- | --- |
| `POST` | `/event-board/create` | 공연 게시물 등록 |
| `PUT` | `/event-board/{id}/approve` | 공연 게시물 관리자 승인 |
| `GET` | `/event-board/{id}` | 승인된 공연 상세 조회 |
| `GET` | `/event-board/viewAll` | 승인된 공연 전체 조회 |
| `PUT` | `/event-board/{id}/edit` | 공연 게시물 수정 |
| `DELETE` | `/event-board/{id}/delete` | 공연 게시물 삭제 |

### 공연 예약

| Method | Endpoint | 설명 |
| --- | --- | --- |
| `POST` | `/event-rsv/{id}/reserve` | 특정 공연 예약 생성 |
| `GET` | `/event-rsv/{userId}` | 사용자별 활성 예약 목록 조회 |

자세한 요청 및 응답 예시는 애플리케이션 실행 후 Swagger UI에서 확인할 수 있습니다.

---

## 공통 API 응답

API 응답은 `data`, `message`, `code`를 포함하는 공통 형식으로 구성했습니다.

```json
{
  "data": {
    "savedEventBoardId": 1
  },
  "message": "게시글 저장 성공 : 관리자의 승인이 필요합니다.",
  "code": "B002"
}
```

응답 코드 접두사는 도메인별로 구분합니다.

| 접두사 | 도메인 |
| --- | --- |
| `S` | SMS 인증 |
| `U` | 사용자 및 로그인 |
| `B` | 공연 게시물 |
| `R` | 공연 예약 |

---

## 기술 스택

### Backend

| 구분 | 기술 |
| --- | --- |
| Language | Java 17 |
| Framework | Spring Boot 3.3.3 |
| Web | Spring MVC |
| Security | Spring Security, JWT, BCrypt |
| Database | MySQL |
| ORM | Spring Data JPA |
| Validation | Jakarta Bean Validation |
| API Documentation | Springdoc OpenAPI, Swagger UI |
| File Storage | Amazon S3 |
| SMS | CoolSMS API |
| Map | Naver Map API |
| Build | Gradle |
| Deployment | CloudType |

---

## 데이터 모델

![역할 기반 UML 다이어그램](https://github.com/user-attachments/assets/4636fecf-5f09-4581-9101-348ff7185402)

```text
User 1 ───── N EventBoard
User 1 ───── N EventReservation
EventBoard 1 ───── N EventReservation
```

- 한 사용자는 여러 공연 게시물을 작성할 수 있습니다.
- 한 사용자는 여러 공연 예약을 가질 수 있습니다.
- 하나의 공연에는 여러 예약이 연결될 수 있습니다.
- 공연 예약은 입금, 확정 및 취소 상태를 관리합니다.

---

## 실행 환경

### 요구 사항

- Java 17
- MySQL
- Gradle
- AWS S3 Bucket
- CoolSMS API Key
- JWT Secret Key

### 실행

```bash
./gradlew bootRun
```

외부 서비스 키와 데이터베이스 인증정보는 저장소에 직접 커밋하지 않고 환경 변수 또는 별도의 설정 파일로 관리해야 합니다.

---

## 현재 코드에서 확인된 개선 과제

이 프로젝트는 학습 및 초기 서비스 구현 단계의 결과물이며, 다음 항목은 후속 개선이 필요합니다.

- 예약 저장 시 조회한 `User`와 `EventReservation`의 연관관계를 설정하도록 수정
- 공연 게시물 수정·삭제 로직을 Controller에서 Service 계층으로 이동
- `IllegalArgumentException` 하나로 처리되는 예약 오류를 도메인 예외로 분리
- `@RestControllerAdvice`를 이용해 예외 응답을 전역 처리
- Entity를 API 응답으로 직접 반환하지 않고 응답 DTO 사용
- JWT 토큰과 인증번호 등 민감정보의 콘솔 출력 제거
- 로그인 실패에 적절한 HTTP 상태 코드 적용
- 예약 중복 방지를 위한 데이터베이스 제약조건 및 트랜잭션 보강
- Repository 및 Service 계층 테스트 추가
- Refresh Token과 토큰 재발급 기능 도입
- CI/CD와 컨테이너 기반 배포 환경 구축

---

## 개발을 통해 배운 점

- 클라이언트 입력만 신뢰하지 않고 서버에서 사용자·공연·예약 상태를 다시 검증해야 데이터 정합성을 유지할 수 있음을 경험했습니다.
- SMS, S3와 같은 외부 서비스는 실패 가능성을 고려해 별도 오류 응답으로 처리해야 함을 학습했습니다.
- 인증, 공연, 예약 도메인별 응답 코드를 정의하면서 일관된 API 응답 구조의 필요성을 이해했습니다.
- 기능 구현뿐 아니라 계층 간 책임 분리와 도메인 예외 설계가 유지보수성에 중요하다는 점을 확인했습니다.
