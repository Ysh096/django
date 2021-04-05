# 1. SQL문법

Structured Query Language

RDBMS(Relational Data Base Management System)의 데이터를 관리하기 위해 설계된 언어이다.

간단히 말해 DB를 조작하기 위해 만든 언어이고, 문법에 세 가지 종류가 있다.

- DDL(Data Definition Language)
  - 데이터를 정의하기 위한 언어 
  - 테이블, 스키마 등의 관계형 DB 구조를 정의하기 위한 명령어
  - CREATE, DROP, ALTER

- DML(Data Manipulation Language)
  - 데이터를 조작하기 위한 언어
  - 저장, 수정, 삭제, 조회!
  - INSERT, UPDATE, DELETE, SELECT
- DCL(Data Control Language)
  - DB 사용자의 권한 제어를 위해 사용되는 언어
  - GRANT, REVOKE, COMMIT, ROLLBACK



### 스키마(schema)

스키마란, 데이터베이스에서 자료의 구조와 표현 방법, 관계를 정의한 구조를 의미한다.

1) Table 이름 지정

2) Column에 들어갈 이름과 타입 설정

예를 들면 다음과 같다.

|  pk  | column | datatype |
| :--: | :----: | :------: |
|  1   |   id   |   INT    |
|  2   |  age   |   INT    |
|  3   | phone  |   TEXT   |
|  4   | email  |   TEXT   |



## 2. SQLite

파일형 데이터베이스 관리 시스템

