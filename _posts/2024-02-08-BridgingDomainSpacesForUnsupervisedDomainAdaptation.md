---
layout: post
title: Bridging Domain Spaces for Unsupervised Domain Adaptation [논문]
author: Hun
tags:
- Paper
- Domain Adaptation
date: 2024-02-08 20:18 +0800
last_modified_at: 2024-02-09 20:18:25 +0800
toc: true
---

저자 : Wonjun Hwang

## 1. Introduction
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

## 2. Related Work

semi supervised learning(준지도 학습)
- 레이블 부족 문제 개선
- 데이터 증강 기반 최근 연구 활발 (MixMatch, FixMatch, ReMixMatch 등)
> MixMatch: 데이터 증강, 엔트로피 최소화
> 
> FixMatch: 일관성 정규화, 의사 레이블, 강화된 증강
> 
> ReMixMatch: MixMatch + FixMatch + 분포 일치 손실 + 자동 증강
- 주로 레이블/레이블되지 않은 데이터가 유사 도메인 가정

SSDA(semi supervised domain adaptation)에 비해 UDA는 더 많은 target lable의 정보를 가지고 있다.
> semi supervised의 신호를 활용한 연구
>
> [Semi-supervised domain adaptation via minimax entropy] : 적응적 few-shot 모델 최적화를 위한 minimax 엔트로피 접근
> [Opposite structure learning for semi-supervised domain adaptation] : 생성기와 두 개의 분류기를 이용하여 반대 구조 학습 통합

[Mixup co-training for semi-supervised domain adaptation. - 44] 과 본 논문 비교. 
- [44]에서는 SSDA를 SSL과 UDA로 분류하여 해결하고자 함.
- 두 가지 모델이 각 하위 문제를 담당하며 co-training 기반으로 학습.
- 한 모델은 labeled source sample 과 labeled target sample로 훈련됨.
- 다른 한 모델은 unlabeled target sample과 labeled target sample로 훈련됨.

반면 ([44]와 다르게) 이 논문에서는 양방향 매칭을 통해 target 도메인의 의사 레이블을 훈련한다. 또한 [44]에서는 일부 target sample을 선택 사용하지만 본 논문에서는 모든 target sample을 사용한다. 

최근 여러 연구에서는 도메인 정렬 및 차별적인 도메인 불변적 특성 학습 방법을 기반으로 UDA에 중점을 두고 있다.

- Deep Adaptation Network(DAN)은 도메인 특정 계층에 대한 MMD를 최소화하고 Joint Adaptation Network는 JMMD를 기반으로 여러 도메인에 걸쳐 도메인 특정 레이어의 공동 분포를 정렬했습니다.
- Deep Domain Confusion(DDC)는 discriminative 도메인과 transferable 도메인을 모두 학습시키기 위해 fully connected layer에서 MMD metric을 사용했다.
- Domain Adversarial Neural Network(DANN)은 도메인 분류기의 reverse gradients를 back-propagating해 도메인 불변 표현을 학습했다.
- Adversarial Discriminative Domain Adaptation(ADDA)는 source label을 사용하여 판별적 표현을 학습한 후 도메인-적대 손실을 기반으로 대상 데이터를 동일한 공간에 매핑하는 별도의 인코딩을 사용한다.
- Maximum classification discrepancy(MCD)는 대상 샘플의 불일치를 최대화한 다음 이 불일치를 최소화하는 기능을 생성하여 작업별 결정 경계를 고려하여 대상 도메인의 분포를 정렬하려고 했다.
- Contrastive adaptation network(CAN)는 클래스 내 도메인 불일치와 클래스 간 도메인 불일치를 명시적으로 모델링하는 도메인 불일치를 최소화하기 위한 metric을 최적화했다.
- Robust spherical domain adaptation(RSDA)는 구형 분류기와 구형 도메인 구별기를 사용하여 레이블 예측과 도메인 구별을 수행하며, 구형 특징 공간에서 강인한 의사 레이블 손실을 활용한다.
- Structurally regularized deep clustering(SRDC) 중간 네트워크 특징을 군집화하고 source example 선택의 구조적 정규화를 통해 target 식별을 개선합니다.
- Dual mixup regularized learning(DMRL) 샘플 간 일관적인 예측 향상과 잠재 공간의 내재 구조 강화를 위해 분류기를 유도합니다. 소스와 타겟 도메인 혼합을 위해 랜덤성 기반 믹스업 정규화 두 가지를 제안합니다.

본 연구에서는 기존 연구 ([41, 43] 등)과 달리 랜덤성에만 의존하여 증강 도메인을 생성하지 않는다. 대신 두 개의 Fixed ratio mixup을 사용해 두 가지 증강 도메인을 생성한다.

1. source closed augmented domain : 명확한 label 존재, target domain과는 거리가 있음.
2. target closed augmented domain : target domain 특성 존재, source domain과는 거리가 있음.

서로 다른 특징을 가진 두 증강 도메인을 사용하여 서로 "가르치는" 방식으로 학습
이를 통해 도메인 지식을 타겟 도메인으로 효과적으로 전달 

## 3. Proposed methods

