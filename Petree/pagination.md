## Offset pagination

```sql
SELECT *
FROM items
WHERE 조건문
ORDER BY id DESC OFFSET 페이지번호
LIMIT 페이지사이즈
```
- offset(몇번째부터 가져올 것인가?)과 limit(몇개를 가져올 것인가?)예약어를 통하여 select전체 결과 중 일부만 가져오는 방법이다.


- 단점
  - **페이지 요청 시, 데이터 변화가 있는 경우 중복 데이터 발생**
    - 예를 들어, 1페이지에서 20개의 row를 불러와서 유저에게 1페이지를 띄워주었다.
    - 고객이 1페이지의 상품을 보고 있는 사이, 상품 운영팀에서 5개의 상품을 새로 올렸다면?
      - 유저가 1페이지 상품을 다 둘러보고 2페이지를 눌렀을 때, 1페이지에서 보았던 상품 20개중 마지막 5개를 다시 2페이지에서 보게 된다.
  - **대부분이 RDBMS에서 OFFSET쿼리의 퍼포먼스 이슈**
    - 페이지가 뒤로 갈수록, 앞에서 읽었던 행을 다시 읽어야하며, 버리지만 읽어야할 행의 개수가 많아져 뒤로 갈수록 느려지게 된다.
      - 예를 들어, 극단적으로 10억번째 페이지에 있는 값을 찾고 싶다면 OFFSET에 매우 큰 숫자가 들어가게 된다.
      - 이렇게 된다면, 페이지가 뒤로 갈수록 읽었던 행을 다시 읽으며, 실제 읽어야할 ROW보다 버려야할 ROW가 훨씬 많아지게 된다.

---

OFFSET 의 성능을 개선해보자.

가장 먼저 NO-OFFSET을 확인해볼 수 있다.

## Cursor pagination (=NO-OFFSET)

- cursor는 어떠한 레코드를 가리키는 포인터이고 이 cursor가 가리키는 레코드부터 일정 개수만큼 가져오는 방식이다.

```sql
SELECT *
FROM items
WHERE 조건문
AND id < 마지막조회ID# 직전 조회 결과의 마지막 id ORDER BY id DESC LIMIT 페이지사이즈
```

- 조회 시작부분을 인덱스로 빠르게 찾아 매번 첫 페이지만 읽도록 하는 방식으로 offset과는 다르게 버려야할 행이 발생X
  - 즉, 매번 이전 페이지 전체를 건너뛸 수 있음을 의미한다.
  - 따라서, offset기반 페이지네이션 보다 훨씬 더 좋은 성능을 보여준다.
- UI상으로는 무한스크롤이 될 수 있다.

- 사용 시 유의사항
  - where에 사용되는 기준 key가 중복이 가능한 경우
    - 정확한 결과를 반환할 수 없다.
  - 서비스 정책상 무조건 페이징 버튼이 필요하다면 cursor를 사용하면 안된다.

- 참조블로그
  - https://jojoldu.tistory.com/528
  - https://wonyong-jang.github.io/database/2020/09/06/DB-Pagination.html

---

## Covering Index

No-Offset페이징을 사용할 수 없는 상황이라면 커버링 인덱스로 성능을 개선할 수 있다.

- 커버링 인덱스는 쿼리를 충족시키는 데 필요한 모든 데이터를 갖고 있는 인덱스를 이야기한다.
  - `select, where, order by, limit, group by` 등에서 사용되는 컬럼이 index컬럼 안에 다 포함된 경우를 말한다.

- 하지만, `select` 절까지 포함하게 되면 너무 많은 컬럼이 인덱스에 포함되므로, `select`를 제외한 나머지만 포함하여 인덱스를 생성한다.

```sql
SELECT *
FROM items
WHERE 조건문
ORDER BY id DESC
OFFSET 페이지번호
LIMIT 페이지사이즈
```

```sql
SELECT  *
FROM  items as i
JOIN (SELECT id
FROM items
WHERE 조건문
ORDER BY id DESC
OFFSET 페이지번호
LIMIT 페이지사이즈) as temp on temp.id = i.id
```

즉, 첫번째 페이징쿼리를 두번째처럼 사용한다.

- Covering Index가 빠른 이유
  - 일반적인 조회 쿼리는 `order by, offset ~ limit`을 수행할 때 데이터 블록으로 접근하게 된다.
  - 하지만 Covering Index를 사용하면 `where, order by, offset ~ limit`을 인덱스 검색으로 빠르게 처리하고 이미 걸러진 row에 대해서만 데이터 블록으로 접근하기 때문에 성능을 향상시킨다.

  - QueryDSL로 구현할 경우
    - QueryDSL은 from절의 서브쿼리를 지원하지 않는다.
    - 그래서 이를 우회할 방법은 다음과 같다.
      - Covering Index를 활용해 조회 대상의 PK를 조회한다.
      - 해당 PK로 필요한 컬럼 항목을 조회한다.

- 단점
  - 너무 많은 인덱스가 필요하다.
  - 인덱스 크기가 너무 커진다.
  - 데이터 양이 많아지고 페이지 번호가 뒤로 갈수록 NoOffset보다 느리다.
    - 시작 지점을 PK로 지정하고 조회하는 NoOffset 방식에 비해서 성능 차이가 있음 (NoOffset과 동일한 데이터 조회시 커버링 인덱스 방식은 272ms, No Offset은 83ms)
    - 테이블 사이즈가 계속 커지면 No Offset 방식에 비해서는 성능 차이가 발생한다.

- 참조블로그
  - https://jojoldu.tistory.com/529?category=637935