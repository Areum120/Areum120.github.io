```python
---
title: "BMW Project: BMW SUV VS Sedan 모델 별 판매량 EDA 분석 Report"
date: 2021-07-25
read_time: false
comments: false
share: false
related: false
categories:
  - BMW Project
  - Uncategorized 
tags:
   - BMW
   - data analysis
   - matplotlib
   - pandas
---
```


```python
# 데이터 분석
import numpy as np
import pandas as pd
import os
import json
import pickle

# 시각화
import matplotlib as mpl
import matplotlib.font_manager as fm
import matplotlib.pyplot as plt
import seaborn as sns
import folium
from folium import plugins
from folium.features import CustomIcon
import googlemaps

# 한글 폰트 사용
font_name = fm.FontProperties(fname="C:/Windows/Fonts/malgun.ttf").get_name()
plt.rc('font', family=font_name)

# 경고창 무시
import warnings
warnings.filterwarnings(action='ignore')
```

이번에는 BMW 모델별로 어떤 모델이 가장 많이 팔렸는지 2017년 부터 2021년 5월까지 누적판매량 순위를 EDA를 살펴볼 것이다
모델 중 SUV가 요즘 수입차와 국내차 브랜드를 가리지 않고 많이 팔리는 추세인데 SUV와 세단의 판매 추이와
연간 차량 등록 수 대비 BMW SUV 판매량은 어느정도 차지하는지 EDA로 함께 살펴볼 예정이다

## 2017-2021(1-5월) BMW 임포터 & 딜러사 동향 분석 Report 

   - BMW 모델 별 누적판매량 순위 
   - BMW SUV Vs Sedan 연간 판매 추이비교
   - 연간 차량 등록 수 및 BMW SUV 판매추이 비교

### 연간 수입차브랜드 판매량 데이터 불러오기


```python
df = pd.read_csv('./17_21_6brand_year_sales.csv', encoding='utf-8')
df
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
      <th>Unnamed: 0</th>
      <th>brand</th>
      <th>year</th>
      <th>car_name</th>
      <th>sub_name</th>
      <th>quentity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>BMW</td>
      <td>2017</td>
      <td>5 Series</td>
      <td>520d</td>
      <td>10,249</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>BMW</td>
      <td>2017</td>
      <td>5 Series</td>
      <td>520d xDrive</td>
      <td>5,712</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>BMW</td>
      <td>2017</td>
      <td>5 Series</td>
      <td>530i xDrive</td>
      <td>4,690</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>BMW</td>
      <td>2017</td>
      <td>5 Series</td>
      <td>530i</td>
      <td>4,560</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>BMW</td>
      <td>2017</td>
      <td>5 Series</td>
      <td>530d</td>
      <td>569</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1710</th>
      <td>150</td>
      <td>Volvo</td>
      <td>2021</td>
      <td>V90 Cross Country</td>
      <td>B5 AWD</td>
      <td>205</td>
    </tr>
    <tr>
      <th>1711</th>
      <td>151</td>
      <td>Volvo</td>
      <td>2021</td>
      <td>V90 Cross Country</td>
      <td>B6 AWD</td>
      <td>7</td>
    </tr>
    <tr>
      <th>1712</th>
      <td>152</td>
      <td>Volvo</td>
      <td>2021</td>
      <td>V90 Cross Country</td>
      <td>B6 AWD</td>
      <td>7</td>
    </tr>
    <tr>
      <th>1713</th>
      <td>153</td>
      <td>Volvo</td>
      <td>2021</td>
      <td>S90</td>
      <td>2.0 T8 AWD</td>
      <td>53</td>
    </tr>
    <tr>
      <th>1714</th>
      <td>154</td>
      <td>Volvo</td>
      <td>2021</td>
      <td>S90</td>
      <td>2.0 T8 AWD</td>
      <td>53</td>
    </tr>
  </tbody>
</table>
<p>1715 rows × 6 columns</p>
</div>



### 데이터 전처리


```python
# 칼럼 선택
df = df[['brand', 'year', 'car_name', 'sub_name', 'quentity']]
df
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
      <th>brand</th>
      <th>year</th>
      <th>car_name</th>
      <th>sub_name</th>
      <th>quentity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>BMW</td>
      <td>2017</td>
      <td>5 Series</td>
      <td>520d</td>
      <td>10,249</td>
    </tr>
    <tr>
      <th>1</th>
      <td>BMW</td>
      <td>2017</td>
      <td>5 Series</td>
      <td>520d xDrive</td>
      <td>5,712</td>
    </tr>
    <tr>
      <th>2</th>
      <td>BMW</td>
      <td>2017</td>
      <td>5 Series</td>
      <td>530i xDrive</td>
      <td>4,690</td>
    </tr>
    <tr>
      <th>3</th>
      <td>BMW</td>
      <td>2017</td>
      <td>5 Series</td>
      <td>530i</td>
      <td>4,560</td>
    </tr>
    <tr>
      <th>4</th>
      <td>BMW</td>
      <td>2017</td>
      <td>5 Series</td>
      <td>530d</td>
      <td>569</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1710</th>
      <td>Volvo</td>
      <td>2021</td>
      <td>V90 Cross Country</td>
      <td>B5 AWD</td>
      <td>205</td>
    </tr>
    <tr>
      <th>1711</th>
      <td>Volvo</td>
      <td>2021</td>
      <td>V90 Cross Country</td>
      <td>B6 AWD</td>
      <td>7</td>
    </tr>
    <tr>
      <th>1712</th>
      <td>Volvo</td>
      <td>2021</td>
      <td>V90 Cross Country</td>
      <td>B6 AWD</td>
      <td>7</td>
    </tr>
    <tr>
      <th>1713</th>
      <td>Volvo</td>
      <td>2021</td>
      <td>S90</td>
      <td>2.0 T8 AWD</td>
      <td>53</td>
    </tr>
    <tr>
      <th>1714</th>
      <td>Volvo</td>
      <td>2021</td>
      <td>S90</td>
      <td>2.0 T8 AWD</td>
      <td>53</td>
    </tr>
  </tbody>
