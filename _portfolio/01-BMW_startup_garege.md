---
title: "BMW Startup Garage"
permalink: /portfolio/BMW Startup Garage/
date: 2021-11-26
categories:
    - BMW Project
    - BMW startup garage
tags:
    - BMW
    - pytorch
    - tesorflow
    - flask
    - mysql
    - vue.js
    - pandas
    - AWS RDS
    - AWS EC2
    - datagrip
toc: true
---

## BMW Garage Startup Program

![proposal]({{ site.url }}{{ site.baseurl }}/assets/images/bmwgs (20).png)

BMW Startup Garage는 개발, 생산, 서비스 등의 부문에서<br>
IT를 비롯한 4차 산업 기술과 비전을 보유한 스타트업을 발굴하여<br>
수십조 달러 규모의 자동차 산업과 함께하는 파트너사를 선정하는 프로그램입니다.<br>

2021년 10월 5일~11월 26일 동안 PoC 검증 기간을 거쳐 <br>
11월 26일 Demoday 코엑스 스타트업 브랜치 홀에서 각 스타트업이 발표한 후 각 부문에서 최종 파트너사를 선정하게 됩니다.

---
- 프로젝트 명 :  
  - BMW 콜센터 상담 대시보드 NLU 대시보드 개발
  - BMW 7시리즈 재구매 고객 예측
- RNR : PM
- task : 데이터 분석, 엔지니어링, 데이터 I/O, ML 개발 
- 프로젝트 기간 : 2021-09 ~ 2021-11
- 총 참여 인원 : 8명
  - CTO
  - 백엔드 개발자
  - 프론트엔드 개발자
  - 데이터 분석가
  - 머신러닝 엔지니어
  - 퍼블리셔
  - 디자이너

---
데이터 연구소팀의 BMWgs 프로젝트 PM을 맡아 진행하였습니다.<br>
BMW 마케팅팀의 요청한 문제 정의 및 해결을 위한 솔루션 제안부터 
전반적인 커뮤니케이션과 운영관리, 데이분석 및 데이터엔지니어링 업무를 맡았습니다.<br>

그 결과 총 23개 기업이 신청한 Business Intelligence 분야에서<br>
파트너사로 단독 선정되는 성과를 이루었습니다.

---

### data status

1. CIC 상담 데이터
- 116,000 건

2. 구매고객 데이터
- 83,000 건

3. 리드 데이터
- 19YTD : 287,108 건
- 20YTD : 383,150 건
- 2106 YTD: 255,309 건
- 총 : 925,567 건

4. 드라이빙센터 이용 고객 데이터
- 54,750 건

5. 밴티지 데이터
- 1 -7 월 : 709,010건
- 10-12월 : 75,655 건
  총 : 784,665 건

6. AS 서비스 이용 고객 데이터 (20-21년 file 전달 받을 예정)
- 19년 1-6 월 : 477,883건
- 19년 7-12월 : 593,850건
  총 : 1,071,733 건

================================ 전체 데이터 총 : 3,035,715 건

### RFP


![rfp]({{ site.url }}{{ site.baseurl }}/assets/images/bmwgs (23).png)


**1.BMW 콜센터 상담 대시보드 NLU 대시보드 개발**

BMW Korea사는 상담 콜센터의 고객의 목소리가 버려지고 있어 실제 고객의 불만사항과
이슈 파악이 어렵습니다. 이에 고객 만족도를 높이고 서비스를 개선하여 비즈니스로 활용하고자
솔루션을 요청했습니다. 

< 제안 >
CIC 고객 상담 이력을 NLU 분석한 대시보드를 구현합니다. 대시보드는 고객의 주요 이슈를 한눈에 파악하도록 시각화합니다. 주요 키워드 도출을 통해 고객 불만사항과 니즈를 파악합니다. 의사결정 보조수단으로서 고객 문제를 빠르게 해결합니다. 서비스를 지속적으로 개선하여 고객 만족도를 높이는 데 기여합니다.
<br><br>
제공 기능:
1.  CIC 고객 상담 이력 대시보드 구현(Web)
2.  상담 주제 별 카테고리 분류 및 시각화
3.  차량모델 별 주요 키워드 랭킹, Wordcloud 시각화
4.  일자, 상담 주제, 모델 별 고객 상담 데이터 조회

**2.BMW 7시리즈 재구매 고객 예측**

구매 데이터 , 리드 데이터 , AS, 드라이빙 센터 데이터 등의 방대한
데이터를 보유하고 있지만 고객의 재구매를 딜러사와 Sales Consultant에게 의존하고 
있습니다. 

< 제안 >

BMW 7시리즈 재구매 예측 모델(ML algoruthm)을 통해 고객의 구매 패턴을 예측하고 적절한 타겟에게
마케팅을 적용하도록 제안했습니다. 고객 데이터 분석을 통해 재구매 요인을 도출합니다. 어떤 행동이력의 고객이 차량을 구매할 지 파악할 수 있습니다.

