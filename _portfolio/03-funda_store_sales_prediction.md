---
title: "Funda sales prediction"
permalink: /portfolio/Funda sales prediction/
date: 2021-05-01
categories:
- 펀다상점매출예측
- 데이콘경진대회
- ARIMA
tags:
- numpy
- pandas
- Scikit-learn
- ML
toc: true
---
## 펀다 상점매출 예측 경진대회

![funda00]({{ site.url }}{{ site.baseurl }}/assets/images/funda0.png)
![funda01]({{ site.url }}{{ site.baseurl }}/assets/images/funda01.png)

데이콘 펀다 전자 상거래 카드 매출 데이터 머신러닝 알고리즘을 통한 ‘3개월 후 상점 매출 예측＇개인 프로젝트를 진행하였습니다.
- 시계열 ARIMA 모델로 3개월 후 상점 매출 예측 진행

---

| Project                               | 데이콘 펀다 전자 상점 매출 예측                                         |
| ------------------------------------------- | ----------------------------------------------------- |
| 배경 및 목적 |(배경) 핀테크 기업인 ‘FUNDA(펀다)’는 상환 기간의 매출을 예측하여 신용 점수가 낮거나 담보를 가지지 못하는 우수 상점들에 금융 기회를 제공하려 하려고 함<br>(목적) 2016년 6월 1일부터 2019년 2월 28일까지의 카드 거래 데이터를 이용해 2019-03-01부터 2019-05-31까지의 각 상점별 3개월 총 매출을 예측
|
| 수행 기간 | 2020.05.01 ~ 2020.06.01(약 1개월)|
| RNR | 개인 / 데이터 전처리, 데이터 탐색, 데이터 시각화, 데이터 마이닝, ML| 
| 활용 데이터 | Dacon funda 상점 매출 데이터|

---
result :
- 계절성 패턴이 보이지 않아 ARIMA 시계열 모델 채택하여 3개월 후 매출 예측하였음

---

- 655만건 데이터를 통해 모델링에 적합한 변수 선택 및 데이터 탐색을 통해 각 변수 별 데이터 분포 파악
- 날짜와 시간을 합친 transacted_datetime 변수를 생성, 환불 변수는 로그변환시 무한대로 나와 제거
- 선형회귀분석시 총 매출량과 유의한 변수 파악이 안되므로 회귀모델 대신 시계열 모델로 분석하기로 함

```python
import pandas as pd
import numpy as np
import warnings
warnings.filterwarnings(action='ignore')
```

### 데이터 불러오기

```python
df = pd.read_csv(r'funda_train.csv', encoding='utf-8-sig')
print(df)
```

             store_id  card_id card_company transacted_date transacted_time  \
    0               0        0            b      2016-06-01           13:13   
    1               0        1            h      2016-06-01           18:12   
    2               0        2            c      2016-06-01           18:52   
    3               0        3            a      2016-06-01           20:22   
    4               0        4            c      2016-06-02           11:06   
    ...           ...      ...          ...             ...             ...   
    6556608      2136  4663855            d      2019-02-28           23:20   
    6556609      2136  4663855            d      2019-02-28           23:24   
    6556610      2136  4663489            a      2019-02-28           23:24   
    6556611      2136  4663856            d      2019-02-28           23:27   
    6556612      2136  4658616            c      2019-02-28           23:54   
    
             installment_term  region type_of_business       amount  
    0                       0     NaN           기타 미용업  1857.142857  
    1                       0     NaN           기타 미용업   857.142857  
    2                       0     NaN           기타 미용업  2000.000000  
    3                       0     NaN           기타 미용업  7857.142857  
    4                       0     NaN           기타 미용업  2000.000000  
    ...                   ...     ...              ...          ...  
    6556608                 0  제주 제주시           기타 주점업 -4500.000000  
    6556609                 0  제주 제주시           기타 주점업  4142.857143  
    6556610                 0  제주 제주시           기타 주점업  4500.000000  
    6556611                 0  제주 제주시           기타 주점업   571.428571  
    6556612                 0  제주 제주시           기타 주점업  5857.142857  
    
    [6556613 rows x 9 columns]
    


```python
pd.set_option('display.max_columns', 15)
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>store_id</th>
      <th>card_id</th>
      <th>card_company</th>
      <th>transacted_date</th>
      <th>transacted_time</th>
      <th>installment_term</th>
      <th>region</th>
      <th>type_of_business</th>
      <th>amount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>b</td>
      <td>2016-06-01</td>
      <td>13:13</td>
      <td>0</td>
      <td>NaN</td>
      <td>기타 미용업</td>
      <td>1857.142857</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>1</td>
      <td>h</td>
      <td>2016-06-01</td>
      <td>18:12</td>
      <td>0</td>
      <td>NaN</td>
      <td>기타 미용업</td>
      <td>857.142857</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>2</td>
      <td>c</td>
      <td>2016-06-01</td>
      <td>18:52</td>
      <td>0</td>
      <td>NaN</td>
      <td>기타 미용업</td>
      <td>2000.000000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>3</td>
      <td>a</td>
      <td>2016-06-01</td>
      <td>20:22</td>
      <td>0</td>
      <td>NaN</td>
      <td>기타 미용업</td>
      <td>7857.142857</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>4</td>
      <td>c</td>
      <td>2016-06-02</td>
      <td>11:06</td>
      <td>0</td>
      <td>NaN</td>
      <td>기타 미용업</td>
      <td>2000.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.info()
#자료형 확인
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 6556613 entries, 0 to 6556612
    Data columns (total 9 columns):
     #   Column            Dtype  
    ---  ------            -----  
     0   store_id          int64  
     1   card_id           int64  
     2   card_company      object 
     3   transacted_date   object 
     4   transacted_time   object 
     5   installment_term  int64  
     6   region            object 
     7   type_of_business  object 
     8   amount            float64
    dtypes: float64(1), int64(3), object(5)
    memory usage: 450.2+ MB
    


```python
df.isnull().sum()
#결측치 확인
```




    store_id                  0
    card_id                   0
    card_company              0
    transacted_date           0
    transacted_time           0
    installment_term          0
    region              2042766
    type_of_business    3952609
    amount                    0
    dtype: int64




```python
df['installment_term'].value_counts()#할부기간
```




    0     6327632
    3      134709
    2       42101
    5       23751
    6       10792
    10       6241
    4        4816
    12       2699
    60       1290
    7         553
    8         413
    24        404
    9         349
    18        332
    15        130
    20        116
    80         83
    11         47
    30         43
    36         36
    16         23
    14         12
    63          8
    83          6
    65          6
    72          4
    19          4
    13          3
    82          2
    35          2
    23          2
    93          2
    22          1
    17          1
    Name: installment_term, dtype: int64




```python
df['store_id'].value_counts()
# 상점은 총 1967갯수
```




    1330    9518
    1196    9471
    1171    9391
    710     9347
    826     9328
            ... 
    1974     429
    1240     426
    795      231
    2119     112
    1063      72
    Name: store_id, Length: 1967, dtype: int64




```python
df['region'].value_counts()
```




    경기 수원시    122029
    충북 청주시    116766
    경남 창원시    107147
    경남 김해시    100673
    경기 평택시     82138
               ...  
    경남 거창군      1143
    서울 관악구      1037
    경남 함안군       878
    경북 영천시       849
    전남 완도군       681
    Name: region, Length: 180, dtype: int64




```python
df['type_of_business'].value_counts()
```




    한식 음식점업                     745905
    두발 미용업                      178475
    의복 소매업                      158234
    기타 주점업                      102413
    치킨 전문점                       89277
                                 ...  
    곡물 및 기타 식량작물 재배업               569
    주방용품 및 가정용 유리, 요업 제품 소매업       551
    배전반 및 전기 자동제어반 제조업             533
    그 외 기타 생활용품 도매업                519
    신선식품 및 단순 가공식품 도매업             231
    Name: type_of_business, Length: 145, dtype: int64




```python
df['transacted_time'].value_counts()
```




    19:55    10980
    19:56    10943
    19:57    10942
    20:00    10917
    19:50    10917
             ...  
    06:23      123
    05:54      121
    06:07      117
    05:59      116
    06:01      109
    Name: transacted_time, Length: 1440, dtype: int64



### 결측치처리1: 0으로 채우기 


```python
df0 = df.fillna(0)
df0
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>store_id</th>
      <th>card_id</th>
      <th>card_company</th>
      <th>transacted_date</th>
      <th>transacted_time</th>
      <th>installment_term</th>
      <th>region</th>
      <th>type_of_business</th>
      <th>amount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>b</td>
      <td>2016-06-01</td>
      <td>13:13</td>
      <td>0</td>
      <td>0</td>
      <td>기타 미용업</td>
      <td>1857.142857</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>1</td>
      <td>h</td>
      <td>2016-06-01</td>
      <td>18:12</td>
      <td>0</td>
      <td>0</td>
      <td>기타 미용업</td>
      <td>857.142857</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>2</td>
      <td>c</td>
      <td>2016-06-01</td>
      <td>18:52</td>
      <td>0</td>
      <td>0</td>
      <td>기타 미용업</td>
      <td>2000.000000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>3</td>
      <td>a</td>
      <td>2016-06-01</td>
      <td>20:22</td>
      <td>0</td>
      <td>0</td>
      <td>기타 미용업</td>
      <td>7857.142857</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>4</td>
      <td>c</td>
      <td>2016-06-02</td>
      <td>11:06</td>
      <td>0</td>
      <td>0</td>
      <td>기타 미용업</td>
      <td>2000.000000</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>6556608</th>
      <td>2136</td>
      <td>4663855</td>
      <td>d</td>
      <td>2019-02-28</td>
      <td>23:20</td>
      <td>0</td>
      <td>제주 제주시</td>
      <td>기타 주점업</td>
      <td>-4500.000000</td>
    </tr>
    <tr>
      <th>6556609</th>
      <td>2136</td>
      <td>4663855</td>
      <td>d</td>
      <td>2019-02-28</td>
      <td>23:24</td>
      <td>0</td>
      <td>제주 제주시</td>
      <td>기타 주점업</td>
      <td>4142.857143</td>
    </tr>
    <tr>
      <th>6556610</th>
      <td>2136</td>
      <td>4663489</td>
      <td>a</td>
      <td>2019-02-28</td>
      <td>23:24</td>
      <td>0</td>
      <td>제주 제주시</td>
      <td>기타 주점업</td>
      <td>4500.000000</td>
    </tr>
    <tr>
      <th>6556611</th>
      <td>2136</td>
      <td>4663856</td>
      <td>d</td>
      <td>2019-02-28</td>
      <td>23:27</td>
      <td>0</td>
      <td>제주 제주시</td>
      <td>기타 주점업</td>
      <td>571.428571</td>
    </tr>
    <tr>
      <th>6556612</th>
      <td>2136</td>
      <td>4658616</td>
      <td>c</td>
      <td>2019-02-28</td>
      <td>23:54</td>
      <td>0</td>
      <td>제주 제주시</td>
      <td>기타 주점업</td>
      <td>5857.142857</td>
    </tr>
  </tbody>
</table>
<p>6556613 rows × 9 columns</p>
</div>




```python
df0.isnull().sum()
```




    store_id            0
    card_id             0
    card_company        0
    transacted_date     0
    transacted_time     0
    installment_term    0
    region              0
    type_of_business    0
    amount              0
    dtype: int64




```python
df0
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>store_id</th>
      <th>card_id</th>
      <th>card_company</th>
      <th>transacted_date</th>
      <th>transacted_time</th>
      <th>installment_term</th>
      <th>region</th>
      <th>type_of_business</th>
      <th>amount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>b</td>
      <td>2016-06-01</td>
      <td>13:13</td>
      <td>0</td>
      <td>0</td>
      <td>기타 미용업</td>
      <td>1857.142857</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>1</td>
      <td>h</td>
      <td>2016-06-01</td>
      <td>18:12</td>
      <td>0</td>
      <td>0</td>
      <td>기타 미용업</td>
      <td>857.142857</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>2</td>
      <td>c</td>
      <td>2016-06-01</td>
      <td>18:52</td>
      <td>0</td>
      <td>0</td>
      <td>기타 미용업</td>
      <td>2000.000000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>3</td>
      <td>a</td>
      <td>2016-06-01</td>
      <td>20:22</td>
      <td>0</td>
      <td>0</td>
      <td>기타 미용업</td>
      <td>7857.142857</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>4</td>
      <td>c</td>
      <td>2016-06-02</td>
      <td>11:06</td>
      <td>0</td>
      <td>0</td>
      <td>기타 미용업</td>
      <td>2000.000000</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>6556608</th>
      <td>2136</td>
      <td>4663855</td>
      <td>d</td>
      <td>2019-02-28</td>
      <td>23:20</td>
      <td>0</td>
      <td>제주 제주시</td>
      <td>기타 주점업</td>
      <td>-4500.000000</td>
    </tr>
    <tr>
      <th>6556609</th>
      <td>2136</td>
      <td>4663855</td>
      <td>d</td>
      <td>2019-02-28</td>
      <td>23:24</td>
      <td>0</td>
      <td>제주 제주시</td>
      <td>기타 주점업</td>
      <td>4142.857143</td>
    </tr>
    <tr>
      <th>6556610</th>
      <td>2136</td>
      <td>4663489</td>
      <td>a</td>
      <td>2019-02-28</td>
      <td>23:24</td>
      <td>0</td>
      <td>제주 제주시</td>
      <td>기타 주점업</td>
      <td>4500.000000</td>
    </tr>
    <tr>
      <th>6556611</th>
      <td>2136</td>
      <td>4663856</td>
      <td>d</td>
      <td>2019-02-28</td>
      <td>23:27</td>
      <td>0</td>
      <td>제주 제주시</td>
      <td>기타 주점업</td>
      <td>571.428571</td>
    </tr>
    <tr>
      <th>6556612</th>
      <td>2136</td>
      <td>4658616</td>
      <td>c</td>
      <td>2019-02-28</td>
      <td>23:54</td>
      <td>0</td>
      <td>제주 제주시</td>
      <td>기타 주점업</td>
      <td>5857.142857</td>
    </tr>
  </tbody>
</table>
<p>6556613 rows × 9 columns</p>
</div>



## EDA


```python
import matplotlib.pyplot as plt
import seaborn as sns
```


```python
#그룹바이
# 상점별 총매출
df0_1 = df0.groupby(['store_id'])['amount'].sum()
df0_1

```




    store_id
    0       2.417447e+07
    1       3.668643e+06
    2       8.843000e+06
    4       2.990096e+07
    5       1.086164e+07
                ...     
    2132    2.263390e+07
    2133    1.104016e+07
    2134    8.147500e+06
    2135    1.814721e+07
    2136    5.090033e+07
    Name: amount, Length: 1967, dtype: float64




```python
df0_1=df0_1.reset_index()
```


```python
df0_1
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>store_id</th>
      <th>amount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>2.417447e+07</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>3.668643e+06</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>8.843000e+06</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>2.990096e+07</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>1.086164e+07</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1962</th>
      <td>2132</td>
      <td>2.263390e+07</td>
    </tr>
    <tr>
      <th>1963</th>
      <td>2133</td>
      <td>1.104016e+07</td>
    </tr>
    <tr>
      <th>1964</th>
      <td>2134</td>
      <td>8.147500e+06</td>
    </tr>
    <tr>
      <th>1965</th>
      <td>2135</td>
      <td>1.814721e+07</td>
    </tr>
    <tr>
      <th>1966</th>
      <td>2136</td>
      <td>5.090033e+07</td>
    </tr>
  </tbody>