</table>
<p>1715 rows × 5 columns</p>
</div>




```python
# 콤마제거, 정수변환
df['quentity'] = df.quentity.str.replace(',', '').astype('int64')
```


```python
#Benz
MB = df.loc[df['brand'] == 'MB']
```


```python
#Benz 판매량
MB_quentity = MB.groupby(['year'])[['quentity']].sum()
MB_quentity
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
      <th>quentity</th>
    </tr>
    <tr>
      <th>year</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2017</th>
      <td>77873</td>
    </tr>
    <tr>
      <th>2018</th>
      <td>80525</td>
    </tr>
    <tr>
      <th>2019</th>
      <td>86924</td>
    </tr>
    <tr>
      <th>2020</th>
      <td>81349</td>
    </tr>
    <tr>
      <th>2021</th>
      <td>37022</td>
    </tr>
  </tbody>
</table>
</div>




```python
#BMW
BMW = df.loc[df['brand'] == 'BMW']
```


```python
# BMW SUV 모델 연간 판매량 증가 추이

BMWYear = BMW.groupby(['year','car_name'])[['quentity']].sum()
BMWYear 
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
      <th></th>
      <th>quentity</th>
    </tr>
    <tr>
      <th>year</th>
      <th>car_name</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="5" valign="top">2017</th>
      <th>1 Series</th>
      <td>7726</td>
    </tr>
    <tr>
      <th>2 Series Active Tourer</th>
      <td>1900</td>
    </tr>
    <tr>
      <th>3 Series</th>
      <td>12608</td>
    </tr>
    <tr>
      <th>4 Series</th>
      <td>2723</td>
    </tr>
    <tr>
      <th>5 Series</th>
      <td>26892</td>
    </tr>
    <tr>
      <th>...</th>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th rowspan="5" valign="top">2021</th>
      <th>X6</th>
      <td>1793</td>
    </tr>
    <tr>
      <th>X6 M</th>
      <td>132</td>
    </tr>
    <tr>
      <th>X7</th>
      <td>1701</td>
    </tr>
    <tr>
      <th>Z4</th>
      <td>226</td>
    </tr>
    <tr>
      <th>i3</th>
      <td>130</td>
    </tr>
  </tbody>
</table>
<p>120 rows × 1 columns</p>
</div>




```python
BMWYear = BMWYear.reset_index()
```


```python
BMW_quentity = BMW.groupby(['year'])[['quentity']].sum()
BMW_quentity
```

### BMW SUV, Sedan 데이터 전처리


```python
# sedan
BMWSedanYear = BMWYear[BMWYear['car_name'].isin(['5 Series','3 Series','1 Series','7 Series','6 Series','4 Series','2 Series Active Tourer','Gran Turismo','M','2 Series'
,'M5','Z4','M2','8 Series','New 4 Series','M4','i8','M3','M8','The new M3','The new M4 '])]
BMWSedanYear
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
      <th>year</th>
      <th>car_name</th>
      <th>quentity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2017</td>
      <td>1 Series</td>
      <td>7726</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2017</td>
      <td>2 Series Active Tourer</td>
      <td>1900</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2017</td>
      <td>3 Series</td>
      <td>12608</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2017</td>
      <td>4 Series</td>
      <td>2723</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017</td>
      <td>5 Series</td>
      <td>26892</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>102</th>
      <td>2021</td>
      <td>M5</td>
      <td>150</td>
    </tr>
    <tr>
      <th>103</th>
      <td>2021</td>
      <td>M8</td>
      <td>18</td>
    </tr>
    <tr>
      <th>104</th>
      <td>2021</td>
      <td>New 4 Series</td>
      <td>537</td>
    </tr>
    <tr>
      <th>105</th>
      <td>2021</td>
      <td>The new M3</td>
      <td>72</td>
    </tr>
    <tr>
      <th>118</th>
      <td>2021</td>
      <td>Z4</td>
      <td>226</td>
    </tr>
  </tbody>
</table>
<p>72 rows × 3 columns</p>
</div>




```python
BMWSedanYearQuentity = BMWSedanYear.groupby(['year'])[['quentity']].sum()
BMWSedanYearQuentity
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
      <th>quentity</th>
    </tr>
    <tr>
      <th>year</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2017</th>
      <td>59492</td>
    </tr>
    <tr>
      <th>2018</th>
      <td>48226</td>
    </tr>
    <tr>
      <th>2019</th>
      <td>39470</td>
    </tr>
    <tr>
      <th>2020</th>
      <td>42709</td>
    </tr>
    <tr>
      <th>2021</th>
      <td>20711</td>
    </tr>
  </tbody>
</table>
</div>




