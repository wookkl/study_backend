# Pagination

모든 DRF Pagination 추상 클래스 `BasePagination`을 상속 받는다.

## 1. PageNumberPagination

- `page_query_param`과 `page_size_query_param`을 사용해서 페이지네이션하는 가장 기본적인 방법
- 기본적으로 장고의 `Paginator`를 사용한다.
- 주어진 쿼리셋에서 `[(페이지넘버 - 1) * 페이지사이즈:(페이지넘버 - 1) * 페이지사이즈 + 페이지사이즈]` 를 슬라이싱하여 페이지네이션된 쿼리셋을 반환한다.
- 데이터를 조회할 때 `select * from article limit 10 offset 20` 같은 식으로 조회를 한다.
- 따라서 쿼리에서 limit은 `페이지사이즈`가 되고 offset은 `(페이지넘버 -1) * 페이지사이즈` 가 된다.

## 2. LimitOffsetPagination

- PageNumberPagination과 거의 유사하다.
- 장고의 `Paginator`를 사용하지 않는다.
- query param으로 `limit_query_param`과 `offset_query_param` 을 사용해서 조회한다.
- PageNumberPagination과 같은 방법으로 데이터를 조회한다.
- 쿼리에서 limit은 `limit_query_param`가 되고 offset은 `offset_query_param` 가 된다.

## 3. PageNumberPagination과 LimitOffsetPagination

### 차이점

- PNP는 offset이 limit의 배수이지만 LOP는 자유롭다.
- PNP는 Django Paginator를 사용하지만 LOP는 사용하지 않는다.

### 장점

- 유저가 페이지를 선택하고, 이동할 수 있다.
- 전체 페이지 개수를 알 수 있다.

### 단점

- offset 위치를 계산하고, 필요한 데이터를 찾을 때까지 테이블 전체를 스캔한다.
- offset이 커질 수록 데이터베이스의 부하는 커진다.

## 4. CursorPagination

- Cursor Pagination은 ordering기준으로 페이지는 구하는 방식을 말한다.
- 이 방식을 사용하면 인덱스가 적용된 값을 비교하기 때문에 테이블을 풀 스캔하지 않는다.
- id 값으로 데이터를 조회하기 때문에, 데이터 쓰기가 빈번한 테이블이여도 다음 페이지네이션 조회 시 값이 누락되지 않는다.
- 전체의 페이지 개수는 알 수 없다.
- 인코딩된 커서 쿼리 파라미터를 가지고있다.
- `http://127.0.0.1:8000/api/vi/app/post/?cursor=cD05OTQ%` 처럼 인코딩된 쿼리 파라미터가 디코딩된 후 그에 맞는 sql 문을 작성한다.
- `SELECT "app_post"."id", "app_post"."title" FROM "app_post" WHERE "app_post"."id" < 994 ORDER BY "app_post"."id" DESC` 쿼리에 따라 ASC/DESC 와 ORDER_BY 절, WHERE절이 구성된다.
- 그 다음에 `page_size` 필드를 통해 `LIMIT` 된다.
