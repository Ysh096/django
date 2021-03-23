# django Authentication System

지난 시간

- Form
- ModelForm
- View decorators
- Managing staticfiles
  - static
  - media

---

**Authentication**

- 인증
- 자신이 누구라고 주장하는 사람의 신원을 확인하는 것
- 로그인을 요구하는 과정



**Authorization**

- 권한, 허가
- 가고 싶은 곳으로 가도록 혹은 원하는 정보를 얻도록 허용하는 과정
- 로그인 이후 권한이 있는지 없는지 따지는 과정



**Django Authentication System**

- 인증과 권한 부여를 함께 제공, 그래서 authentication system이라고 부른다.



두 개의 키워드! **User object** , **Web request**



**Authentication Built-in Forms**

- 장고는 회원가입(UserCreationForm), 로그인(AuthenticationForm) 등에 대해 built-in form을 제공한다.
- 커스터마이징은 얘네를 상속받아서 부분적으로 할 것이다.
- 장고가 회원관리에 관한 내용을 만들어놓은 이유는...
  - 모든 웹사이트, 서비스에서 공통적으로 회원관리 기능이 필요하기 때문에
  - 회원관리 로직을 내부적으로 관리하는 것이 복잡하기 때문에
  - (Article 등 지금까지 만든 것들은 웹서버마다 구성이 다르지만 회원 관리는 공통적이다.)

---

### crud가 구현되어있는 상태에서 시작한다.

### 1. 가상환경 만들기

`python -m venv venv`

`ptyhon venv/Scripts/activate`



### 2. 라이브러리 다운로드

`$ pip install -r requirements.txt`



### 3. 유저에 관련된 앱 만들고 등록

`$ python manage.py startapp accounts`

다른 이름 말고 accounts로 만들자.

settings.py에서 등록!

```python
# Application definition

INSTALLED_APPS = [
    'articles',
    'accounts', 
    'bootstrap5',
    'django.contrib.admin',
    ...
```



### 4. urls.py 만들기

```python
# project폴더/urls.py

from django.contrib import admin
from django.urls import path, include


urlpatterns = [
    path('admin/', admin.site.urls),
    path('articles/', include('articles.urls')),
    path('accounts/', include('accounts.urls')),
]
```

```python
# accounts/urls.py
from django.urls import path

app_name = 'accounts'
urlpatterns = [
    
]
```



### 5. migrate 후 서버 켜보기



### 6. Login & Logout

---

- Django는 세션과 미들웨어를 사용하여 **인증 시스템을 request 객체에 연결**해 놓았다.

- 이를 통해 사용자를 나타내는 모든 요청에 request.user를 제공

- 현재 사용자가 로그인하지 않은 경우 AnonymousUser 클래스의 인스턴스로 설정되며, 그렇지 않으면 User클래스의 인스턴스로 설정됨

##### 로그인

- 로그인은 **Session을 Create하는 로직**과 같음(CRUD라는 흐름을 벗어나지 않음)
- login()
  - 현재 세션에 연결하려는 인증된 사용자가 있는 경우 login() 함수로 로그인 진행
  - request 객체와 User 객체를 통해 로그인 진행
  - Django의 session framework를 통해 사용자의 ID를 세션에 저장

##### 로그아웃

- 로그아웃은 세션을 Delete하는 로직과 같음
- logout()
  - request 객체를 받으며 return이 없음. (요청만 받음)
  - 현재 요청에 대한 DB의 세션 데이터를 삭제하고 클라이언트 쿠키에서도 sessionid를 삭제
  - 쿠키 - 브라우저 정리할 때 등장하는 것!

##### HTTP(HyperText Transfer Protocol)

- HTML 문서와 같은 리소스들을 가져올 수 있도록 해주는 프로토콜(규칙, 약속)
- 웹에서 이루어지는 모든 데이터 교환의 기초
- 클라이언트-서버 프로토콜이라고도 한다.
- 요청(requests)
  - 클라이언트에 의해 전송되는 메시지
- 응답(responses)
  - 서버에서 응답으로 전송되는 메시지