```python
BMWSedanYearQuentity = BMWSedanYearQuentity.reset_index()
BMWSedanYearQuentity
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
      <th>year</th>
      <th>quentity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2017</td>
      <td>59492</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2018</td>
      <td>48226</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2019</td>
      <td>39470</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2020</td>
      <td>42709</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2021</td>
      <td>20711</td>
    </tr>
  </tbody>
</table>
</div>




```python
# suv 특정 행값만 가져오기
BMWSuvYear = BMWYear[BMWYear['car_name'].isin(['X5', 'X3', 'X4', 'X6', 'X1', 'X7', 'X2', 'i3', 'X3 M', 'X4 M', 'X5 M', 'X6 M'])]
BMWSuvYear
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
      <th>year</th>
      <th>car_name</th>
      <th>quentity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>9</th>
      <td>2017</td>
      <td>X1</td>
      <td>1530</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2017</td>
      <td>X3</td>
      <td>1917</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2017</td>
      <td>X4</td>
      <td>2854</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2017</td>
      <td>X5</td>
      <td>3343</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2017</td>
      <td>X6</td>
      <td>2607</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2017</td>
      <td>i3</td>
      <td>386</td>
    </tr>
    <tr>
      <th>30</th>
      <td>2018</td>
      <td>X1</td>
      <td>842</td>
    </tr>
    <tr>
      <th>31</th>
      <td>2018</td>
      <td>X2</td>
      <td>108</td>
    </tr>
    <tr>
      <th>32</th>
      <td>2018</td>
      <td>X3</td>
      <td>2701</td>
    </tr>
    <tr>
      <th>33</th>
      <td>2018</td>
      <td>X4</td>
      <td>1156</td>
    </tr>
    <tr>
      <th>34</th>
      <td>2018</td>
      <td>X5</td>
      <td>1959</td>
    </tr>
    <tr>
      <th>35</th>
      <td>2018</td>
      <td>X6</td>
      <td>1775</td>
    </tr>
    <tr>
      <th>36</th>
      <td>2018</td>
      <td>i3</td>
      <td>382</td>
    </tr>
    <tr>
      <th>51</th>
      <td>2019</td>
      <td>X1</td>
      <td>1509</td>
    </tr>
    <tr>
      <th>52</th>
      <td>2019</td>
      <td>X2</td>
      <td>765</td>
    </tr>
    <tr>
      <th>53</th>
      <td>2019</td>
      <td>X3</td>
      <td>2407</td>
    </tr>
    <tr>
      <th>54</th>
      <td>2019</td>
      <td>X3 M</td>
      <td>112</td>
    </tr>
    <tr>
      <th>55</th>
      <td>2019</td>
      <td>X4</td>
      <td>1786</td>
    </tr>
    <tr>
      <th>56</th>
      <td>2019</td>
      <td>X4 M</td>
      <td>106</td>
    </tr>
    <tr>
      <th>57</th>
      <td>2019</td>
      <td>X5</td>
      <td>2461</td>
    </tr>
    <tr>
      <th>58</th>
      <td>2019</td>
      <td>X6</td>
      <td>1087</td>
    </tr>
    <tr>
      <th>59</th>
      <td>2019</td>
      <td>X7</td>
      <td>522</td>
    </tr>
    <tr>
      <th>61</th>
      <td>2019</td>
      <td>i3</td>
      <td>508</td>
    </tr>
    <tr>
      <th>76</th>
      <td>2020</td>
      <td>X1</td>
      <td>1591</td>
    </tr>
    <tr>
      <th>77</th>
      <td>2020</td>
      <td>X2</td>
      <td>547</td>
    </tr>
    <tr>
      <th>78</th>
      <td>2020</td>
      <td>X3</td>
      <td>3688</td>
    </tr>
    <tr>
      <th>79</th>
      <td>2020</td>
      <td>X3 M</td>
      <td>294</td>
    </tr>
    <tr>
      <th>80</th>
      <td>2020</td>
      <td>X4</td>
      <td>3732</td>
    </tr>
    <tr>
      <th>81</th>
      <td>2020</td>
      <td>X4 M</td>
      <td>310</td>
    </tr>
    <tr>
      <th>82</th>
      <td>2020</td>
      <td>X5</td>
      <td>3638</td>
    </tr>
    <tr>
      <th>83</th>
      <td>2020</td>
      <td>X5 M</td>
      <td>86</td>
    </tr>
    <tr>
      <th>84</th>
      <td>2020</td>
      <td>X6</td>
      <td>2645</td>
    </tr>
    <tr>
      <th>85</th>
      <td>2020</td>
      <td>X6 M</td>
      <td>160</td>
    </tr>
    <tr>
      <th>86</th>
      <td>2020</td>
      <td>X7</td>
      <td>2783</td>
    </tr>
    <tr>
      <th>88</th>
      <td>2020</td>
      <td>i3</td>
      <td>304</td>
    </tr>
    <tr>
      <th>107</th>
      <td>2021</td>
      <td>X1</td>
      <td>1417</td>
    </tr>
    <tr>
      <th>108</th>
      <td>2021</td>
      <td>X2</td>
      <td>322</td>
    </tr>
    <tr>
      <th>109</th>
      <td>2021</td>
      <td>X3</td>
      <td>2048</td>
    </tr>
    <tr>
      <th>110</th>
      <td>2021</td>
      <td>X3 M</td>
      <td>100</td>
    </tr>
    <tr>
      <th>111</th>
      <td>2021</td>
      <td>X4</td>
      <td>1433</td>
    </tr>
    <tr>
      <th>112</th>
      <td>2021</td>
      <td>X4 M</td>
      <td>78</td>
    </tr>
    <tr>
      <th>113</th>
      <td>2021</td>
      <td>X5</td>
      <td>2326</td>
    </tr>
    <tr>
      <th>114</th>
      <td>2021</td>
      <td>X5 M</td>
      <td>88</td>
    </tr>
    <tr>
      <th>115</th>
      <td>2021</td>
      <td>X6</td>
      <td>1793</td>
    </tr>
    <tr>
      <th>116</th>
      <td>2021</td>
      <td>X6 M</td>
      <td>132</td>
    </tr>
    <tr>
      <th>117</th>
      <td>2021</td>
      <td>X7</td>
      <td>1701</td>
    </tr>
    <tr>
      <th>119</th>
      <td>2021</td>
      <td>i3</td>
      <td>130</td>
    </tr>
  </tbody>
</table>
</div>