</table>
<p>1967 rows × 2 columns</p>
</div>




```python
# 상점별 내림차순 정리
dfSort = df0_1.sort_values(by=['amount'], axis=0, ascending=False)
dfSort
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>store_id</th>
      <th>amount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>501</th>
      <td>538</td>
      <td>1.568540e+09</td>
    </tr>
    <tr>
      <th>112</th>
      <td>119</td>
      <td>5.183212e+08</td>
    </tr>
    <tr>
      <th>1245</th>
      <td>1348</td>
      <td>4.323834e+08</td>
    </tr>
    <tr>
      <th>381</th>
      <td>410</td>
      <td>4.252075e+08</td>
    </tr>
    <tr>
      <th>1283</th>
      <td>1388</td>
      <td>4.106306e+08</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1151</th>
      <td>1240</td>
      <td>1.226900e+06</td>
    </tr>
    <tr>
      <th>1326</th>
      <td>1436</td>
      <td>1.127101e+06</td>
    </tr>
    <tr>
      <th>1793</th>
      <td>1948</td>
      <td>1.016600e+06</td>
    </tr>
    <tr>
      <th>1322</th>
      <td>1432</td>
      <td>1.009622e+06</td>
    </tr>
    <tr>
      <th>742</th>
      <td>795</td>
      <td>5.063271e+05</td>
    </tr>
  </tbody>
</table>
<p>1967 rows × 2 columns</p>
</div>




```python
# 상위 50개 자르기
dfTop50 = dfSort.iloc[:49]
dfTop50
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>store_id</th>
      <th>amount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>501</th>
      <td>538</td>
      <td>1.568540e+09</td>
    </tr>
    <tr>
      <th>112</th>
      <td>119</td>
      <td>5.183212e+08</td>
    </tr>
    <tr>
      <th>1245</th>
      <td>1348</td>
      <td>4.323834e+08</td>
    </tr>
    <tr>
      <th>381</th>
      <td>410</td>
      <td>4.252075e+08</td>
    </tr>
    <tr>
      <th>1283</th>
      <td>1388</td>
      <td>4.106306e+08</td>
    </tr>
    <tr>
      <th>1480</th>
      <td>1601</td>
      <td>3.609477e+08</td>
    </tr>
    <tr>
      <th>1300</th>
      <td>1408</td>
      <td>3.189488e+08</td>
    </tr>
    <tr>
      <th>1615</th>
      <td>1754</td>
      <td>2.569563e+08</td>
    </tr>
    <tr>
      <th>743</th>
      <td>796</td>
      <td>2.555919e+08</td>
    </tr>
    <tr>
      <th>690</th>
      <td>741</td>
      <td>2.539858e+08</td>
    </tr>
    <tr>
      <th>1666</th>
      <td>1807</td>
      <td>2.353777e+08</td>
    </tr>
    <tr>
      <th>1348</th>
      <td>1459</td>
      <td>2.333294e+08</td>
    </tr>
    <tr>
      <th>325</th>
      <td>351</td>
      <td>2.252583e+08</td>
    </tr>
    <tr>
      <th>109</th>
      <td>116</td>
      <td>2.059473e+08</td>
    </tr>
    <tr>
      <th>631</th>
      <td>676</td>
      <td>2.022401e+08</td>
    </tr>
    <tr>
      <th>176</th>
      <td>190</td>
      <td>1.936846e+08</td>
    </tr>
    <tr>
      <th>1849</th>
      <td>2009</td>
      <td>1.919383e+08</td>
    </tr>
    <tr>
      <th>378</th>
      <td>407</td>
      <td>1.897254e+08</td>
    </tr>
    <tr>
      <th>57</th>
      <td>62</td>
      <td>1.880562e+08</td>
    </tr>
    <tr>
      <th>1027</th>
      <td>1104</td>
      <td>1.849641e+08</td>
    </tr>
    <tr>
      <th>1178</th>
      <td>1269</td>
      <td>1.825317e+08</td>
    </tr>
    <tr>
      <th>530</th>
      <td>568</td>
      <td>1.781835e+08</td>
    </tr>
    <tr>
      <th>135</th>
      <td>145</td>
      <td>1.776961e+08</td>
    </tr>
    <tr>
      <th>52</th>
      <td>57</td>
      <td>1.728053e+08</td>
    </tr>
    <tr>
      <th>930</th>
      <td>992</td>
      <td>1.722357e+08</td>
    </tr>
    <tr>
      <th>574</th>
      <td>615</td>
      <td>1.691869e+08</td>
    </tr>
    <tr>
      <th>42</th>
      <td>46</td>
      <td>1.632024e+08</td>
    </tr>
    <tr>
      <th>753</th>
      <td>806</td>
      <td>1.625547e+08</td>
    </tr>
    <tr>
      <th>34</th>
      <td>36</td>
      <td>1.610075e+08</td>
    </tr>
    <tr>
      <th>745</th>
      <td>798</td>
      <td>1.595403e+08</td>
    </tr>
    <tr>
      <th>143</th>
      <td>154</td>
      <td>1.574755e+08</td>
    </tr>
    <tr>
      <th>677</th>
      <td>727</td>
      <td>1.561133e+08</td>
    </tr>
    <tr>
      <th>276</th>
      <td>298</td>
      <td>1.554024e+08</td>
    </tr>
    <tr>
      <th>223</th>
      <td>239</td>
      <td>1.536927e+08</td>
    </tr>
    <tr>
      <th>1540</th>
      <td>1670</td>
      <td>1.526231e+08</td>
    </tr>
    <tr>
      <th>1077</th>
      <td>1161</td>
      <td>1.523340e+08</td>
    </tr>
    <tr>
      <th>969</th>
      <td>1038</td>
      <td>1.508819e+08</td>
    </tr>
    <tr>
      <th>1366</th>
      <td>1477</td>
      <td>1.508449e+08</td>
    </tr>
    <tr>
      <th>839</th>
      <td>895</td>
      <td>1.496762e+08</td>
    </tr>
    <tr>
      <th>1951</th>
      <td>2120</td>
      <td>1.489887e+08</td>
    </tr>
    <tr>
      <th>1694</th>
      <td>1840</td>
      <td>1.486908e+08</td>
    </tr>
    <tr>
      <th>183</th>
      <td>198</td>
      <td>1.470178e+08</td>
    </tr>
    <tr>
      <th>1232</th>
      <td>1335</td>
      <td>1.464655e+08</td>
    </tr>
    <tr>
      <th>740</th>
      <td>793</td>
      <td>1.443766e+08</td>
    </tr>
    <tr>
      <th>607</th>
      <td>651</td>
      <td>1.437911e+08</td>
    </tr>
    <tr>
      <th>1061</th>
      <td>1142</td>
      <td>1.430307e+08</td>
    </tr>
    <tr>
      <th>1000</th>
      <td>1072</td>
      <td>1.402394e+08</td>
    </tr>
    <tr>
      <th>274</th>
      <td>295</td>
      <td>1.359518e+08</td>
    </tr>
    <tr>
      <th>58</th>
      <td>64</td>
      <td>1.354969e+08</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 상점별 상위50 내림차순 막대그래프 시각화

#스타일 서식 지정
plt.style.use('ggplot')

# 데이터프레임의 인덱스를 정수형으로 변경(x축 눈금 라벨 표시)
dfTop50.index = dfTop50.index.map(int)

# 막대 그래프 그리기

dfTop50.plot(kind='bar', figsize=(20,10), width=0.7,
          color=['orange'])

plt.title('상점 별 상위 50개 총 매출순위(내림차순)', size=30)
plt.ylabel('총 매출',size=20 )
plt.xlabel('상점 번호',size=20)

plt.ylim(5000, 1800000000)
plt.legend(loc='best', fontsize=15 )

plt.show()

```


![funda03]({{ site.url }}{{ site.baseurl }}/assets/images/funda3.png)


```python
# 상점별 하위 50개 총 매출 시각화

dfSort2 = df0_1.sort_values(by=['amount'], axis=0)
dfSort2

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>store_id</th>
      <th>amount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>742</th>
      <td>795</td>
      <td>5.063271e+05</td>
    </tr>
    <tr>
      <th>1322</th>
      <td>1432</td>
      <td>1.009622e+06</td>
    </tr>
    <tr>
      <th>1793</th>
      <td>1948</td>
      <td>1.016600e+06</td>
    </tr>
    <tr>
      <th>1326</th>
      <td>1436</td>
      <td>1.127101e+06</td>
    </tr>
    <tr>
      <th>1151</th>
      <td>1240</td>
      <td>1.226900e+06</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1283</th>
      <td>1388</td>
      <td>4.106306e+08</td>
    </tr>
    <tr>
      <th>381</th>
      <td>410</td>
      <td>4.252075e+08</td>
    </tr>
    <tr>
      <th>1245</th>
      <td>1348</td>
      <td>4.323834e+08</td>
    </tr>
    <tr>
      <th>112</th>
      <td>119</td>
      <td>5.183212e+08</td>
    </tr>
    <tr>
      <th>501</th>
      <td>538</td>
      <td>1.568540e+09</td>
    </tr>
  </tbody>
</table>
<p>1967 rows × 2 columns</p>
</div>




```python
# 히위 50개 자르기
dfbottom50 = dfSort2.iloc[:49]
dfbottom50
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>store_id</th>
      <th>amount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>742</th>
      <td>795</td>
      <td>5.063271e+05</td>
    </tr>
    <tr>
      <th>1322</th>
      <td>1432</td>
      <td>1.009622e+06</td>
    </tr>
    <tr>
      <th>1793</th>
      <td>1948</td>
      <td>1.016600e+06</td>
    </tr>
    <tr>
      <th>1326</th>
      <td>1436</td>
      <td>1.127101e+06</td>
    </tr>
    <tr>
      <th>1151</th>
      <td>1240</td>
      <td>1.226900e+06</td>
    </tr>
    <tr>
      <th>383</th>
      <td>412</td>
      <td>1.230595e+06</td>
    </tr>
    <tr>
      <th>755</th>
      <td>808</td>
      <td>1.635599e+06</td>
    </tr>
    <tr>
      <th>1922</th>
      <td>2091</td>
      <td>1.658629e+06</td>
    </tr>
    <tr>
      <th>410</th>
      <td>443</td>
      <td>1.679607e+06</td>
    </tr>
    <tr>
      <th>590</th>
      <td>632</td>
      <td>1.887714e+06</td>
    </tr>
    <tr>
      <th>920</th>
      <td>982</td>
      <td>1.917929e+06</td>
    </tr>
    <tr>
      <th>99</th>
      <td>106</td>
      <td>1.972814e+06</td>
    </tr>
    <tr>
      <th>319</th>
      <td>343</td>
      <td>1.972837e+06</td>
    </tr>
    <tr>
      <th>971</th>
      <td>1040</td>
      <td>2.018116e+06</td>
    </tr>
    <tr>
      <th>1780</th>
      <td>1935</td>
      <td>2.114114e+06</td>
    </tr>
    <tr>
      <th>129</th>
      <td>138</td>
      <td>2.199929e+06</td>
    </tr>
    <tr>
      <th>320</th>
      <td>344</td>
      <td>2.242236e+06</td>
    </tr>
    <tr>
      <th>233</th>
      <td>251</td>
      <td>2.267514e+06</td>
    </tr>
    <tr>
      <th>1117</th>
      <td>1204</td>
      <td>2.295566e+06</td>
    </tr>
    <tr>
      <th>1656</th>
      <td>1797</td>
      <td>2.338029e+06</td>
    </tr>
    <tr>
      <th>1406</th>
      <td>1521</td>
      <td>2.361786e+06</td>
    </tr>
    <tr>
      <th>62</th>
      <td>69</td>
      <td>2.389071e+06</td>
    </tr>
    <tr>
      <th>1802</th>
      <td>1959</td>
      <td>2.395862e+06</td>
    </tr>
    <tr>
      <th>1957</th>
      <td>2126</td>
      <td>2.429629e+06</td>
    </tr>
    <tr>
      <th>45</th>
      <td>49</td>
      <td>2.437453e+06</td>
    </tr>
    <tr>
      <th>1732</th>
      <td>1881</td>
      <td>2.443496e+06</td>
    </tr>
    <tr>
      <th>445</th>
      <td>480</td>
      <td>2.451429e+06</td>
    </tr>
    <tr>
      <th>1269</th>
      <td>1373</td>
      <td>2.495829e+06</td>
    </tr>
    <tr>
      <th>1314</th>
      <td>1423</td>
      <td>2.498371e+06</td>
    </tr>
    <tr>
      <th>23</th>
      <td>25</td>
      <td>2.532857e+06</td>
    </tr>
    <tr>
      <th>478</th>
      <td>515</td>
      <td>2.616043e+06</td>
    </tr>
    <tr>
      <th>959</th>
      <td>1027</td>
      <td>2.638651e+06</td>
    </tr>
    <tr>
      <th>245</th>
      <td>265</td>
      <td>2.665871e+06</td>
    </tr>
    <tr>
      <th>898</th>
      <td>959</td>
      <td>2.670800e+06</td>
    </tr>
    <tr>
      <th>1070</th>
      <td>1152</td>
      <td>2.712586e+06</td>
    </tr>
    <tr>
      <th>843</th>
      <td>899</td>
      <td>2.761306e+06</td>
    </tr>
    <tr>
      <th>1038</th>
      <td>1115</td>
      <td>2.774429e+06</td>
    </tr>
    <tr>
      <th>931</th>
      <td>993</td>
      <td>2.839643e+06</td>
    </tr>
    <tr>
      <th>56</th>
      <td>61</td>
      <td>2.848766e+06</td>
    </tr>
    <tr>
      <th>67</th>
      <td>74</td>
      <td>2.856741e+06</td>
    </tr>
    <tr>
      <th>1961</th>
      <td>2130</td>
      <td>2.874914e+06</td>
    </tr>
    <tr>
      <th>1634</th>
      <td>1774</td>
      <td>2.969586e+06</td>
    </tr>
    <tr>
      <th>659</th>
      <td>706</td>
      <td>2.971571e+06</td>
    </tr>
    <tr>
      <th>1058</th>
      <td>1139</td>
      <td>3.059000e+06</td>
    </tr>
    <tr>
      <th>1529</th>
      <td>1659</td>
      <td>3.067435e+06</td>
    </tr>
    <tr>
      <th>1312</th>
      <td>1421</td>
      <td>3.071143e+06</td>
    </tr>
    <tr>
      <th>1759</th>
      <td>1911</td>
      <td>3.109429e+06</td>
    </tr>
    <tr>
      <th>1938</th>
      <td>2107</td>
      <td>3.123857e+06</td>
    </tr>
    <tr>
      <th>801</th>
      <td>856</td>
      <td>3.172929e+06</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 상점별 하위 50 오름차순 막대그래프 시각화

#스타일 서식 지정
plt.style.use('ggplot')

# 데이터프레임의 인덱스를 정수형으로 변경(x축 눈금 라벨 표시)
dfbottom50.index = dfbottom50.index.map(int)

# 막대 그래프 그리기

dfbottom50.plot(kind='bar', figsize=(20,10), width=0.7,
          color=['green'])

plt.title('상점 별 하위 50개 총 매출순위(오름차순)', size=30)
plt.ylabel('총 매출',size=20 )
plt.xlabel('상점 번호',size=20)

plt.ylim(5000, 3500000)
plt.legend(loc='best', fontsize=15 )

plt.show()
```

