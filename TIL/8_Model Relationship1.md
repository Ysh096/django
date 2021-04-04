# 1. Model Relationship

테이블 사이에 관계가 지어질 수 있다.

article table

- id				--> comment table
- name                  - content
- content
- ...

그런데 만약 댓글을 다는 상황이라면, article table에 필드를 하나 추가해서 그 필드에 댓글들을 다 추가하면 되지 않을까?

-> 하나의 필드에는 하나의 값만 저장해야 한다.

article table에 필드를 여러 개 추가해서 댓글을 하나씩 추가하면 되지 않을까?

-> 댓글이 달릴 때마다 필드를 변경해야 하고, 컬럼 이름은 고유하게 하나로 만든다는 약속에 위배된다.

그러면 댓글만 바꿔서 여러 개의 row data를 저장하면 되지 않을까?

-> pk가 중복되고, 레코드 값 중복도 피하는 것이 좋다.

=> 테이블을 쪼개고 관계를 지어주자.



### Model Relationship

모델 간 관계를 나타내는 필드

- Many to one(1:N)
  - ForeignKey()
- Many to Many(M:N)
  - ManyToManyField()
- One to One(1:1)
  - OneToOneField()
  - 기존 테이블의 필드값 수정이 어려울 때



### 게시글과 댓글의 관계

하나의 게시글은 여러 개의 댓글을 가진다. 1 => N

하나의 댓글은 여러 개의 게시글에 속할 수 없다. 1 => N이 아님!!

=> 한 방향으로만 1:N관계가 성립할 때, 1:N 관계라고 한다. 게시글과 댓글은 1:N의 관계!



### Many to one(1:N)

- Foreign Key(외래 키)

  - **자식이 부모 테이블을 참조하기 위해 저장한 부모 테이블의 PK를 말한다.**

  - **한 테이블의 필드 중 다른 테이블의 행을 식별할 수 있는 키**
  - 하나의 테이블이 여러 개의 외래 키를 포함할 수 있음
    - 각각의 외래 키들은 서로 다른 테이블을 참조할 수 있음
  - 참조하는 테이블의 행 여러 개가 참조되는 테이블의 한 행을 참조할 수 있다. (여러 댓글이 하나의 기사를 가리킬 수 있다.)
  - 참조하는 테이블과 참조되는 테이블이 동일할 수 있다. (재귀적 외래 키) -> 대댓글
  
- 외래 키의 특징

  - **키를 사용하여 부모 테이블의 유일한 값을 참조(참조 무결성)**
  - 외래 키의 값이 반드시 부모 테이블의 **기본 키일 필요는 없지만** **유일**해야 함



### django의 many-to-one 관계

```python
	class Comment(models.Model):
    # 참조하는 모델(여기서는 Article), on_delete 가 반드시 들어가야 한다.
    article = models.ForeignKey(Article, on_delete=models.CASCADE)
```

---

##### 데이터 무결성 - 참조 무결성

외래 키 값이 데이터베이스의 특정 테이블의 기본 키 값을 참조하거나, 유일한 값을 참조하는 것

---

Django에서 many-to-one을 표현하기 위한 모델

- ForeignKey()
  - on_delete
    - ForeignKey가 **참조하는 객체(1)가 사라졌을 때** **ForeignKey를 가진 객체(N)**를 어떻게 처리할 지 정의
      - 게시글이 사라지면 댓글을 어떻게 할지!
    - CASCADE, PROTECT, SET_NULL, SET_DEFAULT, SET(), DO_NOTHING, RESTRICT(새로 생김)
      - PROTECT: 참조가 되어있는 경우 오류 발생(삭제를 막음)
      - **CASCADE**: 부모 객체(참조된 객체)가 삭제됐을 때 이를 참조하는 객체도 삭제
- Comment(N)가 Article(1)을 참조할 때
  - `article = comment.article`
- Article(1)이 Comment(N)을 역참조할 때
  - `article.comment_set.all()`



# 2. Foreign Key 실습

