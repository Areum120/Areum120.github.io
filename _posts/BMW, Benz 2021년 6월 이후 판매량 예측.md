```python
## BMW, Benz 2021년 6월 이후 판매량 예측
```


```python
# 데이터 분석
import pandas as pd
import numpy as np

# 시계열 예측
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf
from pylab import rcParams
import statsmodels.api as sm
import itertools
from statsmodels.tsa.ar_model import AR
from statsmodels.tsa.arima_model import ARIMA
from statsmodels.tsa.seasonal import seasonal_decompose
from statsmodels.tsa.stattools import adfuller#statiionaty검증

# datetime
from pandas import datetime

#경고창무시 
import warnings
warnings.filterwarnings(action='ignore')

# 데이터 시각화
import matplotlib.pyplot as plt
import seaborn as sns
import matplotlib as mpl
import matplotlib.font_manager as fm
font_name = fm.FontProperties(fname="C:/Windows/Fonts/malgun.ttf").get_name()
plt.rc('font', family=font_name)
```


```python
df = pd.read_csv('.\\17_21_6brands_month_sales.csv', encoding='utf-8')
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
      <th>브랜드</th>
      <th>2017-01-01</th>
      <th>2017-02-01</th>
      <th>2017-03-01</th>
      <th>2017-04-01</th>
      <th>2017-05-01</th>
      <th>2017-06-01</th>
      <th>2017-07-01</th>
      <th>2017-08-01</th>
      <th>...</th>
      <th>2020-08-01</th>
      <th>2020-09-01</th>
      <th>2020-10-01</th>
      <th>2020-11-01</th>
      <th>2020-12-01</th>
      <th>2021-01-01</th>
      <th>2021-02-01</th>
      <th>2021-03-01</th>
      <th>2021-04-01</th>
      <th>2021-05-01</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>bmw</td>
      <td>2415</td>
      <td>3202</td>
      <td>6164</td>
      <td>6334</td>
      <td>5373</td>
      <td>5510</td>
      <td>3188</td>
      <td>4105</td>
      <td>...</td>
      <td>7252</td>
      <td>5275</td>
      <td>5320</td>
      <td>5551</td>
      <td>5749</td>
      <td>5717</td>
      <td>5660</td>
      <td>6012</td>
      <td>6113</td>
      <td>6257</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>benz</td>
      <td>6848</td>
      <td>5534</td>
      <td>6737</td>
      <td>5758</td>
      <td>5063</td>
      <td>7783</td>
      <td>5471</td>
      <td>5267</td>
      <td>...</td>
      <td>6030</td>
      <td>5958</td>
      <td>6576</td>
      <td>7186</td>
      <td>9546</td>
      <td>5918</td>
      <td>5707</td>
      <td>7597</td>
      <td>8430</td>
      <td>7690</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>audi</td>
      <td>474</td>
      <td>360</td>
      <td>83</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>2022</td>
      <td>2528</td>
      <td>2527</td>
      <td>2906</td>
      <td>3109</td>
      <td>2302</td>
      <td>2362</td>
      <td>2737</td>
      <td>1320</td>
      <td>229</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>volkswagen</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>881</td>
      <td>872</td>
      <td>1933</td>
      <td>2677</td>
      <td>2729</td>
      <td>1236</td>
      <td>1783</td>
      <td>1628</td>
      <td>1080</td>
      <td>1358</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>mini</td>
      <td>541</td>
      <td>582</td>
      <td>624</td>
      <td>743</td>
      <td>1013</td>
      <td>841</td>
      <td>794</td>
      <td>826</td>
      <td>...</td>
      <td>1107</td>
      <td>1108</td>
      <td>890</td>
      <td>940</td>
      <td>1093</td>
      <td>712</td>
      <td>895</td>
      <td>1224</td>
      <td>1051</td>
      <td>1095</td>
    </tr>
    <tr>
      <th>5</th>
      <td>5</td>
      <td>volvo</td>
      <td>436</td>
      <td>570</td>
      <td>675</td>
      <td>542</td>
      <td>596</td>
      <td>693</td>
      <td>624</td>
      <td>602</td>
      <td>...</td>
      <td>336</td>
      <td>801</td>
      <td>1449</td>
      <td>1267</td>
      <td>1352</td>
      <td>1198</td>
      <td>1202</td>
      <td>1251</td>
      <td>1263</td>
      <td>1264</td>
    </tr>
  </tbody>
</table>
<p>6 rows × 55 columns</p>
</div>




```python
# BMW data 추출
bmw = df.loc[0]
bmw = bmw.reset_index()
bmw
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
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Unnamed: 0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>브랜드</td>
      <td>bmw</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2017-01-01</td>
      <td>2415</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2017-02-01</td>
      <td>3202</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-03-01</td>
      <td>6164</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2017-04-01</td>
      <td>6334</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2017-05-01</td>
      <td>5373</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2017-06-01</td>
      <td>5510</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2017-07-01</td>
      <td>3188</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2017-08-01</td>
      <td>4105</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2017-09-01</td>
      <td>5299</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2017-10-01</td>
      <td>4400</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2017-11-01</td>
      <td>6827</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2017-12-01</td>
      <td>6807</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2018-01-01</td>
      <td>5407</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2018-02-01</td>
      <td>6118</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2018-03-01</td>
      <td>7052</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2018-04-01</td>
      <td>6573</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2018-05-01</td>
      <td>5222</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2018-06-01</td>
      <td>4196</td>
    </tr>
    <tr>
      <th>20</th>
      <td>2018-07-01</td>
      <td>3959</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2018-08-01</td>
      <td>2383</td>
    </tr>
    <tr>
      <th>22</th>
      <td>2018-09-01</td>
      <td>2052</td>
    </tr>
    <tr>
      <th>23</th>
      <td>2018-10-01</td>
      <td>2131</td>
    </tr>
    <tr>
      <th>24</th>
      <td>2018-11-01</td>
      <td>2476</td>
    </tr>
    <tr>
      <th>25</th>
      <td>2018-12-01</td>
      <td>2955</td>
    </tr>
    <tr>
      <th>26</th>
      <td>2019-01-01</td>
      <td>2726</td>
    </tr>
    <tr>
      <th>27</th>
      <td>2019-02-01</td>
      <td>2340</td>
    </tr>
    <tr>
      <th>28</th>
      <td>2019-03-01</td>
      <td>2999</td>
    </tr>
    <tr>
      <th>29</th>
      <td>2019-04-01</td>
      <td>3226</td>
    </tr>
    <tr>
      <th>30</th>
      <td>2019-05-01</td>
      <td>3383</td>
    </tr>
    <tr>
      <th>31</th>
      <td>2019-06-01</td>
      <td>3292</td>
    </tr>
    <tr>
      <th>32</th>
      <td>2019-07-01</td>
      <td>3755</td>
    </tr>
    <tr>
      <th>33</th>
      <td>2019-08-01</td>
      <td>4291</td>
    </tr>
    <tr>
      <th>34</th>
      <td>2019-09-01</td>
      <td>4249</td>
    </tr>
    <tr>
      <th>35</th>
      <td>2019-10-01</td>
      <td>4122</td>
    </tr>
    <tr>
      <th>36</th>
      <td>2019-11-01</td>
      <td>4678</td>
    </tr>
    <tr>
      <th>37</th>
      <td>2019-12-01</td>
      <td>5130</td>
    </tr>
    <tr>
      <th>38</th>
      <td>2020-01-01</td>
      <td>2708</td>
    </tr>
    <tr>
      <th>39</th>
      <td>2020-02-01</td>
      <td>3812</td>
    </tr>
    <tr>
      <th>40</th>
      <td>2020-03-01</td>
      <td>4811</td>
    </tr>
    <tr>
      <th>41</th>
      <td>2020-04-01</td>
      <td>5123</td>
    </tr>
    <tr>
      <th>42</th>
      <td>2020-05-01</td>
      <td>4907</td>
    </tr>
    <tr>
      <th>43</th>
      <td>2020-06-01</td>
      <td>4069</td>
    </tr>
    <tr>
      <th>44</th>
      <td>2020-07-01</td>
      <td>3816</td>
    </tr>
    <tr>
      <th>45</th>
      <td>2020-08-01</td>
      <td>7252</td>
    </tr>
    <tr>
      <th>46</th>
      <td>2020-09-01</td>
      <td>5275</td>
    </tr>
    <tr>
      <th>47</th>
      <td>2020-10-01</td>
      <td>5320</td>
    </tr>
    <tr>
      <th>48</th>
      <td>2020-11-01</td>
      <td>5551</td>
    </tr>
    <tr>
      <th>49</th>
      <td>2020-12-01</td>
      <td>5749</td>
    </tr>
    <tr>
      <th>50</th>
      <td>2021-01-01</td>
      <td>5717</td>
    </tr>
    <tr>
      <th>51</th>
      <td>2021-02-01</td>
      <td>5660</td>
    </tr>
    <tr>
      <th>52</th>
      <td>2021-03-01</td>
      <td>6012</td>
    </tr>
    <tr>
      <th>53</th>
      <td>2021-04-01</td>
      <td>6113</td>
    </tr>
    <tr>
      <th>54</th>
      <td>2021-05-01</td>
      <td>6257</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Unnamed: 0 행 삭제