![funda03_1]({{ site.url }}{{ site.baseurl }}/assets/images/funda3_1.png)
    

```python
#그룹바이
# 상점별, 카드회사별 매출집계, card_id, card_company, transacted_date, transacted_time, installment_term, region, type_of_business, amount
df1 = df0.groupby(['store_id', 'card_company'])['amount'].sum()
df1
```




    store_id  card_company
    0         a               7.612571e+06
              b               5.550071e+06
              c               4.716829e+06
              e               1.626571e+06
              f               1.366000e+06
                                  ...     
    2136      d               1.663301e+07
              e               2.868786e+06
              f               3.433571e+06
              g               1.516143e+06
              h               6.401429e+05
    Name: amount, Length: 15468, dtype: float64




```python
df1=df1.reset_index()
```


```python
df1
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>store_id</th>
      <th>card_company</th>
      <th>amount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>a</td>
      <td>7.612571e+06</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>b</td>
      <td>5.550071e+06</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>c</td>
      <td>4.716829e+06</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>e</td>
      <td>1.626571e+06</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>f</td>
      <td>1.366000e+06</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>15463</th>
      <td>2136</td>
      <td>d</td>
      <td>1.663301e+07</td>
    </tr>
    <tr>
      <th>15464</th>
      <td>2136</td>
      <td>e</td>
      <td>2.868786e+06</td>
    </tr>
    <tr>
      <th>15465</th>
      <td>2136</td>
      <td>f</td>
      <td>3.433571e+06</td>
    </tr>
    <tr>
      <th>15466</th>
      <td>2136</td>
      <td>g</td>
      <td>1.516143e+06</td>
    </tr>
    <tr>
      <th>15467</th>
      <td>2136</td>
      <td>h</td>
      <td>6.401429e+05</td>
    </tr>
  </tbody>
</table>
<p>15468 rows × 3 columns</p>
</div>




```python
# 상점별 인덱스 
df1.index = df1['store_id']
df1.set_index('store_id', inplace=True)
```


```python
df1
```


```python
# 상점별 총 매출 시각화

df1.plot()
plt.show()

```


```python
df1_1 = df1.pivot('store_id','card_company','amount')
df1_1
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>card_company</th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
      <th>d</th>
      <th>e</th>
      <th>f</th>
      <th>g</th>
      <th>h</th>
    </tr>
    <tr>
      <th>store_id</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>7.612571e+06</td>
      <td>5.550071e+06</td>
      <td>4.716829e+06</td>
      <td>NaN</td>
      <td>1.626571e+06</td>
      <td>1.366000e+06</td>
      <td>1.936286e+06</td>
      <td>1.366143e+06</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.243286e+06</td>
      <td>8.707143e+05</td>
      <td>8.215714e+05</td>
      <td>NaN</td>
      <td>1.624286e+05</td>
      <td>2.459286e+05</td>
      <td>2.408571e+05</td>
      <td>8.385714e+04</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2.818714e+06</td>
      <td>1.636857e+06</td>
      <td>1.759714e+06</td>
      <td>NaN</td>
      <td>5.814286e+05</td>
      <td>1.335571e+06</td>
      <td>6.071429e+05</td>
      <td>1.035714e+05</td>
    </tr>
    <tr>
      <th>4</th>
      <td>7.951957e+06</td>
      <td>6.286607e+06</td>
      <td>6.508314e+06</td>
      <td>1.313936e+06</td>
      <td>2.245950e+06</td>
      <td>2.063793e+06</td>
      <td>2.763257e+06</td>
      <td>7.671429e+05</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2.236829e+06</td>
      <td>2.088157e+06</td>
      <td>2.148543e+06</td>
      <td>1.011386e+06</td>
      <td>1.651714e+06</td>
      <td>9.716429e+05</td>
      <td>4.838571e+05</td>
      <td>2.695143e+05</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2132</th>
      <td>6.858686e+06</td>
      <td>4.096214e+06</td>
      <td>5.000643e+06</td>
      <td>1.120214e+06</td>
      <td>1.297000e+06</td>
      <td>1.851500e+06</td>
      <td>1.949000e+06</td>
      <td>4.606429e+05</td>
    </tr>
    <tr>
      <th>2133</th>
      <td>2.329443e+06</td>
      <td>2.502329e+06</td>
      <td>1.919786e+06</td>
      <td>7.439429e+05</td>
      <td>1.430371e+06</td>
      <td>1.221143e+06</td>
      <td>4.773286e+05</td>
      <td>4.158143e+05</td>
    </tr>
    <tr>
      <th>2134</th>
      <td>2.107071e+06</td>
      <td>1.160214e+06</td>
      <td>9.319286e+05</td>
      <td>2.111071e+06</td>
      <td>6.781429e+05</td>
      <td>7.220714e+05</td>
      <td>3.522857e+05</td>
      <td>8.471429e+04</td>
    </tr>
    <tr>
      <th>2135</th>
      <td>7.476286e+06</td>
      <td>2.233143e+06</td>
      <td>3.230857e+06</td>
      <td>1.078643e+06</td>
      <td>1.301429e+06</td>
      <td>1.330714e+06</td>
      <td>1.068857e+06</td>
      <td>4.272857e+05</td>
    </tr>
    <tr>
      <th>2136</th>
      <td>1.337443e+07</td>
      <td>5.640000e+06</td>
      <td>6.794243e+06</td>
      <td>1.663301e+07</td>
      <td>2.868786e+06</td>
      <td>3.433571e+06</td>
      <td>1.516143e+06</td>
      <td>6.401429e+05</td>
    </tr>
  </tbody>
</table>
<p>1967 rows × 8 columns</p>
</div>




```python
df1_1.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 1967 entries, 0 to 2136
    Data columns (total 8 columns):
     #   Column  Non-Null Count  Dtype  
    ---  ------  --------------  -----  
     0   a       1964 non-null   float64
     1   b       1963 non-null   float64
     2   c       1964 non-null   float64
     3   d       1770 non-null   float64
     4   e       1949 non-null   float64
     5   f       1949 non-null   float64
     6   g       1963 non-null   float64
     7   h       1946 non-null   float64
    dtypes: float64(8)
    memory usage: 138.3 KB
    


```python
# b,d,e,f,g,h결측치 0으로 메우기
df1_1 = df1_1.fillna(0)
df1_1
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>card_company</th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
      <th>d</th>
      <th>e</th>
      <th>f</th>
      <th>g</th>
      <th>h</th>
    </tr>
    <tr>
      <th>store_id</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>7.612571e+06</td>
      <td>5.550071e+06</td>
      <td>4.716829e+06</td>
      <td>0.000000e+00</td>
      <td>1.626571e+06</td>
      <td>1.366000e+06</td>
      <td>1.936286e+06</td>
      <td>1.366143e+06</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.243286e+06</td>
      <td>8.707143e+05</td>
      <td>8.215714e+05</td>
      <td>0.000000e+00</td>
      <td>1.624286e+05</td>
      <td>2.459286e+05</td>
      <td>2.408571e+05</td>
      <td>8.385714e+04</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2.818714e+06</td>
      <td>1.636857e+06</td>
      <td>1.759714e+06</td>
      <td>0.000000e+00</td>
      <td>5.814286e+05</td>
      <td>1.335571e+06</td>
      <td>6.071429e+05</td>
      <td>1.035714e+05</td>
    </tr>
    <tr>
      <th>4</th>
      <td>7.951957e+06</td>
      <td>6.286607e+06</td>
      <td>6.508314e+06</td>
      <td>1.313936e+06</td>
      <td>2.245950e+06</td>
      <td>2.063793e+06</td>
      <td>2.763257e+06</td>
      <td>7.671429e+05</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2.236829e+06</td>
      <td>2.088157e+06</td>
      <td>2.148543e+06</td>
      <td>1.011386e+06</td>
      <td>1.651714e+06</td>
      <td>9.716429e+05</td>
      <td>4.838571e+05</td>
      <td>2.695143e+05</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2132</th>
      <td>6.858686e+06</td>
      <td>4.096214e+06</td>
      <td>5.000643e+06</td>
      <td>1.120214e+06</td>
      <td>1.297000e+06</td>
      <td>1.851500e+06</td>
      <td>1.949000e+06</td>
      <td>4.606429e+05</td>
    </tr>
    <tr>
      <th>2133</th>
      <td>2.329443e+06</td>
      <td>2.502329e+06</td>
      <td>1.919786e+06</td>
      <td>7.439429e+05</td>
      <td>1.430371e+06</td>
      <td>1.221143e+06</td>
      <td>4.773286e+05</td>
      <td>4.158143e+05</td>
    </tr>
    <tr>
      <th>2134</th>
      <td>2.107071e+06</td>
      <td>1.160214e+06</td>
      <td>9.319286e+05</td>
      <td>2.111071e+06</td>
      <td>6.781429e+05</td>
      <td>7.220714e+05</td>
      <td>3.522857e+05</td>
      <td>8.471429e+04</td>
    </tr>
    <tr>
      <th>2135</th>
      <td>7.476286e+06</td>
      <td>2.233143e+06</td>
      <td>3.230857e+06</td>
      <td>1.078643e+06</td>
      <td>1.301429e+06</td>
      <td>1.330714e+06</td>
      <td>1.068857e+06</td>
      <td>4.272857e+05</td>
    </tr>
    <tr>
      <th>2136</th>
      <td>1.337443e+07</td>
      <td>5.640000e+06</td>
      <td>6.794243e+06</td>
      <td>1.663301e+07</td>
      <td>2.868786e+06</td>
      <td>3.433571e+06</td>
      <td>1.516143e+06</td>
      <td>6.401429e+05</td>
    </tr>
  </tbody>
</table>
<p>1967 rows × 8 columns</p>
</div>




```python
df1_1.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 1967 entries, 0 to 2136
    Data columns (total 8 columns):
     #   Column  Non-Null Count  Dtype  
    ---  ------  --------------  -----  
     0   a       1967 non-null   float64
     1   b       1967 non-null   float64
     2   c       1967 non-null   float64
     3   d       1967 non-null   float64
     4   e       1967 non-null   float64
     5   f       1967 non-null   float64
     6   g       1967 non-null   float64
     7   h       1967 non-null   float64
    dtypes: float64(8)
    memory usage: 138.3 KB
    


```python
df1_1['a']
```




    store_id
    0       7.612571e+06
    1       1.243286e+06
    2       2.818714e+06
    4       7.951957e+06
    5       2.236829e+06
                ...     
    2132    6.858686e+06
    2133    2.329443e+06
    2134    2.107071e+06
    2135    7.476286e+06
    2136    1.337443e+07
    Name: a, Length: 1967, dtype: float64




```python
# a,b,c,d,e,f,g 카드회사 기준 매출순위 많은 상점 별 내림차순 정리
df1_1Sort = df1_1.sort_values(by=['a','b','c','d','e','f','g'], axis=0, ascending=False)
df1_1Sort
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>card_company</th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
      <th>d</th>
      <th>e</th>
      <th>f</th>
      <th>g</th>
      <th>h</th>
    </tr>
    <tr>
      <th>store_id</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>538</th>
      <td>4.542472e+08</td>
      <td>2.157272e+08</td>
      <td>4.149227e+08</td>
      <td>2.155058e+07</td>
      <td>1.745392e+08</td>
      <td>5.002352e+07</td>
      <td>2.256929e+08</td>
      <td>1.183687e+07</td>
    </tr>
    <tr>
      <th>410</th>
      <td>1.781840e+08</td>
      <td>5.559257e+07</td>
      <td>5.141529e+07</td>
      <td>2.129100e+07</td>
      <td>3.725637e+07</td>
      <td>2.548814e+07</td>
      <td>4.751726e+07</td>
      <td>8.462857e+06</td>
    </tr>
    <tr>
      <th>1388</th>
      <td>1.466222e+08</td>
      <td>6.060811e+07</td>
      <td>6.120388e+07</td>
      <td>9.833057e+06</td>
      <td>3.433963e+07</td>
      <td>4.053314e+07</td>
      <td>4.549983e+07</td>
      <td>1.199069e+07</td>
    </tr>
    <tr>
      <th>119</th>
      <td>1.023371e+08</td>
      <td>9.383636e+07</td>
      <td>9.135754e+07</td>
      <td>2.498700e+07</td>
      <td>7.377343e+07</td>
      <td>6.116500e+07</td>
      <td>3.116179e+07</td>
      <td>3.970300e+07</td>
    </tr>
    <tr>
      <th>1348</th>
      <td>9.583371e+07</td>
      <td>4.727857e+07</td>
      <td>4.525871e+07</td>
      <td>4.648171e+07</td>
      <td>7.827971e+07</td>
      <td>4.331543e+07</td>
      <td>2.173286e+07</td>
      <td>5.420271e+07</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>594</th>
      <td>1.159143e+05</td>
      <td>8.111429e+04</td>
      <td>1.484429e+05</td>
      <td>5.138571e+04</td>
      <td>5.887140e+06</td>
      <td>1.153000e+05</td>
      <td>1.553143e+05</td>
      <td>5.342857e+03</td>
    </tr>
    <tr>
      <th>795</th>
      <td>7.368857e+04</td>
      <td>4.607143e+04</td>
      <td>1.182000e+05</td>
      <td>6.550000e+04</td>
      <td>5.057143e+04</td>
      <td>4.524286e+04</td>
      <td>7.341000e+04</td>
      <td>3.364286e+04</td>
    </tr>
    <tr>
      <th>1285</th>
      <td>0.000000e+00</td>
      <td>8.090357e+06</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>1901</th>
      <td>0.000000e+00</td>
      <td>3.189400e+06</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
    </tr>
    <tr>
      <th>166</th>
      <td>0.000000e+00</td>
      <td>7.824186e+05</td>
      <td>6.362857e+05</td>
      <td>5.629714e+05</td>
      <td>2.816144e+05</td>
      <td>6.841643e+05</td>
      <td>3.678857e+05</td>
      <td>1.844714e+05</td>
    </tr>
  </tbody>
</table>
<p>1967 rows × 8 columns</p>
</div>