```python
# articles/models.py

from django.db import models

class Article(models.Model):
    title = models.CharField(max_length=10)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

# 1:N으로 댓글을 작성하기 위한 모델
class Comment(models.Model):
    # 가장 위에 foreign key를 적어주면 다른 사람이 보기 편하다.
    # 누구랑 연결할지, 연결이 삭제되면 어떻게 할지 적어줘야 한다.
    article = models.ForeignKey(Article, on_delete=models.CASCADE)
    content = models.CharField(max_length=100)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.content
```

편의를 위해 comment에 ForeignKey를 설정할 때 **참조하는 모델의 소문자 형태로 변수를 설정**한다. (외래 키의 id를 만들 때 사용함)

**모델을 수정했으므로**

`python manage.py makemigrations`

`python manage.py migrate` 

**필수!**



**주의**: article 테이블은 댓글 관련된 어떠한 변화도 없다.

articles_coment 테이블의 구성 요소를 보면, id, content, created_at, updated_at, article_id 이다.

여기서 article_id는 comment 클래스의 첫 줄에 작성한 article 이라는 변수 이름에 _id를 붙인 것으로, 하나의 column 이름이다.



# 3. 댓글이 어떻게 작성되는지 알아보기

`python manage.py shell_plus`

`comment = Comment()`

`comment.content = '댓글1'`

`comment.save()`

그냥 이렇게 저장을 하려고 하면 NOT NULL constaint failed가 뜨며 실패한다. 왜냐하면, 몇 번 게시글에 댓글을 작성하려고 하는지를 모르기 때문이다.

article_id 값을 알려줘야 함!

일단은 게시글을 만들어야 한다.

`Article.objects.create(title='제목1', content='내용1') `

현재 comment.content에는 '댓글1'이 저장된 상태

**게시글을 가져와서 comment의 article에 그대로 넣어주기만 하면 알아서 pk값이 article_id에 저장된다.**

`article = Article.objects.get(pk=1)`

` comment.article = article`

`comment.save()`



이제 comment.pk를 타이핑 해 보면 1 이라는 comment의 번호가 출력된다.

comment.article을 쳐보면 comment가 가리키는 article 객체를 얻을 수 있고, comment.article_id 혹은 comment.article.pk 는 comment가 가리키는 article의 번호(id)를 뜻한다.



두 번째 댓글 만들기

`comment = Comment(content='댓글2', article=article) `

`comment.save()`

---

**admin 사이트에서 댓글 보여주기**

```python
# articles/admin.py
from .models import Article, Comment

admin.site.register(Article)
admin.site.register(Comment)
```

admin 페이지에서 댓글의 위치를 수정할 수도 있다!

---



# 4.0 article에서 comment 참조(역참조)

Article은 Comment를 기본적으로 참조할 수가 없는데(역참조 필요), django는 comment_set이라고 하여 역참조를 도와주는 모델 매니저가 있다.

`모델이름_set` 형식!

```shell
# shell_plus
In[1]: article = Article.objects.get(pk=1)
In[2]: article.comment_set.all()
Out[2]: <QuerySet [<Comment: Comment object (1)>, <Comment: Comment object (2)>]>
```



# 4.1 댓글 작성(comment form)

```python
# articles/forms.py
from .models import Article, Comment


class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = '__all__'
        # exclude = ('title',)

class CommentForm(forms.ModelForm):
    class Meta:
        model = Comment
        fields = '__all__'
```

댓글 입력에 사용할 form이 만들어짐!



게시글의 아래에 댓글을 작성하는 형태를 구현해보자.

detail 페이지에 표현하기 위해 views.py부터 들어가보자.

댓글은 따로 페이지가 있는게 아니라 detail페이지 내부에서 따로 댓글을 받을 것이다.

우선 디테일 페이지에 댓글 입력 폼을 전달해주자.

```python
from .forms import ArticleForm, CommentForm

@require_safe
def detail(request, pk):
    article = get_object_or_404(Article, pk=pk)
    comment_form = CommentForm()
    context = {
        'article': article,
        'comment_form': comment_form,
    }
    return render(request, 'articles/detail.html', context)
```

전달된 폼(`comment_form`)을 하단에 출력해주자.

