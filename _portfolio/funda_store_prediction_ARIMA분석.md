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


    
![png](../../Users/tjfsu/Downloads/funda_store_prediction_ARIMA분석/output_14_0.png)
    


## ARIMA 모델 ##

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




    
![png](../../Users/tjfsu/Downloads/funda_store_prediction_ARIMA분석/output_21_1.png)
    



```python
# stationary 확인

timeseries = salesbymonth['amount']
timeseries.rolling(12).mean().plot()#12개월평균
```




    <AxesSubplot:xlabel='transacted_date'>




    
![png](../../Users/tjfsu/Downloads/funda_store_prediction_ARIMA분석/output_22_1.png)
    



```python
timeseries.rolling(12).std().plot()#12개월분산
```




    <AxesSubplot:xlabel='transacted_date'>




    
![png](../../Users/tjfsu/Downloads/funda_store_prediction_ARIMA분석/output_23_1.png)
    



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



    
![png](../../Users/tjfsu/Downloads/funda_store_prediction_ARIMA분석/output_26_1.png)
    


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
    

## differencing ##


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




    
![png](../../Users/tjfsu/Downloads/funda_store_prediction_ARIMA분석/output_38_1.png)
    


## 2차 differencing ##


```python
salesbymonth['2nd diff'] = salesbymonth['1st diff'] - salesbymonth['1st diff'].shift(1)
```


```python
salesbymonth['2nd diff'].plot()
```




    <AxesSubplot:xlabel='transacted_date'>




    
![png](../../Users/tjfsu/Downloads/funda_store_prediction_ARIMA분석/output_41_1.png)
    



```python
adf_check(salesbymonth['2nd diff'].dropna())
```

    Stationary5.406048999294243e-07
    

## 3차 differencing ##


```python
salesbymonth['3rd diff'] = salesbymonth['2nd diff'] - salesbymonth['2nd diff'].shift(1)
```

### 1차 differencing보다 더 작아짐 1st, 2nd 둘다 사용 가능 ###


```python
salesbymonth['seasonal diff'] = salesbymonth['amount'] - salesbymonth['amount'].shift(12)
```


```python
salesbymonth['seasonal diff'].plot()
```




    <AxesSubplot:xlabel='transacted_date'>




    
![png](../../Users/tjfsu/Downloads/funda_store_prediction_ARIMA분석/output_47_1.png)
    



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




    
![png](../../Users/tjfsu/Downloads/funda_store_prediction_ARIMA분석/output_51_1.png)
    



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


    
![png](../../Users/tjfsu/Downloads/funda_store_prediction_ARIMA분석/output_59_0.png)
    



```python
plot_acf(salesbymonth['2nd diff'].dropna());
```


    
![png](../../Users/tjfsu/Downloads/funda_store_prediction_ARIMA분석/output_60_0.png)
    



```python
plot_acf(salesbymonth['seasonal 2nd diff'].dropna());
```


    
![png](../../Users/tjfsu/Downloads/funda_store_prediction_ARIMA분석/output_61_0.png)
    



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



    
![png](../../Users/tjfsu/Downloads/funda_store_prediction_ARIMA분석/output_62_1.png)
    



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



    
![png](../../Users/tjfsu/Downloads/funda_store_prediction_ARIMA분석/output_64_1.png)
    


### 2017년 3월 2018년 2월까지 주기성이 보임, 점차적으로 증가하다 2018에서는 유지에서 머무르고 있다 ###

### 파라미터 찾기 ###

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
!pip install pmdarima
```

    Requirement already satisfied: pmdarima in c:\programdata\anaconda3\envs\pydata\lib\site-packages (1.8.2)
    Requirement already satisfied: statsmodels!=0.12.0,>=0.11 in c:\programdata\anaconda3\envs\pydata\lib\site-packages (from pmdarima) (0.12.2)
    Requirement already satisfied: scikit-learn>=0.22 in c:\programdata\anaconda3\envs\pydata\lib\site-packages (from pmdarima) (0.22.2.post1)
    Requirement already satisfied: setuptools!=50.0.0,>=38.6.0 in c:\programdata\anaconda3\envs\pydata\lib\site-packages (from pmdarima) (52.0.0.post20210125)
    Requirement already satisfied: scipy>=1.3.2 in c:\programdata\anaconda3\envs\pydata\lib\site-packages (from pmdarima) (1.6.2)
    Requirement already satisfied: urllib3 in c:\programdata\anaconda3\envs\pydata\lib\site-packages (from pmdarima) (1.26.4)
    Requirement already satisfied: joblib>=0.11 in c:\programdata\anaconda3\envs\pydata\lib\site-packages (from pmdarima) (1.0.1)
    Requirement already satisfied: numpy~=1.19.0 in c:\programdata\anaconda3\envs\pydata\lib\site-packages (from pmdarima) (1.19.2)
    Requirement already satisfied: Cython!=0.29.18,>=0.29 in c:\programdata\anaconda3\envs\pydata\lib\site-packages (from pmdarima) (0.29.23)
    Requirement already satisfied: pandas>=0.19 in c:\programdata\anaconda3\envs\pydata\lib\site-packages (from pmdarima) (1.2.3)
    Requirement already satisfied: python-dateutil>=2.7.3 in c:\programdata\anaconda3\envs\pydata\lib\site-packages (from pandas>=0.19->pmdarima) (2.8.1)
    Requirement already satisfied: pytz>=2017.3 in c:\programdata\anaconda3\envs\pydata\lib\site-packages (from pandas>=0.19->pmdarima) (2021.1)
    Requirement already satisfied: six>=1.5 in c:\programdata\anaconda3\envs\pydata\lib\site-packages (from python-dateutil>=2.7.3->pandas>=0.19->pmdarima) (1.15.0)
    Requirement already satisfied: patsy>=0.5 in c:\programdata\anaconda3\envs\pydata\lib\site-packages (from statsmodels!=0.12.0,>=0.11->pmdarima) (0.5.1)
    


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
    

## 하이퍼 파라미터 정하기 ##

p=0, d=1, q=0



### AIC(Akaike Information Criterion) 값이 제일 작은 조합을 선택 ###

### 학습 및 검증 ###


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




    
![png](../../Users/tjfsu/Downloads/funda_store_prediction_ARIMA분석/output_77_1.png)
    



```python
result.resid.plot(kind='kde')
# 가우스분포 0이므로 모델 피팅이 어느정도 된 것
```




    <AxesSubplot:ylabel='Density'>




    
![png](../../Users/tjfsu/Downloads/funda_store_prediction_ARIMA분석/output_78_1.png)
    


## 예측  ##


```python
result.plot_predict()
```




    
![png](../../Users/tjfsu/Downloads/funda_store_prediction_ARIMA분석/output_80_0.png)
    




    
![png](../../Users/tjfsu/Downloads/funda_store_prediction_ARIMA분석/output_80_1.png)
    



```python
from sklearn.metrics import mean_squared_error
from math import sqrt
from pandas import datetime
from datetime import timedelta
```

## Out-of- sample 예측(5개월) ##


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




    
![png](../../Users/tjfsu/Downloads/funda_store_prediction_ARIMA분석/output_96_1.png)
    



```python
# predict함수로 예측할 때 
result.plot_predict(1,38)
```
