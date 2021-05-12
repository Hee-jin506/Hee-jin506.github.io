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

### 장르 간 관계 찾기와 시각화

- 우리는 방금 만든 표를 갖고 장르간의 상관 계수를 만들어볼 수가 있다. 

  - 숫자가 0에 가까울수록 상관관계가 없다.
  - 숫자가 1에 가까울수록 두 장르가 자주 같이 출현
  - 숫자가 -1에 가까울수록 두 장르가 아주 드물게 출현, 겹치는 영역이 없음

  이 상관 계수를 나타내는 표를 만드는 함수는 corr() 함수이다.

  ```python
  genres_dummies.corr()
  ```

  ![]({{site.url}}/assets/images/116.png)

- 그리고 이것을 가장 잘 시각화할 수 있는 예시 중 하나는 seaborn의 heatmap이다. 크기는 (30, 15)로 하고, 상관관계를 숫자로 표시하기 위해서 annot를 True로 설정해주면 된다.

  ```python
  plt.figure(figsize=(30, 15))
  sns.heatmap(genres_dummies.corr(), annot=True) 
  ```

  ![]({{site.url}}/assets/images/117.png)

### 기초 통계

- 일단, 평점에 대한 정보를 살펴보기 위해서, ml-latest-small 폴더 밑에 ratings.csv 파일을 읽어와보자.

  ```python
  ratings = pd.read_csv('/content/drive/MyDrive/추천 시스템 입문편/data/ml-latest-small/ratings.csv')
  ```

  ![]({{site.url}}/assets/images/118.png) 

  여기서 movieId는 영화정보의 movieId와 정확히 일치하고, rating는 평점이며, timestamp는 평점을 기록한 시각이 된다.

- row와 column의 개수를 알고싶다면 shape이라는 속성을 사용할 수 있다.

  ```python
  ratings.shape
  ```

- userId가 몇번까지 있는지 확인하기 위해서는 unique() 함수를 사용하면 되고, 총 몇명이 있는 지를 알기 위해서는 len() 함수로 이를 감싸면 된다.

  ```python
  len(ratings['userId'].unique()) 
  ```

  총 610명의 유저 데이터가 존재한다.

- 마찬가지로 movieId의 개수도 같은 방법으로 알아낼 수 있다.

  ```python
  len(ratings['movieId'].unique()) 
  ```

  총 9724개의 영화 데이터가 존재한다. 이것은 movies 테이블에서의 영화 개수와도 같은 숫자이다. 즉, movies라는 테이블을 먼저 만든 것이 아니라, ratings 테이블에 출현했던 영화 정보를 정리한 것이 movies 테이블이라는 추측을 할 수가 있다.

- 각 컬럼값들의 평균을 알기 위한 함수는 mean(), 최솟값은 min(), 최댓값은 max()이지만, 이 모든 정보를 동시에 띄울 수 있는 함수는 describe() 함수이다.

  ```python
  ratings['rating'].describe()
  ```

  ```
  count    100836.000000
  mean          3.501557
  std           1.042529
  min           0.500000
  25%           3.000000
  50%           3.500000
  75%           4.000000
  max           5.000000
  Name: rating, dtype: float64
  ```

- 이것을 시각화해보려면, seaborn 라이브러리 사용없이 pandas에서 제공해주는 hist()라는 함수를 사용해줘도 간단한 그래프가 나타난다.

  단, pandas에서도 시각화된 그래프를 화면에 띄우기 위해서는 matplotlib inline은 필요하다. 

  ```python
  %matplotlib inline
  ratings['rating'].hist()
  ```

  ![]({{site.url}}/assets/images/119.png)

### 평점의 분포

- 사람들은 평균적으로 몇 개의 영화에 대해서 rating을 남겼는지 알기 위해서 groupby() 함수를 사용하여 표를 userId 별로 자르고, 각각의 userId에 대하여 movieId 데이터 개수를 세기 위해 count() 함수를 사용한다.

  ```python
  users = ratings.groupby('userId')['movieId'].count()
  ```

  ```
  userId
  1       232
  2        29
  3        39
  4       216
  5        44
         ... 
  606    1115
  607     187
  608     831
  609      37
  610    1302
  Name: movieId, Length: 610, dtype: int64
  ```

- 또 다시, 이 만들어진 표에 대해서 describe() 함수를 호출하면, 다음과 같은 결과가 나타난다.

  ```python
  users.describe()
  ```

  ```
  count     610.000000
  mean      165.304918
  std       269.480584
  min        20.000000
  25%        35.000000
  50%        70.500000
  75%       168.000000
  max      2698.000000
  Name: movieId, dtype: float64
  ```

  user는 총 610명이고, 평균적으로 한명당 165개의 영화에 대해 평점을 매겼으며, 최댓값은 2598개, 최솟값은 20개이다. 

