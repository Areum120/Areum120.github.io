---
title: "BMW Project: BMW 신규 거점 선정을 위한 서울시 타겟층, 입지비용 분석"
date: 2021-08-21
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
   - folium 
   - matplotlib
   - pandas
---


```python
# 데이터 분석
import pandas as pd
import openpyxl
import json

# 시각화
import matplotlib as mpl
import matplotlib.font_manager as fm
import matplotlib.pyplot as plt
import seaborn as sns

# 지도시각화
import folium
from folium import plugins
from folium.features import CustomIcon
import googlemaps

# 한글 폰트 사용
font_name = fm.FontProperties(fname="C:/Windows/Fonts/malgun.ttf").get_name()
plt.rc('font', family=font_name)

#경고창 무시
import warnings
warnings.filterwarnings(action='ignore')
```

지난번 서울시 교통사고량과 BMW 서비스센터 위치, 자동차부품점 등을 함께 분석해보았다. <br>
BMW 전시장과 서비스센터 신규 거점 선정을 위해서는 서울시 고객층인 서울시민의 인구통계 대비 차량 이용 수를 분석해볼 필요가 있었다. <br>
또한 입지 선정을 위한 상가 월세 등 지역 별 입점 비용을 함께 조사하여 최적의 입지를 추가로 도출해보기로 했다

## BMW 신규 거점 선정을 위한 분석 Report 

### 서울시 전시장 & 서비스센터 신규 거점 분석을 위한 고객층, 입지 비용 분석

     1) 서울시 타겟 고객층 분석  
           - 인구통계 분석
           - 차량 등록 수

     2) 서울시 입지 비용 분석
           - 아파트/오피스텔 월세
           - 상권별 중대형 상가 월세(원/㎡)

## 1) 서울시 타겟 고객층 분석  

### 서울시 인구통계 데이터 불러오기


```python
data_01 = pd.read_excel("./2019_seoul_population.xlsx", engine='openpyxl')
```

### 인구통계 데이터 전처리


```python
#dataframe 재구성

seoul_people=pd.DataFrame({"성별":data_01["성별"],
            "자치구":data_01["행정구역별(읍면동)"],
            "20대":data_01['20~24세']+data_01['25~29세'],
            "30대":data_01['30~34세']+data_01['35~39세'],
            "40대":data_01['40~44세']+data_01['45~49세'],
            "50대":data_01['50~54세']+data_01['55~59세'],               
                          })
seoul_people.head(2)
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
      <th>성별</th>
      <th>자치구</th>
      <th>20대</th>
      <th>30대</th>
      <th>40대</th>
      <th>50대</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>서울특별시</td>
      <td>685756</td>
      <td>718284</td>
      <td>725432</td>
      <td>711068</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>종로구</td>
      <td>11874</td>
      <td>9457</td>
      <td>10124</td>
      <td>11471</td>
    </tr>
  </tbody>
</table>
</div>




```python
seoul_sum_people = seoul_people[seoul_people['자치구'] == "서울특별시"]
seoul_sum_people.head(2)
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
      <th>성별</th>
      <th>자치구</th>
      <th>20대</th>
      <th>30대</th>
      <th>40대</th>
      <th>50대</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>서울특별시</td>
      <td>685756</td>
      <td>718284</td>
      <td>725432</td>
      <td>711068</td>
    </tr>
    <tr>
      <th>26</th>
      <td>2</td>
      <td>서울특별시</td>
      <td>722514</td>
      <td>713036</td>
      <td>736992</td>
      <td>754354</td>
    </tr>
  </tbody>
</table>
</div>




```python
seoul_people = seoul_people[seoul_people['자치구'] != "서울특별시"]
seoul_people_m = seoul_people[seoul_people['성별'] == 1]
seoul_people_f = seoul_people[seoul_people['성별'] == 2]
```


```python
seoul_people_m_idx = seoul_people_m.set_index("자치구")
seoul_people_f_idx = seoul_people_f.set_index("자치구")
```

## 인구통계 분석

- 전체 연령대 중 20대는 여성의 비율이 높고, 30대는 남성의 비율이 높음
- 송파구, 강서구 지역 주 고객 타겟층 3050 남성,여성 인구 밀도 높음 
- 거점이 없는 지역구 중 노원구,은평구,강동구,관악구 지역이 잠재고객이 많다.


