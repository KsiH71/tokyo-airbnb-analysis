# tokyo-airbnb-analysis
도쿄 에어비앤비 숙박 가격 분석 - 데이터마이닝 팀 프로젝트
###### 도쿄 에어비앤비 숙박 가격 분석을 위한 데이터 전처리 및 공간 파생 변수 구축

**[데이터마이닝] 도쿄 에어비앤비 분석**  
**팀원:** 곽정원, 강시현, 강준우, 장민석

---

###### 1. 프로젝트 배경 및 목표

최근 일본 도쿄는 한국인 여행객에게 가장 인기 있는 여행지 중 하나입니다. 숙소 선택 시 위치, 숙소 유형, 편의시설 등 다양한 요소가 가격에 영향을 미칩니다.

본 프로젝트는 도쿄 에어비앤비 숙소 데이터와 일본 국토교통성의 공간(GIS) 데이터를 융합하여 **"어떤 요인이 도쿄 에어비앤비 가격을 결정하는가"** 를 분석하고, 신규 호스트를 위한 데이터 기반 가격 전략을 도출하는 것을 목표로 합니다.

머신러닝 모델(RandomForest, Ridge, GradientBoosting)을 활용한 가격 회귀 분석을 수행하였으며, 위치·내부스펙·호스트·편의시설 등 다양한 변수군의 영향력을 비교 분석하였습니다. 특히 호스트 전략 도출 단계에서는 단순 그룹별 중앙값 비교 대신 **부분 의존성(Partial Dependence) 기반의 한계 효과 분석**을 적용하여, 위치·지가 등 교란 변수를 통제한 각 변수의 순수 가격 기여도를 정량화하였습니다.

---

###### 2. 활용 데이터 (Data Sources)

