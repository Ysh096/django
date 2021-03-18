# Static and Media

## 정적 파일(static files) <-DB와 상관없는 정보들!

- 웹 사이트의 구성 요소 중에서 image, css, js 파일과 같이 해당 내용이 고정되어, 응답을 할 때 별도의 처리 없이 파일 내용을 그대로 보여주면 되는 파일
- 사용자의 요청에 따라 내용이 바뀌는 것이 아니라 요청한 것을 그대로 응답! 하면 되는 파일

- 기본 static 경로: app_name/static/

배경 이미지, css 코드, ...

사용자의 요청에 따라 변경하고, 저장하고 하는 것은 미디어 파일이라고 불릴 것이다.



**settings.py 설정**

- STATIC_ROOT
  - collectstatic이 배포를 위해 정적 파일을 수집하는 절대 경로
  - collectstatic: 프로젝트 배포 시 흩어져 있는 정적 파일들을 모아 특정 디렉토리로 옮기는 작업

- STATIC_URL
  - STATIC_ROOT에 있는 정적 파일을 참조 할 때 사용할 URL

- STATICFILES_DIRS
  - app내의 static 경로를 사용하는 것 외에 추가적인 정적 파일 경로 정의



```python
INSTALLED_APPS = [
    'articles',
    
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
맨 마지막 줄에 static 파일을 관리하기 위한 앱이 있고,

STATIC_URL = '/static/'

위 URL이 기본값으로 잡혀있다. 기본적으로 app_name 내에서 static 폴더를 찾아 그 아래의 경로를 찾아가게 된다. 만약 앱이 아닌 프로젝트 내에 있는 base.html에 적용하고 싶다면 다음의 코드를 추가해야 한다.

# 추가적으로 DIRS를 등록하는 경우
# static의 기본 시작 경로가 app 단위라서 앱 단위가 아닌 base.html에서 사용하려면 아래처럼 해야함!
STATICFILES_DIRS = [
    # base.html에서 static을 사용하기!
    BASE_DIR / 'django_form' / 'static',
    # 프로젝트 밖에다 구성한다면?
    BASE_DIR / 'static',
]
```



경로는 확인했고, 우리가 어디선가 가져온 static image를 사용하기 위해서는 html 파일에 다음과 같이 작성해야 한다.

이미지 주소: `articles(app_name)/static/articles/images/muyaho.jpg`

```django
# articles/index.html의 상단

{% extends 'base.html' %}
{% load static %}
{% block content %}
<img src="{% static 'articles/images/muyaho.jpg' %}" % alt="">

<table class="table">
  <thead>
    <tr>
      <th scope="col">#</th>
      <th scope="col">제목</th>
      <th scope="col">작성시간</th>
    </tr>
  </thead>
```



앞서 추가한 경로를 이용하여 base.html에 사진을 적용하고 싶다면, 프로젝트 단위에 static 폴더를 새로 만들고 사진을 넣어야 한다.

이미지 주소: `django_form/static/images/muyaho.jpg`

base.html 파일에 작성해야 할 것은 다음과 같다.

```django
{% load static %}
...
 <img src="{% static 'images/muyaho.jpg' %}" alt="">
```



이미지 뿐만 아니라 css 파일도 static 적용이 가능하다.

css는 우선 큰 틀에 적용한 뒤 세부 사항에 적용하는 것이 좋다. 따라서 base.html에 큰 css 파일을 적용하는데, css파일의 경로는 일단 다음과 같다.

`django_form/static/stylesheets/style.css`

```css
# style.css 파일 예시
h1 {
  color: red;
}
h2 {
  color: mediumaquamarine;
}
```



이런 스타일 시트가 있으면, 이를 적용하기 위해 base.html에 다음과 같이 코드를 작성해야 한다.

```django
{% load static %}
...
  <link rel="stylesheet" href="{% static 'stylesheets/style.css' %}">
  {% block css %}
  {% endblock css %}
  <title>Document</title>
</head>
...
  <h1>그만큼 신나시는거지</h1>
  <h2>123123123</h2>
```

block 부분은 base.html을 상속받는 앱의 html 파일들에 적용할 세부 css 파일을 위해 작성한 것이다. 이제 확인을 해보면 articles 경로의 h1은 빨간색, h2는 미디엄아쿠아마린 색으로 변해있다.



