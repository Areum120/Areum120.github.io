---
title: "BMW Project: BMW 신규 거점 선정을 위한 서울시 교통사고량, BMW 서비스센터 분석"
date: 2021-08-10
read_time: false
comments: false
share: false
related: false

categories:
   - BMW Project
tags:
   - BMW
   - data analysis
   - folium
   - googlemaps
   - pandas 
toc: true
---
BMW 신규 전시장과 서비스센터 거점 선정을 위해<br>
서울지역 별 교통사고량 대비 BMW, Benz의 서비스센터 수를 지도 시각화로 비교 분석하기로 했다.
```python
# 데이터 분석
import pandas as pd
import openpyxl
import json

# 지도시각화
import folium
from folium import plugins
from folium.features import CustomIcon
import googlemaps

#경고창 무시
import warnings
warnings.filterwarnings(action='ignore')

```

## BMW 신규 거점 선정을 위한 분석 Report 

### 전시장 & 서비스센터 신규 거점 분석
  - 서울시 차대차 교통사고 수 분석
  - 서울시 BMW, Benz 서비스 센터 지도 시각화 
  - 서울시 자동차부품점 지도 시각화         

### 서울시 차대차 교통사고 데이터 불러오기


```python
seoul_acci = pd.read_table('./17_20_seoul_car_accident.txt',sep='\t')#탭구분
seoul_acci
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
      <th>기간</th>
      <th>자치구</th>
      <th>구분</th>
      <th>합계</th>
      <th>사고유형별</th>
      <th>사고유형별.1</th>
      <th>사고유형별.2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>기간</td>
      <td>자치구</td>
      <td>구분</td>
      <td>합계</td>
      <td>차대사람</td>
      <td>차대차</td>
      <td>차량단독</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2017</td>
      <td>합계</td>
      <td>발생건수</td>
      <td>38,625</td>
      <td>10,249</td>
      <td>27,103</td>
      <td>1,273</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2017</td>
      <td>합계</td>
      <td>사망자수</td>
      <td>343</td>
      <td>189</td>
      <td>117</td>
      <td>37</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2017</td>
      <td>합계</td>
      <td>부상자수</td>
      <td>53,810</td>
      <td>10,706</td>
      <td>41,642</td>
      <td>1,462</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017</td>
      <td>종로구</td>
      <td>발생건수</td>
      <td>1,190</td>
      <td>333</td>
      <td>810</td>
      <td>47</td>
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
    </tr>
    <tr>
      <th>308</th>
      <td>2020</td>
      <td>송파구</td>
      <td>사망자수</td>
      <td>11</td>
      <td>4</td>
      <td>5</td>
      <td>2</td>
    </tr>
    <tr>
      <th>309</th>
      <td>2020</td>
      <td>송파구</td>
      <td>부상자수</td>
      <td>3,509</td>
      <td>502</td>
      <td>2,954</td>
      <td>53</td>
    </tr>
    <tr>
      <th>310</th>
      <td>2020</td>
      <td>강동구</td>
      <td>발생건수</td>
      <td>1,268</td>
      <td>342</td>
      <td>890</td>
      <td>36</td>
    </tr>
    <tr>
      <th>311</th>
      <td>2020</td>
      <td>강동구</td>
      <td>사망자수</td>
      <td>7</td>
      <td>3</td>
      <td>4</td>
      <td>-</td>
    </tr>
    <tr>
      <th>312</th>
      <td>2020</td>
      <td>강동구</td>
      <td>부상자수</td>
      <td>1,600</td>
      <td>352</td>
      <td>1,207</td>
      <td>41</td>
    </tr>
  </tbody>
</table>
<p>313 rows × 7 columns</p>
</div>




```python
seoul_acci = seoul_acci.drop(['합계','사고유형별','사고유형별.2'], axis=1)
```


```python
seoul_acci.columns = ['기간','자치구','구분','차대차사고']
```


```python
seoul_acci
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
      <th>기간</th>
      <th>자치구</th>
      <th>구분</th>
      <th>차대차사고</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>기간</td>
      <td>자치구</td>
      <td>구분</td>
      <td>차대차</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2017</td>
      <td>합계</td>
      <td>발생건수</td>
      <td>27,103</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2017</td>
      <td>합계</td>
      <td>사망자수</td>
      <td>117</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2017</td>
      <td>합계</td>
      <td>부상자수</td>
      <td>41,642</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017</td>
      <td>종로구</td>
      <td>발생건수</td>
      <td>810</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>308</th>
      <td>2020</td>
      <td>송파구</td>
      <td>사망자수</td>
      <td>5</td>
    </tr>
    <tr>
      <th>309</th>
      <td>2020</td>
      <td>송파구</td>
      <td>부상자수</td>
      <td>2,954</td>
    </tr>
    <tr>
      <th>310</th>
      <td>2020</td>
      <td>강동구</td>
      <td>발생건수</td>
      <td>890</td>
    </tr>
    <tr>
      <th>311</th>
      <td>2020</td>
      <td>강동구</td>
      <td>사망자수</td>
      <td>4</td>
    </tr>
    <tr>
      <th>312</th>
      <td>2020</td>
      <td>강동구</td>
      <td>부상자수</td>
      <td>1,207</td>
    </tr>
  </tbody>
