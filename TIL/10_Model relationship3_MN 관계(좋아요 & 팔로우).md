# Model Relationship3

## M:N 관계(좋아요 & 팔로우)

A many-to-many relationship



1) django, django-extensions, ipython 설치

2) 프로젝트 이름 crud, 앱 이름 hospitals

앱 등록! (hospitals, django-extensions)



3) models.py 작성

```python
# hospitals/models.py
from django.db import models

# Create your models here.
class Doctor(models.Model):
    name = models.TextField()

    def __str__(self):
        return f'{self.pk}번 의사 {self.name}'

class Patient(models.Model):
    name = models.TextField()
    doctor = models.ForeignKey(Doctor, on_delete=models.CASCADE)

    def __str__(self):
        return f'{self.pk}번 환자 {self.name}'
```

4) makemigrations, migrate 후 django-extensions 실행!

**현재 1:N 관계이다.**

5) 다음과 같이 의사와 환자를 만들어보자.

```sql
In [1]: doctor1 = Doctor.objects.create(name='justin')
In [2]: doctor2 = Doctor.objects.create(name='eric')
In [3]: patient1 = Patient.objects.create(name='tony', doctor=doctor1)
In [4]: patient2 = Patient.objects.create(name='harry', doctor=doctor2)

--확인--
In [5]: patient1
Out[5]: <Patient: 1번 환자 tony>

In [6]: patient2
Out[6]: <Patient: 2번 환자 harry>
```



환자가 의사를 바꾸려고 하면?

6) 의사 바꾸기

```sql
In [7]: patient3 = Patient.objects.create(name='tony', doctor=doctor2)
```

1:N 관계에서는 patient1을 바꾸는 것은 불가능하고 새로운 patient3를 만드는 방법으로 의사를 바꾸어야 한다.

7) 2명의 의사에게 예약하고 싶다면?

```sql
In [8]: patient4 = Patient.objects.create(name='harry', doctor=doctor1, doctor2)
  File "<ipython-input-8-6edaf3ffb4e6>", line 1
    patient4 = Patient.objects.create(name='harry', doctor=doctor1, doctor2)
                                                                    ^
SyntaxError: positional argument follows keyword argument


In [9]: patient4 = Patient.objects.create(name='harry', doctor=doctor1, doctor=doctor2)
  File "<ipython-input-9-2775590f4f3f>", line 1
    patient4 = Patient.objects.create(name='harry', doctor=doctor1, doctor=doctor2)
                                                                    ^
SyntaxError: keyword argument repeated
```

harry를 2명의 의사에게 예약을 하고 싶은데, 안된다!

아, 이 둘로는 이런 관계를 표현할 수가 없다.

=> 중개 테이블 만들기



8) models.py 수정

외래키를 지우고 세 번째 테이블을 만든다.

```python
from django.db import models

# Create your models here.
class Doctor(models.Model):
    name = models.TextField()

    def __str__(self):
        return f'{self.pk}번 의사 {self.name}'

class Patient(models.Model):
    name = models.TextField()
    # doctor = models.ForeignKey(Doctor, on_delete=models.CASCADE)

    def __str__(self):
        return f'{self.pk}번 환자 {self.name}'

class Reservation(models.Model):
    doctor = models.ForeignKey(Doctor, on_delete=models.CASCADE)
    patient = models.ForeignKey(Patient, on_delete=models.CASCADE)

    def __str__(self):
        return f'{self.doctor_id}번 의사의 {self.patient_id}번 환자'
```



9) (필요하다면)DB 초기화 및 makemigrations, migrate

그 후 DB를 보면

- hospitals_doctor
  - id
  - name

- hospitals_patient
  - id
  - name

- hospitals_reservation
  - id
  - doctor_id
  - patione_id

위의 형태로 테이블이 만들어진다!



10) shell_plus 실행 및 의사, 환자 만들기

