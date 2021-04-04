# QuerySet

전달받은 **모델**의 객체 목록

ModelName.objects.all()로 모델의 객체 목록을 불러오면 볼 수 있는 것이 쿼리셋이다.



**filter, create, save, delete 등의 기능을 활용하여 모델에 데이터를 추가, 삭제, 저장, 조회하는 데에 쿼리셋을 이용할 수 있다.**

```sql
# create
Article.objects.create(name="jason", title="number_one", content="nothing")

# 꼭 create가 아니어도 다음과 같이 할 수 있다.
article=Article()
article.name="jason"
article.title="number_one"
article.content="nothing"
article.save()
```

만약 모델이 name, title, content라는 column을 가진다면, 위와 같이 create을 통해 모델의 객체를 추가할 수가 있고, 이렇게 추가한 객체를 조회하기 위해서는 다음과 같은 방법을 사용할 수 있다.

```sql
# all
Article.objects.all()
=> Article모델의 모든 객체를 불러옴.

# get
Article.objects.get(pk=1)
=> pk값이 1인 Article 모델의 객체를 불러옴

# filter
Article.objects.filter(name="jason")
=> name이 jason인 Article 모델의 객체를 불러온다.

# order_by
Article.objects.order_by('-pk')
=> pk의 역순으로 Article 모델의 객체를(모든 쿼리셋) 불러온다.

# values
Article.objects.values('pk')
=> queryset 타입으로 pk만 출력

# exclude
Article.objects.exclude(name="jason")
=> name이 jason인 객체만 빼고 반환
```

특히 get과 filter에는 조건을 걸 수 있으며, 가능한 조건은 다음과 같다.

- `__lt/__gt`: 보다 작다 / 보다 크다
- `__lte/__gte`: 같거나 보다 작다 / 같거나 보다 크다
  - ex) `Article.objects.filter(id__gt=1)` <= id가 1보다 큰 객체만 불러옴
- `__in=[list]`: 조건이 list 안에 존재하는 자료 검색 
  - ex) `Article.objects.filter(id__in=[2, 3, 5])` <= id가 2 or 3 or 5 인 객체를 불러옴

- `__year / __month / __day`: 해당 년도, 월, 일 자료 검색
  - ex) `Article.objects.filter(created_at__year=2020)` <= 2020년에 만든 객체만 불러옴
- `__isnull`: 해당 열의 값이 null인 자료 검색
  - ex) `Artifcle.objects.filter(name__isnull=True)`
- `__contains / __icontains`: 지정한 문자열을 포함하는 자료 검색
  - ex) `Artifcle.objects.filter(name__contains='jason')` <= 이름이 jason을 포함하는 객체를 불러옴
  - icontains는 대소문자를 구별하지 않음
- `__startswith / __istartswith`: 해당 열의 값이 지정한 문자열로 시작하는 자료 검색
  - ex) `Artifcle.objects.filter(name__startswith='ja')` <= 이름이 ja로 시작하는 객체를 불러옴