</table>
<p>313 rows × 4 columns</p>
</div>




```python
seoul_acci2 = seoul_acci[seoul_acci['차대차사고'].isin(['-'])]
seoul_acci2
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
      <th>기간</th>
      <th>자치구</th>
      <th>구분</th>
      <th>차대차사고</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>134</th>
      <td>2018</td>
      <td>금천구</td>
      <td>사망자수</td>
      <td>-</td>
    </tr>
    <tr>
      <th>140</th>
      <td>2018</td>
      <td>동작구</td>
      <td>사망자수</td>
      <td>-</td>
    </tr>
    <tr>
      <th>143</th>
      <td>2018</td>
      <td>관악구</td>
      <td>사망자수</td>
      <td>-</td>
    </tr>
    <tr>
      <th>173</th>
      <td>2019</td>
      <td>광진구</td>
      <td>사망자수</td>
      <td>-</td>
    </tr>
    <tr>
      <th>188</th>
      <td>2019</td>
      <td>도봉구</td>
      <td>사망자수</td>
      <td>-</td>
    </tr>
    <tr>
      <th>257</th>
      <td>2020</td>
      <td>중랑구</td>
      <td>사망자수</td>
      <td>-</td>
    </tr>
    <tr>
      <th>275</th>
      <td>2020</td>
      <td>서대문구</td>
      <td>사망자수</td>
      <td>-</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 불필요한 행 삭제
seoul_acci = seoul_acci.drop(index = [134, 140, 143, 173, 188, 257, 275])
seoul_acci
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
      <th>기간</th>
      <th>자치구</th>
      <th>구분</th>
      <th>차대차사고</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>기간</td>
      <td>자치구</td>
      <td>구분</td>
      <td>차대차</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2017</td>
      <td>합계</td>
      <td>발생건수</td>
      <td>27,103</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2017</td>
      <td>합계</td>
      <td>사망자수</td>
      <td>117</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2017</td>
      <td>합계</td>
      <td>부상자수</td>
      <td>41,642</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017</td>
      <td>종로구</td>
      <td>발생건수</td>
      <td>810</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>308</th>
      <td>2020</td>
      <td>송파구</td>
      <td>사망자수</td>
      <td>5</td>
    </tr>
    <tr>
      <th>309</th>
      <td>2020</td>
      <td>송파구</td>
      <td>부상자수</td>
      <td>2,954</td>
    </tr>
    <tr>
      <th>310</th>
      <td>2020</td>
      <td>강동구</td>
      <td>발생건수</td>
      <td>890</td>
    </tr>
    <tr>
      <th>311</th>
      <td>2020</td>
      <td>강동구</td>
      <td>사망자수</td>
      <td>4</td>
    </tr>
    <tr>
      <th>312</th>
      <td>2020</td>
      <td>강동구</td>
      <td>부상자수</td>
      <td>1,207</td>
    </tr>
  </tbody>
</table>
<p>306 rows × 4 columns</p>
</div>




```python
seoul_acci = seoul_acci.drop(0, axis=0)#0번째 row 삭제
```


```python
# 콤마제거, int변환
seoul_acci['차대차사고'] = seoul_acci.차대차사고.str.replace(',', '').astype('int64')
```


```python
# 특정 행값만 가져오기
seoul_acci= seoul_acci[seoul_acci['구분'].isin(['발생건수'])]
seoul_acci
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
      <th>기간</th>
      <th>자치구</th>
      <th>구분</th>
      <th>차대차사고</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>2017</td>
      <td>합계</td>
      <td>발생건수</td>
      <td>27103</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017</td>
      <td>종로구</td>
      <td>발생건수</td>
      <td>810</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2017</td>
      <td>중구</td>
      <td>발생건수</td>
      <td>896</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2017</td>
      <td>용산구</td>
      <td>발생건수</td>
      <td>994</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2017</td>
      <td>성동구</td>
      <td>발생건수</td>
      <td>788</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>298</th>
      <td>2020</td>
      <td>관악구</td>
      <td>발생건수</td>
      <td>1087</td>
    </tr>
    <tr>
      <th>301</th>
      <td>2020</td>
      <td>서초구</td>
      <td>발생건수</td>
      <td>1813</td>
    </tr>
    <tr>
      <th>304</th>
      <td>2020</td>
      <td>강남구</td>
      <td>발생건수</td>
      <td>2964</td>
    </tr>
    <tr>
      <th>307</th>
      <td>2020</td>
      <td>송파구</td>
      <td>발생건수</td>
      <td>2064</td>
    </tr>
    <tr>
      <th>310</th>
      <td>2020</td>
      <td>강동구</td>
      <td>발생건수</td>
      <td>890</td>
    </tr>
  </tbody>