```sql
In [1]: doctor1 = Doctor.objects.create(name='justin')

In [2]: patient1 = Patient.objects.create(name='tony')
```

환자 하나와 의사 하나를 만들었다.



11) 예약 하나 만들어보기

```sql
In [3]: Reservation.objects.create(doctor=doctor1, patient=patient1)
Out[3]: <Reservation: 1번 의사의 1번 환자>
```



12) 예약 조회(의사와 환자 입장에서)

의사 입장에서 예약을 조회한다는 것은 Reservation class를 역참조 하는 것과 같다.

```sql
In [4]: doctor1.reservation_set.all()
Out[4]: <QuerySet [<Reservation: 1번 의사의 1번 환자>]>
```



환자 역시 예약을 조회한다는 것은 Reservation class를 역참조 하는 것과 같다.

```sql
In [5]: patient1.reservation_set.all()
Out[5]: <QuerySet [<Reservation: 1번 의사의 1번 환자>]>
```

**둘 다 하나의 예약 테이블을 역참조하여 조회를 할 수 있음!**



13) 두 번째 환자 만들어보기

환자를 만들어서 1번 의사에게 예약해보자.

```sql
In [6]: patient2 = Patient.objects.create(name='harry')

In [7]: Reservation.objects.create(doctor=doctor1, patient=patient2)
Out[7]: <Reservation: 1번 의사의 2번 환자>
```



14) 예약 확인해보기

의사가 확인을 해보면,

```sql
In [9]: doctor1.reservation_set.all()
Out[9]: <QuerySet [<Reservation: 1번 의사의 1번 환자>, <Reservation: 1번 의사의 2번 환자>]>
```



2번 환자가 확인해보면,

```sql
In [10]: patient2.reservation_set.all()
Out[10]: <QuerySet [<Reservation: 1번 의사의 2번 환자>]>
```



QuerySet은 다음과 같이 출력이 가능함!

```sql
In [14]: for reservation in doctor1.reservation_set.all():
    ...:     print(reservation.patient.name)
    ...: 
tony
harry
```



15) 

M:N(다대다) 관계: 두 개의 1:N 관계를 가진 중개 테이블을 기준으로 만들어지는 관계

django에서는 M:N 관계를 위해 만들어놓은 field가 따로 있다.



models.py를 수정할 건데, Patient에 두든, Doctor에 두든 상관없다. 편한 곳에 다음과 같이 작성한다.

```python
from django.db import models

# Create your models here.
class Doctor(models.Model):
    name = models.TextField()

    def __str__(self):
        return f'{self.pk}번 의사 {self.name}'

class Patient(models.Model):
    name = models.TextField()
    # doctor = models.ForeignKey(Doctor, on_delete=models.CASCADE)
    # 참조하는 클래스의 복수형으로 이름 작성
    doctors = models.ManyToManyField(Doctor)


    def __str__(self):
        return f'{self.pk}번 환자 {self.name}'
# 이제 reservation은 주석처리!
# class Reservation(models.Model):
#     doctor = models.ForeignKey(Doctor, on_delete=models.CASCADE)
#     patient = models.ForeignKey(Patient, on_delete=models.CASCADE)

#     def __str__(self):
#         return f'{self.doctor_id}번 의사의 {self.patient_id}번 환자'
```

이제 makemigrations를 하면, 두 개의 모델이 설계도에 형성된다.

중개 테이블이 없다?

=> **db.sqlite3 확인해보면 세 개의 테이블이 만들어져 있다.**



- hospitals_doctor
  - id
  - name
- hospitals_patient
  - id
  - name
- hospitals_patient_doctors
  - id
  - patient_id
  - doctor_id

위의 테이블이 만들어져 있음!

중개 테이블 이름은

 **`앱 이름_ManyToManyField를 쓴 클래스_ManyToMnayField를 참조한 인스턴스 이름`**



16) 예약 생성

