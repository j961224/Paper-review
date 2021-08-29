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

### 그러면 여기서 말하는 self-attention은 무엇일까?!!

우선, 간단히 말하면 Query weight matrix, Key weight matrix, Value weight matrix와 Query, Key, Value를 가지고 계산하여 value vector들의 weight를 구하는 것이 목표이다!

~~~
논문에서는 512차원으로 임베딩하여 유지하는데 이 수치는 하이퍼 파라미터이므로 알아서 더 해도 되는데 내 생각은 512차원으로 단어들을 커버 가능하고 그 이상으로 할 경우, 계산적 효율이 매우 안 좋다고 생각합니다.. -> 이 부분에 대해서 좋은 의견있으면 적극 수용하겠습니다!
~~~

![1](https://user-images.githubusercontent.com/59636424/131238532-5884c017-c2cb-4cec-9c2f-fdf4da6b5f23.png)

단어마다 512차원으로 임베딩된 입력 시퀀스가 있을 때, 각 단어의 Query, Key, Value를 구한다!

~~~
이때, Query, Key, Value 벡터는 무엇을 의미하는가?!

: attention에 대해 계산하려 할 때 도움이 되는 추상적인 개념으로, Query vector는 현재 단어(영향을 받는 단어 변수), Key vector는 점수를 매기려는 다른 위치에 있는 단어(영향을 주는 변수), Value vector는 입력의 각 단어(그 영향에 대한 가중치)들이다!
~~~

이러한 Query와 Key와 Value는 **weight matrix와 입력 시퀀스를 곱해서** 만들어진다!

![2](https://user-images.githubusercontent.com/59636424/131239123-9578926e-dfd7-4109-b04e-93ef8b55ebfc.png)

앞서, 구한 Query와 Key를 dot product하여 attention score를 구한다.

예를 들면 위의 사진을 보면, 단어 'I'가 Key와 dot-product를 하면, **나머지 단어와 얼마나 연관이 있냐에 따라 연관성이 깊다면 score가 크게** 나올 것이다. (한마디로, 영향을 받는 단어와 주는 단어들의 유사도 측정 과정)

![score](https://user-images.githubusercontent.com/59636424/131239246-d2661d24-3d0d-4985-a266-6252c8b6ec88.png)

이렇게 구한 attention score에 **softmax로 0~1사이의 확률값으로 표현**하려 한다!

~~~
논문에서 attention score에 softmax함수를 사용하기 전에 루트(Key 차원값)으로 나누는데 그 이유는?!

그렇게 하는 이유는 dot-product가 크다면, softmax 기울기가 작아지는 현상이 발생하여 나눈다!!(연산 결과 커지는 것을 방지!)
~~~

또한 그 값을 Value vector(입력에 대한 가중치)를 곱해 해당 단어의 임베딩이 갱신된다! (각 단어와의 연관성이 고려된다!)

* **이렇게 구한 attention score의 식**

![식](https://user-images.githubusercontent.com/59636424/131239678-697d030e-7cd8-4b3f-82a4-da292eac2102.PNG)

-> 최종적으로 이렇게 **한 단어씩 구한 value weight sum이 구해준 것을 z-vector로 표현**한다!!

-> 이 말이 즉, **input sequence (x_1,....,x_n)을 z=(z_1,....,z_n)으로 매핑하는 역할**을 뜻한다.

* 전반적으로 진행되는 self-attetion 과정

![과정](https://user-images.githubusercontent.com/59636424/131239680-7d587a9e-23e9-4ac9-9e74-7d7ded66475e.PNG)

* **또한 논문에서 Scaled Dot-Product Attention 과정이다.**

![ㄴㅇㄹㅇㄹㅇㄹ](https://user-images.githubusercontent.com/59636424/131239759-015194cf-a5b4-4857-8274-787a28e14261.PNG)

~~~
self-attention 시, 주의할 점!

* Query vector와 Key vector는 내적하므로 차원이 같아야 한다! => value vector는 달라다 된다!

* value vector 차원은 encoding vector 차원가 같아야 된다!
~~~

### 그렇다면 논문에서 쓰이는 Multi-head attention은 무엇인가?!

self-attention layer를 다중으로 구현한 것이 Multi-head attention이다!

Transformer는 **self-attention의 head를 8개로 병렬적으로 attention output을 구하는 방식**을 채택했다!

이렇게 구한 8개의 attention output을 concatenate를 한다!

![ㅋㅋ](https://user-images.githubusercontent.com/59636424/131240550-8d3be7ac-1312-4315-9b7e-101afac963a9.png)

~~~
이렇게 self-attention말고 multi-head attention을 하는 이유는?

다른 포지션에 attention하는 모델의 능력을 향상시키기 위함이다. self-attention은 다른 단어 뿐만 아니라 자신의 단어에 더 많은 영향을 받는 것을 볼 수 있기 때문이다.

또한, Attention을 가지는 head는 무작위로 Query, Key, Value가 초기화 되므로, 다른 표현 subspace에 임베딩되므로 좀 더 일반적? 이므로 사용한다.
~~~

그런데, **Feed Forward layer에 8개의 matrix 처리가 불가하므로 weight matrix W0으로 변환시켜준다!**

(이 때, output은 입출력 차원과 맞추기 위해서 weight matrix W0 dimension을 그에 맞게 설정해준다.)

![jrur](https://user-images.githubusercontent.com/59636424/131241465-cf64f4f4-d140-48bf-9bc5-9f402806adbe.png)