> ### 2.1 SQLite 설치
>
> [sqlite 공식 홈페이지](https://www.sqlite.org/download.html)에서 다음의 두 파일을 다운로드 받는다.
>
> - sqlite-dll-win64-x64-??????.zip
> - sqlite-tools-win32-x86-??????.zip
>
> 그 후 C드라이브에 sqlite 폴더를 생성하여 내부에 위 두 파일의 압축을 푼다.
>
> ### 2.2 환경 변수 설정
>
> 시작 - 시스템 환경 변수 편집
>
> 시스템 변수 => Path => C:\sqlite 등록
>
> 정상적으로 설정 되었다면 cmd 창에서 다음의 명령어를 실행하였을 때 sqlite3가 실행된다.
>
> ```bash
> $ winpty sqlite3
> ```
>
> 보다 편리한 접근을 위해 alias를 등록한다.
>
> ### 2.3 alias 등록
>
> ```bash
> ~/.bashrc
> 
> vscode로 파일이 열리면, 다음의 코드를 작성한다.
> 
> alias sqlite3="winpty sqlite3"
> ```
>
> ```bash
> $ source ~/.bashrc
> ```
>
> 이제 bash창에 다음과 같이 쓰는 것 만으로 sqlite3 실행이 가능하다.
>
> ```bash
> $ sqlite3
> ```
>
> 
>
> ### 2.4 Database 생성
>
> 이미 db 파일이 있으면 다음과 같이 해당 DB를 콘솔로 연다.
>
> **. 으로 시작하는 모든 명령어는 SQLite에서 제공하는 명령어로, SQL 문법에 속하지 않는다.**
>
> ```sql
> $ sqlite3 database
> ex)
> $ sqlite3 tutorial.sqlite3 # 콘솔로 DB 열기
> sqlite> .databases # DB 목록 확인
> ```
>
> csv 파일을 불러오려면 다음과 같이 한다.
>
> ```sql
> .mode csv
> .import 파일명.csv 테이블명
> ex)
> sqlite> .mode csv
> sqlite> .import users.csv users_user
> ```



## 3. SQLite 문법

> ### 3.1 테이블 만들기
>
> ```sql
> sqlite> CREATE TABLE classmates (
>    ...> id INTEGER PRIMARY KEY,
>    ...> name TEXT
>    ...> );
> ```
>
> ;을 치기 전까지 입력이 이어진다.  Primary key는 표시해준다.
>
> ### 3.2 데이터 조회(SELECT)
>
> ```sql
> sqlite> SELECT * FROM examples;
> 
> 1,"길동","홍",600,"충청도",010-2424-1232
> ```
>
> `*` 표시는 전부라는 뜻. 즉 examples라는 테이블에서 모든 정보를 조회하는 명령어임.
>
> ### 3.3 보기 좋게 조회하기
>
> ```sql
> sqlite> .headers on
> sqlite> .mode column
> sqlite> SELECT * FROM examples;
> id  first_name  last_name  age  country  phone
> --  ----------  ---------  ---  -------  -------------
> 1   길동          홍          600  충청도      010-2424-1232
> ```
>
> ### 3.4 특정한 컬럼만 가져오기
>
> ```sql
> sqlite> SELECT name, age FROM classmates;
> name  age
> ----  ---
> 홍길동   30
> 김철수   23
> 박나래   23
> 이요셉   33
> ```
>
> name, age 컬럼만 classmates 테이블로부터 가져오기
>
> ### 3.5 특정한 테이블에서 원하는 개수(행)만큼 column 가져오기(LIMIT)
>
> ```sql
> sqlite> SELECT name, age FROM classmates LIMIT 2;
> name  age
> ----  ---
> 홍길동   30
> 김철수   23
> ```
>
> ### 3.6 특정 위치에서부터 몇 개 가져오기(LIMIT+OFFSET)
>
> ```sql
> SELECT column1, column2 FROM table LIMIT num OFFSET num2
> ```
>
> 앞에서 num2개만큼 제외하고 num개를 가져온다!
>
> ### 3.7 특정한 값만 가져오기(WHERE)
>
> - `SELECT column1, column2 FROM table WHERE column=value;`
>
> ```sql
> SELECT * FROM classmates WHERE name='tony';
> ```
>
> ### 3.7 컬럼 값을 중복 없이 가져오기(DISTINCT)
>
> ```sql
> --age column의 값들을 중복 없이 가져옴--
> SELECT DISTINCT age FROM classmates;
> ```
>
> ### 3.8 테이블 및 스키마 조회
>
> ```sql
> 1. 테이블 목록 조회
> sqlite> .tables
> classmates  examples // classmates와 examples 테이블이 있음을 확인 가능
> 
> 2. classmates 스키마 확인
> sqlite> .schema classmates
> 
> CREATE TABLE classmates (
> id INTEGER PRIMARY KEY,
> name TEXT
> );
> 
> 3. example 스키마 확인2
> sqlite> .schema examples
> 
> CREATE TABLE IF NOT EXISTS "examples"(
>   "id" TEXT,
>   "first_name" TEXT,
>   "last_name" TEXT,
>   "age" TEXT,
>   "country" TEXT,
>   "phone" TEXT
> );
> ```
>
> ### 3.9 테이블 제거(DROP)
>
> ```sql
> sqlite> DROP TABLE classmates;
> sqlite> .tables // 테이블 제거 확인
> ```
>
> ### 3.10 데이터 추가(INSERT) != CREATE
>
> - INSERT INTO table (column1, column2, ...) VALUES (value1, value2, ...)
> - 모든 열에 데이터를 넣을 때에는 column을 명시할 필요가 없음
>   - INSERT INTO table VALUES (value1, value2, ...)
>
> ```sql
> # ex1
> sqlite> INSERT INTO classmates (name, age)
>    ...> VALUES ('홍길동', 23);
> sqlite> SELECT * FROM classmates;
> 
> name  age  address
> ----  ---  -------
> 홍길동   23
> 
> 
> # ex2
> sqlite> INSERT INTO classmates (name, age, address)
>    ...> VALUES ('김싸피', 25, '대전');
>    
> sqlite> SELECT * FROM classmates;
> name  age  address
> ----  ---  -------
> 홍길동   23
> 김싸피   25   대전
> ```
>
> ### 3.11 rowid
>
> - SQLite는 따로 PRIMARY KEY 속성의 칼럼을 작성하지 않아도 값이 자동으로 증가하는 PK 옵션을 가진 rowid 컬럼을 정의한다.
> - `sqlite> SELECT rowid, * FROM classmates;` 를 입력하면 rowid 칼럼을 볼 수 있다.
>
> 만약 PRIMARY KEY를 함께 설정해주고 나면 INSERT 할 때 넣을 위치를 명시하거나, 모든 컬럼에 값을 넣는 경우에는 VALUES에 id까지 포함한 값을 넣어줘야 작성할 수가 있다.
>
> 그래서 PK 컬럼을 직접 작성하기보다는 rowid로 그냥 사용하는 것이 좋다.
>
> (id 값을 반드시 지정해야 하는 경우에는 지정해줘야 한다.)
>
> ### 3.12 Not Null option
>
> - not null 조건을 설정하면 해당 필드는 null값을 저장할 수 없다.
>
> ```sql
> sqlite> CREATE TABLE classmates (
>    ...> name TEXT NOT NULL,
>    ...> age INT NOT NULL,
>    ...> address TEXT NOT NULL);
> sqlite> INSERT INTO classmates VALUES ('홍길동', 30, '서울'), ('김철수', 23, '대전'), ('박나래', 23, '광주'), ('이요셉', 33, '구미');
> ```
>
> ### 3.13 데이터 삭제
>
> - 중복 불가능한 값인 rowid를 기준으로 삭제!
> - `DELETE FROM table WHERE rowid=?;`
>
> ```sql
> DELETE FROM classmates WHERE rowid=1;
> ```
>
> - 삭제 후 다시 insert를 하면 맨 마지막 rowid에 이어서 만들어짐. (중간 부분이 삭제되어도 채워지지는 않음) => 사용했던 rowid여도 재사용한다.
>
> - 재사용을 방지하려면 AUTOINCREMENT 조건을 쓰면 된다.
>
> ```sql
> CREATE TABLE tests (
> id INTEGER PRIMARY KEY AUTOINCREMENT,
> name TEXT NOT NULL
> );
> 
> sqlite> INSERT INTO tests (name) VALUES ('최철순');
> sqlite> SELECT * FROM tests;
> id  name
> --  ----
> 1   최철순
> sqlite> INSERT INTO tests (name) VALUES ('박순호');
> sqlite> SELECT * FROM tests;
> id  name
> --  ----
> 1   최철순
> 2   박순호
> sqlite> DELETE FROM tests WHERE id=2;
> sqlite> INSERT INTO tests (name) VALUES ('김김김');
> sqlite> SELECT * FROM tests;
> id  name
> --  ----
> 1   최철순
> 3   김김김
> ```
>
> SQLite는 특정 요구사항이 없다면 AUTOINCREMENT 속성을 사용하지 않아야 한다고 말함. 리소스를 더 차지하기 때문인데, **django에서는 데이터 관리가 중요하기 때문에 그냥 사용**한다!
>
> ### 3.14 데이터 수정(UPDATE)
>
> - `UPDATE classmates SET name='홍길동', address='제주도'  WHERE rowid=5;`
> - 조건이 잘못되어 있던가 없는 값을 바꾸려고 하면 그냥 변경 및 출력이 되지 않는다.



## 4. 간단 실습

```sql
-- csv 파일을 사용할 것임 --
sqlite> .mode csv
-- users.csv import하기(table 이름은 users, 모두 TEXT 타입으로 들어감) --
-- TEXT 타입이지만 SUM, AVG, > < >= 등의 연산은 가능함 --
-- 그러나 정렬은 숫자 순서대로 제대로 되지 않는다. --
sqlite> .import users.csv users

-- 모두 출력 --
sqlite> SELECT * FROM users;

-- users에서 age가 30이상인 사람만 가져오기 --
sqlite> SELECT * FROM users WHERE age >= 30;

-- users에서 age가 30이상인 사람의 이름만 가져오기 --
sqlite> SELECT first_name, last_name FROM users WHERE age >= 30;

-- users에서 age가 30이상이고 성이 김인 사람의 성과 나이만 가져오기 --
sqlite> SELECT age, last_name FROM users WHERE age>=30 AND last_name="김";

-- users에서 age가 30이상이고 성이 김인 사람 세 명의 성과 나이만 가져오기 --
sqlite> SELECT age, last_name FROM users WHERE age>=30 AND last_name="김" LIMIT 3;

-- users 테이블의 레코드 총 개수는? --
sqlite> SELECT COUNT(*) FROM users; -- *자리에 아무거나 들어가도 지금은 다 같다. --

-- 30살 이상인 사람들의 평균나이는? --
sqlite> SELECT AVG(age) FROM users WHERE age>=30;

-- users에서 계좌 잔액(balance)이 가장 높은 사람과 액수는? --
sqlite> SELECT first_name, MAX(balance) FROM users;

-- users에서 나이가 30이상인 사람의 계좌 잔액(balance)의 평균 액수는? --
sqlite> SELECT AVG(balance) FROM users WHERE age>=30;
```

COUNT, AVG, MAX...



## 5. 추가 기능

> ### 5.1 LIKE(wild cards)
>
> `_`: 반드시 이 자리에 한 개의 문자가 존재해야 한다.
>
> `%`: 이 자리에 문자열이 있을 수도, 없을 수도 있다.
>
> `2%`: 2로 시작하는 값
>
> `%2`: 2로 끝나는 값
>
> `%2%`: 2가 들어가는 값
>
> `_2`%: 아무값이나 들어가고 두번째가 2로 시작하는 값
>
> `1_ _ _`: 1로 시작하고 4자리인 값
>
> `2_%_%/2_ _%`: 2로 시작하고 적어도 3 자리인 값
>
> ```sql
> -- users에서 20대인 사람은? (age가 현재 TEXT 타입)--
> sqlite> SELECT * FROM users WHERE age LIKE '2_';
> 
> -- users에서 지역번호가 02인 사람은? --
> sqlite> SELECT * FROM users WHERE phone LIKE '02-%';
> 
> -- users에서 이름이 '준'으로 끝나는 사람만? --
> sqlite> SELECT * FROM users WHERE first_name LIKE '%준'
> 
> -- users에서 중간 번호가 5114인 사람만? --
> sqlite> SELECT * FROM users WHERE phone LIKE '%-5114-%'
> ```
>
> 
>
> ### 5.2 ORDER
>
> - `SELECT columns FROM table ORDER BY column1, column2 ASC|DESC`
>
> ```sql
> -- 글자형 정렬 주의! --
> sqlite> SELECT * FROM users ORDER BY balance DESC; -- 내림차순 정렬이 되지 않음 --
> 
> -- default값 = ASC. ASC는 안써도 됨! --
> sqlite> SELECT * FROM users ORDER BY age ASC;
> sqlite> SELECT * FROM users ORDER BY age LIMIT 10;
> 
> -- users에서 나이순, 성 순으로 오름차순 정렬하여 상위 10개만 뽑아보면? --
> sqlite> SELECT * FROM users ORDER BY age, last_name LIMIT 10;
> 
> -- users에서 계좌잔액순으로 내림차순 정렬하여 해당하는 사람의 성과 이름을 10개만 뽑아보면 --
> sqlite> SELECT last_name, first_name FROM users ORDER BY balance DESC LIMIT 10;
> ```
>
> ### 5.3 GROUP BY
>
> 같은 것들끼리 묶어서 보여주기
>
> 데이터 요약시에 사용
>
> 잘 모르겠으니 공부하자.
>
> ```sql
> --users에서 각 성(last_name)씨가 몇 명씩 있는지 조회하시오.--
> sqlite> SELECT last_name, COUNT(*) FROM users GROUP BY last_name;
> ```
>
> ### 5.4 ALTER
>
> - 테이블명 변경
> - 새로운 컬럼 추가
>
> ```sql
> # 테이블명 변경
> ALTER TABLE 기존테이블명
> RENAME TO 새로운테이블명;
> 
> # 새로운 컬럼 추가(INSERT는 행을 추가)
> ALTER TABLE 테이블명
> ADD COLUMN 컬럼명 datatype 제한조건;
> 
> 
> # 예시!
> CREATE TABLE articles (
> title TEXT NOT NULL,
> content TEXT NOT NULL
> );
> 
> INSERT INTO articles VALUES('제목', '내용'), ('제목2', '내용2');
> 
> -- articles에서 news로 테이블의 이름 바꾸기 --
> ALTER TABLE articles RENAME TO news;
> 
> -- NOT NULL 조건으로 컬럼 추가해보기 --
> ALTER TABLE news ADD COLUMN created_at TEXT NOT NULL;
> --Error: Cannot add a NOT NULL column with default value NULL--
> 새로 추가되는 컬럼 자리에 아무 값도 없는데 NOT NULL 조건을 쓰면 말이 안되므로 오류가 난다.
> NOT NULL을 지우거나 DEFAULT 값을 넣으면 잘 추가된다.
> 
> -- NULL인 column 추가 --
> ALTER TABLE news ADD COLUMN created_at TEXT;
> 
> -- 디폴트 --
> ALTER TABLE news ADD COLUMN created_at TEXT DEFAULT '2021-03-25';
> ```





SQL 문법을 알아보았으니 이제 sqlite3를 활용하는 방법을 알아보자.

# 2. sqlite3 활용 & ORM

## 2.1 DB 불러오기, 데이터 import

db가 없을 때, migrate후 db.sqlite3에 접근하여 csv 파일을 import 해준다.

```bash
$ ls
db.sqlite3 manage.py ...
$ sqlite3 db.sqlite3

진행 순서(db 지우고 다시하는 법)
rm -rf db.sqlite3
python manage.py migrate
sqlite3 db.sqlite3
.tables # 테이블 확인
.mode csv # csv 파일을 이용할 것
.import users.csv users_user # migrate로 만들어진 users_user테이블에 csv파일 데이터 넣기
```

users_user라는 테이블에 users.csv의 정보를 넣는 과정이다.

데이터가 잘 들어갔는지 확인하려면, database를 직접 확인하거나, **sqlite3를 이용하면 다음과 같이 할 수 있다.**

```bash
sqlite > SELECT COUNT(*) FROM users_user;
100
```

**스키마 확인**

```bash
sqlite > .schema users_user

CREATE TABLE IF NOT EXISTS "users_user" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "first_name" varchar(10) NOT NULL, "last_name" varchar(10) NOT NULL, "age" integer NOT NULL, "country" varchar(10) NOT NULL, "phone" varchar(15) NOT NULL, "balance" integer NOT NULL);
```



## 2.2 ORM 과 sql 비교

### 1. 모든 user 레코드 조회

```sql
# orm
User.objects.all()

-- sql
SELECT * FROM users_user
```

### 2. user 레코드 생성

```sql
# orm
User.objects.create(first_name='길동', last_name='홍', age=600, country='제주도', phone='010-1234-5678', balance=1000000000)

-- sql
주의: CREATE TABLE!! 테이블 만들때만 CREATE
INSERT INTO users_user VALUES (102, '길동', '홍', 600, '제주도', '010-1234-5678', balance=1000000000);
```

### 3. 특정 user의 레코드 조회

```sql
# orm
User.objects.get(pk=101)

-- sql
-- 모든 필드(열)를 가져올건데 id가 101번인 사람만! --
SELECT * FROM users_user WHERE id=101;
```

### 4. 특정 user의 레코드 수정

```sql
# orm
user = User.objects.get(pk=101)
user.last_name='김'
user.first_name='철수'
user.save()

-- sql
sqlite> UPDATE users_user
   ...> SET first_name='길동', last_name='홍'
   ...> WHERE id=101;
```

### 5. 특정 user 레코드 삭제

```sql
# orm
user = User.objects.get(pk=101)
user.delete()

-- sql
DELETE FROM users_user WHERE id=101;
```

### 6. count

user의 인원 수 세기!

```sql
# orm
User.objects.count()

-- sql
SELECT COUNT(*) from users_user;
```

### 7. filter(ORM) & WHERE(sql)

나이가 30인 사람의 이름 가져오기!

```sql
# orm
User.objects.filter(age=30).values('first_name')

-- sql
SELECT first_name from users_user WHERE age=30;
```

orm에서 이름만 가져오려면 .values를 활용한다. filter나 WHERE에는 조건을 넣어준다.

### 8. filter(ORM) & WHERE(sql) 조건 사용

`__gte`, `__lte`, `__gt`, `__lt`, `__startswith`, `__endswith`, `contains`

나이가 30살 이상인 사람의 인원 수!

```sql
# orm
User.objects.filter(age__gte=30).count()

-- sql
SELECT COUNT(*) FROM users_user WHERE age>=30;
```



나이가 20살 이하인 사람의 인원 수

```sql
# orm
User.objects.filter(age__lte=20).count()

-- sql
SELECT COUNT(*) FROM users_user WHERE age<=20;
```



나이가 30살 이면서 성이 김씨인 사람의 인원 수

```sql
# orm
# 두 가지 방법
User.objects.filter(age=30, last_name='김').count()
User.objects.filter(age=30).filter(last_name='김').count()

-- sql
SELECT COUNT(*) FROM users_user WHERE age=30 AND last_name='김';
```



나이가 30살 이거나 성이 김씨인 사람의 인원 수

```sql
# orm
User.objects.filter(Q(age=30) | Q(last_name='김'))

-- sql
SELECT COUNT(*) FROM users_user WHERE age=30 OR last_name='김';
```



지역번호가 02인 사람의 인원 수

```sql
# orm
User.objects.filter(phone__startswith='02').count()

-- sql : sql like escape % ..
SELECT COUNT(*) FROM users_user WHERE phone LIKE '02-%';
```



거주지역이 강원도이면서 성이 황씨인 사람의 이름

```sql
# orm
User.objects.filter(country="강원도", last_name="황").values('first_name')

-- sql
SELECT first_name FROM users_user WHERE country='강원도' AND last_name='황';
```

### 9. 정렬, LIMIT, OFFSET

나이가 많은 순으로 10명

```sql
# orm
User.objects.order_by('age')[:10]

-- sql
SELECT * FROM users_user ORDER BY age DESC LIMIT 10;
```



잔액이 적은 순으로 10명

```sql
# orm
User.objects.order_by('balance')[:10]
User.objects.order_by('balance').values('pk')[:10]

-- sql ASC는 생략해도 괜찮음.
SELECT * FROM users_user ORDER BY balance ASC LIMIT 10;
```



잔고는 오름차순, 나이는 내림차순으로 10명

```sql
# orm
User.objects.order_by('balance', '-age')[:10]

-- sql
SELECT * FROM users_user ORDER BY balance ASC, age DESC LIMIT 10;
```



성, 이름 내림차순으로 5번째 있는 사람

```sql
# orm
User.objects.order_by('-last_name', '-first_name')[4]

-- sql 
SELECT * FROM users_user ORDER BY last_name DESC, first_name DESC LIMIT 1 OFFSET 1;
```



### 10. aggregate(ORM) & AVG, MAX, SUM(sql)

전체 평균 나이

```sql
# orm
User.objects.all().aggregate(Avg('age'))
# all을 사용하지 않아도 된다.
User.objects.aggregate(Avg('age'))

-- sql
SELECT AVG(age) FROM users_user;
```



김씨의 평균 나이

```sql
# orm
User.objects.filter(last_name='김').aggregate(Avg('age'))

-- sql
SELECT AVG(age) FROM users_user WHERE last_name='김';
```



강원도에 사는 사람의 평균 계좌  잔고

```sql
# orm
User.objects.filter(country='강원도').aggregate(Avg('balance'))

-- sql
SELECT AVG(balance) FROM users_user WHERE country='강원도';
```



계좌 잔액 중 가장 높은 값

```sql
# orm
User.objects.aggregate(Max('balance'))

-- sql
SELECT MAX(balance) FROM users_user;
```



계좌 잔액 총액

```sql
# orm
User.objects.aggregate(Sum('balance'))

-- sql
SELECT SUM(balance) FROM users_user;
```



### 11. annotate & GROUP BY

- 주석을 달다
- 필드를 하나 만들고 거기에 내용을 채워 넣는 것으로, 컬럼 하나를 추가하는 것과 같다고 함.

1:N 관계에서 사용하기 좋음! 한 영화의 댓글들이 매긴 평점의 평균값을 다른 데이터와 함께 받아오고 싶다면..

```python
from django.db.models import Avg

movie = Movie.objects.annotate(score_avg=Avg('comment__score')).get(pk=movies_pk)
```

score_avg라는 컬럼을 하나 만들어서 comment의 score를 평균낸 값을 넣어준 후 pk에 해당하는 레코드만 반환! 어우 어려워

```sql
# orm
User.objects.values('country').annotate(country_count=Count('country'))

# -- sql
SELECT * FROM users_user GROUP BY country;
SELECT country, COUNT(country) FROM users_user GROUP BY country;
```

country 컬럼만 가져온 후 country_count라는 컬럼을 만들고 country의 수를 세서 넣어준 후 반환!

# 3. Practice

1) user 테이블 전체 데이터 조회하기

