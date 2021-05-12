# setting.py 파헤치기

## BASE_DIR

```python
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent
```

pathlib 모듈은 다른 운영 체제에 적합한 의미 체계를 가진 파일 시스템 경로를 나타내는 클래스를 제공한다.
코드가 실행되는 플랫폼의 구상 경로를 인스턴스화 한다!
따라서 Path(**file**)은 현재 파일의 구상 경로를 인스턴스화 하고 resolve() 메서드를 통해 절대경로를 반환한다.
그리고 부모 경로로 2번 옮겨가면 django 프로젝트가 생성된 위치가 BASE_DIR 변수로 인스턴스화 된 것!

## SECRET_KEY

```python
SECRET_KEY = "@xo=xu*=\*p6%9@qm9a9stnkqmi0y#m#i!$02pfhk=7s=y3w@z-"
```

SECRET_KEY는 특정한 장고 설치를 위한 키이다. 암호화 서명을 제공하는데 사용되기도 하고, 유니크한 값으로 설정된다.
프로젝트를 생성할때 자동으로 SECRET_KEY가 생성된다. 키를 사용할 때 텍스트 또는 바이트라고 가정하면 안된다. 사용할 떈 꼭
force_str() 또는 force_byte()해서 변환해야 한다. SECRET_KEY가 없으면 프로젝트는 실행되지 않는다.
SECRET_KEY는 다음 경우에 사용된다.

- django.contrib.sessions.backends.cache 이외에 다른 백엔드 세션을 사용하거나, default인 get_session_auth_hash()메서드로 SHA-256 해싱할때의 모든 세션
- CookieStorage나 FallbackStorage를 사용할 때의 모든 메세지
- 다른 키가 제공되지 않았을 때 암호화 서명

## DEBUG

```python
DEBUG = True
```

앱이 디버그 실행인지 아닌지 설정할때 사용하는 변수

## ALLOWED_HOSTS

```python
ALLOWED_HOSTS = []
```

이 앱이 제공할 수 있는 호스트/도메인 이름을 나타내는 리스트이다. Http Host header attack을 방지하기 위한 조치이다.
Http Host header attack (Cache poisoning)이란?

하나의 ip 주소를 사용하는 웹-서버에 두개 이상의 웹 서비스 (www.abc.com, www.xyz.com)를 구현할 경우에, 가상 호스팅 기능을 사용하는데, 이때 Host header를 이용하여 각각의 도메인을 구분한다. 이떄 공격자가 Host header를 변조하면 피싱 사이트 접근, XSS 실행, 암호 재설정등 취약점이 발생한다.

## INSTALLED_APPS

```python
INSTALLED_APPS = [
"django.contrib.admin",
"django.contrib.auth",
"django.contrib.contenttypes",
"django.contrib.sessions",
"django.contrib.messages",
"django.contrib.staticfiles",
]
```

기본적으로 설치되아있는 앱들

- django.contrib.admin: 장고의 가장 큰 장점중에 하나인 admin interface
- django.contrib.auth: 장고 인증 시스템
- django.contrib.contenttypes: 프로젝트에 설치된 모든 모델을 추적할 수 있음 높은 수준의 모델 작업에 인터페이스를 제공
- django.contrib.sessions: 익명 세션에 데이터를 저장하기 위한 프레임워크
- django.contrib.messages: 웹페이지에 1회성 알림 메세지를 띄울 때 사용하는 프레임워크
- django.contrib.staticfiles: 배포환경에서 쉽게 제공될 수 있게 각각의 앱의 정적 파일들을 한곳에 모아주는 프레임워크

## MIDDLEWARE

```python
MIDDLEWARE = [
"django.middleware.security.SecurityMiddleware",
"django.contrib.sessions.middleware.SessionMiddleware",
"django.middleware.common.CommonMiddleware",
"django.middleware.csrf.CsrfViewMiddleware",
"django.contrib.auth.middleware.AuthenticationMiddleware",
"django.contrib.messages.middleware.MessageMiddleware",
"django.middleware.clickjacking.XFrameOptionsMiddleware",
]
```

### SecurityMiddleware

보안 미들웨어는 request/response 싸이클에 향상된 보안 기능을 제공한다. 각각의 기능들은 독립적으로 enable, disable 가능하다.

- SECURE_BROWSER_XSS_FILTER (Default: False) : True로 지정하면 모든 응답 헤더에 `X-XSS-Protection: 1;mode=block`가 추가된다. 최신 브라우저는 더 이상 이 헤더를 따르지 않는다. 이 헤더가 실질적으로 이점이 거의 없지만 오래된 브라우저를 지원하는 경우에 설정해주는 것이 좋다.

- SECURE_CONTENT_TYPE_NOSNIFF (Default: True): True로 지정하면 모든 응답 헤더에 `X-Content-Type-Options: nosniff`가 추가된다. HTML, Javascript로 만들어진 파일을 업로드 할 수 있을 때 악의적인 사용자가 위험한 파일들을 업로드할 수 있다. 이 헤더는 MIME 타입의 스니핑을 방지한다.

**HTTP Strict Transport Security(HSTS)**

