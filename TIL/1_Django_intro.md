# Django Intro

## 설치

```shell
# 최신 버전
$ pip install django

# 버전 명시
& pip install django==3.0.8
```



## 프로젝트 생성

```shell
# django-admin startproject 프로젝트이름 경로
# . 을 적게되면 현재 경로에 바로 생성
# 안쓰면 폴더 하나 더 만들고 그 안에 생성
& django-admin startproject firstpjt .
# 이게 안되면
& python -m django startproject firstpjt
```

## 프로젝트 폴더 구성

`settings.py`

-django의 설정 사항(장고 내부의 설정 사항이 있다? 대부분 settings.py 이용)

`urls.py`

- url과 함수를 연결

`asgi.py`

- django가 비동기식 웹 서버와 연결하는 것을 설정

`wsgi.py`

- 웹 서버와 연결하는 것을 설정(배포 단계에서 고민할 내용)

## 애플리케이션 생성

```shell
# 일반적으로 복수형으로 쓴다.
# python manage.py startapp 앱이름
& python manage.py startapp articles
```

## 애플리케이션 등록

> 앱 생성이후에는 반드시 등록! (잊어버리기 전에!!)
> 
> 반드시 생성 후에 등록! (순서를 잘 지키자.)

```python
# 프로젝트메인폴더/settings.py >>> INSTALLED_APPS에 등록
# firstpjt/settings.py

INSTALLED_APPS = [
    # 1. local apps(사용자 생성 앱)
    'articles',
    # 2. 3rd-party appsin
    # 3. django apps(빌트인)
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

## 애플리케이션 폴더

`migrations/`

- 마이그레이션 파일들이 들어감
- 데이터베이스와 연결했을 때 사용할 폴더다.

`templates/`

- 탬플릿 폴더(생성해야함!)
- `s` 오타 주의!

중요!

`models.py`

- 앱에서 사용하는 모델을 정의

`urls.py`

- 생성해야함! (url을 분리할 때)

`views.py`

- url이랑 연결 할 함수들을 정의

덜 중요!

`admin.py`

- 관리자 페이지를 위한 설정


`tests.py`

- 테스트 코드를 작성

`apps.py`

- 앱에 관한 설정(거의 사용하지 않음)


---

## 요청 처리하기

> HTTP request에 따라 각기 다른 동작을 결정
>
> - 메인 페이지 보여주세요 => 메인 페이지를 돌려준다.
>
> - 회원가입 페이지 보여주세요 => 회원가입 페이지를 돌려준다.
>

요청을 구분하는 2가지

- **url**
- **method**

url과 method의 조합으로 각기 다른 동작을 결정

### 요청하기 - (1) URL과 views 함수 연결하기

모든 요청은 메인 프로젝트 폴더의 `urls.py`로 전달된다고 생각하자.

어떤 url로 들어오면 어떤 함수를 실행할지를 결정!

(url을 분리할 수 있는데, 이 때 필요한 것이 include)

```python
from django.contrib import admin
from django.urls import path
# articles라는 앱의 views 파이썬 파일을 불러오고
from articles import views
urlpatterns = [
    path('admin/', admin.site.urls),
    # 불러온 views 파일의 특정 함수를 개발자가 의도한 url이랑 연결
    # ~/index/ 로 접근하면 view.index 함수가 동작하도록 할거야!
    path('index/', views.index),
    path('greeting/', views.greeting),
    path('dinner/', views.dinner),
]
```

### 요청 처리하기 - (2) view 함수 작성하기
```python
import random
from django.shortcuts import render

# Create your views here.
# 약속: view 함수의 첫 번째 인자는 반드시 request여야 한다.
def index(request):
    # url : ~/index/ 로 접근했을 때 동작하는 함수

    # 어떤 동작? : render
    return render(request, 'index.html')
```

### 요청 처리하기 - (3) template 작성하기

> `index.html`은 `articles`라는 앱의 `templates`라는 폴더 아래에 위치해야 한다.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <h1>만나서 반가워요!!!</h1>
  <a href="/greeting/">greeting</a>
  <a href="/dinner/">dinner</a>
</body>
</html>
```

정리) 기본적인 요청 처리 순서

1. urls - 함수 연결하기
2. views - 함수 작성하기
3. templates - 함수가 탬플릿을 렌더 한다면 탬플릿 작성하기

---


장고 디자인 패턴 (MTV == MVC)

1. Model
2. Template
3. View

---
## Django Template Language

DTL 문법은 파이썬과 비슷하게는 생겼지만 별개의 문법이다.

