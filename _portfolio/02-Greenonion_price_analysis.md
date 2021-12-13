---
title: "Analysis of factors that increase greenonion prices"
permalink: /portfolio/Analysis of factors that increase greenonion prices/
date: 2021-03-15
categories:
    - 대파가격분석
    - 상관관계분석
    - EDA
tags:
    - 공공데이터
    - pandas
    - matplotlib
    - plotly
    - json
    - requests

toc: true
---

## 대파 가격 상승 요인 분석
![GO_00]({{ site.url }}{{ site.baseurl }}/assets/images/greenonion0.png)
![GO_01]({{ site.url }}{{ site.baseurl }}/assets/images/greenonion1.jpg)

대파의 가격이 급등하는 것을 보면서 호기심을 느끼고 실제로 공공 데이터를 수집하고 분석해보면서
실생활에 데이터 분석 경험을 녹여 ‘농수산물 대파 가격 상승 요인 분석＇개인 프로젝트를 진행하였습니다.

---

| Project                               | 대파 가격 상승 요인 분석                                          |
| ------------------------------------------- | ----------------------------------------------------- |
| 배경 및 목적 | (배경) 2021년 3월 전년대비 대파 농수산물 가격 4배 이상 상승하였음 <br>(목적) 대파의 가격 상승 요인을 도매시장경락가격 및 기상청 날씨 데이터를 통해 분석, 파악하고 향후 대파 가격 예측에 활용할 수 있도록 함 |
| 수행 기간 | 2021.03.15 ~ 2021.04.15(약 1개월)|
| RNR | 개인 / 공공 데이터 API 수집 및 데이터 탐색, 데이터 시각화, 통계 분석| 
| 활용 데이터 | 농림축산물 도매시장 상세경락가격(공공데이터포털), 기상청 종관기상관측자료(ASOS)|


---
result : 
- 기상 요인 추가하여 EDA 분석 실시, '평균 최저 기온' feature가 대파 거래가 상승 주요 요인으로 도출하였습니다.

---

## Process
### 데이터 수집

1. 공공 데이터 포털 사이트에서 오픈API 활용가이드 분석 및 사용 신청합니다.

![GO_02]{{ site.url }}{{ site.baseurl }}/assets/images/greenonion2.png)
![GO_03]{{ site.url }}{{ site.baseurl }}/assets/images/greenonion3.png)

2. 2021 01~03월 3개월 기간의 60만건 데이터 Json 타입으로 데이터를 불러옵니다.

### 공공 데이터 API CAll
```python

import requests
import json
from datetime import datetime
import pandas


def call_api(prd_cd, date):

    # 공공데이터 API Call (농수산물 도매시장 경락가격 data)
    key='CTFUEzKcx7CIIBqAvwD%2BJ9MObIpZoglK5TtrmnaBqeaiaFS%2FDlPS2Bt%2Fq52xl%2B33kK8hAHvCMK1b7Mu77hN17w%3D%3D'

    # 제한갯수
    limit = '10000'
    url=f'http://apis.data.go.kr/B552895/openapi/service/OrgPriceAuctionService/getExactProdPriceList?ServiceKey={key}&pageNo=1&numOfRows={limit}&delngDe={date}&prdlstCd={prd_cd}&_type=json'
    #parameter 서버로 넘겨주는 정보

    res = requests.get(url)
    jo = json.loads(res.content)
    body = jo['response']['body']
    item = body['items']#items 안에 item

    # print(f'date:{date} len:{len(item)} body: {body["totalCount"]}')

    if item != "":  # item이 널이 아닐때(공휴일, 휴일 등 data가 없음)
        # json으로 저장
        file_name = f'res_js{date}_{prd_cd}.json'

        # 파일 저장
        with open(file_name, 'w+') as f:
            f.write(json.dumps(item))
            # print(file_name, 'save success')#로그 기록 찍기`


# dates 일정 기간의 날짜 출력 방법(2021 1월~2021 3월)
rct3 = pandas.date_range(start='20210227', end='20210315')
dt_list = rct3.strftime("%Y%m%d").to_list()

