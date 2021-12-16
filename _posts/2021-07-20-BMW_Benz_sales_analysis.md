---
title: "BMW Project: BMW VS 수입차브랜드 판매량 EDA 분석"
date: 2021-07-20
read_time: false
comments: false
share: false
related: false
categories:
  - BMW Project
tags:
  - BMW
  - data analysis
  - matplotlib
  - pandas

toc: true
---

수입차 브랜드하면 BMW, Benz 등의 대표적인 브랜드가 있다. (아우디는 최근 판매가 부진하다.)  <br>최근 BMW 데이터 분석 파트 업무를 맡게 되면서  BMW와 다른 수입차 브랜드와의 판매량을 비교하는 주제로 분석해보았다. <br> 한국의 BMW 판매 회사는 차를 수입하는 임포터사와 판매를 주로 담당하는 딜러사로 나뉜다. <br>임포터사는 BMW Korea로 독일 본사에서 세운 한국 지점이며 판매는 도이치모터스, 코오롱모터스, 바바리안 모터스 등의 딜러사와 Sales consultant가 주로 담당한다. <br>  여기서의 판매량 분석은 임포터사와 딜러사 통틀은 차량 판매 수를 의미한다. <br> 한국수입자동차협회에서 공개하는 데이터는 유료이므로 다나와 포털사이트의 자동차 데이터로 분석했다.

```python
# 데이터 분석
import pandas as pd
import openpyxl

# 시각화
import matplotlib.font_manager as fm
import matplotlib.pyplot as plt

# 한글 폰트 설정
font_name = fm.FontProperties(fname="C:/Windows/Fonts/malgun.ttf").get_name()
plt.rc('font', family=font_name)

# 경고창 무시
import warnings
warnings.filterwarnings(action='ignore')
```

## 2017-2021(1-5월) BMW 임포터 & 딜러사 동향 분석 Report 

   - 수입자동차 분기별 판매추이
   - BMW 연월간 판매추이
   - BMW Vs Benz 연월간 판매추이
   
### 연간 수입차브랜드 판매량 데이터 불러오기

```python
df = pd.read_csv('./17_21_6brand_year_sales.csv', encoding='utf-8')
df
# 2017
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
MB_quentity = MB.groupby(['year'])[['quentity']].sum()
#BMWMB_quentity
```


```python
#BMW
BMW = df.loc[df['brand'] == 'BMW']
```


```python
# BMW 
BMWYear = BMW.groupby(['year','car_name'])[['quentity']].sum()
BMWYear = BMWYear.reset_index()
```


```python
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
      <th>118</th>
      <td>2021</td>
      <td>Z4</td>
      <td>226</td>
    </tr>
    <tr>
      <th>119</th>
      <td>2021</td>
      <td>i3</td>
      <td>130</td>
    </tr>
  </tbody>
</table>
<p>120 rows × 3 columns</p>
</div>




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
BMW_quentity = BMW.groupby(['year'])[['quentity']].sum()
BMW_quentity
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
      <td>72129</td>
    </tr>
    <tr>
      <th>2018</th>
      <td>57149</td>
    </tr>
    <tr>
      <th>2019</th>
      <td>50733</td>
    </tr>
    <tr>
      <th>2020</th>
      <td>62487</td>
    </tr>
    <tr>
      <th>2021</th>
      <td>32337</td>
    </tr>
  </tbody>
</table>
</div>



### 분기별 수입차브랜드 판매량 데이터 불러오기

- BMW 2018년 3분기 화재사건 이후 판매량 회복 성공적
- Benz 판매량 변동이 큰 반면 BMW 판매량은 꾸준히 증가 두 브랜드 격차가 좁혀지고 있다.
{: .notice--info}

```python
Q_data2 = pd.read_excel("./17~21_quarterly_sales.xlsx", engine='openpyxl')
```

## 2017-2021(1-5월) BMW 외 5개 수입차 브랜드 분기별 판매량 시각화


```python
fig = plt.figure(figsize=(15,12)) ## 캔버스 생성
fig.set_facecolor('white') ## 캔버스 색상 설정
ax = fig.add_subplot() ## 그림 뼈대(프레임) 생성

ax.spines['right'].set_visible(False) ## 오른쪽 축 숨김
ax.spines['top'].set_visible(False) ## 위쪽 축 숨김


