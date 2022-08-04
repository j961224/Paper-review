# [A DEEP REINFORCED MODEL FOR ABSTRACTIVE SUMMARIZATION](https://arxiv.org/pdf/1705.04304.pdf)

## 0. Abstract

* input과 연속적으로 생성된 output에 attend하는 **새로운 intra-attention과 표준 supervised 단어 예측과 강화 학습을 결합**한 새로운 훈련 방법을 소개
* **표준 단어 예측은 강화 학습의 global sequence prediction이 결합**되면 결과 요약이 더 쉽다.

## 1. Introduction

* 반복 구문 문제(repeating phrase problem)을 해결하기 위해서 새로운 learning objective 소개

> * 각 input token의 이전 attention weight를 기록한 encoder에 intra-temporal attention과 이미 생성된 단어에 신경을 쓰는 decoder에 intra-attention model
> * 새로운 objective function → **maximum likelihood cross entropy loss + policy gradient RL(exposure bias 줄이기 위해서)**

* exposure bias란?

추론 단계에서 중간 sequence의 잘못된 예측으로 뒤의 sequence까지 영향을 받을 수 있는 문제 → sequence 생성 모델에 대하여 근본적으로 해결해야 하는 문제

## 2. NEURAL INTRA-ATTENTION MODEL

* input token sequence

![input](https://user-images.githubusercontent.com/59636424/182806052-a5f732e5-c02e-42e8-89b0-8731718cddc8.PNG)

* output token sequence

![output](https://user-images.githubusercontent.com/59636424/182806118-c6561195-f3ea-4ef5-a834-0ae7480f5daf.PNG)


### 2.1 INTRA-TEMPORAL ATTENTION ON INPUT SEQUENCE

* 각 decoding step t에서, decoder hidden state와 이전에 생성된 단어 외에도 encoded input sequence의 특정 부분을 관리하기 위해 intra-temporal attention 기능을 사용한다.
* past decoding step에 높은 attention score에서 얻은 input tokens 불이익(패널티)를 주는 방법으로 attention weights를 정규화 시켜준다.

![dsds](https://user-images.githubusercontent.com/59636424/182806380-5b7eec7a-52f4-4bc8-a4c3-26563c0d9adf.PNG)

(e_ti: decoding time step t에 hidden input state h_i의 attention score)


### 2.2 INTRA-DECODER ATTENTION

* decoder는 반복적인 구문(repeated phrases)을 여전히 생성 → 이전에 decoded sequence에 대해 데 많은 정보를 decoder에 통합 → **intra decoder attention mechanism 도입**

### 2.3 TOKEN GENERATION AND POINTER

* token generation softmax layer or pointer mechanism 사용 (입력 시퀀스에서 보이지 않는 것을 copy) → decoding 단계에서 토큰 생성할지, 포인터를 사용할지 결정하는 스위치 기능 사용

* token generation softmax layer

![121](https://user-images.githubusercontent.com/59636424/182806773-a214eaf7-e747-4bb9-b111-e25144ab427f.PNG)

* pointer mechanism → temporal attention weights

![122](https://user-images.githubusercontent.com/59636424/182806797-46f3be34-3464-4d4d-bdef-aae8dc0128e9.PNG)

* output token y_t로 final 확률분포를 얻는다. -> 위의 token generation softmax layer와 pointer mechanism 둘 다 사용

![123](https://user-images.githubusercontent.com/59636424/182806844-f9402d86-9b0e-43ae-a1ca-cd2e2e236dea.PNG)

## 3. Hybrid Learning Objective

### 3.1 Supervised learning with teacher forcing

* input sequence x에 ground-truth output sequence

![ㄴㅇ](https://user-images.githubusercontent.com/59636424/182807105-27e3a889-e9d1-4109-83bf-9b4674bc3a52.PNG)

* 기존에는 아래 loss를 최소화 하여 maximum likelihood training objective를 구함

![3 1](https://user-images.githubusercontent.com/59636424/182807209-3abfdcd6-0c33-4a3a-8fdb-08205fa2e92c.PNG)

: 이는 보통 이렇게 학습하지만, Rouge와 같은 discrete한 metric에도 좋다고 말할 수는 없음

### 3.2 Policy learning

* y^s: 매 decoding step마다 확률 분포에서 sampling해서 얻은 sequence 출력

![dddddd](https://user-images.githubusercontent.com/59636424/182807408-3b5a0a67-099b-4e05-8fc7-edf6d48b3cf2.PNG)

* y_hat: greedy search로 얻은 baseline output
* r(y): squence y에 대한 reward function
* discrete metric을 maximize하는 policy를 학습

![policy learning](https://user-images.githubusercontent.com/59636424/182807502-3134146c-c6df-4a30-a246-315b211756fa.PNG)

**L_rl을 최소화 → y^s가 y^보다 higher reward를 얻었을 때, conditional likelihood를 높여 모델이 더 higher reward를 얻도록 한다.**

### 3.3 MIXED TRAINING OBJECTIVE FUNCTION

* 기존 supervised learning과 policy learning 혼합된 방법

![ddda](https://user-images.githubusercontent.com/59636424/182807603-2bd8b14c-2f56-4e6a-8696-00483bf5b0c8.PNG)

: 논문에서는 알파를 0.9984로 잡고 실험!