bmw = bmw.drop([bmw.index[0]])
```


```python
# Benz data 추출
benz = df.loc[1]
benz =benz.reset_index()
```


```python
df2 = df.stack()
```


```python
df2.reset_index()
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
      <th>level_0</th>
      <th>level_1</th>
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>Unnamed: 0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>브랜드</td>
      <td>bmw</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>2017-01-01</td>
      <td>2415</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>2017-02-01</td>
      <td>3202</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>2017-03-01</td>
      <td>6164</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>325</th>
      <td>5</td>
      <td>2021-01-01</td>
      <td>1198</td>
    </tr>
    <tr>
      <th>326</th>
      <td>5</td>
      <td>2021-02-01</td>
      <td>1202</td>
    </tr>
    <tr>
      <th>327</th>
      <td>5</td>
      <td>2021-03-01</td>
      <td>1251</td>
    </tr>
    <tr>
      <th>328</th>
      <td>5</td>
      <td>2021-04-01</td>
      <td>1263</td>
    </tr>
    <tr>
      <th>329</th>
      <td>5</td>
      <td>2021-05-01</td>
      <td>1264</td>
    </tr>
  </tbody>
</table>
<p>330 rows × 3 columns</p>
</div>



### BMW, Benz data 병합


```python
result = pd.merge(bmw, benz, on=['index'], how='outer')
result.head()
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
      <th>0</th>
      <th>1</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>브랜드</td>
      <td>bmw</td>
      <td>benz</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2017-01-01</td>
      <td>2415</td>
      <td>6848</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2017-02-01</td>
      <td>3202</td>
      <td>5534</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2017-03-01</td>
      <td>6164</td>
      <td>6737</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-04-01</td>
      <td>6334</td>
      <td>5758</td>
    </tr>
  </tbody>
</table>
</div>




```python
result.columns = ['year','BMW','Benz']
```


```python
result = result.drop(result.index[0], axis=0)
```


```python
result = result.drop(result.index[53], axis=0)
```


```python
result
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
      <th>BMW</th>
      <th>Benz</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>2017-01-01</td>
      <td>2415</td>
      <td>6848</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2017-02-01</td>
      <td>3202</td>
      <td>5534</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2017-03-01</td>
      <td>6164</td>
      <td>6737</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-04-01</td>
      <td>6334</td>
      <td>5758</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2017-05-01</td>
      <td>5373</td>
      <td>5063</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2017-06-01</td>
      <td>5510</td>
      <td>7783</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2017-07-01</td>
      <td>3188</td>
      <td>5471</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2017-08-01</td>
      <td>4105</td>
      <td>5267</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2017-09-01</td>
      <td>5299</td>
      <td>5606</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2017-10-01</td>
      <td>4400</td>
      <td>4539</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2017-11-01</td>
      <td>6827</td>
      <td>6296</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2017-12-01</td>
      <td>6807</td>
      <td>3959</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2018-01-01</td>
      <td>5407</td>
      <td>7509</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2018-02-01</td>
      <td>6118</td>
      <td>6192</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2018-03-01</td>
      <td>7052</td>
      <td>7932</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2018-04-01</td>
      <td>6573</td>
      <td>7349</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2018-05-01</td>
      <td>5222</td>
      <td>5839</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2018-06-01</td>
      <td>4196</td>
      <td>6248</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2018-07-01</td>
      <td>3959</td>
      <td>4715</td>
    </tr>
    <tr>
      <th>20</th>
      <td>2018-08-01</td>
      <td>2383</td>
      <td>3019</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2018-09-01</td>
      <td>2052</td>
      <td>1943</td>
    </tr>
    <tr>
      <th>22</th>
      <td>2018-10-01</td>
      <td>2131</td>
      <td>6371</td>
    </tr>
    <tr>
      <th>23</th>
      <td>2018-11-01</td>
      <td>2476</td>
      <td>7208</td>
    </tr>
    <tr>
      <th>24</th>
      <td>2018-12-01</td>
      <td>2955</td>
      <td>6473</td>
    </tr>
    <tr>
      <th>25</th>
      <td>2019-01-01</td>
      <td>2726</td>
      <td>5796</td>
    </tr>
    <tr>
      <th>26</th>
      <td>2019-02-01</td>
      <td>2340</td>
      <td>3611</td>
    </tr>
    <tr>
      <th>27</th>
      <td>2019-03-01</td>
      <td>2999</td>
      <td>4442</td>
    </tr>
    <tr>
      <th>28</th>
      <td>2019-04-01</td>
      <td>3226</td>
      <td>6543</td>
    </tr>
    <tr>
      <th>29</th>
      <td>2019-05-01</td>
      <td>3383</td>
      <td>6092</td>
    </tr>
    <tr>
      <th>30</th>
      <td>2019-06-01</td>
      <td>3292</td>
      <td>6632</td>
    </tr>
    <tr>
      <th>31</th>
      <td>2019-07-01</td>
      <td>3755</td>
      <td>7345</td>
    </tr>
    <tr>
      <th>32</th>
      <td>2019-08-01</td>
      <td>4291</td>
      <td>6740</td>
    </tr>
    <tr>
      <th>33</th>
      <td>2019-09-01</td>
      <td>4249</td>
      <td>7707</td>
    </tr>
    <tr>
      <th>34</th>
      <td>2019-10-01</td>
      <td>4122</td>
      <td>8025</td>
    </tr>
    <tr>
      <th>35</th>
      <td>2019-11-01</td>
      <td>4678</td>
      <td>6779</td>
    </tr>
    <tr>
      <th>36</th>
      <td>2019-12-01</td>
      <td>5130</td>
      <td>8421</td>
    </tr>
    <tr>
      <th>37</th>
      <td>2020-01-01</td>
      <td>2708</td>
      <td>5492</td>
    </tr>
    <tr>
      <th>38</th>
      <td>2020-02-01</td>
      <td>3812</td>
      <td>4815</td>
    </tr>
    <tr>
      <th>39</th>
      <td>2020-03-01</td>
      <td>4811</td>
      <td>5093</td>
    </tr>
    <tr>
      <th>40</th>
      <td>2020-04-01</td>
      <td>5123</td>
      <td>6745</td>
    </tr>
    <tr>
      <th>41</th>
      <td>2020-05-01</td>
      <td>4907</td>
      <td>6551</td>
    </tr>
    <tr>
      <th>42</th>
      <td>2020-06-01</td>
      <td>4069</td>
      <td>7672</td>
    </tr>
    <tr>
      <th>43</th>
      <td>2020-07-01</td>
      <td>3816</td>
      <td>5215</td>
    </tr>
    <tr>
      <th>44</th>
      <td>2020-08-01</td>
      <td>7252</td>
      <td>6030</td>
    </tr>
    <tr>
      <th>45</th>
      <td>2020-09-01</td>
      <td>5275</td>
      <td>5958</td>
    </tr>
    <tr>
      <th>46</th>
      <td>2020-10-01</td>
      <td>5320</td>
      <td>6576</td>
    </tr>
    <tr>
      <th>47</th>
      <td>2020-11-01</td>
      <td>5551</td>
      <td>7186</td>
    </tr>
    <tr>
      <th>48</th>
      <td>2020-12-01</td>
      <td>5749</td>
      <td>9546</td>
    </tr>
    <tr>
      <th>49</th>
      <td>2021-01-01</td>
      <td>5717</td>
      <td>5918</td>
    </tr>
    <tr>
      <th>50</th>
      <td>2021-02-01</td>
      <td>5660</td>
      <td>5707</td>
    </tr>
    <tr>
      <th>51</th>
      <td>2021-03-01</td>
      <td>6012</td>
      <td>7597</td>
    </tr>
    <tr>
      <th>52</th>
      <td>2021-04-01</td>
      <td>6113</td>
      <td>8430</td>
    </tr>
    <tr>
      <th>53</th>
      <td>2021-05-01</td>
      <td>6257</td>
      <td>7690</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 데이터타입 바꾸기