```python
BMWSuvYearQuentity = BMWSuvYear.groupby(['year','car_name'])[['quentity']].sum()
BMWSuvYearQuentity
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
      <th></th>
      <th>quentity</th>
    </tr>
    <tr>
      <th>year</th>
      <th>car_name</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="6" valign="top">2017</th>
      <th>X1</th>
      <td>1530</td>
    </tr>
    <tr>
      <th>X3</th>
      <td>1917</td>
    </tr>
    <tr>
      <th>X4</th>
      <td>2854</td>
    </tr>
    <tr>
      <th>X5</th>
      <td>3343</td>
    </tr>
    <tr>
      <th>X6</th>
      <td>2607</td>
    </tr>
    <tr>
      <th>i3</th>
      <td>386</td>
    </tr>
    <tr>
      <th rowspan="7" valign="top">2018</th>
      <th>X1</th>
      <td>842</td>
    </tr>
    <tr>
      <th>X2</th>
      <td>108</td>
    </tr>
    <tr>
      <th>X3</th>
      <td>2701</td>
    </tr>
    <tr>
      <th>X4</th>
      <td>1156</td>
    </tr>
    <tr>
      <th>X5</th>
      <td>1959</td>
    </tr>
    <tr>
      <th>X6</th>
      <td>1775</td>
    </tr>
    <tr>
      <th>i3</th>
      <td>382</td>
    </tr>
    <tr>
      <th rowspan="10" valign="top">2019</th>
      <th>X1</th>
      <td>1509</td>
    </tr>
    <tr>
      <th>X2</th>
      <td>765</td>
    </tr>
    <tr>
      <th>X3</th>
      <td>2407</td>
    </tr>
    <tr>
      <th>X3 M</th>
      <td>112</td>
    </tr>
    <tr>
      <th>X4</th>
      <td>1786</td>
    </tr>
    <tr>
      <th>X4 M</th>
      <td>106</td>
    </tr>
    <tr>
      <th>X5</th>
      <td>2461</td>
    </tr>
    <tr>
      <th>X6</th>
      <td>1087</td>
    </tr>
    <tr>
      <th>X7</th>
      <td>522</td>
    </tr>
    <tr>
      <th>i3</th>
      <td>508</td>
    </tr>
    <tr>
      <th rowspan="12" valign="top">2020</th>
      <th>X1</th>
      <td>1591</td>
    </tr>
    <tr>
      <th>X2</th>
      <td>547</td>
    </tr>
    <tr>
      <th>X3</th>
      <td>3688</td>
    </tr>
    <tr>
      <th>X3 M</th>
      <td>294</td>
    </tr>
    <tr>
      <th>X4</th>
      <td>3732</td>
    </tr>
    <tr>
      <th>X4 M</th>
      <td>310</td>
    </tr>
    <tr>
      <th>X5</th>
      <td>3638</td>
    </tr>
    <tr>
      <th>X5 M</th>
      <td>86</td>
    </tr>
    <tr>
      <th>X6</th>
      <td>2645</td>
    </tr>
    <tr>
      <th>X6 M</th>
      <td>160</td>
    </tr>
    <tr>
      <th>X7</th>
      <td>2783</td>
    </tr>
    <tr>
      <th>i3</th>
      <td>304</td>
    </tr>
    <tr>
      <th rowspan="12" valign="top">2021</th>
      <th>X1</th>
      <td>1417</td>
    </tr>
    <tr>
      <th>X2</th>
      <td>322</td>
    </tr>
    <tr>
      <th>X3</th>
      <td>2048</td>
    </tr>
    <tr>
      <th>X3 M</th>
      <td>100</td>
    </tr>
    <tr>
      <th>X4</th>
      <td>1433</td>
    </tr>
    <tr>
      <th>X4 M</th>
      <td>78</td>
    </tr>
    <tr>
      <th>X5</th>
      <td>2326</td>
    </tr>
    <tr>
      <th>X5 M</th>
      <td>88</td>
    </tr>
    <tr>
      <th>X6</th>
      <td>1793</td>
    </tr>
    <tr>
      <th>X6 M</th>
      <td>132</td>
    </tr>
    <tr>
      <th>X7</th>
      <td>1701</td>
    </tr>
    <tr>
      <th>i3</th>
      <td>130</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 두개의 그룹의 경우 피봇테이블로 변환
BMWSuvYearQuentity = BMWSuvYearQuentity.reset_index()
BMWSuvYearQuentity_pivot = BMWSuvYearQuentity.pivot(index='year',columns='car_name', values='quentity')
```


```python
BMWSuvYearQuentity_pivot
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
      <th>car_name</th>
      <th>X1</th>
      <th>X2</th>
      <th>X3</th>
      <th>X3 M</th>
      <th>X4</th>
      <th>X4 M</th>
      <th>X5</th>
      <th>X5 M</th>
      <th>X6</th>
      <th>X6 M</th>
      <th>X7</th>
      <th>i3</th>
    </tr>
    <tr>
      <th>year</th>
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
      <th>2017</th>
      <td>1530.0</td>
      <td>NaN</td>
      <td>1917.0</td>
      <td>NaN</td>
      <td>2854.0</td>
      <td>NaN</td>
      <td>3343.0</td>
      <td>NaN</td>
      <td>2607.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>386.0</td>
    </tr>
    <tr>
      <th>2018</th>
      <td>842.0</td>
      <td>108.0</td>
      <td>2701.0</td>
      <td>NaN</td>
      <td>1156.0</td>
      <td>NaN</td>
      <td>1959.0</td>
      <td>NaN</td>
      <td>1775.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>382.0</td>
    </tr>
    <tr>
      <th>2019</th>
      <td>1509.0</td>
      <td>765.0</td>
      <td>2407.0</td>
      <td>112.0</td>
      <td>1786.0</td>
      <td>106.0</td>
      <td>2461.0</td>
      <td>NaN</td>
      <td>1087.0</td>
      <td>NaN</td>
      <td>522.0</td>
      <td>508.0</td>
    </tr>
    <tr>
      <th>2020</th>
      <td>1591.0</td>
      <td>547.0</td>
      <td>3688.0</td>
      <td>294.0</td>
      <td>3732.0</td>
      <td>310.0</td>
      <td>3638.0</td>
      <td>86.0</td>
      <td>2645.0</td>
      <td>160.0</td>
      <td>2783.0</td>
      <td>304.0</td>
    </tr>
    <tr>
      <th>2021</th>
      <td>1417.0</td>
      <td>322.0</td>
      <td>2048.0</td>
      <td>100.0</td>
      <td>1433.0</td>
      <td>78.0</td>
      <td>2326.0</td>
      <td>88.0</td>
      <td>1793.0</td>
      <td>132.0</td>
      <td>1701.0</td>
      <td>130.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
BMWSuvYearQuentity2 = BMWSuvYear.groupby(['year'])[['quentity']].sum()
BMWSuvYearQuentity2
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
      <th>quentity</th>
    </tr>
    <tr>
      <th>year</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2017</th>
      <td>12637</td>
    </tr>
    <tr>
      <th>2018</th>
      <td>8923</td>
    </tr>
    <tr>
      <th>2019</th>
      <td>11263</td>
    </tr>
    <tr>
      <th>2020</th>
      <td>19778</td>
    </tr>
    <tr>
      <th>2021</th>
      <td>11568</td>
    </tr>
  </tbody>
</table>
</div>




```python
BMWSuvYearQuentity2 = BMWSuvYearQuentity2.reset_index()
BMWSuvYearQuentity2
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
      <th>year</th>
      <th>quentity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2017</td>
      <td>12637</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2018</td>
      <td>8923</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2019</td>
      <td>11263</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2020</td>
      <td>19778</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2021</td>
      <td>11568</td>
    </tr>
  </tbody>
