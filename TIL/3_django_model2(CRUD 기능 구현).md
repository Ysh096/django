# django models2 (CRUD 기능 구현)

3/11 workshop을 따라가며 CRUD 기능을 구현해보자.





### 1. Project 만들기

1) `django-admin startproject crud`

2) manage.py가 있는 폴더로 들어가 다음 명령어 실행

`python manage.py startapp articles`

위의 과정을 거치면 crud라는 프로젝트와 그에 속한 articles라는 앱이 만들어진다.

3) settings.py에 들어가 INSTALLED_APP 에 articles를 추가해준다.

```python
INSTALLED_APPS = [
    'articles',
    'django_extensions',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

여기서 django_extensions는 후에 DB를 조작할 때 shell_plus를 이용하기 위한 것임을 알아두자.

---

잠깐! **실행해보기**

`python manage.py runserver` 명령어를 통해 서버를 실행해볼 수 있다. 정상 실행되면 로켓이 발사되는 모습을 웹페이지를 통해 확인할 수 있다. 주소는 127.0.0.1.8000: 이런 느낌이었다.

---

4) base.html 만들기

모든 템플릿의 기본이 되는 base.html을 만든다. 위치는 임의로 할 수 있겠지만 우선 crud, articles 폴더와 같은 위치(manage.py의 위치)에 만들어보자.

- 해당 경로에 templates 폴더 생성 -> base.html 생성

- base.html이 적용되게 하려면 경로를 설정해주어야 한다. settings.py에 들어가 TEMPLATES에서 DIRS를 채워주자.

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                ...
```

여기서 BASE_DIR은 settings.py에 존재하며 다음과 같다.

```python
BASE_DIR = Path(__file__).resolve().parent.parent
```

`__file__`은 settings.py의 절대 경로를 의미하고, 따라서 나의 경우에는 settings.py의 위치가 `~/workshop/crud/crud/settings.py` 이므로 두 번째 parent 경로, 즉 `~/workshop/crud/` 가 BASE_DIR이 된다.

`DIRS': [BASE_DIR / 'templates']`의 의미는 말 그대로, BASE_DIR 바로 아래에 있는 templates를 가장 먼저 확인하겠다는 뜻이고, 그 밑의 `APP_DIRS: True`에 의해 앱의 템플릿들을 확인하게 된다.



base.html의 양식은 기본적으로 내 자유지만, 부트스트랩을 배운 김에 기본 HTML에 부트스트랩 CDN을 추가해주고, 다른 템플릿의 내용들이 들어갈 자리를 뚫어준다.

```django
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <!-- CSS only -->
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-BmbxuPwQa2lc/FVzBcNJ7UAyJxM6wuqIj61tLrc4wSX0szH/Ev+nYRRuWlolflfl" crossorigin="anonymous">
  <title>Document</title>
</head>
<body>
  <div class="container">
    {% block content %}
    {% endblock content %}
  </div>
  <!-- JavaScript Bundle with Popper -->
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta2/dist/js/bootstrap.bundle.min.js" integrity="sha384-b5kHyXgcpbZJO/tY9Ul7kGkf1S0CWuKcCD38l8YkeH8z8QjE0GmW1gYU5S9FOnJ0" crossorigin="anonymous"></script>
</body>
</html>
```

위의 block 부분이 다른 템플릿의 내용이 들어갈 부분이다. 앞으로 모든 템플릿은 base.html을 상속받아 사용한다. (extends -> block, 어떻게 상속받는지는 조금 있다가 확인!)



### 2. urls 설정

앱의 개수가 많아지고 템플릿이 점점 늘어날수록 하나의 urls.py로 모든 템플릿을 관리하기에는 점점 복잡해지게 된다. 따라서 앱별로 템플릿을 분리하여 관리하기 위해 urls.py 또한 앱별로 만들어주게 되고, 이는 `include`를 import 하여 이루어진다.

```python
# crud/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('articles/', include('articles.urls')),
]
```