아까는 Reservation이라는 중개 테이블을 중심으로 예약을 만들었는데, 이제 서로 바로바로 참조하여 예약을 할 수 있게 되었다.

한 번 해보자! shell_plus 실행

```sql
In [1]: doctor1 = Doctor.objects.create(name='justin')

In [2]: doctor2 = Doctor.objects.create(name='tony')

In [3]: patient1 = Patient.objects.create(name='eric')

In [4]: patient2 = Patient.objects.create(name='harry')
```

아까는 이렇게 했다.

```sql
Reservation.objects.create(key1=~, key2=~)
```

이번에는 다음과 같이 예약을 생성한다.

```sql
In [5]: patient1.doctors.add(doctor1)
```

환자가 ManyToMany 관계에 있는 의사를 추가했다.

**환자 입장에서 예약한 의사를 확인**하려면?

```sql
In [6]: patient1.doctors.all()
Out[6]: <QuerySet [<Doctor: 1번 의사 justin>]>
```

그러면 **의사는 자기한테 예약된 환자를 어떻게 볼까**?

```sql
In [7]: doctor1.patient_set.all()
Out[7]: <QuerySet [<Patient: 1번 환자 eric>]>
```

자신을 참조하고 있는 field를 가진 애를 참조하는 **역참조**!



**1이 N을 참조하는 것이 역참조가 아니라, 자신을 참조하고 있는 field를 가진 애를 참조하는 것이 역참조다**



의사가 환자를 예약해보자. 역참조이기 때문에 중간 모델 매니저만 달라진다.

```sql
In [8]: doctor1.patient_set.add(patient2)
```

확인해보자.

```sql
In [10]: doctor1.patient_set.all()
Out[10]: <QuerySet [<Patient: 1번 환자 eric>, <Patient: 2번 환자 harry>]>

In [11]: patient2.doctors.all()
Out[11]: <QuerySet [<Doctor: 1번 의사 justin>]>
```

의사 입장에서 **예약 취소**를 하려면?

```sql
In [12]: doctor1.patient_set.remove(patient1)
```

관계를 끊는다 == 삭제

확인해보자.

```sql
In [13]: doctor1.patient_set.all()
Out[13]: <QuerySet [<Patient: 2번 환자 harry>]>
```

환자가 예약을 취소하려면?

```sql
In [14]: patient2.doctors.remove(doctor1)
```



이렇게 중개 테이블이 자동으로 생기기 때문에 중개 모델이 필요 없는건가?

=> 그건 아니다. 중개 테이블을 직접 작성해야 하는 경우가 있다.

- 중개 테이블에서 추가 데이터를 사용할 때(외래 키뿐만 아니라 다른 정보가 필요할 때)
- 예를 들어 병원 예약에서 시간을 함께 설정할 때..

이렇게 중개 테이블을 만들었다면, ManyToManyField에서 하나의 명령어를 추가해야 한다.

`doctors = models.ManyToManyField(Doctor, through='Reservation')`



### 역참조 모델 매니저의 이름을 변경하고 싶다면?

`related_name`

`doctors = models.ManyToManyField(Doctor, related_name='patients')`

이렇게 하면, 

- `patient.doctors.all()`

- `doctor.patients.all()`
  - 원래 `doctor.patient_set.all()` 이었음

이제는 `patient_set`은 **사용할 수가 없다**.

```sql
In [3]: doctor1.patients.all()
Out[3]: <QuerySet []>

In [4]: doctor1.patient_set.all()
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-4-e81b89c43a95> in <module>
----> 1 doctor1.patient_set.all()

AttributeError: 'Doctor' object has no attribute 'patient_set'
```



### ManyToMany 간단 정리

- 실제로 **M, N 두 테이블 사이에 변하는 것은 없다**.
- M:N은 1:N과 달리 의사에게 진찰받는 환자, 환자를 진찰하는 의사 처럼 관계가 두 방향 모두 성립한다.