result = result.astype({'BMW':'int'})
result = result.astype({'Benz':'int'})
```


```python
# transacted_date, transacted_time 데이터 타입 string으로 인식하여 datetime64로 변환(column.dt.시간단위를 통해서 원하는 시간 부분을 추출해서 활용할 수 있음)
result['year'] = pd.to_datetime(result['year'])
```


```python
result.head()
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
      <th>BMW</th>
      <th>Benz</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>2017-01-01</td>
      <td>2415</td>
      <td>6848</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2017-02-01</td>
      <td>3202</td>
      <td>5534</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2017-03-01</td>
      <td>6164</td>
      <td>6737</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-04-01</td>
      <td>6334</td>
      <td>5758</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2017-05-01</td>
      <td>5373</td>
      <td>5063</td>
    </tr>
  </tbody>
</table>
</div>




```python
#인덱스를 연도로 재설정
result.index = result['year']
result.set_index('year', inplace=True)
result.head()
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
      <th>Benz</th>
    </tr>
    <tr>
      <th>year</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2017-01-01</th>
      <td>2415</td>
      <td>6848</td>
    </tr>
    <tr>
      <th>2017-02-01</th>
      <td>3202</td>
      <td>5534</td>
    </tr>
    <tr>
      <th>2017-03-01</th>
      <td>6164</td>
      <td>6737</td>
    </tr>
    <tr>
      <th>2017-04-01</th>
      <td>6334</td>
      <td>5758</td>
    </tr>
    <tr>
      <th>2017-05-01</th>
      <td>5373</td>
      <td>5063</td>
    </tr>
  </tbody>
</table>
</div>




```python
result.plot()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x25ccc4bdc50>




    
![png](output_18_1.png)
    



```python
result = result.reset_index()
result.head()
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
      <th>BMW</th>
      <th>Benz</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2017-01-01</td>
      <td>2415</td>
      <td>6848</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2017-02-01</td>
      <td>3202</td>
      <td>5534</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2017-03-01</td>
      <td>6164</td>
      <td>6737</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2017-04-01</td>
      <td>6334</td>
      <td>5758</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-05-01</td>
      <td>5373</td>
      <td>5063</td>
    </tr>
  </tbody>
</table>
</div>




```python
result = result.drop('index', axis=1)
```


```python
# BMW, Benz stationary 확인
timeseries = result
timeseries.rolling(12).mean().plot()#12개월평균
```




    <matplotlib.axes._subplots.AxesSubplot at 0x25ccc5d9cf8>




    
![png](output_21_1.png)
    


## Set up and plot the training data


```python
# Benz data만 따로 
result_Benz = result['Benz']
result_Benz = result_Benz.reset_index()
```


```python
len(result_Benz)
```




    53




```python
result_Benz.head()
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
      <th>Benz</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>6848</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>5534</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>6737</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>5758</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>5063</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Benz 인덱스 year 재설정
result_Benz.index = result_Benz['year']
result_Benz.set_index('year', inplace=True)

```


```python
result_Benz
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
      <th>Benz</th>
      <th>Benz_forecast</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>6848</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>5534</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>6737</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>5758</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>5063</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>5</td>
      <td>7783</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6</th>
      <td>6</td>
      <td>5471</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7</th>
      <td>7</td>
      <td>5267</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td>8</td>
      <td>5606</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9</th>
      <td>9</td>
      <td>4539</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>10</th>
      <td>10</td>
      <td>6296</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>11</th>
      <td>11</td>
      <td>3959</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>12</th>
      <td>12</td>
      <td>7509</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>13</th>
      <td>13</td>
      <td>6192</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>14</td>
      <td>7932</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>15</th>
      <td>15</td>
      <td>7349</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>16</th>
      <td>16</td>
      <td>5839</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>17</th>
      <td>17</td>
      <td>6248</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>18</th>
      <td>18</td>
      <td>4715</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>19</th>
      <td>19</td>
      <td>3019</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>20</th>
      <td>20</td>
      <td>1943</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>21</th>
      <td>21</td>
      <td>6371</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>22</th>
      <td>22</td>
      <td>7208</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>23</th>
      <td>23</td>
      <td>6473</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>24</th>
      <td>24</td>
      <td>5796</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>25</th>
      <td>25</td>
      <td>3611</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>26</th>
      <td>26</td>
      <td>4442</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>27</th>
      <td>27</td>
      <td>6543</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>28</th>
      <td>28</td>
      <td>6092</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>29</th>
      <td>29</td>
      <td>6632</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>30</th>
      <td>30</td>
      <td>7345</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>31</th>
      <td>31</td>
      <td>6740</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>32</th>
      <td>32</td>
      <td>7707</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>33</th>
      <td>33</td>
      <td>8025</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>34</th>
      <td>34</td>
      <td>6779</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>35</th>
      <td>35</td>
      <td>8421</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>36</th>
      <td>36</td>
      <td>5492</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>37</th>
      <td>37</td>
      <td>4815</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>38</th>
      <td>38</td>
      <td>5093</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>39</th>
      <td>39</td>
      <td>6745</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>40</th>
      <td>40</td>
      <td>6551</td>
      <td>-6150.503392</td>
    </tr>
    <tr>
      <th>41</th>
      <td>41</td>
      <td>7672</td>
      <td>-10363.503392</td>
    </tr>
    <tr>
      <th>42</th>
      <td>42</td>
      <td>5215</td>
      <td>-10338.503392</td>
    </tr>
    <tr>
      <th>43</th>
      <td>43</td>
      <td>6030</td>
      <td>-15018.503392</td>
    </tr>
    <tr>
      <th>44</th>
      <td>44</td>
      <td>5958</td>
      <td>-18924.503392</td>
    </tr>
    <tr>
      <th>45</th>
      <td>45</td>
      <td>6576</td>
      <td>-21592.004192</td>
    </tr>
    <tr>
      <th>46</th>
      <td>46</td>
      <td>7186</td>
      <td>-17199.502874</td>
    </tr>
    <tr>
      <th>47</th>
      <td>47</td>
      <td>9546</td>
      <td>-23042.004627</td>
    </tr>
    <tr>
      <th>48</th>
      <td>48</td>
      <td>5918</td>
      <td>-17591.006784</td>
    </tr>
    <tr>
      <th>49</th>
      <td>49</td>
      <td>5707</td>
      <td>-18917.006784</td>
    </tr>
    <tr>
      <th>50</th>
      <td>50</td>
      <td>7597</td>
      <td>-15566.006784</td>
    </tr>
    <tr>
      <th>51</th>
      <td>51</td>
      <td>8430</td>
      <td>-14961.006784</td>
    </tr>
    <tr>
      <th>52</th>
      <td>52</td>
      <td>7690</td>
      <td>-18916.006784</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Bmw data만 따로 