제공 기능:
1.   BMW 7series 재구매 예측 머신러닝 알고리즘 개발
2.   BMW 7series 재구매 요인 도출

### Retention Prediction
![retention1]({{ site.url }}{{ site.baseurl }}/assets/images/bmwgs (21).png)
![retention2]({{ site.url }}{{ site.baseurl }}/assets/images/bmwgs (22).png)


## BMW 콜센터 상담 대시보드 NLU 대시보드 개발

### 아키텍처

![archi1]({{ site.url }}{{ site.baseurl }}/assets/images/bmwgs (24).png)
![archi2]({{ site.url }}{{ site.baseurl }}/assets/images/bmwgs (18).png)
![archi3]({{ site.url }}{{ site.baseurl }}/assets/images/bmwgs (19).png)

### ERD

![erd]({{ site.url }}{{ site.baseurl }}/assets/images/erd.png)

### 데이터 전처리

![datapreprocessing2]({{ site.url }}{{ site.baseurl }}/assets/images/bmwgs (4).png)
![datapreprocessing3]({{ site.url }}{{ site.baseurl }}/assets/images/bmwgs (5).png)
![datapreprocessing4]({{ site.url }}{{ site.baseurl }}/assets/images/bmwgs (6).png)

### koBert모델 개발

![kobert1]({{ site.url }}{{ site.baseurl }}/assets/images/bmwgs (7).png)
![kobert2]({{ site.url }}{{ site.baseurl }}/assets/images/bmwgs (8).png)
![kobert3]({{ site.url }}{{ site.baseurl }}/assets/images/bmwgs (9).png)
![kobert4]({{ site.url }}{{ site.baseurl }}/assets/images/bmwgs (10).png)

bert 모델은 transformer 계열로 이 모델의 특징은 positional embedding을 고려합니다. <br>
단어의 의미를 파악할 때 단어로서 뿐만 아니라 그 단어가 문장내에서 어느 위치에 있는지, 
그리고 주변의 단어(조사 포함)는 어떤 단어가 있는지도 같이 파악하기 때문에 
발화자의 의도를 더 정확히 분류할 수 있습니다. <br>
그중에서도 KoBert 모델은 Skt에서 개발한 한국어 성능을 강화한 모델로 사용하였습니다.
약 3만건의 데이터를 학습하여 **accuracy 81% 모델**을 개발하였으며 
세부 카테고리 분류에 따른 정의서 데이터와 추가적인 상담데이터 학습을 통해 정확도 개선할 예정입니다.

### 디자인
![design]({{ site.url }}{{ site.baseurl }}/assets/images/bmwgs (15).png)
![design]({{ site.url }}{{ site.baseurl }}/assets/images/bmwgs (16).png)


## [대시보드 결과물 보러 가기](https://www.bmwboard.co.kr/#/sign-in)

---
## 진행하면서 어려웠던 점
<br>
**데이터 전달 시점이 늦어 시간이 부족했던 이슈**

진행하면서 가장 큰 어려웠던 점은 11월 26일 데모데이 발표일이었는데
3주 전에 데이터를 받아 시간이 부족했던 것입니다. 2개월 PoC 기간 중 실제 데이터를 분석할 시간이 3주밖에 되지 않았으며
분석을 통해 인사이트를 도출하는 것과 개발을 동시에 진행해야 하는 점, 
커뮤니케이션 비용으로 인해 분석과 개발에 투자할 시간이 부족한 게 
아쉬웠습니다. <br> 데이터를 늦게 받게 된 이유로는 외부의 아직 선정되지 않은 여러 기업들에게 NDA와 DPA 같은
협약을 체결 후에 데이터를 보내야 하는 고객 정보에 민감한 이슈가 내부에 있었던 것 같습니다.

다행히 고객에게서 시간이 부족한 것을 알고 STT 솔루션 단계를 생략한
상담로그이력 텍스트를 전달 줄 수 있었습니다. 사전에 데이터가 빨리 오지 않을 것을 대비하여 샘플 데이터를 받아 미리 데이터 분석 로직을 설계하고
코드를 짰습니다.
NLU 대시보드 개발 경우 PoC 기업 대상으로 선정되자마자 빠르게 진행하여 데이터를 받은 시점에서
데이터를 가공하여 DB에 Insert를 하면 되는 상황이었습니다.


![7series_vis3]({{ site.url }}{{ site.baseurl }}/assets/images/bmwgs (14).png)