앱의 html 파일에 다른 스타일을 적용하기 위해서는, 새로운 style 시트를 다음의 경로에 만들어야 한다.

`articles/static/articles/stylesheets/style.css`

```css
# style 예시
h2 {
  color: brown;
}
```

이를 detail.html에 적용하고 싶다고 해보자.

```django
# articles/templates/articles/detatil.html

{% extends 'base.html' %}
{% load static %}

{% block css %}
<link rel="stylesheet" href="{% static 'articles/stylesheets/style.css' %}">
{% endblock css %}

{% block content %}
<h2 class="title-color">{{ article.title }}</h2>
<p>{{ article.content }}</p>

<a href="{% url 'articles:update' article.pk %}">수정</a>
<a href="{% url 'articles:delete' article.pk %}">GET으로 삭제시도</a>
<form action="{% url 'articles:delete' article.pk %}" method="POST">
  {% csrf_token %}
  <button>POST로 삭제</button>
</form>
{% endblock content %}
```

상단에서 load static을 해주고, base.html에서 뚫어준 block 부분에 stylesheet의 link를 가져오면 된다. 이제 세부 내용에 들어가서 스타일을 보면 h2태그는 갈색으로 바뀌어 있다.



# Media

사용자가 웹에서 업로드 하는 정적 파일

- image, pdf, video 등



- MEDIA_ROOT
  - 사용자가 업로드 한 파일을 보관할 디렉토리 절대 경로
  - 실제 해당 파일의 업로드가 끝나면 어디에 파일에 저장되게 할 지 경로

- MEDIA_URL
  - MEDIA_ROOT에서 제공되는 미디어를 처리하는 URL
  - 업로드 된 파일의 주소(URL)를 만들어 주는 역할 



- MEDIA_URL 및 STATIC_URL은 서로 다른 값을 가져야 함



1) django media file 검색

2) Managing files 를 읽어보면, MEDIA_ROOT와 MEDIA_URL을 참고하고 있음을 알 수 있다.

3) models.py에 다음과 같이 작성

```python
from django.db import models


class Article(models.Model):
    title = models.CharField(max_length=10)
    content = models.TextField()
    image = models.ImageField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

image를 추가하였고, 비어있어도 되도록 blank=True(비어있으면 비어있는 스트링이 들어가게됨)



4) 이렇게만 저장하면 Pillow라는 것을 설치해야 한다고 알려줌.

`pip install Pillow`



5) requirements에 저장

`pip freeze > requirements.txt`



6) models.py에 변경사항이 발생했으므로 makemigrations - migrate



7) 서버를 돌려 새 글 작성을 눌러보면 파일 선택이라는 버튼이 생겼음을 알 수 있다.

이 때 인풋 타입은 file이다.

눌러서 사진을 고르고 저장을 해보면 사진이 저장이 된 것 같지는 않다!

모델만 추가해서는 이미지 파일을 제대로 사용할 수 없다. form은 모델을 바탕으로 form 태그를 생성해주고 있기 때문에 관련 설정을 마무리 해줘야 한다.



8) new.html로 이동

```django
{% extends 'base.html' %}

{% block content %}
<form method="POST" enctype="multipart/form-data">
  {% csrf_token %}
  {{ form.as_p }}
  <button class="btn btn-primary">제출</button>
</form>
{% endblock content %}
```

enctype을 검색해서 살펴보면, input 타입이 file이면 multipart/form-data를 추가해줘서 file이 들어갈 수 있는 form 데이터임을 알려주어야 함을 알 수 있다.



9) views.py의 create 함수

ArticleForm의 두 번째 인자로 files가 들어가야 한다.

```python
def create(request):
    """
    GET : 새 게시글을 작성하는 탬플릿을 랜더
    POST : DB에 새 게시글 정보 저장(생성)
    """
    # url은 하나만 쓰고
    # method로 동작을 구분!

    if request.method == 'POST':
        form = ArticleForm(request.POST, request.FILES)
...
```

request POST를 담는 주머니가 따로 있고, FILES를 담는 주머니가 따로 있다!

여기까지 작성해도 새 글 작성에 이미지를 첨부해도 저장이 되지 않는다. 하지만 image 전달은 잘 되고 있다. 이게 어디에 저장이 되는가가 문제!



10) 경로를 살펴보면 가장 바깥(장고의 기본 경로)에 사진이 저장이 되어 있음을 알 수 있다. 그러나 admin 페이지에서 확인을 해보면 사진을 눌러도 볼 수가 없다. 경로가 설정이 되어 있지 않기 때문이다.

11) settings.py 에 다음의 코드를 추가

```python
# Media: 사용자가 우리의 서비스를 사용하면서 업로드하는 파일
MEDIA_URL = '/media/'