```sql
--sql
SELECT * FROM users_user;

# orm
User.objects.all()
```



2) id가 19인 사람의 age를 조회

```sql
--sql
SELECT age FROM users_user WHERE id = 19;

# orm
User.objects.get(pk=19).values('age')
```



3) 모든 사람의 age를 조회

```sql
--sql
SELECT age FROM users_user;

# orm
User.objects.all().values('age')
```



4) age가 40 이하인 사람들의 id와 balance를 조회

```sql
--sql
SELECT id, balance FROM users_user WHERE age <= 40;

# orm
User.objects.filter(age <= 40).values('id', 'balance')
```



5) last_name이 '김'이고 balance가 500 이상인 사람들의 first_name을 조회

```sql
--sql
SELECT first_name FROM users_user
WHERE last_name = '김' AND balance >= 500;

# orm
User.objects.filter(last_name='김', balance >= 500).values('first_name')
```



6) first_name이 '수'로 끝나면서 행정구역이 경기도인 사람들의 balance를 조회

```sql
--sql
SELECT balance FROM users_user
WHERE first_name LIKE '%수' AND country = '경기도';

# orm
User.objects.filter(first_name__endswith='수', country='경기도').values('balance')
```



7) balance가 2000이상이거나 age가 40이하인 사람의 총 인원수 구하기