- HTTP 특징
  - 비연결지향(connectionless)
    - 서버는 응답 후 접속을 끊음(네이버에 접속하면 받은 문서를 보고 있을 뿐, 계속 연결되어 있는 것이 아니다.)
  - 무상태(stateless)
    - 접속이 끊어지면 클라이언트와 서버 간의 통신이 끝나며 상태를 저장하지 않음
      - 네이버 메인페이지에서 로그인을 했으면 뉴스 탭으로 이동했을 때 로그인이 풀리지는 않는다. 그러나 무상태의 특성에 의하면 로그인이 풀려야 한다. 페이지를 이동하는 순간 로그인 상태가 사라지는 것. 결국은 이 상태를 계속해서 유지시키기 위한 방법으로 **쿠키**가 등장한다.
- Cookie(쿠키)
  - 서버가 사용자의 웹 브라우저에 전송하는 작은 데이터 조각
  - 브라우저는 전송 받은 쿠키를 로컬에 key-value 데이터 형식으로 저장
    - 동일한 서버에 재 요청 시 저장된 쿠키를 함께 전송
    - (요청 + 서버로부터 받은 쿠키)
  - 웹 페이지에 접속하면 요청한 웹 페이지를 받으며 쿠키를 로컬에 저장하고, 클라이언트가 재요청시마다 웹 페이지 요청과 함께 쿠키 값도 같이 전송
- Cookie 사용 목적
  - **세션 관리(상태)**
    - 로그인, 아이디 자동완성, 공지 하루 안보기, 팝업 체크, 장바구니 등의 정보 관리
  - 개인화
    - 사용자 선호, 테마 등의 세팅
  - 트래킹
    - 사용자 행동을 기록 및 분석하는 용도

- Session

  - 사이트와 특정 부라우저 사이의 state(상태)를 유지시키는 것
  - 클라이언트가 서버에 접속하면 서버가 특정 session id 를 발급하고 클라이언트는 session id를 쿠키에 저장. 클라이언트가 다시 서버에 접속할 때 해당 쿠키(session id가 저장된 쿠키)를 이용해 서버에 session id를 전달
  - 우리가 네이버에 로그인을 해서 문서와 session id가 포함된 쿠키를 받았다고 하자. 우리가 다시 다른 페이지로 들어가면(ex 뉴스), 우리는 뉴스 페이지를 달라는 요청과 함께 쿠키를 같이 보내고, 네이버는 뉴스 페이지와 함께 로그인 유지된 상태를 보내준다.
  - 쿠키는 우리가 요청을 보낼 때 마다 계속해서 보내야 한다. HTTP의 비연결지향, 무상태 특징 때문에.

  - 장고는 특정 session id를 포함하는 쿠키를 사용해서 각각의 브라우저와 사이트가 연결된 세션을 알아냄. 세션 정보는 django DB의 django_session 테이블에 저장되어 있다.
    - `INSTALLED_APPS`에 `django.contrib.sessions` 가 이미 등록되어 있다.
  - 주로 로그인 상태 유지에 사용

- Cookie lifetime(쿠키의 일생)
  - Session cookie
    - 현재 세션이 종료되면 삭제
    - 브라우저는 현재 세션이 종료되는 시기를 정의
    - 일부 브라우저는 다시 시작할 때 세션 복원(session restoring)을 사용해 계속 지속될 수 있도록 함.
  - Permanent cookie
    - Expires 속성에 지정된 날짜 혹은 Max-Age 속성에 지정된 기간이 지나면 삭제



##### 쿠팡 장바구니 시스템을 이용해 쿠키가 무엇인지 알아보자.

쿠키 확인: 개발자 도구 - Application - Storage - Cookie - https://cart.coupang.com

장바구니에 물건을 담으면 sid 라는 key에 Value가 들어있는 형태로 쿠키에 저장이 된다.

이 sid를 삭제하면 장바구니에 담은 물건이 사라진다. (비로그인 상태일 때 이야기인 듯)

##### Gitlab에 로그인하여 로그인 상태를 유지시키는 쿠키 정보를 알아보자.