1. **Airbnb 도쿄 숙소 데이터 (listings.csv)**
   * 출처: [Inside Airbnb](https://insideairbnb.com/get-the-data/)
   * 내용: 숙소 유형, 수용 인원, 침실/욕실 수, 편의시설, 평점, 리뷰 수, 호스트 정보 등
> ⚠️ listings.csv는 용량 문제로 파일이 레포에 포함되지 않습니다. 위 링크에서 직접 다운로드하세요.

2. **도쿄 철도역 데이터 (N02-25_Station.geojson)**
   * 출처: [일본 국토교통성 MLIT](https://nlftp.mlit.go.jp/ksj/gml/datalist/KsjTmplt-N02-2025.html) (2025년)
   * 내용: 도쿄 시내 철도역의 위경도 좌표 데이터

3. **도쿄 공시지가 데이터 (L01-26_13.geojson)**
   * 출처: [일본 국토교통성 MLIT](https://nlftp.mlit.go.jp/ksj/gml/datalist/KsjTmplt-L01-2026.html) (2026년)
   * 내용: 2026년 기준 도쿄 지역 공시지가(엔/㎡) 데이터

4. **도쿄 행정구역 데이터 (neighbourhoods.geojson)**
   * 출처: [Inside Airbnb](https://insideairbnb.com/get-the-data/)
   * 내용: 도쿄 행정구역 경계 데이터 (지도 시각화용)

---

###### 3. 기술 스택 (Tech Stack)

* **Language:** Python 3.x / Google Colab
* **Data Manipulation:** pandas, numpy
* **Geospatial Analysis:** geopandas, shapely
* **Modeling:** scikit-learn
* **Visualization:** matplotlib, seaborn, koreanize-matplotlib

---

###### 4. 프로젝트 구조

```
📁 tokyo-airbnb-analysis/
│
├── 📁 data/
│   ├── final_airbnb.csv                  ← 신주쿠 포함 원본
│   ├── final_airbnb_without_Shinjuku.csv ← 분석에 사용한 최종 데이터
│   ├── N02-25_Station.geojson
│   ├── L01-26_13.geojson
│   ├── neighbourhoods.geojson
│   └── README_data.md
│
├── 📁 notebooks/
│   ├── 01_preprocessing.ipynb
│   ├── 02_cleaning.ipynb
│   └── tokyo_airbnb_price_analysis.ipynb  ← 모델 학습 및 결과 시각화 (최종)
│
├── 📁 presentation/
│   └── 도쿄_에어비앤비_가격_분석_및_호스트_전략.pptx
│
└── README.md
```

---

###### 5. 전처리 파이프라인

```
listings.csv (원본 27,945행)
    ↓ 2~4인 숙소 필터링
    ↓ 역 거리 결합 (최근접 역 거리 + 500m 내 역 개수)
    ↓ 주요 중심지 거리 계산 (도쿄역 / 시부야)
    ↓ 공시지가 결합
    ↓ 결측치 제거
final_airbnb.csv (신주쿠 포함)
    ↓ 신주쿠 제외 (아래 참조)
    ↓ IQR 기반 가격 이상치 제거
final_airbnb_without_Shinjuku.csv (최종 분석 데이터, 11,977행)
```

###### 생성된 주요 파생 변수

| 변수명 | 설명 |
|---|---|
| dist_to_station | 가장 가까운 철도역까지의 거리 (m) |
| stations_within_500m | 반경 500m 내 역 개수 |
| dist_to_Tokyo | 도쿄역까지의 거리 (m) |
| dist_to_Shibuya | 시부야역까지의 거리 (m) |
| land_price | 인근 공시지가 (엔/㎡) |
| log_land_price | 공시지가 로그 변환값 |
| dist_to_station_km | 역까지 거리 km 변환값 |

---

###### 6. 분석 방법론 — 부분 의존성(Partial Dependence) 기반 한계 효과 분석

본 프로젝트의 호스트 전략 도출 단계는 단순 그룹별 중앙값 비교가 아닌, 학습된 RandomForest 모델을 활용한 **부분 의존성 분석**으로 수행되었습니다.

### 방법

도쿄 마켓 표준 가상 인스턴스(`base_host`)를 기준으로, 다른 모든 변수를 고정한 채 관심 변수만 변화시켰을 때의 예측 가격 변화량을 측정하여 각 변수의 **순수 가격 기여도**를 산출합니다. (예측값은 로그 스케일이므로 `np.expm1`로 실제 엔화 환원 후 계산)

### 중앙값 비교 방식 대비 개선점

단순 중앙값 비교는 위치·지가 등 교란 변수가 통제되지 않아 효과가 과대평가되는 문제가 있습니다. 예를 들어 슈퍼호스트 프리미엄은 중앙값 기준 약 +19.9%로 나타났으나, 교란 변수를 통제한 부분 의존성 기준으로는 +0.5%(약 +68엔)로, 상당 부분이 입지 프리미엄에 의한 착시였음이 확인되었습니다.

### 주요 분석 결과

- 교란 변수 통제 후 **수용 인원 확장(3→4인, +약 3,200엔)** 이 가장 강력한 단가 상승 요인으로 확인
- 편의시설 중에서는 세탁기·주방 등이 유의미한 순수 기여도를 보인 반면, 일부 시설은 효과가 미미하거나 음(-)의 값으로 반전
- 공시지가 대비 숙박 수익 효율은 역 1.0~1.25km 구간에서 최고치

---

###### 7. ⚠️ 데이터 전처리 노트 — 신주쿠(Shinjuku) 제외

### 제외 이유

본 프로젝트는 3개의 Airbnb 데이터셋을 병합하여 사용했으며, 신주쿠구의 경우 데이터셋 간 **중복 숙소가 과도하게 발생**하는 문제가 확인됨.

신주쿠역 일대는 행정구역상 '신주쿠(Shinjuku)', '시부야(Shibuya)', '타이토(Taito)' 등 **여러 neighbourhood에 걸쳐 중복 등록**되는 구조적 특성을 가짐. 이는 서울의 강남역·역삼역처럼 두 역의 실질 상권과 거주 구역이 거의 겹치는 것과 동일한 원리로, 신주쿠 일대는 행정 경계와 실제 생활권 경계가 불일치하여 단일 숙소가 복수의 neighbourhood로 집계되는 현상이 나타남.

이를 그대로 분석에 포함할 경우:

- 신주쿠 일대 숙소가 **실제보다 과대 대표**되어 모델 편향 유발
- 기타 지역 대비 중앙 가격이 유의미하게 높아 **상관관계 분석 왜곡**
- 본 분석 목적인 **일반 호스트 대상 가격 전략 도출**과 맞지 않음

신주쿠 포함/제외 시 상관관계 히트맵 비교는 `tokyo_airbnb_price_analysis.ipynb` 마지막 부록 셀 참조.

### 데이터 파일

| 파일 | 설명 |
|------|------|
| `final_airbnb.csv` | 신주쿠 포함 원본 |
| `final_airbnb_without_Shinjuku.csv` | 분석에 사용한 최종 데이터 |

---

###### 8. 실행 방법 (Getting Started)

본 프로젝트는 geopandas를 활용한 공간 데이터 연산을 포함하고 있으므로, 전처리 코드 실행 전 아래 패키지 설치가 필수적입니다.

> ⚠️ **Windows 환경 주의:** geopandas는 내부적으로 GDAL 등 C 라이브러리에 의존합니다. pip 설치가 실패할 경우 [Anaconda](https://www.anaconda.com/) 환경 사용을 권장합니다.

**Google Colab 또는 로컬 환경에서 아래 명령어 실행:**

```bash
pip install pandas geopandas shapely scikit-learn seaborn matplotlib scipy koreanize-matplotlib
```

**노트북 실행 순서:**

```
01_preprocessing.ipynb              →  공간 파생 변수 생성
02_cleaning.ipynb                   →  final_airbnb.csv 생성
tokyo_airbnb_price_analysis.ipynb   →  모델 학습 및 결과 시각화 (최종)
```
