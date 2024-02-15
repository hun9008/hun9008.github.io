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