`path('articles/', include('articles.urls'))`의 의미는, `기본주소/articles`로 접근할 경우 articles 앱에 있는 urls.py로 모든 것을 넘겨주겠다는 뜻이다.

이렇게 urls를 설정하고 나서, app(지금은 articles)의 하위 디렉토리를 보면 urls.py가 없다는 것을 알 수 있다. 앱의 urls.py는 직접 만들어줘야 하며, 기본적으로 들어가야 할 요소들은 다음과 같다.

```python
from django.urls import path
from . import views #현재 경로에 있는 views.py를 불러온다

app_name = 'articles'

urlpatterns = [
    path('', views.index, name='index'),
]
```

app_name은 설정하지 않아도 되지만 후에 보다 편하게 동적으로 주소를 할당하기 위해 사용해주는 것이 좋을 것 같다.

- urlpatterns의 내용 뒤에는 반드시 쉼표를 찍어줘야 한다.

- `path('', views.index, name='index')`의 의미는, 주소가 입력 되었을 때 views.py에 있는 index 함수를 실행한다는 의미이며, name='index'는 이 주소의 이름이 index임을 뜻한다.

- 따라서 지금 index라는 함수를 실행하게 되는 주소는 `기본주소/articles/`가 된다.

- 만약 path가  `path('index', views.index, name='index')` 라면, 주소는 

  `기본주소/articles/index/`가 된다.



이대로 서버를 실행하면 장고에서 index라는 함수가 없다고 메세지를 보내온다. 이제 views.py를 수정해줘야 한다. 그 전에 models.py를 먼저 수정해보자.



### 3. models.py 설정

models.py는 database가 어떤 column 값들을 가지는 row로 이루어져 있는지 설정할 수 있는 파일이다. 우리 프로젝트에서는 다음과 같이 작성하면 된다.

```python
from django.db import models

class Article(models.Model):
    title = models.CharField(max_length=10)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

Article이라는 클래스에 네 개의 요소가 있음을 나타내었다. auto_now_add는 객체(게시글)가 최초로 만들어진 시간을 기록하겠다는 뜻이며, updated_at은 해당 객체의 변화가 있을 시에 기록하겠다는 뜻이다.

이렇게 모델을 설정을 했으면 migration을 해야 한다. 순서는 다음과 같다.

1) `python manage.py makemigrations`: initial_0001.py 등의 model에 대한 설계도가 만들어진다.

2) `python manage.py migrate`: 만들어진 설계도를 DB에 반영한다.

객체의 column 등에 변화가 생기면 반드시 위의 순서로 명령어를 실행해주어야 한다. 

두 개의 명령어가 더 있는데, 한번 살펴보자.

- `python manage.py sqlmigrate`: 마이그레이션에 대한 SQL 구문을 보기 위해 사용한다.
- `python manage.py showmigrations`: 프로젝트 전체의 마이그레이션 상태를 확인하기 위해 사용하며, X 표시가 migrate 되었다는 뜻이다.

위의 두 명령어는 참고용!



# 4. views.py 설정

누군가 주소로 접근 -> views.py의 함수 실행!

여기에서 데이터베이스에 입력되는 데이터를 저장하거나 삭제할 수 있고, 웹페이지에 나타나게 만들수도 있다!

우선 코드를 먼저 살펴보자.

```python
from django.shortcuts import render
from .models import Article # 방금 만든 모델인 Article을 가져온다.

def index(request):
    articles = Article.objects.all() # Article이 가진 모든 내용을 articles에 저장
    context = {
        'articles': articles,
    }
    return render(request, 'articles/index.html', context)