```python
#히트맵 : 정렬된 결과를 한번에 쉽게 확인할 수 있게 해주는 그래프

plt.figure(figsize=(15,10))
sns.heatmap(seoul_people_m_idx[["20대","30대","40대","50대"]], cmap="Greens"
        , linewidths=3, annot=True, fmt="f")#linewidths 칸수띄우기 
plt.title("남성 인구 분포현황",size=20)
```




    Text(0.5, 1.0, '남성 인구 분포현황')



![pop_male]({{ site.url }}{{ site.baseurl }}/assets/images/output_14_1.png)
    



```python
#히트맵 : 정렬된 결과를 한번에 쉽게 확인할 수 있게 해주는 그래프

plt.figure(figsize=(15,10))
sns.heatmap(seoul_people_f_idx[["20대","30대","40대","50대"]], cmap="Blues"
        , linewidths=3, annot=True, fmt="f")#linewidths 칸수띄우기 
plt.title("여성 인구 분포현황",size=20)
```




    Text(0.5, 1.0, '여성 인구 분포현황')





![pop_female]({{ site.url }}{{ site.baseurl }}/assets/images/output_15_1.png)
    



```python
labels = ["남성","여성"]
colors = ['#56B567', '#6CAED6']

plt.figure(figsize = (15,10))
plt.subplot(221)
wedgeprops={'width': 0.7, 'edgecolor': 'w', 'linewidth': 5}
plt.pie(seoul_sum_people["20대"], 
        labels=labels, autopct='%.1f%%', startangle=90, counterclock=False, colors=colors, wedgeprops=wedgeprops,
        textprops = {'fontsize':15})
plt.title("20대",size=20)


plt.subplot(222)
wedgeprops={'width': 0.7, 'edgecolor': 'w', 'linewidth': 5}
plt.pie(seoul_sum_people["30대"],
        labels=labels, autopct='%.1f%%', startangle=90, counterclock=False, colors=colors, wedgeprops=wedgeprops,
        textprops = {'fontsize':15})
plt.title("30대",size=20)

plt.subplot(223)
wedgeprops={'width': 0.7, 'edgecolor': 'w', 'linewidth': 5}
plt.pie(seoul_sum_people["40대"],
        labels=labels, autopct='%.1f%%', startangle=90, counterclock=False, colors=colors, wedgeprops=wedgeprops,
        textprops = {'fontsize':15})
plt.title("40대",size=20)

plt.subplot(224)
wedgeprops={'width': 0.7, 'edgecolor': 'w', 'linewidth': 5}
plt.pie(seoul_sum_people["50대"],
        labels=labels, autopct='%.1f%%', startangle=90, counterclock=False, colors=colors, wedgeprops=wedgeprops,
        textprops = {'fontsize':15})
plt.title("50대",size=20)


plt.show()
```



![pop_percent]({{ site.url }}{{ site.baseurl }}/assets/images/output_16_0.png)
    



```python
seoulmap = pd.read_csv('./2020_population_density.txt', sep = "\t", encoding = "utf-8-sig")
seoulmap.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 26 entries, 0 to 25
    Data columns (total 5 columns):
     #   Column     Non-Null Count  Dtype  
    ---  ------     --------------  -----  
     0   기간         26 non-null     int64  
     1   지역         26 non-null     object 
     2   인구         26 non-null     object 
     3   면적         26 non-null     float64
     4   인구밀도(명/㎢)  26 non-null     object 
    dtypes: float64(1), int64(1), object(3)
    memory usage: 1.1+ KB
    


```python
seoulmap = seoulmap[seoulmap['지역'] != "합계"]
seoulmap.head(2)
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
      <th>지역</th>
      <th>인구</th>
      <th>면적</th>
      <th>인구밀도(명/㎢)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>2020</td>
      <td>종로구</td>
      <td>158,996</td>
      <td>23.91</td>
      <td>6,649</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2020</td>
      <td>중구</td>
      <td>134,635</td>
      <td>9.96</td>
      <td>13,517</td>
    </tr>
  </tbody>
</table>
</div>