### DTL - Variable: {{}}
함수의 내용은 다음과 같이 구성할 수 있다.
```python
def greeting(request):
    foods = ['apple', 'banana', 'coconut',]
    info = {
        'name': 'Harry'
    }

    context = {
        'info': info,
        'foods': foods,
    }
```
이 때 실제 greeting에서는 greeting 함수에 있는 변수에 다음과 같이 접근한다.
```html
<!DOCTYPE html>
...
생략
...
<body>
  <h1>안녕하세요. 저는 {{ info.name }} 입니다.</h1>
  <p>제가 좋아하는 음식은 {{ foods }}입니다. </p>
  <p>{{ foods.0 }}을 가장 좋아합니다. </p>
  <a href="/">메인페이지로 가기</a>
  <a href="{% url 'greeting' %}">현재페이지</a>
</body>
</html>
```

특이한 점은 인덱스 접근을 . 으로 한다는 것!
python 문법과 다르게 foods[0]이 아닌 foods.0 이라는 것에 주의하자.

새로운 url인 dinner를 만들어보자.
1. urls에서 `path('dinner/', views.dinner, name='dinner'),` 추가하기

2. views.py 에서 함수 만들기
```python
def dinner(request):
    foods = ['족발', '피자', '햄버거', '초밥',]
    pick = random.choice(foods)
    context = {
        'pick': pick,
        'foods': foods,
    }
    return render(request, 'dinner.html', context)
```

3. `dinner.html` 만들기
4. 원한다면 `index.html`에 `<a href='/dinner/'>저녁메뉴 추천</a>` 꼴로 추가 가능!

`return render(request, 'dinner.html', context)` 를 보면, context를 매개변수처럼 넣어 주었다. 이렇게 하면 context.pick 이 아니라 그냥 pick이라고만 입력해도 사용할 수 있게 된다. 예를 들어
```html
<!DOCTYPE html>
<html lang="en">
...생략...
</head>
<body>
  <h1>오늘 저녁은 {{ pick }}</h1>
```
이와 같이 작성 가능하다.


### DTL - Filter : 변수|필터
length, join 등 html 파일에서 원하는 filter를 적용할 수 있다.
그러나 장고에서는 views에서 주로 파일을 조작하는 것이 좋다.
그래도 사용법을 알아보면
```html
<p>{{ pick }}은 {{ pick|length }}</p>
```
위와 같이 사용할 수 있다.

### DTL - Tag : {% %}

- for
- if
- extends
- block
- comment
- url : `{% url '별명' %}`

```html
{% for food in foods %} 
<!-- 장고의 for tag가 동작한 것. 파이썬의 for문이 아님! -->
      <li>{{ food }}</li>
{% endfor %}
```

주석도 태그이다.
`{% comment %} 주석입니당 {% endcomment %}`
이렇게 사용하는 DTL 주석은 F12를 통해 확인할 수 없는 반면,
`<!-- 주석입니당 -->` 
이렇게 사용하는 html 주석은 F12를 통해 사용 가능하다.

# Template 상속하기

```python
# main폴더의 settings.py
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```
위 코드를 보면 DIRS가 아주 써먹기 좋게 비어 있음을 알 수 있다. 우리는 여기에 경로를 작성해서 템플릿을 상속할 수 있다.


```python
# main폴더의 settings.py
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'firstpjt' / 'templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```
articles의 templates는 기본적으로 사용이 가능한데, 이렇게 새로운 경로를 추가하면 해당 템플릿에 있는 html을 사용할 수가 있다.

이렇게 추가한 템플릿을 사용하려면 우선 맨 처음 extends를 사용해야 한다.
그 후 block 표시한 부분에 원하는 내용을 넣을 수 있다.
```html
  {% extends 'base.html' %} <!-- 부모 템플릿의 경로 -->

  {% block content %}
    <h1>만나서 반가워요!!!</h1>
    <a href="/greeting/">greeting</a>
    <a href="/dinner/">dinner</a>
    <a href="/">뒤로가기</a>
  {% endblock %}
```
일단은 index와 greeting에 이와 같이 처리해주었다.


### throw 예제 (던지고 받기)
1. `path('throw/', views.throw),` 를 urls.py에 추가한다.
2. `def throw(request):
    return render(request, 'throw.html')` 를 views.py에 추가한다.
3. `throw.html`을 만들 것이다.
```html
{% extends 'base.html' %}

{% block content %}

  <h1>THROW</h1>
  <form action="" method="GET">
    <label for="message">Throw: </label>
    <input type="text" name="message" id="message">
    <input type="submit">
    <button>제출></button>  <-- button은 submit이 기본 타입이다. 이것만 써도 됨
  </form>
{% endblock %}
```
이렇게 작성하는데, method를 작성하지 않으면 기본적으로 GET을 사용한다.