```

위의 코드는 DB가 가진 Article object의 모든 내용을 index 함수에서 변수 형태로 받아서 index.html로 넘겨주는 형태이다. 

- 함수에서 데이터베이스를 조작하기 위해서는 DB API에 대해 알아야 한다. CRUD를 가능케 하는 명령어들에 대해 알아보자.



## DB API(CRUD 동작!)

우선 앞서 settings.py에 추가했던 django_extensions에 의해 우리는 shell_plus를 사용할 수 있게 되었다.

`python manage.py shell_plus`

이제 터미널 창이 파이썬 인터프리터처럼 표현되기 시작한다.



여기서 우리는 DB를 조작할 수 있다.

##### 1) 게시글 추가하는 방법(CREATE)

```python
article = Article() # <Article: Article object (None)>
# 아무것도 입력되지 않은 인스턴스 article 생성
article.title = 'first'
article.content = 'django!'
# 아직도 <Article: Article object (None)>
# DB에 저장하지는 않았기 때문!
article.save()
# 이제는 <Article: Article object (1)>
# DB에 하나의 객체가 생성되었음!

# 다른 방법으로도 추가할 수 있다.
# 방법2
article = Article(title='이거슨 제목', content='이거슨 내용')
article.save()

# 방법3
Article.objects.create(title = '이거슨 제목3', content='이거슨 내용3')
# save할 필요x
```



##### 2) 데이터를 조회하는 방법(READ)

DB에 추가한 데이터를 확인하고자 할 때 다음과 같이 할 수 있다.

 ```python
# 1. DB의 모든 내용 가져오기
article = Article.objects.all()
# DB의 Article의 Query를 모두 article에 저장한다.

# 2. DB의 일부 내용 가져오기
article = Article.objects.get(pk=1)
# pk(primary key, id, 고유값)번째 데이터를 article에 저장한다.
 ```



데이터를 조회할 때 출력형식을 `<Article: Article object (1)>`이런 형태가 아니라 사람이 보기 편하게 바꾸려면 models.py에 다음 코드를 추가하면 된다. (makemigrations는 할 필요x)

```python
#models.py
def __str__(self): # 인스턴스의 출력 형식을 지정하는 매직 메서드 
    return self.title # content나 updated_at 등도 가능!
```



필터를 이용하여 조건에 맞는 데이터만 추려낼 수도 있다.

```python
# content가 'django!'인 객체 가져오기
article = Article.objects.filter(content='django!')


# 컨텐츠에 !가 있는 모든 객체를 가져오기
article1 = Article.objects.filter(content__contains='!')

# Field lookups
# field__lookuptype=value 형태로 사용됨
# pk가 2보다 큰 객체 가져오기(greater than)
lookup_articles = Article.objects.filter(pk__gt=2)
# 쿼리셋을 리턴한다. 개수는 조건 충족하는만큼!

결과 예시
<QuerySet [<Article: first>]>
```

https://docs.djangoproject.com/en/3.1/ref/models/querysets/#gt



##### 3) 내용 수정하는 법(UPDATE)

쉬우니까 코드만 쓰자!

```python
# 우선 수정할 대상이 있어야함
article = Article.objects.get(pk=1)
article.title = '이거슨 변경된 제목'
article.content = '이거슨 변경된 내용'
article.save()
```



##### 4) 내용 삭제하는 법(DELETE)

```python
# 삭제할 대상이 있어야함
article = Article.objects.get(pk=2)
article.delete()
# 대상이 없으면 DoesNotExist 에러가 난다.
```

가져와서 바로 삭제!

**이렇게 CRUD를 가능케하는 명령어들에 대해 알아보았다. 이를 views.py에 적용하여 페이지의 기능을 구현한다.**





# 5. Index.html 만들기

앞서 base.html을 만들어 모든 템플릿이 상속하도록 만들려고 했다. 이게 가능하려면 모든 템플릿의 가장 상단에(주석까지 포함하여 최상단) `{% extends 'base.html' %}`이 작성되어 있어야 한다. (템플릿의 위치는` ~/articles/templates/articles/index.html`)

```django
{% extends 'base.html' %}

