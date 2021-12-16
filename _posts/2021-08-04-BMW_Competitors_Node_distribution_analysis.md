---
title: "BMW Project: BMW 경쟁사 및 딜러사 거점 분포 분석"
date: 2021-08-04
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
---

BMW의 최대 경쟁사라면 Benz와 Audi라고 할 수 있다. (물론 Audi는 최근에 핀메기 많이 부진해졌다.)<br> 임포터사인 BMW Korea에서 Benz와 Audi의 거점 분포를 확인하고<br>각 딜러사별 거점 분포와 점유율을 살펴보는 EDA를 진행하였다.
   

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

## BMW 경쟁사 및 딜러사 거점 분포 분석 Report 


1. 2017-2021(1-5월) BMW 임포터 & 딜러사 동향

    1) 임포터
           - 경쟁사 거점 분포
    2) 딜러사
           - 딜러사 거점 분포
           - 딜러사 거점 점유율


### BMW, Benz, Audi, 서비스센터, 전시장 데이터 불러오기


```python
BMW = pd.read_csv('./bmw_shop_df/00_BMW_ALLdata.csv')
Benz = pd.read_csv('./benz_shop_df/00_Benz_ALLdata.csv')
Audi = pd.read_csv('./audi_shop_df/00_Audi_ALLdata.csv')
BMW_sc = pd.read_csv('./bmw_shop_df/01_BMW_ALL서비스센터.csv')
BMW_sr = pd.read_csv('./bmw_shop_df/02_BMW_ALL전시장.csv')
BMW_bps_sr = pd.read_csv('./bmw_shop_df/03_BMW_ALLBps전시장.csv')
Benz_sr = pd.read_csv('./benz_shop_df/02_Benz_ALL전시장.csv')
```


```python
Benz_sr["brand"] = "Benz"
BMW_sr["brand"] = "BMW"
```

### BMW 딜러사 위치 데이터 불러오기


```python
Kolon = BMW.loc[BMW['DealerName'].str.contains('코오롱모터스', na=False)]
Handok = BMW.loc[BMW['DealerName'].str.contains('한독모터스', na=False)]
DongSung = BMW.loc[BMW['DealerName'].str.contains('동성모터스', na=False)]
Bavarian = BMW.loc[BMW['DealerName'].str.contains('바바리안모터스', na=False)]
Deutsch = BMW.loc[BMW['DealerName'].str.contains('도이치모터스', na=False)]
National = BMW.loc[BMW['DealerName'].str.contains('내쇼날모터스', na=False)]
Samchully = BMW.loc[BMW['DealerName'].str.contains('삼천리모터스', na=False)]
```


```python
#인덱스 재설정
Kolon.reset_index(inplace=True)
Handok.reset_index(inplace=True)
DongSung.reset_index(inplace=True)
Bavarian.reset_index(inplace=True)
Deutsch.reset_index(inplace=True)
National.reset_index(inplace=True)
Samchully.reset_index(inplace=True)
```


```python
#기존 인덱스 삭제
del Kolon["index"]
del Handok["index"]
del DongSung["index"]
del Bavarian["index"]
del Deutsch["index"]
del National["index"]
del Samchully["index"]
```

### 1) 임포터사 

### 경쟁사 거점 분포


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
map = folium.Map(location=[35.775301, 128.029343], zoom_start=6.5, tiles=None)

#전체화면 버튼
plugins.Fullscreen(
    position='topright',
    title='전체화면',
    title_cancel='나가기',
    force_separate_button=True
).add_to(map)

#folium.TileLayer('OpenStreetMap', name='Maptype1').add_to(map)
#folium.TileLayer('Stamen Terrain', name='Maptype2').add_to(map) #산악맵
#folium.TileLayer('Stamen Toner', name='Maptype3').add_to(map)   #다크맵
#folium.TileLayer('Stamen Watercolor', name='Maptype4').add_to(map)
#folium.TileLayer('CartoDB positron', name='Maptype5').add_to(map)   #화이트맵
#folium.TileLayer('CartoDB dark_matter', name='Maptype6').add_to(map)

folium.TileLayer('CartoDB positron', name='경쟁사 거점 분포').add_to(map)
#folium.TileLayer('Stamen Toner', name='Maptype2').add_to(map)
#folium.TileLayer('Stamen Terrain', name='Maptype3').add_to(map)


BMW_group = folium.FeatureGroup(name="BMW").add_to(map)
Benz_group = folium.FeatureGroup(name="Benz").add_to(map)
Audi_group = folium.FeatureGroup(name="Audi").add_to(map)



for idx, r in BMW.iterrows():
    BMW_group.add_child(make_maker(r))

for idx, r in Benz.iterrows():
    Benz_group.add_child(make_maker(r))

for idx, r in Audi.iterrows():
    Audi_group.add_child(make_maker(r))
    