오직 https에서 액세스해야 하는 경우에 이 헤더를 설정하여 최신 브라우저가 안전하지 않은 연결(주어진 기간동안)을 통해 도메인 이름에 대한 연결을 거부하도록 할 수 있다.

- SECURE_HSTS_INCLUDE_SUBDOMAINS: 서브도메인에도 이 헤더를 추가시킬 수 있다.
- SECURE_HSTS_PRELOAD: True로 지정하면 preload list에 등록되어 브라우저가 이 도메인에 안전하지 않은 연결을 하지 않게 된다.
- SECURE_HSTS_SECONDS: 웹브라우저가 HSTS 헤더를 볼 때 마다 주어진 기간동안 비보안 통신을 거부한다.
- SECURE_REDIRECT_EXEMPT: url이 이 리스트의 정규식과 일치하면 https로 리디렉션되지 않는다.
- SECURE_REFERRER_POLICY: Referrer Policy 헤더에 추가되어 응답된다.
- SECURE_SSL_HOST: 모든 SSL 리디렉션들이 원래의 요청 호스트로 리디렉션되는 것이 아니라 여기에 설정된 호스트로 리디렉션된다.
- SECURE_SSL_REDIRECT(Default: False): True면 http가 아닌 요청들은 https로 리디렉션된다.

### SessionMiddleware

세선을 지원해준다.

### CommonMiddleware

- DISALLOWED_USER_AGENT: 이 사용자 에이전트에 대한 액세스를 금지시킨다.
- APPEND_SLASH: True면, 초기 url 끝에 백슬래시(/)가 붙지 않는다.
- PREPEND_WWW: True면, www가 없는 url은 www이 붙어서 리디렉션된다.

### CsrfViewMiddleware

Csrf에 대해서 보호해주는 미들웨어이다.

### AuthenticationMiddleware

모든 요청되는 HttpRequest 객체에 현재 로그인되어있는지 나타내는 user객체가 담겨있다.

### MessageMiddleware

꼭 세션 미들웨어 뒤에 추가해주어야한다. 이유는 세션기반 미들웨어이기 떄문이다.

### XFrameOptionsMiddleware

X-Frame-Options 헤더와 함께 클릭재킹에 보호해주는 미들웨어이다.

## ROOT_URLCONF

```python
ROOT_URLCONF = "config.urls"
```

Root url을 구성하는 string

## TEMPLATES

```
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
```

템플렛 엔진을 사용하기 위한 설정들이 들어있다.

### BACKEND

사용할 템플릿 백엔드를 세팅한다. 기본 제공 템블릿 백엔드는 DjangoTemplates과 Jinja2가 있고 따로 사용자가 경로를 지정하고 오픈소스를 사용할 수도 있다.

### DIRS

엔진이 템플릿 소스파일을 찾아야하는 디렉토리 경로이다.

### APP_DIRS

엔진이 설치된 애플리케이션 내에서 템플릿 소스파일을 찾아야하는지 여부이다.

### OPTIONS

엔진에게 전달할 추가 매개변수이다.

## WSGI_APPLICATION

```python
WSGI_APPLICATION = "config.wsgi.application"
```

장고가 사용할 wsgi 애플리케이션의 경로이다.

## DATABASES

```python
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": BASE_DIR / "db.sqlite3",
    }
}
```

장고가 사용할 데이터베이스를 설정하는 부분이다.
sqlite3이외의 데이터베이스를 사용하려면 추가적으로 데이터베이스에 필요한 파라미터들을 넣어주어야한다.

만약 postresql을 사용한다면

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mydatabase',
        'USER': 'mydatabaseuser',
        'PASSWORD': 'mypassword',
        'HOST': '127.0.0.1',
        'PORT': '5432',
    }
}
```

## AUTH_PASSWORD_VALIDATORS

```python
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
```

패스워드의 보안 강도를 체크하는 validatar 리스트이다.

## LANGUAGE_CODE

```python
LANGUAGE_CODE = "en-us"
```

이 설치에 대한 언어 코드를 나타낸다. 표준 언어 ID 형식이어야 한다.USE_I18N이 꼭 True이어야 한다.

## TIME_ZONE

```python
TIME_ZONE = "Asia/Seoul"
```

설치된 곳의 time zone을 나타낸다. 반드시 서버의 시간대는 아니다. 하나의 서버가 각각 별도의 시간대 설정이 있는 장고 기반 사이트를 제공할 수 있다.

## USE_I18N

```python
USE_I18N = True
```

장고의 번역 시스템을 활성화 하는지의 여부이다.

### USE_L10N

```python
USE_L10N = True
```

locaization된 데이터 형식을 활성화하는지 안하는지의 여부이다. True라면, 장고는 현재 위치에 맞춰진 숫자와 날짜를 보여준다.

### USE_TZ

```python
USE_TZ = True
```

True면, 장고는 내부적으로 날짜 / 시간을 사용한다.

## STATIC_URL

```python
STATIC_URL = "/static/"
```

정적파일들을 참조할 때 사용하기 위한 url이다.

## AUTH_USER_MODEL

```python
AUTH_USER_MODEL = "auth.User"
```

유저를 나타내기 위해 사용될 모델이다.