</table>
</div>




```python
BMWSuvSedanYear = pd.merge(BMWSedanYearQuentity, BMWSuvYearQuentity2, how = 'outer', on='year')
BMWSuvSedanYear               
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
      <th>year</th>
      <th>quentity_x</th>
      <th>quentity_y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2017</td>
      <td>59492</td>
      <td>12637</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2018</td>
      <td>48226</td>
      <td>8923</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2019</td>
      <td>39470</td>
      <td>11263</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2020</td>
      <td>42709</td>
      <td>19778</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2021</td>
      <td>20711</td>
      <td>11568</td>
    </tr>
  </tbody>
</table>
</div>




```python
BMWSuvSedanYear.columns = ['year', 'BMW_Sedan', 'BMW_SUV']
BMWSuvSedanYear
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
      <th>year</th>
      <th>BMW_Sedan</th>
      <th>BMW_SUV</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2017</td>
      <td>59492</td>
      <td>12637</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2018</td>
      <td>48226</td>
      <td>8923</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2019</td>
      <td>39470</td>
      <td>11263</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2020</td>
      <td>42709</td>
      <td>19778</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2021</td>
      <td>20711</td>
      <td>11568</td>
    </tr>
  </tbody>
</table>
</div>



### BMW 모델별 누적 판매량 데이터 전처리


```python
# 2017~2021(1월~5월)총 누적 판매 순위
all_BMW_model_quentity = BMW.groupby(['car_name'])[['quentity']].sum()
all_BMW_model_quentity
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
      <th>quentity</th>
    </tr>
    <tr>
      <th>car_name</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1 Series</th>
      <td>18338</td>
    </tr>
    <tr>
      <th>2 Series</th>
      <td>1294</td>
    </tr>
    <tr>
      <th>2 Series Active Tourer</th>
      <td>7002</td>
    </tr>
    <tr>
      <th>3 Series</th>
      <td>42392</td>
    </tr>
    <tr>
      <th>4 Series</th>
      <td>8030</td>
    </tr>
    <tr>
      <th>5 Series</th>
      <td>101035</td>
    </tr>
    <tr>
      <th>6 Series</th>
      <td>10243</td>
    </tr>
    <tr>
      <th>7 Series</th>
      <td>11762</td>
    </tr>
    <tr>
      <th>8 Series</th>
      <td>633</td>
    </tr>
    <tr>
      <th>Gran Turismo</th>
      <td>2934</td>
    </tr>
    <tr>
      <th>M</th>
      <td>1571</td>
    </tr>
    <tr>
      <th>M2</th>
      <td>1148</td>
    </tr>
    <tr>
      <th>M3</th>
      <td>184</td>
    </tr>
    <tr>
      <th>M4</th>
      <td>465</td>
    </tr>
    <tr>
      <th>M5</th>
      <td>1216</td>
    </tr>
    <tr>
      <th>M8</th>
      <td>107</td>
    </tr>
    <tr>
      <th>New 4 Series</th>
      <td>537</td>
    </tr>
    <tr>
      <th>The new M3</th>
      <td>72</td>
    </tr>
    <tr>
      <th>The new M4</th>
      <td>58</td>
    </tr>
    <tr>
      <th>X1</th>
      <td>6889</td>
    </tr>
    <tr>
      <th>X2</th>
      <td>1742</td>
    </tr>
    <tr>
      <th>X3</th>
      <td>12761</td>
    </tr>
    <tr>
      <th>X3 M</th>
      <td>506</td>
    </tr>
    <tr>
      <th>X4</th>
      <td>10961</td>
    </tr>
    <tr>
      <th>X4 M</th>
      <td>494</td>
    </tr>
    <tr>
      <th>X5</th>
      <td>13727</td>
    </tr>
    <tr>
      <th>X5 M</th>
      <td>174</td>
    </tr>
    <tr>
      <th>X6</th>
      <td>9907</td>
    </tr>
    <tr>
      <th>X6 M</th>
      <td>292</td>
    </tr>
    <tr>
      <th>X7</th>
      <td>5006</td>
    </tr>
    <tr>
      <th>Z4</th>
      <td>1211</td>
    </tr>
    <tr>
      <th>i3</th>
      <td>1710</td>
    </tr>
    <tr>
      <th>i8</th>
      <td>434</td>
    </tr>
  </tbody>
</table>
</div>




```python
all_BMW_model_quentity = all_BMW_model_quentity.reset_index()
```


```python
# 내림차순 순위 정렬
BMWSalesRanking = all_BMW_model_quentity.sort_values('quentity', ascending=False)
BMWSalesRanking
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
      <th>car_name</th>
      <th>quentity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5</th>
      <td>5 Series</td>
      <td>101035</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3 Series</td>
      <td>42392</td>
    </tr>
    <tr>
      <th>0</th>
      <td>1 Series</td>
      <td>18338</td>
    </tr>
    <tr>
      <th>25</th>
      <td>X5</td>
      <td>13727</td>
    </tr>
    <tr>
      <th>21</th>
      <td>X3</td>
      <td>12761</td>
    </tr>
    <tr>
      <th>7</th>
      <td>7 Series</td>
      <td>11762</td>
    </tr>
    <tr>
      <th>23</th>
      <td>X4</td>
      <td>10961</td>
    </tr>
    <tr>
      <th>6</th>
      <td>6 Series</td>
      <td>10243</td>
    </tr>
    <tr>
      <th>27</th>
      <td>X6</td>
      <td>9907</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4 Series</td>
      <td>8030</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2 Series Active Tourer</td>
      <td>7002</td>
    </tr>
    <tr>
      <th>19</th>
      <td>X1</td>
      <td>6889</td>
    </tr>
    <tr>
      <th>29</th>
      <td>X7</td>
      <td>5006</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Gran Turismo</td>
      <td>2934</td>
    </tr>
    <tr>
      <th>20</th>
      <td>X2</td>
      <td>1742</td>
    </tr>
    <tr>
      <th>31</th>
      <td>i3</td>
      <td>1710</td>
    </tr>
    <tr>
      <th>10</th>
      <td>M</td>
      <td>1571</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2 Series</td>
      <td>1294</td>
    </tr>
    <tr>
      <th>14</th>
      <td>M5</td>
      <td>1216</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Z4</td>
      <td>1211</td>
    </tr>
    <tr>
      <th>11</th>
      <td>M2</td>
      <td>1148</td>
    </tr>
    <tr>
      <th>8</th>
      <td>8 Series</td>
      <td>633</td>
    </tr>
    <tr>
      <th>16</th>
      <td>New 4 Series</td>
      <td>537</td>
    </tr>
    <tr>
      <th>22</th>
      <td>X3 M</td>
      <td>506</td>
    </tr>
    <tr>
      <th>24</th>
      <td>X4 M</td>
      <td>494</td>
    </tr>
    <tr>
      <th>13</th>
      <td>M4</td>
      <td>465</td>
    </tr>
    <tr>
      <th>32</th>
      <td>i8</td>
      <td>434</td>
    </tr>
    <tr>
      <th>28</th>
      <td>X6 M</td>
      <td>292</td>
    </tr>
    <tr>
      <th>12</th>
      <td>M3</td>
      <td>184</td>
    </tr>
    <tr>
      <th>26</th>
      <td>X5 M</td>
      <td>174</td>
    </tr>
    <tr>
      <th>15</th>
      <td>M8</td>
      <td>107</td>
    </tr>
    <tr>
      <th>17</th>
      <td>The new M3</td>
      <td>72</td>
    </tr>
    <tr>
      <th>18</th>
      <td>The new M4</td>
      <td>58</td>
    </tr>
  </tbody>
