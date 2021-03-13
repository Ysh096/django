# 관통 PJT - pair programming

### Virtual Environment

우리는 배쉬 창에서 python이라는 명령어를 치는 순간에 환경변수 경로에 있는 python.exe 파일을 불러오게된다. 

- 각 가상 환경은 고유한 파이썬 환경을 가지며 독립적으로 설치된 패키지 집합을 가짐

가상환경 지원 시스템

- venv, virtualenv, conda, pyenv
- 파이썬 3.3부터 venv가 기본 모듈로 내장

- scripts 경로에 있는 pip..exe로 설치한 패키지들은 각 파이썬 버전의 Lib에 저장되는데, 이는 인터넷에 있는 어떤 파이썬 패키지를 그대로 가져오는 것과 같다.
- 프로젝트마다 달라질 수 있는 라이브러리!



99_virtual 폴더에 들어가 vs코드로 열기

`python -m venv venv`

python.exe를 실행(python)하는데, 오른쪽에 등장하는 모듈을 실행시켜줘(-m)

그 모듈은 venv이고, 이는 가상화 작업인데 그 가상화 작업을 하는 폴더의 이름은 venv이다!

이 코드를 실행하면 venv라는 폴더가 생성된다.

---

**주의!**

**git에서의 -m은 message**

**python에서의 -m은 module의 약자**

---

### venv 활성화

venv 폴더에 들어가보면 Lib-site-packages라는폴더에 여러 패키지들이 들어가게 됨을 알 수 있다.

`source venv/Scripts/activate`를 입력하면 (venv)라는 출력이 나타나고,

pip list를 치면 패키지가 pip와 setuptools밖에 없게 된다! 즉 가상환경이 실행된 것!



이러한 새로운 가상환경에서 작업을 할 때에는 새롭게 라이브러리를 설치하면서 작업을 해야한다.

새로운 가상환경은 완전히 독립된 상태라고 할 수 있다. 각 프로젝트에 필요한 라이브러리를 설치하면 된다.



### venv를 비활성화 하는 방법

`deactivate` 입력



### django 설치

pip install django를 하면 venv/Lib/site-packages에 django가 설치됨.

우리가 프로젝트를 git으로 관리할 때, 이러한 라이브러리가 저장된 venv는 관리하지 않는다.

`.gitignore로!`

python django windows macos vscode..를 쳐서 .gitignore를 만들어보면 .venv가 들어있다.



앞으로 모든 프로젝트를 진행할때마다 venv를 만들 것이다.

다른 폴더에 있는 venv를 불러와서 사용할 수도 있지만 우리는 프로젝트 폴더 내에 venv를 만들어 사용하고, gitignore로 예외처리 해줄 것!



### 프로젝트 시작

폴더 생성

`python -m venv venv`

가상환경 시작

`source venv/Scripts/activate`



`django-admin startproject crud .`

crud 폴더 바로 아래에 urls.py나 settings.py 등의 프로젝트 파일을 만들어준다.



### 패키지 관리

README에 우리가 어떤 버전의 라이브러리를 사용하는지 적어도 좋다.

`pip freeze`를 치면

```python
$ pip freeze
asgiref==3.3.1
Django==3.1.7
pytz==2021.1
sqlparse==0.4.1
(venv) 
```

위와 같은 출력을 해준다. 이 출력된 결과를 requirements.txt 파일에 넣어줄 것이다.

직접 txt파일을 만들어도 되지만, Linux 명령어를 이용해서 만들어보자.

`pip freeze > requirements.txt`

` 왼쪽의 실행 결과물을 오른쪽에 넣어주세요.` 라는 뜻!