```python
# 상위 10개
df1_1Top = df1_1Sort[:9]
df1_1Top
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>card_company</th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
      <th>d</th>
      <th>e</th>
      <th>f</th>
      <th>g</th>
      <th>h</th>
    </tr>
    <tr>
      <th>store_id</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>538</th>
      <td>4.542472e+08</td>
      <td>2.157272e+08</td>
      <td>4.149227e+08</td>
      <td>2.155058e+07</td>
      <td>1.745392e+08</td>
      <td>5.002352e+07</td>
      <td>2.256929e+08</td>
      <td>1.183687e+07</td>
    </tr>
    <tr>
      <th>410</th>
      <td>1.781840e+08</td>
      <td>5.559257e+07</td>
      <td>5.141529e+07</td>
      <td>2.129100e+07</td>
      <td>3.725637e+07</td>
      <td>2.548814e+07</td>
      <td>4.751726e+07</td>
      <td>8.462857e+06</td>
    </tr>
    <tr>
      <th>1388</th>
      <td>1.466222e+08</td>
      <td>6.060811e+07</td>
      <td>6.120388e+07</td>
      <td>9.833057e+06</td>
      <td>3.433963e+07</td>
      <td>4.053314e+07</td>
      <td>4.549983e+07</td>
      <td>1.199069e+07</td>
    </tr>
    <tr>
      <th>119</th>
      <td>1.023371e+08</td>
      <td>9.383636e+07</td>
      <td>9.135754e+07</td>
      <td>2.498700e+07</td>
      <td>7.377343e+07</td>
      <td>6.116500e+07</td>
      <td>3.116179e+07</td>
      <td>3.970300e+07</td>
    </tr>
    <tr>
      <th>1348</th>
      <td>9.583371e+07</td>
      <td>4.727857e+07</td>
      <td>4.525871e+07</td>
      <td>4.648171e+07</td>
      <td>7.827971e+07</td>
      <td>4.331543e+07</td>
      <td>2.173286e+07</td>
      <td>5.420271e+07</td>
    </tr>
    <tr>
      <th>1408</th>
      <td>9.520864e+07</td>
      <td>4.700886e+07</td>
      <td>8.389286e+06</td>
      <td>1.085171e+07</td>
      <td>6.682900e+07</td>
      <td>2.329071e+07</td>
      <td>4.647514e+07</td>
      <td>2.089543e+07</td>
    </tr>
    <tr>
      <th>1754</th>
      <td>7.472263e+07</td>
      <td>3.973693e+07</td>
      <td>3.226043e+07</td>
      <td>0.000000e+00</td>
      <td>4.882294e+07</td>
      <td>2.001629e+07</td>
      <td>2.146514e+07</td>
      <td>1.993193e+07</td>
    </tr>
    <tr>
      <th>796</th>
      <td>6.359436e+07</td>
      <td>3.735579e+07</td>
      <td>4.261744e+07</td>
      <td>3.305319e+07</td>
      <td>2.901319e+07</td>
      <td>2.917647e+07</td>
      <td>1.317273e+07</td>
      <td>7.608700e+06</td>
    </tr>
    <tr>
      <th>1269</th>
      <td>5.926286e+07</td>
      <td>2.966200e+07</td>
      <td>1.930614e+07</td>
      <td>2.523914e+07</td>
      <td>9.983429e+06</td>
      <td>2.257986e+07</td>
      <td>8.896000e+06</td>
      <td>7.602286e+06</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 카드회사별 막대그래프

#스타일 서식 지정
plt.style.use('ggplot')

# 데이터프레임의 인덱스를 정수형으로 변경(x축 눈금 라벨 표시)
df1_1Top.index = df1_1Top.index.map(int)

# 막대 그래프 그리기

df1_1Top.plot(kind='bar', figsize=(20,10), width=0.7,
          color=['grey','blue','orange','red','green','pink','skyblue','brown'])

plt.title('카드회사별 상위 매출 10개 점포 순위(내림차순)', size=30)
plt.ylabel('총 매출',size=20 )
plt.xlabel('카드회사',size=20)

plt.ylim(5000, 500000000)
plt.legend(loc='best', fontsize=15 )

plt.show()
```


![funda04]({{ site.url }}{{ site.baseurl }}/assets/images/funda4.png)


```python
# 카드회사별 박스플롯

# 한글폰트 오류 해결

from matplotlib import font_manager, rc
font_path = "./malgun.ttf"#폰트파일 위치
font_name = font_manager.FontProperties(fname=font_path).get_name()
rc('font', family=font_name)

plt.style.use('seaborn-poster')#스타일 서식 지정
plt.rcParams['axes.unicode_minus']=False #마이너스 부호 출력 지정

# 그래프 객체 생성
fig = plt.figure(figsize=(20, 10))
ax1 = fig.add_subplot(1, 2, 1)

# axe 객체에 boxplot 메소드로 그래프 출력 
data = df1_1

green_diamond = dict(markerfacecolor='r', marker='s')
plt.boxplot(data, flierprops=green_diamond)

ax1.set_title('상점 별 카드회사별 총 매출분포(수직 박스 플롯)')
plt.show()
```



![funda02]({{ site.url }}{{ site.baseurl }}/assets/images/funda2.png)

    


a(=1)회사가 이상치제일 많고 중앙값 최대값 총 매출에 가장 많이 기여한다 이상치는 스케일러가 필요하다




```python
# 그룹바이
# 상점별 installment_term 할부개월수 별 amount

df2= df0.groupby(['store_id', 'installment_term'])['amount'].sum()
df2
```




    store_id  installment_term
    0         0                   2.163076e+07
              2                   1.123857e+06
              3                   1.204000e+06
              4                   7.714286e+04
              5                   4.142857e+04
                                      ...     
    2135      0                   1.811407e+07
              2                   3.314286e+04
    2136      0                   5.079504e+07
              2                   2.542857e+04
              3                   7.985714e+04
    Name: amount, Length: 10924, dtype: float64




```python
df2=df2.reset_index()
```


```python
df2
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>store_id</th>
      <th>installment_term</th>
      <th>amount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>2.163076e+07</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>2</td>
      <td>1.123857e+06</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>3</td>
      <td>1.204000e+06</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>4</td>
      <td>7.714286e+04</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>5</td>
      <td>4.142857e+04</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>10919</th>
      <td>2135</td>
      <td>0</td>
      <td>1.811407e+07</td>
    </tr>
    <tr>
      <th>10920</th>
      <td>2135</td>
      <td>2</td>
      <td>3.314286e+04</td>
    </tr>
    <tr>
      <th>10921</th>
      <td>2136</td>
      <td>0</td>
      <td>5.079504e+07</td>
    </tr>
    <tr>
      <th>10922</th>
      <td>2136</td>
      <td>2</td>
      <td>2.542857e+04</td>
    </tr>
    <tr>
      <th>10923</th>
      <td>2136</td>
      <td>3</td>
      <td>7.985714e+04</td>
    </tr>
  </tbody>
</table>
<p>10924 rows × 3 columns</p>
</div>




```python
df2_1 = df2.pivot('store_id','installment_term','amount')
df2_1
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>installment_term</th>
      <th>0</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>...</th>
      <th>63</th>
      <th>65</th>
      <th>72</th>
      <th>80</th>
      <th>82</th>
      <th>83</th>
      <th>93</th>
    </tr>
    <tr>
      <th>store_id</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2.163076e+07</td>
      <td>1.123857e+06</td>
      <td>1.204000e+06</td>
      <td>77142.857143</td>
      <td>4.142857e+04</td>
      <td>75714.285714</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3.668643e+06</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7.511143e+06</td>
      <td>4.525714e+05</td>
      <td>7.735714e+05</td>
      <td>NaN</td>
      <td>1.857143e+04</td>
      <td>87142.857143</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2.984553e+07</td>
      <td>2.271429e+04</td>
      <td>3.214286e+04</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>8.773071e+06</td>
      <td>7.710000e+05</td>
      <td>1.125143e+06</td>
      <td>92714.285714</td>
      <td>7.057143e+04</td>
      <td>29142.857143</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2132</th>
      <td>2.226747e+07</td>
      <td>2.285714e+04</td>
      <td>3.435714e+05</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2133</th>
      <td>7.258614e+06</td>
      <td>2.794286e+05</td>
      <td>1.009286e+06</td>
      <td>142857.142857</td>
      <td>1.088714e+06</td>
      <td>407571.428571</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2134</th>
      <td>8.026786e+06</td>
      <td>8.714286e+03</td>
      <td>3.485714e+04</td>
      <td>NaN</td>
      <td>7.714286e+04</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2135</th>
      <td>1.811407e+07</td>
      <td>3.314286e+04</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2136</th>
      <td>5.079504e+07</td>
      <td>2.542857e+04</td>
      <td>7.985714e+04</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>1967 rows × 34 columns</p>
</div>




```python
# 결측치 0으로 메우기
df2_1 = df2_1.fillna(0)
df2_1
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>installment_term</th>
      <th>0</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>...</th>
      <th>63</th>
      <th>65</th>
      <th>72</th>
      <th>80</th>
      <th>82</th>
      <th>83</th>
      <th>93</th>
    </tr>
    <tr>
      <th>store_id</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2.163076e+07</td>
      <td>1.123857e+06</td>
      <td>1.204000e+06</td>
      <td>77142.857143</td>
      <td>4.142857e+04</td>
      <td>75714.285714</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3.668643e+06</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7.511143e+06</td>
      <td>4.525714e+05</td>
      <td>7.735714e+05</td>
      <td>0.000000</td>
      <td>1.857143e+04</td>
      <td>87142.857143</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2.984553e+07</td>
      <td>2.271429e+04</td>
      <td>3.214286e+04</td>
      <td>0.000000</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>8.773071e+06</td>
      <td>7.710000e+05</td>
      <td>1.125143e+06</td>
      <td>92714.285714</td>
      <td>7.057143e+04</td>
      <td>29142.857143</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2132</th>
      <td>2.226747e+07</td>
      <td>2.285714e+04</td>
      <td>3.435714e+05</td>
      <td>0.000000</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2133</th>
      <td>7.258614e+06</td>
      <td>2.794286e+05</td>
      <td>1.009286e+06</td>
      <td>142857.142857</td>
      <td>1.088714e+06</td>
      <td>407571.428571</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2134</th>
      <td>8.026786e+06</td>
      <td>8.714286e+03</td>
      <td>3.485714e+04</td>
      <td>0.000000</td>
      <td>7.714286e+04</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2135</th>
      <td>1.811407e+07</td>
      <td>3.314286e+04</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2136</th>
      <td>5.079504e+07</td>
      <td>2.542857e+04</td>
      <td>7.985714e+04</td>
      <td>0.000000</td>
      <td>0.000000e+00</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
<p>1967 rows × 34 columns</p>
</div>




```python
# 할부개월수 별 박스플롯

# 한글폰트 오류 해결

from matplotlib import font_manager, rc
font_path = "./malgun.ttf"#폰트파일 위치
font_name = font_manager.FontProperties(fname=font_path).get_name()
rc('font', family=font_name)

plt.style.use('seaborn-poster')#스타일 서식 지정
plt.rcParams['axes.unicode_minus']=False #마이너스 부호 출력 지정

# 그래프 객체 생성
fig = plt.figure(figsize=(30, 10))
ax1 = fig.add_subplot(1, 2, 1)

# axe 객체에 boxplot 메소드로 그래프 출력 
data = df2_1

green_diamond = dict(markerfacecolor='r', marker='s')
plt.boxplot(data, flierprops=green_diamond)

ax1.set_title('상점 별 할부개월 수 별 총 매출분포')
plt.show()
```


![funda14]({{ site.url }}{{ site.baseurl }}/assets/images/funda14.png)


할부개월 수가 적을 수록 총매출이 크며 이상치도 제일 크다 (1,3,5,6,10, 12달 이상치 스케일러가 필요


```python
# 그룹바이
# 상점별 지역별 총매출,  region  amount

df3 = df0.groupby(['region'])['amount'].sum()
df3
```




    region
    0         1.936992e+10
    강원 강릉시    2.343820e+08
    강원 삼척시    6.896129e+07
    강원 속초시    2.961477e+08
    강원 양구군    3.075680e+07
                  ...     
    충북 제천시    1.534382e+08
    충북 증평군    4.346315e+07
    충북 진천군    3.545923e+07
    충북 청주시    9.396727e+08
    충북 충주시    3.534095e+08
    Name: amount, Length: 181, dtype: float64




```python
df3=df3.reset_index()
```


```python
df3
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>region</th>
      <th>amount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>1.936992e+10</td>
    </tr>
    <tr>
      <th>1</th>
      <td>강원 강릉시</td>
      <td>2.343820e+08</td>
    </tr>
    <tr>
      <th>2</th>
      <td>강원 삼척시</td>
      <td>6.896129e+07</td>
    </tr>
    <tr>
      <th>3</th>
      <td>강원 속초시</td>
      <td>2.961477e+08</td>
    </tr>
    <tr>
      <th>4</th>
      <td>강원 양구군</td>
      <td>3.075680e+07</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>176</th>
      <td>충북 제천시</td>
      <td>1.534382e+08</td>
    </tr>
    <tr>
      <th>177</th>
      <td>충북 증평군</td>
      <td>4.346315e+07</td>
    </tr>
    <tr>
      <th>178</th>
      <td>충북 진천군</td>
      <td>3.545923e+07</td>
    </tr>
    <tr>
      <th>179</th>
      <td>충북 청주시</td>
      <td>9.396727e+08</td>
    </tr>
    <tr>
      <th>180</th>
      <td>충북 충주시</td>
      <td>3.534095e+08</td>
    </tr>
  </tbody>
</table>
<p>181 rows × 2 columns</p>
</div>




```python
# 특정 열값 기준으로 행삭제하기
df3.drop(df3.loc[df3['region']==0].index, inplace=True)
```


```python
# 지역별 인덱스 
df3.index = df3['region']
df3.set_index('region', inplace=True)
```


```python
# 지역별 매출이 가장 많은 내림차순
df3Sort = df3.sort_values(by=['amount'], axis=0, ascending=False)
df3Sort
# 문자열과 숫자 섞여 있으면 오류->0 값 삭제하기
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>amount</th>
    </tr>
    <tr>
      <th>region</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>서울 강남구</th>
      <td>2.164762e+09</td>
    </tr>
    <tr>
      <th>서울 광진구</th>
      <td>1.905546e+09</td>
    </tr>
    <tr>
      <th>경기 수원시</th>
      <td>1.382518e+09</td>
    </tr>
    <tr>
      <th>경기 용인시</th>
      <td>1.118154e+09</td>
    </tr>
    <tr>
      <th>서울 마포구</th>
      <td>1.071436e+09</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>경북 칠곡군</th>
      <td>1.068006e+07</td>
    </tr>
    <tr>
      <th>경북 안동시</th>
      <td>1.051727e+07</td>
    </tr>
    <tr>
      <th>충남 예산군</th>
      <td>8.147500e+06</td>
    </tr>
    <tr>
      <th>세종 도움3로</th>
      <td>8.090357e+06</td>
    </tr>
    <tr>
      <th>경남 거창군</th>
      <td>5.423000e+06</td>
    </tr>
  </tbody>
</table>
<p>180 rows × 1 columns</p>
</div>




```python
# 지역별 상위 10순위
df3Sort2 = df3Sort.iloc[:9]
```


```python
df3Sort.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>amount</th>
    </tr>
    <tr>
      <th>region</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>경북 칠곡군</th>
      <td>1.068006e+07</td>
    </tr>
    <tr>
      <th>경북 안동시</th>
      <td>1.051727e+07</td>
    </tr>
    <tr>
      <th>충남 예산군</th>
      <td>8.147500e+06</td>
    </tr>
    <tr>
      <th>세종 도움3로</th>
      <td>8.090357e+06</td>
    </tr>
    <tr>
      <th>경남 거창군</th>
      <td>5.423000e+06</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 지역별 하위 10순위