</table>
</div>




```python
BMWSalesRanking.columns = ['BMW모델','판매량']#컬럼 변경
```


```python
# 점유율칼럼추가
BMWSalesRanking['점유율'] = BMWSalesRanking['판매량']/274835*100
BMWSalesRanking
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
      <th>BMW모델</th>
      <th>판매량</th>
      <th>점유율</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5</th>
      <td>5 Series</td>
      <td>101035</td>
      <td>36.762057</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3 Series</td>
      <td>42392</td>
      <td>15.424527</td>
    </tr>
    <tr>
      <th>0</th>
      <td>1 Series</td>
      <td>18338</td>
      <td>6.672367</td>
    </tr>
    <tr>
      <th>25</th>
      <td>X5</td>
      <td>13727</td>
      <td>4.994633</td>
    </tr>
    <tr>
      <th>21</th>
      <td>X3</td>
      <td>12761</td>
      <td>4.643150</td>
    </tr>
    <tr>
      <th>7</th>
      <td>7 Series</td>
      <td>11762</td>
      <td>4.279659</td>
    </tr>
    <tr>
      <th>23</th>
      <td>X4</td>
      <td>10961</td>
      <td>3.988211</td>
    </tr>
    <tr>
      <th>6</th>
      <td>6 Series</td>
      <td>10243</td>
      <td>3.726963</td>
    </tr>
    <tr>
      <th>27</th>
      <td>X6</td>
      <td>9907</td>
      <td>3.604708</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4 Series</td>
      <td>8030</td>
      <td>2.921753</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2 Series Active Tourer</td>
      <td>7002</td>
      <td>2.547710</td>
    </tr>
    <tr>
      <th>19</th>
      <td>X1</td>
      <td>6889</td>
      <td>2.506595</td>
    </tr>
    <tr>
      <th>29</th>
      <td>X7</td>
      <td>5006</td>
      <td>1.821457</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Gran Turismo</td>
      <td>2934</td>
      <td>1.067550</td>
    </tr>
    <tr>
      <th>20</th>
      <td>X2</td>
      <td>1742</td>
      <td>0.633835</td>
    </tr>
    <tr>
      <th>31</th>
      <td>i3</td>
      <td>1710</td>
      <td>0.622191</td>
    </tr>
    <tr>
      <th>10</th>
      <td>M</td>
      <td>1571</td>
      <td>0.571616</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2 Series</td>
      <td>1294</td>
      <td>0.470828</td>
    </tr>
    <tr>
      <th>14</th>
      <td>M5</td>
      <td>1216</td>
      <td>0.442447</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Z4</td>
      <td>1211</td>
      <td>0.440628</td>
    </tr>
    <tr>
      <th>11</th>
      <td>M2</td>
      <td>1148</td>
      <td>0.417705</td>
    </tr>
    <tr>
      <th>8</th>
      <td>8 Series</td>
      <td>633</td>
      <td>0.230320</td>
    </tr>
    <tr>
      <th>16</th>
      <td>New 4 Series</td>
      <td>537</td>
      <td>0.195390</td>
    </tr>
    <tr>
      <th>22</th>
      <td>X3 M</td>
      <td>506</td>
      <td>0.184110</td>
    </tr>
    <tr>
      <th>24</th>
      <td>X4 M</td>
      <td>494</td>
      <td>0.179744</td>
    </tr>
    <tr>
      <th>13</th>
      <td>M4</td>
      <td>465</td>
      <td>0.169192</td>
    </tr>
    <tr>
      <th>32</th>
      <td>i8</td>
      <td>434</td>
      <td>0.157913</td>
    </tr>
    <tr>
      <th>28</th>
      <td>X6 M</td>
      <td>292</td>
      <td>0.106246</td>
    </tr>
    <tr>
      <th>12</th>
      <td>M3</td>
      <td>184</td>
      <td>0.066949</td>
    </tr>
    <tr>
      <th>26</th>
      <td>X5 M</td>
      <td>174</td>
      <td>0.063311</td>
    </tr>
    <tr>
      <th>15</th>
      <td>M8</td>
      <td>107</td>
      <td>0.038932</td>
    </tr>
    <tr>
      <th>17</th>
      <td>The new M3</td>
      <td>72</td>
      <td>0.026198</td>
    </tr>
    <tr>
      <th>18</th>
      <td>The new M4</td>
      <td>58</td>
      <td>0.021104</td>
    </tr>
  </tbody>
</table>
</div>




```python
# SUV 차량 점유율
BMWSuv = BMWSalesRanking.loc[[25,21,23,27,19,29,20,31,22,24,28,26], :]
BMWSuv
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
      <th>BMW모델</th>
      <th>판매량</th>
      <th>점유율</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>25</th>
      <td>X5</td>
      <td>13727</td>
      <td>4.994633</td>
    </tr>
    <tr>
      <th>21</th>
      <td>X3</td>
      <td>12761</td>
      <td>4.643150</td>
    </tr>
    <tr>
      <th>23</th>
      <td>X4</td>
      <td>10961</td>
      <td>3.988211</td>
    </tr>
    <tr>
      <th>27</th>
      <td>X6</td>
      <td>9907</td>
      <td>3.604708</td>
    </tr>
    <tr>
      <th>19</th>
      <td>X1</td>
      <td>6889</td>
      <td>2.506595</td>
    </tr>
    <tr>
      <th>29</th>
      <td>X7</td>
      <td>5006</td>
      <td>1.821457</td>
    </tr>
    <tr>
      <th>20</th>
      <td>X2</td>
      <td>1742</td>
      <td>0.633835</td>
    </tr>
    <tr>
      <th>31</th>
      <td>i3</td>
      <td>1710</td>
      <td>0.622191</td>
    </tr>
    <tr>
      <th>22</th>
      <td>X3 M</td>
      <td>506</td>
      <td>0.184110</td>
    </tr>
    <tr>
      <th>24</th>
      <td>X4 M</td>
      <td>494</td>
      <td>0.179744</td>
    </tr>
    <tr>
      <th>28</th>
      <td>X6 M</td>
      <td>292</td>
      <td>0.106246</td>
    </tr>
    <tr>
      <th>26</th>
      <td>X5 M</td>
      <td>174</td>
      <td>0.063311</td>
    </tr>
  </tbody>
