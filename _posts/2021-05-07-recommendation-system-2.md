---
layout: post
title: "[윤선미의 추천 시스템 입문편] 2. 분석 환경 세팅하기"
categories:
  - Bigdata/Machine learning/Deep learning
tags:
  - 데이터 분석
  - 추천 시스템
---

# 분석 환경 세팅하기

### Google Colaboratory

- Google Drive에서 새 폴더를 만든다.  폴더의 이름은 '추천 시스템 입문편'이다.
- 해당 폴더 밑에 'data'라는 이름의 폴더를 만든다. 이 폴더 밑에는 우리가 사용할 데이터를 모두 저장할 것이다.
- 여러 개의 data set들을 구성할 가능성에 대비해 'data' 폴더 밑에 'ml-latest-small' 폴더를 만든다. 

### Movielens

- 분석할 영화 데이터를 Movielens에서 가져올 것이다.
- https://grouplens.org/datasets/movielens/에서 'recommended for education and development'에 해당하는 데이터 zip파일을 다운받는다.
- 다운받은 파일을 열어서, 다섯개의 파일들을 'ml-latest-small' 폴더 밑에 위치시킨다.

### 주피터 노트북

- '추천 시스템 입문편' 폴더 바로 밑에 colaboratory 파일을 만든다. 파일 이름은 '1. movies.ipynb'이다.
- 이 파일의 background에는 파이썬이 돌아가고 있다.
- 셀을 클릭하면 활성화시킬 수 있으므로, 셀 클릭 후 이 안에 파이썬 코드를 작성하고 실행버튼을 클릭하면 바로 밑에 출력값이 나타난다.
- esc를 누르면 셀이 비활성화되고, 비활성화 상태에서 a를 누르면 해당 셀 위에, b버튼을 누르면 해당 셀 아래에 새로운 셀이 생긴다.
- 이 노트북에서 텍스트는 마크다운 문법을 지원한다.

### 영화 데이터 읽어오기

- 첫 셀에 다음과 같이 적는다.

  ```python
  import pandas as pd
  ```

  colaboratory에 이미 설치되어있는 pandas 라이브러리를 import하고 이 라이브러리 정보를 pd라고 부르겠다는 의미이다.

- pandas는 데이터 분석을 위한 오픈 소스 패키지로, 구체적인 정보는 다음 페이지에서 확인할 수 있다. https://pandas.pydata.org/

- colaboratory에서 설치된 pandas의 버전을 확인하려면 다음 셀에 아래와 같이 작성하고 실행해보면 알 수 있다.

  ```python
  pd.__version__
  # 2021-5-7 기준 1.1.5
  ```

- 구글 드라이브의 data 폴더 밑에 위치한 데이터들에 접근하려면 구글 드라이브를 해당 노트북에 마운트해줘야 한다.
  다음과 같이 작성한다.

  ```py
  movies = pd.read_csv('파일의 위치')
  ```

  그리고 왼쪽 사이드바에서 폴더 모양의 버튼을 누르면 접근할 수 있는 데이터들의 파일 구조가 보인다. 이 때 위 편에 구글 드라이브 모양이 새겨진 세번째 폴더 버튼을 누르면 Google 드라이브가 마운트된다. 

- 다시 왼쪽 사이드바의 파일 구조를 확인하면, drive라는 폴더가 새로 생겼음을 알 수 있다. 이 폴더 밑에 있는 'ml-latest-small' 폴더를 찾아 그 하위의 movies.csv 파일에 대한 컨텍스트 메뉴를 띄우면 경로 복사가 가능하다. 복사된 경로를 read_csv의 파라미터로 넣어준 후 실행하면 데이터들이 밑에 테이블 형태로 뜨게 된다.
   ![]({{site.url}}/assets/images/108.png)

- 이때 colboratory에서 임의로 붙여준 index가 아니라 movieId 컬럼을 인덱스로 사용하고 싶다면, 두번째 파라미터로 다음과 같이 작성한다.

  ```py
  movies = pd.read_csv('파일의 위치', index_col='movieId')
  ```

  

- pandas에서 제공하는 대표적인 기능,

  ```python
  # 테이블의 entity, column 개수 확인
  movies.shape
  # (9742, 2)
  
  # 테이블의 상위 다섯개 entity 보여줌
  movies.head()
  # entity 개수를 파라미터로 지정 가능
  movies.head(10)
  
  # 테이블의 하위 다섯개 entity 보여줌
  movies.tail()
  # entity 개수를 파라미터로 지정 가능
  movies.tail(10)
  
  # 테이블에서 다섯개의 entity를 임의로 뽑음
  movies.sample()
  # entity 개수를 파라미터로 지정 가능
  movies.sample(10)
  
  # 테이블의 column 정보를 가져옴
  movies.columns
  # Index(['movieId', 'title', 'genres'], dtype='object')
  ```

  - 데이터를 저장하려면 다음과 같은 함수를 사용한다. 다음 예는 데이터를 csv 파일 형태로 저장하는 함수이다. 

    ```python
    movies.to_csv('[저장할 위치의 상위 폴더]/[파일 이름].csv')
    ```

    이 함수를 실행하고 왼쪽 사이드 바의 refresh 버튼을 누르면 해당 위치에 파일이 새로 생긴 것을 확인할 수 있다.