```sql
--sql
SELECT COUNT(*) FROM users_user
WHERE balance >= 2000 OR age <= 40;

# orm
User.objects.filter(Q(age<=40)|Q(balance>=2000)).count()
```



8) phone 앞자리가 '010'으로 시작하는 사람의 총원 구하기

```sql
--sql
SELECT COUNT(*) FROM users_user
WHERE phone LIKE '010-%';

# orm
User.objects.filter(phone__startswith='010').count()
```



9) 이름이 '김옥자'인 사람의 행정구역을 경기도로 수정

```sql
--sql
UPDATE users_user SET country = '경기도'
WHERE first_name='옥자' AND last_name='김';

# orm
users = User.objects.filter(last_name='김', first_name='옥자')
for user in users:
	user.country = '경기도'
	user.save()
```



10) 이름이 '백진호'인 사람 삭제

```sql
--sql
DELETE FROM users_user
WHERE first_name = '진호' AND last_name = '백';

# orm
users = user.objects.filter(last_name='백', first_name='진호')
for user in users:
	user.delete()
```



11) balance를 기준으로 상위 4명의 first_name, last_name, balance를 조회

```sql
--sql
SELECT first_name, last_name, balance FROM users_user
ORDER BY balance DESC LIMIT 4;

# orm
User.objects.order_by('-balance')[:4].values('first_name', 'last_name', 'balance')
```



