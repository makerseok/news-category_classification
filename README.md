# 네이버 뉴스 카테고리 분류
![화면 캡처 2021-06-25 173855](https://user-images.githubusercontent.com/67940953/123396443-47429780-d5dc-11eb-873b-144db1a7d05c.png)
## 프로젝트 개요
네이버 뉴스의 6개 카테고리를 beattifulsoup을 사용해 크롤링한 후 tensorflow로 카테고리 예측
<br>

## 사용 언어
- python: 3.7.10

## 개발 환경 및 사용 모듈
- numpy: 1.20.2
- pandas: 1.2.4
- selenium: 3.141.0
- konlpy: 0.5.2
- scikit-learn: 0.24.2
- tenforflow: 2.3.0

## 프로젝트 수행 방법
### 크롤링
네이버 뉴스에서 정치부터 IT/과학까지 총 6개의 카테고리의 뉴스들을 100페이지씩 크롤링한다. 이때 동적으로 접근하기 위해 크롤링에는 selenium을 사용한다.

카테고리별 크롤링된 파일은 하나의 파일로 concat해 저장한다.

### 전처리
뉴스의 제목을 feature, 카테고리를 target으로 분리하고 target을 one-hot encoding한다. 이때 사용한 LabelEncoder 객체는 추후 검증에 사용하기 위해 pickle을 사용해 저장한다.

feature는 자연어 처리를 진행하는데, konlpy의 okt를 사용한다. 이번 프로젝트에선 morphs로 형태소를 분류한 뒤 stopword를 제거한 후 토큰화했다. 토큰화에 사용한 객체 또한 pickle로 저장하였다.

자연어 처리까지 마친 데이터는 각 행마다 길이가 다르기 때문에 가장 형태소 수가 많은 뉴스를 기준으로 길이를 맞추었다. LSTM을 진행하기 때문에 뒷쪽이 아닌 앞쪽에 0을 추가했다.

모든 전처리가 끝난 데이터는 npy 파일로 저장했다.

### 모델 레이어 구성
![화면 캡처 2021-06-25 173855](https://user-images.githubusercontent.com/67940953/123400912-39dbdc00-d5e1-11eb-9f6c-afa81a71b5b4.png)

단어 벡터라이징에는 keras의 Embedding을 사용했다. 학습데이터 수가 차원의 수보다 적어져 성능이 저하되는 것을 막기 위해 output_dim을 적절히 조절했다.


주변 값과의 연관성을 분석하기 위해 Conv1D를 사용했으며, 동작에 영향을 주진 않지만 코드 형식을 통일하기 위해 MaxPool1D를 pool_size를 1로 설정한 뒤 사용했다.


LSTM을 총 3번 사용했는데, 16개의 단어 모두 다음 layer에 전달하기 위해 앞단 2개에는 return_sequences를 True로 설정했다.

이렇게 모델을 학습시킨 결과 과적합이 발생했기 때문에 Dropout layer를 적절히 추가했다.