이렇게 작성한 Throw를 받으려면 catch를 작성해야 한다.

1. catch url 만들기
`path('catch/', views.catch, name='catch'),`
2. views에 가서 다음의 코드 작성
```html
def catch(request):
    # throw에서 던진 데이터를 받아야 함
    message = request.GET.get('message') 
    # GET은 request의 GET이라는 주머니에 접근하는 방법이고, 이 주머니에는 dictionary가 들어있다. get은 이 딕셔너리의 밸류에 접근하기 위함
    context = {
        'message': message,
    }
    return render(request, 'catch.html', context)
```

### Variable Routing
url을 유동적으로 활용하고 싶다!

1. path('hello/<str:name>/', views.hello)
2. hello 함수를 views에 추가, 이 때 두 번째 매개변수로 name이 들어간다.
```python
def hello(request, name):
    context = {
        'name': name,
    }
    return render(request, 'hello.html', context)
```
3. hello.html을 만들어준다.
```html
{% extends 'base.html' %}
{% block content %}
  <h1>hello</h1>
  <h2>만나서 반가워요 {{ name }}님!!</h2>
{% endblock content %}
```

name 뿐만 아니라 다른 변수도 넣을 수 있다.
```python
def hello(request, name):
    context = {
        'name': name,
        'age': 40
    }
    return render(request, 'hello.html', context)
```

```html
{% extends 'base.html' %}
{% block content %}
  <h1>hello</h1>
  <h2>만나서 반가워요 {{ name }}님!! {{ age }}살 이시군요!</h2>
{% endblock content %}
```



### url 분리하기

모든 요청은 메인 폴더의 urls.py로 전달되는데, 우리는 이 urls.py가 너무 복잡해지면 불편하다. 보다 편리하게 사용하기 위해 앱별로 urls.py를 만들어 구분하는 작업이다.



1. 메인폴더인 firstpjt에서 urls.py에 들어가 import include를 해준다.

```python
# firstpjt urls.py
from django.contrib import admin
from django.urls import path, include
```



2. url을 분리하고자 하는 앱에 urls.py를 만들어준다.

`python manage.py startapp pages` # pages라는 앱을 만들었다.

앱을 등록하고(settings.py의 installed_apps에 등록)

urls.py를 pages에 만들어주면 됨.



3. firstpjt의 urls.py에서 path를 조작해준다.

   include 함수를 사용하여 요청을 넘겨주게 됨

```python
# firstpjt urls.py
urlpatterns = [
    path('admin/', admin.site.urls),
    #articles의 urls로 연결
    path('articles/', include('articles.urls')),
    # url 분리하기 : include('앱이름.urls')
    # ~/pages/ 라는 기본 url로 접근
    # 입력한 url이 pages로 시작한다면 pages에 있는 urls로 이동한다.
    path('pages/', include('pages.urls')), 
]
```

위와 같이 바꿨다면, 

1) 예를 들어 127.0.0.1/articles/~ 경로를 요청하면 해당 요청을 firstpjt의 urls.py가 받아 articles 폴더의 urls.py로 넘겨준다.

2) 127.0.0.1/pages/~ 경로로 요청하면 해당 요청을 firstpjt의 urls.py가 받아 pages 폴더의 urls.py로 넘겨준다.

4. articles나 pages 등의 앱에 있는 urls.py의 내용을 채워준다.

```python
# pages urls.py
from django.urls import path
from . import views

# 반드시 필요한 변수
urlpatterns = [
    path('', views.index),
]
```

```python
# articles urls.py
from django.urls import path #include는 이제 쓰지 않음
# 명시적으로 상대경로 표현
from . import views

urlpatterns = [
    # 불러온 views 파일의 특정 함수를 개발자가 의도한 url이랑 연결
    path('index/', views.index),
    path('greeting/', views.greeting, name='greeting'),
    path('dinner/', views.dinner),
    path('throw/', views.throw),
    path('catch/', views.catch),
    path('hello/<str:name>/', views.hello),
]
```

위의 내용은 articles의 urls.py를 작성한 후의 모습이다.

path가 기존에 `http://127.0.0.1:8000/index` 와 같았다면, 이제는 `http://127.0.0.1:8000/articles/index`와 같다.

5. 당연히 articles나 pages 등의 views.py도 만들어놓아야 한다. 

```python
# pages views.py
from django.shortcuts import render

# Create your views here.
def index(request):
    return render(request)
```

6. views.py가 있으면 당연히 pages 내부에 templates도 있어야 한다. 간단하게 index.html을 만들어보자.

