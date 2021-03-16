# README

## form을 활용하는 방법!



base.html까지 만든 후에 가장 먼저 해야 할 것은 모델을 설계하는 것이다. 그런데 우리는 모델이 어떻게 생겼는지 알기 때문에 그대로 적어준다.

### 1. model 설계 및 DB 시작

```python
from django.db import models

# Create your models here.
class Article(models.Model):
    title = models.CharField(max_length=10)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

```python
$ python manage.py makemigrations

$ python manage.py migrate
```

이렇게 하고 나면 db.sqlite3가 생기는데, 우리는 DB를 git에서 관리하지 않으므로 gitignore에 다음 사항을 추가한다.

`*.sqlite3`

sqlite3 확장자를 가지는 모든 파일을 git에서 제외한다는 뜻이다.

`__pycache__` 얘도 써주자.



### 2. URL 설정

```python
# django_form/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('articles/', include('articles.urls')),
]
```



```python
# articles/urls.py
from django.urls import path
from . import views

app_name = 'articles'
urlpatterns = [
    path('', views.index, name='index'),
]
```



### 3. index함수와 index.html 만들기

```python
# articles/views.py
from django.shortcuts import render

# Create your views here.
def index(request):
    '''
    모든 게시물을 보여주는 템플릿을 렌더하는 함수
    '''
    Article.objects.order_by('-pk') #pk 역순으로 가져옴
    context = {
        'articles': articles # index.html에서 쓰일 데이터
    }
    return render(request, 'articles/index.html', context)
```

```python
# articles/index.html
{% extends 'base.html' %}

{% block content %}
{{ articles }} # querySet 형태의 결과를 보여줌 
{% endblock content %}
```



### 4. detail 페이지 만들기

```python
# articles/urls.py
path('<int:pk>/', views.detail, name='detail'),

# articles/views.py
from .models import Article # 추가!
def detail(request, pk):
    '''
    특정 게시물(pk)의 상세 내용을 보여주는 템플릿을 렌더하는 함수
    '''
    article = Article.objects.get(pk=pk)
    context = {
        'article': article,
    }
    return render(request, 'articles/detail.html', context)


# detail.html
{% extends 'base.html' %}

{% block content %}
{{ article }}
{% endblock content %}
```



### 5. new 만들기(나중에 create와 합침)

```python
나중에 수정하지 않을 new.html만 써보자.
# new.html
{% extends 'base.html' %}

{% block content %}
<form action="{% url 'articles:create' %}" method="POST">
  {% csrf_token %}
  <input type="text" name="title">
  <textarea name="content" id="" cols="30" rows="10"></textarea>
  <button>제출</button>
</form>
{% endblock content %}
```



### 6. create 만들기

```python
# new에서 create로 연결이 되면 실행됨
# articles/urls.py
path('create/', views.create, name='create')

# articles/views.py
def create(request):
    article = Article()
    article.title = request.POST.get('title')
    article.content = request.POST.get('content')
    article.save()
    return redirect('articles:index')
```



### 7. 입력 요구조건에 맞지 않는 경우?

```python
예를 들어 타이틀 new에서 입력시에 10자 제한을 줬는데, 보통은 그럴 일 없겠지만 누군가 개발자 도구(F12)에 들어가 100자로 제한을 풀어버리고 작성을 할 수도 있다. 이걸 방지하기 위해 우리는 views.py의 create 함수를 아래와 같이 수정할 수 있다.

def create(request):
    article = Article()
    title = request.POST.get('title')
    if len(title) > 10:
        return redirect('articles:new')

    article.title = title
    article.content = request.POST.get('content')
    article.save()
    return redirect('articles:index')
```

이제 누군가 F12를 눌러서 조건 수정을 해서 제출을 하면 new 페이지로 redirect 되고, 결과가 저장되지 않는다.

이렇게 데이터 관리를 위해서는 데이터를 저장하기 전에 유효성 검사가 꼭 필요하다. 장고는 이러한 **유효성 검사 기능**을 따로 제공하는데, **이 기능이 Form 이다!**



이제 new와 create를 합치자.

### 8. new와 create 합치기

urls에서 new path를 지우고, create 함수에서 동작을 구분한다.

```python
# articles/urls.py
from django.urls import path
from . import views