</table>
</div>




```python
# SUV 판매량 합계
BMWSuvSum = BMWSuv['판매량'].sum()
BMWSuvSum
```




    64169




```python
# SUV 판매량 합계 dataframe
BMWSuvSum = pd.DataFrame({'BMW':['SUV'], '판매량':[64169]})
BMWSuvSum
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
      <th>BMW</th>
      <th>판매량</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>SUV</td>
      <td>64169</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Sedan 칼럼 선택
BMWSedan = BMWSalesRanking.loc[[5,3,0,7,6,4,2,9,10,1,14,30,11,8,16,13,32,12,15,17,18], :]
BMWSedan
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-1-a2e4fec173da> in <module>
          1 # Sedan 칼럼 선택
    ----> 2 BMWSedan = BMWSalesRanking.loc[[5,3,0,7,6,4,2,9,10,1,14,30,11,8,16,13,32,12,15,17,18], :]
          3 BMWSedan
    

    NameError: name 'BMWSalesRanking' is not defined



```python
# Sedan 판매량 합계
BMWSedanSum = BMWSedan['판매량'].sum()
BMWSedanSum
```




    210666




```python
# Sedan 판매량 dataframe
BMWSedanSum = pd.DataFrame({'BMW':['Sedan'], '판매량':[210666]})
BMWSedanSum
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
      <th>BMW</th>
      <th>판매량</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Sedan</td>
      <td>210666</td>
    </tr>
  </tbody>
</table>
</div>




```python
# SUV, Sedan 합치기
BMWModel = pd.concat([BMWSuvSum, BMWSedanSum])
BMWModel                     
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
      <th>BMW</th>
      <th>판매량</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>SUV</td>
      <td>64169</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Sedan</td>
      <td>210666</td>
    </tr>
  </tbody>
</table>
</div>



### 연간 차량 총 등록대수 (승용, 승합)


```python
car_registration = pd.read_excel("D:\\data\\21.07_BMW미팅\\df_data/2021승용_승합_등록현황3.xlsx")
car_registration
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-2-206a83b6a539> in <module>
    ----> 1 car_registration = pd.read_excel("D:\\data\\21.07_BMW미팅\\df_data/2021승용_승합_등록현황3.xlsx")
          2 car_registration
    

    NameError: name 'pd' is not defined


## EDA 

### BMW 모델 별 누적판매량 순위

- 누적 판매량 순위 : 5Series >3Series >1Series >X5 >X3


```python
# seaborn 의 set 기능을 통해 폰트 설정
# "Malgun Gothic"

plt.figure(figsize=(15,10)) # size
plt.title('BMW 모델 별 누적판매량 순위', size= 20)
plt.grid(True) #눈금
plt.rotation=45
sns.barplot(x="판매량", y="BMW모델", data=BMWSalesRanking)
```




    <AxesSubplot:title={'center':'BMW 모델 별 누적판매량 순위'}, xlabel='판매량', ylabel='BMW모델'>




    
![png](../../Users/웍스컴바인/Downloads/2021-07-25-BMW_SUV_Sedan_sales_analysis/output_44_1.png)
    