```django
{% extends 'base.html' %}

{% block content %}
<h1>여기는 pages입니다!</h1>
{% endblock content %}
```

만약 pages에서 위와 같이 만든 후에 http://127.0.0.1:8000/pages/index 로 접근하면 제대로 접근이 안 될 수 있는데, 이는 settings.py에서 INSTALLED_APPS에 articles가 pages보다 위에 적혀 있기 때문일 수 있다. 즉, articles에 의해 만들어진 templates 폴더에 있는 index로 접근하기 때문일 수 있다는 것!

이를 방지하기 위해 pages폴더의 templates 폴더 아래에 앱 이름과 같은 pages 폴더를 만들고, 그 아래에 index.html을 넣어준다. 그 후 views.py를 다음과 같이 수정한다.

```python
from django.shortcuts import render

# Create your views here.
def index(request):
    return render(request, 'pages/index.html')
```



### 네이밍 시스템

경로가 길어질수록 코드로 관리하기가 어렵다. 이를 완화하기 위해 name을 설정할 수 있다. 예를 들면 다음과 같다.

```python
# articles urls.py
urlpatterns = [
    # 불러온 views 파일의 특정 함수를 개발자가 의도한 url이랑 연결
    path('index/', views.index, name='index'),
    path('greeting/', views.greeting, name='greeting'),
    path('dinner/', views.dinner, name='dinner'),
    path('throw/', views.throw, name='throw'),
    path('catch/', views.catch, name='catch'),
    path('hello/<str:name>/', views.hello, name='hello'),
]
```

이제 템플릿에서 name=''의 내용을 변수로서 사용할 수 있게 된다.

어떻게 하느냐? 태그 이용!

기존에 `<a href="/dinner/">저녁메뉴 추천</a>` 처럼 작성했다면, 이제는 {% url %} 태그를 사용한다. 

--> `<a href="{% url 'dinner' %}">dinner</a>`

```django
<!--index.html-->
  {% extends 'base.html' %} <!-- 부모 템플릿의 경로 -->

  {% block content %}
    <h1>만나서 반가워요!!!</h1>
    <a href="{% url 'greeting' %}">greeting</a>
    <a href="{% url 'dinner' %}">dinner</a>
    <a href="{% url 'throw' %}">throw</a>
  {% endblock %}
```

이렇게 하고 나면 url을 path에서 변경하더라도 신경 쓰지 않아도 된다!

### {{}} 를 HTML에서 보이도록 쓰고 싶다면?

HTML특수문자 리스트를 참고하면 된다!

`&#123;` : {

`&#125;` : }

```django
<p> &#123; &#123; Escape 문자! &#125; &#125;</p>
```



### app_name 설정하기

app_name을 설정하여 구분을 더 명확하게 할 수 있다.

각 앱의 urls.py에 app_name = 'articles'나

app_name = 'pages' 처럼 앱 이름을 적어 준 후, 다른 모든 연관된 url에 app_name을 붙여준다. 예를 들면 다음과 같다.

```python
# articles urls.py
from django.urls import path
# 명시적으로 상대경로 표현
from . import views

app_name = 'articles'
# 얘가 없으면 아예 돌아가지 않는다. 빈 리스트라도 들어있어야 함.
urlpatterns = [
    # 불러온 views 파일의 특정 함수를 개발자가 의도한 url이랑 연결
    path('', views.index, name='index'),
    path('greeting/', views.greeting, name='greeting'),
    path('dinner/', views.dinner, name='dinner'),
    path('throw/', views.throw, name='throw'),
    path('catch/', views.catch, name='catch'),
    path('hello/<str:name>/<int:age>', views.hello, name='hello'),
]
```

이제 이와 관련된 url에 app_name을 붙인다.

```django
<!-- firstpjt/templates/base.html -->
...생략
<nav class="navbar navbar-light bg-light">
    <div class="container-fluid">
      <a class="navbar-brand" href="{% url 'articles:index' %}">Home</a>
      <a class="navbar-brand" href="{% url 'articles:greeting' %}">인사</a>
    </div>
  </nav>
...생략
```



```django
<!-- articles/index.html  -->
{% extends 'base.html' %} <!-- 부모 템플릿의 경로 -->

  {% block content %}
    <h1>만나서 반가워요!!!</h1>
    <!-- 기존 코드는..<a href="/greeting/">인사</a> -->
    <a href="{% url 'articles:greeting' %}">greeting</a>
    <a href="{% url 'articles:dinner' %}">dinner</a>
    <a href="{% url 'articles:throw' %}">throw</a>
  {% endblock %}
```