folium.LayerControl(collapsed=False).add_to(map)
map
```

### 2) 딜러사

### 딜러사 거점 분포


```python
map = folium.Map(location=[35.775301, 128.029343], zoom_start=6.5, tiles=None)
folium.TileLayer('Stamen Toner', name='딜러사별 거점 현황').add_to(map)
mcg = folium.plugins.MarkerCluster(control=False)
map.add_child(mcg)

g1 = folium.plugins.FeatureGroupSubGroup(mcg, '코오롱모터스(red)')
map.add_child(g1)
g2 = folium.plugins.FeatureGroupSubGroup(mcg, '한독모터스(blue)')
map.add_child(g2)
g3 = folium.plugins.FeatureGroupSubGroup(mcg, '바바리안모터스(green)')
map.add_child(g3)
g4 = folium.plugins.FeatureGroupSubGroup(mcg, '도이치모터스(purple)')
map.add_child(g4)
g5 = folium.plugins.FeatureGroupSubGroup(mcg, '내쇼날모터스(orange)')
map.add_child(g5)
g6 = folium.plugins.FeatureGroupSubGroup(mcg, '삼천리모터스(darkred)')
map.add_child(g6)
g7 = folium.plugins.FeatureGroupSubGroup(mcg, '동성모터스(pink)')
map.add_child(g7)




#코오롱모터스
for n in Kolon.index:
    folium.Marker(location=[Kolon["lat"][n],Kolon["lng"][n]],
                                tooltip=('<b>매장정보'+'<p><b> 매장이름 : '+Kolon["ShopName"][n] + '<p><b> 주소 : '
                                         +Kolon["address"][n]+
                                        '<p><b> 전화 : '+ Kolon["Tel"][n] + '<p><b> 딜러 : '+ Kolon["DealerName"][n]),
                      radius=10,
                      icon=folium.Icon(color = 'red')).add_to(g1)

#한독모터스
for n in Handok.index:
    folium.Marker(location=[Handok["lat"][n],Handok["lng"][n]],
                                tooltip=('<b>매장정보'+'<p><b> 매장이름 : '+Handok["ShopName"][n] + '<p><b> 주소 : '
                                         +Handok["address"][n]+
                                        '<p><b> 전화 : '+ Handok["Tel"][n] + '<p><b> 딜러 : '+ Handok["DealerName"][n]),
                      radius=10,
                      icon=folium.Icon(color = 'blue')).add_to(g2)
#바바리안모터스
for n in Bavarian.index:
    folium.Marker(location=[Bavarian["lat"][n],Bavarian["lng"][n]],
                                tooltip=('<b>매장정보'+'<p><b> 매장이름 : '+Bavarian["ShopName"][n] + '<p><b> 주소 : '
                                         +Bavarian["address"][n]+
                                        '<p><b> 전화 : '+ Bavarian["Tel"][n] + '<p><b> 딜러 : '+ Bavarian["DealerName"][n]),
                      radius=10,
                      icon=folium.Icon(color = 'green')).add_to(g3)
#도이치모터스
for n in Deutsch.index:
    folium.Marker(location=[Deutsch["lat"][n],Deutsch["lng"][n]],
                                tooltip=('<b>매장정보'+'<p><b> 매장이름 : '+Deutsch["ShopName"][n] + '<p><b> 주소 : '
                                         +Deutsch["address"][n]+
                                        '<p><b> 전화 : '+ Deutsch["Tel"][n] + '<p><b> 딜러 : '+ Deutsch["DealerName"][n]),
                      radius=10,
                      icon=folium.Icon(color = 'purple')).add_to(g4)
#내쇼날모터스
for n in National.index:
    folium.Marker(location=[National["lat"][n],National["lng"][n]],
                                tooltip=('<b>매장정보'+'<p><b> 매장이름 : '+National["ShopName"][n] + '<p><b> 주소 : '
                                         +National["address"][n]+
                                        '<p><b> 전화 : '+ National["Tel"][n] + '<p><b> 딜러 : '+ National["DealerName"][n]),
                      radius=10,
                      icon=folium.Icon(color = 'orange')).add_to(g5)
#삼천리모터스
for n in Samchully.index:
    folium.Marker(location=[Samchully["lat"][n],Samchully["lng"][n]],
                                tooltip=('<b>매장정보'+'<p><b> 매장이름 : '+Samchully["ShopName"][n] + '<p><b> 주소 : '
                                         +Samchully["address"][n]+
                                        '<p><b> 전화 : '+ Samchully["Tel"][n] + '<p><b> 딜러 : '+ Samchully["DealerName"][n]),
                      radius=10,
                      icon=folium.Icon(color = 'darkred')).add_to(g6)

#동성모터스
for n in DongSung.index:
    folium.Marker(location=[DongSung["lat"][n],DongSung["lng"][n]],
                                tooltip=('<b>매장정보'+'<p><b> 매장이름 : '+DongSung["ShopName"][n] + '<p><b> 주소 : '
                                         +DongSung["address"][n]+
                                        '<p><b> 전화 : '+ DongSung["Tel"][n] + '<p><b> 딜러 : '+ DongSung["DealerName"][n]),
                      radius=10,
                      icon=folium.Icon(color = 'pink')).add_to(g7)
    