##### ManyToManyField()

- M:N 관계를 나타내기 위해 사용하는 필드
- 하나의 필수 위치 인자(M:N 관계로 설정할 모델 클래스)가 필요
- 필드의 RelatedManager를 사용하여 관련 개체를 추가, 제거 또는 만들 수 있음
  - Realate Manager란?
    - 1:N 또는 M:N 관련 컨텍스트에서 사용되는 매니저
    - patient.`doctors`.all()
    - doctor.`patient_set`.all()
    - methods
      - 같은 이름의 메서드여도 각 관계(1:N, M:N)에 따라 다르게 동작
      - 1:N에서는 target 모델 객체만 사용 가능
        - Article, Comment 관계에서 FK는 Comment가 가지고 있고, target모델은 Article, Source 모델은 Comment.
        - 역참조는 target모델이 source 모델을 참조하는 것
      - M:N 에서는 `관련된 두 객체에서 모두 사용 가능`
      - add(), create(), remove(), clear(), set()
        - add(): 지정된 객체를 관련 객체 집합에 추가, 이미 존재하는 관계에 다시 사용하면 관계가 복제되지 않음.
        - remove(): 관련 객체 집합에서 지정된 모델 객체를 제거, 관계를 끊는다!
- Arguments
  - related_name
    - target model이 source model을 참조할 때 사용할 manager name
    - ForeignKey의 related_name과 동일
    - Relate Manager의 이름을 바꿀 때
  - symmetrical
    - 자기 자신을 참조하는 경우(재귀적), 대칭적이라고 간주
    - source 인스턴스가 target인스턴스를 참조하면, target 인스턴스도 source 인스턴스를 참조하게 됨(내가 상대를 팔로우하면, 자동으로 상대도 나를 팔로우하게 됨)
    - self와의 M:N 관계에서 대칭을 원하지 않는 경우 symmetrical을 False로 설정
    - 재귀적일 때 from_user_id / to_user_id 이런 식으로 테이블이 만들어진다.
  - through
    - 중개 모델을 직접 만들 때
    - 추가 데이터를 다대다 관계와 연결하려는 경우
  - limit_choices_to, through_fields, db_table, ...



개인적으로 기능을 알아봐야 할 때에는..

django model field 공식문서

ManyToManyField 열어보기



# LIKE & FOLLOW

08_django_model_relationship_오후



### 좋아요 기능

article 입장에서 좋아요를 누른 모든 유저를 조회해야 한다. (게시글-유저의 다대다 관계)

`article.like_users.all()`

```python
from django.db import models
from django.conf import settings

# Create your models here.
class Article(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    like_users = models.ManyToManyField(settings.AUTH_USER_MODEL)
    title = models.CharField(max_length=10)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.title
```

이대로 makemigrations를 하려고 하면 오류가 뜬다.

```python
$ python manage.py makemigrations
SystemCheckError: System check identified some issues:

ERRORS:
articles.Article.like_users: (fields.E304) Reverse accessor for 'Article.like_users' clashes with reverse accessor for 'Article.user'.     
        HINT: Add or change a related_name argument to the definition for 'Article.like_users' or 'Article.user'.
articles.Article.user: (fields.E304) Reverse accessor for 'Article.user' clashes with reverse accessor for 'Article.like_users'.
        HINT: Add or change a related_name argument to the definition for 'Article.user' or 'Article.like_users'.
```

왜냐하면 현재 User와 Article은 1:N과 M:N으로 묶이고, 각각 article.user. ~ 와 article.like_users.~ 로 표현이 되는데, user에서 역참조 시에는 user.article_set.~ user.article_set.~ 으로 동일한 이름을 갖게 되기 때문이다.

따라서 둘 중 하나에서 related_name을 통해 manager의 이름을 바꿔줘야 한다.