df3Sort3 = df3Sort.iloc[-10:]
df3Sort3
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>amount</th>
    </tr>
    <tr>
      <th>region</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>경남 함안군</th>
      <td>1.861686e+07</td>
    </tr>
    <tr>
      <th>전북 남원시</th>
      <td>1.573779e+07</td>
    </tr>
    <tr>
      <th>경북 영천시</th>
      <td>1.202041e+07</td>
    </tr>
    <tr>
      <th>경북 울진군</th>
      <td>1.191007e+07</td>
    </tr>
    <tr>
      <th>강원 철원군</th>
      <td>1.123496e+07</td>
    </tr>
    <tr>
      <th>경북 칠곡군</th>
      <td>1.068006e+07</td>
    </tr>
    <tr>
      <th>경북 안동시</th>
      <td>1.051727e+07</td>
    </tr>
    <tr>
      <th>충남 예산군</th>
      <td>8.147500e+06</td>
    </tr>
    <tr>
      <th>세종 도움3로</th>
      <td>8.090357e+06</td>
    </tr>
    <tr>
      <th>경남 거창군</th>
      <td>5.423000e+06</td>
    </tr>
  </tbody>
</table>
</div>




```python
#가로 막대그래프 시각화

#스타일 서식 지정
plt.style.use('ggplot')

# 수평 막대 그리기

df3Sort2.plot(kind='barh', figsize=(20,10), width=0.5,
          color='cornflowerblue')

plt.title('지역별 총 매출 상위 10', size=30)
plt.ylabel('지역',size=20 )
plt.xlabel('총매출',size=20)

plt.show()
```


![funda15]({{ site.url }}{{ site.baseurl }}/assets/images/funda15.png)




```python
#가로 막대그래프 시각화

#스타일 서식 지정
plt.style.use('ggplot')

# 수평 막대 그리기

df3Sort3.plot(kind='barh', figsize=(20,10), width=0.5,
          color='cornflowerblue')

plt.title('지역별 총 매출 하위 10', size=30)
plt.ylabel('지역',size=20 )
plt.xlabel('총매출',size=20)

plt.show()
```




![funda16]{{ site.url }}{{ site.baseurl }}/assets/images/funda16.png)


```python
#그룹바이
#업종별 총매출 

df4 = df0.groupby(['type_of_business'])['amount'].sum()
df4
```




    type_of_business
    0                    3.948875e+10
    가구 소매업               2.228384e+08
    가전제품 소매업             2.151943e+08
    가정용 세탁업              5.915388e+07
    가정용 직물제품 소매업         9.278691e+07
                             ...     
    한의원                  2.135252e+07
    화장품 및 화장용품 도매업       3.711348e+08
    화장품, 비누 및 방향제 소매업    4.286110e+08
    화초 및 식물 소매업          1.282732e+08
    화훼류 및 식물 도매업         2.903707e+07
    Name: amount, Length: 146, dtype: float64




```python
df4=df4.reset_index()
```


```python
df4
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>type_of_business</th>
      <th>amount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>3.948875e+10</td>
    </tr>
    <tr>
      <th>1</th>
      <td>가구 소매업</td>
      <td>2.228384e+08</td>
    </tr>
    <tr>
      <th>2</th>
      <td>가전제품 소매업</td>
      <td>2.151943e+08</td>
    </tr>
    <tr>
      <th>3</th>
      <td>가정용 세탁업</td>
      <td>5.915388e+07</td>
    </tr>
    <tr>
      <th>4</th>
      <td>가정용 직물제품 소매업</td>
      <td>9.278691e+07</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>141</th>
      <td>한의원</td>
      <td>2.135252e+07</td>
    </tr>
    <tr>
      <th>142</th>
      <td>화장품 및 화장용품 도매업</td>
      <td>3.711348e+08</td>
    </tr>
    <tr>
      <th>143</th>
      <td>화장품, 비누 및 방향제 소매업</td>
      <td>4.286110e+08</td>
    </tr>
    <tr>
      <th>144</th>
      <td>화초 및 식물 소매업</td>
      <td>1.282732e+08</td>
    </tr>
    <tr>
      <th>145</th>
      <td>화훼류 및 식물 도매업</td>
      <td>2.903707e+07</td>
    </tr>
  </tbody>
</table>
<p>146 rows × 2 columns</p>
</div>




```python
# 특정 열값 기준으로 행삭제하기
df4.drop(df4.loc[df4['type_of_business']==0].index, inplace=True)
```


```python
# 지역별 인덱스 
df4.index = df4['type_of_business']
df4.set_index('type_of_business', inplace=True)
```


```python
# 지역별 매출이 가장 많은 내림차순
df4Sort = df4.sort_values(by=['amount'], axis=0, ascending=False)
df4Sort
# 문자열과 숫자 섞여 있으면 오류->0 값 삭제하기
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>amount</th>
    </tr>
    <tr>
      <th>type_of_business</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>한식 음식점업</th>
      <td>4.810450e+09</td>
    </tr>
    <tr>
      <th>의복 소매업</th>
      <td>2.403674e+09</td>
    </tr>
    <tr>
      <th>의약품 도매업</th>
      <td>1.568540e+09</td>
    </tr>
    <tr>
      <th>일반 교과 학원</th>
      <td>1.293611e+09</td>
    </tr>
    <tr>
      <th>두발 미용업</th>
      <td>1.087475e+09</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>예술품 및 골동품 소매업</th>
      <td>3.172929e+06</td>
    </tr>
    <tr>
      <th>채소, 과실 및 뿌리작물 소매업</th>
      <td>2.532857e+06</td>
    </tr>
    <tr>
      <th>곡물 및 기타 식량작물 재배업</th>
      <td>1.972814e+06</td>
    </tr>
    <tr>
      <th>배전반 및 전기 자동제어반 제조업</th>
      <td>1.635599e+06</td>
    </tr>
    <tr>
      <th>신선식품 및 단순 가공식품 도매업</th>
      <td>5.063271e+05</td>
    </tr>
  </tbody>
</table>
<p>145 rows × 1 columns</p>
</div>




```python
# 지역별 상위 10순위
df4Sort2 = df4Sort.iloc[:9]
```


```python
# 지역별 하위 10순위
df4Sort3 = df4Sort.iloc[-10:]
df4Sort3
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>amount</th>
    </tr>
    <tr>
      <th>type_of_business</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>문구용품, 회화용품, 사무용품·· 도매업</th>
      <td>6.033071e+06</td>
    </tr>
    <tr>
      <th>그 외 기타 나무제품 제조업</th>
      <td>4.462571e+06</td>
    </tr>
    <tr>
      <th>기록매체 복제업</th>
      <td>4.330941e+06</td>
    </tr>
    <tr>
      <th>일반 의원</th>
      <td>4.106983e+06</td>
    </tr>
    <tr>
      <th>체인화 편의점</th>
      <td>3.871520e+06</td>
    </tr>
    <tr>
      <th>예술품 및 골동품 소매업</th>
      <td>3.172929e+06</td>
    </tr>
    <tr>
      <th>채소, 과실 및 뿌리작물 소매업</th>
      <td>2.532857e+06</td>
    </tr>
    <tr>
      <th>곡물 및 기타 식량작물 재배업</th>
      <td>1.972814e+06</td>
    </tr>
    <tr>
      <th>배전반 및 전기 자동제어반 제조업</th>
      <td>1.635599e+06</td>
    </tr>
    <tr>
      <th>신선식품 및 단순 가공식품 도매업</th>
      <td>5.063271e+05</td>
    </tr>
  </tbody>
</table>
</div>


### 업종별 총 매출 상위 10

```python
#가로 막대그래프 시각화

#스타일 서식 지정
plt.style.use('ggplot')

# 수평 막대 그리기

df4Sort2.plot(kind='barh', figsize=(20,10), width=0.5,
          color='cornflowerblue')

plt.title('업종별 총 매출 상위 10', size=30)
plt.ylabel('업종',size=20 )
plt.xlabel('총매출',size=20)

plt.show()
```

### 업종별 총 매출 하위 10

![funda17]{{ site.url }}{{ site.baseurl }}/assets/images/funda17.png)



```python
#가로 막대그래프 시각화

#스타일 서식 지정
plt.style.use('ggplot')

# 수평 막대 그리기

df4Sort3.plot(kind='barh', figsize=(20,10), width=0.5,
          color='cornflowerblue')

plt.title('지역별 총 매출 하위 10', size=30)
plt.ylabel('지역',size=20 )
plt.xlabel('총매출',size=20)

plt.show()
```
![funda18]{{ site.url }}{{ site.baseurl }}/assets/images/funda18.png)


## 펀다 상점 매출 ARIMA 예측 모델

- ARIMA 모델 외 계절성 요인을 추가한 SARIMAX 모델이 있는데 Seasonal_Decompose 모델로
계절성 주기별 반복 요인이 있는지 확인하였으나 패턴은 있지만, 계절성 요인이 일정하게 반복되지 않는 걸 확인하여
SARIMAX 모델 대신 계절성 요인을 제거한 ARIMA 모델로 분석 실시


```python
import pandas as pd
import numpy as np
import warnings
warnings.filterwarnings(action='ignore')
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf
```


```python
df = pd.read_csv(r'funda_train.csv', encoding='utf-8-sig')
print(df)
```

             store_id  card_id card_company transacted_date transacted_time  \
    0               0        0            b      2016-06-01           13:13   
    1               0        1            h      2016-06-01           18:12   
    2               0        2            c      2016-06-01           18:52   
    3               0        3            a      2016-06-01           20:22   
    4               0        4            c      2016-06-02           11:06   
    ...           ...      ...          ...             ...             ...   
    6556608      2136  4663855            d      2019-02-28           23:20   
    6556609      2136  4663855            d      2019-02-28           23:24   
    6556610      2136  4663489            a      2019-02-28           23:24   
    6556611      2136  4663856            d      2019-02-28           23:27   
    6556612      2136  4658616            c      2019-02-28           23:54   
    
             installment_term  region type_of_business       amount  
    0                       0     NaN           기타 미용업  1857.142857  
    1                       0     NaN           기타 미용업   857.142857  
    2                       0     NaN           기타 미용업  2000.000000  
    3                       0     NaN           기타 미용업  7857.142857  
    4                       0     NaN           기타 미용업  2000.000000  
    ...                   ...     ...              ...          ...  
    6556608                 0  제주 제주시           기타 주점업 -4500.000000  
    6556609                 0  제주 제주시           기타 주점업  4142.857143  
    6556610                 0  제주 제주시           기타 주점업  4500.000000  
    6556611                 0  제주 제주시           기타 주점업   571.428571  
    6556612                 0  제주 제주시           기타 주점업  5857.142857  
    
    [6556613 rows x 9 columns]



```python
pd.set_option('display.max_columns', 15)
df.head()
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>store_id</th>
      <th>card_id</th>
      <th>card_company</th>
      <th>transacted_date</th>
      <th>transacted_time</th>
      <th>installment_term</th>
      <th>region</th>
      <th>type_of_business</th>
      <th>amount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>b</td>
      <td>2016-06-01</td>
      <td>13:13</td>
      <td>0</td>
      <td>NaN</td>
      <td>기타 미용업</td>
      <td>1857.142857</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>1</td>
      <td>h</td>
      <td>2016-06-01</td>
      <td>18:12</td>
      <td>0</td>
      <td>NaN</td>
      <td>기타 미용업</td>
      <td>857.142857</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>2</td>
      <td>c</td>
      <td>2016-06-01</td>
      <td>18:52</td>
      <td>0</td>
      <td>NaN</td>
      <td>기타 미용업</td>
      <td>2000.000000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>3</td>
      <td>a</td>
      <td>2016-06-01</td>
      <td>20:22</td>
      <td>0</td>
      <td>NaN</td>
      <td>기타 미용업</td>
      <td>7857.142857</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>4</td>
      <td>c</td>
      <td>2016-06-02</td>
      <td>11:06</td>
      <td>0</td>
      <td>NaN</td>
      <td>기타 미용업</td>
      <td>2000.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
df0 = df.fillna(0)
df0
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>store_id</th>
      <th>card_id</th>
      <th>card_company</th>
      <th>transacted_date</th>
      <th>transacted_time</th>
      <th>installment_term</th>
      <th>region</th>
      <th>type_of_business</th>
      <th>amount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>b</td>
      <td>2016-06-01</td>
      <td>13:13</td>
      <td>0</td>
      <td>0</td>
      <td>기타 미용업</td>
      <td>1857.142857</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>1</td>
      <td>h</td>
      <td>2016-06-01</td>
      <td>18:12</td>
      <td>0</td>
      <td>0</td>
      <td>기타 미용업</td>
      <td>857.142857</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>2</td>
      <td>c</td>
      <td>2016-06-01</td>
      <td>18:52</td>
      <td>0</td>
      <td>0</td>
      <td>기타 미용업</td>
      <td>2000.000000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>3</td>
      <td>a</td>
      <td>2016-06-01</td>
      <td>20:22</td>
      <td>0</td>
      <td>0</td>
      <td>기타 미용업</td>
      <td>7857.142857</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>4</td>
      <td>c</td>
      <td>2016-06-02</td>
      <td>11:06</td>
      <td>0</td>
      <td>0</td>
      <td>기타 미용업</td>
      <td>2000.000000</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>6556608</th>
      <td>2136</td>
      <td>4663855</td>
      <td>d</td>
      <td>2019-02-28</td>
      <td>23:20</td>
      <td>0</td>
      <td>제주 제주시</td>
      <td>기타 주점업</td>
      <td>-4500.000000</td>
    </tr>
    <tr>
      <th>6556609</th>
      <td>2136</td>
      <td>4663855</td>
      <td>d</td>
      <td>2019-02-28</td>
      <td>23:24</td>
      <td>0</td>
      <td>제주 제주시</td>
      <td>기타 주점업</td>
      <td>4142.857143</td>
    </tr>
    <tr>
      <th>6556610</th>
      <td>2136</td>
      <td>4663489</td>
      <td>a</td>
      <td>2019-02-28</td>
      <td>23:24</td>
      <td>0</td>
      <td>제주 제주시</td>
      <td>기타 주점업</td>
      <td>4500.000000</td>
    </tr>
    <tr>
      <th>6556611</th>
      <td>2136</td>
      <td>4663856</td>
      <td>d</td>
      <td>2019-02-28</td>
      <td>23:27</td>
      <td>0</td>
      <td>제주 제주시</td>
      <td>기타 주점업</td>
      <td>571.428571</td>
    </tr>
    <tr>
      <th>6556612</th>
      <td>2136</td>
      <td>4658616</td>
      <td>c</td>
      <td>2019-02-28</td>
      <td>23:54</td>
      <td>0</td>
      <td>제주 제주시</td>
      <td>기타 주점업</td>
      <td>5857.142857</td>
    </tr>
  </tbody>