result_BMW = result['BMW']
result_BMW = result_BMW.reset_index()
```


```python
result_BMW
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
      <th>BMW</th>
      <th>forecast</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>2415</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>3202</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>6164</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>6334</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>5373</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>5</td>
      <td>5510</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6</th>
      <td>6</td>
      <td>3188</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7</th>
      <td>7</td>
      <td>4105</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td>8</td>
      <td>5299</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9</th>
      <td>9</td>
      <td>4400</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>10</th>
      <td>10</td>
      <td>6827</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>11</th>
      <td>11</td>
      <td>6807</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>12</th>
      <td>12</td>
      <td>5407</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>13</th>
      <td>13</td>
      <td>6118</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>14</td>
      <td>7052</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>15</th>
      <td>15</td>
      <td>6573</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>16</th>
      <td>16</td>
      <td>5222</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>17</th>
      <td>17</td>
      <td>4196</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>18</th>
      <td>18</td>
      <td>3959</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>19</th>
      <td>19</td>
      <td>2383</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>20</th>
      <td>20</td>
      <td>2052</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>21</th>
      <td>21</td>
      <td>2131</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>22</th>
      <td>22</td>
      <td>2476</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>23</th>
      <td>23</td>
      <td>2955</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>24</th>
      <td>24</td>
      <td>2726</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>25</th>
      <td>25</td>
      <td>2340</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>26</th>
      <td>26</td>
      <td>2999</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>27</th>
      <td>27</td>
      <td>3226</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>28</th>
      <td>28</td>
      <td>3383</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>29</th>
      <td>29</td>
      <td>3292</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>30</th>
      <td>30</td>
      <td>3755</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>31</th>
      <td>31</td>
      <td>4291</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>32</th>
      <td>32</td>
      <td>4249</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>33</th>
      <td>33</td>
      <td>4122</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>34</th>
      <td>34</td>
      <td>4678</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>35</th>
      <td>35</td>
      <td>5130</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>36</th>
      <td>36</td>
      <td>2708</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>37</th>
      <td>37</td>
      <td>3812</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>38</th>
      <td>38</td>
      <td>4811</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>39</th>
      <td>39</td>
      <td>5123</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>40</th>
      <td>40</td>
      <td>4907</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>41</th>
      <td>41</td>
      <td>4069</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>42</th>
      <td>42</td>
      <td>3816</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>43</th>
      <td>43</td>
      <td>7252</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>44</th>
      <td>44</td>
      <td>5275</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>45</th>
      <td>45</td>
      <td>5320</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>46</th>
      <td>46</td>
      <td>5551</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>47</th>
      <td>47</td>
      <td>5749</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>48</th>
      <td>48</td>
      <td>5717</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>49</th>
      <td>49</td>
      <td>5660</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>50</th>
      <td>50</td>
      <td>6012</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>51</th>
      <td>51</td>
      <td>6113</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>52</th>
      <td>52</td>
      <td>6257</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
result_BMW = result[['year', 'BMW']]
```


```python
# BMW 인덱스 year 재설정
result_BMW.index = result_BMW['year']
result_BMW.set_index('year', inplace=True)
result_BMW.head()
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
    </tr>
    <tr>
      <th>year</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2017-01-01</th>
      <td>2415</td>
    </tr>
    <tr>
      <th>2017-02-01</th>
      <td>3202</td>
    </tr>
    <tr>
      <th>2017-03-01</th>
      <td>6164</td>
    </tr>
    <tr>
      <th>2017-04-01</th>
      <td>6334</td>
    </tr>
    <tr>
      <th>2017-05-01</th>
      <td>5373</td>
    </tr>
  </tbody>
</table>
</div>




```python
result_Benz = result[['year', 'Benz']]
```


```python
result_Benz.index = result_Benz['year']
result_Benz.set_index('year', inplace=True)
result_Benz.head()
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
      <th>Benz</th>
    </tr>
    <tr>
      <th>year</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2017-01-01</th>
      <td>6848</td>
    </tr>
    <tr>
      <th>2017-02-01</th>
      <td>5534</td>
    </tr>
    <tr>
      <th>2017-03-01</th>
      <td>6737</td>
    </tr>
    <tr>
      <th>2017-04-01</th>
      <td>5758</td>
    </tr>
    <tr>
      <th>2017-05-01</th>
      <td>5063</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Benz train data
Benz_train = result_Benz.loc[:'2020-05-01']
Benz_test = result_Benz.loc['2020-05-01':]
Benz_train.plot(figsize = (15,6))
```




    <matplotlib.axes._subplots.AxesSubplot at 0x25cd2cb2ef0>




    
![png](output_34_1.png)
    



```python
# BMW
BMW_train = result_BMW.loc[:'2020-05-01']
BMW_test = result_BMW.loc['2020-05-01':]
BMW_train.plot(figsize = (15,6))
# train data
```




    <matplotlib.axes._subplots.AxesSubplot at 0x25ccc73e128>




    
![png](output_35_1.png)
    



```python
decomposition = seasonal_decompose(result_BMW['BMW'], model='additive', extrapolate_trend='freq')
# ValueError: You must specify a period or x must be a pandas object with a DatetimeIndex with a freq not set to None
# index를 year로 재설정
fig = plt.figure()
fig = decomposition.plot()
fig.set_size_inches(15,7)
```


    <Figure size 432x288 with 0 Axes>



    
![png](output_36_1.png)
    



```python
decomposition2 = seasonal_decompose(result_Benz['Benz'], model='additive', extrapolate_trend='freq')
fig = plt.figure()
fig = decomposition2.plot()
fig.set_size_inches(15,7)
```


    <Figure size 432x288 with 0 Axes>



    
![png](output_37_1.png)
    


## stationary 검증 방법 


```python
# bmw
result2 = adfuller(result_BMW['BMW'])
result2
```




    (-3.0147427042140826,
     0.03354036669069423,
     0,
     52,
     {'1%': -3.562878534649522,
      '5%': -2.918973284023669,
      '10%': -2.597393446745562},
     676.2432252758988)




```python
#p-vslue 값이 0.03으로 정산성 확인
# stationary check 기능 함수
def adf_check(ts):
    result = adfuller(ts)
    if result[1] <=0.05:
        print('Stationary{}'.format(result[1]))
    else:
        print('Non-stationary {}'.format(result[1]))
```


