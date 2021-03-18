# django project 순서



1) `python -m venv venv && source venv/Scripts/activate`

2) `pip install django`

3) .gitignore, README 만들기

4) `django-admin startproject practice .`

5) `python manage.py startapp articles`

6) articles 앱 등록

7) model 설계

*주의: blank=True는 이미지가 없더라도 글 작성이 가능하도록 해준다.*

```python
class Article(models.Model):
    title = models.CharField(max_length=10)
    content = models.TextField()
    image = models.ImageField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

8) `python manage.py makemigrations`

해보면 Pillow가 없다는 말이 나온다. Pillow는 이미지를 조작하기 위한 라이브러리!

9) `pip install Pillow`

10) `python manage.py makemigrations` 다시 해주고 `python manage.py migrate`

11) include 불러와서 app으로 articles/ 경로 넘겨주는 작업 하기

12) urls.py 만들고 app_name = articles, view 파일 불러오기, index함수 만들기, 템플릿 만들기

```python
from django.urls import path
from . import views

app_name = 'articles'
urlpatterns = [
    path('', views.index, name='index')
]
```

```python
# views.py
from django.shortcuts import render
from .models import Article

def index(request):
    articles = Article.objects.all()
    context = {
        'articles': articles
    }
    return render(request, 'articles/index.html', context)
```

```django
# templates/articles/index.html

{{ articles }}
```



13) create 만들기

```python
from django.urls import path
from . import views

app_name = 'articles'
urlpatterns = [
    path('', views.index, name='index'),
    path('create/', views.create, name='create')
]
```

```python
# views.py

def create(request):
    if request.method = "POST":
        pass
    else:
        # form 템플릿이 필요하네?
        return render()
```



14) forms.py 생성

```python
# forms.py
from django import forms
from .models import Article

class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        field = '__all__'
```



15) 다시 create 함수 만들기

```python
# views.py
from .forms import ArticleForm
def create(request):
    if request.method = "POST":
        pass
    else:
        form = ArticleForm()
    context = {
        'form': form
    }
    return render(request, 'articles/create.html', context)
```

```django
# create.html

<form method="POST"> # 무작정 action을 지우는게 아니다.
    {% csrf token %}
    {{ form.as_p }}  
    <button>제출</button>
</form>
```

이제 pass 부분을 작성해야 한다. 이 상태로 실행을 해보면 형식은 article 형식에서처럼 잘 만들어져 있지만 POST 요청을 보내면 아무것도 하지 않게 된다.



```python
# views.py - create
from .forms import ArticleForm
def create(request):
    if request.method = "POST":
        form = ArticleForm(request.POST, request.FILES)
        
        if form.is_valid():
            article = form.save() # form.save()은 리턴 값이 있음.
        	return redirect('articles:detail', article.pk)
    else:
        form = ArticleForm()
    context = {
        'form': form
    }
    return render(request, 'articles/create.html', context)
```

이제 create 파일은 다 만들었다. detail을 만들 차례!



16) detail 만들기

```python
path('<int:pk>/', views.detail, name='detail')
```

```python
# views.py
from django.shortcuts import render, redirect, get_object_or_404
from .models import Article
from .forms import ArticleForm
def detail(request, pk):
    # Article이 있으면 들고오고 없으면 404
    article = get_object_or_404(Article, pk=pk)
    context = {
        'article': article
    }
    return render(request, 'articles/detail.html', context)
```

```django
# detail.html
{{ article }}
```

이제 detail 페이지를 확인해보려고 create에 들어가서 제목, 내용 입력 후 파일을 첨부하고 전송을 하려고 하니 This field is required 라는 메세지가 뜨며 첨부가 되지 않는다. 우리는 **form 태그를 통해 어떤 파일을 첨부하려고 할 때 enctype을 설정해주어야 하고**, 다음과 같다.

```django
# create.html

<form method="POST" enctype="multipart/form-data">
    {% csrf token %}
    {{ form.as_p }}  
    <button>제출</button>
</form>
```

multipart/form-data를 추가!

이제 **파일을 첨부해서 글을 작성**할 수 있다.



그런데 아직 media에 대한 설정을 하지 않았기 때문에 파일을 첨부해서 글을 작성했더라도 실제로 사용할 수는 없는 상태이다.

예를 들어 image를 보고 싶어서 detail 페이지에 다음과 같이 작성해도 우리는 정보들만 볼 수 있을 뿐 이미지를 볼 수는 없다.

```django
# detail.html
{{ article.image }} # model 설정에 image를 했었음
{{ article.image.url }}
```



우리는 media url을 추가해줘야 이러한 이미지를 볼 수가 있게 된다.

17) media 설정 추가

```python
# Media

MEDIA_URL = '/media/'

