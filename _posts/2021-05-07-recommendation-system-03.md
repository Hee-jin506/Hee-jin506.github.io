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

  해당 테이블에서 NaN은 결측치로, 우리가 추출하려고 했던 문자열의 패턴을 발견하지 못했음을 의미한다.

- 우리가 찾은 결측치를 채울 수 있는 함수는 fillna()이며, 파라미터로 결측치를 채울 값을 넣어주면 된다.

  ```python
  movies['year'] = movies['year'].fillna('2050')
  ```

- 보통 결측치를 채울 때에는 엉뚱한 값을 채우기보다, 출현 빈도가 높은 숫자나 평균값 같은 값을 넣어 데이터의 왜곡을 최대한 줄이려고 한다. 우리는 테이블에서 가장 많이 출현한 개봉연도를 찾아 결측치에 넣어줄 것이다.

  가장 많이 출현한 컬럼 값을 구하려면 value_counts() 함수를 사용하면 된다.

  ```python
  movies['year'].value_counts()
  ```

  결과로 다음과 같이 가장 많이 출현한 컬럼값순서대로 나타나고 오른쪽에는 출현빈도가 나타난다.

  ![]({{site.url}}/assets/images/111.png)