</table>
<p>6556613 rows × 9 columns</p>
</div>




```python
# transacted_date, transacted_time 데이터 타입 string으로 인식하여 datetime64로 변환(column.dt.시간단위를 통해서 원하는 시간 부분을 추출해서 활용할 수 있음)
df0['transacted_date'] = pd.to_datetime(df0['transacted_date'])
df0['transacted_time'] = pd.to_datetime(df0['transacted_time'])
```


```python
# 2016년 이전 데이터 확인
year1970 = (df0['transacted_date'] < '2016-06-01')
```


```python
filtered_df=df0.loc[year1970]
```


```python
filtered_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>store_id</th>
      <th>card_id</th>
      <th>card_company</th>
      <th>transacted_date</th>
      <th>transacted_time</th>
      <th>installment_term</th>
      <th>region</th>
      <th>type_of_business</th>
      <th>amount</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
from datetime import *
```


```python
#일자별 매출 그룹바이
daily_amount = df0.groupby(['store_id','transacted_date'])['amount'].sum()
daily_amount
```




    store_id  transacted_date
    0         2016-06-01         12571.428571
              2016-06-02         40571.428571
              2016-06-03         18142.857143
              2016-06-04         31714.285714
              2016-06-05         10428.571429
                                     ...     
    2136      2019-02-24         85357.142857
              2019-02-25         37214.285714
              2019-02-26         47142.857143
              2019-02-27         65071.428571
              2019-02-28         65857.142857
    Name: amount, Length: 1286113, dtype: float64




```python
daily_amount = daily_amount.reset_index()
```


```python
#인덱스를 거래일자로 재설정
daily_amount.index = daily_amount['transacted_date']
daily_amount.set_index('transacted_date', inplace=True)
daily_amount.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>store_id</th>
      <th>amount</th>
    </tr>
    <tr>
      <th>transacted_date</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-06-01</th>
      <td>0</td>
      <td>12571.428571</td>
    </tr>
    <tr>
      <th>2016-06-02</th>
      <td>0</td>
      <td>40571.428571</td>
    </tr>
    <tr>
      <th>2016-06-03</th>
      <td>0</td>
      <td>18142.857143</td>
    </tr>
    <tr>
      <th>2016-06-04</th>
      <td>0</td>
      <td>31714.285714</td>
    </tr>
    <tr>
      <th>2016-06-05</th>
      <td>0</td>
      <td>10428.571429</td>
    </tr>
  </tbody>
</table>
</div>




```python
import matplotlib.pyplot as plt
import seaborn as sns
```


```python
# 한글폰트 오류 해결

from matplotlib import font_manager, rc
font_path = "./malgun.ttf"#폰트파일 위치
font_name = font_manager.FontProperties(fname=font_path).get_name()
rc('font', family=font_name)

plt.style.use('seaborn-poster')#스타일 서식 지정
plt.rcParams['axes.unicode_minus']=False #마이너스 부호 출력 지정
```


```python
#일자별 매출 시각화
daily_amount.plot()
plt.show()
```



![funda19_1]({{ site.url }}{{ site.baseurl }}/assets/images/funda19 (1).png)



### 전처리 ###


```python
from pylab import rcParams
import statsmodels.api as sm
import itertools
from statsmodels.tsa.ar_model import AR
from statsmodels.tsa.arima_model import ARIMA
```


```python
# 월별 판매 예측
salesbymonth = daily_amount.amount.resample('M').sum()
salesbymonth
```




    transacted_date
    2016-06-30    1.565927e+09
    2016-07-31    1.631030e+09
    2016-08-31    1.631913e+09
    2016-09-30    1.616485e+09
    2016-10-31    1.770040e+09
    2016-11-30    1.789135e+09
    2016-12-31    1.993133e+09
    2017-01-31    1.787146e+09
    2017-02-28    1.824069e+09
    2017-03-31    2.144957e+09
    2017-04-30    2.144388e+09
    2017-05-31    2.316093e+09
    2017-06-30    2.266990e+09
    2017-07-31    2.284421e+09
    2017-08-31    2.208400e+09
    2017-09-30    2.278157e+09
    2017-10-31    2.081597e+09
    2017-11-30    2.188137e+09
    2017-12-31    2.312373e+09
    2018-01-31    2.114483e+09
    2018-02-28    1.953435e+09
    2018-03-31    2.356962e+09
    2018-04-30    2.280303e+09
    2018-05-31    2.321865e+09
    2018-06-30    2.194311e+09
    2018-07-31    2.241846e+09
    2018-08-31    2.174102e+09
    2018-09-30    2.129888e+09
    2018-10-31    2.279253e+09
    2018-11-30    2.208414e+09
    2018-12-31    2.263730e+09
    2019-01-31    2.165602e+09
    2019-02-28    1.900401e+09
    Freq: M, Name: amount, dtype: float64




```python
salesbymonth = salesbymonth.reset_index()
```


```python
#인덱스를 거래월자로 재설정
salesbymonth.index = salesbymonth['transacted_date']
salesbymonth.set_index('transacted_date', inplace=True)
salesbymonth.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>amount</th>
    </tr>
    <tr>
      <th>transacted_date</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-06-30</th>
      <td>1.565927e+09</td>
    </tr>
    <tr>
      <th>2016-07-31</th>
      <td>1.631030e+09</td>
    </tr>
    <tr>
      <th>2016-08-31</th>
      <td>1.631913e+09</td>
    </tr>
    <tr>
      <th>2016-09-30</th>
      <td>1.616485e+09</td>
    </tr>
    <tr>
      <th>2016-10-31</th>
      <td>1.770040e+09</td>
    </tr>
  </tbody>
</table>
</div>




```python
salesbymonth.plot()
```




    <AxesSubplot:xlabel='transacted_date'>





![funda19_2]({{ site.url }}{{ site.baseurl }}/assets/images/funda19 (2).png)




```python
# stationary 확인

timeseries = salesbymonth['amount']
timeseries.rolling(12).mean().plot()#12개월평균
```




    <AxesSubplot:xlabel='transacted_date'>





![funda19_3]({{ site.url }}{{ site.baseurl }}/assets/images/funda19 (3).png)



```python
timeseries.rolling(12).std().plot()#12개월분산
```




    <AxesSubplot:xlabel='transacted_date'>




![funda19_4]({{ site.url }}{{ site.baseurl }}/assets/images/funda19 (4).png)


```python
from statsmodels.tsa.seasonal import seasonal_decompose
```


```python
decomposition = seasonal_decompose(salesbymonth['amount'], model='additive', 
                            extrapolate_trend='freq')
```


```python
fig = plt.figure()
fig = decomposition.plot()
fig.set_size_inches(15,7)
```


    <Figure size 921.6x633.6 with 0 Axes>




![funda19_5]({{ site.url }}{{ site.baseurl }}/assets/images/funda19 (5).png)



## stationary 검증 방법 ##


```python
from statsmodels.tsa.stattools import adfuller
```


```python
result = adfuller(salesbymonth['amount'])
```


```python
result
#  0.12033314079559654-> p값 0.05보다 작아야 stationary -> 즉 stationary가 아니다
```




    (-2.4803330016522223,
     0.12033314079559654,
     0,
     32,
     {'1%': -3.653519805908203,
      '5%': -2.9572185644531253,
      '10%': -2.6175881640625},
     882.2547886536667)



###    - 검증 조건 ( p-value : 5% 이내면 reject으로 대체가설 선택됨 )
     -> 귀무가설(H0): non-stationary.
     -> 대체가설 (H1): stationary. 


```python
# stationary check 기능 함수
def adf_check(ts):
    result = adfuller(ts)
    if result[1] <=0.05:
        print('Stationary{}'.format(result[1]))
    else:
        print('Non-stationary {}'.format(result[1]))
```


```python
adf_check(salesbymonth['amount'])
```

    Non-stationary 0.12033314079559654


### differencing 

-일자별 매출을 월별 매출로 변환, ARIMA모델은 시계열 데이터가 stationary 특성을 보일 때 효과적이므로
데이터가 stationary 특성을 보이는지 확인할 수 있어야 함.
-안정성은 연속되는 숫자들의 평균, 분산, 공분산이 시간에 따라 변하지 않는 것(time invariant)을 의미하는데
Stationary 검증 조건 미만으로 비안정성 확인하여 차분 과정 진행


```python
salesbymonth['1st diff'] = salesbymonth['amount'] - salesbymonth['amount'].shift(1)
```


```python
salesbymonth.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>amount</th>
      <th>1st diff</th>
    </tr>
    <tr>
      <th>transacted_date</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-06-30</th>
      <td>1.565927e+09</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2016-07-31</th>
      <td>1.631030e+09</td>
      <td>6.510308e+07</td>
    </tr>
    <tr>
      <th>2016-08-31</th>
      <td>1.631913e+09</td>
      <td>8.831253e+05</td>
    </tr>
    <tr>
      <th>2016-09-30</th>
      <td>1.616485e+09</td>
      <td>-1.542843e+07</td>
    </tr>
    <tr>
      <th>2016-10-31</th>
      <td>1.770040e+09</td>
      <td>1.535552e+08</td>
    </tr>
  </tbody>
</table>
</div>




```python
adf_check(salesbymonth['1st diff'].dropna())
```

    Stationary7.548025158012783e-05



```python
salesbymonth['1st diff'].plot()
```




    <AxesSubplot:xlabel='transacted_date'>




![funda19_6]({{ site.url }}{{ site.baseurl }}/assets/images/funda19 (6).png)


### 2차 differencing 


```python
salesbymonth['2nd diff'] = salesbymonth['1st diff'] - salesbymonth['1st diff'].shift(1)
```


```python
salesbymonth['2nd diff'].plot()
```




    <AxesSubplot:xlabel='transacted_date'>





![funda19_7]({{ site.url }}{{ site.baseurl }}/assets/images/funda19 (7).png)





```python
adf_check(salesbymonth['2nd diff'].dropna())
```

    Stationary5.406048999294243e-07


### 3차 differencing 


```python
salesbymonth['3rd diff'] = salesbymonth['2nd diff'] - salesbymonth['2nd diff'].shift(1)
```

### 1차 differencing보다 더 작아짐 1st, 2nd 둘다 사용 가능 ###


- 데이터의 비안정성을 확인하였으므로 안정성있는 데이터로 2차에 걸쳐서 차분 진행,
  1차, 2차 모두 안정성 검증 통과하여 둘 다 사용 가능 확인


```python
salesbymonth['seasonal diff'] = salesbymonth['amount'] - salesbymonth['amount'].shift(12)
```


```python
salesbymonth['seasonal diff'].plot()
```




    <AxesSubplot:xlabel='transacted_date'>




![funda19_8]({{ site.url }}{{ site.baseurl }}/assets/images/funda19 (8).png)


```python
adf_check(salesbymonth['seasonal diff'].dropna())
```

    Non-stationary 0.1219271533339864



```python
# 1st differencing에 seasonal differencing을 하는 것도 방법임
```


```python
salesbymonth['seasonal 1st diff'] = salesbymonth['1st diff'] - salesbymonth['1st diff'].shift(12)
```


```python
salesbymonth['seasonal 1st diff'].plot()
```




    <AxesSubplot:xlabel='transacted_date'>



![funda19_9]({{ site.url }}{{ site.baseurl }}/assets/images/funda19 (9).png)



```python
adf_check(salesbymonth['seasonal 1st diff'].dropna())
```

    Non-stationary 1.0



```python
# 한번 더 differencing 하기 
salesbymonth['seasonal 2nd diff'] = salesbymonth['2nd diff'] - salesbymonth['2nd diff'].shift(12)
```


```python
adf_check(salesbymonth['seasonal 2nd diff'].dropna())
# 해당 데이터 쓰면 됨
```

    Stationary7.700713631511948e-12



```python
# 3차 seasonal differencing 하기 
salesbymonth['seasonal 3rd diff'] = salesbymonth['3rd diff'] - salesbymonth['3rd diff'].shift(12)
```


```python
adf_check(salesbymonth['seasonal 3rd diff'].dropna())
```

    Stationary0.0



```python
# d값은 2차 differencing이므로 2를 씀
d = 2, D = 2
```


      File "<ipython-input-53-fcbf216f370d>", line 2
        d = 2, D = 2
           ^
    SyntaxError: can't assign to literal




```python
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf
```


```python
plot_acf(salesbymonth['1st diff'].dropna());
```



![funda19_10]({{ site.url }}{{ site.baseurl }}/assets/images/funda19 (10).png)



```python
plot_acf(salesbymonth['2nd diff'].dropna());
```




![funda19_11]({{ site.url }}{{ site.baseurl }}/assets/images/funda19 (11).png)



```python
plot_acf(salesbymonth['seasonal 2nd diff'].dropna());
```



![funda19_12]({{ site.url }}{{ site.baseurl }}/assets/images/funda19 (12).png)




```python
plot_pacf(salesbymonth['seasonal 2nd diff'].dropna(), method='ywm')#partial AutoCorrelation
```


    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    <ipython-input-58-704005197490> in <module>
    ----> 1 plot_pacf(salesbymonth['seasonal 2nd diff'].dropna(), method='ywm')#partial AutoCorrelation
    

    C:\ProgramData\Anaconda3\envs\pydata\lib\site-packages\statsmodels\graphics\tsaplots.py in plot_pacf(x, ax, lags, alpha, method, use_vlines, title, zero, vlines_kwargs, **kwargs)
        305         acf_x = pacf(x, nlags=nlags, alpha=alpha, method=method)
        306     else:
    --> 307         acf_x, confint = pacf(x, nlags=nlags, alpha=alpha, method=method)
        308 
        309     _plot_corr(
    

    C:\ProgramData\Anaconda3\envs\pydata\lib\site-packages\statsmodels\tsa\stattools.py in pacf(x, nlags, method, alpha)
       1032     if nlags >= x.shape[0] // 2:
       1033         raise ValueError(
    -> 1034             "Can only compute partial correlations for lags up to 50% of the "
       1035             f"sample size. The requested nlags {nlags} must be < "
       1036             f"{x.shape[0] // 2}."
    

    ValueError: Can only compute partial correlations for lags up to 50% of the sample size. The requested nlags 13 must be < 9.



![funda19_13]({{ site.url }}{{ site.baseurl }}/assets/images/funda19 (13).png)



```python
# acf에서 음수 1개씩 나왔으므로 p, q값은 각 P=1. Q=1씀
```


```python
plot_pacf(salesbymonth['1st diff'].dropna(), method='ywm')#partial AutoCorrelation
```


    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    <ipython-input-60-63b23e61b596> in <module>
    ----> 1 plot_pacf(salesbymonth['1st diff'].dropna(), method='ywm')#partial AutoCorrelation
    

    C:\ProgramData\Anaconda3\envs\pydata\lib\site-packages\statsmodels\graphics\tsaplots.py in plot_pacf(x, ax, lags, alpha, method, use_vlines, title, zero, vlines_kwargs, **kwargs)
        305         acf_x = pacf(x, nlags=nlags, alpha=alpha, method=method)
        306     else:
    --> 307         acf_x, confint = pacf(x, nlags=nlags, alpha=alpha, method=method)
        308 
        309     _plot_corr(
    

    C:\ProgramData\Anaconda3\envs\pydata\lib\site-packages\statsmodels\tsa\stattools.py in pacf(x, nlags, method, alpha)
       1032     if nlags >= x.shape[0] // 2:
       1033         raise ValueError(
    -> 1034             "Can only compute partial correlations for lags up to 50% of the "
       1035             f"sample size. The requested nlags {nlags} must be < "
       1036             f"{x.shape[0] // 2}."
    

    ValueError: Can only compute partial correlations for lags up to 50% of the sample size. The requested nlags 16 must be < 16.



![funda19_14]({{ site.url }}{{ site.baseurl }}/assets/images/funda19 (14).png)



### 2017년 3월 2018년 2월까지 주기성이 보임, 점차적으로 증가하다 2018에서는 유지에서 머무르고 있다 

### 파라미터 찾기 

이제 본격적으로 ARIMA 모델을 만들어야 한다. ARIMA 모델을 만들기 위해 p, d, q 값을 지정해 주어야 하는데 일반적으로 p가 0이면 MA 모델을 따르고, q가 0이면 AR 모델을 따른다고 한다. d가 0이면 정상성을 보유한 모델이라고 보는데, 발생확률이 변하지 않는 ARMA 모델이라고 간주하면 된다고 한다. 우린 아직 이 데이터가 어떤 모형을 따르는 지 모르기 때문에 최적의 결과를 가져오는 p,d,q 값을 찾아야한다. 그 방법은 크게 두가지로 임의의 값들을 다 넣어 테스트 해보거나, pmdarima 패키지의 auto_arima 함수로 찾아내면 된다.


```python
#방법 1. p,d,q의 조합을 만들어 하나하나 ARIMA 모델을 돌려봄