개발자 도구 - Network - 맨 위으 Name을 선택 - cookies 선택

Request Cookies에서_gitlab_session을 제거해보자. 

제거한 후 새로고침하면 로그아웃이 되어 있을 것! (난 왜 제거가 안되지?)



# 간단 정리

##### Cookie & Session

- Cookie
  - 클라이언트 로컬에 파일로 저장
- Session
  - 서버에 저장
  - 이때 서버에 저장된 세션데이터를 구별하기 위한 session id는 쿠키에 저장

### HTTP 쿠키는 상태가 있는 세션을 만들도록 해준다.

---

Login 기능 구현

```python
# accounts/urls.py

from django.urls import path
from . import views
app_name = 'accounts'

urlpatterns = [
    path('login/', views.login, name='login'),
]
```

```python
# accounts/views.py

from django.shortcuts import render
# 빌트인 form 가져오기
from django.contrib.auth.forms import AuthenticationForm
# Create your views here.
# 지금까지 한 것들과 같은 형태!
def login(request):
    if request.method == 'POST':
        pass
    else: # 로그인 페이지를 보는게 먼저!
        form = AuthenticationForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/login.html', context)
```

```django
# accounts/templates/login.html

{% extends 'base.html' %}

{% block content %}
<h1>로그인</h1>
<form action="{% url 'accounts:login' %}" method="POST">
  {% csrf_token %}
  {{ form.as_p }}
  <input type="submit">
</form>
{% endblock %}
```

이제 로그인 화면으로는 잘 넘어갈 수 있게 되었다.

로그인 요청에 관한 코드를 작성해보자.

```python
# accounts/views.py

from django.shortcuts import render
from django.contrib.auth import login as auth_login
from django.contrib.auth.forms import AuthenticationForm
# Create your views here.
def login(request):
    if request.method == 'POST':
        # ModelForm이 아니라 Form이다.
        form = AuthenticationForm(request, request.POST)
        # form = AuthenticationForm(request, data=request.POST)
        if form.is_valid():
            # 세션 CREATE
            # form에서 유저 정보 가져와서 로그인 정보에 넣어주기
            auth_login(request, form.get_user())
    else: # 로그인 페이지를 보는게 먼저!
        form = AuthenticationForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/login.html', context)
```

이제 createsuperuser를 통해 관리자 아이디를 만들어서 로그인을 시도해보자.



개발자 도구-application에서 확인해보면 sessionid 라는 쿠키가 새로 생김을 알 수 있다.

우리가 가진 sqlite DB를 보면 django_session이 있는데, 이를 보면 sessionid의 쿠키와 같은 값임을 알 수 있다. **이 두 가지가 일치해야 로그인이 되는 것이다.** 장고는 **서버에 session의 정보를 저장**해 놓고, **우리가 그에 맞는 값을 입력해야 로그인이 가능**!



누가 로그인했는지 보기!

```django
# 프로젝트폴더/templates/base.html

{% load bootstrap5 %}

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  {% bootstrap_css %}
  <title>Document</title>
</head>
<body>
  <div class="container">
  {% comment %} 로그인되어 있지 않다면 anonymousUser {% endcomment %}
  {% comment %} django가 템플릿에서 바로 사용할 수 있는 컨텍스트들이 있는데,
  그 리스트를 settings.py에서 확인할 수 있다. templates에 있는 debug, request, auth, ...{% endcomment %}
  <h3>Hello, {{ request.user }}</h3>
    {% block content %}
    {% endblock %}
  </div>
  {% bootstrap_javascript %}
</body>
</html>
```

개발자 도구에서 sessionid를 또 삭제하면, Hello, AnonymousUser 라는 메세지가 출력된다. 이게 기본값!

session은 기본적으로 2주 유지되는데, 이걸 수정하고 싶으면 settings.py에서 다음과 같이 입력한다.

```django
...
USE_L10N = True

USE_TZ = True


# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/3.1/howto/static-files/

STATIC_URL = '/static/'

DAY_IN_SECONDS = 86400
SESSION_COOKIE_AGE = DAY_IN_SECONDS
```



