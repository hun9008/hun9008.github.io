---
layout: post
title: Transformer
author: Hun
tags:
- paper
- deeplearning
date: 2024-09-10 09:18 +0800
last_modified_at: 2024-09-10 09:28:25 +0800
toc: true
---

<img src="/Transformer_architecture.png">

학교 AI 수업 프로젝트에 앞서, 이전 ML 시간에 배웠던 Transformer를 정리해봤다.

## Transformer 등장 배경

이전 RNN, LSTM은 input data를 순차적으로 처리하기 때문에 {정보 손실, 병렬처리}가 불가능. 그래서 새로운 Attention Mechanism 등장했다. 

## Keyword

<img src="/Transformer_qkv.png">

Input에 대해 $W^Q, W^K, W^V$를 곱해 Q, K, V를 만든다. (초기 W들은 일반적으로 Gaussian 분포에서 무작위로 정해지고, train 단계에서 backpropagation으로 학습되는 대상.) 다른 단어(input)과의 연관성을 고려하기 위한 방법론.

 

1. Query(Q) : 현재 토큰이 다른 토큰들과 얼마나 관련이 있는지를 평가.
2. Key(K) : 다른 토큰들이 제공하는 정보를 나타냄. (DB의 Key)
3. Value(V) : 실제로 전달하고자 하는 정보.

## Core Mechanism

### Self Attention (Batch Calculation)

<img src="/Transformer_qkv_2.png">

$$
Attention(Q,K,V) = softmax(\frac{QK^T}{\sqrt{d_k}}) V
$$

사실 이 공식이 핵심.

예를들어 아래와 같은 문장이 Input 이라 할때, Self-Attention 과정은 아래와 같다.

1. Input Embedding
    
    ```sql
    "The"  -> 벡터 A
    "cat"  -> 벡터 B
    "eat"  -> 벡터 C
    "meat"   -> 벡터 D
    ```
    
    Input String을 각 단어 벡터로 embedding한다.
    
2. Q, K, V 생성
    
    예를 들어 “cat”의 단어 벡터를 B라 하면,
    
    - Q(cat) = $W^Q * B$
    - K(cat) = $W^K * B$
    - V(cat) = $W^V * B$
    
    이와 같이 “cat”에 대한 Q, K, V를 생성한다.
    
3. Attention Score 계산
    
    “cat”이 다른 단어들과 어떻게 연관되는지 계산하는 과정.
    
    - Q(cat)K(the) : “cat”과 “the”의 관련성
    - Q(cat)K(eat) : “cat”과 “eat”의 관련성
    - Q(cat)K(meat) : “cat”과 “meat”의 관련성
    
    > QK를 $\sqrt{d_k}$ 로 나눠 scaling. 이후 softmax를 거치면 각 raw별로 0~1사이로 scaling되며 각 raw는 해당 query가 다른 단어들과 얼마나 관련이 있는지를 나타내는 확률 분포를 의미함.
    > 
4. V를 가중합하여 결과 생성
    
    앞서 계산된 $softmax(\frac{QK^T}{\sqrt{d_k}})$에 V를 곱해 나온 output은 관련성이 높은 단어의 정보가 반영된다. 
    
    예를들어 “cat”과 “meat”의 연관성이 높다면 V(meat)의 값이 더 많이 반영된다.
    

### MHA(Multi Head Attention)

아래는 Multi Head Attention의 구조.

<img src="/Transformer_mha.png">

아래 식에서 볼 수 있듯이, MHA는 self-Attention이 전체 input에 대해 병렬적으로 여러 번 일어난다. 

하나의 head는 1번의 self-Attention을 의미하고 이 head를 여러개 병렬로 묶은 것이 MHA.

<img src="/Transformer_mha_2.png">

head를 여러 개를 병렬로 계산하고 합침으로써, 단어 간 다양한 관계를 학습할 수 있음.

예를들어 하나의 head는 인접한 단어간 관계를 계산하고, 다른 head는 문장 전역적인 의미에 집중하는 등, 독립적인 처리가 가능.

## Encoder

Encoder는 MHA-Norm-FeedForward-Norm의 과정을 거쳐 x→z 생성.

MHA 전후로, Residual Connection 존재.

위 구조가 N개가 병렬로 묶여 있는 구조.

## Decoder

Decoder의 input으로는 shifted target sequence가 들어간다. shifted target sequence는 예측하려는 단어 바로 앞까지의 단어들로 이루어진 sequence로 예를들어 “I am a student”를 예측하려 할 때, “I am a”까지를 받아 “student”를 예측함.

Decoder의 두 번째 attention sublayer에서 이전 Encoder의 output이 결합됨.

두번째 MHA에서 Decoder의 input으로 들어와 나온 벡터가 Q로 사용되고,

Encoder의 output이 K, V로 사용된다.

마찬가지 Decoder도 N개가 병렬로 묶여 있는 구조를 가진다.

## Transformer가 병렬처리가 가능한 이유

1. Self Attention을 통해 모든 단어 간 관계를 한 번에 계산.
2. Positional Encoding을 통해 순서를 유지하면서 병렬처리가 가능해짐.