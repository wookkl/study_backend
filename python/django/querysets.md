Django ORM Frame

```sql
SELECT *
FROM 'Model' m (inner ON left outer)
    JOIN '정방향 참조' r in m.r_id=r.id
        WHERE '조건절';

SELECT *
    FROM '역방향 참조 필드'
        WHERE id in ('첫번째 쿼리 결과의 id 리스트');
```

대체로
select_related는 join을 의도하고
prefetch_related는 +1 query를 의도한다.

```python
(Model.objects
    .filter("조건절")
    .select_related("정방향 참조")      # 해당 필드를 join
    .prefetch_related("역방향 참조")    # 해당 필드를 추가 쿼리
 )
```

select_relatd 또는 prefetch_related 없는 쿼리셋에서 join 또는 + 1 query가 발생했다면
명시적으로라도 selected_related 와 prefetch_related를 붙여주는 것이 좋다.

```python
company_qs: QuerySet = Company.objects.prefetch_related(
    'product_set').filter(name='company_name')

company_list: List[Company] = list(company_qs)

```

```sql
SELECT "app_company"."id", "app_company"."name"
    FROM "app_company"
        WHERE "app_company"."name" = "company_name1";

SELECT "app_company"."name"
    FROM "app_company"
        WHERE "app_product"."product_owned_company_id" IN (1, 21);
```

만약 `company_qs: QuerySet = Company.objects.prefetch_related('product_set').filter(name='company_name')`뒤에 product 관련 필터링이 더 추가 된다면

```sql
SELECT *
    FROM "app_company"
        INNER JOIN "app_product" ON ("app_company"."id" = "app_product"."product_owned_company_id")
        --여기서 join이 발생했으면 join으로 product row를 다 조회 해야 하지만 join을 통해 조건절을 검사 히지 않고 --
        -- 두번 째 쿼리에서 product를 한번 더 조회한다 --
    WHERE ("app_company"."name" = "company_name1" AND "app_product"."name" =  "product_name3") LIMIT 21;

-- prefetch_related 옵션으로 인해 해당 쿼리가 발생 -> 결고적으로 product를 불필요하게 2번 조회한다 --
SELECT *
    FROM "app_product"
        WHERE "app_product"."product_owned_company_id" IN ();
```

이런 경우는

1. prefetch_related('product_set') 이 구절을 없애거나
2. Prefetch()를 통해 필터링을 한다.

```python
(Company.objects
        .prefetch_related(
            Prefetch("product_set", queryset=Product.objects.filter(product__name="product_name3")))
        .filter(name="company_name1")
)

```

INNER JOIN의 경우에는 JOIN ON 절에 조건을 걸어주는 것과 WHERE절에 조건을 걸어주는 것은 성능 차이를 보일 수 있음

ON 절을 JOIN 하면서 조건절을 체크하지만

WHERE절은 JOIN 결과를 완성시킨 후에 조건절을 체크

ON 절에 조건을 주고 싶다면 FilteredRelation을 사용한다.

```python
(Product.objects
     .annotate(this_is_join_table_name=FilteredRelation('product_owned_company',
                                                        condition=Q(product_owned_company__name='company_name34'),
                                                       ),
              )
     .filter(this_is_join_table_name__isnull=False)
)
```

```sql
SELECT *
    FROM "app_product"
    INNER JOIN "app_company" this_is_join_table_name
    ON "app_product"."product_owned_company_id" = this_is_join_table_name."id"
        AND (this_is_join_table_name."name" = 'company_name3'))
    WHERE this_is_join_table_name."id" is NOT NULL;
```

Queryset이 의도한대로 나오지 않는다면

- selected_related -> FilteredRelation
- prefetch_related -> Prefetch
  를 사용!

## QuerySet 구조

- QuerySet은 1개의 Query와 N개의 QuerySet으로 구성된다.

하나의 QuerySet은 반드시 1개의 Main Query를 가지고 0 ~ N개의 SubQuerySet을 가진다.

```python
queryset = (Model.objects.
            .annotate(field=F('field'))
            .select_related('select_model')
            .filter(id__in=[1,2,3])                # 여기 까지가 메인 쿼리에 영향을 주는 옵션
            .prefetch_related('prefetch_model')    # 1개의 추가 쿼리(셋)이 생성
            )
```

이 ORM의 SQL은

```sql
SELECT *, field as "field",
    FROM "app_model" m
    INNER JOIN "app_select" ON m."select_id" = "app_select"."id"
    WHERE m."id" IN (1, 2, 3)

SELECT *,
    FROM "app_prefetch_model" pm
    WHERE pm."id" IN (1, 2, 3)
```
