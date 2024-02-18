---
layout: post
title: Going Deeper with convolutions [논문]
author: Hun
tags:
- Paper
- GoogLeNet
- DeepLearning
date: 2024-02-15 00:18 +0800
last_modified_at: 2024-02-15 01:08:25 +0800
toc: true
math: true
---

저자 : Christian Szegedy 등등

## Introduction

GoogLeNet의 특징
- 연산을 하는 데 소모되는 자원의 사용 효율이 개선되었다.
- Inception은 GoogLeNet의 코드네임.

딥러닝의 발전 동향이 더 좋은 하드웨어의 성능, 더 큰 dataset, 더 큰 모델 때문이기 보다는 새로운 아이디어와 알고리즘, 그리고 개선된 신경망 구조 덕분이다라고 언급.

GoogLeNet은 AlexNet보다 파라미터가 12배나 더 적음에도 불구하고 훨씬 정확했다고 함. 이런 개선은 R-CNN과 같이 deep한 구조와 클래식한 컴퓨터 비전의 시너지 덕분이다.

Mobile 및 Embedded 환경에서는 특히 전력 및 메모리 사용량 관점에서 효율적인 알고리즘의 중요성이 대두되고 있기에, 이 논문에서는 모델이 엄격한 고정된 구조를 가지는 것보다 유연한 구조를 가지게끔 한다. 또한 추론 시간에 1.5B 이하의 연산만을 수행하도록 설계했다.

GoogLeNet의 코드네임인 Inception은 영화 인셉션에 "we need to go deeper"에서 유래했고 여기서 deep은 두가지 의미를 가짐.
1. "Inception module"의 형태로 새로운 차원의 구조 도입
2. 두 번째는 네트워크의 깊이가 증가하였다는 직접적인 의미

## Related Work

LeNet-5을 시작으로 CNN은 일반적인 표준 구조를 가지는데, Convolutional layer가 쌓이고 그 뒤에 1개 또는 그 이상의 FC layer가 따라오는 구조이다. 또한 대용량 데이터에서 layer 수와 사이즈를 늘리고, overfitting을 피하기 위해 dropout을 적용한다.

GoogLeNet도 이와 같은 구조를 가진다.

Network-in-Network : 1 X 1 Convolutional layer를 사용해 네트워크의 표현력을 증가시키고 계산 효율성을 높이는 접근 방법이다.
이 방법은 1 X 1 Convolutional layer가 추가되며, ReLU activation이 뒤따른다.

이때, 1 X 1 Convolutional layer의 목적은 2가지로 이 이유로 GoogLeNet에서 사용된다.

1. 병목현상을 제거하기 위한 차원 축소
2. 네트워크 크기 제안한다

<details>
<summary>
Network-in-Network에서는 왜 1 X 1 Convolutional layer를 사용했고, 이는 어떻게 위 두가지 장점을 가지게 되는가?
</summary>

<br />
<img src="/GoogLeNet_11Convolution.png">

NIN(Network-in-Network)에서는 CCCP(Casecaded Cross Channel Pooling)이란 기법이 사용되었다. <br />이는 하나의 feature map에 대해 수행하는 일반적인 pooling 기법과 달리 channel을 직렬로 묶어 픽셀 별로 pooling을 수행하는 것으로, feature map의 크기는 그대로이고, channel의 수만 줄어들게 하여 차원 축소의 효과가 있다. 
<br /><br />
CCCP 작동 방식:
<br /><br />
1. 채널 직렬 연결: 먼저, 서로 연관성이 높은 채널들을 그룹화하여 직렬로 연결합니다.<br />
2. 픽셀별 풀링: 각 채널 그룹에 대해 픽셀별 최대 풀링 또는 평균 풀링을 수행합니다.<br />
3. 채널 축소: 풀링 결과를 통해 채널 수를 줄여 계산 효율성을 높입니다.(3D -> 1D)<br />
4. 다단계 풀링: 위의 과정을 여러 단계 반복하여 더욱 강력한 특징 추출을 수행합니다.<br /><br />

