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

핀테크 기업인 ‘FUNDA(펀다)’는 상환 기간의 매출을 예측하여 신용 점수가 낮거나 담보를 가지지 못하는 우수 상점들에 금융 기회를 제공하려 합니다.
이번 대회에서는 2년 전 부터 2019년 2월 28일까지의 카드 거래 데이터를 이용해 2019-03-01부터 2019-05-31까지의 각 상점별 3개월 총 매출을 예측하는 것입니다.

![funda00]{{ site.url }}{{ site.baseurl }}/assets/images/funda0.png)
![funda01]{{ site.url }}{{ site.baseurl }}/assets/images/funda1.png)


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


![funda03]{{ site.url }}{{ site.baseurl }}/assets/images/funda3.png)


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

![funda03_1]{{ site.url }}{{ site.baseurl }}/assets/images/funda3_1.png)
    

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


![funda04]{{ site.url }}{{ site.baseurl }}/assets/images/funda4.png)


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



![funda02]{{ site.url }}{{ site.baseurl }}/assets/images/funda2.png)

    


## a(=1)회사가 이상치제일 많고 중앙값 최대값 총 매출에 가장 많이 기여한다 이상치는 스케일러가 필요하다 ##




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


![funda14]{{ site.url }}{{ site.baseurl }}/assets/images/funda14.png)

    


## 할부개월 수가 적을 수록 총매출이 크며 이상치도 제일 크다 (1,3,5,6,10, 12달 이상치 스케일러가 필요##


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


![funda15]{{ site.url }}{{ site.baseurl }}/assets/images/funda15.png)




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