##### Logout()

DB의 세션 데이터와 클라이언트 쿠키에서 sessionid 모두 삭제

```python
# accounts/urls.py

from django.urls import path
from . import views
app_name = 'accounts'

urlpatterns = [
    path('login/', views.login, name='login'),
    path('logout/', views.logout, name='logout'),
]
```



```python
# accounts/views.py

from django.contrib.auth import logout as auth_logout
from django.views.decorators.http import require_POST

@require_POST
def logout(request):
    # 현재 들어온 요청에 관한 session data와 그것에 연결된 DB의 세션 데이터를 삭제한다.
    auth_logout(request)
    return redirect('articles:index')
```



Logout  버튼이 보이도록 하기

```django
base.html

{% load bootstrap5 %}

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  {% bootstrap_css %}
  <title>Document</title>
</head>
<body>
  <div class="container">
  {% comment %} 로그인되어 있지 않다면 anonymousUser {% endcomment %}
  {% comment %} django가 템플릿에서 바로 사용할 수 있는 컨텍스트들이 있는데,
  그 리스트를 settings.py에서 확인할 수 있다. templates에 있는 debug, request, auth, ...{% endcomment %}
  <h3>Hello, {{ request.user }}</h3>
    <a href="{% url 'accounts:login' %}">Login</a>
    <form action="{% url 'accounts:logout' %}" method="POST">
      {% csrf_token %}
      <input type="submit" value="Logout">
    </form>
    {% block content %}
    {% endblock %}
  </div>
  {% bootstrap_javascript %}
</body>
</html>
```



**로그인 상태에서만 logout 버튼이 보이도록 만들려면?**



### 로그인 사용자에 대한 접근 제한

- `is_authenticated attribute`

  - User class의 속성
  - 사용자가 인증되었는지 확인하는 방법
  - User에 항상 True이며, AnonymousUser에 대해서만 항상 False
  - 단, 이것은 권한과는 관련이 없으며 사용자가 활성화 상태(active)이거나 유효한 세션(valid session)을 가지고 있는지도 확인하지 않음. 

- ```django
  # base.html
  ...
    {% bootstrap_css %}
    <title>Document</title>
  </head>
  <body>
    <div class="container">
    {% comment %} 로그인되어 있지 않다면 anonymousUser {% endcomment %}
    {% comment %} django가 템플릿에서 바로 사용할 수 있는 컨텍스트들이 있는데,
    그 리스트를 settings.py에서 확인할 수 있다. templates에 있는 debug, request, auth, ...{% endcomment %}
    <h3>Hello, {{ request.user }}</h3>
    {% if request.user.is_authenticated %}
      <form action="{% url 'accounts:logout' %}" method="POST">
        {% csrf_token %}
        <input type="submit" value="Logout">
      </form>
    {% else %}
      <a href="{% url 'accounts:login' %}">Login</a>
    {% endif %}
      {% block content %}
  ...
  ```

- 그런데 이렇게 바꿔도 주소를 입력해서 로그인 창에 들어갈 수 있다. 따라서 따로 제한을 두어야 한다.

- `login_required decorator`

  - 사용자가 로그인 했는지 확인하는 view를 위한 데코레이터
  - 로그인 된 사용자의 경우 해당 view 함수 실행
  - 로그인 하지 않은 사용자는 settings.LOGIN_URL에 설정된 경로로 redirect 시킴
    - LOGIN_URL의 기본 값은 '/accounts/login/'

- ```python
  from django.contrib.auth.decorators import login_required
  
  @login_required
  @require_http_methods(['GET', 'POST'])
  def create(request):
      if request.method == 'POST':
          form = ArticleForm(request.POST)
          if form.is_valid():
              article = form.save()
              return redirect('articles:detail', article.pk)
      else:
          form = ArticleForm()
      context = {
          'form': form,
      }
      return render(request, 'articles/create.html', context)
  ```

- delete와 update에도 붙여주자.



**delete logic의 함정**

