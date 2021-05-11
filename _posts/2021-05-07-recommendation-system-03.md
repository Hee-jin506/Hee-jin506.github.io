---
layout: post
title: "[윤선미의 추천 시스템 입문편] 3. 데이터 읽고 기초 분석하기"
categories:
  - Bigdata/Machine learning/Deep learning
tags:
  - 데이터 분석
  - 추천 시스템

---

# 데이터 읽고 기초 분석하기

### 개봉 연도 데이터 정제하기 (데이터 전처리, Preprocessing)

- 정규표현식을 사용해 개봉연도만 title 문자열에서 뽑아야한다.

  ```python
  Toystory (1995)
  Jumanji (1995)
  Grumpier Old Men (1995)
  ```

- 원하는 문자열 부분을 추출하는 함수는 str.extract()이다. 파라미터 안에는 '([원하는 문자열에 대한 정규식])' 과 같은 형태여야 한다.

  ```python
  movies['year'] = movies['title'].str.extract('(\(\d\d\d\d\))')
  ```

- 그런데 이렇게만 두면, year 컬럼의 정보들들은 모두 괄호로 싸여진 채로 저장되게 된다. 이 괄호를 빼주기 위해 다음과 같이 한번 더 문자열을 추출한다.

  ```python
  movies['year'] = movies['year'].str.extract('(\d\d\d\d)')
  ```

  그럼 year의 컬럼 정보는 다음과 같이 나타난다.
  ![]({{site.url}}/assets/images/109.png)

- 정말로 올바른 데이터만 추출되었는지 다시 한번 더 확인하기 위해 중복된 값을 제외하고 유일한 값만 출력하는 unique() 함수를 호출한다.

  ```python
  movies['year'].unique()
  ```

  출력값은 다음과 같다.

  ```
  array(['1995', '1994', '1996', '1976', '1992', '1967', '1993', '1964',
         '1977', '1965', '1982', '1990', '1991', '1989', '1937', '1940',
         '1969', '1981', '1973', '1970', '1955', '1959', '1968', '1988',
         '1997', '1972', '1943', '1952', '1951', '1957', '1961', '1958',
         '1954', '1934', '1944', '1960', '1963', '1942', '1941', '1953',
         '1939', '1950', '1946', '1945', '1938', '1947', '1935', '1936',
         '1956', '1949', '1932', '1975', '1974', '1971', '1979', '1987',
         '1986', '1980', '1978', '1985', '1966', '1962', '1983', '1984',
         '1948', '1933', '1931', '1922', '1998', '1929', '1930', '1927',
         '1928', '1999', '2000', '1926', '1919', '1921', '1925', '1923',
         '2001', '2002', '2003', '1920', '1915', '1924', '2004', '1916',
         '1917', '2005', '2006', '1902', nan, '1903', '2007', '2008',
         '2009', '2010', '2011', '2012', '2013', '2014', '2015', '2016',
         '2017', '2018', '1908'], dtype=object)
  ```

  

- 이때 출력 값 중에 nan이라는 엉뚱한 값을 확인할 수 있는데, 이 값은 결측값이다. 

### 결측값 핸들링하기

- 결측값이 어떤 entity에 위치하는 지 찾아내려면 isnull()이라는 함수를 사용할 수가 있다. 이 함수를 호출하면 결측값의 존재 여부에 따라 boolean 값을 테이블 형태로 제공한다.

  ```python
  movies['year'].isnull()
  ```

  출력값이 True인 엔트리들이 결측값을 갖는 엔트리들이다. 이 엔트리만을 출력하려면 isnull의 결과 값을 한번 더 movies[]로 감싸준다.

  ```py
  movies[movies['year'].isnull()]
  ```

  이 결과로 다음과 같은 테이블이 나타난다.

  ![]({{site.url}}/assets/images/110.png)

  해당 테이블에서 NaN(Not a Number)은 결측치로, 우리가 추출하려고 했던 문자열의 패턴을 발견하지 못했음을 의미한다.