```python
seoul_people_m.info()
seoul_people_f.info()

seoul_people_f.reset_index(inplace=True)
seoul_people_m.reset_index(inplace=True)
seoulmap.reset_index(inplace=True)

del seoul_people_f["index"]
del seoul_people_m["index"]
del seoulmap["index"]
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 25 entries, 1 to 25
    Data columns (total 6 columns):
     #   Column  Non-Null Count  Dtype 
    ---  ------  --------------  ----- 
     0   성별      25 non-null     int64 
     1   자치구     25 non-null     object
     2   20대     25 non-null     int64 
     3   30대     25 non-null     int64 
     4   40대     25 non-null     int64 
     5   50대     25 non-null     int64 
    dtypes: int64(5), object(1)
    memory usage: 1.4+ KB
    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 25 entries, 27 to 51
    Data columns (total 6 columns):
     #   Column  Non-Null Count  Dtype 
    ---  ------  --------------  ----- 
     0   성별      25 non-null     int64 
     1   자치구     25 non-null     object
     2   20대     25 non-null     int64 
     3   30대     25 non-null     int64 
     4   40대     25 non-null     int64 
     5   50대     25 non-null     int64 
    dtypes: int64(5), object(1)
    memory usage: 1.4+ KB
    


```python
density = pd.DataFrame({
    "자치구":seoulmap["지역"],
    "20-50인구_남자":seoul_people_m["20대"]+seoul_people_m["30대"]+seoul_people_m["40대"]+seoul_people_m["50대"],
    "20-50인구_여자":seoul_people_f["20대"]+seoul_people_f["30대"]+seoul_people_f["40대"]+seoul_people_f["50대"],
    "20-50인구_종합":seoul_people_f["20대"]+seoul_people_f["30대"]+seoul_people_f["40대"]+seoul_people_f["50대"]+
                seoul_people_m["20대"]+seoul_people_m["30대"]+seoul_people_m["40대"]+seoul_people_m["50대"],
    "면적":seoulmap["면적"]
    
    })
```


```python
density['20-50인구밀도(명/면적)'] = density['20-50인구_종합']/density['면적']
```


```python
density['20-50인구밀도(명/면적)'] = density['20-50인구밀도(명/면적)'].round(2)
density.head(2)
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
      <th>20-50인구_남자</th>
      <th>20-50인구_여자</th>
      <th>20-50인구_종합</th>
      <th>면적</th>
      <th>20-50인구밀도(명/면적)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>종로구</td>
      <td>42926</td>
      <td>44253</td>
      <td>87179</td>
      <td>23.91</td>
      <td>3646.13</td>
    </tr>
    <tr>
      <th>1</th>
      <td>중구</td>
      <td>37076</td>
      <td>37510</td>
      <td>74586</td>
      <td>9.96</td>
      <td>7488.55</td>
    </tr>
  </tbody>
</table>
</div>




```python
density.iloc[:,[0,-1]].head(2)
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
      <th>20-50인구밀도(명/면적)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>종로구</td>
      <td>3646.13</td>
    </tr>
    <tr>
      <th>1</th>
      <td>중구</td>
      <td>7488.55</td>
    </tr>
  </tbody>
</table>
</div>




```python
density_gu = density.iloc[:,[0,-1]]
```

### 인구밀도, 차량 등록 수

- 인구밀도가 높은지역 : 양천구, 동작구, 동대문구
    - 일구밀도가 높으면서, BMW거점이 없는 지역 : 강동구, 관악구, 노원구, 은평구, 동작구
- 차량 등록수가 많은지역 : 강서구, 강남구, 송파구
    - 차량 등록수가 많으면서, BMW거점이 없는지역 : 강동구, 관악구, 노원구, 은평구
- 동작구는 인구밀도가 높지만 상대적으로 30~50대 비중보다 20대 비중이 높다.

### 서울 자동차부품점 데이터 불러오기


```python
seoul = pd.read_csv("./2021_seoul_shop_info.csv")
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
seoul_car_sr = seoul_car.loc[(seoul_car['상권업종소분류명']=='자동차판매')|(seoul_car['상권업종소분류명']=='수입자동차판매')]
```


```python
seoul_car_sc = seoul_car.loc[seoul_car['상권업종소분류명'] == '자동차부품판매']
```

### 서울 차량 등록 대수 데이터 불러오기


```python
seoulcar = pd.read_csv('./seoul_car_reg_num.csv')
seoulcar = seoulcar.set_index("구")
seoulcar.head(2)
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
      <th>승용차등록수</th>
    </tr>
    <tr>
      <th>구</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>종로구</th>
      <td>49926.0</td>
    </tr>
    <tr>
      <th>중구</th>
      <td>52894.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