비로그인 상태로 글을 삭제하려 할 때 login_required 때문에 login  페이지로 이동한다. 그럼 login을 하면 지워져야 하는데, 에러가 나버린다. 로그인에 성공을 해도 첫 번째 데코레이터를 지나서 다시 요청이 갈 때, 이 때 요청이 GET이 되어버려서(redirect는 get 요청) method not allowed 에러가 나는 것.

@login_required를 지우고, 함수 내부에서 로그인 여부를 판단하면 올바르게 동작하게 된다.

```python
# delete 수정 전
@login_required
@require_POST
def delete(request, pk):
    article = get_object_or_404(Article, pk=pk)
    article.delete()
    return redirect('articles:index')

# delete 수정 후
@require_POST
def delete(request, pk):
    if request.user.is_authenticated:
        article = get_object_or_404(Article, pk=pk)
        article.delete()
    return redirect('articles:index')
```



# User 만들기

```python
# accounts/urls.py
from django.urls import path
from . import views
app_name = 'accounts'

urlpatterns = [
    path('login/', views.login, name='login'),
    path('logout/', views.logout, name='logout'),
    path('signup/', views.signup, name='signup'),
]
```

회원가입을 위한 form = UserCreationForm

```python
# accounts/views.py
from django.contrib.auth.forms import AuthenticationForm, UserCreationForm

def signup(request):
    if request.method == "POST":
        pass
    else:
        form = UserCreationForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/signup.html', context)
```

login과 완전히 같은 형태..

```django
# accounts/signup.html

{% extends 'base.html' %}

{% block content %}
<h1>회원가입</h1>
<form action="" method="POST">
  {% csrf_token %}
  {{ form.as_p }}
  <input type="submit">
</form>
{% endblock %}
```



**회원가입에 조건 걸기**

```django
...
{% if request.user.is_authenticated %}
    <form action="{% url 'accounts:logout' %}" method="POST">
      {% csrf_token %}
      <input type="submit" value="Logout">
    </form>
  {% else %}
    <a href="{% url 'accounts:login' %}">Login</a>
    <a href="{% url 'accounts:signup' %}">Signup</a>
  {% endif %}
    {% block content %}
    {% endblock %}
```



django-authentication system에서 built-in forms 들어가기

class UserCreationForm을 보면 ModelForm임을 알 수 있다.

(ModelForm의 첫 번째 인자에는 data가 들어간다.)

```python
def signup(request):
    if request.method == "POST":
        form = UserCreationForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('articles:index')
    else:
        form = UserCreationForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/signup.html', context)
```

이제 회원가입 가능!



회원가입을 하고 나면 index 화면으로 넘어가는데, 그 대신 로그인 된 후의 화면으로 넘어가게 만들 수 있을까?

```python
# 회원 가입 후 로그인까지 해주는 코드!

def signup(request):
    if request.method == "POST":
        form = UserCreationForm(request.POST)
        if form.is_valid():
            user = form.save() # save()의 리턴 값이 user다! github에서 확인 가능
            auth_login(request, user)
            return redirect('articles:index')
    else:
        form = UserCreationForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/signup.html', context)
```

이제 user에 대해 C를 완성했다. 나머지를 해보자.



# User Delete

```python
# accounts/urls.py
path('delete/', views.delete, name='delete'), 추가!
```

```python
# accounts/views.py

@require_POST
def delete(request):
    if request.user.is_authenticated:
        request.user.delete()
    return redirect('articles:index')
```

```django
# base.html
... 원하는 위치에
    <form action="{% url 'accounts:delete' %}" method="POST">
    {% csrf_token %}
    <input type="submit" value="탈퇴">
    </form>
...
```



# User Update

```python
# accounts/urls.py
path('update/', views.update, name='update'), 추가!
```

```python
# accounts/views.py
from django.contrib.auth.forms import AuthenticationForm, UserCreationForm, UserChangeForm

def update(request):
    if request.method == "POST":
        pass
    else:
        form = UserChangeForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/update.html', context)
```

```django
# base.html
적절한 위치에 다음 코드 추가
<a href="{% url 'accounts:update' %}">[회원정보수정]</a>
```