그런데 이 CCCP 기법은 1 x 1 Convolutional layer과 그 연산 방식 및 효과가 매우 유사하다. 따라서 GoogLeNet에서 1 x 1 Convolutional layer를 Inception module에 적용한 것이다.
</details>

## Motivation and High Level Considerations

최근 deep neural network의 성능은 모델의 사이즈가 증가함으로써 좋아졌는데 depth와 width가 커졌다.

1. depth : level의 수
2. width : level의 유닛의 수

이는 좋은 성능을 얻을 수 있는 방법이지만, 두 가지 문제점이 있다. 

첫번째로, 큰 사이즈를 가진다는 것은 많은 파라미터를 가진다는 것인데, training set이 충분하지 않은 모델에서 이는 오버피팅을 야기한다.(특히, 세부 분류가 중요한 ImageNet에서)

두번째로, 사이즈가 커짐에 따라 많은 컴퓨팅 성능이 필요해진다. 한 예로 두 Convolutional layer가 연결되어 있다면, 필터의 수가 증가함에 따라 연산량은 quadratic하게 증가한다. 그런데 만약 대부분의 기울기가 0이라면 이러한 연산은 의미가 없고 리소스 낭비이다.

<details>
<summary>왜 필터 수에 따라 연산량이 quadratic하게 증가하는가?</summary>

3 X 3 Convolutional layer에서<br />
Conv filter C1과 C2가 있을때, 두 Convolutional layer가 연결되어 있다면 연산량은<br />
3 X 3 X C1 X C2 인데, C1과 C2의 크기가 동일하다면 3 X 3 X C1^2 이다.
</details>

<img src="/GoogLeNet_sparse.png">

위 두 문제를 해결하는 방법은 dense한 Fully-Connected 구조에서 Sparsely Connected 구조로 바꾸는 것이다.<br />
data를 sparse하면서도 큰 구조로 바꾼다면, 이를 통계 분석해 최적의 학습경로를 찾을 수 있다.<br />
하지만 현재 컴퓨터 인프라는 sparse한 데이터를 처리하는데 비효율적이다.<br />
초기 CNN에서는 sparse Connection을 사용했지만, 병렬 컴퓨팅에 더 최적화하기 위해 다시 Full Connection으로 돌아갔다.

<img src="/GoogLeNet_clustering.png">

이때, sparse한 matrix를 다룬 다른 논문에서는 Sparse한 matrix를 Clustering하여 상대적으로 Dense한 Submatrix를 만드는 것을 제안했고, 좋은 성능을 보였다고 한다.

Google팀의 Inception 구조는 이런 Sparse 구조를 시험하기 위해 시작되었고, running-rate와 hyper-parameter를 조정하고 개선한 결과, Localization 및 Object detection 분야에서 특히 좋은 성능을 보였다고 한다.

## Architectural Details

Inception 구조의 주요 아이디어는 Sparse matrix을 서로 묶어(clustering) 상대적으로 Dense한 Submatrix를 만드는 것이다.

이떄, 이전 layer의 각 유닛이 입력 이미지의 특정 부분에 해당한다고 가정했는데,<br />
입력 이미지와 가까운 낮은 layer에서는 특정 부분에 Correlated unit들이 집중되어 있어, 1 X 1 Convolution으로 처리할 수 있다.

<img src="/GoogLeNet_mouse.png">

하지만, 몇몇 위치에서는 위 사진과 같이 더 넓은 Convolutional filter가 필요해 1 X 1, 3 X 3, 5 X 5 Convolution 연산을 병렬적으로 수행한다.

1 X 1, 3 X 3, 5 X 5 Convolutional filter의 수는 Network가 깊어짐에 따라 달라진다.<br />
높은 layer에서만 포착 가능한 추상적 개념의 특징이 있다면 3 X 3, 5 X 5 Convolutional filter의 수가 늘어나야 한다.