```django
{% extends 'base.html' %}

{% block content %}
  <h2>DETAIL</h2>
  <h3>{{ article.pk }} 번째 글</h3>
  <hr>
  <p>제목 : {{ article.title }}</p>
  <p>내용 : {{ article.content }}</p>
  <p>작성시각 : {{ article.created_at }}</p>
  <p>수정시각 : {{ article.updated_at }}</p>
  <hr>
  <a href="{% url 'articles:update' article.pk %}" class="btn btn-primary">[UPDATE]</a>
  <form action="{% url 'articles:delete' article.pk %}" method="POST">
    {% csrf_token %}
    <button class="btn btn-danger">DELETE</button>
  </form>
  <a href="{% url 'articles:index' %}">[back]</a>
  <hr>
  <form action="#" mehtod="POST">
    {% csrf_token %}
    {{ comment_form }}
    <input type="submit"> 
  </form>

{% endblock %}

```

이렇게 작성해주면 comment 작성란에 **Article을 선택할 수 있는 바**가 하나 있다. (Admin 사이트에서 보이는 것과 동일)

이는 우리가 댓글을 게시글에 들어가서 쓰기 때문에 매우 불필요한 옵션이다. **제거**해주자.



```python
# articles/forms.py
class CommentForm(forms.ModelForm):

    class Meta:
        model = Comment
        fields = '__all__'
        exclude = ('article',)
```

이제 형식은 갖췄다. 댓글이 작성되도록 해보자!

1) url 설정

```python
# articles/urls.py
path('<int:pk>/comments/', views.comments_create, name='comments_create'),
```

2) html에서 url 설정

**주의**: 같은 페이지(detail)의 url로 요청을 보내는 것이 아니라서 url 설정을 해 주는 것이다.

```django
# articles/detail.html
...
    <button class="btn btn-danger">DELETE</button>
  </form>
  <a href="{% url 'articles:index' %}">[back]</a>
  <hr>
  <form action="{% url 'articles:comments_create' article.pk %}" mehtod="POST">
    {% csrf_token %}
    {{ comment_form }}
    <input type="submit"> 
  </form>

{% endblock %}
```

3) views.py에서 함수 설정

*comment의 경우 댓글 작성을 위해 들어가야 할 페이지가 detail 페이지이므로 따로 GET 요청이 필요없다. POST만 처리하면 됨!

*그러니까 detail 페이지에 들어가서 comment 작성을 누르기만 하면 되므로 comment 페이지에 따로 들어갈 필요가 없다는 것! (당연한 소리긴 하다)

```python
# articles/views.py

@require_POST
def comments_create(request, pk):
    article = get_object_or_404(Article, pk=pk)
    comment_form = CommentForm(request.POST)
    if comment_form.is_valid():
        comment_form.save()
        return redirect('articles:detail', article.pk)
    context = {
        'comment_form': comment_form,
        'article': article,
    }
    return render(request, 'articles/detail.html', context)
```

여기까지만 하면 앞서 article 선택 옵션을 제거했기 때문에 comment가 달려야 할 article_id를 몰라서 **오류**가 난다.



```python
  ...
    if comment_form.is_valid():
        # instance는 만들지만 DB에 아직 저장은 안 한 상태
  		comment = comment_form.save(commit=False)
        # article_id 정보 저장
        comment.article = article
        # 작성
        comment.save()
        return redirect('articles:detail', article.pk)
  ...
```

DB에 저장은 하지 않고 form을 저장만 한 상태에서, article 정보를 넣어준다. 이제 댓글을 작성하면 DB에 저장이 된다.



# 4.2 댓글 출력

detail 페이지에서 역시 보여준다.

1) 역참조

```python
# articles/views.py

@require_safe
def detail(request, pk):
    article = get_object_or_404(Article, pk=pk)
    comment_form = CommentForm()
    comments = article.comment_set.all()
    context = {
        'article': article,
        'comment_form': comment_form,
        'comments': comments,
    }
    return render(request, 'articles/detail.html', context)
```

2) detail 수정