```python
adf_check(result_BMW['BMW'])#정상성 획득
```

    Stationary0.03354036669069423
    


```python
result3 = adfuller(Benz_train['Benz'])
result3
#정상성 X
```




    (-2.5220364766930845,
     0.11020425850865029,
     8,
     12,
     {'1%': -4.137829282407408,
      '5%': -3.1549724074074077,
      '10%': -2.7144769444444443},
     196.30079301504284)



## differencing ##


```python
Benz_train['1st diff'] = Benz_train['Benz'] - Benz_train['Benz'].shift(1)
Benz_train.head()
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
      <th>Benz</th>
      <th>1st diff</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>6848</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>5534</td>
      <td>-1314.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>6737</td>
      <td>1203.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>5758</td>
      <td>-979.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>5063</td>
      <td>-695.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Nan 삭제
Benz_train['1st diff'].dropna(axis=0, how='any',inplace=False)
```




    1    -1314.0
    2     1203.0
    3     -979.0
    4     -695.0
    5     2720.0
    6    -2312.0
    7     -204.0
    8      339.0
    9    -1067.0
    10    1757.0
    11   -2337.0
    12    3550.0
    13   -1317.0
    14    1740.0
    15    -583.0
    16   -1510.0
    17     409.0
    18   -1533.0
    19   -1696.0
    20   -1076.0
    Name: 1st diff, dtype: float64




```python
# Nan 평균 대체
Benz_train['1st diff'].fillna(Benz_train['1st diff'].mean())
```




    0     -245.25
    1    -1314.00
    2     1203.00
    3     -979.00
    4     -695.00
    5     2720.00
    6    -2312.00
    7     -204.00
    8      339.00
    9    -1067.00
    10    1757.00
    11   -2337.00
    12    3550.00
    13   -1317.00
    14    1740.00
    15    -583.00
    16   -1510.00
    17     409.00
    18   -1533.00
    19   -1696.00
    20   -1076.00
    Name: 1st diff, dtype: float64




```python
Benz_train.info()
# Nan 평균 대체
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 21 entries, 0 to 20
    Data columns (total 3 columns):
     #   Column    Non-Null Count  Dtype  
    ---  ------    --------------  -----  
     0   index     21 non-null     int64  
     1   Benz      21 non-null     int32  
     2   1st diff  20 non-null     float64
    dtypes: float64(1), int32(1), int64(1)
    memory usage: 548.0 bytes
    


```python
adf_check(Benz_train['1st diff'].dropna())
```

    Stationary2.7731412970864258e-05
    


```python
Benz_train['1st diff'].plot()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x25ccc4089b0>




    
![png](output_49_1.png)
    



```python
# d값은 1차 differencing이므로 1를 씀
# d = 1, D = 1
```

## differencing한 데이터로 acf, pacf 그리기

- acf란(자기상관함수, AutoCorrelation Function)
- 시차에 따른 일련의 자기 상관 의미, 시차가 커질수록 acf는 0에 가까워짐



```python
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf
```


```python
plot_acf(Benz_train['1st diff'].dropna());
```


    
![png](output_53_0.png)
    



```python
plot_pacf(Benz_train['1st diff'].dropna(), method='ywm')#partial AutoCorrelation
```


```python
# acf에서 음수 1개씩 나왔으므로 p, q값은 각 P=1. Q=1씀
```

## 하이퍼 파라미터 정하기


```python
# auto_arima 함수로 자동 추출
# Benz
from pmdarima import auto_arima
stepwise_model = auto_arima(Benz_train['Benz'], start_p=1, start_q=1,
                           max_p=3, max_q=3, m=12,
                           start_P=0, seasonal=True,
                           d=1, D=1, trace=True,
                           error_action='ignore',  
                           suppress_warnings=True, 
                           stepwise=True)
```

    Performing stepwise search to minimize aic
     ARIMA(1,1,1)(0,1,1)[12]             : AIC=inf, Time=0.31 sec
     ARIMA(0,1,0)(0,1,0)[12]             : AIC=137.980, Time=0.01 sec
     ARIMA(1,1,0)(1,1,0)[12]             : AIC=inf, Time=0.14 sec
     ARIMA(0,1,1)(0,1,1)[12]             : AIC=inf, Time=0.20 sec
     ARIMA(0,1,0)(1,1,0)[12]             : AIC=137.357, Time=0.08 sec
     ARIMA(0,1,0)(2,1,0)[12]             : AIC=139.357, Time=0.18 sec
     ARIMA(0,1,0)(1,1,1)[12]             : AIC=139.357, Time=0.14 sec
     ARIMA(0,1,0)(0,1,1)[12]             : AIC=inf, Time=0.08 sec
     ARIMA(0,1,0)(2,1,1)[12]             : AIC=141.357, Time=0.22 sec
     ARIMA(0,1,1)(1,1,0)[12]             : AIC=138.099, Time=0.13 sec
     ARIMA(1,1,1)(1,1,0)[12]             : AIC=140.759, Time=0.20 sec
     ARIMA(0,1,0)(1,1,0)[12] intercept   : AIC=inf, Time=0.18 sec
    
    Best model:  ARIMA(0,1,0)(1,1,0)[12]          
    Total fit time: 1.882 seconds
    

# ARIMA 학습 및 검증

## Benz 자동차 수 판매 예측


```python
# 오류시 train data float64로 바꾸기
Benz_train  = Benz_train.astype({'Benz':'float64'})
# How to fix numpy TypeError: Cannot cast ufunc subtract output from dtype(‘float64’) to dtype(‘int64’) with casting rule ‘same_kind’
```


```python
model = ARIMA(Benz_train['Benz'], order=(0,1,0))
# model = ARIMA(Benz_train, order=(0,1,1))
```


```python
bresult= model.fit(trend='c', full_output=True, disp=-1)
```


```python
bresult.summary()
```




<table class="simpletable">
<caption>ARIMA Model Results</caption>
<tr>
  <th>Dep. Variable:</th>      <td>D.Benz</td>      <th>  No. Observations:  </th>    <td>40</td>   
</tr>
<tr>
  <th>Model:</th>          <td>ARIMA(0, 1, 0)</td>  <th>  Log Likelihood     </th> <td>-352.073</td>
</tr>
<tr>
  <th>Method:</th>               <td>css</td>       <th>  S.D. of innovations</th> <td>1608.241</td>
</tr>
<tr>
  <th>Date:</th>          <td>Thu, 16 Dec 2021</td> <th>  AIC                </th>  <td>708.147</td>
</tr>
<tr>
  <th>Time:</th>              <td>17:22:23</td>     <th>  BIC                </th>  <td>711.525</td>
</tr>
<tr>
  <th>Sample:</th>           <td>02-01-2017</td>    <th>  HQIC               </th>  <td>709.368</td>
</tr>
<tr>
  <th></th>                 <td>- 05-01-2020</td>   <th>                     </th>     <td> </td>   
</tr>
</table>
<table class="simpletable">
<tr>
    <td></td>       <th>coef</th>     <th>std err</th>      <th>z</th>      <th>P>|z|</th>  <th>[0.025</th>    <th>0.975]</th>  
</tr>
<tr>
  <th>const</th> <td>   -7.4250</td> <td>  254.285</td> <td>   -0.029</td> <td> 0.977</td> <td> -505.815</td> <td>  490.965</td>
</tr>
</table>




```python
bresult
```




    <statsmodels.tsa.arima_model.ARIMAResultsWrapper at 0x25cd34ff0b8>




