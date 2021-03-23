# RESIZE & GIT UNDO

# 1. RESIZE

사진의 크기가 제멋대로인 상황을 어떻게 해결할까?



__pair programming__

1) ptyhon -m venv venv

2) source venv/Scrips/activate

3) pip install -r requirements.txt

(나중에 할거라서 imagekit 주석처리)

4) python manage.py migrate

5) python manage.py runserver



핀터레스트와 인스타그램의 사진 레이아웃 차이!

인스타는 정사각형으로 규격화가 되어있고, 오른쪽은 이미지를 블록맞추기처럼 끼워맞춰놓음.

우리는 인스타처럼 해 볼 것이다.

우선 width를 조절해보자.

```django
{% extends 'base.html' %}

{% block content %}
  {% if post.image %}
    <img src="{{ post.image.url }}" alt="{{ post.image }}" width="100%">
  {% else %}
    <h1>이미지가 없습니다.</h1>
  {% endif %}
  <div class="bg-light fs-4 w-100 text-center my-2 p-5">{{ post.content }}</div>
  <div class="d-flex justify-content-center my-3">
    <a class="btn btn-warning" href="{% url 'posts:update' post.pk %}">수정</a>
    <a class="btn btn-danger" href="{% url 'posts:update' post.pk %}">삭제</a>
  </div>

{% endblock content %}
```

width = 100% 로 바꾸자 폭을 화면에 맞게 조정하면서 세로도 같이 조정이 된다. 그런데 이렇게 100%를 억지로 맞추다보면 작은 사진은 깨지고, 큰 사진은 압축되어 용량이 매우 큰 값을 썸네일로 보여주게 된다.



인스타에서는 사진을 올릴 때 필요없는 부분을 잘라낼수가 있는데, 이를 크롭이라고 한다. 이미지 속성을 살펴보면 1080*1080으로 크기가 변해있다. 



오늘은 이렇게 이미지 리사이징을 하는 방법을 알아본다.

# django-imagekit

다른 라이브러리를 쓸 때도 공식문서를 잘 읽고 사용할 수 있도록 연습하자.

https://pypi.org/project/django-imagekit/

1) Pillow 설치 우리는 model에 ImageField를 쓰고 있으니 Pillow를 설치했다.

2) pip install django-imagekit

3) Installed apps에 imagekit 추가

4)

```python
# models.py
from imagekit.models import ImageSpecField

from imagekit.processors import ResizeToFill

image_thumbnail = ImageSpecField 복붙하기
```

source 항목: 뭘 기준으로 내가 썸네일을 만들지에 대한 원본 데이터 우리는 image로 바꿔줘야함

processors: 이미지를 어떻게 처리할거냐

	- [ResizeToFill(가로, 세로)], # 자르고자 하는 크기
	- format = 'JPEG' # 파일 확장자 정하기
	- options = {'quality': 60} # 퀄리티 조절 가능

수정을 해주고 makemigrations를 해주면 변화 사항이 없다고 나온다. imagespecfield는 원본 데이터에서 이미지 썸네일을 잘라서 그 상황마다 사용하는 것이기 때문에 DB에 들어가는 정보가 아니다. 출력을 어떻게 하는지 살펴보면, img src에 적은 url을 다음과 같이 수정해줘야 한다고 한다.

`post.image_thumbnail.url`



폴더를 확인해 보면, 크롭된 사진을 따로 저장함을 알 수 있다.

```python
from imagekit.models import ImageSpecField
from imagekit.processors import ResizeToFill
from imagekit.models import ProcessedImageField
image_thumbnail = ImageSpecField 복붙하기
```

얘는 완성된 결과를 저장하는 것이기 때문에 DB에 저장을 한다. 얘는 makemigrations와 migrate를 해줘야한다.

### 나중에 직접 하면서 확인을 좀 해보자..



### Imagekit 날짜별, 사용자별 image 관리?

사용자가 많아지면 무수한 사진이 올라오게 될 텐데, 이러한 경로를 어떻게 설정할까?

models에 들어있는 upload_to를 조작하여 날짜별로 사진을 저장하는 방법을 알아보자.

```python
image = ProcessedImageField(upload_to='images/&Y/%m/%d/',
                            processors=~
                            ...
```



imagekit processos 검색

processor의 종류.. train border column, resize, ... 그냥

https://pypi.org/project/django-imagekit/ 들어가서 다 읽어보는게 제일 좋을 것 같다.



**사용자별 폴더는 사용자 개념을 배우고 만들어볼 것.**



# 2. form.html

### 2-1. form의 action이 가지는 의미

form action = "" 처럼 비어있다면 현재 위치로 요청을 보내겠다는 뜻이다.



### 2-2. update와 create의 차이

둘 다 form.html을 참조하지만 update와 create라는 이름의 차이가 있다.

form.html에 들어가보면 if 문으로

`if request.resolver_match.url_name == 'create'` 이런 식으로 나뉘고 있다.

이게 무슨 뜻일까?

그 전에...

`render(request, 'posts/form.html', context)` 에서 request는 뭘까?