- 우리가 찾은 결측치를 채울 수 있는 함수는 fillna()이며, 파라미터로 결측치를 채울 값을 넣어주면 된다.

  ```python
  movies['year'] = movies['year'].fillna('2050')
  ```

- 가장 많이 출현한 컬럼 값을 구하려면 value_counts() 함수를 사용하면 된다.

  ```python
movies['year'].value_counts()
  ```
  
  결과로 다음과 같이 가장 많이 출현한 컬럼값순서대로 나타나고 오른쪽에는 출현빈도가 나타난다.

  ![]({{site.url}}/assets/images/111.png)

- 데이터 시각화를 위한 라이브러리는 seaborn을 사용할 수가 있다. 이 라이브러리를 통해서 영화들의 개봉연도를 시각화할 수가 있다. seaborn은  matplotlib이라는 파이썬의 데이터 시각화 라이브러리는 예쁘게 매핑해놓은 것을 말한다. 따라서 matplotlib보다도 훨씬 간단한 코드로 시각화가 가능하다. seaborn 페이지에 들어가보면, 어떤 방식으로 시각화가 가능한지 알 수가 있다. 
  http://seaborn.pydata.org/examples/

  여기서 원하는 시각화된 그림을 찾아 클릭해보면, 해당 방법으로 시각화하기 위한 코드가 밑에 제시되어 있다. 파이썬에는 seaborn 이외에도 다양한 시각화 패키지가 있다. seaborn은 그 중에서 초보자들이 사용하기 가장 좋은 라이브러리라고 할 수 있다.

- seaborn에서 그린 figure을 노트북에서 보여주기 위해서는 matplotlib이라는 inline이라는 것을 넣어줘야한다. pyplot은 시각화 사이즈를 조절하기 위해 사용되는 패키지이다.

  ```python
  %matplotlib inline
  
  import seaborn as sns
  import matplotlib.pyplot as plt # seaborn figure 크기 조절을 위해서
  ```

- countplot 시각화를 사용하여 영화 개봉연도를 시각화하려고 한다. 

  ```python
  sns.countplot(data=movies, x='year')
  ```

  그럼 다음과 같은 표가 밑에 나타난다.

  ![]({{site.url}}/assets/images/112.png)

- 그런데 이 표로는 x값이 제대로 표현되지 않아서 무슨 연도에 얼마나 많은 영화가 있는지 알아보기가 힘들다. 따라서 사이즈를 조절해줘야 한다.

  ```python
  plt.figure(figsize=(50, 10))
  sns.countplot(data=movies, x='year')
  ```

  ![]({{site.url}}/assets/images/113.png)

- 이 밖에도 분포도를 보여주는 표는 distplot, 산포도를 보여주는 lmplot 등 다양한 시각화 기능을 사용할 수가 있다.

### 장르 분석

- 장르 컬럼은 '|'을 구분자로 하여 한 엔티티당 한 개 이상의 장르들로 구성되어있다.

  ```
  0       Adventure|Animation|Children|Comedy|Fantasy
  1                        Adventure|Children|Fantasy
  2                                    Comedy|Romance
  3                              Comedy|Drama|Romance
  4                                            Comedy
                             ...                     
  9737                Action|Animation|Comedy|Fantasy
  9738                       Animation|Comedy|Fantasy
  9739                                          Drama
  9740                               Action|Animation
  9741                                         Comedy
  Name: genres, Length: 9742, dtype: object
  ```

- 우리가 가장 먼저 알아볼 것은 몇 종류의 장르가 이 테이블에 존재하느냐이다. 

- 일단 첫번째 엔티티의 genre 값을 샘플로 가져와서 간단히 파싱해보자. '|'을 기준으로 문자열을 분류하기 위해서 split()이라는 함수를 사용할 수가 있다.

  ```python
  sample_genre = movies['genres'][1]
  sample_genre.split("|")
  ```

  그럼 다음과 같이 리스트의 형태로 장르 문자열이 나눠지게 된다.

  ```
  ['Adventure', 'Children', 'Fantasy']
  ```