</table>
<p>104 rows × 4 columns</p>
</div>




```python
seoul_acci = seoul_acci.drop(['구분'], axis=1)
seoul_acci = seoul_acci.reset_index()
```


```python
seoul_acci
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
      <th>index</th>
      <th>기간</th>
      <th>자치구</th>
      <th>차대차사고</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>2017</td>
      <td>합계</td>
      <td>27103</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4</td>
      <td>2017</td>
      <td>종로구</td>
      <td>810</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7</td>
      <td>2017</td>
      <td>중구</td>
      <td>896</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10</td>
      <td>2017</td>
      <td>용산구</td>
      <td>994</td>
    </tr>
    <tr>
      <th>4</th>
      <td>13</td>
      <td>2017</td>
      <td>성동구</td>
      <td>788</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>99</th>
      <td>298</td>
      <td>2020</td>
      <td>관악구</td>
      <td>1087</td>
    </tr>
    <tr>
      <th>100</th>
      <td>301</td>
      <td>2020</td>
      <td>서초구</td>
      <td>1813</td>
    </tr>
    <tr>
      <th>101</th>
      <td>304</td>
      <td>2020</td>
      <td>강남구</td>
      <td>2964</td>
    </tr>
    <tr>
      <th>102</th>
      <td>307</td>
      <td>2020</td>
      <td>송파구</td>
      <td>2064</td>
    </tr>
    <tr>
      <th>103</th>
      <td>310</td>
      <td>2020</td>
      <td>강동구</td>
      <td>890</td>
    </tr>
  </tbody>
</table>
<p>104 rows × 4 columns</p>
</div>




```python
# index 열 삭제
seoul_acci = seoul_acci.drop(['index'], axis=1)
```


```python
seoul_acci_gu = seoul_acci.groupby(['자치구'])[['차대차사고']].sum()
seoul_acci_gu = seoul_acci_gu.reset_index()
```


```python
seoul_acci_gu
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
      <th>자치구</th>
      <th>차대차사고</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>강남구</td>
      <td>11025</td>
    </tr>
    <tr>
      <th>1</th>
      <td>강동구</td>
      <td>3718</td>
    </tr>
    <tr>
      <th>2</th>
      <td>강북구</td>
      <td>3113</td>
    </tr>
    <tr>
      <th>3</th>
      <td>강서구</td>
      <td>4654</td>
    </tr>
    <tr>
      <th>4</th>
      <td>관악구</td>
      <td>3806</td>
    </tr>
    <tr>
      <th>5</th>
      <td>광진구</td>
      <td>2885</td>
    </tr>
    <tr>
      <th>6</th>
      <td>구로구</td>
      <td>4151</td>
    </tr>
    <tr>
      <th>7</th>
      <td>금천구</td>
      <td>2368</td>
    </tr>
    <tr>
      <th>8</th>
      <td>노원구</td>
      <td>4609</td>
    </tr>
    <tr>
      <th>9</th>
      <td>도봉구</td>
      <td>2141</td>
    </tr>
    <tr>
      <th>10</th>
      <td>동대문구</td>
      <td>4439</td>
    </tr>
    <tr>
      <th>11</th>
      <td>동작구</td>
      <td>3802</td>
    </tr>
    <tr>
      <th>12</th>
      <td>마포구</td>
      <td>4628</td>
    </tr>
    <tr>
      <th>13</th>
      <td>서대문구</td>
      <td>2755</td>
    </tr>
    <tr>
      <th>14</th>
      <td>서초구</td>
      <td>7361</td>
    </tr>
    <tr>
      <th>15</th>
      <td>성동구</td>
      <td>3283</td>
    </tr>
    <tr>
      <th>16</th>
      <td>성북구</td>
      <td>3985</td>
    </tr>
    <tr>
      <th>17</th>
      <td>송파구</td>
      <td>8441</td>
    </tr>
    <tr>
      <th>18</th>
      <td>양천구</td>
      <td>4075</td>
    </tr>
    <tr>
      <th>19</th>
      <td>영등포구</td>
      <td>7208</td>
    </tr>
    <tr>
      <th>20</th>
      <td>용산구</td>
      <td>3577</td>
    </tr>
    <tr>
      <th>21</th>
      <td>은평구</td>
      <td>2679</td>
    </tr>
    <tr>
      <th>22</th>
      <td>종로구</td>
      <td>2996</td>
    </tr>
    <tr>
      <th>23</th>
      <td>중구</td>
      <td>3232</td>
    </tr>
    <tr>
      <th>24</th>
      <td>중랑구</td>
      <td>4352</td>
    </tr>
    <tr>
      <th>25</th>
      <td>합계</td>
      <td>109283</td>
    </tr>
  </tbody>
