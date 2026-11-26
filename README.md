# Project_Yolov8

YOLOv8 차량 부품 세그멘테이션 · 가위바위보 분류

<br/>

<img src="image/car_result.png">

<br/><br/>

## Object

<br/>

- 차량 이미지·영상에서 부품 14종을 실시간으로 인식
- Colab(GPU)에서 학습, 로컬(Visual Studio)에서 예측
- epoch, 해상도, 부품 수를 바꿔가며 성능 개선 테스트
- 가위바위보 분류로 해상도 차이에 따른 결과 비교

<br/><br/>

## Content

<br/>

1. 관련 저장소

2. File 소개

3. 진행 과정

4. Result

5. Version

6. Feedback

<br/><br/>

## 1. 관련 저장소

<br/>

포트폴리오 **스마트 안전 블랙박스** 구성

- 차량 부품 인식 (YOLOv8) : 이 저장소
- 차간거리 추정 (YOLOv5) : [sub-Yolov5_Car_Distance](https://github.com/sinheechan/sub-Yolov5_Car_Distance)
- 안전모 인식 서버 (YOLOv8) : [sub-Visual_intelligence_Hardhat](https://github.com/sinheechan/sub-Visual_intelligence_Hardhat)

<br/><br/>

## 2. File 소개

<br/>

- `Yolov8_seg_car_0227.ipynb` : 차량 부품 세그멘테이션 학습
- `Yolov8_cls_rps_0228.ipynb` : 가위바위보 분류 학습
- `test_car.py` : 로컬 데이터 정리 및 이미지·영상 예측
- `test_rps.py` : 스마트폰 영상 테스트, 해상도별 결과 메모
- `my_seg/` : 학습된 가중치 2종

<br/><br/>

## 3. 진행 과정

<br/>

### ① 데이터 준비

<br/>

- car : [Roboflow part-autolabeld v5](https://universe.roboflow.com/alpaco5-f3woi/part-autolabeld/dataset/5)
  - train 6,603장, valid 1,887장, 14 클래스
- rps : [Roboflow Rock Paper Scissors](https://public.roboflow.com/classification/rock-paper-scissors)
  - train 2,520장, 3 클래스
- train / val 경로를 txt로 만들어 `data.yaml` 에 연결

<br/>

### ② 학습

<br/>

```bash
yolo segment train model=yolov8s-seg.pt data=data.yaml epochs=10
yolo classify train data=rps model=yolov8n-cls.yaml epochs=10
```

<br/>

### ③ 예측

<br/>

```bash
yolo segment predict model=my_seg/my_seg_car_240227.pt source=<경로>
yolo classify predict show model=my_seg/my_seg_rps_240229.pt source=<경로>
```

<br/><br/>

## 4. Result

<br/>

### ① 차량 부품 세그멘테이션

<br/>

- 맨 위 이미지 : 차량 영상 예측 결과
- valid 1,887장, 10 epoch, Colab Tesla T4

<br/>

| 구분 | Precision | Recall | mAP50 | mAP50-95 |
|---|---|---|---|---|
| Box | 0.828 | 0.834 | 0.897 | 0.834 |
| Mask | 0.825 | 0.831 | 0.892 | 0.826 |

<br/>

- 범퍼·펜더·도어 등 큰 부품 : mask mAP50 0.93 이상
- 리어램프 0.774, 윈드실드 0.500으로 상대적으로 낮음
- 윈드실드는 검증 데이터 8개뿐이라 데이터 보강 필요
- 추론 속도 : 이미지당 약 4.9ms

<br/>

### ② 가위바위보 분류

<br/>

<img src="image/rps_result.png">

<br/>

- test top-1 정확도 0.939 (33장)
- 스마트폰 720x1280 영상은 대부분 Rock으로 인식
- 원인 : 학습 이미지가 모두 300x300 정사각형
- 1080x1080으로 바꾸자 Paper 인식 개선
- Scissors 인식은 여전히 낮음

<br/>

### ③ 테스트에서 확인한 점

<br/>

- loss 하락·precision 상승 구간에서 epoch 조정 시 성능 향상
- 학습 데이터와 입력 영상의 해상도를 맞추면 소폭 향상
- 인식 부품 수를 줄이면(바퀴·사이드미러·범퍼) 성능 개선

<br/><br/>

## 5. Version

<br/>

- python 3.12.2 (로컬) / 3.10 (Colab)
- numpy 1.26.4
- pandas 2.2.1
- opencv-python 4.9.0.80
- ultralytics 8.1.19

<br/><br/>

## 6. Feedback

<br/>

- 10 epoch만 학습, epoch 확대·증강으로 약한 클래스 개선 여지
- 스크립트 경로가 로컬 절대경로라 환경별 수정 필요
