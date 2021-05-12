# Model

## GenericForeignKey

```python
class Category(models.Model):
    name = models.CharField(max_length=100)
    class Meta:
        verbose_name_plural = 'Categories'

class Hero(models.Model):
    name = models.CharField(max_length=100)
    category = models.ForeignKey('Category', on_delete=models.CASCADE)

class Villian(models.Model):
    name = models.CharField(max_length=100)
    category = models.ForeignKey('Category', on_delete=models.CASCADE)
```

이렇게 두개의 모델을 `ForeignKey` 필드로 지정하면 좋은 프로그래밍 습관이 아니다. 이를 해결하기 위해, Django의 `contenttypes` 라는 프레임워크를 지원하여 어떤 모델들과도 관계를 정의하게 해주는 **`GenericForeignKey`** 라는 특별한 필드 타입을 제공한다.

`GenerignForeignKey`를 세팅하기 위해 세 가지 단계가 있다.

1. ContentType 객체를 `ForeignKey` 로 정의한다. 이 필드의 이름은 대부분 `content_type`이다.
   - 이것은 django_content_type 테이블에서 지정된 특정 테이블을 지칭한다. 해당 모델 타입의 객체를 얻고 싶다면 django_content_type.id와 매칭되는 `Category` 테이블의 content_type.jd를 유의하면 된다.
2. 연관된 모델의 기본키 값을 저장할 수 있는 필드를 지정한다. 대부분 `PositiveIntegerField` 타입이며 이름은 `object_id` 이다.
   - 반드시 Primary Key이어야 한다.
3. `GenericForeignKey` 타입 필드를 만들고 위의 두 필드의 이름을 인자로 넘겨준다. 이 인자들의 값은 `content_type` 과 `object_id` 이다.
4. 범용 모델을 이용할 모델에 범용 관계 필드 `GenericRelation`을 추가한다.
   - 이것은 컨텐츠 모델이 ContentType 프레임워크를 통해 어떤 모델과 연관성이 있다는 것을 알기 위해 정의되어 있는 것이다

따라서, 세 단계를 진행한 후 만든 `Category`의 모델은

```python
class Category(models.Model):
    name = models.CharField(max_length=100)
    content_type = models.ForeignKey(ContentType, on_delete=models.CASCADE)
    object_id = models.PositiveIntegerField()
    content_object = models.GenericForeignKey('content_type', 'object_id')
    class Meta:
        verbose_name_plural = 'Categories'

class Hero(models.Model):
    name = models.CharField(max_length=100)
    category = models.GenericRelation('Category', on_delete=models.CASCADE)

class Villian(models.Model):
    name = models.CharField(max_length=100)
    category = models.GenericRelation('Category', on_delete=models.CASCADE)
```

## related_name과 related_query_name의 차이점

```python
class Article(models.Model):
    pass

class Tag(models.Model):
    article = models.FoerignKey(Article,
        on_delete=models.CASCADE,
        related_name='tags',
        related_query_name='tag'
    )
    name = models.CharField(max_length=255)
```

Article과 Tag의 관계는 1:N이다.

위에서 지정한 related_name과 related_query_name 을 어떻게 사용하는지 ORM 쿼리를 본다.

```python
# Article 인스턴스에 연결된 Tag들을 가져온다.
article = Article.object.get(id=1)
for tag in article.tags.all():
    print(tag.name)

# Queryset의 filter 조건으로 사용한다.
summer_tag_articles = Article.objects.filter(tag__name="summer")
```

related_query_name 미지정시, 기본값은 related_name이다.

따라서, related_name은 1:N의 관계에 있는 모델 Article:Tag에서 Article 인스턴스가 관계된 Tag객체들을 불러올 떄 사용하는 변수이다.

## Q() 와 F()

### Q()

    Q 객체는 get(), exclude(), filter()와 같은 키워드 인자를 전달받을 수 있는 django query 메소드 인자를 캡슐화하여 좀 더 복잡한 query를 만드는데 사용하는 객체다.

    1. or 조건

    ```python
    from django.db.models import Q

    Product.objects.filter(Q(name='backpack')|Q(price='9000'))
    # SELECT * FROM 'product' WHERE 'name'='backpack' OR 'price'=9000
    ```

    2. and 조건

    ```python
    Product.objects.filter(Q(name='backpack')&Q(price='9000'))
    # SELECT * FROM 'product' WHERE 'name'='backpack' AND 'price'=9000
    ```

    3. not 조건

    ```python
    Product.objects.filter(~Q(price__lt=5000))
    # SELECT * FROM 'product' WHERE NOT ('price'<5000)

    ```

### F()

    파이썬으로 메모리를 캐시하지 않고, 데이터베이스 내에서 작업을 처리할 때 사용한다.

    ```python
    from django.db.models import F

    post = Post.objects.get(id=1)
    post.view_count = F('view_count') + 1
    post.save(update_fields=['view_count'])
    ```

    dango는 F() 인스턴스를 만나면 뒤에 이어지는 python 연산자를 오버라이드하여 F('view_count') + 1 자체를 쿼리한다. 따라서 파이썬을 view_count의 값을 fetch해오지 않아 값을 알 수 없다.

    - 주의할 점

      F() 객체를 사용하여 DB의 값을 직접 조정했을 경우

      refresh_from_db 메서드를 통해 리로드한다.

## ManytoMany through 속성

```python
class Tag(models.Model):
    name = models.CharField(max_length=255)

class Product(models.Model):
    name = models.CharField(max_length=255)
    tags = models.ManyToManyField(Tag, through=ProductTag)

class ProductTag(models.Model):
    product = models.ForeignKey(Product, on_delete=models.CASCADE)
    tag = models.ForeignKey(Tag, on_delete=models.CASCADE)
```

ManyToMany 테이블을 확장하여 관리할 수 있고 ManyToMany의 테이블을 알려주기 위해 through에 ManyToMany 테이블을 넣어준다.