{% block content %}
  <h1>INDEX</h1>
  <a href="{% url 'articles:new' %}">NEW</a>
  <br>
  <br>
  {% for article in articles %}
    <h2>제목: {{ article.title }}</h2>
    <p>내용: {{ article.content }}</p>
    <!-- url에 변수가 필요하면 옆에 써주기! -->
    <a href="{% url 'articles:detail' article.pk %}">DETAIL</a>
    <hr>
  {% endfor %}
{% endblock content %}
```



index.html을 시작으로 필요한 템플릿과 함수를 늘려가보자.

인덱스 페이지에는 세 가지 눈여겨볼 점이 있다.

1) 상단의 NEW

- NEW라는 링크는 `articles:new`라는 url로 연결이 된다. 좀 더 자세히 설명해보자.

- 아까 `articles/urls.py`에서 `app_name`을 `articles`로 설정했는데, 여기에서 그 이름이 쓰인다. `articles`라는 앱의 urls.py에서(`기본주소/articles`)에서 `new`라는 이름의 urlpattern을 찾아 연결하게 된다.

- ```python
  # ~articles/urls.py
  # 주소: ~articles/new
      path('new/', views.new, name='new'),
      
      
  # ~articles/views.py
  def new(request):
      # 작성을 완료했을 때
      if request.method == 'POST':
          title = request.POST.get('title')
          content = request.POST.get('content')
  
          article = Article(title=title, content=content)
          article.save()
          # 핵심 부분
          return redirect('articles:detail', article.pk)
      else:
          # new 링크를 눌러 들어올 때    
          return render(request, 'articles/new.html')
      
  
  # ~articles/templates/articles/new.html
  {% extends 'base.html' %}
  
  {% block content %}
    <h1>NEW</h1>
  
    <form method="POST">
      {% csrf_token %}
      <div class="mb-3">
        <label for="title" class="form-label">TITLE</label>
        <input type="text" name="title" class="form-control" id="title" placeholder="please write in with 10 characters or less">
      </div>
      <div class="mb-3">
        <label for="content" class="form-label">CONTENT</label>
        <textarea name="content" id="content" class="form-control" rows="3"></textarea>
      </div>
      <button type="submit" class="btn btn-primary">작성</button>
    </form>
  
    <a href="{% url 'articles:index' %}">BACK</a>
  {% endblock content %}
  
  ```

- 한번 쭉 살펴보고 넘어가자. 만약 index.html 페이지를 누군가 방문해서 NEW라는 링크를 눌렀다고 하자. 그러면 urls.py의 new라는 이름을 갖는 urlpattern에 의해 views.py의 new 함수가 동작하게 되고, 요청(request)이 기본값인 GET으로 이뤄지게 된다. 이 때에는 단순히 new 페이지를 렌더하여 NEW링크를 누른 사람에게 보여주는 역할을 하게 된다.

- NEW 페이지에 들어온 누군가가 내용을 작성하고 작성 버튼을 누르면 POST 메서드를 통해 요청이 전달된다. id를 title과 content로 설정해 놓았기 때문에 

- ```python
  if request.method == 'POST':
      title = request.POST.get('title')
      content = request.POST.get('content')
  ```

  위의 코드를 통해 새롭게 작성된 title과 content값을 가져올 수 있게 되고, 이를 그대로 DB에 저장하는 모습을 볼 수 있다. (CREATE 동작!)

- 그 후 상세정보 페이지를 보기 위해 articles:detail로 redirect하는 모습을 볼 수 있고, detail 페이지에 접근하기 위해서는 pk값이 필요하므로 전달해줘야 함을 알 수 있다.



2) 중단의 for문

- 아까 작성한 views.py를 보면 다음의 코드를 확인할 수 있다.

  ```python
  def index(request):
      articles = Article.objects.all() # Article이 가진 모든 내용을 articles에 저장
      context = {
          'articles': articles,
      }
      return render(request, 'articles/index.html', context)
  ```

- 여기서 context를 통해 `'articles'`라는 데이터베이스의 모든 내용이 담긴 인스턴스가 html 문서로 넘어오게 되고, 문서에서 for문을 사용해 데이터베이스의 내용을 보여주고 있는 것이다.

3) 하단의 DETAIL

- NEW 링크와 마찬가지로 articles의 detail이라는 이름을 가지는 urlpattern을 찾아가는 DETAIL 링크가 있다.

- DETAIL이라는 페이지는 주소가 게시글의 pk에 따라 달라지므로 이를 할당하는 variable routing 방법이 필요하다.

- urls.py를 보면 `path('<int:pk>/', views.detail, name='detail'),`와 같이 표현을 하고 있는데, 이는 int형의 pk 라는 변수가 `~/articles/pk`의 형태로 주소로 입력될 것임을 알려준다. 따라서 우리는 변수 pk가 필요하다!

- 변수 pk를 전달하는 방법으로, 단순히 url 옆에 써주기만 하면 된다!

   `<a href="{% url 'articles:detail' article.pk %}">DETAIL</a>`



다른 페이지들도 이와 같은 형태로 구성할 수 있다. 한 가지 주의해야 했던 점과 함수, url, html 코드를 소개하고 글을 마친다.



# 주의해야 했던 점

delete를 구현할 때, 결과적으로 delete라는 html 페이지는 없음에도 불구하고 해당 주소를 할당해주어야 했다. 예를 들어 path는 다음과 같았다.

```python
# articles/urls.py
path('<int:pk>/delete/', views.delete, name='delete')
```

이는 아래와 같은 detail 페이지의 주소와 delete 명령이 겹치는 것을 막기 위한 장치라고 생각할 수도 있겠다.

```python
path('<int:pk>/', views.detail, name='detail'),
# /delete가 없으면 주소가 겹침!
```



또 detail.html에 위치한 delete 버튼에 대한 form 태그는 다음과 같은데,

```html
<form action="{% url 'articles:delete' article.pk %}" method="POST">
  {% csrf_token %}
  <!-- articles앱의 delete 경로로 보낸다. -->
  <button type="submit" class="btn btn-primary btn-sm m-1">DELETE</button>