app_name = 'articles'
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:pk>/', views.detail, name='detail'),
    path('create/', views.create, name='create'),
]
```

new 함수는 단순히 렌더하는 함수로서, 불필요하므로 지우고 create에 조건을 추가한다.

그 전에 form을 만들어보자.



#### 8-1 forms.py 만들기

articles/forms.py 를 생성한다.

```python
from django import forms

# form class를 정의
# 1. HTML form tag가 자동 생성된다.
# 2. 유효성 검사
class ArticleForm(forms.Form):
    title = forms.CharField(max_length=10)
    content = forms.CharField(widget=forms.Textarea)
```

이제 ArticleForm을 불러오면 자동으로 form 태그가 만들어지고, input 태그는 그 밑에 설정한 것들로 만들어진다. CharField를 하면 기본적으로 text input이 입력이 되는 input 태그가 만들어지게 되고, widget을 textarea로 입력하면 textarea형 input 태그가 만들어지게 된다.



#### 8-2 views.py에 내용 추가하기

```python
# articles/views.py
# 우선 다음의 코드를 추가해준다.
from .forms import ArticleForm

# 나중에 수정해 줄거지만, 일단 다음과 같이 쓴다.
def create(request):
    """
    GET : 새 게시글을 작성하는 템플릿을 렌더
    POST: DB에 새 게시글 정보 저장(생성)
    """
    # url은 하나만 쓰고 method로 동작을 구분!
    if request.method == "POST":
        article = Article()
        title = request.POST.get('title')
        if len(title) > 10:
            return redirect('articles:new')
        
        # 유효성 검사
        article.title = title
        article.content = request.POST.get('content')
        article.save()
        return redirect('articles:index')
    else:
        form = ArticleForm()

        context = {
            'form': form
        }
        return render(request, 'articles/new.html', context)
```



#### 8-3 new.html에 form 추가

```django
{% extends 'base.html' %}

{% block content %}
<form action="{% url 'articles:create' %}" method="POST">
  {% csrf_token %}
  <!-- 기존의 코드  
  <input type="text" name="title">
  <textarea name="content" id="" cols="30" rows="10"></textarea>
  -->
  
  {{ form }} <!-- form을 이용한 코드 -->
  <button>제출</button>
</form>
{% endblock content %}
```

form을 이용하여 태그를 사용하지 않고 간결하게 html 파일을 표현 가능하다. (라벨까지 써줌)

form 태그의 인풋 태그를 p tag나 ul 태그, table 처럼 쓸 수도 있다!

```
{{ form.as_p }}
{{ form.as_ul }}
{{ form.as_table }}
```

그런데 이 form의 경우에는 모델과 직접적인 연관이 없고, 사실 우리가 자주 쓰지는 않을 것이다.

(forms.py의 다음 코드는 모델과 이름을 통일했기 때문에 연관이 생긴 것이고, 이름이 다르면 연관이 없어진다.)

    title = forms.CharField(max_length=10)
    content = forms.CharField(widget=forms.Textarea)
대신 우리가 자주 쓸 form은 ModelForm이라는 것이다.



### 9. ModelForm

```python
from django import forms
from .models import Article # 꼭 추가해줘야 한다.
# ModelForm : 이미 정의한 Model과 연관있는 Form
class ArticleForm(forms.ModelForm):
    class Meta:
        # Model 연결
        model = Article
        fields = '__all__' #model의 모든 field값을 사용하겠다는 뜻
