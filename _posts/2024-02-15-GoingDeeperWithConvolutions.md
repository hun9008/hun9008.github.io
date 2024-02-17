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