12) phone에 '123'을 포함하고 age가 30미만인 정보를 조회

```sql
--sql
SELECT * FROM users_user
WHERE phone LIKE '%123%' AND age < 30;

# orm
User.objects.filter(phone__contains='123', age<30)
```



13) phone이 '010'으로 시작하는 사람들의 행정 구역을 중복 없이 조회

```sql
--sql
SELECT DISTINCT country FROM users_user
WHERE phone LIKE '010%';

# orm
User.objects.filter(phone__startswith('010')).values('country').distinct()
```



14) 모든 인원의 평균 age

```sql
--sql
SELECT AVG(age) FROM users_user;

# orm
User.objects.aggregate(Avg('age'))
```



15) 박씨의 평균 balance

```sql
--sql
SELECT AVG(balance) FROM users_user
WHERE last_name='박';

# orm
User.objects.filter(last_name='박').aggregate(Avg('balance'))
```



16) 경상북도에 사는 사람 중 가장 많은 balance의 액수 구하기

```sql
--sql
SELECT MAX(balance) FROM users_user
WHERE country = '경상북도';

# orm
User.objects.filter(country='경상북도').aggregate(Max('balance'))
```



17) 제주특별자치도에 사는 사람 중 balance가 가장 많은 사람의 first_name 구하기

```sql
--sql
SELECT first_name FROM users_user
WHERE country = '제주특별자치도' ORDER BY balance DESC LIMIT 1;

# orm
User.objects.filter(country='제주특별자치도').aggregate(Max('balance')).values('first_name')

User.objects.filter(country='제주특별자치도').order_by('-balance')[0].values('first_name')
```