- 중앙값은 70개이다. 평균과 중앙값이 다른 이유는 분포가 다소 특이하기 때문인데, 이것을 한번 확인해보자. values라는 속성을 사용하면, 한 명의 user가 가진 데이터, 즉 평점을 매긴 영화 개수를 뽑아 리스트로 나타낼 수 있다.

  ```python
  users.values
  ```

  ```
  array([ 232,   29,   39,  216,   44,  314,  152,   47,   46,  140,   64,
           32,   31,   48,  135,   98,  105,  502,  703,  242,  443,  119,
          121,  110,   26,   21,  135,  570,   81,   34,   50,  102,  156,
           86,   23,   60,   21,   78,  100,  103,  217,  440,  114,   48,
          399,   42,  140,   33,   21,  310,  359,  130,   20,   33,   25,
           46,  476,  112,  107,   22,   39,  366,  271,  517,   34,  345,
           36, 1260,   46,   62,   35,   45,  210,  177,   69,  119,   29,
           61,   64,  167,   26,  227,  118,  293,   34,   70,   21,   56,
          518,   54,  575,   24,   97,   56,  168,   78,   36,   92,   53,
          148,   61,   56,  377,  273,  722,   33,   34,   76,  127,   51,
          646,   65,  150,   31,  112,   87,  165,   22,  215,   22,   58,
          292,   56,   50,  360,   38,   22,   33,  140,   28,   69,  347,
           35,   35,  279,  111,  141,   22,  194,  608,  168,   38,   71,
          128,   23,   32,   20,   48,   58,   26,   59,   63,  179,   34,
           46,  398,   21,   26,   97,  437,   39,   38,   23,   36,   65,
          190,  173,   94,  269,   50,   82,   26,   25,   67,   24,   36, ...])
  ```

- 이것을 seaborn의 distplot으로 그려보면 다음과 같은 그래프가 나타난다.

  ```python
  sns.distplot(users.values)
  ```

  ![]({{site.url}}/assets/images/120.png)

  그래프가 생긴것을 보면, 평범한 정규분포처럼 생긴 것이 아니라, 대부분의 데이터가 0 값에 몰려있지만 2500개까지 넓게 분포되어있는 것을 확인할 수가 있다. 이렇게 생긴 것을 멱함수라고 한다. 즉, 적은 개수의 영화를 보는 사람들이 대부분이고, 소수의 몇명이 아주 많은 영화를 보고, 평점을 매기고 있음을 내포하고 있는 그래프이다. 이러한 분포에서는 평균보다는 중앙값이 더 데이터를 잘 설명할 수가 있다.

- 사람들이 많이 보는 영화를 알아보자. 일단, 특정 영화에 대해서, 평점을 매긴 사람들의 명수를 알기 위해서 userId를 기준으로 자르지 않고, movieId를 기준으로 자른 후, 각각의 userId의 데이터 개수를 세보면 될 것이다.

  ```python
  films = ratings.groupby('movieId')['userId'].count()
  ```

  ```
  movieId
  1         215
  2         110
  3          52
  4           7
  5          49
           ... 
  193581      1
  193583      1
  193585      1
  193587      1
  193609      1
  Name: userId, Length: 9724, dtype: int64
  ```

- 이제 이것에 대해서 describe()를 호출하면, 평균값과 최소, 최댓값 등의 정보를 알아낼 수 있다.

  ```python
  films.describe()
  ```

  ```
  count    9724.000000
  mean       10.369807
  std        22.401005
  min         1.000000
  25%         1.000000
  50%         3.000000
  75%         9.000000
  max       329.000000
  Name: userId, dtype: float64
  ```

  총 9724개의 영화가 있고, 평균적으로는 영화 한 개당 10명의 사람들이 평점을 매겼으며, 최솟값은 1, 최댓값은 329개임을 알 수 있다.

- 분포도를 한번 출력해보면 다음과 같다.

  ```python
  sns.distplot(films.values)
  ```

  ![]({{site.url}}/assets/images/121.png)

  이것도 마찬가지로 멱함수의 형태를 띄고 있으며, 소수의 영화만이 아주 많은 사랑을 받고, 대부분의 영화는 적은 수의 사람들에게만 평점이 매겨지고 있음을 알 수 있다.

### 유저별 평점 패턴 분석

- 이 films 테이블에 대해서 sort_values()를 호출하면, 데이터를 정렬해준다. 함수를 적어주고 Tab키를 누르면, 어떤 인자를 적어주면 되는 지에 대한 안내 메시지를 확인할 수 있는데, 이때 ascending 속성이 디폴트로 True로 되어있음을 알 수 있다. 즉, 이것을 아무 인자 없이 호출하면 오름차순으로 정렬해준다는 뜻이다. 그런데 우리는 가장 평점이 많이 매겨진 영화를 알고 싶기 때문에 ascending 속성을 false로 설정하여, 내림차순으로 정렬해보자.

  ```python
  films.sort_values(ascending=False)
  ```

  ```
  movieId
  356       329
  318       317
  296       307
  593       279
  2571      278
           ... 
  57502       1
  57522       1
  57526       1
  4032        1
  193609      1
  Name: userId, Length: 9724, dtype: int64
  ```

- moveId가 356인 영화는 329명으로부터 평점이 매겨졌다. 이 영화가 무엇인지 알아보기 위해 movies 테이블을 확인해보자. 이때 movieId를 인덱스로 하여, 원하는 영화의 movieId를 갖고 정보를 찾아볼 수 있도록 한다.

  ```python
  movies = pd.read_csv('/content/drive/MyDrive/추천 시스템 입문편/data/ml-latest-small/movies.csv', index_col='movieId')
  ```