</table>
</div>




```python
seoul_acci_gu = seoul_acci_gu.drop(index = [25])#합계 삭제
```

### BMW, Benz 서비스센터 데이터 불러오기


```python
BMW_sc = pd.read_csv('./BMW_all_sc.csv')
BMW_sc
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
      <th>ShopName</th>
      <th>address</th>
      <th>DealerName</th>
      <th>Tel</th>
      <th>lat</th>
      <th>lng</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>강남역 서비스 센터</td>
      <td>서울특별시 서초구 서초대로71길 10</td>
      <td>코오롱모터스</td>
      <td>02-586-3331</td>
      <td>37.498113</td>
      <td>127.024903</td>
    </tr>
    <tr>
      <th>1</th>
      <td>강북 서비스 센터</td>
      <td>서울특별시 강북구 도봉로 101</td>
      <td>한독모터스</td>
      <td>02-3444-7301</td>
      <td>37.618057</td>
      <td>127.029050</td>
    </tr>
    <tr>
      <th>2</th>
      <td>강서 서비스 센터</td>
      <td>서울특별시 강서구 공항대로 59</td>
      <td>바바리안모터스</td>
      <td>02-2661-7401</td>
      <td>37.561364</td>
      <td>126.813979</td>
    </tr>
    <tr>
      <th>3</th>
      <td>교대 서비스 센터</td>
      <td>서울특별시 서초구 효령로53길 62</td>
      <td>코오롱모터스</td>
      <td>02-3472-7301</td>
      <td>37.487519</td>
      <td>127.013255</td>
    </tr>
    <tr>
      <th>4</th>
      <td>대치 서비스 센터</td>
      <td>서울특별시 강남구 영동대로 340</td>
      <td>코오롱모터스</td>
      <td>02-569-7401</td>
      <td>37.505064</td>
      <td>127.065829</td>
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
      <th>60</th>
      <td>창원 서비스 센터</td>
      <td>경상남도 창원시 마산회원구 봉양로 163</td>
      <td>동성모터스</td>
      <td>055-256-7301</td>
      <td>35.218263</td>
      <td>128.604576</td>
    </tr>
    <tr>
      <th>61</th>
      <td>포항 서비스 센터</td>
      <td>경상북도 포항시 북구 새천년대로 569</td>
      <td>동성모터스</td>
      <td>054-272-7306</td>
      <td>36.025119</td>
      <td>129.352454</td>
    </tr>
    <tr>
      <th>62</th>
      <td>부산 롯데 패스트레인 서비스 센터</td>
      <td>부산광역시 부산진구 신천대로 241</td>
      <td>동성모터스</td>
      <td>051-792-1810</td>
      <td>35.163307</td>
      <td>129.049316</td>
    </tr>
    <tr>
      <th>63</th>
      <td>울산 진장 롯데 패스트레인</td>
      <td>울산 북구 진장유통로 64</td>
      <td>동성모터스</td>
      <td>052-702-8361</td>
      <td>35.576004</td>
      <td>129.359025</td>
    </tr>
    <tr>
      <th>64</th>
      <td>제주 서비스 센터</td>
      <td>제주특별자치도 제주시 동화로 66-2</td>
      <td>도이치모터스</td>
      <td>064-757-7601</td>
      <td>33.514418</td>
      <td>126.572234</td>
    </tr>
  </tbody>
</table>
<p>65 rows × 6 columns</p>
</div>




```python
# brand 칼럼 추가
BMW_sc["brand"] = "BMW"
BMW_sc
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
      <th>ShopName</th>
      <th>address</th>
      <th>DealerName</th>
      <th>Tel</th>
      <th>lat</th>
      <th>lng</th>
      <th>brand</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>강남역 서비스 센터</td>
      <td>서울특별시 서초구 서초대로71길 10</td>
      <td>코오롱모터스</td>
      <td>02-586-3331</td>
      <td>37.498113</td>
      <td>127.024903</td>
      <td>BMW</td>
    </tr>
    <tr>
      <th>1</th>
      <td>강북 서비스 센터</td>
      <td>서울특별시 강북구 도봉로 101</td>
      <td>한독모터스</td>
      <td>02-3444-7301</td>
      <td>37.618057</td>
      <td>127.029050</td>
      <td>BMW</td>
    </tr>
    <tr>
      <th>2</th>
      <td>강서 서비스 센터</td>
      <td>서울특별시 강서구 공항대로 59</td>
      <td>바바리안모터스</td>
      <td>02-2661-7401</td>
      <td>37.561364</td>
      <td>126.813979</td>
      <td>BMW</td>
    </tr>
    <tr>
      <th>3</th>
      <td>교대 서비스 센터</td>
      <td>서울특별시 서초구 효령로53길 62</td>
      <td>코오롱모터스</td>
      <td>02-3472-7301</td>
      <td>37.487519</td>
      <td>127.013255</td>
      <td>BMW</td>
    </tr>
    <tr>
      <th>4</th>
      <td>대치 서비스 센터</td>
      <td>서울특별시 강남구 영동대로 340</td>
      <td>코오롱모터스</td>
      <td>02-569-7401</td>
      <td>37.505064</td>
      <td>127.065829</td>
      <td>BMW</td>
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
    </tr>
    <tr>
      <th>60</th>
      <td>창원 서비스 센터</td>
      <td>경상남도 창원시 마산회원구 봉양로 163</td>
      <td>동성모터스</td>
      <td>055-256-7301</td>
      <td>35.218263</td>
      <td>128.604576</td>
      <td>BMW</td>
    </tr>
    <tr>
      <th>61</th>
      <td>포항 서비스 센터</td>
      <td>경상북도 포항시 북구 새천년대로 569</td>
      <td>동성모터스</td>
      <td>054-272-7306</td>
      <td>36.025119</td>
      <td>129.352454</td>
      <td>BMW</td>
    </tr>
    <tr>
      <th>62</th>
      <td>부산 롯데 패스트레인 서비스 센터</td>
      <td>부산광역시 부산진구 신천대로 241</td>
      <td>동성모터스</td>
      <td>051-792-1810</td>
      <td>35.163307</td>
      <td>129.049316</td>
      <td>BMW</td>
    </tr>
    <tr>
      <th>63</th>
      <td>울산 진장 롯데 패스트레인</td>
      <td>울산 북구 진장유통로 64</td>
      <td>동성모터스</td>
      <td>052-702-8361</td>
      <td>35.576004</td>
      <td>129.359025</td>
      <td>BMW</td>
    </tr>
    <tr>
      <th>64</th>
      <td>제주 서비스 센터</td>
      <td>제주특별자치도 제주시 동화로 66-2</td>
      <td>도이치모터스</td>
      <td>064-757-7601</td>
      <td>33.514418</td>
      <td>126.572234</td>
      <td>BMW</td>
    </tr>
  </tbody>