MEDIA_ROOT = BASE_DIR
```



12) 프로젝트 폴더의 urls.py에 들어가 다음의 같이 변경

```python
from django.contrib import admin
from django.urls import path, include
# Media를 위해 추가
from django.conf.urls.static import static
from django.conf import settings
# URL patterns에 대한 리스트를 자동으로 실행해주는 static을 임포트
urlpatterns = [
    path('admin/', admin.site.urls),
    path('articles/', include('articles.urls')),
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
# 앞에는 media url(settings.py의 내용)
# 뒤에는 document root(settings의 MEDIA_ROOT를 넣어라 라고 알려줌)
```



13) 이렇게까지 해도 이미지를 볼 수가 없다. 이미지가 이상한 곳에 저장되기 때문에(기본 경로)

settings.py에 들어가 다음과 같이 수정한다.

```python
# Media: 사용자가 우리의 서비스를 사용하면서 업로드하는 파일
# template에서 사용하는 미디어 파일 참조 URL
# ~/media/실제파일이름
MEDIA_URL = '/media/'

# 프로젝트에서 미디어 파일이 저장되는 공간
MEDIA_ROOT = BASE_DIR / 'media'

```

이제 새 글 작성에 사진을 첨부하면 media 폴더가 생기며 거기에 사진이 저장된다.

사용자로부터 받은 미디어 파일이 이 폴더에 저장이 되며, 이는 .gitignore에 추가해줘야 한다. 



14) 이제 detail에 다음의 코드를 추가하면 사진을 볼 수 있다.

```django
<img src="{{ article.image.url }}">
```

image의 url에 접근한다는 뜻! (지금까지 그에 대한 설정을 해왔음)



15) article에 이미지가 없어도 파일이 작성되게 만들기

```django
{% extends 'base.html' %}
{% load static %}

{% block css %}
<link rel="stylesheet" href="{% static 'articles/stylesheets/style.css' %}">
{% endblock css %}

{% block content %}
<div class="card text-center">
  <div class="card-header">
    {{ article.title }}
  </div>
  <div class="card-body">
    {% if article.image %}
    <img class="card-img-top" src="{{ article.image.url }}">
    {% else %}
    <p>이미지 없음</p>
    {% endif %}
    <p class="card-text">{{ article.content }}</p>
    <a href="{% url 'articles:update' article.pk %}" class="btn btn-success">수정</a>
    <form action="{% url 'articles:delete' article.pk %}" method="POST">
      {% csrf_token %}
      <button class="btn btn-danger">삭제</button>
    </form>
  </div>
  <div class="card-footer text-muted">
    {{ article.updated_at }}
  </div>
</div>
{% endblock content %}
```

if문으로 분기!



### update

1) edit.html 수정

```django
# articles/edit.html

{% extends 'base.html' %}

{% block content %}
<form method="POST" enctype="multipart/form-data">
  {% csrf_token %}
  {{ form.as_p }}
  <button>수정</button>
</form>
{% endblock content %}
```



2) views.py의 update 함수 수정

```python
# views.py update 함수

@require_http_methods(['GET', 'POST'])
def update(request, pk):
    """
    GET : 특정 게시글(pk)을 수정하는 탬플릿을 랜더
    POST : DB에 특정 게시글(pk) 정보 수정
    """
    # 어떤 게시물? pk로 찾기!
    # article = Article.objects.get(pk=pk)
    article = get_object_or_404(Article, pk=pk)

    if request.method == 'POST':
        # POST : DB 특정 게시글 정보 수정
        # ModelForm 클래스로 인스턴스를 생성!
        form = ArticleForm(request.POST, request.FILES, instance=article)
        # request.FILES를 추가했다.
```





### admin

`python manage.py createsuperuser`



```python
from django.contrib import admin
from .models import Article
# Register your models here.

class ArticleAdmin(admin.ModelAdmin):
    list_display = ('pk', 'title', 'content', 'image', 'created_at', 'updated_at')

admin.site.register(Article, ArticleAdmin)
```