</form>
```

그동안 `articles:delete`의 의미가 articles 경로 아래의 delete.html로 전달한다는 의미라고 생각했던 나로서는 당황스러운 코드였다. 곰곰히 고민해본 결과 urlpattern의 delete라는 이름을 가진 경로를 pk값과 함께 찾아간다는 의미라는 생각이 들었고,  순서를 재구성하면 다음과 같을 것이라고 생각했다.

1) 누군가 detail 페이지에 접속

2) delete 버튼을 누름 

3) articles 앱의 delete라는 urlpattern에 pk값을 가지고 찾아감

4) 결국 `path('<int:pk>/delete/', views.delete, name='delete')`에 따라 articles/pk/delete 라는 url이 입력되는 것!

5) 이러한 url에 대한 접근이 감지되면 views.delete 함수가 요청되게 됨

6) 페이지 내부에서 발생한 POST 요청이므로 해당 article을 삭제하는 동작을 수행함!

**{% csrf_token %}**은 페이지에 대한 외부의 공격(내용을 억지로 생성하고 제거하는 등..)을 막는 역할!



# codes

### urls.py

```python
# articles/urls.py
from django.urls import path
from . import views

app_name = 'articles'

urlpatterns = [
    path('', views.index, name='index'),

    # ~articles/new
    path('new/', views.new, name='new'),

    #detail
    # ~articles/번호
    path('<int:pk>/', views.detail, name='detail'),

    #edit
    # ~articles/번호
    path('<int:pk>/edit/', views.edit, name='edit'),

    #delete 경로에 접근하게 되면 views.delete 함수 호출!
    # ~articles/번호/delete
    path('<int:pk>/delete/', views.delete, name='delete')

]
```



### views.py

```python
# articles/views.py
from django.shortcuts import render, redirect
from .models import Article

def index(request):
    articles = Article.objects.all()
    context = {
        'articles': articles,
    }
    return render(request, 'articles/index.html', context)