</table>
<p>65 rows × 7 columns</p>
</div>




```python
Benz_sc = pd.read_csv('./Benz_all_sc.csv')
Benz_sc
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
      <th>ShopName</th>
      <th>address</th>
      <th>DealerName</th>
      <th>Tel</th>
      <th>lat</th>
      <th>lng</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>파주 서비스센터</td>
      <td>경기 파주시 문산읍 당동1로 66</td>
      <td>모터원</td>
      <td>031-972-5588</td>
      <td>37.864689</td>
      <td>126.772257</td>
    </tr>
    <tr>
      <th>1</th>
      <td>춘천 서비스센터</td>
      <td>강원 춘천시 온의동 228-17</td>
      <td>한성자동차</td>
      <td>033-903-5100</td>
      <td>37.862731</td>
      <td>127.717513</td>
    </tr>
    <tr>
      <th>2</th>
      <td>의정부 서비스센터</td>
      <td>경기 의정부시 시민로 309</td>
      <td>모터원</td>
      <td>031-841-5588</td>
      <td>37.737649</td>
      <td>127.069030</td>
    </tr>
    <tr>
      <th>3</th>
      <td>김포 서비스센터</td>
      <td>경기 김포시 양촌읍 모산로 14</td>
      <td>케이씨씨오토</td>
      <td>031-991-9700</td>
      <td>37.650996</td>
      <td>126.657015</td>
    </tr>
    <tr>
      <th>4</th>
      <td>일산 서비스센터</td>
      <td>경기 고양시 일산서구 법곳길 82</td>
      <td>모터원</td>
      <td>031-905-5588</td>
      <td>37.665676</td>
      <td>126.727816</td>
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
      <th>60</th>
      <td>해운대 서비스센터</td>
      <td>부산 해운대구 해운대로 287</td>
      <td>스타자동차</td>
      <td>1688-2369</td>
      <td>35.174866</td>
      <td>129.131849</td>
    </tr>
    <tr>
      <th>61</th>
      <td>남천 서비스센터</td>
      <td>부산 수영구 수영로 434</td>
      <td>한성자동차</td>
      <td>051-750-2800</td>
      <td>35.144918</td>
      <td>129.109999</td>
    </tr>
    <tr>
      <th>62</th>
      <td>울산 서비스센터</td>
      <td>울산 남구 남중로74번길 19</td>
      <td>스타자동차</td>
      <td>1688-2369</td>
      <td>35.541180</td>
      <td>129.349987</td>
    </tr>
    <tr>
      <th>63</th>
      <td>포항 서비스센터</td>
      <td>경북 포항시 남구 새천년대로 210</td>
      <td>중앙모터스</td>
      <td>054-256-9004</td>
      <td>36.004707</td>
      <td>129.324468</td>
    </tr>
    <tr>
      <th>64</th>
      <td>제주 서비스센터</td>
      <td>제주 제주시 연삼로 101</td>
      <td>케이씨씨오토</td>
      <td>064-800-9898</td>
      <td>33.493724</td>
      <td>126.503405</td>
    </tr>
  </tbody>
</table>
<p>65 rows × 6 columns</p>
</div>




```python
# brand 칼럼 추가
Benz_sc["brand"] = "Benz"
Benz_sc
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
      <th>ShopName</th>
      <th>address</th>
      <th>DealerName</th>
      <th>Tel</th>
      <th>lat</th>
      <th>lng</th>
      <th>brand</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>파주 서비스센터</td>
      <td>경기 파주시 문산읍 당동1로 66</td>
      <td>모터원</td>
      <td>031-972-5588</td>
      <td>37.864689</td>
      <td>126.772257</td>
      <td>Benz</td>
    </tr>
    <tr>
      <th>1</th>
      <td>춘천 서비스센터</td>
      <td>강원 춘천시 온의동 228-17</td>
      <td>한성자동차</td>
      <td>033-903-5100</td>
      <td>37.862731</td>
      <td>127.717513</td>
      <td>Benz</td>
    </tr>
    <tr>
      <th>2</th>
      <td>의정부 서비스센터</td>
      <td>경기 의정부시 시민로 309</td>
      <td>모터원</td>
      <td>031-841-5588</td>
      <td>37.737649</td>
      <td>127.069030</td>
      <td>Benz</td>
    </tr>
    <tr>
      <th>3</th>
      <td>김포 서비스센터</td>
      <td>경기 김포시 양촌읍 모산로 14</td>
      <td>케이씨씨오토</td>
      <td>031-991-9700</td>
      <td>37.650996</td>
      <td>126.657015</td>
      <td>Benz</td>
    </tr>
    <tr>
      <th>4</th>
      <td>일산 서비스센터</td>
      <td>경기 고양시 일산서구 법곳길 82</td>
      <td>모터원</td>
      <td>031-905-5588</td>
      <td>37.665676</td>
      <td>126.727816</td>
      <td>Benz</td>
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
    </tr>
    <tr>
      <th>60</th>
      <td>해운대 서비스센터</td>
      <td>부산 해운대구 해운대로 287</td>
      <td>스타자동차</td>
      <td>1688-2369</td>
      <td>35.174866</td>
      <td>129.131849</td>
      <td>Benz</td>
    </tr>
    <tr>
      <th>61</th>
      <td>남천 서비스센터</td>
      <td>부산 수영구 수영로 434</td>
      <td>한성자동차</td>
      <td>051-750-2800</td>
      <td>35.144918</td>
      <td>129.109999</td>
      <td>Benz</td>
    </tr>
    <tr>
      <th>62</th>
      <td>울산 서비스센터</td>
      <td>울산 남구 남중로74번길 19</td>
      <td>스타자동차</td>
      <td>1688-2369</td>
      <td>35.541180</td>
      <td>129.349987</td>
      <td>Benz</td>
    </tr>
    <tr>
      <th>63</th>
      <td>포항 서비스센터</td>
      <td>경북 포항시 남구 새천년대로 210</td>
      <td>중앙모터스</td>
      <td>054-256-9004</td>
      <td>36.004707</td>
      <td>129.324468</td>
      <td>Benz</td>
    </tr>
    <tr>
      <th>64</th>
      <td>제주 서비스센터</td>
      <td>제주 제주시 연삼로 101</td>
      <td>케이씨씨오토</td>
      <td>064-800-9898</td>
      <td>33.493724</td>
      <td>126.503405</td>
      <td>Benz</td>
    </tr>
  </tbody>