여기서 문제가 발생하는데,<br />
3 X 3, 5 X 5 Convolutional filter의 수가 증가하면 연산량이 매우 커지게 된다.

<img src="/GoogLeNet_figure2.png">

따라서 이 문제를 해결하기 위해 1 X 1 Convolutional filter를 이용해 차원을 축소한다. 3 X 3, 5 X 5 Convolutional filter 앞에 1 X 1을 두어 차원을 줄인다.<br />
이를 통해 여러 scale을 확보하면서도 연산량을 낮출 수 있다.
추가적으로 Convolution 연산 이후 ReLU를 추가해 비선형적 특징을 더 추가할 수 있다.

Google팀에서는 효율적인 메모리 사용을 위해 낮은 layer에서는 기본적인 CNN모델을 사용하고 높은 layer에서는 Inception module을 사용하는 것이 좋다고 한다.

이를 통해 얻을 수 있는 장점

1. 과도한 연산량 문제없이 각 단계에서 유닛 수를 상당히 증가시킬 수 있다.(차원 축소를 통해 다음 layer의 input을 조절할 수 있기 때문)
2. Visual 정보다 다양한 scale로 처리되고, 다음 layer는 동시에 서로 다른 layer에서 특징을 추출할 수 있다. (1 X 1, 3 X 3, 5 X 5 Convolution 연산을 통해) 

## GoogLeNet

Inception module이 적용된 전체 GoogLeNet은 LeNet으로 부터 유래했다.

GoogLeNet의 모든 Convolution layer에는 ReLU가 적용된다. 또한 receptive field(수용 영역)는 224 X 224로 RGB 컬러 채널을 가지고 mean subtraction을 적용한다.

<img src="/GoogLeNet_table1.png">

- 3 X 3(5 X 5) reduce : 3 X 3(5 X 5) Convolutional layer 앞에 사용되는 1 X 1 filter의 채널 수를 의미.
- pool proj : max pooling layer뒤에 오는 1 X 1 filter의 채널 수를 의미.

GoogLeNet을 4개의 part로 나누어 보면 다음과 같다. (전체 모델은 페이지 최하단에 첨부. Go to Model Architecture)

### Part 1

<img src="/GoogLeNet_part1.png">

Part 1은 입력 이미지와 가까운 낮은 layer가 위치한 부분으로 효율적인 메모리 사용을 위해 낮은 layer에서는 기본적인 CNN을 사용했다. (선형적임)

### Part 2

<img src="/GoogLeNet_part2.png">

Part 2는 Inception module로 1 X 1, 3 X 3, 5 X 5 Convolutional layer가 병렬적으로 연산을 수행한다. 차원 축소를 위해 1 X 1 Convolutional layer가 적용되어 있다.

### Part 3

<img src="/GoogLeNet_part3.png">

Part 3는 auxiliary classifier가 적용된 부분이다. <br />
모델의 깊이가 매우 깊을 경우, 기울기가 0으로 수렴할 수 있는데 이때 중간 layer에 auxiliary classifier를 추가해 추가적인 역전파를 일으켜 gradient가 전달될 수 있게끔 하면서 정규화 효과가 나타나도록 한다.

추가로 지나치게 영향을 주는 것을 막기 위해 auxiliary classifier의 loss에 0.3을 곱했다.

<details>
<summary>Auxiliary classifier란</summary>

 GoogLeNet (ILSVRC challenge 2014 winner) 에서 처음 도입된 개념 (Training을 잘하도록 도와주는 보조 역할 )

- gradient 전달이 잘 되지 않는 하위 layer을 training하기 위해 사용
- 쉽게 설명하자면 classification의 문제를 해결하는 Neural Network는 softmax를 맨 마지막 layer에 딱 하나만 놓는데, Auxiliary classifier는 중간중간 에 softmax를 두어 중간에서도 Backpropagation을 하게 함
- 이를 통해 gradient가 잘 전달되지 않는 문제 해결함
</details>

