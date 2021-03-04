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
    unique_together = ("field1", "field2    ")
```

# setting.py 파헤치기

```python



from pathlib import Path

"""
pathlib 모듈은 다른 운영 체제에 적합한 의미 체계를 가진 파일 시스템 경로를 나타내는 클래스를 제공한다.
코드가 실행되는 플랫폼의 구상 경로를 인스턴스화 한다!
따라서 Path(__file__)은 현재 파일의 구상 경로를 인스턴스화 하고 resolve() 메서드를 통해 절대경로를 반환한다.
그리고 부모 경로로 2번 옮겨가면 django 프로젝트가 생성된 위치가 BASE_DIR 변수로 인스턴스화 된 것!
"""

BASE_DIR = Path(__file__).resolve().parent.parent

"""
SECRET_KEY는 특정한 장고 설치를 위한 키이다. 암호화 서명을 제공하는데 사용되기도 하고, 유니크한 값으로 설정된다.
프로젝트를 생성할때 자동으로 SECRET_KEY가 생성된다. 키를 사용할 때 텍스트 또는 바이트라고 가정하면 안된다. 사용할 떈 꼭
force_str() 또는 force_byte()해서 변환해야 한다. SECRET_KEY가 없으면 프로젝트는 실행되지 않는다.
SECRET_KEY는 다음 경우에 사용된다.
-  django.contrib.sessions.backends.cache 이외에 다른 백엔드 세션을 사용하거나, default인 get_session_auth_hash()메서드로 SHA-256 해싱할때의 모든 세션
- CookieStorage나 FallbackStorage를 사용할 때의 모든 메세지
- 다른 키가 제공되지 않았을 때 암호화 서명
"""
SECRET_KEY = "@xo=xu_=*p6%9@qm9a9stnkqmi0y#m#i!$02pfhk=7s=y3w@z-"

"""
앱이 디버그 실행인지 아닌지 설정할때 사용하는 변수
"""
DEBUG = True

"""
이 앱이 제공할 수 있는 호스트/도메인 이름을 나타내는 리스트이다. Http Host header attack을 방지하기 위한 조치이다.
Http Host header attack (Cache poisoning)이란?

하나의 ip 주소를 사용하는 웹-서버에 두개 이상의 웹 서비스 (www.abc.com, www.xyz.com)를 구현할 경우에, 가상 호스팅 기능을 사용하는데, 이때 Host header를 이용하여 각각의 도메인을 구분한다. 이떄 공격자가 Host header를 변조하면 피싱 사이트 접근, XSS 실행, 암호 재설정등 취약점이 발생한다.
"""
ALLOWED_HOSTS = []

"""
기본적으로 설치되아있는 앱들
- django.contrib.admin 장고의 가장 큰 장점중에 하나인 admin interface
- django.contrib.auth 장고 인증 시스템
- django.contrib.contenttypes 프로젝트에 설치된 모든 모델을 추적할 수 있음 높은 수준의 모델 작업에 인터페이스를 제공
- django.contrib.sessions 익명 세션에 데이터를 저장하기 위한 프레임워크
- django.contrib.messages 웹페이지에 1회성 알림 메세지를 띄울 때 사용하는 프레임워크
- django.contrib.staticfiles 배포환경에서 쉽게 제공될 수 있게 각각의 앱의 정적 파일들을 한곳에 모아주는 프레임워크
"""

INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
]

"""
장고 미들웨어
"""
MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.middleware.common.CommonMiddleware",
    "django.middleware.csrf.CsrfViewMiddleware",
    "django.contrib.auth.middleware.AuthenticationMiddleware",
    "django.contrib.messages.middleware.MessageMiddleware",
    "django.middleware.clickjacking.XFrameOptionsMiddleware",
]

ROOT_URLCONF = "config.urls"

TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",
        "DIRS": [],
        "APP_DIRS": True,
        "OPTIONS": {
            "context_processors": [
                "django.template.context_processors.debug",
                "django.template.context_processors.request",
                "django.contrib.auth.context_processors.auth",
                "django.contrib.messages.context_processors.messages",
            ],
        },
    },
]

WSGI_APPLICATION = "config.wsgi.application"


# Database
# https://docs.djangoproject.com/en/3.1/ref/settings/#databases

DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": BASE_DIR / "db.sqlite3",
    }
}


# Password validation
# https://docs.djangoproject.com/en/3.1/ref/settings/#auth-password-validators

AUTH_PASSWORD_VALIDATORS = [
    {
        "NAME": "django.contrib.auth.password_validation.UserAttributeSimilarityValidator",
    },
    {
        "NAME": "django.contrib.auth.password_validation.MinimumLengthValidator",
    },
    {
        "NAME": "django.contrib.auth.password_validation.CommonPasswordValidator",
    },
    {
        "NAME": "django.contrib.auth.password_validation.NumericPasswordValidator",
    },
]


# Internationalization
# https://docs.djangoproject.com/en/3.1/topics/i18n/

LANGUAGE_CODE = "en-us"

TIME_ZONE = "Asia/Seoul"

USE_I18N = True

USE_L10N = True

USE_TZ = True


# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/3.1/howto/static-files/

STATIC_URL = "/static/"

AUTH_USER_MODEL = "users.User"
```