gu_area = pd.read_excel("./gu_area.xlsx", engine='openpyxl')
```


```python
car_brand = {
'BMW' : './2020_BMW.png',
'Benz' : './benz.png',
'Audi' : './Audi.png',
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
# icon img 용량 문제로 비워두기
gu_icon = {
    '강남구' : './logo/자치구(txt)/강남구.png',
    '강동구' : './logo/자치구(txt)/강동구.png',
    '강북구' : './logo/자치구(txt)/강북구.png',
    '강서구' : './logo/자치구(txt)/강서구.png',
    '관악구' : './logo/자치구(txt)/관악구.png',
    '광진구' : './logo/자치구(txt)/광진구.png',
    '구로구' : './logo/자치구(txt)/구로구.png',
    '금천구' : './logo/자치구(txt)/금천구.png',
    '노원구' : './logo/자치구(txt)/노원구.png',
    '도봉구' : './logo/자치구(txt)/도봉구.png',
    '동대문구' : './logo/자치구(txt)/동대문구.png',
    '동작구' : './logo/자치구(txt)/동작구.png',
    '마포구' : './logo/자치구(txt)/마포구.png',
    '서대문구' : './logo/자치구(txt)/서대문구.png',
    '서초구' : './logo/자치구(txt)/서초구.png',
    '성동구' : './logo/자치구(txt)/성동구.png',
    '성북구' : './logo/자치구(txt)/성북구.png',
    '송파구' : './logo/자치구(txt)/송파구.png',
    '양천구' : './logo/자치구(txt)/양천구.png',
    '영등포구' : './logo/자치구(txt)/영등포구.png',
    '용산구' : './logo/자치구(txt)/용산구.png',
    '은평구' : './logo/자치구(txt)/은평구.png',
    '종로구' : './logo/자치구(txt)/종로구.png',
    '중구' : './logo/자치구(txt)/중구.png',
    '중랑구' : './logo/자치구(txt)/중랑구.png'
    
}

```


```python
BMW = pd.read_csv('./bmw_shop_df/00_BMW_ALLdata.csv')
Benz = pd.read_csv('./benz_shop_df/00_Benz_ALLdata.csv')
Audi = pd.read_csv('./audi_shop_df/00_Audi_ALLdata.csv')
BMW_sc = pd.read_csv('./bmw_shop_df/01_BMW_ALL서비스센터.csv')
BMW_sr = pd.read_csv('./bmw_shop_df/02_BMW_ALL전시장.csv')
BMW_bps_sr = pd.read_csv('./bmw_shop_df/03_BMW_ALLBps전시장.csv')
Benz_sr = pd.read_csv('./benz_shop_df/02_Benz_ALL전시장.csv')

Benz_sr["brand"] = "Benz"
BMW_sr["brand"] = "BMW"
```


```python
#서울시구별영역표시
geo_path = "./skorea_municipalities_geo_simple.json"
geo_str = json.load(open(geo_path, encoding="utf-8-sig"))


#지도시작점
map = folium.Map(location=[37.501826, 127.039780], zoom_start=10, tiles=None)
folium.TileLayer('CartoDB positron', name='인구밀도, 차량 등록 수').add_to(map)


Area_group = folium.FeatureGroup(name="자치구별").add_to(map)
BMW_group = folium.FeatureGroup(name="BMW전시장").add_to(map)
Benz_group = folium.FeatureGroup(name="Benz전시장").add_to(map)
carshop_group = folium.FeatureGroup(name="기타자동차판매점 🔘").add_to(map)


# 2)choropleth
map.choropleth(geo_data=geo_str, data=density_gu,
               columns=[density_gu["자치구"], "20-50인구밀도(명/면적)"],
               key_on="feature.id", fill_color="YlOrBr",
               name="인구밀도")

map.choropleth(geo_data=geo_str, data=seoulcar,
               columns=[seoulcar.index, "승용차등록수"],
               key_on="feature.id", fill_color="YlGn",
               name="자동차등록수")

for n in seoul_car_sr.index:
    folium.CircleMarker(location=[seoul_car_sr["위도"][n],seoul_car_sr["경도"][n]],
                                tooltip=('<b>매장정보'+'<p><b> 매장이름 : '+seoul_car_sr["상호명"][n] + '<p><b> 주소 : '
                                         +seoul_car_sr["도로명주소"][n]+
                                        '<p><b> 업종분류 : '+ seoul_car_sr["상권업종소분류명"][n]),
                      radius=5,
                      color="#ffffgg",fill_color="#777777",fill_opacity=0.4,fill=True).add_to(carshop_group)

for idx, gu in gu_area.iterrows():
    Area_group.add_child(gu_maker(gu))

for idx, r in BMW_sr.iterrows():
    BMW_group.add_child(make_maker(r))

for idx, r in Benz_sr.iterrows():
    Benz_group.add_child(make_maker(r))


folium.LayerControl(collapsed=False).add_to(map)
    

map
```

![seoul_shop_loc1]({{ site.url }}{{ site.baseurl }}/assets/images/loc_cost1.png)

## 2) 서울시 입지 비용 분석

### 아파트/오피스텔 월세

- 부유한 잠재적 고객이 있는 지역 : 강남구, 서초구, 성동구, 송파구, 용산구,
- 전월세가 비교적 높고, BMW 거점이 없는지역 : 강동구, 노원구, 영등포구

### 서울 전월세 및 임대료 데이터 불러오기


```python
month_home2 = pd.read_csv('./2021_seoul_monthly.csv')
```


```python
month_home3 = pd.read_csv('./2021_seoul_charter.csv')
```


```python
month_home = pd.read_csv('./seoul_rent_average.csv')
month_home_gu = month_home.set_index("구")
month_home_gu.head(2)
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
      <th>임대료</th>
    </tr>
    <tr>
      <th>구</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>강남구</th>
      <td>93.200313</td>
    </tr>
    <tr>
      <th>강동구</th>
      <td>41.944282</td>
    </tr>
  </tbody>
</table>
</div>



### 서울보증금 평균값 불러오기


```python
month_home4 = pd.read_csv('./seoul_deposit_average.csv')
month_home_gu2 = month_home.set_index("구")
month_home_gu2.head(2)
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
      <th>임대료</th>
    </tr>
    <tr>
      <th>구</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>강남구</th>
      <td>93.200313</td>
    </tr>
    <tr>
      <th>강동구</th>
      <td>41.944282</td>
    </tr>
  </tbody>
</table>
</div>




```python
month_home4 = pd.pivot_table(month_home3, index=["구"], values=["보증금"])
```


```python
month_home4.to_csv('./seoul_deposit_average.csv')
```


```python
plt.figure(figsize = (15,10))
ax = sns.boxplot(month_home2['구'],month_home2['임대료'],palette="Set3")
ax.set_xticklabels(ax.get_xticklabels(), rotation=45, horizontalalignment='right')
plt.title('서울시 구별 아파트/오피스텔 월세 그래프',size= 20)
plt.ylabel("임대료(단위:만원)")
plt.grid(True)
plt.show()
```



![seoul_gu_monthly]({{ site.url }}{{ site.baseurl }}/assets/images/output_51_0.png)
    



```python
plt.figure(figsize = (15,10))
ax = sns.boxplot(month_home3['구'],month_home3['보증금'],palette="Set3")
ax.set_xticklabels(ax.get_xticklabels(), rotation=45, horizontalalignment='right')
plt.title('서울시 구별 아파트/오피스텔 전세 그래프',size= 20)
plt.ylabel("보증금(단위:만원)", size = 15)
plt.grid(True)
plt.show()
```



![seoul_gu_year]({{ site.url }}{{ site.baseurl }}/assets/images/output_52_0.png)
    



```python
plt.figure(figsize = (15,10))
ax = sns.barplot(month_home2['구'],month_home2['임대료'])
ax.set_xticklabels(ax.get_xticklabels(), rotation=45, horizontalalignment='right')
plt.title('서울시 자치구별 아파트/오피스텔 월세 평균값', size = 20)
plt.ylabel("임대료(단위:만원)", size = 15)
plt.show()
```



![seoul_gu_monthly_average]({{ site.url }}{{ site.baseurl }}/assets/images/output_53_0.png)

    



```python
plt.figure(figsize = (15,10))
ax = sns.barplot(month_home3['구'],month_home3['보증금'])
ax.set_xticklabels(ax.get_xticklabels(), rotation=45, horizontalalignment='right')
plt.title('서울시 자치구별 아파트/오피스텔 전세 평균값', size = 20)
plt.ylabel("보증금(단위:만원)", size = 15)
plt.show()
```



![seoul_gu_year_average]({{ site.url }}{{ site.baseurl }}/assets/images/output_54_0.png)

    



```python
month_home3
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
      <th>구</th>
      <th>임대건물</th>
      <th>전월세구분</th>
      <th>임대면적</th>
      <th>보증금</th>
      <th>임대료</th>
      <th>계약년도</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>마포구</td>
      <td>아파트</td>
      <td>전세</td>
      <td>84.98</td>
      <td>86100</td>
      <td>0</td>
      <td>2021</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3</td>
      <td>서대문구</td>
      <td>아파트</td>
      <td>전세</td>
      <td>114.90</td>
      <td>70000</td>
      <td>0</td>
      <td>2021</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4</td>
      <td>마포구</td>
      <td>다세대/연립</td>
      <td>전세</td>
      <td>32.81</td>
      <td>24150</td>
      <td>0</td>
      <td>2021</td>
    </tr>
    <tr>
      <th>3</th>
      <td>5</td>
      <td>마포구</td>
      <td>아파트</td>
      <td>전세</td>
      <td>84.98</td>
      <td>100000</td>
      <td>0</td>
      <td>2021</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6</td>
      <td>성동구</td>
      <td>아파트</td>
      <td>전세</td>
      <td>84.99</td>
      <td>100000</td>
      <td>0</td>
      <td>2021</td>
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
      <th>111621</th>
      <td>231975</td>
      <td>강서구</td>
      <td>오피스텔</td>
      <td>전세</td>
      <td>19.74</td>
      <td>10080</td>
      <td>0</td>
      <td>2021</td>
    </tr>
    <tr>
      <th>111622</th>
      <td>231982</td>
      <td>금천구</td>
      <td>오피스텔</td>
      <td>전세</td>
      <td>52.22</td>
      <td>30600</td>
      <td>0</td>
      <td>2021</td>
    </tr>
    <tr>
      <th>111623</th>
      <td>231983</td>
      <td>금천구</td>
      <td>오피스텔</td>
      <td>전세</td>
      <td>33.42</td>
      <td>24600</td>
      <td>0</td>
      <td>2021</td>
    </tr>
    <tr>
      <th>111624</th>
      <td>231984</td>
      <td>금천구</td>
      <td>오피스텔</td>
      <td>전세</td>
      <td>38.76</td>
      <td>25200</td>
      <td>0</td>
      <td>2021</td>
    </tr>
    <tr>
      <th>111625</th>
      <td>231985</td>
      <td>성북구</td>
      <td>오피스텔</td>
      <td>전세</td>
      <td>45.80</td>
      <td>31000</td>
      <td>0</td>
      <td>2021</td>
    </tr>
  </tbody>
</table>
<p>111626 rows × 8 columns</p>
</div>



### 서울시 상가 데이터 불러오기


```python
data_01 = pd.read_excel("./seoul_medium-sized_shop.xlsx", engine='openpyxl')
```


```python
data_01 = data_01.fillna(0)
```


```python
data_01['2021년 1분기'] = data_01['2021년 1분기'].astype(str)
```


```python
data_01["2021년 1분기"] = data_01["2021년 1분기"].str.replace(pat=r'[^\w]', repl=r"", regex=True)
```


```python
data_01["2021년 1분기"] = data_01["2021년 1분기"].str[0:]+"00"
```


```python
data_01['2021년 1분기'] = data_01['2021년 1분기'].astype(int)
data_01
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
      <th>상권</th>
      <th>2021년 1분기</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>남대문</td>
      <td>191400</td>
    </tr>
    <tr>
      <th>1</th>
      <td>동대문</td>
      <td>95600</td>
    </tr>
    <tr>
      <th>2</th>
      <td>을지로</td>
      <td>50700</td>
    </tr>
    <tr>
      <th>3</th>
      <td>종로</td>
      <td>45200</td>
    </tr>
    <tr>
      <th>4</th>
      <td>강남대로</td>
      <td>74300</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>236</th>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>237</th>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>238</th>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>239</th>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>240</th>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>241 rows × 2 columns</p>
</div>




```python
data_02 = data_01.sort_values('2021년 1분기',ascending=False)
```

### 상권별 중대형 상가 월세(원/㎡)

- 상권 임대료가 저렴하면서, BMW 거점이 없는 지역 : 관악구, 영등포구, 은평구, 강동구


```python
plt.figure(figsize=(15,10)) # size
plt.title('서울시 상권별 월세', size= 20)
plt.grid(True) #눈금
#plt.ylabel("지역 상권", size = 15)
#plt.xlabel("2021년 1분기 가격", size = 15)
plt.rotation=45
sns.barplot(x="2021년 1분기", y="상권", data=data_02)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x2ba2cfa4978>


![seoul_shop_monthly]({{ site.url }}{{ site.baseurl }}/assets/images/output_65_0.png)
    

### 서울 상가 위치 데이터 불러오기


```python
data_03 = pd.read_excel("./seoul_medium-sized_shop_loc.xlsx", engine='openpyxl')
```


```python
data_03['가격'] = data_03['가격'].astype(str)
```


```python
del data_03["Unnamed: 0"]
del data_03["Unnamed: 0.1"]
data_03.head(2)
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
      <th>상권</th>
      <th>2021년 1분기</th>
      <th>가격</th>
      <th>lat</th>
      <th>lng</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>중구</td>
      <td>남대문</td>
      <td>191400</td>
      <td>191400</td>
      <td>37.559741</td>
      <td>126.976957</td>
    </tr>
    <tr>
      <th>1</th>
      <td>동대문구</td>
      <td>동대문</td>
      <td>95600</td>
      <td>95600</td>
      <td>37.576329</td>
      <td>127.038193</td>
    </tr>
  </tbody>
</table>
</div>




```python
month_5 = data_03[data_03['2021년 1분기']<50000]
month_5_7 = data_03[(data_03['2021년 1분기'] > 50000) & (data_03['2021년 1분기'] <= 70000)]
month_7  = data_03[data_03['2021년 1분기']>70000]
month_10 = data_03[data_03['2021년 1분기']>100000]
month_7_10  = data_03[(data_03['2021년 1분기'] > 70000) & (data_03['2021년 1분기'] < 100000)]
```


```python
map = folium.Map(location=[37.501826, 127.039780], zoom_start=10,tiles=None)
folium.TileLayer('CartoDB positron', name='서울시 상권별 월세(원/㎡)').add_to(map)
#전체화면 버튼
plugins.Fullscreen(
    position='topright',
    title='전체화면',
    title_cancel='나가기',
    force_separate_button=True
).add_to(map)

group_5 = folium.FeatureGroup(name="5만원 미만 🔵").add_to(map)
group_5_7 = folium.FeatureGroup(name="5~7만원 이하 🟢").add_to(map)
group_7 = folium.FeatureGroup(name="7만원 초과 🔴").add_to(map)
for n in month_7.index:
    folium.CircleMarker(location=[month_7["lat"][n],
                            month_7["lng"][n]],
                        tooltip=('<b>상권정보'+
                                 '<br>상권지역 : '+month_7["자치구"][n] +                                  
                                 '<p><b> 상권이름 : '+month_7["상권"][n] + 
                                 '<p><b> 가격 : '+month_7["가격"][n]+"원"),
                        radius=35,
                           color="#ffffgg", fill_color="#DD3922",fill_opacity=0.4
                          ).add_to(group_7)

for n in month_5_7.index:
    folium.CircleMarker(location=[month_5_7["lat"][n],
                            month_5_7["lng"][n]],
                        tooltip=('<b>상권정보'+
                                 '<br>상권지역 : '+month_5_7["자치구"][n] +                                  
                                 '<p><b> 상권이름 : '+month_5_7["상권"][n] + 
                                 '<p><b> 가격 : '+month_5_7["가격"][n]+"원"),
                        radius=35,
                           color="#ffffgg", fill_color="#4EB26D",fill_opacity=0.4
                          ).add_to(group_5_7)

for n in month_5.index:
    folium.CircleMarker(location=[month_5["lat"][n],
                            month_5["lng"][n]],
                        tooltip=('<b>상권정보'+
                                 '<br>상권지역 : '+month_5["자치구"][n] +                                  
                                 '<p><b> 상권이름 : '+month_5["상권"][n] + 
                                 '<p><b> 가격 : '+month_5["가격"][n]+"원"),
                        radius=35,
                           color="#ffffgg", fill_color="#3354CC",fill_opacity=0.4
                          ).add_to(group_5)

folium.LayerControl(collapsed=False).add_to(map)
map
```

![seoul_shop_loc2]({{ site.url }}{{ site.baseurl }}/assets/images/loc_cost2.png)

```python
map = folium.Map(location=[37.501826, 127.039780], zoom_start=10,tiles=None)
folium.TileLayer('CartoDB positron', name='서울시 상권별 월세(원/㎡)').add_to(map)
#전체화면 버튼
plugins.Fullscreen(
    position='topright',
    title='전체화면',
    title_cancel='나가기',
    force_separate_button=True
).add_to(map)

group_5 = folium.FeatureGroup(name="5만원 미만 🔵").add_to(map)
group_5_7 = folium.FeatureGroup(name="5~7만원 이하 🟢").add_to(map)
group_7 = folium.FeatureGroup(name="7만원 초과 🔴").add_to(map)

for n in month_10.index:
    folium.CircleMarker(location=[month_10["lat"][n],
                            month_10["lng"][n]],
                        tooltip=('<b>상권정보'+
                                 '<br>상권지역 : '+month_10["자치구"][n] +                                  
                                 '<p><b> 상권이름 : '+month_10["상권"][n] + 
                                 '<p><b> 가격 : '+month_10["가격"][n]+"원"),
                        radius=month_10["2021년 1분기"][n]/4500,
                           color="#ffffgg", fill_color="#DD3922",fill_opacity=0.4
                          ).add_to(group_7)

for n in month_7_10.index:
    folium.CircleMarker(location=[month_7_10["lat"][n],
                            month_7_10["lng"][n]],
                        tooltip=('<b>상권정보'+
                                 '<br>상권지역 : '+month_7_10["자치구"][n] +                                  
                                 '<p><b> 상권이름 : '+month_7_10["상권"][n] + 
                                 '<p><b> 가격 : '+month_7_10["가격"][n]+"원"),
                        radius=month_7_10["2021년 1분기"][n]/3000,
                           color="#ffffgg", fill_color="#DD3922",fill_opacity=0.4
                          ).add_to(group_7)
for n in month_5_7.index:
    folium.CircleMarker(location=[month_5_7["lat"][n],
                            month_5_7["lng"][n]],
                        tooltip=('<b>상권정보'+
                                 '<br>상권지역 : '+month_5_7["자치구"][n] +                                  
                                 '<p><b> 상권이름 : '+month_5_7["상권"][n] + 
                                 '<p><b> 가격 : '+month_5_7["가격"][n]+"원"),
                        radius=month_5_7["2021년 1분기"][n]/3000,
                           color="#ffffgg", fill_color="#4EB26D",fill_opacity=0.4
                          ).add_to(group_5_7)
for n in month_5.index:
    folium.CircleMarker(location=[month_5["lat"][n],
                            month_5["lng"][n]],
                        tooltip=('<b>상권정보'+
                                 '<br>상권지역 : '+month_5["자치구"][n] +                                  
                                 '<p><b> 상권이름 : '+month_5["상권"][n] + 
                                 '<p><b> 가격 : '+month_5["가격"][n]+"원"),
                        radius=month_5["2021년 1분기"][n]/3000,
                           color="#ffffgg", fill_color="#3354CC",fill_opacity=0.4
                          ).add_to(group_5)
folium.LayerControl(collapsed=False).add_to(map)
map
```

![seoul_shop_loc3]({{ site.url }}{{ site.baseurl }}/assets/images/loc_cost3.png)


### 신규 거점 추천 지역 : 강동구, 관악구, 강서구, 영등포구 

- 강동구는 인구밀도,차량등록,사고발생 수, 아파트 전월세가 비교적 높고 상권 임대료가 저렴
	- 신규 전시장&서비스센터 거점으로 타당하다.   
- 관악구와 은평구는 인구밀도와 차량등록은 높지만 사고발생 수가 적고 임대료가 저렴
	- 신규 전시장 거점으로 타당하다.
- 강서구는 30-50남성,여성 인구가 많고 영등포구는 사고발생 수가 높으며 임대료가 비교적 저렴
	- 신규 서비스센터 거점으로 타당하다.


```python

```


```python

```