### Part 4

<img src="/GoogLeNet_part4.png">

Part 4는 예측 결과가 나오는 모델의 끝부분이다.<br />
여기서 최종 Classifier 이전에 average pooling layer를 사용했는데, 이는 GAP(Global Average Pooling)이 적용된 것으로 이전 layer에서 추출된 feature map을 각각 평균 낸 것을 이어서 1차원 백터로 만들어 준다.(Softmax와 연결하기 위함)

<img src="/GoogLeNet_GAP.png">

위 사진처럼 평균을 내 1차원 백터로 만드는데 FC와 비교해서 가중치의 개수를 줄일 수 있다. FC의 경우 7 * 7 * 1024 = 50176이지만, GAP을 사용하면 단 1개의 가중치도 필요하지 않다. 또한 GAP을 사용할 경우 fine tuning을 하기 쉽게 만든다.

## Training Methodology

여기서는 모델 훈련을 어떻게 하였는지에 대해 설명하고 있다.
Google 팀에서는 0.9 momentum의 Stochastic gradient descent를 이용하였고, learning rate는 8 epochs 마다 4%씩 감소시켰다.
 
또한, 이미지의 가로, 세로 비율을 3 : 4와 4 : 3 사이로 유지하며 본래 사이즈의 8% ~ 100%가 포함되도록 다양한 크기의 patch를 사용하였다. 그리고 photometric distortions(광도 왜곡)를 통해 학습 데이터를 늘렸다고 한다.

<details>
<summary>Photometric distortions란?</summary>

목적:

- 모델의 과적합을 방지하여 일반화 성능을 향상시킵니다.
- 다양한 조명 환경에서도 정확하게 작동하도록 모델을 - 훈련합니다.
- 이미지의 중요한 특징을 강조하고 불필요한 정보를 제거합니다.

종류:

- 밝기 변형:
- 이미지의 전체적인 밝기를 밝게 하거나 어둡게 변형합니다.
- 그림자 영역이나 과도하게 노출된 영역을 보완하는 데 - 효과적입니다.
- 대비 변형:
- 이미지의 대비를 높이거나 낮추어 명암의 차이를 강조하거나 - 완화합니다.
- 물체의 윤곽선을 더욱 선명하게 하거나 이미지의 전체적인 - 분위기를 조절하는 데 효과적입니다.
- 채도 변형:
- 이미지의 채도를 높이거나 낮추어 색상의 선명도를 강조하거나 - 흐릿하게 합니다.
- 특정 색상 채널을 강조하거나 이미지의 색상 톤을 변환하는 데 - 효과적입니다.
- 색상 변형:
- 이미지의 색상을 회색조로 변환하거나 다른 색상 공간으로 - 변환합니다.
- 특정 색상 채널을 제거하거나 이미지의 색상 균형을 조절하는 - 데 효과적입니다.

적용 방법:

- 랜덤하게 이미지의 밝기, 대비, 채도, 색상을 변형합니다.
- 특정 범위 내에서 정해진 값만큼 변형합니다.
- 이미지의 특정 영역만 선택적으로 변형합니다.

장점:

- 모델의 일반화 성능을 향상시킵니다.
- 다양한 조명 환경에서도 정확하게 작동하도록 모델을 - 훈련합니다.
- 이미지의 중요한 특징을 강조하고 불필요한 정보를 제거합니다.

단점:

- 과도하게 적용하면 모델의 성능을 저하시킬 수 있습니다.
- 이미지의 자연스러움을 손상시킬 수 있습니다.
</details>

## Conclutions

Inception 구조는 Sparse 구조를 Dense 구조로 근사화하여 성능을 개선하였다. 이는 기존에 CNN 성능을 높이기 위한 방법과는 다른 새로운 방법이었으며, 성능은 대폭 상승하지만 연산량은 약간만 증가한다는 장점이 있다.

## Model Architecture
<img src="/GoogLeNet_all.png">