#p,d,q 범위 생성 0~2
p = d = q = range(0, 2)

import itertools#반복되는 요소를 이어붙이는 모듈
pdqa = list(itertools.product(p, d, q))

seasonal_pdq = [(x[0], x[1], x[2], 12) for x in list(itertools.product(p, d, q))]
#x의 0~2범위 요소를 하나씩 꺼내서 pdq를 리스트안에 붙여서 for 반복문 돌리기

for param in pdqa:
    for param_seasonal in seasonal_pdq:
        try:
            mod = sm.tsa.statespace.SARIMAX(salesbymonth_train, order=param, seasonal_order=param_seasonal,enforce_stationarity=False,enforce_invertibility=False)                                
            results = mod.fit()
            print('ARIMA{}x{}12 - AIC:{}'.format(param, param_seasonal, results.aic))
        except:
            continue
```


```python
#방법 2. auto_arima 함수로 자동 추출
from pmdarima import auto_arima
stepwise_model = auto_arima(salesbymonth['amount'], start_p=1, start_q=1,
                           max_p=3, max_q=3, m=12,
                           start_P=0, seasonal=True,
                           d=1, D=1, trace=True,
                           error_action='ignore',  
                           suppress_warnings=True, 
                           stepwise=True)
```

    Performing stepwise search to minimize aic
     ARIMA(1,1,1)(0,1,1)[12]             : AIC=814.578, Time=0.20 sec
     ARIMA(0,1,0)(0,1,0)[12]             : AIC=813.295, Time=0.01 sec
     ARIMA(1,1,0)(1,1,0)[12]             : AIC=814.333, Time=0.05 sec
     ARIMA(0,1,1)(0,1,1)[12]             : AIC=815.580, Time=0.06 sec
     ARIMA(0,1,0)(1,1,0)[12]             : AIC=813.897, Time=0.03 sec
     ARIMA(0,1,0)(0,1,1)[12]             : AIC=812.354, Time=0.02 sec
     ARIMA(0,1,0)(1,1,1)[12]             : AIC=inf, Time=0.13 sec
     ARIMA(0,1,0)(0,1,2)[12]             : AIC=810.659, Time=0.12 sec
     ARIMA(0,1,0)(1,1,2)[12]             : AIC=812.198, Time=0.17 sec
     ARIMA(1,1,0)(0,1,2)[12]             : AIC=812.454, Time=0.11 sec
     ARIMA(0,1,1)(0,1,2)[12]             : AIC=inf, Time=0.20 sec
     ARIMA(1,1,1)(0,1,2)[12]             : AIC=811.579, Time=0.43 sec
     ARIMA(0,1,0)(0,1,2)[12] intercept   : AIC=812.505, Time=0.14 sec
    
    Best model:  ARIMA(0,1,0)(0,1,2)[12]          
    Total fit time: 1.678 seconds


### 하이퍼 파라미터 정하기 

p=0, d=1, q=0



### AIC(Akaike Information Criterion) 값이 제일 작은 조합을 선택 

- ARIMA 모델을 만들기 위해 p, d, q 값을 지정해 주어야 하는데 p는 과거 관측 갯수, d는 차분횟수, q는 오차갯수를 의미,
  일반적으로 p가 0이면 MA 모델을 따르고, q가 0이면 AR 모델을 따르기 때문에 d가 0이면 정상성을 보유한 모델이라고 봄
  최적의 결과를 가져오는 p,d,q 값을 찾기위해 pmdarima 패키지의 auto_arima 함수로 p,d,q 값 찾음
  학습 모델 검정하였을 때 AIC값이 낮을 수록 모델 품질이 높으므로 ARIMA(0, 1, 0)모델 채택

### 학습 및 검증 


```python
from statsmodels.tsa.arima_model import ARIMA
```


```python
model = ARIMA(salesbymonth['amount'], order=(0,1,0))
```

    C:\ProgramData\Anaconda3\envs\pydata\lib\site-packages\statsmodels\tsa\base\tsa_model.py:527: ValueWarning: No frequency information was provided, so inferred frequency M will be used.
      % freq, ValueWarning)
    C:\ProgramData\Anaconda3\envs\pydata\lib\site-packages\statsmodels\tsa\base\tsa_model.py:527: ValueWarning: No frequency information was provided, so inferred frequency M will be used.
      % freq, ValueWarning)



```python
result= model.fit(trend='c', full_output=True, disp=True)
```


```python
result.summary()
#AIC	1296.617 모델 성능 의미 
```




<table class="simpletable">
<caption>ARIMA Model Results</caption>
<tr>
  <th>Dep. Variable:</th>     <td>D.amount</td>     <th>  No. Observations:  </th>      <td>32</td>      
</tr>
<tr>
  <th>Model:</th>          <td>ARIMA(0, 1, 0)</td>  <th>  Log Likelihood     </th>   <td>-647.220</td>   
</tr>
<tr>
  <th>Method:</th>               <td>css</td>       <th>  S.D. of innovations</th> <td>147110102.426</td>
</tr>
<tr>
  <th>Date:</th>          <td>Wed, 02 Jun 2021</td> <th>  AIC                </th>   <td>1298.440</td>   
</tr>
<tr>
  <th>Time:</th>              <td>22:32:03</td>     <th>  BIC                </th>   <td>1301.372</td>   
</tr>
<tr>
  <th>Sample:</th>           <td>07-31-2016</td>    <th>  HQIC               </th>   <td>1299.412</td>   
</tr>
<tr>
  <th></th>                 <td>- 02-28-2019</td>   <th>                     </th>       <td> </td>      
</tr>
</table>
<table class="simpletable">
<tr>
    <td></td>       <th>coef</th>     <th>std err</th>      <th>z</th>      <th>P>|z|</th>  <th>[0.025</th>    <th>0.975]</th>  
</tr>
<tr>
  <th>const</th> <td> 1.045e+07</td> <td>  2.6e+07</td> <td>    0.402</td> <td> 0.688</td> <td>-4.05e+07</td> <td> 6.14e+07</td>
</tr>
</table>




```python
result.resid.plot()
```




    <AxesSubplot:xlabel='transacted_date'>





![funda19_15]({{ site.url }}{{ site.baseurl }}/assets/images/funda19 (15).png)





```python
result.resid.plot(kind='kde')
# 가우스분포 0이므로 모델 피팅이 어느정도 된 것
```




    <AxesSubplot:ylabel='Density'>





![funda19_16]({{ site.url }}{{ site.baseurl }}/assets/images/funda19 (16).png)



## 예측  


```python
result.plot_predict()
```





![funda19_17]({{ site.url }}{{ site.baseurl }}/assets/images/funda19 (17).png)
![funda19_18]({{ site.url }}{{ site.baseurl }}/assets/images/funda19 (18).png)





```python
from sklearn.metrics import mean_squared_error
from math import sqrt
from pandas import datetime
from datetime import timedelta
```

### Out-of- sample 예측(5개월) 


- 가우스 분포로 0에 가까운지 그래프 모형 확인
  ARIMA(0,1,0)모델로 예측 실시 전의 데이터를 기준으로 예측을 따라가는 것을 볼 수 있음
  Out-of-sample로 앞으로의 매출 값 예측하였을 때 그래프 모형 확인


```python
from pandas.tseries.offsets import DateOffset
```


```python
future_dates=[salesbymonth['amount'].index[-1]+ DateOffset(months=x)for x in range(0,6)]
```


```python
type(salesbymonth['amount'])
```




    pandas.core.series.Series




```python
salesbymonth
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>amount</th>
      <th>1st diff</th>
      <th>2nd diff</th>
      <th>3rd diff</th>
      <th>seasonal diff</th>
      <th>seasonal 1st diff</th>
      <th>seasonal 2nd diff</th>
      <th>seasonal 3rd diff</th>
    </tr>
    <tr>
      <th>transacted_date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-06-30</th>
      <td>1.565927e+09</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2016-07-31</th>
      <td>1.631030e+09</td>
      <td>6.510308e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2016-08-31</th>
      <td>1.631913e+09</td>
      <td>8.831253e+05</td>
      <td>-6.421996e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2016-09-30</th>
      <td>1.616485e+09</td>
      <td>-1.542843e+07</td>
      <td>-1.631156e+07</td>
      <td>4.790840e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2016-10-31</th>
      <td>1.770040e+09</td>
      <td>1.535552e+08</td>
      <td>1.689836e+08</td>
      <td>1.852952e+08</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2016-11-30</th>
      <td>1.789135e+09</td>
      <td>1.909530e+07</td>
      <td>-1.344599e+08</td>
      <td>-3.034435e+08</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2016-12-31</th>
      <td>1.993133e+09</td>
      <td>2.039973e+08</td>
      <td>1.849020e+08</td>
      <td>3.193619e+08</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-01-31</th>
      <td>1.787146e+09</td>
      <td>-2.059864e+08</td>
      <td>-4.099837e+08</td>
      <td>-5.948857e+08</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-02-28</th>
      <td>1.824069e+09</td>
      <td>3.692261e+07</td>
      <td>2.429090e+08</td>
      <td>6.528928e+08</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-03-31</th>
      <td>2.144957e+09</td>
      <td>3.208885e+08</td>
      <td>2.839659e+08</td>
      <td>4.105683e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-04-30</th>
      <td>2.144388e+09</td>
      <td>-5.693449e+05</td>
      <td>-3.214578e+08</td>
      <td>-6.054237e+08</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-05-31</th>
      <td>2.316093e+09</td>
      <td>1.717051e+08</td>
      <td>1.722745e+08</td>
      <td>4.937323e+08</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-06-30</th>
      <td>2.266990e+09</td>
      <td>-4.910315e+07</td>
      <td>-2.208083e+08</td>
      <td>-3.930827e+08</td>
      <td>7.010628e+08</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-07-31</th>
      <td>2.284421e+09</td>
      <td>1.743098e+07</td>
      <td>6.653413e+07</td>
      <td>2.873424e+08</td>
      <td>6.533907e+08</td>
      <td>-4.767210e+07</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-08-31</th>
      <td>2.208400e+09</td>
      <td>-7.602059e+07</td>
      <td>-9.345157e+07</td>
      <td>-1.599857e+08</td>
      <td>5.764870e+08</td>
      <td>-7.690372e+07</td>
      <td>-2.923162e+07</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-09-30</th>
      <td>2.278157e+09</td>
      <td>6.975645e+07</td>
      <td>1.457770e+08</td>
      <td>2.392286e+08</td>
      <td>6.616719e+08</td>
      <td>8.518488e+07</td>
      <td>1.620886e+08</td>
      <td>1.913202e+08</td>
    </tr>
    <tr>
      <th>2017-10-31</th>
      <td>2.081597e+09</td>
      <td>-1.965599e+08</td>
      <td>-2.663164e+08</td>
      <td>-4.120934e+08</td>
      <td>3.115568e+08</td>
      <td>-3.501151e+08</td>
      <td>-4.353000e+08</td>
      <td>-5.973886e+08</td>
    </tr>
    <tr>
      <th>2017-11-30</th>
      <td>2.188137e+09</td>
      <td>1.065405e+08</td>
      <td>3.031004e+08</td>
      <td>5.694168e+08</td>
      <td>3.990019e+08</td>
      <td>8.744519e+07</td>
      <td>4.375603e+08</td>
      <td>8.728603e+08</td>
    </tr>
    <tr>
      <th>2017-12-31</th>
      <td>2.312373e+09</td>
      <td>1.242353e+08</td>
      <td>1.769483e+07</td>
      <td>-2.854056e+08</td>
      <td>3.192400e+08</td>
      <td>-7.976197e+07</td>
      <td>-1.672072e+08</td>
      <td>-6.047675e+08</td>
    </tr>
    <tr>
      <th>2018-01-31</th>
      <td>2.114483e+09</td>
      <td>-1.978897e+08</td>
      <td>-3.221251e+08</td>
      <td>-3.398199e+08</td>
      <td>3.273367e+08</td>
      <td>8.096684e+06</td>
      <td>8.785865e+07</td>
      <td>2.550658e+08</td>
    </tr>
    <tr>
      <th>2018-02-28</th>
      <td>1.953435e+09</td>
      <td>-1.610476e+08</td>
      <td>3.684213e+07</td>
      <td>3.589672e+08</td>
      <td>1.293664e+08</td>
      <td>-1.979702e+08</td>
      <td>-2.060669e+08</td>
      <td>-2.939256e+08</td>
    </tr>
    <tr>
      <th>2018-03-31</th>
      <td>2.356962e+09</td>
      <td>4.035267e+08</td>
      <td>5.645743e+08</td>
      <td>5.277321e+08</td>
      <td>2.120046e+08</td>
      <td>8.263818e+07</td>
      <td>2.806084e+08</td>
      <td>4.866753e+08</td>
    </tr>
    <tr>
      <th>2018-04-30</th>
      <td>2.280303e+09</td>
      <td>-7.665848e+07</td>
      <td>-4.801851e+08</td>
      <td>-1.044759e+09</td>
      <td>1.359155e+08</td>
      <td>-7.608913e+07</td>
      <td>-1.587273e+08</td>
      <td>-4.393357e+08</td>
    </tr>
    <tr>
      <th>2018-05-31</th>
      <td>2.321865e+09</td>
      <td>4.156173e+07</td>
      <td>1.182202e+08</td>
      <td>5.984053e+08</td>
      <td>5.772091e+06</td>
      <td>-1.301434e+08</td>
      <td>-5.405426e+07</td>
      <td>1.046731e+08</td>
    </tr>
    <tr>
      <th>2018-06-30</th>
      <td>2.194311e+09</td>
      <td>-1.275543e+08</td>
      <td>-1.691161e+08</td>
      <td>-2.873363e+08</td>
      <td>-7.267911e+07</td>
      <td>-7.845120e+07</td>
      <td>5.169219e+07</td>
      <td>1.057464e+08</td>
    </tr>
    <tr>
      <th>2018-07-31</th>
      <td>2.241846e+09</td>
      <td>4.753548e+07</td>
      <td>1.750898e+08</td>
      <td>3.442059e+08</td>
      <td>-4.257460e+07</td>
      <td>3.010450e+07</td>
      <td>1.085557e+08</td>
      <td>5.686351e+07</td>
    </tr>
    <tr>
      <th>2018-08-31</th>
      <td>2.174102e+09</td>
      <td>-6.774396e+07</td>
      <td>-1.152794e+08</td>
      <td>-2.903693e+08</td>
      <td>-3.429797e+07</td>
      <td>8.276632e+06</td>
      <td>-2.182787e+07</td>
      <td>-1.303836e+08</td>
    </tr>
    <tr>
      <th>2018-09-30</th>
      <td>2.129888e+09</td>
      <td>-4.421403e+07</td>
      <td>2.352993e+07</td>
      <td>1.388094e+08</td>
      <td>-1.482685e+08</td>
      <td>-1.139705e+08</td>
      <td>-1.222471e+08</td>
      <td>-1.004192e+08</td>
    </tr>
    <tr>
      <th>2018-10-31</th>
      <td>2.279253e+09</td>
      <td>1.493645e+08</td>
      <td>1.935785e+08</td>
      <td>1.700486e+08</td>
      <td>1.976559e+08</td>
      <td>3.459244e+08</td>
      <td>4.598949e+08</td>
      <td>5.821420e+08</td>
    </tr>
    <tr>
      <th>2018-11-30</th>
      <td>2.208414e+09</td>
      <td>-7.083914e+07</td>
      <td>-2.202036e+08</td>
      <td>-4.137821e+08</td>
      <td>2.027632e+07</td>
      <td>-1.773796e+08</td>
      <td>-5.233040e+08</td>
      <td>-9.831989e+08</td>
    </tr>
    <tr>
      <th>2018-12-31</th>
      <td>2.263730e+09</td>
      <td>5.531690e+07</td>
      <td>1.261560e+08</td>
      <td>3.463596e+08</td>
      <td>-4.864209e+07</td>
      <td>-6.891842e+07</td>
      <td>1.084612e+08</td>
      <td>6.317652e+08</td>
    </tr>
    <tr>
      <th>2019-01-31</th>
      <td>2.165602e+09</td>
      <td>-9.812826e+07</td>
      <td>-1.534452e+08</td>
      <td>-2.796012e+08</td>
      <td>5.111939e+07</td>
      <td>9.976149e+07</td>
      <td>1.686799e+08</td>
      <td>6.021870e+07</td>
    </tr>
    <tr>
      <th>2019-02-28</th>
      <td>1.900401e+09</td>
      <td>-2.652015e+08</td>
      <td>-1.670732e+08</td>
      <td>-1.362808e+07</td>
      <td>-5.303449e+07</td>
      <td>-1.041539e+08</td>
      <td>-2.039154e+08</td>
      <td>-3.725953e+08</td>
    </tr>
  </tbody>
