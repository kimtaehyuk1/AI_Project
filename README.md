# 모듈프로젝트2 ReadME(회귀)
-------------------------------------------------------------
# 🙌<u>목적</u>
- 주어진 데이터로 캘리포니아 집값 예측(회귀문제)

# ❓<u>데이터 수집</u>
- 캐글 경쟁대회 놀이터2 데이터
- fetch_california_housing() 데이터

# 🙋‍♀️<u>데이터 분석 결과</u>

|히스토그램|회귀|
|:-:|:-:|
|<img src="https://user-images.githubusercontent.com/67897827/219931885-dbdea2bc-16ea-41b5-9c44-af6bdb86a729.PNG" width="700" height="300"/>|<img src="https://user-images.githubusercontent.com/67897827/219931884-0a388a77-2af8-4ab2-8dd5-f459150c6866.PNG" width="700" height="300"/>| 

|상관관계|위,경도|
|:-:|:-:|
|<img src="https://user-images.githubusercontent.com/67897827/219931886-6325f25a-7836-4b0b-8e19-aaf6faf78906.PNG" width="700" height="300"/>|<img src="https://user-images.githubusercontent.com/67897827/219931887-aff4d8b7-aee6-4ec4-9ba7-1c72d52983d2.PNG" width="700" height="300"/>| 

1. 이상치 제거 항목
  - MedInc, AveRooms, AveBedrms, Population, AveOccup
2. 차원 정리
  - Latitude, Longitude 
3. 강한 상관관계
  - AveRooms & MedInc 간의 강한 양의 상관관계
    - 하나만 사용
    - 둘다 사용
  - Latitude & Longitude 간의 강한 음의 상관관계
    - 하나만 사용
    - 둘다 사용

# 👆<u>피쳐 엔지니어링</u>
## 이상치 제거
  1. 이상치 제거할 피처를 시각화
  2. 시각화 결과를 바탕으로, 이상치 제거 대상 피처마다 범위를 지정한 후 IQR(분위수) 방식을 이용하여 이상치 제거.

|이상치제거 히스토그램|이상치제거 회귀|
|:-:|:-:|
|<img src="https://user-images.githubusercontent.com/67897827/219931878-b1047385-739b-4f0e-8f73-09855d1cbc06.PNG" width="700" height="300"/>|<img src="https://user-images.githubusercontent.com/67897827/219931880-84036f70-0dad-4fb1-82dd-98d001f3c347.PNG" width="700" height="300"/>| 

## 상관관계 상쇄
- MedInc & AveRooms 
- Latitude & Longitude

|df|MedInc|AveRooms|Latitude|Longitude|
|--|--|--|--|--|
|tmp1|o|x|o|x|
|tmp2|o|x|x|o|
|tmp3|x|o|o|x|
|tmp4|x|o|x|o|
|tmp_D_1|o|x|o|o|
|tmp_D_2|x|o|o|o|

- tmp 1 ~ 4 : 차원 축소 할 필요 없음
- tmp_D_1 ~ 2 : 차원축소 필요

