# Tips

- 절대 애플리케이션 폴더를 크게 만들면 안된다. (많은 기능들을 한 폴더에 넣으면 안된다.)
- 한뭉장으로 애플리케이션을 설명할 수 있어야한다.
- Linter로는 flake8 Formatter로는 black 좋다.

## mark_safe

admin 페이지서 썸네일을 띄우려고 하는 데 장고는 스크립트에 대한 보안을 기본적으로 가지고 있다. 그래서 따로 html을 사용하기위해 장고에게 알려주어야 하는데 이 mark_safe 모듈을 사용하면 된다.

## raw_id_fields

예를들어 수천개의 user들이 있을 때 일일히 스크롤하기 힘드니 raw_id_fields를 사용하면 유저 어드민 패널에 들어가서 검색후 선택할 수 있다.

## InlineAdmins

어드민 패널을 보면 해당 Room의 외래 키를 가지고 있는 사진들을 나타내준다. \_\_왜냐하면 relationship name이 room이고 class name이 room이기 때문에 장고가 자동으로 뭘 말하는지 안다.

```python

class PhotoInline(admin.TabularInline)
     model = models.Photo

@admin.register(models.Room)
class RoomAdmin(admin.ModelAdmin):
    inlines = (PhotoInlines,)

```

## save() 와 admin_save()

save()에서는 model.py에서 직접 데이터가 저장되는 메서드이고, admin_save()는 admin 패널에서 저장될 때의 메서드이다. 호출 순서는 admin_save() -> save()이다. 따라서 admin_save()만 참조하여 오버라이딩한다면 admin패널에서 일어나는 부분만 수정이 가능하다.

## ManyToMany Field에서 필드를 추가하는 방법

`room.amenities.add(field)`

## extends

미리 만들어준 html골격을 사용한다.

## block

일종의 창 같읕 것이다. 자식 템플릿이 부모템플릿에게 넘겨주는 창

## room_models.Room.objects.all()

사실 이것은 즉시 데이터를 리턴하는 것이 아니라 쿼리셋만 생성한 뿐이다. 호출이 되는 순간에 즉시 데이터베이스로부터 불려 올 것이다.

## Field lookups

필드 필터링할 때 사용한다.
\_\_gte: greater than equal

**lte: less than equal
**pk: primary key
\_\_startswith: 해당 데이터로 시작하는 필드

## order_by

필터링링된 객체들은 순서가 뒤섞여 있기 때문에 `qs = qs.order_by("created")`를 통해 정렬할 수 있다.

## dotenv

중요한 정보를 key-value로 이루어진 .env 파일에 저장하여 암호화한다.

```python
SECRET_KEY = os.environ.get("SECRET_KEY")
```

## simple_tag

takes_context=True 해주면 장고가 전달해주는 유저나 다른 컨텍스트를 받을 수 있다.
태그는 필터보다 더 많은 아규먼트를 보낼 수 있다. 템플릿에서 {% is_booked room day as is_booked_bool %} << 이렇게 하면 is_booked_bool이라는 변수를 가지게 됨! room이랑 day는 아규먼트

## managers

objects.get() objects.filter() 이런 것들이 다 manager이다.

### managers 상속 받는 법

1. managers.py 파일을 만든다.
2.

```python
from django.db import models

class CustomManager(models.Manager):
    def get_or_none(self, **kwargs):
        try:
            return self.get(**kwargs):
        except self.models.DoesNotExist:
            return None
```

3. models.py 에 `object = managers.CustomManager()` 넣어준다.

## Q objects

hard quaries하는 방법이다. 복잡한 쿼리는 만들 때 사용한다.

## Decorator

함수의 앞 뒤를 꾸며준다.

### Class decorator

일반적인 FBV와 다르게 @method_decorator를 클래스 위에 먼저 써줘야한다.
그 안에 decorator를 너어주고 decorating하고자 하는 메서드를 넣어준다.

```python
@method_decorator(login_required, 'get')
class CBV():
    ...
    ...

    def get(*args, **kwargs):
        pass
```

## unique_together

두개의 필드 조합을 가지는 객체가 오직 하나만 존재할 수 있도록 메타 클래스에 정의한다.

```python
class Meta:
    unique_together = ("field1", "field2")
```