```html
{% extends 'base.html' %}

{% block content %}
  <h2>DETAIL</h2>
  <h3>{{ article.pk }} 번째 글</h3>
  <hr>
  <p>제목 : {{ article.title }}</p>
  <p>내용 : {{ article.content }}</p>
  <p>작성시각 : {{ article.created_at }}</p>
  <p>수정시각 : {{ article.updated_at }}</p>
  <hr>
  <a href="{% url 'articles:update' article.pk %}" class="btn btn-primary">[UPDATE]</a>
  <form action="{% url 'articles:delete' article.pk %}" method="POST">
    {% csrf_token %}
    <button class="btn btn-danger">DELETE</button>
  </form>
  <a href="{% url 'articles:index' %}">[back]</a>
  <hr>
  <h4>댓글 목록</h4>
  <ul>
  {% for comment in comments %}
   <li>{{ comment }}</li>
  {% endfor %}
  </ul>
  <hr>
  <form action="{% url 'articles:comments_create' article.pk %}" method="POST">
    {% csrf_token %}
    {{ comment_form }}
    <input type="submit"> 
  </form>

{% endblock %}
```





# 4.3 댓글 삭제

1) urls.py 수정

```python
# articles/urls.py
# 함수에서 article_pk와 comment_pk로 구분할거면 여기에서도 해줘야 오류가 나지 않는다.
path('<int:article_pk>/comments/<int:comment_pk>/delete/', views.comments_delete, name='comments_delete'),
```

2) views.py

```python
# articles/views.py
from .models import Article, Comment
@require_POST
def comments_delete(request, article_pk, comment_pk):
    comment = get_object_or_404(Comment, pk=comment_pk)
    comment.delete()
    return redirect('articles:detail', article_pk)
```

3) detail.html

```html
{% extends 'base.html' %}

{% block content %}
  <h2>DETAIL</h2>
  <h3>{{ article.pk }} 번째 글</h3>
  <hr>
  <p>제목 : {{ article.title }}</p>
  <p>내용 : {{ article.content }}</p>
  <p>작성시각 : {{ article.created_at }}</p>
  <p>수정시각 : {{ article.updated_at }}</p>
  <hr>
  <a href="{% url 'articles:update' article.pk %}" class="btn btn-primary">[UPDATE]</a>
  <form action="{% url 'articles:delete' article.pk %}" method="POST">
    {% csrf_token %}
    <button class="btn btn-danger">DELETE</button>
  </form>
  <a href="{% url 'articles:index' %}">[back]</a>
  <hr>
  <h4>댓글 목록</h4>
  <ul>
  {% for comment in comments %}
   <li>{{ comment }}
   <form action="{% url 'articles:comments_delete' article.pk comment.pk %}" method="POST">
    {% csrf_token %}
    <input type="submit" value="DELETE">
   </form>
   </li>
  {% endfor %}
  </ul>
  <hr>
  <form action="{% url 'articles:comments_create' article.pk %}" method="POST">
    {% csrf_token %}
    {{ comment_form }}
    <input type="submit"> 
  </form>

{% endblock %}
```



# 4.4 댓글 수정

댓글 수정은 보통 detail 페이지에서 그대로 변경하는데, 우리가 지금까지 배운걸로는 아직 이렇게는 못하고 수정 페이지를 새로 만들어서 수정을 해야한다. 이는 어색하므로 지금은 배우지 않고, 더 배운 다음 댓글 수정 방법에 대해 알아보자. (js)





# 5. 코드 개선

1) 로그인 된 사용자만 댓글 작성 가능

```python
@require_POST
def comments_create(request, pk):
    # 로그인한 유저에 대해서
    if request.user.is_authenticated:
        article = get_object_or_404(Article, pk=pk)
        comment_form = CommentForm(request.POST)
        if comment_form.is_valid():
            comment = comment_form.save(commit=False)
            comment.article = article
            comment.save()
            return redirect('articles:detail', article.pk)
        context = {
            'comment_form': comment_form,
            'article': article,
        }
        return render(request, 'articles/detail.html', context)
    return redirect('accouts:login')
```



2) 로그인 된 사용자만 댓글 삭제 가능

```python
@require_POST
def comments_delete(request, article_pk, comment_pk):
    if request.user.is_authenticated:
        comment = get_object_or_404(Comment, pk=comment_pk)
        comment.delete()
        return redirect('articles:detail', article_pk)
    return redirect('accounts:login')
```

**주의!**

@login_required를 사용하려면, GET request를 받았을 때도 함수가 동작해야 한다. login_required는 name 이후의 어떤 요청값을 보내서 로그인이 되고 난 후 GET 요청으로 함수를 다시 실행하려고 하기 때문에..

