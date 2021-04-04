# Model Relationship2(1:N 총정리)

# 1:N

### 지난 시간

- ForeignKey()
- Comment(N) - Article(1)
  - 외래 키는 N에 작성이 되었다.



### Customizing Authentication

- 일부 프로젝트에서는 빌트인 유저 모델과 다른 인증 요구사항이 필요할 수 있음.
- django는 custom model을 참조하는 AUTH_USER_MODEL 설정을 제공하여 기본 user model을 재정의 할 수 있도록 함.
- 새 프로젝트를 시작하는 경우, 기본 사용자 모델이 충분하더라도 커스텀 유저 모델을 설정하자!
- 프로젝트의 모든 migrations 혹은 첫 migrate 를 실행하기 전에 custom 해야한다.



##### AUTH_USER_MODEL

- User를 나타내는데 사용하는 모델
- 기본 값은 'auth.User'

##### AbstractBaseUser & AbstractUser

- AbstractBaseUser
  - 기본적으로 비밀번호와 last_login만 제공
  - 자유도가 높지만 필요한 다른 필드는 모두 직접 작성해야 한다.
- AbstractUser
  - 관리자 권한과 함께 완전한 기능을 갖춘 사용자 모델을 구현하는 기본 클래스

##### Abstract Base Classes

- abstract = True 라는 설정이 되어 있는 모델.
- 몇 가지 공통 정보를 다른 클래스에 넣을 때 사용하는 모델
- 데이터베이스를 구성할 때 직접 사용되지는 않으며, 상속할 수 있는 형태..

##### Custom users & Built-in auth forms

- User와 연결되어 있어서 CustomUserModel을 사용하려면 다시 작성해야 하는 forms
  - UserCreationForm
  - UserChangeForm



django custom authentication 검색

1) AUTH_USER_MODEL 작성

```python
# settings.py
AUTH_USER_MODEL = 'accounts.User'
```



2) accounts/models.py 수정

```python
# accounts/models.py
from django.db import models
from django.contrib.auth.models import AbstractUser
# Create your models here.

class User(AbstractUser):
    pass
```



3) accounts/admin.py 수정

```python
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
# Register your models here.
from .models import User

admin.site.register(User, UserAdmin)
```



4) 설계도 다 지우고(만들어져 있다면) makemigrations, migrate하기



5) CustomUserCreationForm, CustomUserChangeForm (get_user_model을 사용하는게 핵심)

```python
from django.contrib.auth.forms import UserChangeForm, UserCreationForm
from django.contrib.auth import get_user_model


class CustomUserChangeForm(UserChangeForm):

    class Meta:
        model = get_user_model()
        fields = ('email', 'first_name', 'last_name',)

class CustomUserCreationForm(UserCreationForm):

    class Meta(UserCreationForm.Meta):
        model = get_user_model()
        fields = UserCreationForm.Meta.fields
```



### 1:N with User

##### 유저 모델 참조하는 방법

- settings.AUTH_USER_MODEL
  - 유저 모델에 대한 외래 키 또는 M:N 관계를 정의할 때 사용
  - models.py에서 유저 모델을 참조할 때 사용
- get_user_model()
  - models.py가 아닌 경우 전부



##### User(1)-Article(N)

- 외래 키 작성시 인스턴스 이름을 관계지으려는 대상을 소문자로 쓰는 것이 좋다.
- 예를 들어  User와 연결짓는 경우, user라고 써야 테이블에 user_id 로 만들어진다.

```python
# articles/models.py
from django.db import models
from django.conf import settings
# Create your models here.
class Article(models.Model):
    # 참조할 모델, on_delete
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    title = models.CharField(max_length=10)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.title
```

models.py 에 변경이 생겼으므로 makemigrations를 해줘야 하는데, 기본값이 없어서 바로 되지 않는다. 우리가 직접 어떤 값을 넣을지 지정해준다. 간단하게 1을 입력하자.



이제 runserver를 해서 게시글 작성을 확인해 보면, 사용자까지 선택하여 게시글을 작성할 수 있는 창이 뜬다. 이는 불필요한 기능이므로 제거하기 위해 forms.py를 수정해준다.

```python
# articles/forms.py
class ArticleForm(forms.ModelForm):

    class Meta:
        model = Article
        fields = ('title', 'content')
```

그런데 이렇게 하면 user_id 가 없어서 게시글을 작성할 수가 없다고 한다. 누가 작성한 게시글인지 모르니까 작성이 안 되는 것!

이를 수정하기 위해 user에 대한 정보가 들어있는 request.user를 이용할 것이다.



views.py에서 이를 적용해보자.

```python
# articles/views.py
@login_required
@require_http_methods(['GET', 'POST'])
def create(request):
    if request.method == 'POST':
        form = ArticleForm(request.POST)
        if form.is_valid():
            article = form.save(commit=False)
            article.user = request.user
            article.save()
            return redirect('articles:detail', article.pk)
    else:
        form = ArticleForm()
    context = {
        'form': form,
    }
    return render(request, 'articles/create.html', context)
```

form.save()를 바로 진행하지 않고, form을 인스턴스로 받아놓은 다음에, user값을 입력해주고 다시 저장하는 형식이다.