</table>
<p>65 rows × 7 columns</p>
</div>



### 서울시 자동차 용품점 데이터 불러오기


```python
seoul = pd.read_csv("./shop_seoul_202103.csv")
```


```python
seoul['상권업종대분류명'].unique()
```




    array(['음식', '소매', '학문/교육', '생활서비스', '부동산', '관광/여가/오락', '숙박', '스포츠'],
          dtype=object)




```python
seoul = seoul.loc[(seoul['상권업종대분류명']=='소매')|(seoul['상권업종대분류명']=='생활서비스')]
seoul_car = seoul.loc[seoul['상권업종중분류명'] == '자동차/자동차용품']
```


```python
seoul_car['상권업종소분류명'].unique()
```




    array(['타이어판매', '카오디오전문', '자동차부품판매', '중고자동차판매', '자동차판매', '네비게이션판매',
           '수입자동차판매', '자동차장식품판매', '자동차유리전문점', '수입중고자동차판매', '중고타이어판매'],
          dtype=object)




```python
seoul_car_sc = seoul_car.loc[seoul_car['상권업종소분류명'] == '자동차부품판매']
```

## BMW 서비스센터 & 차대차 교통사고 수&자동차 부품점 구별 지도시각화



```python
car_brand = {
'BMW' : './logo/2020_BMW.png',
'Benz' : './logo/benz.png',
}

#로고 아이콘
def makeicon(icon_image):
    icon = CustomIcon(
    icon_image=icon_image,
    icon_size=(25, 25),
    icon_anchor=(25, 25),
    shadow_image='',
    shadow_size=(5, 5),
    shadow_anchor=(4, 4),
    popup_anchor=(-3, -3)
    )
    return icon


# 구별이미지
def makeicon2(icon_image):
    icon2 = CustomIcon(
    icon_image=icon_image,
    icon_size=(35, 35),
    icon_anchor=(25, 25),
    shadow_image='',
    shadow_size=(5, 5),
    shadow_anchor=(4, 4),
    popup_anchor=(-3, -3)
    )
    return icon2


def make_maker(r):
    tmp = folium.Marker(
        location = [r['lat'],r['lng']], 
        tooltip='<b>매장정보<br>'+
                 '<br>매장이름 : '+r["ShopName"]+ 
                 '<br>주소 : '+r["address"]+
                 '<br>전화 : '+ r["Tel"]+ 
                 '<br>딜러 : '+ r["DealerName"],
        icon = makeicon(car_brand[r['brand']]))
    return tmp
```