- 이제 모든 엔티티의 장르 데이터를 위처럼 나눠보자. 이때 람다 문법이 사용된다.

  ```python
  movies['genres'].apply(lambda x: x.split("|"))
  ```

  > **파이썬의 lambda**
  >
  > 런타임에서 생성하여 사용할 수 있는 익명 함수. 함수형 프로그래밍 언어에서 lambda와 정확히 똑같은 것은 아니지만 파이썬에 잘 통합된 매우 강력한 개념이다.
  >
  > lambda는 일회적으로 사용하는 일시적인 함수이며, 생성된 곳에서만 필요하다. 
  >
  > lambda의 정의
  >
  > ```python
  > # lambda [인자리스트]: 표현식
  > g = lambda x: x**2
  > print(g(8)); # 64
  > 
  > f = lamda x,y: x + y
  > print(f(4,4)) # 8
  > ```

  > **apply() 함수**
  >
  > apply() 함수는 DataFrame 컬럼에 복잡한 연산을 vectorize할 수 있게 해주는 함수로 많이 활용된다.
  >
  > apply() 함수는 간단한 경우 lambda() 함수를 적용할 수 있으며, 복잡한 경우 사용자 정의 함수를 적용할 수도 있다.

  이 명령어를 실행시키면 다음과 같은 결과가 나타난다.

  ```python
  0       [Adventure, Animation, Children, Comedy, Fantasy]
  1                          [Adventure, Children, Fantasy]
  2                                       [Comedy, Romance]
  3                                [Comedy, Drama, Romance]
  4                                                [Comedy]
                                ...                        
  9737                 [Action, Animation, Comedy, Fantasy]
  9738                         [Animation, Comedy, Fantasy]
  9739                                              [Drama]
  9740                                  [Action, Animation]
  9741                                             [Comedy]
  Name: genres, Length: 9742, dtype: object
  ```

- 이 결과 값을 리스트로 감싸면, 각각의 컬럼값이 되었던 리스트가 또 다른 리스트의 요소가 되어, 큰 리스트가 형성된다.

  ```python
  genres_list = list(movies['genres'].apply(lambda x: x.split("|")))
  genres_list
  ```

  ```
  [['Adventure', 'Animation', 'Children', 'Comedy', 'Fantasy'],
   ['Adventure', 'Children', 'Fantasy'],
   ['Comedy', 'Romance'],
   ['Comedy', 'Drama', 'Romance'],
   ['Comedy'],
   ['Action', 'Crime', 'Thriller'],
   ['Comedy', 'Romance'],
   ['Adventure', 'Children'],
   ['Action'],
   ['Action', 'Adventure', 'Thriller'],
   ['Comedy', 'Drama', 'Romance'],
   ['Comedy', 'Horror'],
   ['Adventure', 'Animation', 'Children'],
   ['Drama'],
   ['Action', 'Adventure', 'Romance'],
   ['Crime', 'Drama'],
   ['Drama', 'Romance'],
   ['Comedy'],
   ['Comedy'],
   ['Action', 'Comedy', 'Crime', 'Drama', 'Thriller'],
   ['Comedy', 'Crime', 'Thriller'],
   ...]
  ```

- 그런데 우리는 각각의 리스트들을 또 다른 리스트에 넣는 형식이 아닌, 모든 리스트의 요소들을 한 리스트 안에 몰아넣고자 한다. 이것을 flatten이라고 한다. 다음과 같이 코드를 작성한다.

  ```python
  flat_list = []
  for sublist in genres_list:
    for item in sublist:
      flat_list.append(item)
  flat_list
  ```

  그럼 각각의 리스트 요소들을 하나씩 새로운 리스트에 집어넣은 결과가 나타난다.

  ```
  ['Adventure',
   'Animation',
   'Children',
   'Comedy',
   'Fantasy',
   'Adventure',
   'Children',
   'Fantasy',
   'Comedy',
   'Romance',
   'Comedy',
   'Drama',
   'Romance',
   'Comedy',
   'Action',
   'Crime',
   'Thriller',
   'Comedy',
   ...]
  ```