게시글의 작성자를 출력해보려면, index.html에 다음과 같이 작성해주면 된다.

```html
<p><b>작성자 : {{ article.user }}</b></p>
```



##### 내 글만 수정할 수 있도록 만들기

데이터가 POST인 것을 확인하기 전에, 게시글의 유저가 요청하는 유저와 같은 유저인지 판단해야 한다.

```python
# articles/views.py
@login_required
@require_http_methods(['GET', 'POST'])
def update(request, pk):
    article = get_object_or_404(Article, pk=pk)
    if request.user = article.user: # 게시글 작성자가 현재 요청하는 유저와 동일한지?
        if request.method == 'POST':
            form = ArticleForm(request.POST, instance=article)
            if form.is_valid():
                form.save()
                return redirect('articles:detail', article.pk)
        else:
            form = ArticleForm(instance=article)
    else:
        return redirect('articles:index')
    context = {
        'form': form,
        'article': article,
    }
    return render(request, 'articles/update.html', context)
```



delete도 비슷하게 수정한다.

```python
# articles/views.py
@require_POST
def delete(request, pk):
    article = get_object_or_404(Article, pk=pk)
    if request.user.is_authenticated:
        if request.user == article.user:
            article.delete()
            return redirect('articles:index')
    return redirect('articles:detail', article.pk)
```



아예 로그인 한 사용자만 버튼을 볼 수 있도록 하려면, html 파일에서 다음과 같이 작성할 수도 있다.

```html
<!-- articles/detail.html -->

  {% if request.user == article.user %}
    <a href="{% url 'articles:update' article.pk %}" class="btn btn-primary">[UPDATE]</a>
    <form action="{% url 'articles:delete' article.pk %}" method="POST">
      {% csrf_token %}
      <button class="btn btn-danger">DELETE</button>
    </form>
  {% endif %}
```



##### User(1)-Comment(N)

이제 comment를 user와 관계지어보자.

우선 models.py를 수정해준다. (ForeignKey)

```python
# articles/models.py

class Comment(models.Model):
    article = models.ForeignKey(Article, on_delete=models.CASCADE)
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    content = models.CharField(max_length=200)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.content
```

models.py가 수정되었으므로 makemigrations, migrate를 해준다. 역시 아무 값이 없는데 새로 추가하려고 하므로 기본값을 설정해주어야 한다. 그냥 1을 넣어주자.

이제 

```python
# articles/comments_create

@require_POST
def comments_create(request, pk):
    if request.user.is_authenticated:
        article = get_object_or_404(Article, pk=pk)
        comment_form = CommentForm(request.POST)
        if comment_form.is_valid():
            comment = comment_form.save(commit=False)
            comment.article = article
            comment.user = request.user
            comment.save()
            return redirect('articles:detail', article.pk)
        context = {
            'comment_form': comment_form,
            'article': article,
        }
        return render(request, 'articles/detail.html', context)
    return redirect('accounts:login')
```

comment.user가 request.user와 같은지 확인!



##### 로그인되어 있을 때만 댓글 작성이 가능하도록 만들기

```html
<!-- articles/detail.html -->

  {% if request.user.is_authenticated %}
    <form action="{% url 'articles:comments_create' article.pk %}" method="POST">
      {% csrf_token %}
      {{ comment_form }}
      <input type="submit">
    </form>
  {% else %}
    <a href="{% url 'accounts:login' %}">[댓글을 작성하려면 로그인 하세요.]</a>
  {% endif %}
```



##### 댓글 작성자 출력하기

```html
<!-- articles/detail.html -->
...
<ul>
    {% for comment in comments %}
      <li>
        {% comment %} 댓글 작성자 출력 {% endcomment %}
        {{ comment.user }} - {{ comment }}
        <form action="{% url 'articles:comments_delete' article.pk comment.pk %}" method="POST" class="d-inline">
...
```



##### 댓글 삭제도 본인만 가능하도록 만들기

```python
# articles/views.py

@require_POST
def comments_delete(request, article_pk, comment_pk):
    if request.user.is_authenticated:
        comment = get_object_or_404(Comment, pk=comment_pk)
        # 유저 확인
        if request.user == comment.user:
            comment.delete()
            # return HttpResponseForbidden()
    return redirect('articles:detail', article_pk)
    # return HttpResponse(status=401)
```

```html
<!-- articles/detail.html -->

    {% if request.user == comment.user %}
      <form action="{% url 'articles:delete' article.pk %}" method="POST">
        {% csrf_token %}
        <button class="btn btn-danger">DELETE</button>
      </form>
    {% endif %}
```



### humanize

django 내장앱인 humanize를 이용하여 시간을 표현하기

```python
# settings.py
INSTALLED_APPS = [
    ...
    django.contrib.humanize,
]
```

```html
<!-- articles/detail.html -->
  <p>작성시각 : {{ article.created_at|naturalday }}</p>
  <p>수정시각 : {{ article.updated_at|naturaltime }}</p>
```

이제 작성시간은 `오늘`, 수정시각은 `5 분 전 ` 처럼 표현이 된다.