dates = []
for i in dt_list:
    # print(i)
    dates.append(i) #리스트에 일정 기간 날짜 데이터 값 넣기
    # dates = ['20210130']
for date in dates:
    # prd_cd = '1202'  # 품목코드
    print(date, 'start:', datetime.now())
    call_api('1202', date) #데이터 API call
    print(date, 'end:', datetime.now()) #1일 데 소요 기간 12달, 12분 소요
    print('------------------------')

```


### requests한 data 저장하기

3. API call data를 json or dataframe or excel로 저장합니다.

```python
import json, glob
import pandas as pd
import openpyxl

# json으로 requests한 data json으로 저장
filenames = glob.glob('res_json/*.json')
print(filenames)

for filename in filenames:
with open('res_js20210311_1202.json') as f:
jo = json.loads(f.read())
print(len(jo), jo)
print(filename, len(jo))

        # save to excel

# dataframe에 저장
for filename in filenames:
# df = pd.read_json(filename)
# print(df)

#excel에 저장
    df = pd.read_json(filename)
    spl = filename.split('.')
    df.to_excel(f'{spl[0]}.xlsx')
```

### json data csv파일로 저장하는 법

4. json파일을 csv파일로 저장할 수 있습니다.

```python

import glob
import json
import pandas as pd

#파라미터 path 입력하여 get_file_list 호출
def get_file_list(file_location):
    file_names = glob.glob(file_location)
    #glob 특정 파일만 출력하는 기능
    print(len(file_names))#파일 갯수
    return file_names

# 출력한 파일을 한줄씩 열어서 읽는 기능
def json_from_json_file_nm(filename):
    # print(filename)
    with open(filename, encoding='utf-8-sig') as f:
        return json.loads(f.read())
        
#json 경로
path = 'C:/Users/tjfsu/PycharmProjects/pythonProject/CallAPI/*.json'
fl = get_file_list(path)
print(fl)#65개의 json 파일 [] 한눈에 출력

#json파일들을 모두 호출하여 fl에 넣고
# fl에서 필요한 갯수만 호출하여 filename에 저장

json_list = []
#filename에 fl을 넣으면 전체가 하나로 합쳐져서 마지막것만 출력, 그래서 덮어씌운 1개파일만 만들어짐
for filename in fl:
    print(filename)
    jo = json_from_json_file_nm(filename)
    print(jo)
    if len(jo) != 0:#빈데이터는 건너띄기
         data = jo['item']
         json_list.append(data)
print(json_list)
print(json_list[0])#0102파일
print(json_list[1])#0104파일


for i in range(len(json_list)):
    df = pd.DataFrame(json_list[i])#데이터프레임 변환
    df2 = df.to_csv(f'{i}.csv', encoding='utf-8-sig')# csv파일로 저장
    print(df)


```

### EDA

- 2020 전국 도매시장 총 거래량 서울 강서, 가락시장 순으로 많았음.
- 전년대비 boxplot 시각화 4분위를 보았을 때 2021년 3월 대파 거래가 minmax범위가 전년대비 넓음 확인
- 대파 거래량 작년 대비 평균 23.3% 감소 확인
- 전국 대파 최대 산지 전남 지역 출하량이 한달 전 평균 최저기온 변수와 상관관계를 분석했을 때 피어슨 상관계수 -0.71로 강한
  음적 상관관계가 있음 파악, 평균 최저기온이 낮을 수록 대파 거래가격 상승했음을 알 수 있음.

### 전국 도매시장 대파 거래량 합계 시각화



```python
import plotly.offline as pyo
import plotly.graph_objs as go

trace = go.Bar(x=asc_df_MTV['도매시장명'], y=asc_df_MTV['거래량'],)
data = [trace]
layout = go.Layout(title='전국 도매시장 별 대파 거래량 합계 비교')
fig = go.Figure(data=data, layout=layout)
pyo.iplot(fig)
```

![GO_04]{{ site.url }}{{ site.baseurl }}/assets/images/greenonion4.png)

### 2021 1~3월 대파 총 거래량 전년대비 비교 시각화

```python

