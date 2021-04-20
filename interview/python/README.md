# 파이썬 면접 질문들

## list / tuple / dict / set 의 차이점을 설명 해보세요.

- list: 순서가 존재, 가변형 객체, 인덱싱 슬라이싱 가능
- tuple: 순서가 존재, 불변형 객체
- dict: 키, 값으로 구성 중복불가
- set: 키로 구성 중복불가

## dict의 데이터 넣을때와 충돌시 시간 복잡도를 말해보세요.

- 최소 O(1) 최악: O(N)

## 정렬된 리스트에서 찾을때의 시간 / 정렬되지 않았을때의 시간을 말해보세요.

- 정렬된 리스트라면 이분탐색을 사용하여 log N에 검색가능
- 정렬되지 않았다면 N에 검색 가능

## django model에서의 select_related and prefetch_related 동작 방식을 말해보세요.

- selected_related는 조인문이 생성된다
- prefetch_related는 추가 쿼리사 발생한다.

## django에서 어떨때 inner join / outer join으로 나뉘는지 말해보세요.

## testcase에서 setup / setclass의 차이를 말해보세요.

## test 에서 django model 을 안쓰고 싶을때 어떻게 상속 받아야 하는지 말해보세요.

## serializer에서 객체 validate 하는 법을 말해보세요.

## 파이썬의 메모리 최적화에 대해 말해보세요.

## 파이썬의 exception 사례들 모델 작성 후 쿼리 실행 계획 확인 하는지 Prefetch가 왜 +1 쿼리인지 

## 왜 django의 상속되는 함수들을 안쓰는지 말해보세요.

## serializer의 기능들을 잘 쓰는가? (valid, 객체 반환, 400 리턴등)

## 파이썬 generator에 대해 아는 만큼 설명해주세요.

## 파이썬에서 클래스를 상속하면, 메서드는 어떤 식으로 실행되나요?

## 어떤 request가 Django API까지 도달하는 과정을 최대한 자세히 설명해주세요.

## 파이썬에 존재하는 GIL에 대해서 설명해주세요.

## Django ORM의 작동 방식에 대해 설명해주세요.

## Django ORM에서 지연 평가를 하곤 하는데요. 직접 구현한다면 어떻게 구현하겠습니까?

## http와 https의 차이점은?

## 도커 컨테이너 내부에 nat가 존재할 수 있나요?

## 파이썬 GIL이 무엇이고, 왜 성능 문제가 발생하는가?

## 파이썬의 GC는 어떻게 작동하나요?

## 프로세스와 쓰레드는 어떻게 다른가요?

## pypy는 도대체 왜 CPython보다 빠른 걸까요?

## 웹 서비스 응답이 느리다면 어떻게 해결할 수 있을까요?

## python에서 메모리 릭이 발생할 수 있는 경우는 언제일까요?