```python
bresult.resid.plot()
# 잔차도 그래프 
# 잔차 = 실제값-예측값 차이 (잔차도를 줄일수록 예측 good)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x25cd2dd9ac8>




    
![png](output_65_1.png)
    



```python
bresult.resid.plot(kind='kde')
```




    <matplotlib.axes._subplots.AxesSubplot at 0x25cc2798b70>




    
![png](output_66_1.png)
    


## Forecast


```python
# 훈련이 잘됨
bresult.plot_predict(dynamic=False)
plt.show()
```


    
![png](output_68_0.png)
    



```python
# Forecast
fc, se, conf  = bresult.forecast(steps=36, alpha=0.05)
```


```python
#기간별 월단위 자동 데이터 생성

dt_index = pd.date_range(start='20210601',end='20240601', freq='M')
print(dt_index)
dt_list = dt_index.strftime("%Y-%m-%d").tolist()
print(dt_list)
```

    DatetimeIndex(['2021-06-30', '2021-07-31', '2021-08-31', '2021-09-30',
                   '2021-10-31', '2021-11-30', '2021-12-31', '2022-01-31',
                   '2022-02-28', '2022-03-31', '2022-04-30', '2022-05-31',
                   '2022-06-30', '2022-07-31', '2022-08-31', '2022-09-30',
                   '2022-10-31', '2022-11-30', '2022-12-31', '2023-01-31',
                   '2023-02-28', '2023-03-31', '2023-04-30', '2023-05-31',
                   '2023-06-30', '2023-07-31', '2023-08-31', '2023-09-30',
                   '2023-10-31', '2023-11-30', '2023-12-31', '2024-01-31',
                   '2024-02-29', '2024-03-31', '2024-04-30', '2024-05-31'],
                  dtype='datetime64[ns]', freq='M')
    ['2021-06-30', '2021-07-31', '2021-08-31', '2021-09-30', '2021-10-31', '2021-11-30', '2021-12-31', '2022-01-31', '2022-02-28', '2022-03-31', '2022-04-30', '2022-05-31', '2022-06-30', '2022-07-31', '2022-08-31', '2022-09-30', '2022-10-31', '2022-11-30', '2022-12-31', '2023-01-31', '2023-02-28', '2023-03-31', '2023-04-30', '2023-05-31', '2023-06-30', '2023-07-31', '2023-08-31', '2023-09-30', '2023-10-31', '2023-11-30', '2023-12-31', '2024-01-31', '2024-02-29', '2024-03-31', '2024-04-30', '2024-05-31']
    


```python
len(dt_list)
```




    36




```python
import sys
mod = sys.modules[__name__]
```


```python
# 판다스 동적 반복 생성
for i in range(0,35):
    for dt in dt_list:
#     globals()'date{}'.format(i) = dt_list[i]
        setattr(mod, 'date{}'.format(i), dt)        
```


```python
date1 = datetime(2021,6,1)
date2 = datetime(2021,7,1)
date3 = datetime(2021,8,1)
date4 = datetime(2021,9,1)
date5 = datetime(2021,10,1)
date6 = datetime(2021,11,1)
date7 = datetime(2021,12,1)
date8 = datetime(2022,1,1)
date9 = datetime(2022,2,1)
date10 = datetime(2022,3,1)
date11 = datetime(2022,4,1)
date12 = datetime(2022,5,1)
date13 = datetime(2022,6,1)
date14 = datetime(2022,7,1)
date15 = datetime(2022,8,1)
date16 = datetime(2022,9,1)
date17 = datetime(2022,10,1)
date18 = datetime(2022,11,1)
date19 = datetime(2022,12,1)
date20 = datetime(2023,1,1)
date21 = datetime(2023,2,1)
date22 = datetime(2023,3,1)
date23 = datetime(2023,4,1)
date24 = datetime(2023,5,1)
date25 = datetime(2023,6,1)
date26 = datetime(2023,7,1)
date27 = datetime(2023,8,1)
date28 = datetime(2023,9,1)
date29 = datetime(2023,10,1)
date30 = datetime(2023,11,1)
date31 = datetime(2023,12,1)
date32 = datetime(2024,1,1)
date33 = datetime(2024,2,1)
date34 = datetime(2024,3,1)
date35 = datetime(2024,4,1)
date36 = datetime(2024,5,1)
```


```python
# Make as pandas series
fc_series = pd.Series(fc, index=[date1, date2, date3 ,date4 ,date5 ,date6 ,date7 ,date8 ,date9 ,date10 
,date11 ,date12 ,date13 ,date14 ,date15 ,date16 ,date17 ,date18 ,date19 ,date20 
,date21 ,date22 ,date23 ,date24 ,date25 ,date26 ,date27 ,date28 ,date29 ,date30 
,date31 ,date32 ,date33 ,date34 ,date35 ,date36])
# fc_series = pd.Series(fc, index=Benz_test.index)
fc_series
```




    2021-06-01    6543.575
    2021-07-01    6536.150
    2021-08-01    6528.725
    2021-09-01    6521.300
    2021-10-01    6513.875
    2021-11-01    6506.450
    2021-12-01    6499.025
    2022-01-01    6491.600
    2022-02-01    6484.175
    2022-03-01    6476.750
    2022-04-01    6469.325
    2022-05-01    6461.900
    2022-06-01    6454.475
    2022-07-01    6447.050
    2022-08-01    6439.625
    2022-09-01    6432.200
    2022-10-01    6424.775
    2022-11-01    6417.350
    2022-12-01    6409.925
    2023-01-01    6402.500
    2023-02-01    6395.075
    2023-03-01    6387.650
    2023-04-01    6380.225
    2023-05-01    6372.800
    2023-06-01    6365.375
    2023-07-01    6357.950
    2023-08-01    6350.525
    2023-09-01    6343.100
    2023-10-01    6335.675
    2023-11-01    6328.250
    2023-12-01    6320.825
    2024-01-01    6313.400
    2024-02-01    6305.975
    2024-03-01    6298.550
    2024-04-01    6291.125
    2024-05-01    6283.700
    dtype: float64




```python
# plot
plt.figure(figsize=(12,5), dpi=100)
plt.plot(Benz_train, label='training')
plt.plot(Benz_test, label='actual')
plt.plot(fc_series, label='forecast')

