# [DeBERTaV3: Improving DeBERTa using ELECTRA-Style Pre-Training with Gradient-Disentangled Embedding Sharing](https://arxiv.org/abs/2111.09543)


## 0. Abstract

* DebertaV3는 MLM에서 RTD(Replaced Token Detection)로 변경
* Electra는 discriminator와 generator의 train loss가 서로 다른 방향으로 token embedding을 끌어당긴다. -> 단점으로 이를 "tug of war"라고 함
* **gradient disentangled embedding sharing method** 사용
* Deberta와 같은 세팅으로 DebertaV3 훈련


## 1. Introduction

* PLM 구축 시에는 더 적은 파라미터와 더 적은 계산 비용이 중요하다!
* Deberta의 핵심 기술은 relative position encoding mechanism인 **disentangled attention**을 사용함으로 효율적으로 pretrain 진행
* 효율성을 위한 pretrain 방식으로 electra에서 나온 **RTD 사용**

* RTD: Generator는 애매한 단어 token 생성 -> discriminator는 애매한 token을 구별

* 논문에서 효율성 증가시킨 2가지 방법
> * 1. disetangled attention
> * 2. RTD

* 새로운 임베딩 sharing 방법 제시

* Electra의 훈련 단점

discriminator와 generator는 같은 token embedding을 공유하면, discriminator와 generator의 훈련 objective가 매우 다르므로 train loss가 서로 다른 방향으로 당겨지니 훈련 효율성이 떨어진다.

* discriminator의 RTD는 binary classification 정확도를 최적화 하기 위해 유사한 토큰을 구별하고 embedding을 최대한 멀리 끌어당기기 -> tug-of-war 발생

* **gradient disentangled embedding sharing(GDES)**: discriminator의 gradient가 generator embedding으로 back propagation 하는 것을 방지

## 2. Background

### 2.1 Transformer

* 기존 Transformer 방식: 각 input word embedding에 position bias 추가
* position bias: absolute position embedding or relative position embedding

### 2.2 DeBERTa

* disentangled attention + enhanced mask decoder 사용
* disentangled attention은 two separate vector 사용(content vector and psotiion vector)
* disentangled attention은 content와 relative position 모두 disentangled 행렬로 계산
* **disentangled attention은 context word들의 content와 relative positions을 고려하지만, absolute position은 고려하지 않음**
* **enhanced mask decoder에서 decoder layer에 context word의 absolute position 정보를 추가해 MLM 개선**

### 2.3 ELECTRA

#### 2.3.1 Masked Language Model

* token의 15%를 X(물결)로 마스킹 -> X에 조건화된 masked token X(물결)를 예측하여 X를 재구성하기 위해 매개 변수화된 언어 모델 사용

![캡처](https://user-images.githubusercontent.com/59636424/182983949-897f9d8e-d331-43ff-b0e3-0b60bbf54521.PNG)

#### 2.3.2 Replaced token detection

* generator: MLM으로 학습 / discriminator: token-level 이진 분류기로 학습

* discriminator의 training objective = RTD
* Generator loss function

![MLM](https://user-images.githubusercontent.com/59636424/182984779-ac98e863-3ada-4bdf-a6c2-dcb705ed42a3.PNG)

X(물결) _ G: X에 15% masking함으로 generator의 input

* Generator로부터 확률 분포

![wdwd](https://user-images.githubusercontent.com/59636424/182985025-d88a57de-25f0-4e7e-b510-0dbfb373a2ca.PNG)

* discriminator의 loss function

![rtrtrt](https://user-images.githubusercontent.com/59636424/182985089-d438d4b3-1504-4a25-9552-66caf713e692.PNG)

l(.): indicator function(어떤 원소가 어떤 집합에 표함되는지 아닌지를 표시)

* **L = L_MLM + 람다 * L_RTD** -> MLM loss와 RTD loss를 함께 최적화 시킨다. (람다: 50)

## 3. DeBERTaV3










