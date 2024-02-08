---
layout: post
title: Bridging Domain Spaces for Unsupervised Domain Adaptation [논문]
author: Hun
tags:
- Paper
- Domain Adaptation
date: 2024-02-08 20:18 +0800
last_modified_at: 2024-02-08 20:18:25 +0800
toc: true
---

저자 : Wonjun Hwang

# 1. Introduction
최근 컴퓨터 비전 딥러닝 애플리케이션에서 상당한 개선을 보였지만, 대부분 지도학습이었다. 지도학습에는 데이터를 수입하는데 많은 비용이 들어 준지도(semi-supervised), 비지도(unsupervised)학습이 연구되었는데 이는 유사한 도메인에서의 학습이었다. 

<img src="/UDA_figure1.png">

> 이전 UDA에서는 source와 target 도메인의 격차를 줄이기 위해 discriminator(판별자), MMD, JMMD 등을 사용했고 GAN 기반 DA가 사용되었다. 기본적으로 이는 source와 target간의 거리가 큰 경우는 고려되지 않는다.

이 논문에서는 큰 도메인 불일치를 효율적으로 보상하기 위해 서로 상보적인 중간 증강 도메인을 구성한다. 이를 위해 Fixed ratio based mixup을 제안한다. 

Fixed ratio based mixup은 source와 target사이 무작위성을 최소화하고 여러 중간 도메인을 생성한다.
> 예를들어 Augmented domain close to the source domain 은 레이블 정보는 있지만 target 도메인과 상관관계는 낮다.
>
> 반면 Augmented domain close to the target domain은 라벨 정보는 부정확하지만 target domain과 유사성은 높다.

이런 증강된 도메인에서 source와 target 도메인 사이를 연결하도록 서로 가르치는 보완모델을 훈련한다.
구체적으로, target 샘플에 대한 높은 신뢰도 예측을 기반으로 한 양방향 매칭을 도입해 중간 도메인을 target 도메인으로 연결시킨다.

또한 self-traning을 통해 성능을 향상시키기 위해, self-penalization을 적용한다.

또한 각 iteration마다 변화하는 특성을 적절히 부과하가 위해 각 미니배치의 신뢰분포에 의한 적응형 임계값을 사용한다.

마지막으로 서로 다른 증강된 모델의 발산을 막고자, source와 target 샘플의 비율이 동일한 증강 도메인을 사용해 일관성 정규화를 제안한다.

Office-31, Office-Home, VisDA에서 테스트를 수행했다.

# 2. Related Work



# 3. Proposed methods