plt.title('Benz_sales_forecast')
plt.legend(loc='upper right', fontsize=8)
plt.show()
```


    
![png](output_76_0.png)
    


## BMW 자동차 수 판매 예측


```python
BMW_train  = BMW_train.astype({'BMW':'float64'})
```


```python
model = ARIMA(BMW_train['BMW'], order=(0,1,0))
# model = ARIMA(Benz_train, order=(0,1,1))
```


```python
BMW_result= model.fit(trend='c', full_output=True, disp=-1)
```


```python
BMW_result.summary()
```




<table class="simpletable">
<caption>ARIMA Model Results</caption>
<tr>
  <th>Dep. Variable:</th>       <td>D.BMW</td>      <th>  No. Observations:  </th>    <td>20</td>   
</tr>
<tr>
  <th>Model:</th>          <td>ARIMA(0, 1, 0)</td>  <th>  Log Likelihood     </th> <td>-171.784</td>
</tr>
<tr>
  <th>Method:</th>               <td>css</td>       <th>  S.D. of innovations</th> <td>1300.178</td>
</tr>
<tr>
  <th>Date:</th>          <td>Thu, 16 Dec 2021</td> <th>  AIC                </th>  <td>347.568</td>
</tr>
<tr>
  <th>Time:</th>              <td>17:22:36</td>     <th>  BIC                </th>  <td>349.559</td>
</tr>
<tr>
  <th>Sample:</th>                <td>1</td>        <th>  HQIC               </th>  <td>347.957</td>
</tr>
<tr>
  <th></th>                       <td> </td>        <th>                     </th>     <td> </td>   
</tr>
</table>
<table class="simpletable">
<tr>
    <td></td>       <th>coef</th>     <th>std err</th>      <th>z</th>      <th>P>|z|</th>  <th>[0.025</th>    <th>0.975]</th>  
</tr>
<tr>
  <th>const</th> <td>  -18.1500</td> <td>  290.729</td> <td>   -0.062</td> <td> 0.950</td> <td> -587.968</td> <td>  551.668</td>
</tr>
</table>




```python
BMW_train
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
      <th>BMW</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>2415.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>3202.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>6164.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>6334.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>5373.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>5</td>
      <td>5510.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>6</td>
      <td>3188.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>7</td>
      <td>4105.0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>8</td>
      <td>5299.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>9</td>
      <td>4400.0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>10</td>
      <td>6827.0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>11</td>
      <td>6807.0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>12</td>
      <td>5407.0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>13</td>
      <td>6118.0</td>
    </tr>
    <tr>
      <th>14</th>
      <td>14</td>
      <td>7052.0</td>
    </tr>
    <tr>
      <th>15</th>
      <td>15</td>
      <td>6573.0</td>
    </tr>
    <tr>
      <th>16</th>
      <td>16</td>
      <td>5222.0</td>
    </tr>
    <tr>
      <th>17</th>
      <td>17</td>
      <td>4196.0</td>
    </tr>
    <tr>
      <th>18</th>
      <td>18</td>
      <td>3959.0</td>
    </tr>
    <tr>
      <th>19</th>
      <td>19</td>
      <td>2383.0</td>
    </tr>
    <tr>
      <th>20</th>
      <td>20</td>
      <td>2052.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
BMW_test
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
      <th>BMW</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>21</th>
      <td>21</td>
      <td>2131</td>
    </tr>
    <tr>
      <th>22</th>
      <td>22</td>
      <td>2476</td>
    </tr>
    <tr>
      <th>23</th>
      <td>23</td>
      <td>2955</td>
    </tr>
    <tr>
      <th>24</th>
      <td>24</td>
      <td>2726</td>
    </tr>
    <tr>
      <th>25</th>
      <td>25</td>
      <td>2340</td>
    </tr>
    <tr>
      <th>26</th>
      <td>26</td>
      <td>2999</td>
    </tr>
    <tr>
      <th>27</th>
      <td>27</td>
      <td>3226</td>
    </tr>
    <tr>
      <th>28</th>
      <td>28</td>
      <td>3383</td>
    </tr>
    <tr>
      <th>29</th>
      <td>29</td>
      <td>3292</td>
    </tr>
    <tr>
      <th>30</th>
      <td>30</td>
      <td>3755</td>
    </tr>
    <tr>
      <th>31</th>
      <td>31</td>
      <td>4291</td>
    </tr>
    <tr>
      <th>32</th>
      <td>32</td>
      <td>4249</td>
    </tr>
    <tr>
      <th>33</th>
      <td>33</td>
      <td>4122</td>
    </tr>
    <tr>
      <th>34</th>
      <td>34</td>
      <td>4678</td>
    </tr>
    <tr>
      <th>35</th>
      <td>35</td>
      <td>5130</td>
    </tr>
    <tr>
      <th>36</th>
      <td>36</td>
      <td>2708</td>
    </tr>
    <tr>
      <th>37</th>
      <td>37</td>
      <td>3812</td>
    </tr>
    <tr>
      <th>38</th>
      <td>38</td>
      <td>4811</td>
    </tr>
    <tr>
      <th>39</th>
      <td>39</td>
      <td>5123</td>
    </tr>
    <tr>
      <th>40</th>
      <td>40</td>
      <td>4907</td>
    </tr>
    <tr>
      <th>41</th>
      <td>41</td>
      <td>4069</td>
    </tr>
    <tr>
      <th>42</th>
      <td>42</td>
      <td>3816</td>
    </tr>
    <tr>
      <th>43</th>
      <td>43</td>
      <td>7252</td>
    </tr>
    <tr>
      <th>44</th>
      <td>44</td>
      <td>5275</td>
    </tr>
    <tr>
      <th>45</th>
      <td>45</td>
      <td>5320</td>
    </tr>
    <tr>
      <th>46</th>
      <td>46</td>
      <td>5551</td>
    </tr>
    <tr>
      <th>47</th>
      <td>47</td>
      <td>5749</td>
    </tr>
    <tr>
      <th>48</th>
      <td>48</td>
      <td>5717</td>
    </tr>
    <tr>
      <th>49</th>
      <td>49</td>
      <td>5660</td>
    </tr>
    <tr>
      <th>50</th>
      <td>50</td>
      <td>6012</td>
    </tr>
    <tr>
      <th>51</th>
      <td>51</td>
      <td>6113</td>
    </tr>
    <tr>
      <th>52</th>
      <td>52</td>
      <td>6257</td>
    </tr>
  </tbody>
</table>
</div>




```python
from pandas.tseries.offsets import DateOffset
future_dates3=[result_BMW.index[-1]+ DateOffset(months=x)for x in range(0,14)]
future_datest_df3=pd.DataFrame(index=future_dates3[1:],columns=result_BMW.columns)

future_datest_df3.tail()
future_df3=pd.concat([result_BMW,future_datest_df3])
```

# SARIMAX 학습 및 검증

## Benz 자동차 수 판매 예측 


```python
import statsmodels.api as sm
model=sm.tsa.statespace.SARIMAX(Benz_train['Benz'],order=(0,1,0),seasonal_order=(0,2,1,12))
results=model.fit()
```


```python
results.summary()
```




<table class="simpletable">
<caption>SARIMAX Results</caption>
<tr>
  <th>Dep. Variable:</th>                 <td>Benz</td>               <th>  No. Observations:  </th>    <td>41</td>   
</tr>
<tr>
  <th>Model:</th>           <td>SARIMAX(0, 1, 0)x(0, 2, [1], 12)</td> <th>  Log Likelihood     </th> <td>-146.404</td>
</tr>
<tr>
  <th>Date:</th>                    <td>Thu, 16 Dec 2021</td>         <th>  AIC                </th>  <td>296.807</td>
</tr>
<tr>
  <th>Time:</th>                        <td>17:23:20</td>             <th>  BIC                </th>  <td>298.352</td>
</tr>
<tr>
  <th>Sample:</th>                     <td>01-01-2017</td>            <th>  HQIC               </th>  <td>296.886</td>
</tr>
<tr>
  <th></th>                           <td>- 05-01-2020</td>           <th>                     </th>     <td> </td>   
</tr>
<tr>
  <th>Covariance Type:</th>                <td>opg</td>               <th>                     </th>     <td> </td>   
</tr>
</table>
<table class="simpletable">
<tr>
      <td></td>        <th>coef</th>     <th>std err</th>      <th>z</th>      <th>P>|z|</th>  <th>[0.025</th>    <th>0.975]</th>  
</tr>
<tr>
  <th>ma.S.L12</th> <td>   -0.9867</td> <td>    0.313</td> <td>   -3.153</td> <td> 0.002</td> <td>   -1.600</td> <td>   -0.373</td>
</tr>
<tr>
  <th>sigma2</th>   <td> 3.953e+06</td> <td> 7.83e-08</td> <td> 5.05e+13</td> <td> 0.000</td> <td> 3.95e+06</td> <td> 3.95e+06</td>
