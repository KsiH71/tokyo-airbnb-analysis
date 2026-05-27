# tokyo-airbnb-analysis
도쿄 에어비앤비 숙박 가격 분석 - 데이터마이닝 팀 프로젝트
###### 도쿄 에어비앤비 숙박 가격 분석을 위한 데이터 전처리 및 공간 파생 변수 구축

**\[데이터마이닝] 도쿄 에어비엔비 분석**  
**팀원:** 곽정원, 강시현, 강준우, 장민석

---

###### 1\. 프로젝트 배경 및 데이터 전처리 목표

최근 일본 도쿄는 한국인 여행객에게 가장 인기 있는 여행지 중 하나입니다. 숙소 선택 시 위치, 숙소 유형, 편의시설 등 다양한 요소가 가격에 영향을 미칩니다.
본 단계에서는 향후 머신러닝 모델링(가격 구간 예측 및 주요 요인 분석)을 수행하기 위해, 에어비앤비(Airbnb) 숙소 데이터와 일본 국토교통성의 공간(GIS) 데이터를 융합하여 핵심 파생 변수를 생성하고 노이즈를 제거하는 데이터 전처리 파이프라인 구축을 목표로 합니다.

---

###### 2\. 활용 데이터 (Data Sources)

1. **Airbnb 도쿄 숙소 데이터 (listings.csv)**
   * 출처: [Inside Airbnb](https://insideairbnb.com/get-the-data/)
   * 내용: 숙소 유형, 수용 인원, 침실/욕실 수, 편의시설, 평점, 리뷰 수, 호스트 정보 등

2. **도쿄 철도역 데이터 (N02-25\_Station.geojson)**
   * 출처: [일본 국토교통성 MLIT](https://nlftp.mlit.go.jp/ksj/gml/datalist/KsjTmplt-N02-2025.html) (2025년)
   * 내용: 도쿄 시내 철도역의 위경도 좌표 데이터 (Point/Line)

3. **도쿄 공시지가 데이터 (L01-26\_13.geojson)**
   * 출처: [일본 국토교통성 MLIT](https://nlftp.mlit.go.jp/ksj/gml/datalist/KsjTmplt-L01-2026.html) (2026년)
   * 내용: 2026년 기준 도쿄 지역 공시지가(엔/㎡) 데이터

4. **도쿄 행정구역 데이터 (neighbourhoods.geojson)**
   * 출처: [Inside Airbnb](https://insideairbnb.com/get-the-data/)
   * 내용: 도쿄 행정구역 경계 데이터 (지도 시각화용)

> ⚠️ listings.csv는 용량 문제로 레포에 포함되지 않습니다. 위 링크에서 직접 다운로드하세요.

---

###### 3\. 기술 스택 (Tech Stack)

* **Language:** Python 3.x / Google Colab
* **Data Manipulation:** pandas
* **Geospatial Analysis:** geopandas, shapely
* **Modeling:** scikit-learn, xgboost
* **Visualization:** matplotlib, seaborn

---

###### 4\. 프로젝트 구조

```
📁 tokyo-airbnb-analysis/
│
├── 📁 data/
│   ├── final_airbnb.csv
│   ├── N02-25_Station.geojson
│   ├── L01-26_13.geojson
│   ├── neighbourhoods.geojson
│   └── README_data.md
│
├── 📁 notebooks/
│   ├── 01_preprocessing.ipynb
│   ├── 02_cleaning.ipynb
│   └── final_model.ipynb
│
└── README.md
```

---

###### 5\. 전처리 파이프라인

```
listings.csv (원본 27,945행)
    ↓ 2~4인 숙소 필터링
    ↓ 역 거리 결합 (최근접 역 거리 + 500m 내 역 개수)
    ↓ 주요 중심지 거리 계산 (신주쿠 / 도쿄역 / 시부야)
    ↓ 공시지가 결합
    ↓ 결측치 제거
final_airbnb.csv (최종 12,222행, 37개 변수)
```

###### 생성된 주요 파생 변수

| 변수명 | 설명 |
|---|---|
| dist_to_station | 가장 가까운 철도역까지의 거리 (m) |
| stations_within_500m | 반경 500m 내 역 개수 |
| dist_to_Shinjuku | 신주쿠역까지의 거리 (m) |
| dist_to_Tokyo | 도쿄역까지의 거리 (m) |
| dist_to_Shibuya | 시부야역까지의 거리 (m) |
| land_price | 인근 공시지가 (엔/㎡) |

---

###### 6\. 실행 방법 및 필수 라이브러리 설치 (Getting Started)

본 프로젝트는 geopandas를 활용한 공간 데이터 연산을 포함하고 있으므로, 전처리 코드 실행 전 아래의 패키지 설치가 필수적입니다.

**Google Colab 또는 로컬 환경(Terminal)에서 아래 명령어 실행:**

```bash
pip install pandas geopandas shapely scikit-learn seaborn matplotlib xgboost scipy
```

**노트북 실행 순서:**

```
01_preprocessing.ipynb  →  final_airbnb_with_spatial_and_land_features.csv 생성
02_cleaning.ipynb       →  final_airbnb.csv 생성 (최종 산출물)
final_model.ipynb       →  모델 학습 및 결과 시각화
```