=> POST만 허용한 상태에서 login_required를 쓰면 405 error가 난다.



3) 댓글의 개수 나타내어 보기

```html
# articles/detail.html

{% extends 'base.html' %}

{% block content %}

...

  <h4>댓글 목록</h4>
  {{ comments|length }}
  {{ article.comment_set.all|length }}
  {{ comments.count }}
  <ul>

...

{% endblock %}
```

위의 세 가지 방법으로 댓글의 개수 표현 가능!



4) 댓글이 없는 경우

```html
# articles/detail.html

{% extends 'base.html' %}

{% block content %}

...

  <ul>
    {% for comment in comments %}
     <li>{{ comment }}
      <form action="{% url 'articles:comments_delete' article.pk comment.pk %}" method="POST">
        {% csrf_token %}
        <input type="submit" value="DELETE">
      </form>
     </li>
    {% empty %}
      <p>댓글이 없습니다.</p>
    {% endfor %}
    </ul>
    <hr>

...

{% endblock %}
```



# 6. Customizing authentication in Django

### User model을 custom user model로 대체하기

- django는 custom model을 참조하는 AUTH_USER_MODEL 설정을 제공하여 기본 user model을 재정의(override)할 수 있도록 함.
- 새 프로젝트를 시작하는 경우 기본 사용자 모델로 충분하더라도 **커스텀 유저 모델을 설정하는 것을 강력하게 권장**
- 커스텀 유저 모델은 기본 사용자 모델과 동일하게 작동하면서도 **필요한 경우 나중에 맞춤 설정**할 수 있기 때문!

- 단, 프로젝트의 모든 migrations 혹은 첫 migrate를 실행하기 전에 이 작업을 마쳐야 함

=> 프로젝트 시작 직후에 custom user model을 설정하고 가라!

[Customizing authentication in Django][https://docs.djangoproject.com/en/3.1/topics/auth/customizing/]



#### 1) settings.py

아래 코드를 맨 밑에 추가

```python
# AUTH_USER_MODEL = 'myapp.MyUser' 형식, accounts 앱의 User 클래스를 AUTH_USER_MODEL로 설정!
AUTH_USER_MODEL = 'accounts.User'
```

#### 2) accounts/models.py

```python
from django.db import models
from django.contrib.auth.models import AbstractUser

# Create your models here.

class User(AbstractUser):
    pass
```

AbstractUser를 상속받는 이유: AbstracBaseUser에 구현된 필드가 너무 적다.

**AbstractBaseUser**

- 기본적으로 password와 last_login만 제공
- 자유도가 높지만 필요한 필드는 모두 직접 작성해야 함

**AbstractUser**

- 관리자 권한과 함께 완전한 기능을 갖춘 사용자 모델을 구현하는 기본 클래스



**Abstract base classes**

- 몇 가지 공통 정보를 여러 다른 모델에 넣을 때 사용하는 클래스
- 데이터베이스 테이블을 만드는 데 사용되지 않으며, 대신 **다른 모델의 기본 클래스로 사용되는 경우(상속)** 해당 필드가 하위 클래스의 필드에 추가 됨

==> 클래스 내부에  `abstract = True` 가 붙은 경우 table로 직접 만들어지지는 않고, Abstract를 상속받은 하위 클래스의 필드에 Abstract의 필드를 추가해주는 정도로 사용된다. 그래서 이름도 Abstract를 붙여서 만듦!



#### 3) 처음에 User Custom을 안해서 나중에라도 하려는 경우 (초기화 방법)

1) `__init__.py`를 제외한 모든 migrations 내부 .py 파일 삭제(migrations 폴더는 삭제하면 안된다.)

2) db.sqlite3 삭제

초기화는 끝!

3) `python manage.py makemigrations`

4) `python manage.py migrate`



#### 4) Error 해결 

이제 회원가입을 하려고 해보면, auth.User가 accouts.User로 바뀌었다고 하며 에러가 뜬다. (UserCreationForm에서 에러!)

django github에서 UserCreationForm 함수를 보면,

```python
class Meta:
    model = User
    fields = ("username",)
    ...
```

기존에 존재하는 User를 모델로 참조하기 때문에 우리가 대체한 모델이 적용이 되지 않는다.

