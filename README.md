# Project_Yolov8

YOLOv8을 활용한 차량 부품 인스턴스 세그멘테이션 · 이미지 분류 모델 구현

<br/>

<img src="image/car_result.png">

<br/>

<img src="image/rps_result.png">

<br/><br/>

## Object

<br/>

이 프로젝트는 YOLOv8으로 **차량 이미지 · 영상에서 부품(범퍼 · 도어 · 바퀴 · 사이드미러 등 14종)을 실시간으로 인식**하는 것을 목표로 합니다.

Colab(GPU)에서 학습한 세그멘테이션 모델을 로컬(Visual Studio) 환경에서 이미지 · 영상으로 추론하고, 입력 해상도와 클래스 구성을 바꿔가며 인식 성능을 개선하는 테스트를 진행했습니다.

비교 실험으로 가위바위보(RPS) 이미지 분류 모델도 함께 학습해, 학습 데이터와 실제 입력 영상의 해상도 차이가 성능에 미치는 영향을 확인했습니다.

<br/><br/>

## Content

<br/>

1. 관련 저장소

2. File 소개

3. 프로세스 구성

4. Result

5. Version

6. Feedback

<br/><br/>

## 1. 관련 저장소

<br/>

포트폴리오의 **스마트 안전 블랙박스** 항목은 아래 세 저장소로 구성됩니다.

| 기능 | 저장소 |
|---|---|
| 실시간 차량 부품 인식 (YOLOv8 세그멘테이션) | 이 저장소 |
| 블랙박스 영상 차량 · 보행자 탐지 및 차간거리 추정 (YOLOv5) | [sub-Yolov5_Car_Distance](https://github.com/sinheechan/sub-Yolov5_Car_Distance) |
| 건설 현장 CCTV 사람 · 안전모 인식 Flask 분석 서버 (YOLOv8) | [sub-Visual_intelligence_Hardhat](https://github.com/sinheechan/sub-Visual_intelligence_Hardhat) |

<br/><br/>

## 2. File 소개

<br/>

- `Yolov8_seg_car_0227.ipynb` : 차량 부품 세그멘테이션 학습 과정 (Colab)

  ① Roboflow 데이터 다운로드 → train / val 이미지 경로 목록(txt) 생성

  ② `data.yaml` 경로 수정 → `yolov8s-seg.pt` 기반 10 epoch 학습

- `Yolov8_cls_rps_0228.ipynb` : 가위바위보 이미지 분류 학습 과정 (Colab, `yolov8n-cls`)

- `test_car.py` : 로컬 환경에서 데이터 압축 해제 · 경로 구성 후 이미지 / 영상 추론

- `test_rps.py` : 스마트폰 촬영 영상으로 분류 모델 추론 및 해상도별 결과 비교

- `my_seg/` : 학습 완료 가중치 (`my_seg_car_240227.pt`, `my_seg_rps_240229.pt`)

<br/><br/>

## 3. 프로세스 구성

<br/>

### ① 데이터 준비

- car : [Roboflow - part-autolabeld v5](https://universe.roboflow.com/alpaco5-f3woi/part-autolabeld/dataset/5) (train 6,603장 · valid 1,887장, 14 클래스)

  `Bonnet, Frontbumper, Frontdoor, Frontfender, Headlights, Rearbumper, Reardoor, Rearfender, Rearlamp, Rockerpanel, Sidemirror, Trunklid, Wheel, Windshield`

- rps : [Roboflow - Rock Paper Scissors](https://public.roboflow.com/classification/rock-paper-scissors) (train 2,520장, 3 클래스)

<br/>

### ② 학습

```bash
# 차량 부품 세그멘테이션
yolo segment train model=yolov8s-seg.pt data=data.yaml epochs=10

# 가위바위보 분류
yolo classify train data=rps model=yolov8n-cls.yaml epochs=10
```

<br/>

### ③ 추론

```bash
# 이미지 / 영상 예측
yolo segment predict model=my_seg/my_seg_car_240227.pt source=<이미지 폴더 또는 영상 경로>
yolo classify predict show model=my_seg/my_seg_rps_240229.pt source=<영상 경로>
```

<br/><br/>

## 4. Result

<br/>

### ① 차량 부품 세그멘테이션 (valid 1,887장, 10 epoch, Tesla T4)

| 구분 | Precision | Recall | mAP50 | mAP50-95 |
|---|---|---|---|---|
| Box | 0.828 | 0.834 | 0.897 | 0.834 |
| Mask | 0.825 | 0.831 | 0.892 | 0.826 |

<br/>

- 범퍼 · 펜더 · 도어 등 면적이 큰 부품은 mask mAP50 0.93 이상으로 안정적으로 인식했습니다.
- 리어램프(0.774) · 윈드실드(0.500)는 상대적으로 낮았으며, 윈드실드는 검증 데이터 인스턴스가 8개뿐이라 학습 데이터 보강이 필요합니다.
- 추론 속도 : 이미지당 약 4.9ms (T4 GPU)

<br/>

### ② 가위바위보 분류

- test top-1 정확도 0.939 (33장)
- 스마트폰으로 촬영한 720x1280 영상은 대부분 Rock(주먹)으로 인식 → 학습 데이터(300x300 정사각)와 비율이 달랐기 때문
- 영상을 1080x1080 정사각으로 변환하자 Paper(보) 인식 정확도가 개선, Scissors(가위)는 여전히 낮음

<br/>

### ③ 성능 개선 테스트에서 확인한 점

- train / val loss 가 내려가고 precision 이 오르는 구간을 기준으로 epoch 를 조정해 성능을 끌어올릴 수 있었습니다.
- 학습 데이터와 실제 입력의 해상도 · 프레임 비율을 맞췄을 때 성능이 소폭 향상되었습니다.
- 인식 대상 부품 수를 줄였을 때(바퀴 · 사이드미러 · 범퍼 중심) 인식 성능이 개선되었습니다.

<br/><br/>

## 5. Version

<br/>

- python 3.12.2 (로컬) / 3.10 (Colab)
- numpy 1.26.4
- pandas 2.2.1
- opencv-python 4.9.0.80
- ultralytics 8.1.19
- PyYAML

<br/><br/>

## 6. Feedback

<br/>

- 학습을 10 epoch 로 짧게 진행했으므로, epoch 확대와 데이터 증강으로 소수 클래스(윈드실드 · 리어램프) 성능을 높일 여지가 있습니다.
- 스크립트 내 경로가 로컬 절대경로로 되어 있어, 실행 전 본인 환경에 맞게 수정해야 합니다.