</table>
</div>




```python
future_datest_df=pd.DataFrame(index=future_dates[1:],columns=salesbymonth.columns)
```


```python
future_datest_df.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>amount</th>
      <th>1st diff</th>
      <th>2nd diff</th>
      <th>3rd diff</th>
      <th>seasonal diff</th>
      <th>seasonal 1st diff</th>
      <th>seasonal 2nd diff</th>
      <th>seasonal 3rd diff</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2019-03-28</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2019-04-28</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2019-05-28</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2019-06-28</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2019-07-28</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
future_df=pd.concat([salesbymonth,future_datest_df])
future_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>amount</th>
      <th>1st diff</th>
      <th>2nd diff</th>
      <th>3rd diff</th>
      <th>seasonal diff</th>
      <th>seasonal 1st diff</th>
      <th>seasonal 2nd diff</th>
      <th>seasonal 3rd diff</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-06-30</th>
      <td>1.565927e+09</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2016-07-31</th>
      <td>1.631030e+09</td>
      <td>6.510308e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2016-08-31</th>
      <td>1.631913e+09</td>
      <td>8.831253e+05</td>
      <td>-6.421996e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2016-09-30</th>
      <td>1.616485e+09</td>
      <td>-1.542843e+07</td>
      <td>-1.631156e+07</td>
      <td>4.790840e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2016-10-31</th>
      <td>1.770040e+09</td>
      <td>1.535552e+08</td>
      <td>1.689836e+08</td>
      <td>1.852952e+08</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2016-11-30</th>
      <td>1.789135e+09</td>
      <td>1.909530e+07</td>
      <td>-1.344599e+08</td>
      <td>-3.034435e+08</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2016-12-31</th>
      <td>1.993133e+09</td>
      <td>2.039973e+08</td>
      <td>1.849020e+08</td>
      <td>3.193619e+08</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-01-31</th>
      <td>1.787146e+09</td>
      <td>-2.059864e+08</td>
      <td>-4.099837e+08</td>
      <td>-5.948857e+08</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-02-28</th>
      <td>1.824069e+09</td>
      <td>3.692261e+07</td>
      <td>2.429090e+08</td>
      <td>6.528928e+08</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-03-31</th>
      <td>2.144957e+09</td>
      <td>3.208885e+08</td>
      <td>2.839659e+08</td>
      <td>4.105683e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-04-30</th>
      <td>2.144388e+09</td>
      <td>-5.693449e+05</td>
      <td>-3.214578e+08</td>
      <td>-6.054237e+08</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-05-31</th>
      <td>2.316093e+09</td>
      <td>1.717051e+08</td>
      <td>1.722745e+08</td>
      <td>4.937323e+08</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-06-30</th>
      <td>2.266990e+09</td>
      <td>-4.910315e+07</td>
      <td>-2.208083e+08</td>
      <td>-3.930827e+08</td>
      <td>7.010628e+08</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-07-31</th>
      <td>2.284421e+09</td>
      <td>1.743098e+07</td>
      <td>6.653413e+07</td>
      <td>2.873424e+08</td>
      <td>6.533907e+08</td>
      <td>-4.767210e+07</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-08-31</th>
      <td>2.208400e+09</td>
      <td>-7.602059e+07</td>
      <td>-9.345157e+07</td>
      <td>-1.599857e+08</td>
      <td>5.764870e+08</td>
      <td>-7.690372e+07</td>
      <td>-2.923162e+07</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-09-30</th>
      <td>2.278157e+09</td>
      <td>6.975645e+07</td>
      <td>1.457770e+08</td>
      <td>2.392286e+08</td>
      <td>6.616719e+08</td>
      <td>8.518488e+07</td>
      <td>1.620886e+08</td>
      <td>1.913202e+08</td>
    </tr>
    <tr>
      <th>2017-10-31</th>
      <td>2.081597e+09</td>
      <td>-1.965599e+08</td>
      <td>-2.663164e+08</td>
      <td>-4.120934e+08</td>
      <td>3.115568e+08</td>
      <td>-3.501151e+08</td>
      <td>-4.353000e+08</td>
      <td>-5.973886e+08</td>
    </tr>
    <tr>
      <th>2017-11-30</th>
      <td>2.188137e+09</td>
      <td>1.065405e+08</td>
      <td>3.031004e+08</td>
      <td>5.694168e+08</td>
      <td>3.990019e+08</td>
      <td>8.744519e+07</td>
      <td>4.375603e+08</td>
      <td>8.728603e+08</td>
    </tr>
    <tr>
      <th>2017-12-31</th>
      <td>2.312373e+09</td>
      <td>1.242353e+08</td>
      <td>1.769483e+07</td>
      <td>-2.854056e+08</td>
      <td>3.192400e+08</td>
      <td>-7.976197e+07</td>
      <td>-1.672072e+08</td>
      <td>-6.047675e+08</td>
    </tr>
    <tr>
      <th>2018-01-31</th>
      <td>2.114483e+09</td>
      <td>-1.978897e+08</td>
      <td>-3.221251e+08</td>
      <td>-3.398199e+08</td>
      <td>3.273367e+08</td>
      <td>8.096684e+06</td>
      <td>8.785865e+07</td>
      <td>2.550658e+08</td>
    </tr>
    <tr>
      <th>2018-02-28</th>
      <td>1.953435e+09</td>
      <td>-1.610476e+08</td>
      <td>3.684213e+07</td>
      <td>3.589672e+08</td>
      <td>1.293664e+08</td>
      <td>-1.979702e+08</td>
      <td>-2.060669e+08</td>
      <td>-2.939256e+08</td>
    </tr>
    <tr>
      <th>2018-03-31</th>
      <td>2.356962e+09</td>
      <td>4.035267e+08</td>
      <td>5.645743e+08</td>
      <td>5.277321e+08</td>
      <td>2.120046e+08</td>
      <td>8.263818e+07</td>
      <td>2.806084e+08</td>
      <td>4.866753e+08</td>
    </tr>
    <tr>
      <th>2018-04-30</th>
      <td>2.280303e+09</td>
      <td>-7.665848e+07</td>
      <td>-4.801851e+08</td>
      <td>-1.044759e+09</td>
      <td>1.359155e+08</td>
      <td>-7.608913e+07</td>
      <td>-1.587273e+08</td>
      <td>-4.393357e+08</td>
    </tr>
    <tr>
      <th>2018-05-31</th>
      <td>2.321865e+09</td>
      <td>4.156173e+07</td>
      <td>1.182202e+08</td>
      <td>5.984053e+08</td>
      <td>5.772091e+06</td>
      <td>-1.301434e+08</td>
      <td>-5.405426e+07</td>
      <td>1.046731e+08</td>
    </tr>
    <tr>
      <th>2018-06-30</th>
      <td>2.194311e+09</td>
      <td>-1.275543e+08</td>
      <td>-1.691161e+08</td>
      <td>-2.873363e+08</td>
      <td>-7.267911e+07</td>
      <td>-7.845120e+07</td>
      <td>5.169219e+07</td>
      <td>1.057464e+08</td>
    </tr>
    <tr>
      <th>2018-07-31</th>
      <td>2.241846e+09</td>
      <td>4.753548e+07</td>
      <td>1.750898e+08</td>
      <td>3.442059e+08</td>
      <td>-4.257460e+07</td>
      <td>3.010450e+07</td>
      <td>1.085557e+08</td>
      <td>5.686351e+07</td>
    </tr>
    <tr>
      <th>2018-08-31</th>
      <td>2.174102e+09</td>
      <td>-6.774396e+07</td>
      <td>-1.152794e+08</td>
      <td>-2.903693e+08</td>
      <td>-3.429797e+07</td>
      <td>8.276632e+06</td>
      <td>-2.182787e+07</td>
      <td>-1.303836e+08</td>
    </tr>
    <tr>
      <th>2018-09-30</th>
      <td>2.129888e+09</td>
      <td>-4.421403e+07</td>
      <td>2.352993e+07</td>
      <td>1.388094e+08</td>
      <td>-1.482685e+08</td>
      <td>-1.139705e+08</td>
      <td>-1.222471e+08</td>
      <td>-1.004192e+08</td>
    </tr>
    <tr>
      <th>2018-10-31</th>
      <td>2.279253e+09</td>
      <td>1.493645e+08</td>
      <td>1.935785e+08</td>
      <td>1.700486e+08</td>
      <td>1.976559e+08</td>
      <td>3.459244e+08</td>
      <td>4.598949e+08</td>
      <td>5.821420e+08</td>
    </tr>
    <tr>
      <th>2018-11-30</th>
      <td>2.208414e+09</td>
      <td>-7.083914e+07</td>
      <td>-2.202036e+08</td>
      <td>-4.137821e+08</td>
      <td>2.027632e+07</td>
      <td>-1.773796e+08</td>
      <td>-5.233040e+08</td>
      <td>-9.831989e+08</td>
    </tr>
    <tr>
      <th>2018-12-31</th>
      <td>2.263730e+09</td>
      <td>5.531690e+07</td>
      <td>1.261560e+08</td>
      <td>3.463596e+08</td>
      <td>-4.864209e+07</td>
      <td>-6.891842e+07</td>
      <td>1.084612e+08</td>
      <td>6.317652e+08</td>
    </tr>
    <tr>
      <th>2019-01-31</th>
      <td>2.165602e+09</td>
      <td>-9.812826e+07</td>
      <td>-1.534452e+08</td>
      <td>-2.796012e+08</td>
      <td>5.111939e+07</td>
      <td>9.976149e+07</td>
      <td>1.686799e+08</td>
      <td>6.021870e+07</td>
    </tr>
    <tr>
      <th>2019-02-28</th>
      <td>1.900401e+09</td>
      <td>-2.652015e+08</td>
      <td>-1.670732e+08</td>
      <td>-1.362808e+07</td>
      <td>-5.303449e+07</td>
      <td>-1.041539e+08</td>
      <td>-2.039154e+08</td>
      <td>-3.725953e+08</td>
    </tr>
    <tr>
      <th>2019-03-28</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2019-04-28</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2019-05-28</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2019-06-28</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2019-07-28</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
len(future_df['amount'].index)
```




    38




```python
forecast = result.forecast(steps=5)
forecast
# 예측값, stderr, upper bound, lower bound 
```




    (array([1.91085303e+09, 1.92130533e+09, 1.93175763e+09, 1.94220993e+09,
            1.95266223e+09]),
     array([1.47110102e+08, 2.08045102e+08, 2.54802172e+08, 2.94220205e+08,
            3.28948189e+08]),
     array([[1.62252252e+09, 2.19918353e+09],
            [1.51354442e+09, 2.32906623e+09],
            [1.43235455e+09, 2.43116071e+09],
            [1.36554893e+09, 2.51887094e+09],
            [1.30793563e+09, 2.59738884e+09]]))




```python
forecast[0]
```




    array([1.91085303e+09, 1.92130533e+09, 1.93175763e+09, 1.94220993e+09,
           1.95266223e+09])




```python
forecast2 = future_df.loc['2019-03-28':]
```


```python
forecast2
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>amount</th>
      <th>1st diff</th>
      <th>2nd diff</th>
      <th>3rd diff</th>
      <th>seasonal diff</th>
      <th>seasonal 1st diff</th>
      <th>seasonal 2nd diff</th>
      <th>seasonal 3rd diff</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2019-03-28</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2019-04-28</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2019-05-28</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2019-06-28</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2019-07-28</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
future_forecast = pd.DataFrame(forecast[0],index = forecast2.index ,columns=['prediction'])#예측값을 기존 df에 합쳐서 넣을 때
```


```python
pd.concat([future_df['amount'],future_forecast],axis=1).plot()#그래프
```




    <AxesSubplot:>

![funda19_19]({{ site.url }}{{ site.baseurl }}/assets/images/funda19 (19).png)


```python
# predict함수로 예측할 때 
result.plot_predict(1,38)
```