def new(request):
    # 작성을 완료했을 때
    if request.method == 'POST':
        title = request.POST.get('title')
        content = request.POST.get('content')

        article = Article(title=title, content=content)
        article.save()
        # 핵심 부분
        return redirect('articles:detail', article.pk)
    else:
        # new 링크를 눌러 들어올 때    
        return render(request, 'articles/new.html')


def detail(request, pk):
    article = Article.objects.get(pk=pk)
    context = {
        'article': article,
    }
    return render(request, 'articles/detail.html', context)

def edit(request, pk):
    article = Article.objects.get(pk=pk)
    context = {
        'article': article,
    }
    if request.method == "POST":
        title = request.POST.get("title")
        content = request.POST.get("content")

        article = Article.objects.get(pk=pk)
        article.title = title
        article.content = content
        article.save()
        return redirect('articles:detail', pk)
    else:
        return render(request, 'articles/edit.html', context)

def delete(request, pk):
    article = Article.objects.get(pk=pk)
    if request.method == "POST":
        article.delete()
        # articles라는 앱의 index라는 이름의 경로로 보낸다.
        return redirect('articles:index')
    else:
        return redirect('articles:detail', pk)
```



### index.html

```django
{% extends 'base.html' %}

{% block content %}
  <h1>INDEX</h1>
  <a href="{% url 'articles:new' %}">NEW</a>
  <br>
  <br>
  {% for article in articles %}
    <h2>제목: {{ article.title }}</h2>
    <p>내용: {{ article.content }}</p>
    <!-- url에 변수가 필요하면 옆에 써주기! -->
    <a href="{% url 'articles:detail' article.pk %}">DETAIL</a>
    <hr>
  {% endfor %}
{% endblock content %}
```



### detail.html

```django
{% extends 'base.html' %}

{% block content %}
  <h1>DETAIL</h1>
  <hr>
  <h2>{{ article.title }}</h2>
  <p>{{ article.content }}</p>
  <p>작성일: {{ article.created_at }}</p>
  <p>수정일: {{ article.updated_at }}</p>
  <a class="btn btn-primary btn-sm m-1" href="{% url 'articles:edit' article.pk %}" role="button">EDIT</a>
  <!-- articles앱의 delete 경로로 POST -->
  <form action="{% url 'articles:delete' article.pk %}" method="POST">
    {% csrf_token %}
    <!-- articles앱의 delete 경로로 보낸다. -->
    <button type="submit" class="btn btn-primary btn-sm m-1">DELETE</button>
  </form>
  <a class="btn btn-primary btn-sm m-1" href="{% url 'articles:index' %}" role="button">BACK</a>
{% endblock content %}
```



### edit.html

```django
{% extends 'base.html' %}

{% block content %}
  <h1>EDIT</h1>
  <form method="POST">
    {% csrf_token %}
    <div class="mb-3">
      <label for="title" class="form-label">TITLE</label>
      <input type="text" name="title" value="{{ article.title }}"class="form-control" id="title" placeholder="please write in with 10 characters or less">
    </div>
    <div class="mb-3">
      <label for="content" class="form-label">CONTENT</label>
      <textarea name="content" id="content" class="form-control" rows="3">{{ article.content }}</textarea>
    </div>
    {% comment %} type=submit이어야 제출 가능 {% endcomment %}
    <button type="submit" class="btn btn-primary">수정</button>
  </form>
  <a href="{% url 'articles:index' %}">BACK</a>
{% endblock content %}
```



### new.html

```django
{% extends 'base.html' %}

{% block content %}
  <h1>NEW</h1>

  <form method="POST">
    {% csrf_token %}
    <div class="mb-3">
      <label for="title" class="form-label">TITLE</label>
      <input type="text" name="title" class="form-control" id="title" placeholder="please write in with 10 characters or less">
    </div>
    <div class="mb-3">
      <label for="content" class="form-label">CONTENT</label>
      <textarea name="content" id="content" class="form-control" rows="3"></textarea>
    </div>
    <button type="submit" class="btn btn-primary">작성</button>
  </form>

  <a href="{% url 'articles:index' %}">BACK</a>
{% endblock content %}
```