- `__endswith / iendswith`: 해당 열의 값이 지정한 문자열로 끝나는 자료 검색
  - ex) `Artifcle.objects.filter(name__endswith='on') <= 이름이 on으로 끝나는 객체를 불러옴
- `__range`: 문자, 숫자, 날짜의 범위를 지정
  - ex) `Artifcle.objects.filter(id__range=(2, 10))`

## QuerySets are lazy

```sql
q = Entry.objects.filter(headline__startswith="What")
q = q.filter(pub_date__lte=datetime.date.today())
q = q.exclude(body_text__icontains="food")
print(q)
```

쿼리셋이 데이터베이스를 건드리는(Hit, access) 시점은 filter나 excluded를 사용하는 시점이 아니다. 위의 코드를 보면 총 세 번 이상 DB에 대한 접근이 일어날 것 같지만, 실상은 print(q), 즉 DB에 데이터를 요청할 때 단 한 번 데이터베이스를 건드리게 된다.

아직 잘 이해는 안되지만, DB를 다룰 때 filter 등을 통해 조건을 거는 경우가 매우 많고, 여러 개를 중첩하기 때문에, 그 때마다 쿼리셋이 데이터베이스를 건드리게 되면 작업 속도가 떨어지는 원인이 될 수 있다고 한다. 그래서 아무리 긴 쿼리문이라도 **평가**가 일어나지 않으면 DB를 건드리지 않는다고 하고, **평가**가 일어나는 시점에 한번에 지금까지의 쿼리문을 실행하여 DB를 조작하는 것 같다.

공식 문서에 따르면, 평가가 일어나는 순간에는 다음의 종류가 있다.

- **반복문(Iteration)**

  - ```sql
    for e in Enrty.objects.all():
    	print(e.headline)
    ```

- 슬라이싱(slicing)

  - ```sql
    queryset = Entry.objects.all()
    slice_queryset = queryset[:6]
    이렇게 하면 처음 여섯 개의 컬럼을 복사해서 반환하고, DB에 대한 평가는 이루어지지 않는 듯..
    
    하지만 step을 넣으면 평가가 이루어진다. (아직 잘 이해가 안됨)
    queryset = Entry.objects.all()
    slice_queryset = queryset[:6:2]
    ```

- Pickling/Caching

  - pickling = serialization, marshalling, flattening, 파이썬 객체 자체를 파일로 저장하는 것

  - 만약 텍스트에서 필요한 부분을 꺼내 쓰는데, 매번 다시 parsing하여 사용해야 한다면? 매우 번거롭다. 필요한 부분을 따로 잘라내어 저장하면 편할 것이다 => 피클!

  - ```python
    import pickle
    my_list = ['a','b','c']
     
    ## Save pickle
    with open("data.pickle","wb") as fw:
        pickle.dump(my_list, fw) # 객체 저장
     
    ## Load pickle
    with open("data.pickle","rb") as fr:
        data = pickle.load(fr) # 객체 불러오기
    print(data)
    #['a', 'b', 'c']
    
    
    출처: https://korbillgates.tistory.com/173 [생물정보학자의 블로그]
    ```

  - **아직 이게 뭐하는건지 잘 모르겠다. 더 알아보자!**

- repr()

  - repr 메서드 실행 시 평가가 이루어짐(쿼리문이 실행됨)

  - **print**가 호출하는 함수..앞서 print에서 평가가 이루어진 이유!

  - ```sql
    queryset = Entry.objects.all()
    queryset.__repr__()
    ```

- len()

  - len 함수 실행 시 평가가 이루어짐(쿼리문이 실행됨)

  - ```sql
    queryset = Entry.objects.all()
    len(queryset)
    ```

- list()

  - list로 쿼리셋의 리스트를 반환하는 경우 평가가 이루어짐(쿼리문이 실행됨)

  - ```sql
    list(Entry.objects.all())
    ```

- bool()

  - **if문** 또는 **bool** 함수로 참, 거짓을 테스트하는 경우 쿼리문이 실행됨

  - ```sql
    bool(Entry.objects.filter(rating__gt=5))
    
    if Entry.objects.filter(headline="Test"):
    	print("There is at least one Entry with the headline Test")
    ```



정리: 쿼리셋이 만들어지는 것 자체는 DB에 요청하지 않는다. 데이터베이스 쿼리를 실행(평가)하는 경우는 주로, Iteration 상황, bool()로 평가를 하는 경우 등이고, 이 경우 **쿼리셋의 내장 캐시에 저장**이 되어 다음 반복 시 다시 DB에 접근하지 않고 **캐시를 이용**한다.

```python
# for() 처음 반복될 때 쿼리를 실행
for e in Entry.objects.all():
    pass
```

```python
# bool()
if Entry.objects.filter(title__'test'):
    pass
```



쿼리셋을 이용할 때, 평가가 반복적으로 이루어지는 것은 피하는 것이 좋다. 예를 들어보자.

```python
# 나쁜 예
print([e.headline for e in Entry.objects.all()]) # 평가
print([e.pub_date for e in Entry.objects.all()]) # 평가
```

```python
# 좋은 예
queryset = Entry.objects.all()
print([p.headline for p in queryset]) # 평가
print([p.pub_date for p in queryset]) # 캐시에서 재사용
```



그런데 항상 캐시에 저장이 되는 것은 아니다. 인덱스에 접근하는 경우..

```python
queryset = Entry.objects.all()
print(queryset[5]) # 평가
print(queryset[5]) # 평가
```

위 상황을 방지하기 위해서는 강제로 전체 쿼리셋을 평가 시켜버리면 된다.

```python
queryset = Entry.objects.all()
[entry for entry in queryset] # 전체 쿼리셋을 평가시킴(캐시에 저장)
print(queryset[5]) # 캐시 사용
print(queryset[5]) # 캐시 사용
```



like 코드에서 지금까지 배운 내용을 알아보자.

```python
like_set = article.like_users.filter(pk=request.user.pk)
if like_set: # 평가
    # 쿼리셋의 전체 결과가 필요하지 않은 상황이지만 ORM은 전체 결과를 가져옴
    article.like_users.remove(request.user)
```

```python
# 개선 1
# exists() 쿼리셋 캐시를 만들지 않으면서 특정 레코드가 있는지 검사
if like_set.exists():
    # DB에서 가져온 레코드가 없으면 메모리를 절약할 수 있다.
    article.like_users.remove(request.user)
```

```python
# 만약 IF 문 안에서 반복문이 있다면?
if like_set: # 평가 후 캐싱
    for user in like_set: # 이미 위에서 평가를 함 => 캐싱된 쿼리셋을 사용
        print(user.username)
```

```python
# 만약 쿼리셋 자체가 너무 크다면?
# iterator()는 전체 레코드의 일부씩만 DB에서 가져오므로 메모리를 절약
# 데이터를 작은 덩어리로 쪼개서 가져오고, 이미 사용한 레코드는 메모리에서 지움
if like_set: # 한번에 캐싱, 부담이 큼
    for user in like_set.iterator(): # 쪼개서 캐싱하고, 쓰고 나면 지움

# 쿼리셋이 너무너무 크면 if 평가에서도 버거움
if like_set.exists():
	for user in like_set.iterator():
```



안일한 최적화의 함정

=> 성능 때문에 구조적인 원칙을 희생하지 마라.

아직은 최적화를 할 때가 아니다! 일단 성능 구현이 우선..



빠른 프로그램이 아니라 좋은 프로그램을 짜려고 노력하자.