ax.plot(Q_data2['분기'],Q_data2['bmw판매량'],marker='o',label='BMW') ## 선그래프 생성
ax.plot(Q_data2['분기'],Q_data2['benz판매량'],marker='o',label='Benz') ## 선그래프 생성
ax.plot(Q_data2['분기'],Q_data2['audi판매량'],marker='o',label='Audi',alpha=0.4) ## 선그래프 생성
ax.plot(Q_data2['분기'],Q_data2['volkswagen판매량'],marker='o',label='Volkswagen',alpha=0.4) ## 선그래프 생성
ax.plot(Q_data2['분기'],Q_data2['mini판매량'],marker='o',label='MINI',alpha=0.4) ## 선그래프 생성
ax.plot(Q_data2['분기'],Q_data2['volvo판매량'],marker='o',label='VOLVO',alpha=0.4) ## 선그래프 생성

ax.legend(loc='upper left', fontsize=13) ## 범례 생성

## 평균값을 y 눈금에 추가한다.
yticks = list(ax.get_yticks())
yticks = sorted(yticks)

for y in yticks:
    ax.axhline(y,linestyle=(0,(5,2)),color='grey',alpha=0.5)
    

plt.yticks(fontsize=13) ## y축 눈금 크기 설정
plt.xticks(rotation=30,fontsize=13) ## x축 눈금 회전각도와 크기 설정

plt.ylabel('자동차판매 수',rotation=90, fontsize=15, color='black') ## y축 라벨
plt.xlabel('분기',fontsize=15, color='black') ## x축 라벨
plt.title('브랜드별 17-21년 분기 판매량 비교',fontsize=20) ## 타이틀 설정
plt.show()
```

![bmwbenzsales1]({{ site.url }}{{ site.baseurl }}/assets/images/bmwbenzsales1 (1).png)




```python
band6_2017 = pd.read_csv("./17_month_6brand_sales.csv")
band6_2018 = pd.read_csv("./18_month_6brand_sales.csv")
band6_2019 = pd.read_csv("./19_month_6brand_sales.csv")
band6_2020 = pd.read_csv("./20_month_6brand_sales.csv")
band6_2021 = pd.read_csv("./21_month_6brand_sales.csv")
band6_total = pd.read_csv("./17~21_all_sales_6brand.csv")
```


```python
band6_2017_bmw = band6_2017[band6_2017['브랜드'] == 'bmw']
band6_2018_bmw = band6_2018[band6_2018['브랜드'] == 'bmw']
band6_2019_bmw = band6_2019[band6_2019['브랜드'] == 'bmw']
band6_2020_bmw = band6_2020[band6_2020['브랜드'] == 'bmw']
band6_2021_bmw = band6_2021[band6_2021['브랜드'] == 'bmw']
band6_total = band6_total[band6_total['브랜드'] == 'bmw']
```


```python
band6_2017_bmw = band6_2017_bmw.loc[:,'1월':'12월'].transpose()
band6_2018_bmw = band6_2018_bmw.loc[:,'1월':'12월'].transpose()
band6_2019_bmw = band6_2019_bmw.loc[:,'1월':'12월'].transpose()
band6_2020_bmw = band6_2020_bmw.loc[:,'1월':'12월'].transpose()
band6_2021_bmw = band6_2021_bmw.loc[:,'1월':'5월'].transpose()
band6_total = band6_total.loc[:,'2017년':'2021년(1~5월)'].transpose()
```


```python
band6_2017_bmw.columns = ['2017_판매량']
band6_2018_bmw.columns = ['2018_판매량']
band6_2019_bmw.columns = ['2019_판매량']
band6_2020_bmw.columns = ['2020_판매량']
band6_2021_bmw.columns = ['2021_판매량']
band6_total.columns = ['17-21판매량']
band6_total.head(2)
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
      <th>17-21판매량</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2017년</th>
      <td>59624</td>
    </tr>
    <tr>
      <th>2018년</th>
      <td>50524</td>
    </tr>
  </tbody>
</table>
</div>




```python
mean_21= int(np.mean(band6_2021_bmw['2021_판매량']))
```

## 2017-2021(1-5월) BMW 외 5개 수입차 브랜드 연간 월별 판매량 시각화