```

굳이 title = ~, content = ~ 를 쓸 필요 없이 fields = `'__all__'`로 연결이 된다!



이제 create 함수는 다음과 같이 변경한다.

```python
def create(request):
    """
    GET : 새 게시글을 작성하는 템플릿을 렌더
    POST: DB에 새 게시글 정보 저장(생성)
    """
    # url은 하나만 쓰고 method로 동작을 구분!
    if request.method == "POST":
        # form 가져오기(request.POST는 queryDict, Dict를 통째로 넘겨준다.)
        form = ArticleForm(request.POST)
        # 이제 request.POST가 들어간 Form의 인스턴스가 하나 생겼다.
        # 이제 form안에 있는 is_valid를 사용하자.
        if form.is_valid(): # True or False
            # form이 유효하면
            article = form.save() # 얘는 단독으로 쓸 수 없고, valid 이후에만 사용 가능하다! (그냥 쓰면 오류남)
            # 이제 누가 개발자 도구에 들어가 required 조건을 지우고 제출해도 저장이 되지 않음!
            # form.save()는 반환값이 있다.
            return redirect('articles:detail', article.pk)
        return redirect('articles:create') # form이 유효하지 않으면 다시 create 화면이 보이도록!
    else:
        form = ArticleForm()

        context = {
            'form': form
        }
        return render(request, 'articles/new.html', context)
```



### 10. widget이란?

``widgets`` is a dictionary of model field names mapped to a widget.

모델 필드와 관련된 설정들을 딕셔너리 형태로 제공

https://github.com/django/django/blob/main/django/forms/models.py



```python
# form을 다음과 같이 구성하면 input의 부가 기능이나 형태를 따로 설정할 수 있다.
class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = '__all__'
        widgets = {
            'title': forms.TextInput(attrs={
                'class': 'my-title',
                'placeholder': '제목을 입력해주세요!',
            })
        }
```

여기서 부트스트랩의 class 설정을 할 수가 있는데, 다음과 같이 하면 된다.

```python
class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = '__all__'
        widgets = {
            'title': forms.TextInput(attrs={
                'class': 'form-control', # 원하는 부트스트랩 특성 class 넣기
                'placeholder': '제목을 입력해주세요!',
            })
            
        }
```

class는 여러 개 넣을 수 있다. 'form-control my-title bg-white' 등등..

content도 바꿔보고 싶다면 다음과 같이 할 수 있다.

```python
class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = '__all__'
        widgets = {
            'title': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': '제목을 입력해주세요!',
            }),
            'content': forms.Textarea(attrs={
                'class': 'form-control',
            
        })
        }
```

이제 약간 느낌이 온다. attrs에 원하는 설정을 딕셔너리 형태로 넣는다!



그런데 위 방법은 권장하는 방법이 아니라고 한다. 다음의 형태가 권장 형태이다.

```python
class ArticleForm(forms.ModelForm):
    title = forms.CharField(
        label='제목',
        widget=forms.TextInput(
            attrs={
                'class': 'form-control'
            }
        )
    )
    content = forms.CharField(
        label='내용',
        widget=forms.Textarea(
            attrs={
                'class': 'form-control'
            }
        )
    )
    class Meta:
        model = Article
        fields = '__all__'
```



### 11. templates 다듬기

이제 html을 좀 더 다듬고 Update와 delete를 만들어보자.

```django
<!-- base.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-BmbxuPwQa2lc/FVzBcNJ7UAyJxM6wuqIj61tLrc4wSX0szH/Ev+nYRRuWlolflfl" crossorigin="anonymous">
  <title>Document</title>
</head>
<body>
  <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
    <div class="container-fluid">
      <a class="navbar-brand" href="#">Navbar</a>
      <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
      </button>
      <div class="collapse navbar-collapse" id="navbarSupportedContent">
        <ul class="navbar-nav me-auto mb-2 mb-lg-0">
          <li class="nav-item">
            <a class="nav-link active" aria-current="page" href="{% url 'articles:index' %}">Home</a>
          </li>
          <li class="nav-item">
            <a class="nav-link" href="{% url 'articles:create' %}">새 글 쓰기</a>
          </li>
        </ul>
      </div>
    </div>
  </nav>
  <div class="container">
    {% block content %}
    {% endblock content %}
  </div>
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta2/dist/js/bootstrap.bundle.min.js" integrity="sha384-b5kHyXgcpbZJO/tY9Ul7kGkf1S0CWuKcCD38l8YkeH8z8QjE0GmW1gYU5S9FOnJ0" crossorigin="anonymous"></script>
</body>
</html>
```

navbar에 home버튼, 새 글 쓰기 버튼을 추가하였다. 



```django
<!-- index.html -->