- 그런데 이 리스트에는 중복값이 너무 많다. 이 중복을 지우기 위해서 set() 함수를 사용하여, set이라는 중복값을 허용하지 않는 자료구조로 요소들을 감쌀 수가 있다.

  ```python
  set(flat_list)
  ```

  그럼 다음과 같은 결과가 나타난다.

  ```
  {'(no genres listed)',
   'Action',
   'Adventure',
   'Animation',
   'Children',
   'Comedy',
   'Crime',
   'Documentary',
   'Drama',
   'Fantasy',
   'Film-Noir',
   'Horror',
   'IMAX',
   'Musical',
   'Mystery',
   'Romance',
   'Sci-Fi',
   'Thriller',
   'War',
   'Western'}
  ```

- set이라는 자료구조가 익숙치않다면, 이것을 다시 list 형태로 만들어줄 수도 있다.

  ```python
  genres_unique = list(set(flat_list))
  ```

- 이 리스트의 길이를 알기 위해서는 len() 함수를 사용하면 된다.

  ```python
  len(genres_unique)
  ```

  20 개라는 결과가 나타나는데, 이 중에서 한 개는 (no genres listed)이므로 총 19개임을 알 수 있다.

### 텍스트 데이터를 숫자형으로 변환하기

- 우리는 각 영화들의 장르 컬럼 값에 대하여, 특정 장르가 포함되어있는지 여부를 한눈에 확인할 수 있도록 하고 싶다. 이때, 어떤 문자열이 특정 문자열을 포함하는 지 여부를 확인할 수 있는 in이라는 예약어를 사용하면, 문자열 포함 여부를 True와 False로 나타낼 수가 있다.

  ```python
  movies['genres'].apply(lambda x: 'Adventure' in x)
  ```

  결과는 다음과 같다.

  ```
  0        True
  1        True
  2       False
  3       False
  4       False
          ...  
  9737    False
  9738    False
  9739    False
  9740    False
  9741    False
  Name: genres, Length: 9742, dtype: bool
  ```

  이제 이 각 값을 Adventure라는 컬럼을 새로 만들어 각 컬럼 값으로 만들어주자.

  ```python
  movies['Adventure'] = movies['genres'].apply(lambda x: 'Adventure' in x)
  ```

  그럼 movies는 다음과 같은 테이블이 된다.

  ![]({{site.url}}/assets/images/114.png)

- 그런데 이렇게 모든 장르에 대한 컬럼을 직접 만들어주는 것은 너무 번거로으므로, 이것을 대신 해주는 panda의 명령어를 사용해보자.

  get_dummies()는 구분자를 기준으로 해당 문자열이 있는지에 대한 여부를 각각의 컬럼으로 만들어주는 함수이다.

  ```python
  genres_dummies = movies['genres'].str.get_dummies(sep='|')
  ```

  ![]({{site.url}}/assets/images/115.png)

- 이 테이블을 언제 사용할지 모르니 저장하자. panda 라이브러리에서 어떤 테이블을 csv 파일 형태로 작성하는 함수는 to_csv()이지만, 이번에는 pandas의 DataFrame을 저장하는 함수로는 가장 좋은 to_pickle()을 사용해볼 것이다.

  ```python
  genres_dummies.to_pickle('./data/ml-latest-small/genres.p')
  ```

  > **to_pickle() vs to_csv()**
  >
  > pickle은 Pandas 데이터프레임을 저장하는 직렬화된 방법으로, 기본적으로 데이터 프레임의 정확한 표현(인덱스 등의 정보 포함)을 디스크에 기록하지만, csv로 저장하면, 단순히 쉼표로 구분된 목록으로 저장되므로 데이터 셋에 따라 백업을 로드하면 일부 정보가 손실될 수 있다.