# 실제 파일이 저장될 경로
MEDIA_ROOT = BASE_DIR / 'media' # media 폴더에 들어가게 됨
```



18) urls.py 에서 미디어 설정 추가

path('media/~~') 경로로 접근하면 경로에 맞게 파일을 보여주도록 설정을 해 줄 것이다. 실제 파일의 경로와 url 경로는 다르기 때문에 path를 통해 이를 설정해준다. 여기에 static이라는 함수를 사용한다.

```python
from django.contrib import admin
from django.urls import path, include
# media를 위한 설정
from django.conf.urls.static import static
from django.conf import settings
urlpatterns = [
    path('admin/', admin.site.urls),
    path('articles/', include('articles.urls')),
] + static(settings.MEDIA_URL, document_root=settigns.MEDIA_ROOT)
```

`+static ~~`를 추가해주면 urlpatterns에 어떤 media의 url을 붙여주는 것과 같다. 반드시 필요한 설정이다!

이제 이렇게 하고 디테일 경로를 확인해보면 경로가 다음과 같은 식으로 나온다.

`/media/bonobono.jpg`

이제 이걸 사용해서 이미지를 표현할 수 있게 되었다. (저장만 가능하던게 보여주기도 가능하게 됨!)



**사용 예시**

```django
{{ article }}

{% if article.image %}
  <img src="{{ article.image.url }}" alt="{{ article.image }}"
{% else %}
  <p>사진 없음</p>
{% endif %}
```

이렇게 article.image가 있으면 보여주고, 없으면 사진 없음으로 대체하는 방식으로 사용할 수도 있다.



.gitignore

```
venv/
media/
__pychache__/
db.sqlite3
...
```



19) update 만들기

```python
# urls.py
path('<int:pk>/update', views.update, name='update'),
```

```python
# views.py
def update(request, pk):
    if request.method == "POST":
        pass
    else:
        # article의 내용을 가지고 와서 ArticleForm에 넣어준다.
        article = get_object_or_404(Article, pk=pk)
        form = ArticleForm(instance=article)
        # 즉 입력 양식인 form에 pk의 내용을 넣어준다.
    context = {
        'article': article,
        'form': form,
    }
    # 새로 edit이나 update.html을 만들 필요 없이
    # create을 재사용 할 수 있다. (action을 지웠기 때문에)
    # return render(request, 'articles/create.html', context)
	# 재사용 하지 않는 경우
    return render(request, 'articles/update.html', context)
```

```django
# update.html

<form action="{% url 'articles:update' article.pk %}" method="POST" enctype="multipart/form-data">
{% csrf_token %}
{{ form.as_p }}
<button>제출</button>
</form>
```



20) update 함수 최종!

```python
# views.py - update
def update(request, pk):
    article = get_object_or_404(Article, pk=pk)
    if request.method == "POST":
        ArticleForm(request.POST, request.FILES, instance=article)
    	if form.is_valid():
            form.save()
            return redirect('articles:detail', pk)
    else:
        form = ArticleForm(instance=article)
        
    context = {
        'article': article,
        'form': form,
    }
    # 새로 edit이나 update.html을 만들 필요 없이
    # create을 재사용 할 수 있다. (action을 지웠기 때문에)
    # return render(request, 'articles/create.html', context)
	# 재사용 하지 않는 경우
    return render(request, 'articles/update.html', context)
```



21) delete 만들기

```python
# views.py
# decoration 써보기
from django.views.decorators.http import require_POST

@require_POST
def delete(request, pk):
    article = get_object_or_404(Article, pk=pk)
    article.delete()
    return redirect('articles:index')
```

```python
# urls.py
path('<int:pk>/delete', views.delete, name='delete'),
```

url로 접근하면 405error(method가 잘못되었다)가 뜬다. GET 메서드를 잘 막아내고 있는 것!



22) decorators 더 써보기

Update나 create의 경우에는 require http method를 사용해서 GET과 POST만 허용할 수 있다.

```python
# views.py - create
from .forms import ArticleForm
from django.views.decorators.http import require_POST, require_http_methods

@require_http_methods(['GET', 'POST'])
def create(request):
    if request.method = "POST":
        form = ArticleForm(request.POST, request.FILES)
        
        if form.is_valid():
            article = form.save() # form.save()은 리턴 값이 있음.
        	return redirect('articles:detail', article.pk)
    else:
        form = ArticleForm()
    context = {
        'form': form
    }
    return render(request, 'articles/create.html', context)


# views.py - update
@require_http_methods(['GET', 'POST'])
def update(request, pk):
    article = get_object_or_404(Article, pk=pk)
    if request.method == "POST":
        ArticleForm(request.POST, request.FILES, instance=article)
    	if form.is_valid():
            form.save()
            return redirect('articles:detail', pk)
    else:
        form = ArticleForm(instance=article)
        
    context = {
        'article': article,
        'form': form,
    }
    # 새로 edit이나 update.html을 만들 필요 없이
    # create을 재사용 할 수 있다. (action을 지웠기 때문에)
    # return render(request, 'articles/create.html', context)
	# 재사용 하지 않는 경우
    return render(request, 'articles/update.html', context)
```