- 17~20년 7월에 공통적으로 판매량이 적음
- 20년 8월 판매폭이 대폭증가
- 21년 판매추이와 19년도 판매추이와 비슷한 점으로 보아 20년8월 최고 판매량를 넘을것으로 추정
{: .notice--info}

```python
mean_sales = mean_21

fig = plt.figure(figsize=(15,10)) ## 캔버스 생성
fig.set_facecolor('white') ## 캔버스 색상 설정
ax = fig.add_subplot() ## 그림 뼈대(프레임) 생성

ax.spines['right'].set_visible(False) ## 오른쪽 축 숨김
ax.spines['top'].set_visible(False) ## 위쪽 축 숨김


ax.plot(band6_2018_bmw.index,band6_2017_bmw['2017_판매량'],marker='o',label='2017',alpha=0.4) ## 선그래프 생성
ax.plot(band6_2018_bmw.index,band6_2018_bmw['2018_판매량'],marker='o',label='2018',alpha=0.4) ## 선그래프 생성
ax.plot(band6_2019_bmw.index,band6_2019_bmw['2019_판매량'],marker='o',label='2019',alpha=0.4) ## 선그래프 생성
ax.plot(band6_2020_bmw.index,band6_2020_bmw['2020_판매량'],marker='o',label='2020',alpha=0.4) ## 선그래프 생성
ax.plot(band6_2021_bmw.index,band6_2021_bmw['2021_판매량'],marker='o',label='2021',alpha=1,color='red') ## 선그래프 생성

## 평균값을 y 눈금에 추가한다.
yticks2 = list(ax.get_yticks())
yticks2.append(mean_sales)
yticks2 = sorted(yticks2)

for y in yticks2:
    ax.axhline(y,linestyle=(0,(5,2)),color='grey',alpha=0.5)
    

ax.axhline(mean_sales,label='Mean',linestyle=(0,(9,2)),color='red',alpha=0.5) ## 평균값을 y좌표로 하는 수평선 생성


ax.text(0,mean_sales+100,f'2021 Mean of Sales : {mean_sales}',fontsize=13) ## 평균 매출 텍스트 출력
ax.legend(loc='upper left', fontsize=13) ## 범례 생성


plt.yticks(fontsize=13) ## y축 눈금 크기 설정
plt.xticks(rotation=30,fontsize=13) ## x축 눈금 회전각도와 크기 설정

plt.ylabel('자동차판매 수',rotation=90, fontsize=15, color='black') ## y축 라벨
plt.xlabel('날짜',fontsize=15, color='black') ## x축 라벨
plt.title('BMW 17-21년 판매량 비교',fontsize=20) ## 타이틀 설정
plt.show()
```


![bmwbenzsales2]({{ site.url }}{{ site.baseurl }}/assets/images/bmwbenzsales1 (2).png)


##  2017-2021(1-5월) BMW Vs Benz 연간 판매 추이 시각화

- 19년 두 브랜드 간 총 판매량 격차가 가장 큼
- 20-21년 두 브랜드 간 판매량 격차가 현저히 좁아졌으므로 21년 하반기 BMW 판매량이 Benz와 간격이 좁혀질 주목 여지 있음
{: .notice--info}

```python
year = [2017, 2018, 2019, 2020, 2021]
index = np.arange(len(year))

# 차트 그리기
fig = plt.figure(figsize=(20, 12)) # 차트 생성 및 사이즈 설정
ax = fig.add_subplot(1,1,1) # subplot 생성

ax.plot(index, BMW_quentity, label='BMW', marker='o', color='#236E96') 
ax.plot(index, MB_quentity, label='Benz', marker='o', color='#15B2D3') 

ax.set_title('BMW Vs Benz 연간 판매추이', fontsize=20) # 타이틀 설정
ax.set_ylabel('year', fontsize=14) # x축 설정
ax.set_xlabel('판매량', fontsize=14) # y축 설정

ax.legend(fontsize=12, loc='best') # 범례 설정 best로 해놓으면 가장 적절한 위치에 알아서 범례가 놓이게 됩니디

plt.xticks(index, year)
plt.show()
```


![bmwbenzsales3]({{ site.url }}{{ site.baseurl }}/assets/images/bmwbenzsales1 (3).png)