```python
# article/models.py
from django.db import models
from django.conf import settings

# Create your models here.
class Article(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    like_users = models.ManyToManyField(settings.AUTH_USER_MODEL, related_name='like_articles')
    title = models.CharField(max_length=10)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.title
```

이제 역참조시에 user.like_articles.~(M:N) 와 user.articles.~(1:N) 로 구분이 된다.



##### article에 좋아요 기능 추가하기

```python
# articles/urls.py
path('<int:article_pk>/likes/', views.likes, name='likes'),
```

```python
# articles/views.py
@require_POST
def likes(request, article_pk):
    if request.user.is_authenticated: # 로그인 한 사용자만..
        article = get_object_or_404(Article, pk=article_pk)
		# 적어도 하나가 있는지 없는지 판단(전체를 확인하지 않아도 됨)
        if article.like_users.filter(pk=request.user.pk).exists():
        # exists와 in 중 하나를 골라 사용하면 되는데, exists가 낫다.
        #if request.user in article.like_users.all():
            # 이미 좋아요를 눌렀으면 다시 누를 때 좋아요 취소
            article.like_users.remove(request.user)
        else: # 아직 안 눌렀으면
            # 좋아요 누름
            # article에 좋아요를 누른 user를 like_users에 추가
            article.like_users.add(request.user)
        return redirect('articles:index')
    return redirect('accounts:login')
```

```html
# articles/index.html

...
    <p>글 내용 : {{ article.content }}</p>
    <div>
      <form action="{% url 'articles:likes' article.pk %}" method="POST">
        {% csrf_token %}
        {% if request.user in article.like_users.all %}
          <button>좋아요 취소</button>
        {% else %}
          <button>좋아요</button>
        {% endif %}
      </form>
    </div>
    <a href="{% url 'articles:detail' article.pk %}">[DETAIL]</a>
...
```

이제 좋아요를 누르고 DB를 확인해보면..

- 2번 유저(user_id)가 1번 article(article_id)에 좋아요를 눌렀다.

- 1번 article에 2번 유저가 좋아요를 눌렀다.

두 가지로 해석할 수 있는 데이터를 볼 수 있다.



##### 몇 명이나 좋아요를 눌렀는지 확인하기

```html
<p>{{ article.like_users.all|length }}명이 이 글을 좋아합니다.</p>

또는
<p>{{ article.like_users.all.count }}명이 이 글을 좋아합니다.</p>
```



### exists()의 역할

django queryset API 문서 확인

exists() 부분 확인!

- 쿼리셋에 결과가 포함되어 있으면 True를, 그렇지 않으면 False를 반환한다.
- 가능한 가장 간단하고 빠른 방법이다. **in과 동일한 역할이지만 더 빠름!**
- 특히 **큰 쿼리셋 내에 있는 객체를 검색**하는데에 유용하다.

```python
entry = Entry.objects.get(pk=123)
if some_queryset.filter(pk=entry.pk).exists():
    print("Entry contained in queryset")
```

```python
if entry in some_queryset:
   print("Entry contained in QuerySet")
```

아래 것 보다 위가 훨씬 빠르다!!

DB에 데이터가 날아가는 시점은 평가(if문)가 이뤄질 때이다. 그 때 장고는 쿼리셋을 메모리의 어딘가(캐시)에 저장을 해둔다. 다음 평가때 같은 쿼리셋이라면 기존에 저장해놓은 쿼리셋을 활용하려고 하기 때문이다. 그런데 in은 이 캐시 내에 있는 쿼리셋을 다 쓴 다음에 거기서 확인을 한다. 그런데 exists()는 평가를 할 때 저장을 하지 않는다.(캐시를 만들지 않는다.) **우리가 필요한 것은 있는지 없는지 여부이기 때문에 전체 쿼리셋을 확인할  필요는 없다??** 좀 찾아보자.



### Font Awesome

회원가입 후 내 kit를 보면 부트스트랩처럼 CDN을 발급받아 사용할 수 있다.