{% extends 'base.html' %}

{% block content %}
<table class="table">
  <thead>
    <tr>
      <th scope="col">#</th>
      <th scope="col">제목</th>
      <th scope="col">작성시간</th>
    </tr>
  </thead>
  <tbody>
    {% for article in articles %}
      <tr>
        <th scope="row">{{ article.pk }}</th>
        <td>
          <a href="{% url 'articles:detail' article.pk %}" class="text-decoration-none">
            {{ article.title }}
          </a>
        </td>
        <td>{{ article.created_at }}</td>
      </tr>
    {% endfor %}
  </tbody>
</table>
<div class="d-flex justify-content-center">
  <nav aria-label="Page navigation example">
    <ul class="pagination">
      <li class="page-item"><a class="page-link" href="#">Previous</a></li>
      <li class="page-item"><a class="page-link" href="#">1</a></li>
      <li class="page-item"><a class="page-link" href="#">2</a></li>
      <li class="page-item"><a class="page-link" href="#">3</a></li>
      <li class="page-item"><a class="page-link" href="#">Next</a></li>
    </ul>
  </nav>
</div>

{% endblock content %}
```

데이터를 표 형태로 만들었고, pagination 기능을 대충 넣었다. (기능은 안함)



### 12. delete 만들기

```python
# articles/urls.py
path('<int:pk>/delete/', views.delete, name='delete')
```

```python
# articles/views.py
def delete(request, pk):
    '''
    특정 게시물을 DB에서 삭제
    '''
    article = Article.objects.get(pk=pk)
    if request.method == 'POST':
        article.delete()
        return redirect('articles:index')
    return redirect('articles:detail', pk)
```

```python
{% extends 'base.html' %}

{% block content %}
<h2>{{ article.title }}</h2>
<p>{{ article.content }}</p>
<a href="">수정</a>
<form action="{% url 'articles:delete' article.pk %}" method="POST">
  {% csrf_token %}
  <button>POST로 삭제</button>
</form>

{% comment %} <a href="{% url 'articles:delete' article.pk %}">삭제</a> {% endcomment %}  <--얘는 GET으로 접근하는 경우다.. 
{% endblock content %}
```



### 13. update 만들기

```python
# articles/urls.py
path('<int:pk>/update/', views.update, name='update'),
```

```python
# articles/views.py
def update(request, pk):
    '''
    GET: 게시글을 수정하는 템플릿을 렌더
    POST: DB에 특정 게시글(pk) 정보 수정
    '''
    if request.method == 'GET':
        # 어떤 게시물을 수정할지 pk로 찾기
        article = Article.objects.get(pk=pk)

        #forms를 사용할 것! instance에 위에서 찾은 article instance를 넘겨준다.
        form = ArticleForm(instance=article)
        # 생성된 form 인스턴스에는 article에 대한 정보가 담긴다.
        context = {
            'form': form
        }
        return render(request, 'articles/edit.html', context)
        # 이렇게 form을 넘겨주고, html 파일에서 변수 형태로 간단히 {{ form }}만 써주면
        # django에서 알아서 update해야 할 내용(이미 적혀있던 내용)을 적절한 위치에 뿌려준다!
        # ArticleForm에는 이미 input이나 제한 조건 등이 담겨있다. (틀이 잡혀있다.)
    
    # POST인 경우(수정완료 버튼을 누른 경우)
    else:
        # DB 특정 게시글 정보 수정
        article = Article.objects.get(pk=pk)
        
        # ModelForm 클래스로 인스턴스를 생성!
        form = ArticleForm(request.POST, instance=article) 
        #사용자로부터 입력받은 정보 request.POST를 이용하여 내가 가진 정보인 article을 수정한다.
        if form.is_valid():
            form.save() # return값이 발생하지만 굳이 받을 필요는 없음. (pk가 이미 있음)
            return redirect('articles:detail', pk)
        else: # 양식에 맞지 않는 입력인 경우
            return redirect('articles:update', pk)