이렇게 기존의 User를 참조하는 auth form이 두 개 있다. 얘네를 수정해줘야 한다!



https://docs.djangoproject.com/en/3.1/topics/auth/customizing/

위 링크로 들어가 Custom users and the built-in auth forms 항목을 보면

- UserCreationForm
- UserChangeForm

위 둘을 수정해줘야 한다고 하고 있다.

```python
# accounts/forms.py

# 얘는 기존에 배웠던 custom인데, 이제야 확실히 알겠다!
class CustomUserChangeForm(UserChangeForm):
    class Meta:
        # 현재 활성화 되어 있는 user 모델을 참조하는 방법!
        model = get_user_model()
        fields = ('email', 'first_name', 'last_name',)
        
class CustomUserCreationForm(UserCreationForm):
    class Meta(UserCreationForm.Meta):
        model = get_user_model()
        # 상속받은 form에서 field를 모두 가져오고, 거기에 우리가 원하는 field를 추가하는 형식
        fields = UserCreationForm.Meta.fields + ('custom_field',)
```

- 괜히 User()를 이용하여 Custom을 하지 말라고 한 것이 아니었다. get_user_model()을 통해 현재 활성화되어 있는 유저 모델을 참조하면 우리가 커스텀 한 모델을 참조하게 된다!

- custom_field는 user에 있는 field를 써야 한다! ex) email
  - email을 추가하면 가입시 email 까지 물어보게 된다.



custom을 마쳤으므로 이제 accounts/views.py에서 UserChangeForm, UserCreationForm대신 CustomUserChangeForm, CustomUserCreationForm을 사용한다. 물론 import 해줘야 한다.



최종 완성된 accounts/forms.py와 accounts/views.py를 예시로 확인해보자.

```python
# accounts/forms.py

from django.contrib.auth.forms import UserChangeForm, UserCreationForm
from django.contrib.auth import get_user_model


class CustomUserChangeForm(UserChangeForm):

    class Meta:
        model = get_user_model()
        fields = ('email', 'first_name', 'last_name',)

class CustomUserCreationForm(UserCreationForm):
    class Meta(UserCreationForm.Meta):
        model = get_user_model()
        # 상속받은 form에서 field를 모두 가져오고, 거기에 우리가 원하는 field를 추가하는 형식
        fields = UserCreationForm.Meta.fields + ('email',) 
```



```python
# accounts/views.py

from django.shortcuts import render, redirect
from django.contrib.auth import login as auth_login
from django.contrib.auth import logout as auth_logout
# from django.contrib.auth import login as auth_login, logout as auth_logout
from django.views.decorators.http import require_POST, require_http_methods
from django.contrib.auth.decorators import login_required
from django.contrib.auth import update_session_auth_hash
from django.contrib.auth.forms import (
    AuthenticationForm, 
    UserCreationForm, 
    PasswordChangeForm,
)
from .forms import CustomUserChangeForm, CustomUserCreationForm


# Create your views here.
@require_http_methods(['GET', 'POST'])
def login(request):
    if request.user.is_authenticated:
        return redirect('articles:index')

    if request.method == 'POST':
        form = AuthenticationForm(request, request.POST)
        # form = AuthenticationForm(request, data=request.POST)
        if form.is_valid():
            auth_login(request, form.get_user())
            return redirect(request.GET.get('next') or 'articles:index')
    else:
        form = AuthenticationForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/login.html', context)


@require_POST
def logout(request):
    if request.user.is_authenticated:
        auth_logout(request)
    return redirect('articles:index')


@require_http_methods(['GET', 'POST'])
def signup(request):
    if request.user.is_authenticated:
        return redirect('articles:index')

    if request.method == 'POST':
        form = CustomUserCreationForm(request.POST)
        if form.is_valid():
            user = form.save()
            auth_login(request, user)
            return redirect('articles:index')
    else:
        form = CustomUserCreationForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/signup.html', context)


@require_POST
def delete(request):
    if request.user.is_authenticated:
        request.user.delete()
        auth_logout(request)
    return redirect('articles:index')


@login_required
@require_http_methods(['GET', 'POST'])
def update(request):
    if request.method == 'POST':
        form = CustomUserChangeForm(request.POST, instance=request.user)
        if form.is_valid():
            form.save()
            return redirect('articles:index')
    else:
        form = CustomUserChangeForm(instance=request.user)
    context = {
        'form': form,
    }
    return render(request, 'accounts/update.html', context)


@login_required
@require_http_methods(['GET', 'POST'])
def change_password(request):
    if request.method == 'POST':
        form = PasswordChangeForm(request.user, request.POST)
        # form = PasswordChangeForm(user=request.user, data=request.POST)
        if form.is_valid():
            form.save()
            update_session_auth_hash(request, form.user)
            return redirect('articles:index')
    else:
        form = PasswordChangeForm(request.user)
    context = {
        'form': form,
    }
    return render(request, 'accounts/change_password.html', context)
```



