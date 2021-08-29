# Attention is all you need

* 본 논문 링크: https://papers.nips.cc/paper/2017/file/3f5ee243547dee91fbd053c1c4a845aa-Paper.pdf

## Abstract

이때까지 Recurrent model은 순차적으로 처리하는 특징으로 여러 작업을 동시에 수행하지 못 했다!

attention을 통한 encoder와 decoder를 연결하는 모델이 최고의 모델이다!! (이 논문이 2017년에 나왔는데 이때는 맞는 듯 하다!)

이 논문에서는 앞서 말한 문제가 있던 Recurrence를 제거하고 attention mechanism을 이용한 network 구조를 가진 **Transformer**를 제안하려고 한다!

이러한 mechanism은 **병렬 처리 기능**과 **학습 시간이 덜 소요**되므로 성능이 좋다!!

## Introduction

이러한 RNN, LSTM, GRU 등의 언어 모델링은 sequence 모델링으로 순차적 계산의 근본적인 제약이 여전히 있다!

또한, 입출력 사이의 전역 의존성을 이끌어 낼 필요도 있다. 

    * 여기서 입출력 사이의 전역 의존성은 입출력 사이의 거리에 영향없이 무관하게 학습이 가능하다는 말을 뜻한다.

이러한 문제를 attention mechanism을 가진 transforemr를 통해 recurrence를 제거하고 전역 의존성을 이끌어 내려고 한다!

그리고 transformer는 상당한 병렬화와 높은 성능을 자랑한다!

## Model Architecture

모델 구조는 간단히 말하자면 **stacked self-attention**과 **pointwise fully connected layer** 구조를 가진다.

### encoder and Decoder Stacks

![attention구조](https://user-images.githubusercontent.com/59636424/131236887-85f885bf-19e4-4184-878b-35d6b5fb9367.PNG)

: 이 구조는 Attention만으로 encoder와 decoder를 가지는 구조이다!

![encoder decoder](https://user-images.githubusercontent.com/59636424/131237922-1e1f2555-8bbe-4133-a488-76802f0bc4e7.PNG)

: 논문에서는 encoder와 decoder는 각각 6개의 층으로 구성되어있다!

---

### Encoder

input sequence (x_1,....,x_n)을 z=(z_1,....,z_n)으로 매핑하는 역할을 한다!

우선, Encoder는 같은 6개의 층으로 **multihead self-attention layer**와 **feed-forward layer**를 사용한다.

그리고 self-attention과 feed forward가 끝날 때마다, **residual connection**과 **layer normalization**을 진행한다.