## 위경도를 이용하여 새로운 피쳐 생성
  - 방법1: 지역 분할
  - 방법2: K-Means
  - 방법3: 차원축소(PCA, PCA을 군집화, UMAP, UMAP을 군집화
  - 방법4: 해안가와의 거리
  - 방법5: 주요 도시와의 거리
  - 방법6: 위도 경도 회전
  - 방법7: 지역들의 메타 데이터 이용(주별 인구밀도, 미국과 멕시코의 실질 부동산 가격지수)
  - 방법8: 연속형 변수의 범위를 정해서 범주화

|PCA 군집화|UMAP 군집화|
|:-:|:-:|
|<img src="https://user-images.githubusercontent.com/67897827/219931882-e552afa0-4814-4f5e-aa4c-db4121735297.PNG" width="700" height="300"/>|<img src="https://user-images.githubusercontent.com/67897827/219931883-1a129ff5-296f-4669-af63-ef9ef9ce0bb7.PNG" width="700" height="300"/>| 

    
# 🛠<u>베이스라인 모델 구축 및 기본학습</u>
- 학습 시키기 전, 스케일링 함수와 점수처리 함수 생성
- 어떤 피쳐가 의미가 있는지 기본모델로 검증
## 모델 학습
1. LinearRegression
2. Lasso
3. RandomForestRegressor
## 기본모델 학습 결과, 성능 저하시키는 피쳐 정리
1. 연속형 변수의 범위를 정해서 범주화 시킨 피처.
2. 상관관계가 높은 피처를 상쇄한 피처.
3. 주별 인구밀도 피처.
4. 미국과 멕시코의 실질 부동산 가격지수 피처.

- 위의 파생피처를 생성해서 돌렸을때, 성능저하가 되었으므로,위의 피처를 제거하여서 최종 피처를 생성하였다.

# 🙋‍♂️<u>모델 성능 향상</u>
## 피처제거 방법
- 위에서 생성한 최종 피처에서, 상대적으로 피처 중요도가 낮거나, 경험적으로 학습에 방해될 만한 피처 제거.
### 제거 피처 선정
1. pca_cluster제거
2. umap_cluster제거
3. cluster_3제거
4. cluster_20제거

- 위의 항목에 있는 피처를 각각제거하여, 성능평가
- 성능평가 결과가 의미있는 예측값 간의 블렌딩

## 최적 모델 선정
- 모델은 varstack 이라는 lgbm 튜너를 사용
- https://github.com/conversis/varstack
 
# 🔎<u>결론</u>
- umap_cluster피처 제거와, cluster_3피처를 제거하여 lgbm튜너를 돌려서 나온 각각의 예측값을 블렌딩하였다.
- 최종값이 '0.55546'으로 실험 중 가장 높은 결과 값이 나왔다.

<img src="https://user-images.githubusercontent.com/67897827/219930166-6ef32a7d-6a6f-4ad9-9ee5-bf1924bf0e3c.PNG" width="700" height="300"/>

-----------------------------------------------------------------------------------------------------

# 모듈프로젝트2 ReadME(분류)

# 1.개요
- kaggle Playground Series - Season 3, Episode 2, Tabular Classification with a Stroke Prediction Dataset
- https://www.kaggle.com/competitions/playground-series-s3e2


# 2. 데이터 준비 + EDA 분석
## 2.1 정답비율
<table>
      <tr>
            <td style="text-align:center">정답비율</td>
      </tr>
      <tr>
            <td><img width="300" , height="250", alt="image" src="https://user-images.githubusercontent.com/50509845/220023239-99043806-0057-4a12-a6e2-f73d024e1dda.png"></td>
      </tr>
      <tr>
            <td>-	정답 비율의 차이가 크므로, stratifiedkfold 고려
            </td>
      </tr>
</table>

## 2.2 이진형	
- Residence_type를 제외하고 모든 피처들이 구분이 명확하다.
- Residence_type은 명확히 구분이 안가므로 제거도 고려 해볼만하다.
- gender에서 Other 이상치가 존재한다. 최빈값으로 치환하겠다.

<table>
      <tr>
            <td style="text-align:center", colspan="2">이진형 피쳐의 정답 비율</td>
      </tr>
      <tr>
            <td><img width="196" alt="image" src="https://user-images.githubusercontent.com/50509845/220026606-bcfaca7a-bd5b-4eb1-ae1e-88795dec006b.png"></td>
            <td><img width="188" alt="image" src="https://user-images.githubusercontent.com/50509845/220026737-6e6f0831-49b1-4a79-8350-782c77e48538.png"></td>
      </tr>
      <tr>
            <td><img width="190" alt="image" src="https://user-images.githubusercontent.com/50509845/220026831-4f314993-80ea-4ccb-abbc-a0a2ca96800c.png"></td>
            <td><img width="191" alt="image" src="https://user-images.githubusercontent.com/50509845/220026880-3954a0ca-01ab-42e7-a288-40b38633259a.png"></td>
      </tr>
      <tr>
            <td colspan="2"><img width="197" alt="image" src="https://user-images.githubusercontent.com/50509845/220026937-729df07d-3699-4fe7-92d5-85fa437a7e58.png"></td>
      </tr>
</table>

## 2.3 명목형 데이터
- 타겟비율이 비슷한 도메인이 있어서 이것들을 하나로 묶어보는 방법 사용

<table>
      <tr>
            <td style="text-align:center">명목형 데이터 개요</td>
      </tr>
      <tr>
            <td><img width="452" alt="image" src="https://user-images.githubusercontent.com/50509845/220028007-27af9cbd-dd93-48bf-aecc-df34459fce99.png"></td>
      </tr>
</table>

### 2.3.1 밀도그래프
- 'avg_glucose_level, 'bmi', 'age'는 연속형수치형과 범주형-순서형으로 처리하여 시각화 분석 및 학습 진행
- 바로 스케일링하여 학습했을 때와 결과 차이 비교 => 제출 결과 연속형 수치  데이터는 순서형으로 처리하지 않고 그대로 스케일링하여 학습시켰을 때 높은 성능 결과가 나왔다.

<table>
      <tr>
            <td style="text-align:center", colspan="2">밀도그래프(히스토그램)</td>
      </tr>
      <tr>
            <td><img width="195" alt="image" src="https://user-images.githubusercontent.com/50509845/220029083-01770a64-294f-40e8-9a3b-48775da6cee9.png"></td>
            <td><img width="190" alt="image" src="https://user-images.githubusercontent.com/50509845/220029145-d224d7b5-ae9e-4f80-af7d-39f2d4b1d2f0.png"></td>
      </tr>
      <tr>
            <td colspan="2"><img width="189" alt="image" src="https://user-images.githubusercontent.com/50509845/220029223-d14c0aa9-9211-45b2-8f03-97aebfe5c21c.png"></td>
      </tr>
</table>

### 2.3.2 비율 그래프
- 'avg_glucose_level, 'bmi', 'age' 데이터를 범주형-순서형으로 처리하여 데이터 EDA 2차 분석
- 당뇨병이 없는 혈당수치가 정상인 경우와 당뇨병전단계에서는 뇌졸중 타겟값 비율이 낮게 나온다
- 당뇨병 환자인 경우 뇌졸중 타겟값 비율이 매우 높게 나타났다 => 당뇨병과 뇌졸중 간의 관계가 있음을 생각할 수 있다
- bmi지수가 높을수로 뇌졸중 타겟값 비율이 높게 나타났다
- 나이가 고령일수록 뇌졸중 타겟값 비율이 높게 나타났다

<table>
      <tr>
            <td style="text-align:center", colspan="2">비율 그래프</td>
      </tr>
      <tr>
            <td><img width="211" alt="image" src="https://user-images.githubusercontent.com/50509845/220029678-49cc999e-a19f-43b0-8bcd-bb4d8c243fdb.png"></td>
            <td><img width="212" alt="image" src="https://user-images.githubusercontent.com/50509845/220029730-8b5f908a-9bf3-4798-87ca-81a431880afc.png"></td>
      </tr>
      <tr>
            <td colspan="2"><img width="205" alt="image" src="https://user-images.githubusercontent.com/50509845/220029794-a123bb71-b6f5-436b-8104-ddd116c77411.png"></td>
      </tr>
</table>

# 3. EDA 분석 결과
- 이진형
  - 컬럼
    - gender
    -  hypertension 
    - ever_married
    - heart_disease 
    - Residence_type

  - 인코딩
    - 숫자로 치환 후 OneHotEncoding 인코딩 진행

- 명목형
  - 컬럼
    - work _type
    - smoking_status

  - 인코딩
    - OneHotEncoding 인코딩 진행

- 연속형
  - 컬럼
    - age
    - avg_glucose_level
    - bmi 
  - 스케일러
    - MinMax, Standard, MaxAbs, Robust, Log등 다양하게 조합

# 4. 피처엔지니어링 결과
- 피처의 고유값별 정답값1인 데이터의 비율이 기존 train데이터에 부족하다고 판단.
- 캐글의 해당 에피소드 원본데이터 중 stroke 값이 1인 데이터를 불러와 병합 (학습성능 향상에 도움이 될거라 예측)
- 연속형수치인 피처들은 순서형으로 변환하지 않고 그대로 스케일링하여 학습하는 것이 학습성능이 높게 나왔다.

# 5. 베이스라인 모델 구축 및 기본학습
- XGBoost 모델 선정 
- 조원 모두 XGBoost 모델의 결과가 좋았다
- 원본데이터에는 결측치가 존재 -> XGBoost 모델이 결측치를 자동 처리해주므로  XGBoost 모델로 진행하기로 결정 
- 피처 엔지니어링 단계에서, 각피처들의 특성별로 처리해서 만들 수 있는 모든 조합(1000개)으로 XGBoost 모델에 K-Fold교차검증을 진행
- 검증 결과를 통한 점수를 내림차순으로 정렬하여 상위100개 조합을 엑셀에 저장
- 상위100개 중 높은 빈도의 데이터 엔지니어링 방법을 추출(ex) work_type3 / smoking2 or 3 ) -> 여기서 20개의 조합이 생성
- 20개의 조합을 전부 제출하여 조원 개인별 시도결과보다 점수가 향상되었는지 확인
- 위의 일련의 과정의 결과가 조원들의 개인 시도 결과에 비해 높은 점수가 나왔으므로 방향성이 맞다고 판단, 이에 얻어진 결과에 최적의 하이퍼파라미터를 추가로 구성하고자 함


# 6. 모델 성능 향상
- 최적 XGBoost모델의 최적 파라미터를 찾기위해 우선적으로 베이지안 최적화를 통해 하이퍼파라미터를 추출
- 베이지안이 찾아준 파라미터를 기본베이스로, 다른 파라미터를 넣고, 값도 바꾸어보면서 최적의 파라미터 값을 도출!
- 이렇게 찾은 최적의 모델과 최적의 파라미터들을 가지고 다시 모든 조합(1000개)의 성능 값을 확인한 후 캐글에 제출하는 것까지를 자동화 시켜 진행

# 7. 최적 모델 선정
- TOP100 중 높은 빈도의 데이터 엔지니어링 방법을 추출, 약 20개를 반자동화 학습, 파일생성하여 제출

- 최고 점수
  - private  : 0.90044
  - 770팀중 3등!!

<img width="452" alt="image" src="https://user-images.githubusercontent.com/50509845/220031638-277693cb-e013-4652-bbbf-b6f777d5f83e.png">