```python
gu_area = pd.read_excel('./gu_coordinate.xlsx', engine='openpyxl' )
```


```python
gu_icon = {
    '송파구' : './gu_img/송파구.png',
    '영등포구' : './gu_img/영등포구.png',
    
}
def gu_maker(gu):
    tmp = folium.Marker(
        location = [gu['lat'],gu['lng']], 
        icon = makeicon2(gu_icon[gu['구']]))
    return tmp 
```


```python
# 전체화면 버튼
plugins.Fullscreen(
    position='topright',
    title='전체화면',
    title_cancel='나가기',
    force_separate_button=True
).add_to(map)


#서울시구별영역표시
geo_path = "./skorea_municipalities_geo_simple.json"
geo_str = json.load(open(geo_path, encoding="utf-8-sig"))


#지도시작점
map = folium.Map(location=[37.501826, 127.039780], zoom_start=10, tiles=None)
folium.TileLayer('CartoDB positron', name='차대차 교통사고').add_to(map)
#그룹


BMW_group = folium.FeatureGroup(name="BMW서비스센터").add_to(map)
Benz_group = folium.FeatureGroup(name="Benz서비스센터").add_to(map)
carshop_group = folium.FeatureGroup(name="자동차/자동차용품 매장 🔘").add_to(map)

Area_group = folium.FeatureGroup(name="자치구별").add_to(map)

for idx, gu in gu_area.iterrows():
    Area_group.add_child(gu_maker(gu))


# 1)버블차트

# 2)choropleth
map.choropleth(geo_data=geo_str, data=seoul_acci_gu,
               columns=["자치구", "차대차사고"],
               key_on="feature.id", fill_color="BuPu",
               fill_opacity=0.7, line_opacity=0.3,
               name="차대차사고 발생건수")



for idx, r in BMW_sc.iterrows():
    BMW_group.add_child(make_maker(r))

for idx, r in Benz_sc.iterrows():
    Benz_group.add_child(make_maker(r))


folium.LayerControl(collapsed=False).add_to(map)

for n in seoul_car_sc.index:
    folium.CircleMarker(location=[seoul_car_sc["위도"][n],seoul_car_sc["경도"][n]],
                                tooltip=('<b>매장정보'+'<p><b> 매장이름 : '+seoul_car_sc["상호명"][n] + '<p><b> 주소 : '
                                         +seoul_car_sc["도로명주소"][n]+
                                        '<p><b> 업종분류 : '+ seoul_car_sc["상권업종소분류명"][n]),
                      radius=5,
                      color="#ffffgg",fill_color="#777777",fill_opacity=0.4,fill=True).add_to(carshop_group)    

map
```

![car_vis]({{ site.url }}{{ site.baseurl }}/assets/images/bmw_car_vis.png)


- 사고가 많이 발생하는지역 : 강남구, 송파구, 서초구, 영등포구
- 사고가 많이 발생하면서, BMW거점이 없는지역 : 송파구, 영등포구
{: .notice}

### 신규 거점 추천 지역 : 송파구, 영등포구

- 교통사고량이 많고 BMW 서비스센터가 수가 적은 송파구, 영등포구 지역 중에서<br>자동차 부품점이 부족한 지역 부근에 BMW 신규 서비스센터를 설치한다면 고객 유입량이 많을 것으로 기대된다.
{: .notice--info}