folium.LayerControl(collapsed=False).add_to(map)

map
```


```python
bmw_Dealer_sc_cnt = pd.read_excel("./BMW_delears_sc_position.xlsx", engine='openpyxl')
bmw_Dealer_sr_cnt = pd.read_excel("./BMW_delears_sr_position.xlsx", engine='openpyxl')
bmw_Dealer_bps_sr_cnt = pd.read_excel("./BMW__delears_bps_sc_position.xlsx", engine='openpyxl')
```

### 딜러사 거점 점유율

- 코오롱과 도이치 모터스가 대부분의 점유율을 차지함
- 바바리안모터스특징은 세일즈 중심의 운영, 동성모터스는 서비스센터에 비중이 높음
    - 서비스 센터 : 한독 > 동성 > 바바리안
    - 전시장      : 한독 > 바바리안 > 동성
    - BPS         : 한독 > 바바리안 > 동성



```python
labels = ["코오롱모터스","도이치모터스","한독모터스","동성모터스","바바리안모터스"
          ,"삼천리모터스","내쇼날모터스"]

colors = ['#3274A1', '#E1812C','#3A923A','#C03D3E','#9372B2','#845B53','#D684BD']



plt.figure(figsize = (17,17))
plt.subplot(222)
wedgeprops={'width': 0.7, 'edgecolor': 'w', 'linewidth': 5}
plt.pie(bmw_Dealer_sc_cnt["cnt"], 
        labels=labels, autopct='%.1f%%', startangle=180, counterclock=False, colors=colors, wedgeprops=wedgeprops,
        textprops = {'fontsize':15})
plt.title("BMW 서비스센터 점유율",size=20)




plt.subplot(221)
ax = sns.barplot(bmw_Dealer_sc_cnt['DealerName'],bmw_Dealer_sc_cnt['cnt'])
ax.set_xticklabels(ax.get_xticklabels(), rotation=45, horizontalalignment='right')
plt.title('BMW 딜러사별 서비스센터', size = 20)
plt.xlabel("딜러사", size = 13)
plt.ylabel("거점 수", size = 13)
plt.show()
```



![bmwcompet1]({{ site.url }}{{ site.baseurl }}/assets/images/bmwcompet (1).png)



```python
labels = ["코오롱모터스","도이치모터스","한독모터스","바바리안모터스","동성모터스"
          ,"삼천리모터스","내쇼날모터스"]

colors = ['#3274A1', '#E1812C','#3A923A','#C03D3E','#9372B2','#845B53','#D684BD']



plt.figure(figsize = (17,17))
plt.subplot(222)
wedgeprops={'width': 0.7, 'edgecolor': 'w', 'linewidth': 5}
plt.pie(bmw_Dealer_sr_cnt["cnt"], 
        labels=labels, autopct='%.1f%%', startangle=180, counterclock=False, colors=colors, wedgeprops=wedgeprops,
        textprops = {'fontsize':15})
plt.title("BMW 전시장 점유율",size=20)




plt.subplot(221)
ax = sns.barplot(bmw_Dealer_sr_cnt['DealerName'],bmw_Dealer_sr_cnt['cnt'])
ax.set_xticklabels(ax.get_xticklabels(), rotation=45, horizontalalignment='right')
plt.title('BMW 딜러사별 전시장', size = 20)
plt.xlabel("딜러사", size = 13)
plt.ylabel("거점 수", size = 13)
plt.show()
```


![bmwcompet2]({{ site.url }}{{ site.baseurl }}/assets/images/bmwcompet (2).png)



```python
labels = ["코오롱모터스","도이치모터스","한독모터스","바바리안모터스","동성모터스"
          ,"삼천리모터스","내쇼날모터스"]

colors = ['#3274A1', '#E1812C','#3A923A','#C03D3E','#9372B2','#845B53','#D684BD']



plt.figure(figsize = (17,17))
plt.subplot(222)
wedgeprops={'width': 0.7, 'edgecolor': 'w', 'linewidth': 5}
plt.pie(bmw_Dealer_bps_sr_cnt["cnt"], 
        labels=labels, autopct='%.1f%%', startangle=180, counterclock=False, colors=colors, wedgeprops=wedgeprops,
        textprops = {'fontsize':15})
plt.title("BMW 중고차 전시장 점유율",size=20)




plt.subplot(221)
ax = sns.barplot(bmw_Dealer_bps_sr_cnt['DealerName'],bmw_Dealer_bps_sr_cnt['cnt'])
ax.set_xticklabels(ax.get_xticklabels(), rotation=45, horizontalalignment='right')
plt.title('BMW 딜러사별 중고차 전시장', size = 20)
plt.xlabel("딜러사", size = 13)
plt.ylabel("거점 수", size = 13)
plt.show()
```


![bmwcompet3]({{ site.url }}{{ site.baseurl }}/assets/images/bmwcompet (3).png)