이렇게 requirements.txt에 우리가 사용하는 패키지를 저장해놓으면, venv를 가지지 않은 다른 사람이 이 파일을 보고 스스로 가상환경을 설정하고 사용할 수 있게 된다. 여기서 또 pip를 이용해서 requirements에 있는 모든 패키지를 install하게 할 수 있다!! 이렇게 하면 훗날 다시 프로젝트를 실행해야 할 때 프로젝트 진행 당시의 버전으로 venv 환경을 쉽게 구성할 수 있게 된다.

`python -m venv venv`

`source venv/Scripts/activate`

`pip install -r requirements.txt`



*파이썬 버전은 README.md에 적어준다.



### 실습

pair programming을 할 때, db.sqlite 폴더는 작업자가 아니었다면 비어있게 되는데, 이 db를 채우는 방법!

일단 지금은 NAVIGATOR와 DRIVER로 나뉘어서 7:3의 비율로 코딩을 하게 될 것(네비게이터가 하라는 대로 코딩하기)

배우는 입장에서는 잘하는 사람이 coding을, 못하는 사람이 navigator를 해야 한다.



코드를 보내주는 과정에서 깃을 사용한다.

깃랩에 들어가서 pjt04 프로젝트 만들고, 메인테이너로 pair를 추가한다. git pull로 가지고 오면, db.sqlite3가 없다. (gitignore) -> runserver 후 들어가보면 table이 없다고 알려준다.



### fixture

db를 직접 주고받는 것은 지양하고, json 형식으로 데이터를 저장하도록 하자.

django admin dumpdata라고 검색해보자.

dumpdata 항목을 보면, dumpdate 하는 방법이 나온다.

`python manage.py dumpdata movies.movie`

`movies`는 앱 이름(어제의 예: articles)

`movie`는 모델의 이름(어제의 예: article, 소문자로 적음)

`python manage.py dumpdata --indent 4 movies.movie`를 하면 더 깔끔하게 가져온다. 네칸 인덴트하기 때문에..

##### 핵심!

pip freeze > ~ 를 응용해서 데이터를 가져와서 저장할 수가 있다.

`python manage.py dumpdata --indent 4 movies.movie > movies.json`

movies 앱 폴더에 fixtures라는 폴더 생성, **json이라는 파일을 fixtures에 넣어서 git으로 관리할 것이다.**



### pair가 할 일

그러면 pair는 같은 폴더를 가지게 되는데, 아직 migrate가 되지 않았기 때문에 서버를 실행해도 오류가 난다.

`python manage.py migrate`

이렇게 하면 화면은 나오지만 데이터는 없다.

이제 데이터를 불러오기 위해 다음 코드를 작성한다.

폴더 구조는 `pjt04/movies/fixtures/movies/movies.json`

`python manage.py loaddata movies/movies.json`

이제 하나의 fixture로부터 20개 오브젝트가 설치되었다고 나온다.



### migrate의 역할?

`python manage.py migrate`



- 구조선언: models.py에 많은 것을 작성하게 될 건데, 나는 이러한 데이터베이스를 사용할거야 라고 선언하는 과정이다.
- 구조 번역본: 그 후 makemigrations를 하면 번역본이 만들어짐 0001_initial.py 등..
- DB 구조 만들기: migrate, 번역본에 맞는 표를 그려준다.
- dumpdata는 DB의 파일을 전부 뽑아와서 딕셔너리 형태의 json 파일로 만들어주게 된다. loaddata는 이런 json 형태의 파일이 있을 때 DB에 이를 그대로 적용해주는 역할을 한다.

우리가 db.sqlite를 지웠다는 것은 표를 지웠다는 뜻이고, 표를 그려주는 작업 migrate를 다시 해줘야 한다. 그 다음 loaddata를 해야겠지?



OperationalError at/movies/

no such table: movies_movie

위 오류는 표가 없다는 뜻 -> 표를 그려주는 migrate 명령어 사용!



**모델을 수정하는 것은 (DB까지 모두 수정해야해서)굉장히 번거롭다. 미리 구조를 잡아놓은 후에 프로젝트를 진행해야 한다.**

