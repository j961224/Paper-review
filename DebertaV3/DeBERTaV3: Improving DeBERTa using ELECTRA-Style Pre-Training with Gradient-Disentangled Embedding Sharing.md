# [DeBERTaV3: Improving DeBERTa using ELECTRA-Style Pre-Training with Gradient-Disentangled Embedding Sharing](https://arxiv.org/abs/2111.09543)


## 0. Abstract

* DebertaV3는 MLM에서 RTD(Replaced Token Detection)로 변경
* Electra는 discriminator와 generator의 train loss가 서로 다른 방향으로 token embedding을 끌어당긴다. -> 단점으로 이를 "tug of war"라고 함
* **gradient disentangled embedding sharing method** 사용
* Deberta와 같은 세팅으로 DebertaV3 훈련

---

## 1. Introduction

* PLM 구축 시에는 더 적은 파라미터와 더 적은 계산 비용이 중요하다!
* Deberta의 핵심 기술은 relative position encoding mechanism인 **disentangled attention**을 사용함으로 효율적으로 pretrain 진행
* 효율성을 위한 pretrain 방식으로 electra에서 나온 **RTD 사용**

* RTD: Generator는 애매한 단어 token 생성 -> discriminator는 애매한 token을 구별

* 논문에서 효율성 증가시킨 2가지 방법
> * 1. **disetangled attention**
> * 2. **RTD**

* 새로운 임베딩 sharing 방법 제시

* Electra의 훈련 단점

discriminator와 generator는 같은 token embedding을 공유하면, discriminator와 generator의 훈련 objective가 매우 다르므로 train loss가 서로 다른 방향으로 당겨지니 훈련 효율성이 떨어진다.

* discriminator의 RTD는 binary classification 정확도를 최적화 하기 위해 유사한 토큰을 구별하고 embedding을 최대한 멀리 끌어당기기 -> tug-of-war 발생

* **gradient disentangled embedding sharing(GDES)**: discriminator의 gradient가 generator embedding으로 back propagation 하는 것을 방지

---

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

---

## 3. DeBERTaV3

* **RTD train loss + new weight sharing method**

### 3.1 DeBERTa with RTD

* Train data: Wikipedia and bookcorpus
* Batch size: 2048
* Train step: 125000
* learning rate: 5e-4
* warmup step: 10000

* Embedding Sharing 방법 3가지 제안
> * 1. **token Embedding Sharing(ES)**
> * 2. **Gradient-Disentangled Embedding Sharing(GDES)**
> * 3. **No Embedding Sharing(NES)**


### 3.2 Token Embedding Sharing in ELECTRA

  ![3가지](https://user-images.githubusercontent.com/59636424/183005054-9e65733c-b1a2-40c5-91ec-f269c09fe4bf.PNG)
  
**NES는 ES보다 수렴 속도가 더 빠름**

**parameter efficiency -> generator embedding들은 더 좋은 discriminator를 생산하는데 더 유익하다.**

#### ES

* E: token Embedding의 parameters
* g_E: token Embedding의 gradient

* generator의 MLM loss와 discriminator의 RTD loss를 가지고 back propagation 진행

![adadadadad](https://user-images.githubusercontent.com/59636424/183004264-2f13daf3-606b-4fe0-8a81-70b2a21076ed.PNG)

* 이와 같은 경우, generator와 discriminator 작업이 매우 방향이 달라 훈련이 비효율적 -> tug-of-war

* MLM와 RTD의 차이
> * MLM: 서로 가까운 embedding vector와 유사한 token들을 map시켜준다.
> * RTD: classification 정확도를 최적화하기 위해 embedding을 가능한 멀리 당겨서 의미적으로 유사한 토큰 구별

#### NES

**generator 따로 discriminator 따로 forward pass를 거쳐서 업데이트한다.**

### 3.3 Gradient-Disentangled Embedding Sharing

* ES의 장점을 그대로 보유
* **training 시에 sharing은 제한**
* **generator의 gradients**: MLM loss를 기반으로 계산 -> not RTD loss
* **GDES는 단방향으로 gradient 계산 -> MLM은 E_G와 E_D 둘다 사용**
* **RTD는 E_D만 업데이트에 사용**

* GDES는 discriminator의 token embedding들을 re-parameterize

- sg: E를 통해 pgradient propagation을 허락하는 stop gradient operation

![ㅇㅁ](https://user-images.githubusercontent.com/59636424/183007278-6dec2394-4dd6-43b3-b608-3bcc8615d77e.PNG)

* E는 zero matrix로 초기화

#### 진행 과정

* generator forward pass -> backpropa(MLM loss -> E_G(generator와 discriminator 둘다 사용)) -> discriminator forward pass -> backward pass(E_D 업데이트를 위한 Re를 통해서 RTD loss에 관한)

---

## 4. Experiment

### 4.1 Main Results on NLU tasks

* deberta code 수정해서 train 시도
* DeBERTaV3의 generator는 discriminator와 같은 width / discriminator의 half depth

* train 조건
> * 데이터: 160GB data
> * vocab: 128000 token을 포함하는 SentencePiece vocabulary
> * train step: 500000
> * batch size: 8192
> * warmup step: 10000
> * learning rate: 5e-4(small model) / 3e-4(large model)
> * optimizer: AdamW

#### 4.1.1 Performance on Large Models

* Performance

![ㅇㅇㅇㅇㅇㅇㅇ](https://user-images.githubusercontent.com/59636424/183008890-99d45339-903e-4a10-8d59-4d5afa4b7075.PNG)

* Model size

![냨ㄷ](https://user-images.githubusercontent.com/59636424/183009023-9e8bd6f3-27ee-4be3-bf32-64132f480a36.PNG)

---

## 5. Conclusions

* DeBERTa에 ELECTRA를 결합하여 성능 jump 시킴
* gradient-disentangled embedding sharing으로 tug-of-war을 피해서 사전학습 효율성을 증대
* NLU task에 SOTA 달성