다만 7series 재구매 예측 주제 경우, 7시리즈 재구매 고객 수가 200건밖에 되지 않아
데이터 수가 부족하였고, 모델 개발까지는 시간상 진행이 어려워
EDA 분석까지만 실시할 수 있었습니다.
PoC 이후에 5년 이상 기간의 데이터를 확보하여 재구매 예측 모델을 이어서 개발하는 방안을 대안 제시하였습니다.

[comment]: <> (2&#41;형태소분석기에서 가공의 한계)
[comment]: <> (명사분석기로만 쓰면 유의미한 단어가 x, 경고등 -> 고등, 예약자명->자명, 유의미한 단어들 분석이 잘 안됨)


[comment]: <> (**업계 지식 이슈**)

[comment]: <> (BDC, eCOM, EoT고객, NSC캠페인, bsi , CIC, bcc, Poi 등 BMW 내부에서 상요하는)

[comment]: <> (업계용어로 이루어진 데이터라 분석에 어려움을 겪었습니다. )

[comment]: <> (그외 DSS계약일과 구매일, 인도일, 품의서상의 계약일, 차량 구매일 등 같은 구매 내용이라도)

[comment]: <> (모두 칼럼으로 세세히 분류되어 있어 어떤 칼럼을 중요하게 봐야할 지 업계 기반의 지식을 숙지해야 했습니다.)

[comment]: <> (다행히 수입자동차 업계 평균 20년 경력인)

[comment]: <> (임원진 출신인 분들이 회사에 계셔서 각 용어에 대한 주석을 모두 여쭤보고 분석하였습니다.)

[comment]: <> (**DB 설계 이슈**)

[comment]: <> (data를 넣는 table간의 관계도를 설정하고 data를 어떻게 insert할 지)

[comment]: <> (백엔드 API와 같이 만드는 부분이 어려웠습니다. data를 넣는 table 생성 및 ERD는)

[comment]: <> (백엔드 디벨로퍼와 소통하여 만들었으나 이후에 테이블간의 외래키 설정으로 인해)

[comment]: <> (데이터를 삭제하려니 참조 무결성 이슈가 있었습니다. 데이터를 delete 후 Insert할 때 )

[comment]: <> (연결된 테이블에 각각 넣어야 하는 번거로움이 있어 )

[comment]: <> (실제 서비스를 만들 때 외래키를 참조하는 것은 신중히 사용해야 한다는 것을 배웠습니다.)


[comment]: <> (**카테고리 라벨 분류 이슈**)

[comment]: <> (데이터 분석에서 어려웠던 점은 상담 카테고리 데이터 정의서가 없는 탓에 모든 상담 데이터를)

[comment]: <> (형태소 분석기와 TDM을 통해 빈도수 높은 단어를 도출하여 카테고리를 전부 세분화하여 )

[comment]: <> (라벨링을 해야 했습니다. 총 10만건의 데이터 중 일부 잘못 전화함, 끊김, Null 등의 불필요한 데이터를)

[comment]: <> (삭제한 8만 6천 건의 데이터를 라벨링하기에는 어려우므로 LDA와 Bert모델을 사용하여 카테고리 라벨링을 학습 분류하고자 했고)

[comment]: <> (이전에 같이 머신러닝 엔지니어 교육을 받은 친구를 파트타임으로 고용하여 함께 Bert모델을 개발하였습니다. )


**데이터 조회 속도 이슈**

또한 대시보드의 기간 별 키워드를 조회할 때 속도가 느리거나 일부 9천건까지
데이터는 조회되지만 8만 6천건의 데이터 3개월을 넣었을 때 
아예 나오지 않는 이슈가 있었습니다. AWS 서버용량이 t3.small로 9천건의 데이터만 넣어도
메모리가 85% 꽉차 있는 있었습니다. <br> 이후 t3.medium으로 업그레이드하자 8만 6천건의
데이터가 모두 불러와졌으나 30초가 되어서야 화면이 돌아가는 문제가 여전했습니다.<br>
이에 CTO와 커뮤니케이션을 하여 데이터 조회 쿼리를 확인하였고 전체 8만 6천건의 데이터를 다 돌면서 기간별로 
데이터를 row별로 한줄씩 조회하는 로직으로 CPU 연산을 많이 차지하는 걸 알게되었습니다.<br>
CTO는 CPU 연산을 줄이기 위해 카테고리와 갯수를 1:1 맵핑하는 로직으로 수정하였습니다.
이에 8만 6천 건 데이터 조회 쿼리가 실행되는 속도가 30초에서 1초로 훨씬 빨라졌습니다.

[comment]: <> (디자인-> 퍼블리싱-> 웹 구현 100% 안되는 문제)
[comment]: <> (디자인으로 보았을 때는 이쁜데 데이터를 넣으면 가독성이 떨어지고 &#40;pie차트&#41;,)
[comment]: <> (차트가 안예쁘게 나오는 문제가 이뜸)
[comment]: <> (고객 민감 데이터 보안 이슈)