```django
# update.html
{% extends 'base.html' %}

{% block content %}
<form action="" method="POST">
  {% csrf_token %}
  {{ form.as_p }}
</form>

{% endblock content %}
```

이렇게 만들면 superuser를 만들 수도 있꼬 하는 등.. 너무 많은 field를 제공하게 된다.

이를 제한하기 위해 account에 들어서 처음으로 form을 만들어보자.

---

- user objects

  - django 인증 시스템의 핵심
  - Users가 django 인증 시스템에서 표현되는 모델
  - 일반적으로 사이트와 상호작용 하는 사람들을 나타냄
  - django 인증 시스템에서는 오직 하나의 User Class만 존재
  - AbstractUser Class의 상속을 받음

  django github -> django - contrib - auth - models.py

`class User(AbstractUser):` 찾기

얘는 별게 없네? -> AbstractUser 찾기

username, first_name, last_name, email, is_staff 등의 user object 필드명이 이 AbstractUser 에 만들어져 있다.



- **AbstractUser**
  - User model을 구현하는 완전한 기능을 갖춘 **기본 클래스**
- AbstractBaseUser
  - password, last_login, is_active의 세 가지 필드만 정해져있다. 얘가 AbstractUser에 상속되어 완전한 기능을 갖추게 된다!!

상속 관계

models.Model -> AbstractBaseUser -> AbstractUser -> User

---

# Form Custom

UserChangeForm을 커스텀하는 경우

```python
# accounts/forms.py

from django.contrib.auth.forms import UserChangeForm
# from django.contrib.auth.models import User 직접 참조하는건 권장하지 않음
from django.contrib.auth import get_user_model
class CustomUserChangeForm(UserChangeForm):

    class Meta:
        model = get_user_model()
        fields = ('email', 'first_name', 'last_name',)
```

**get_user_model이란?**

django user model 검색 - Referencing the User model 을 읽어보면,

user를 직접 참조하려고 하면 나중에 우리가 user model을 커스텀했을 때 제대로 동작하지 않을 것이므로 현재 활성화되어 있는 user model을 참조하는 get_user_model을 사용하라고 한다.

이제 다시 views.py에서 update를 살짝 바꿔주자.

```python
from .forms import CustomUserChangeForm()
# 이제 from django.contrib.auth.forms import UserChangeForm은 필요 없음

def update(request):
    if request.method == "POST":
        pass
    else:
        form = CustomUserChangeForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/update.html', context)
```

이제 pass 부분을 채워보면?

```python
def update(request):
    if request.method == "POST":
        # ModelForm
        form = CustomUserChangeForm(request.POST, instance=request.user)
        if form.is_valid():
            form.save()
            return redirect('articles:index')
    else:
        form = CustomUserChangeForm()
    context = {
        'form': form,
    }
    return render(request, 'accounts/update.html', context)
```

instance에 request에 들어있는 user 정보를 넣으면 된다.

조금 더 수정해보면,

```python
from django.contrib.auth.decorators import login_required

# 로그인 한 사용자만 계정 정보를 업데이트 할 수 있도록
@login_required
def update(request):
    if request.method == "POST":
        # ModelForm
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
```



# Password change

```python
# accounts/urls.py
path('password/', views.change_password, name='change_password'),
```



```python
# accounts/views.py
# 비밀번호 수정 페이지
# django는 비밀번호 관련 설정은 password 링크에 만들어달라고 해놨음.
# 장고는 또 비밀번호 변경 관련 클래스를 제공한다.
from django.contrib.auth.forms import PasswordChangeForm
@login_required
def change_password(request):
    if request.method == "POST":
        form = PasswordChangeForm(request.user, request.POST)
    else:
        form = PasswordChangeForm(request.user)
        context = {
            'form': form
        }
        return render(request, 'accounts/change_password.html', context)
```



```django
# change_password.html
{% extends 'base.html' %}

{% block content %}
<form method="POST">
  {% csrf_token %}
  {{ form.as_p }}
  <button>비밀번호 수정</button>
</form>
{% endblock content %}
```