</tr>
</table>
<table class="simpletable">
<tr>
  <th>Ljung-Box (L1) (Q):</th>     <td>0.11</td> <th>  Jarque-Bera (JB):  </th> <td>2.45</td> 
</tr>
<tr>
  <th>Prob(Q):</th>                <td>0.75</td> <th>  Prob(JB):          </th> <td>0.29</td> 
</tr>
<tr>
  <th>Heteroskedasticity (H):</th> <td>0.76</td> <th>  Skew:              </th> <td>-0.92</td>
</tr>
<tr>
  <th>Prob(H) (two-sided):</th>    <td>0.77</td> <th>  Kurtosis:          </th> <td>3.54</td> 
</tr>
</table><br/><br/>Warnings:<br/>[1] Covariance matrix calculated using the outer product of gradients (complex-step).<br/>[2] Covariance matrix is singular or near-singular, with condition number 9.59e+28. Standard errors may be unstable.




```python
result_Benz['Benz_forecast']=results.predict(start=40,end=53,dynamic=True)
result_Benz[['Benz','Benz_forecast']].plot(figsize=(12,8))
```




    <matplotlib.axes._subplots.AxesSubplot at 0x25cd13a8978>




    
![png](output_89_1.png)
    


## 2021년 5월 이후 date index 나타내기 

## AIC값 296


```python
from pandas.tseries.offsets import DateOffset
future_dates=[result_Benz.index[-1]+ DateOffset(months=x)for x in range(0,35)]
future_datest_df=pd.DataFrame(index=future_dates[1:],columns=result_Benz.columns)

future_datest_df.tail()

future_df=pd.concat([result_Benz,future_datest_df])
```


```python
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
      <th>Benz</th>
      <th>Benz_forecast</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2017-01-01</th>
      <td>6848</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-02-01</th>
      <td>5534</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-03-01</th>
      <td>6737</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-04-01</th>
      <td>5758</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2017-05-01</th>
      <td>5063</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2023-11-01</th>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2023-12-01</th>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2024-01-01</th>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2024-02-01</th>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2024-03-01</th>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>87 rows × 2 columns</p>
</div>




```python
future_df['Benz_forecast'] = results.predict(start = 53, end = 88, dynamic= True)
future_df[['Benz', 'Benz_forecast']].plot(figsize=(12, 8))
```




    <matplotlib.axes._subplots.AxesSubplot at 0x25cd3ce9588>




    
![png](output_94_1.png)
    


## BMW 자동차 수 판매 예측

# AIC값 279


```python
import statsmodels.api as sm
model2=sm.tsa.statespace.SARIMAX(BMW_train['BMW'],order=(0, 1, 0),seasonal_order=(1,2,0,12))
results2=model2.fit()
```


```python
results2.summary()
```




<table class="simpletable">
<caption>SARIMAX Results</caption>
<tr>
  <th>Dep. Variable:</th>                 <td>BMW</td>              <th>  No. Observations:  </th>  <td>21</td>  
</tr>
<tr>
  <th>Model:</th>           <td>SARIMAX(0, 1, 0)x(1, 2, 0, 12)</td> <th>  Log Likelihood     </th> <td>0.000</td>
</tr>
<tr>
  <th>Date:</th>                   <td>Thu, 16 Dec 2021</td>        <th>  AIC                </th> <td>4.000</td>
</tr>
<tr>
  <th>Time:</th>                       <td>17:23:30</td>            <th>  BIC                </th> <td>  nan</td>
</tr>
<tr>
  <th>Sample:</th>                         <td>0</td>               <th>  HQIC               </th> <td>  nan</td>
</tr>
<tr>
  <th></th>                              <td> - 21</td>             <th>                     </th>   <td> </td>  
</tr>
<tr>
  <th>Covariance Type:</th>               <td>opg</td>              <th>                     </th>   <td> </td>  
</tr>
</table>
<table class="simpletable">
<tr>
      <td></td>        <th>coef</th>     <th>std err</th>      <th>z</th>      <th>P>|z|</th>  <th>[0.025</th>    <th>0.975]</th>  
</tr>
<tr>
  <th>ar.S.L12</th> <td>         0</td> <td>      nan</td> <td>      nan</td> <td>   nan</td> <td>      nan</td> <td>      nan</td>
</tr>
<tr>
  <th>sigma2</th>   <td>       nan</td> <td>      nan</td> <td>      nan</td> <td>   nan</td> <td>      nan</td> <td>      nan</td>
</tr>
</table>
<table class="simpletable">
<tr>
  <th>Ljung-Box (L1) (Q):</th>     <td>nan</td> <th>  Jarque-Bera (JB):  </th> <td>nan</td>
</tr>
<tr>
  <th>Prob(Q):</th>                <td>nan</td> <th>  Prob(JB):          </th> <td>nan</td>
</tr>
<tr>
  <th>Heteroskedasticity (H):</th> <td>nan</td> <th>  Skew:              </th> <td>nan</td>
</tr>
<tr>
  <th>Prob(H) (two-sided):</th>    <td>nan</td> <th>  Kurtosis:          </th> <td>nan</td>
</tr>
</table><br/><br/>Warnings:<br/>[1] Covariance matrix calculated using the outer product of gradients (complex-step).<br/>[2] Covariance matrix is singular or near-singular, with condition number    nan. Standard errors may be unstable.




```python
result_BMW['forecast']=results2.predict(start=40,end=53,dynamic=True)
result_BMW[['BMW','forecast']].plot(figsize=(12,8))
```




    <matplotlib.axes._subplots.AxesSubplot at 0x25cd3f9b2e8>




    
![png](output_99_1.png)
    



```python
from pandas.tseries.offsets import DateOffset
future_dates2=[result_BMW.index[-1]+ DateOffset(months=x)for x in range(0,35)]
future_datest_df2=pd.DataFrame(index=future_dates2[1:],columns=result_BMW.columns)

future_datest_df2.tail()

future_df2=pd.concat([result_BMW,future_datest_df2])
```


```python
future_df2['BMW_forecast'] = results2.predict(start = 53, end = 88, dynamic= True)
future_df2[['BMW', 'BMW_forecast']].plot(figsize=(12, 8))
```




    <matplotlib.axes._subplots.AxesSubplot at 0x25cd13a83c8>




    
![png](output_101_1.png)
    


## Benzk, BMW는 SARIMAX 모델 더 적합

## 1) Benz, BMW 자동차 수 판매 예측 (Benz AIC값 296)


```python
fig = plt.figure(figsize=(15,12)) ## 캔버스 생성
ax = fig.add_subplot() # 그림 뼈대(프레임) 생성

# Benz
ax.plot(future_df.index,future_df['Benz'], label='Benz') ## 선그래프 생성
ax.plot(future_df.index,future_df['Benz_forecast'], label='Benz_forecast') 
# BMW
ax.plot(future_df2.index,future_df2['BMW'], label='BMW') ## 선그래프 생성
ax.plot(future_df2.index,future_df2['BMW_forecast'], label='BMW_forecast') 

plt.yticks(fontsize=13) #y축 눈금 크기 설정
plt.xticks(rotation=30,fontsize=13) # x축 눈금 회전각도와 크기 설정

ax.legend(loc='best',fontsize=13) # 범례 생성

plt.ylabel('일자',rotation=90, fontsize=15, color='black') # y축 라벨
plt.xlabel('자동차판매 수',fontsize=15, color='black') # x축 라벨
          
plt.title('21년 5월 이후 BMW VS Benz 판매 예측',fontsize=20) # 타이틀 설정
plt.show()

```


    
![png](output_104_0.png)
    