```python
# color palette 설정
custom_palette = sns.color_palette("Paired", 33)
sns.palplot(custom_palette)
```


    
![png](../../Users/웍스컴바인/Downloads/2021-07-25-BMW_SUV_Sedan_sales_analysis/output_45_0.png)
    



```python
# color palette 설정
custom_palette2 = sns.color_palette("Paired", 12)
sns.palplot(custom_palette2)
```


    
![png](../../Users/웍스컴바인/Downloads/2021-07-25-BMW_SUV_Sedan_sales_analysis/output_46_0.png)
    


### BMW 모델 별 판매 점유율 시각화 

- 5Series 판매 점유율은 전체 모델 중 41%로 과반수 차지, 3Series, 1Series 17.2% 7.4% 차지
- Sedan, SUV 모델 각 76.7%, 23.3% 판매 점유율 차지 
- 전체 SUV 모델 중 X5, X3, X4, 3개 모델 판매 점유율 총 58.2%로 과반수 차지
- Sedan 17-19년 판매량 급감한 반면 SUV모델 18-19년 소량 증가, 20년 큰폭으로 증가


```python

#도표1
labels = ['5 Series','3 Series','1 Series','X5','X3','7 Series','X4','6 Series','X6','4 Series',
        '2 Series Active Tourer']

colors = custom_palette

plt.figure(figsize = (21,21))
plt.subplot(221)#프레임생성

wedgeprops={'width': 0.7, 'edgecolor': 'w', 'linewidth': 5}
plt.pie(BMWSalesRanking['판매량'][:11],#1%미만 절삭, 14개 모델
        labels=labels, autopct='%.1f%%', startangle=180,#시작점 
        counterclock=False, colors=colors, wedgeprops=wedgeprops,#시계방향, 색, wedgeprops
        textprops = {'fontsize':18})
# plt.legend(labels=labels, loc='upper right')#범례
plt.title("도표1 - BMW 판매순위 별 Top10 모델 점유율",size=25)

# 자간조정 필요


#도표2
labels2 = ['SUV','Sedan']
colors2 = ['#20639B','#F6D55C']

plt.subplot(222)#프레임생성

wedgeprops={'width': 0.7, 'edgecolor': 'w', 'linewidth': 5}
plt.pie(BMWModel['판매량'],
        labels=labels2, autopct='%.1f%%', startangle=180,#시작점 
        counterclock=False, colors=colors2, wedgeprops=wedgeprops,#시계방향, 색, wedgeprops
        textprops = {'fontsize':18})
# plt.legend(labels=labels, loc='upper right')#범례
plt.title("도표2 - BMW SUV vs Sedan모델 판매 점유율",size=25)


#도표3
labels3 = ['X5', 'X3', 'X4', 'X6', 'X1', 'X7', 'X2', 'i3', 'X3 M', 'X4 M']
colors2 = custom_palette2

plt.subplot(223)#프레임생성

wedgeprops={'width': 0.7, 'edgecolor': 'w', 'linewidth': 5}
plt.pie(BMWSuv['판매량'][:10],
        labels=labels3, autopct='%.1f%%', startangle=180,#시작점 
        counterclock=False, colors=colors2, wedgeprops=wedgeprops,#시계방향, 색, wedgeprops
        textprops = {'fontsize':18})
# plt.legend(labels=labels, loc='upper right')#범례
plt.title("도표3 - BMW SUV 모델 별 Top10 판매 점유율",size=25)


#도표4
ax = plt.subplot(224)#프레임생성

#꺾은선 그래프 추가

year = [2017, 2018, 2019, 2020, 2021]
index = np.arange(len(year))

ax.plot(index, BMWSuvSedanYear['BMW_Sedan'], label='Sedan', marker='o', color='#236E96') 
ax.plot(index, BMWSuvSedanYear['BMW_SUV'], label='SUV', marker='o', color='#15B2D3') 

ax.set_title('도표4 - BMW SUV VS Sedan 모델 연간 판매량 추이', fontsize=25) # 타이틀 설정
ax.set_ylabel('판매량', fontsize=12) # x축 설정
ax.set_xlabel('year', fontsize=12) # y축 설정

ax.legend(fontsize=10, loc='best') 

plt.xticks(index, year)
plt.show()
```


    
![png](../../Users/웍스컴바인/Downloads/2021-07-25-BMW_SUV_Sedan_sales_analysis/output_49_0.png)
    


### 연간 차량 등록 수 및 BMW SUV 판매추이 비교

- 자동차 등록 수와 SUV 판매량 증감 추이로 보아 관련성 있음


```python
# 기본 스타일 설정
plt.style.use('default')
plt.rcParams['figure.figsize'] = (8, 5)
plt.rcParams['font.size'] = 12
plt.rcParams['font.family'] = 'Malgun Gothic'

# labels = [2017,2018,2019,2020,'2021(1~5월)']
# colors = custom_palette

x = np.arange(2017, 2022)
fig, ax1 = plt.subplots()

# 그래프1 - SUV 모델 연간 총 판매량 추이

ax1.bar(x,BMWSuvYearQuentity2['quentity'], color='#FFA8A5', label='Demand', alpha=0.7, width=0.7)
ax1.set_ylim(0, 22000)
ax1.set_ylabel('SUV판매량(bar)')
ax1.set_xlabel('year')
ax1.tick_params(axis='y', direction='in')
plt.title('연간 차량 등록 수 및 BMW SUV 판매추이 비교', size = 20)

# 그래프2 -연도별 자동차 등록현황 자가용 총대수(

ax2 = ax1.twinx()

ax2.plot(x, car_registration['자가용총대수'], marker = 'o', color='green', markersize=7, linewidth=5, alpha=0.7, label='Price')
ax2.set_ylim(0, 5)
ax2.set_xlabel('year')
ax2.set_ylabel('전국 자가용등록대수 전년대비 증감비율(%)')
ax2.tick_params(axis='both', direction='in')

plt.show()
```


    
![png](../../Users/웍스컴바인/Downloads/2021-07-25-BMW_SUV_Sedan_sales_analysis/output_51_0.png)
    