import plotly
import plotly.offline as pyo
import plotly.graph_objs as go
from plotly.offline import download_plotlyjs, init_notebook_mode, plot, iplot
init_notebook_mode(connected=True)


#꺾은선,누적막대(stacked)
trace1 = go.Bar(x=result4['month'], y=result4['2020'], name='2020')
trace2 = go.Bar(x=result4['month'], y=result4['2021'], name='2021')
trace3 = go.Scatter(x = result4['month'], y = result4['2020'], mode = 'lines', name = '2020lines')
trace4 = go.Scatter(x = result4['month'], y = result4['2021'], mode = 'lines', name = '2021lines')

data = [trace1, trace2, trace3, trace4]

layout = go.Layout(title='2020-2021 1~3월 대파 총 거래량', barmode='stack')

fig = go.Figure(data=data, layout=layout)
pyo.iplot(fig)

```

![GO_05]{{ site.url }}{{ site.baseurl }}/assets/images/greenonion5.png)

### 전국 산지별 월 대파 출하량 파악

```python
#누적막대
trace1 = go.Bar(x=result5['산지명'], y=result5['202001'], name='2020년1월')
trace2 = go.Bar(x=result5['산지명'], y=result5['202101'], name='2021년1월')
trace3 = go.Bar(x=result5['산지명'], y=result5['202002'], name='2020년2월')
trace4 = go.Bar(x=result5['산지명'], y=result5['202102'], name='2021년2월')
trace5 = go.Bar(x=result5['산지명'], y=result5['202003'], name='2020년3월')
trace6 = go.Bar(x=result5['산지명'], y=result5['202103'], name='2021년3월')

data = [trace1, trace2, trace3, trace4, trace5, trace6]

layout = go.Layout(title='2020-2021 1~3월 산지 별 대파 출하량', barmode='stack')

fig = go.Figure(data=data, layout=layout)
pyo.iplot(fig)
```

![GO_06]{{ site.url }}{{ site.baseurl }}/assets/images/greenonion6.png)

### 2020 VS 2021 3월 대파 가격 Boxplot


```python
# 3월
data = [Mar20['거래가격'], Mar21['거래가격']]

green_diamond = dict(markerfacecolor='r', marker='s')
plt.boxplot(data, flierprops=green_diamond)
plt.title("거래가격 Boxplot 시각화")
plt.show()
```

![GO_07]{{ site.url }}{{ site.baseurl }}/assets/images/greenonion7.png)

### 평균 거래가와 평균 최저기온 상관관계 Heatmap

```python
#전라남도 진도군 연월별 거래량, 거래가격 데이터프레임 만들기
Jindo_TradeVol_pri = pd.DataFrame([[2049449,3695,5.4,0.7,-4],[2107991,3182,5.3,1.9,-4.9],[863131,2783,5.4,0.7,-5],[1871637,12914,3.5,-1.3,-6.3],[1502533,17978,2.7,-2.2,-13.7],[749608,17989,5.6,0.5,-4.4]], columns = ['총거래량','평균거래가격','평균기온(한달 전)','평균최저기온(한달 전)','최저기온(한달 전)'], index = ['202001', '202002', '202003','202101','202102','202103'])   
Jindo_TradeVol_pri

#진도군 데이터 상관관계 (날씨요인 추가)
trade_corr2 = Jindo_TradeVol_pri.corr(method = 'pearson')
trade_corr2#피어슨 상관계수
# 값이 -0.7 ~ -0.3 이면, 뚜렷한 음적 상관관계

#진도군 데이터 상관관계 (날씨요인 추가)
trade_corr2 = Jindo_TradeVol_pri.corr(method = 'pearson')
trade_corr2#피어슨 상관계수
# 값이 -0.7 ~ -0.3 이면, 뚜렷한 음적 상관관계
```

![GO_08]{{ site.url }}{{ site.baseurl }}/assets/images/greenonion8.png)
