# REST API (전체 문서 제목)

## 목차
1. [REST API란?](#rest-api)  


# REST API

## 1. 개념
**API (Application Programming Interface)**
- "너가 이렇게 요청하면, 내가 이렇게 줄게"라고 미리 정해둔 약속(Interface).
- 클라이언트와 서버가 어떻게 요청하고 응답할지 정의한 명세.

**RESTful API**
- API의 다양한 형식 중 가장 널리 사용되는 아키텍처 스타일.
- **RE**presentational **S**tate **T**ransfer의 약자.
- 자원(Resource)을 이름(URI)으로 구분하여 해당 자원의 상태(State)를 주고받는 것.

## 2. 왜 인기 있는가? (CRUD 매핑)
앱이나 웹에서 데이터를 다루는 기본 동작인 **CRUD**를 HTTP Method(동사)에 매핑하여 사용하기 때문에 직관적임.

| HTTP Method | 역할 (CRUD) | 설명 | 멱등성(Idempotent) |
|:---:|:---:|:---|:---:|
| **POST** | Create (생성) | 새로운 리소스를 생성 | X |
| **GET** | Read (조회) | 리소스를 조회 | O |
| **PUT** | Update (전체 수정) | 리소스를 전체 덮어쓰기(교체) | O |
| **PATCH** | Update (부분 수정) | 리소스의 일부분만 수정 | △ (구현따라 다름) |
| **DELETE** | Delete (삭제) | 리소스를 삭제 | O |

## 3. URI 설계 원칙
**"URI는 동사가 아닌, 자원(명사)으로만 표현해야 한다."**
행위(동사)는 URL에 적지 않고, HTTP Method로 구분한다.

- **Bad Case (URL에 동사 포함)**
  - `POST https://api.com/getBooks` (X)
  - `DELETE https://api.com/deleteUser` (X)
  
- **Good Case (자원만 명시 + Method로 행위 결정)**
  - `GET https://api.com/v1/books` (책 목록 조회)
  - `POST https://api.com/v1/books` (책 생성)
  - `DELETE https://api.com/v1/books/10` (10번 책 삭제)

## 4. HTTP 프로토콜 구조 (우편 비유)
HTTP 요청을 우편 서비스에 비유할 수 있음.

**1) 편지 (Envelope): `GET`, `DELETE`**
- 봉투 안에 내용을 많이 담지 않음.
- 주로 주소(URL)나 쿼리 스트링으로 데이터를 전달.
- Body가 비어있는 경우가 많음.

**2) 택배 상자 (Parcel Box): `POST`, `PUT`, `PATCH`**
- 상자 안에 물건(Data Body/Payload)을 담아서 보냄.
- 대용량 데이터를 전송할 수 있음.

## 5. REST API의 핵심 구성 요소

### 1) 응답 상태 코드 (Status Code)
서버는 요청 처리 결과를 숫자로 된 코드로 명확히 알려주어야 함.
- **200번대:** 성공 (예: 200 OK, 201 Created)
- **400번대:** 클라이언트 잘못 (예: 400 Bad Request, 404 Not Found)
- **500번대:** 서버 잘못 (예: 500 Internal Server Error)

### 2) Stateless (무상태성)
- 서버는 클라이언트의 상태(Context)를 저장하지 않음.
- 즉, 서버는 "이 클라이언트가 방금 전에 로그인을 했었지?" 같은 것을 기억하지 않음.
- **장점:** 서버 확장성이 높아짐.
- **조건:** 클라이언트는 요청 시 필요한 모든 정보(토큰 등)를 매번 담아서 보내야 함.

### 3) Idempotent (멱등성)
- **"몇 번을 요청해도 결과가 같은가?"**
- 클라이언트 측에서 동일한 요청을 한 번 보내든, 여러 번 연속으로 보내든 **서버의 리소스 상태 결과가 같아야 함.**
  - `GET`, `PUT`, `DELETE`: 멱등함. (삭제를 10번 요청해도, 삭제된 상태인 건 똑같음)
  - `POST`: 멱등하지 않음. (결제 요청을 10번 보내면, 결제가 10번 됨)

### 4) Cacheability (캐시 가능)
- HTTP라는 기존 웹 표준을 그대로 사용하기 때문에, 웹의 인프라(캐싱)를 그대로 활용 가능.
- `Last-Modified`, `E-Tag` 등을 이용해 한번 가져온 데이터를 저장해두고 재사용 가능.