html에 {{ request }}를 써서 확인해보면 request 객체의 형태를 알 수 있다.

django request and response objects 검색해보자.



요청을 보내면 응답을 하는게 우리가 할 일

요청은 request가, 응답은 return에 넣어줄 render같은 것이 만들어준다.



우선 우리가 많이 사용했던 것 부터 살펴보자.

- HttpRequest.method: 어떤 형태로 요청을 했는가
- HttpRequest.GET: 사용자가 url의 데이터를 가져오려 할 때. QueryDict를 반환
- HttpRequest.POST: ...
- HttpRequest.FILES: ...



오류 페이지에서 Request information을 보면 엄청나게 많은 정보가 있음.



meta, header, ... resolver_match...

resolver_match에는 뭐가 들어있을까?

resolver_match를 눌러보면 어떤 정보가 들어있는지 다 나와있다.

url_name도 그 중 하나고, path에서 적었던 name이 이에 해당한다.

---

`if request.resolver_match.url_name == 'create'`이 뭔지 궁금하다? -> 공식문서에 들어가 resolver_match를 살펴보자.

---



### 2-3. get_object_or_404

?



# 3. GIT UNDO

### 커밋 과거로 돌아가기

`git add a.txt` 를 하면 staging area에 올라가는데,

`git rm --cached a.txt` 를 하면 staging area에서 내려온다.



`git add a.txt`

`git commit -m 'add a'`

여기까지 하고 a.txt의 내용을 수정하고 gti status를 하면

`use "git restore --staged <file>..." to unstage` 라고 알려준다.

rm --cached는 처음 등록한 파일을 돌릴때, 그리고 등록 이후 수정까지 하고 다시 돌리려고 할 때에는 restore --staged를 해야한다.



### commit 내용을 잘못 입력햇을 때

commit 을 잘못했을 때 어떻게 할까?

`git commit --amend` 을 입력하면 색깔이 다른 화면이 나온다. 이를 **vim**이라고 부른다. 마우스가 없을 때 어떤 파일을 수정할 수 있도록 만들어진 프로그램

vim이라는 text editor는 입력모드 / 이동모드가 있다.



- `i` 를 누르면 끼워넣기 라고 해서 입력모드로 전환이 되고, `esc`를 누르면 취소가 된다.
- `i`를 누르면 커밋을 수정할 수 있고 esc를 누르면 끼워넣기 모드가 취소된다. 이제 저장버튼을 눌러줘야 하는데, 커맨드로 `:`를 입력하고 `wq`를 입력한다. (write & quit)
- 그러면 git commit이 수정되었음을 알 수 있다.

[vim 연습 사이트](openvim.com)



### commit을 할 때 add를 하나 빠뜨린 상황

c.txt

d.txt

c만 넣고 git status를 눌러보면 c만 추가되었고 d는 추가되지 않은 상황

이 상황에서 git commit -m 'update c, d' 라고 커밋을 했다고 해보자.

이 때 이 staging area에 d를 넣고 싶다면?



`git add d.txt` d를 staging area에 넣어주고,

`git commit --amend` 를 해주면, c를 staging area에 내려주게 된다.

이제 아무것도 안하고 다시 나가주면 staging area를 다시  commit 단계로 올려주게 된다.



###

README 파일 만들고 다음 흐름을 따라가보자. 

1) 영화를 보기 위해 영화관 도착!!

git init

git add .

git commit -m '영화관 도착'

2) 명량 영화표 구매

git add .

git commit -m '영화표 구매'

3) 팝콘 구매

git add .

git commit -m '팝콘 구매'

4) 상영 시간이 될 때까지 웹서핑

git add .

git  commit -m '웹서핑'

5) 댓글읽다가 주인공 사망 스포당함

git add .

git commit -m '스포당함'

6) 나도 스포해야지 댓글 작성

git add .

git commit -m '댓글작성'

7) 영화를 재미없게 보고 나옴

git add .

git  commit -m '영화보고 나옴'



`git log --oneline` 를 해보면 commit 메세지들이 나오는데, 과거로 돌아가보자.

세 가지 예시를 들어볼 것

git reset이라는 명령어를 써 볼 것이다.

#### 예시1)

git log --oneline

커밋 앞에 있는 이상한 숫자와 문자조합을 사용할 것이다.

**hard**

git reset --hard '가고싶은 커밋의 이름'

`git reset --hard '86c4d61'` (팝콘 구매 시점)

이제 README 파일을 보면 3번까지만 적혀있다.

과거로 완전히 돌아가는 것. 코드 자체까지 변화가 일어남!



**soft**

git log --oneline

`git reset --soft '86c4d61'` (팝콘 구매 시점)

이제 README 파일을 보면 끝까지 다 적혀 있고, 팝콘구매 이후의 내용이 add 한 기록으로 남아있다. (commit 까지는 안함)



**mixed**

git log --oneline

`git reset --mixed 86c4d61`

또는

`git reset 86c4d61`

과거로 가면서 과거에서부터 현재까지 기록이 working directory에 남아있게 된다. (add까지 풀림) 내용은 남아있음.



