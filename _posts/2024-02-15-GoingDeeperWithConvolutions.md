---
layout: post
title: Going Deeper with convolutions [논문]
author: Hun
tags:
- Paper
- GoogLeNet
- Inception 
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


NIN(Network-in-Network)에서는 CCCP(Casecaded Cross Channel Pooling)이란 기법이 사용되었다. 이는 하나의 feature map에 대해 수행하는 일반적인 pooling 기법과 달리 channel을 직렬로 묶어 픽셀 별로 pooling을 수행하는 것으로, feature map의 크기는 그대로이고, channel의 수만 줄어들게 하여 차원 축소의 효과가 있다. 

CCCP 작동 방식:

1. 채널 직렬 연결: 먼저, 서로 연관성이 높은 채널들을 그룹화하여 직렬로 연결합니다.
2. 픽셀별 풀링: 각 채널 그룹에 대해 픽셀별 최대 풀링 또는 평균 풀링을 수행합니다.
3. 채널 축소: 풀링 결과를 통해 채널 수를 줄여 계산 효율성을 높입니다.(3D -> 1D)
4. 다단계 풀링: 위의 과정을 여러 단계 반복하여 더욱 강력한 특징 추출을 수행합니다.

그런데 이 CCCP 기법은 1 x 1 Convolutional layer과 그 연산 방식 및 효과가 매우 유사하다. 따라서 GoogLeNet에서 1 x 1 Convolutional layer를 Inception module에 적용한 것이다.
</details>

