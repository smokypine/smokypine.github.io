## 프로젝트 개요
2022년에 html/css을 이용하여 [여행 사이트 홈페이지](https://smokypine.github.io/)를 제작했습니다.


## Stack
```
html/css
```

## About task
- html을 이용하여 홈페이지를 구현
- css를 이용해 홈페이지 디자인 구성

## 조직도
<img src = "./figure/my_organazation.png" width="80%"><br/><br/>

## 1. 사이트 기능
<img src = "./figure/Main01.png" width="80%"><br/><br/>
- header를 이용해 body와 메뉴를 분리 하였으며 호버링 기능을 통해 하위 메뉴 구현.
- iframe을 이용해 여행 광고 페이지 작성. 광고 내부의 원형 버튼을 이용해 광고 전환 가능.
- Anchor를 통한 페이지 업다운 기능 구현.
- 로그인|회원가입|예약확인/결제|고객센터 기능은 미구현.<br/>

<img src = "./figure/Main02.png" width="80%"><br/><br/>
- hovering 기능을 이용해 호버링된 이미지의 확대/축소 기능 구현.
- textcoloranimation을 이용한 문자의 자동 색상 변화 구현<br/>

<img src = "./figure/Main03.png" width="80%"><br/><br/>
- 페이지 상단 iframe 우측의 앵커 리스트와 연동된 업다운 기능 구현
- [상담 문의 이미지](smokypine.github.io/메인/상담.jpg)에 좌표를 지정하여 상세보기 위치를 클릭하면 ARS 상담 페이지가 출력되는 기능 구현
- footer 로 분리하여 body 하단에 일정한 내용을 출력하였으나 다른 파일로 분리하고 일괄적으로 import를 하는 개념을 몰라 각 파일마다 각각 footer를 구현함.<br/>



## 2. Data preprocessing
- [CLIP](http://proceedings.mlr.press/v139/radford21a/radford21a.pdf)을 이용한 image segmentation ([Image Segmentation Using Text and Image Prompts, CVPR 2022](https://openaccess.thecvf.com/content/CVPR2022/papers/Luddecke_Image_Segmentation_Using_Text_and_Image_Prompts_CVPR_2022_paper.pdf)) 기술 사용
- Text prompt로 "dolphin" 사용 <br/>

<img src = "./figures/clip_seg.PNG" width="65%"><br/><br/>

- Original image를 resize하여 W,H가 같은 image 생성
- 해당 image에 segmentation 수행하여 min, max 좌표 획득
- Resize했던 비율을 역이용하여 original image의 bounding box 좌표 획득

<img src = "./figures/clip_seg2.PNG" width="80%"><br/><br/>

- Original image의 객체와 배경의 비율을 맞추기 위해 zero padding을 사용하여 crop을 하거나 image를 붙여 crop
- 인공지능 모델의 input size를 맞추기 위해 resize 수행

<img src = "./figures/clip_seg3.PNG" width="80%"><br/><br/>

## 3. Re-identification & performance evaluation

3-1 Overall framework

- 총 4단계로 구성

<img src = "./figures/Overall framework.PNG" width="80%"><br/><br/>

3-2 Training settings
```
ResNet-18 & EfficientNet
Input: Anchor, positive, negative data (224 x 224 images)
Output: 512 dim features
Triplet loss, Cross-entropy loss
Adam optimizer, lr 0.0001, 300 epoch training
Metric: MAP@5
```

3-3 Network structure

- $L_{triplet}$과 $L_{species}$를 사용하여 multi-task learning 수행
- $L_{triplet}$은 같은 id를 가진 개체끼리 anchor, positive 구성, 다른 id를 가진 개체의 id를 negative로 구성하여 학습
- $L_{species}$는 30여종의 고래/돌고래 종 분류(classification) 학습
- $\alpha$는 hyperparameter <br/>

<img src = "./figures/Network structure.PNG" width="80%"><br/><br/>

3-4 Make Gallery

- 학습이 끝난 뒤 수행
- 모든 train dataset을 parameter가 고정된 network에 통과시켜 features 생성
- 같은 id의 개체 이미지 feature들을 묶어 gallery set 생성

<img src = "./figures/Make Gallery.PNG" width="80%"><br/><br/>

<img src = "./figures/Make Gallery2.PNG" width="80%"><br/><br/>

3-5 Performance evaluation

- Validation dataset으로 gallery 탐색
  
<img src = "./figures/Performance Evaluation.PNG" width="80%"><br/><br/>

- K-nearest neighbor 알고리듬을 통해 L2 distance가 짧은 K개의 features 선정
- MAP@5 metric에 의거해 K=5로 선정
- Metric에 자세한 설명은 [여기](https://www.kaggle.com/code/pestipeti/explanation-of-map5-scoring-metric) 참고

<img src = "./figures/Performance Evaluation2.PNG" width="80%"><br/><br/>

<img src = "./figures/Performance Evaluation3.PNG" width="80%"><br/><br/>

- $L_{triplet}$에 hard negative mining을 사용한 결과가 더 좋음을 확인할 수 있음
- $||f(A)-f(P)||>|f(A)-f(N)||$인 경우에 loss를 발생시키는 것이 hard negative mining 전략
- Margin parameter $\alpha$를 사용하지 않았을 때가 훨씬 좋음

<img src = "./figures/Performance Evaluation4.PNG" width="80%"><br/><br/>

## 4. Kaggle submission results

- new_indivisual이란 기존 gallery에 있던 개체가 아닌 새로운 개체라고 식별되었을 때 기록
- Re-identification task를 수행한 결과가 더 좋음을 확인할 수 있음
- Threshold는 margin parameter $\alpha$

<img src = "./figures/Test for Kaggle submission.PNG" width="80%">


## 5. Conclusion

- Text와 image를 사용하는 multimodal model을 이용하여  segmentation 진행
- Hard negative mining을 사용하여 성능 향상
- Classifier를 도입하여 multi-task learning 진행