User: Article  <=> 1:N

User: Comment <=> 1:N

Article:Comment <=> 1:N

#### 5) Foreign key 작업

#### 5.1 하나의 유저(1)는 여러 개의 게시글(N)을 가질 수 있다.

N쪽에 Foreign key가 필요하다!

```python
# articles/models.py
from django.db import models
from django.conf import settings
# Create your models here.
class Article(models.Model):
    # 유저를 참조하는 또 다른 방식 ㅠㅠ models.py에서만 AUTH_USER_MODEL을 사용한다.
    # 그 외에는 get_user_model()
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    title = models.CharField(max_length=10)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

**settings.AUTH_USER_MODEL**

- 유저 모델에 대한 외래 키 또는 M:N 관계를 정의할 때 사용
- models.py에서 유저 모델을 참조할 때 사용
- return 값이 문자열 'accounts.User'
- django server가 켜지는 순간 installed app의 앱 순서대로 앱을 실행시키는데, articles, accounts 순으로 저장을 했다면 articles에 대한 모든 것들이 수행이 끝나면 accounts에 대한 것들을 수행한다. 그러면 articles에 위와 같이 설정한 models.py가 accounts 앱이 구동되기 전에 먼저 구동이 된다는 것! 그런데 만약 get_user_model을 사용해서 ForeignKey를 구성하면 현재 활성화되어 있는 User를 불러오려고 할 것이고, 활성화 된 User가 없으므로 에러가 난다. 그래서 우리는 settings의 AUTH_USER_MODEL에 적힌 경로(`AUTH_USER_MODEL = 'accounts.User'`)를 찾아가 User 모델을 참조하도록 위와 같이 작성한 것이다.
  - 만약 settings.py에서 accounts 앱을 articles 앱보다 위에 위치시킨다면 get_user_model을 사용해도 되겠지만.. 굳이 그렇게 할 필요는 없다!

참고: [링크 들어가서 Referencing the User model 검색!](https://docs.djangoproject.com/en/3.1/topics/auth/customizing/)



**get_user_model()**

- django는 User 모델을 직접 참조하는 대신 **get_user_model()을 사용하여 사용자 모델을 참조하라고 권장**
- models.py가 아닌 **다른 모든 곳**에서 유저 모델을 참조할 때 사용
- return 값이 User 객체



여기까지 해주고, python manage.py makemigrations를 해주면 아래의 메세지가 뜬다. (나중에 ForeignKey를  추가한 경우)

```
$ python manage.py makemigrations
You are trying to add a non-nullable field 'user' to article without a default; we can't do that (the database needs something to 
populate existing rows).
Please select a fix:
 1) Provide a one-off default now (will be set on all existing rows with a null value for this column)
 2) Quit, and let me add a default in models.py
Select an option:
```

그냥 1번 두 번 쳐주고 나면 새로운 설계도가 생긴다!



python manage.py migrate 후 새 글 작성을 보면..

누가 작성하는 것인지까지 선택하는 불필요한 field가 있다!

이를 제거하기 위해 forms.py에 들어가 다음과 같이 수정!

```python
from django import forms
from .models import Article, Comment


class ArticleForm(forms.ModelForm):

    class Meta:
        model = Article
        # fields = '__all__' 에서 수정!
        fields = ('title', 'content',)
        # exclude = ('title',)
```

누가 작성하는 것인지는 앞서와 같이 commit=False를 이용하면 될 듯!