```

```django
# articles/edit.html

{% extends 'base.html' %}

{% block content %}
<form method="POST">
  {% csrf_token %}
  {{ form.as_p }}
  <button>수정</button>
</form>
{% endblock content %}
```



### 코드 개선!!

update와 create에서 몇 가지 개선을 할 수 있다.

```python
def update(request, pk):
    '''
    GET: 게시글을 수정하는 템플릿을 렌더
    POST: DB에 특정 게시글(pk) 정보 수정
    '''
    # 어떤 게시물을 수정할지 pk로 찾기
    article = Article.objects.get(pk=pk)

        # POST인 경우(수정완료 버튼을 누른 경우)
    if request.method == 'POST':
        # ModelForm 클래스로 인스턴스를 생성!
        form = ArticleForm(request.POST, instance=article) 
        #사용자로부터 입력받은 정보 request.POST를 이용하여 내가 가진 정보인 article을 수정한다.
        if form.is_valid():
            form.save() # return값이 발생하지만 굳이 받을 필요는 없음. (pk가 이미 있음)
            return redirect('articles:detail', pk)
    else:
        #forms를 사용할 것! instance에 위에서 찾은 article instance를 넘겨준다.
        form = ArticleForm(instance=article)
        # 생성된 form 인스턴스에는 article에 대한 정보가 담긴다.

    context = {
        'form': form # context는 form을 if와 else 두 경우 모두 가져오므로 꺼낼 수 있음.
        # + 이렇게하면 is_valid의 else문이 필요 없어지게 된다. return값이 없으면 여기를 거치게 되니깐!
    }
    return render(request, 'articles/edit.html', context)
        # 이렇게 form을 넘겨주고, html 파일에서 변수 형태로 간단히 {{ form }}만 써주면
        # django에서 알아서 update해야 할 내용(이미 적혀있던 내용)을 적절한 위치에 뿌려준다!
        # ArticleForm에는 이미 input이나 제한 조건 등이 담겨있다. (틀이 잡혀있다.)
```

바뀐 점: GET과 POST의 순서, is_valid의 else 경우가 필요없어짐, context를 밖으로 꺼냄..



```python
# articles/views.py
def create(request):
    """
    GET : 새 게시글을 작성하는 템플릿을 렌더
    POST: DB에 새 게시글 정보 저장(생성)
    """
    # url은 하나만 쓰고 method로 동작을 구분!
    if request.method == "POST":
        # form 가져오기(request.POST는 queryDict, Dict를 통째로 넘겨준다.)
        form = ArticleForm(request.POST)
        # 이제 request.POST가 들어간 Form의 인스턴스가 하나 생겼다.
        # 이제 form안에 있는 is_valid를 사용하자.
        if form.is_valid(): # True or False
            # form이 유효하면
            article = form.save() # 얘는 단독으로 쓸 수 없고, valid 이후에만 사용 가능하다! (그냥 쓰면 오류남)
            # 이제 누가 개발자 도구에 들어가 required 조건을 지우고 제출해도 저장이 되지 않음!
            # form.save()는 반환값이 있다.
            return redirect('articles:detail', article.pk)
    else:
        form = ArticleForm()

    context = {
        'form': form
    }
    return render(request, 'articles/new.html', context)
```

is_valid가 유효하지 않은 경우의 코드를 제거, context 이하를 밖으로 꺼냄!



**POST만 create 동작으로 사용하고, 나머지는 페이지를 렌더링할 때 사용한다.**