base.html이나 원하는 페이지에 이 키를 붙여넣은 다음(head든 body든 상관 없는듯?)

원하는 아이콘을 찾아 선택하면 태그를 어떻게 써야 하는지까지 알려준다.



### django gravatar

링크를 연결하면 gravatar에서 지정한 프로필 이미지가 지정된다. 사이트마다 다른 이미지가 아니라 gravatar에서 저장해놓은 이미지를 가져다 쓰는 것.



### django one-to-one

검색해보고 시간이 나면 도전해보기..



### OAuth

SNS 로그인 기능 구현에 사용. 네이버 기술 블로그를 보고 해보자. [OAuth와 춤을](https://d2.naver.com/helloworld/24942)



### FOLLOW 기능

##### 우선은 유저 프로필 페이지 작성

```python
# accounts/urls.py
path('<username>/', views.profile, name='profile'),
# username은 default값인 str형태
```

```python
# accounts/views.py
def profile(request, username):
    # 변수명은 user를 지양하자.
    person = get_object_or_404(get_user_model(), username=username)
    context = {
        'person': person,
    }
    return render(request, 'accounts/profile.html', context)
```

```html
# accounts/profile.html

{% extends 'base.html' %}

{% block content %}
<h1>{{ person.username }}님의 프로필</h1>

<hr>

<h2>{{ person.username }}'s 게시글</h2>
{% for article in person.article_set.all %}
  <div>{{ article.title }}</div>
{% endfor %}

<hr>

<h2>{{ person.username }}'s 댓글</h2>
{% for comment in person.comment_set.all %}
  <div>{{ comment.content }}</div>
{% endfor %}

<hr>

<h2>{{ person.username }}'s likes</h2>
{% for article in person.like_articles.all %}
  <div>{{ article.title }}</div>
{% endfor %}

<hr>

<a href="{% url 'articles:index' %}">[back]</a>

{% endblock content %}
```



##### 로그인 상태에서만 내 프로필로 가기 만들어주기

```html
# base.html
...
<div class="container">
    <h3>Hello, {{ request.user }}</h3>
    
    {% if request.user.is_authenticated %}
      <a href="{% url 'accounts:profile' request.user.username %}">[내 프로필]</a>
      <a href="{% url 'accounts:update' %}">[회원정보수정]</a>
      <form action="{% url 'accounts:logout' %}" method="POST">
        {% csrf_token %}
        <input type="submit" value="Logout">
      </form>
...
```



##### 게시글 작성자의 프로필로 이동하기

```html
{% extends 'base.html' %}

{% block content %}
  <h1>Articles</h1>
  {% if request.user.is_authenticated %}
    <a href="{% url 'articles:create' %}">[CREATE]</a>
  {% else %}
    <a href="{% url 'accounts:login' %}">[새 글을 작성하려면 로그인하세요.]</a>
  {% endif %}
  <hr>
  {% for article in articles %}
    <p><b>작성자 : <a href="{% url 'accounts:profile' article.user.username %}">{{ article.user }}</a></b></p>
    <p>글 번호 : {{ article.pk }}</p>
    <p>글 제목 : {{ article.title }}</p>
    <p>글 내용 : {{ article.content }}</p>
    <div>
      <form action="{% url 'articles:likes' article.pk %}" method="POST">
        {% csrf_token %}
        {% if request.user in article.like_users.all %}
          <button>좋아요 취소</button>
        {% else %}
          <button>좋아요</button>
        {% endif %}
      </form>
    </div>
    <p>{{ article.like_users.all|length }}명이 이 글을 좋아합니다.</p>
    <a href="{% url 'articles:detail' article.pk %}">[DETAIL]</a>
    <hr>
  {% endfor %}
{% endblock %}
```

article.user.username을 사용해야 게시글의 username을 전달할 수 있음에 주의!

request.user.username이 아님!!(얘는 지금 로그인한 사람의 username)



##### FOLLOW 구현 시작!

user모델을 대체하고나서 처음으로 user모델을 수정해보자.

두 유저가 있을 때 1번 유저가 2번 유저를 팔로잉 하면, 2번 유저한테 1번 유저는 팔로워다. 2번 유저의 팔로잉은 0이다. 1번은 팔로워가 0이고, 팔로잉은 1이다. 

만약 대칭이라는 기능을 사용하게 되면 팔로워와 팔로우(팔로잉)가 같을 것이지만 이런 식으로는 구현하지 않는다. symmetrical의 기본값은 True이므로 False로 바꿔줘야 한다. True일 때 symmetrical은 역참조를 할 필요가 없는데, False로 바꾸면 그때부터 역참조를 해야 해서 related_name이 필요해진다.

```python
# accounts/models.py

from django.db import models
from django.contrib.auth.models import AbstractUser

# Create your models here.
class User(AbstractUser):
    followings = models.ManyToManyField('self', symmetrical=False, related_name='followers')
```

모델에 변경사항이 생겼으므로 python manage.py makemigrations, migrate

이제 중개 테이블에 생겼다. symmetrical=True라면 자동 맞팔이겠지만 그건 용납못함.

accounts_user_followings

- from_user_id
- to_user_id



follow에 필요한 것은 무엇일까? => user_pk값

```python
# accounts/urls.py
path('<int:user_pk>/follow/', views.follow, name='follow'),
```

```python
# accounts/views.py
@require_POST
def follow(request, user_pk):
    if request.user.is_authenticated:
        # User = get_user_model()
        # 팔로우 받는 사람
        you = get_object_or_404(get_user_model(), pk=user_pk)
        me = request.user
        
        #나 자신은 follower할 수 없음
        if you != me:
            # pk를 씀에도 get이 아닌 filter를 사용하는 이유: 값이 없어도 오류가 나지 않도록 하기 위해
            # 네 팔로워들 중에 내가 있다면
            if you.followers.filter(pk=me.pk).exists():
            # if request.user in person.followers.all():
                # 팔로우 신청
                you.followers.remove(me)
            else:
                # 팔로우 끊음
                you.followers.add(me)
            return redirect('accounts:profile', you.username)
        return redirect('accounts:login')
```

```django
# accounts/profile.html

<div>
  <div>
    팔로잉 : {{ person.followings.all|length }} / 팔로워 : {{ person.followers.all|length }}
  </div>
  {% if request.user != person %}
  <div>
    <form action="{% url 'accounts:follow' person.pk %}" method="POST">
      {% csrf_token %}
      {% if request.user in person.followers.all %}
        <button>언팔로우</button>
      {% else %}
        <button>팔로우</button>
      {% endif %}
    </form>
  </div>
  {% endif %}
</div>
```

중간에 위의 코드를 추가해준다.



##### html에서 너무 긴 queryset 이름 줄이는 방법

```django
{% with followings=person.followings.all followers=person.followers.all %}
...
<div>
  <div>
    팔로잉 : {{ person.followings.all|length }} / 팔로워 : {{ person.followers.all|length }}
  </div>
  {% if request.user != person %}
  <div>
    <form action="{% url 'accounts:follow' person.pk %}" method="POST">
      {% csrf_token %}
      {% if request.user in person.followers.all %}
        <button>언팔로우</button>
      {% else %}
        <button>팔로우</button>
      {% endif %}
    </form>
  </div>
  {% endif %}
</div>
...
{% endwith %}
```

해당 변수 이름으로 변경하고자 하는 부분만 with, endwith으로 감싸주고 with안에 바꾸고자 하는 변수명을 써준다!

# 팁

form을 꾸미고 싶다?

=> form을 우선 화면에 띄워보고 F12를 눌러서 코드를 살펴보면 input 태그 내용을 볼 수 있다. 그걸 그대로 가져와서 복붙하고 스타일링만 해주면 된다.