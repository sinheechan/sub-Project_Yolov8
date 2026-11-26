# Project_Yolov8

YOLOv8로 차량 부품 세그멘테이션 · 가위바위보 분류 모델 만들기

<br/>

<img src="image/car_result.png">

<br/><br/>

## Object

<br/>

YOLOv8 세그멘테이션으로 차량 이미지와 영상에서 범퍼, 도어, 바퀴, 사이드미러 같은 부품 14종을 실시간으로 인식하는 프로그램입니다.

Colab(GPU)에서 학습하고, 학습된 모델을 로컬(Visual Studio)에서 이미지와 영상에 돌려 봤습니다.

YOLO를 그냥 실행했을 때보다 인식이 잘 되게 하려고 epoch, 입력 해상도, 인식 부품 수를 바꿔 가면서 테스트했습니다.

같은 흐름으로 가위바위보 분류 모델도 하나 만들어서, 학습 이미지와 실제 영상의 해상도가 다를 때 결과가 어떻게 달라지는지 확인했습니다.

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

포트폴리오의 **스마트 안전 블랙박스** 항목은 아래 세 저장소를 묶은 것입니다.

- 실시간 차량 부품 인식 (YOLOv8) : 이 저장소
- 블랙박스 영상 차량·보행자 탐지, 차간거리 추정 (YOLOv5) : [sub-Yolov5_Car_Distance](https://github.com/sinheechan/sub-Yolov5_Car_Distance)
- 건설 현장 CCTV 사람·안전모 인식 서버 (YOLOv8) : [sub-Visual_intelligence_Hardhat](https://github.com/sinheechan/sub-Visual_intelligence_Hardhat)

<br/><br/>

## 2. File 소개

<br/>

- `Yolov8_seg_car_0227.ipynb` : 차량 부품 세그멘테이션 학습 (Colab)
- `Yolov8_cls_rps_0228.ipynb` : 가위바위보 분류 학습 (Colab)
- `test_car.py` : 로컬에서 데이터 압축 해제, 경로 정리 후 이미지·영상 예측
- `test_rps.py` : 스마트폰으로 찍은 영상으로 분류 모델 테스트, 해상도별 결과 메모
- `my_seg/` : 학습된 가중치 (`my_seg_car_240227.pt`, `my_seg_rps_240229.pt`)

<br/><br/>

## 3. 진행 과정

<br/>

### ① 데이터 준비

<br/>

- car : [Roboflow - part-autolabeld v5](https://universe.roboflow.com/alpaco5-f3woi/part-autolabeld/dataset/5) (train 6,603장, valid 1,887장)

  클래스 14종 : Bonnet, Frontbumper, Frontdoor, Frontfender, Headlights, Rearbumper, Reardoor, Rearfender, Rearlamp, Rockerpanel, Sidemirror, Trunklid, Wheel, Windshield

- rps : [Roboflow - Rock Paper Scissors](https://public.roboflow.com/classification/rock-paper-scissors) (train 2,520장, 3 클래스)

- 받은 데이터의 train / val 이미지 경로를 txt 파일로 만들고, `data.yaml` 의 경로를 그 파일로 바꿔서 학습에 넣었습니다.

<br/>

### ② 학습

<br/>

```bash
# 차량 부품 세그멘테이션
yolo segment train model=yolov8s-seg.pt data=data.yaml epochs=10

# 가위바위보 분류
yolo classify train data=rps model=yolov8n-cls.yaml epochs=10
```

<br/>

### ③ 예측

<br/>

```bash
yolo segment predict model=my_seg/my_seg_car_240227.pt source=<이미지 폴더 또는 영상>
yolo classify predict show model=my_seg/my_seg_rps_240229.pt source=<영상>
```

<br/><br/>

## 4. Result

<br/>

### ① 차량 부품 세그멘테이션

<br/>

맨 위 이미지가 학습한 모델로 차량 영상을 예측한 결과입니다. 아래는 valid 1,887장 기준, 10 epoch 학습 결과입니다. (Colab Tesla T4)

| 구분 | Precision | Recall | mAP50 | mAP50-95 |
|---|---|---|---|---|
| Box | 0.828 | 0.834 | 0.897 | 0.834 |
| Mask | 0.825 | 0.831 | 0.892 | 0.826 |

<br/>

- 범퍼, 펜더, 도어처럼 큰 부품은 mask mAP50이 0.93 이상으로 잘 잡혔습니다.

- 리어램프(0.774)와 윈드실드(0.500)는 낮게 나왔는데, 윈드실드는 검증 데이터에 8개밖에 없어서 데이터를 더 모아야 할 것 같습니다.

- 추론 속도는 이미지 한 장당 약 4.9ms 였습니다.

<br/>

### ② 가위바위보 분류

<br/>

<img src="image/rps_result.png">

<br/>

- 학습 데이터 기준 top-1 정확도는 0.939 (test 33장) 였습니다.

- 그런데 스마트폰으로 찍은 720x1280 영상을 넣으니 손 모양 대부분을 Rock(주먹)으로 인식했습니다.

- 학습 이미지가 전부 300x300 정사각형이라서, 영상도 1080x1080 정사각형으로 바꿔서 다시 돌려 보니 Paper(보) 인식이 좋아졌습니다. Scissors(가위)는 여전히 잘 못 맞춥니다.

<br/>

### ③ 테스트하면서 알게 된 점

<br/>

- train / val loss 가 내려가고 precision 이 올라가는 동안은 epoch 를 늘려서 성능을 더 끌어올릴 수 있었습니다.

- 학습 데이터와 실제 입력 영상의 해상도, 프레임 비율을 맞추면 성능이 조금 좋아졌습니다.

- 인식할 부품 수를 바퀴, 사이드미러, 범퍼 위주로 줄였을 때 인식이 더 잘 됐습니다.

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

- 10 epoch 만 돌렸기 때문에 epoch 를 늘리고 데이터 증강을 하면 리어램프, 윈드실드 같은 약한 클래스를 더 올릴 수 있을 것 같습니다.

- 스크립트 안 경로가 제 PC 기준 절대경로라서, 다른 환경에서 돌릴 때는 경로를 바꿔야 합니다.
