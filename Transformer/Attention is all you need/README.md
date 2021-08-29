# Attention is all you need

* 본 논문 링크: https://papers.nips.cc/paper/2017/file/3f5ee243547dee91fbd053c1c4a845aa-Paper.pdf

## 1. Abstract

이때까지 Recurrent model은 순차적으로 처리하는 특징으로 여러 작업을 동시에 수행하지 못 했다!

attention을 통한 encoder와 decoder를 연결하는 모델이 최고의 모델이다!! (이 논문이 2017년에 나왔는데 이때는 맞는 듯 하다!)

이 논문에서는 앞서 말한 문제가 있던 Recurrence를 제거하고 attention mechanism을 이용한 network 구조를 가진 **Transformer**를 제안하려고 한다!

이러한 mechanism은 **병렬 처리 기능**과 **학습 시간이 덜 소요**되므로 성능이 좋다!!

## 2. Introduction

이러한 RNN, LSTM, GRU 등의 언어 모델링은 sequence 모델링으로 순차적 계산의 근본적인 제약이 여전히 있다!

또한, 입출력 사이의 전역 의존성을 이끌어 낼 필요도 있다. 

    * 여기서 입출력 사이의 전역 의존성은 입출력 사이의 거리에 영향없이 무관하게 학습이 가능하다는 말을 뜻한다.

이러한 문제를 attention mechanism을 가진 transforemr를 통해 recurrence를 제거하고 전역 의존성을 이끌어 내려고 한다!

그리고 transformer는 상당한 병렬화와 높은 성능을 자랑한다!

## 3. Model Architecture

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

: attention에 대해 계산하려 할 때 도움이 되는 추상적인 개념으로, Query vector는 현재 단어(영향을 받는 단어 변수/질문), Key vector는 점수를 매기려는 다른 위치에 있는 단어(영향을 주는 변수/물어보는 단어), Value vector는 입력의 각 단어(그 영향에 대한 가중치/질문에 대한 답)들이다!
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

* Query vector와 Key vector는 내적하므로 차원이 같아야 한다! => value vector는 달라도 된다!

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

이러한 과정을 수식으로 표현하면 아래와 같다.

![mha](https://user-images.githubusercontent.com/59636424/131241939-519fce1f-0bec-493e-9730-d5c52eac09c3.PNG)

각각 head마다 self-attention을 하여 concatenate하고 weight matrix W0가 곱해 구함을 볼 수 있다.

아래의 수식은 위의 수식을 좀 더 자세히 표현한 수식이다!

![ddddd](https://user-images.githubusercontent.com/59636424/131242362-43e41492-a035-40bd-a3bb-6951dea7385c.PNG)

~~~
이 때, d_Q, d_K, d_V는 각각 query와 key, value 개수를 의미한다!
~~~

논문에서는 d_k=d_v=d_model/h=64이므로 **이렇게 각 head들의 줄인 차원으로 총 연산량은 전체 차원의 1개 single-head attention과 유사하다!**

* 아직 Attention(~)=[d_V x d_v]가 이해가 안 간다면 아래의 사진을 보고 이해할 수 있을 것이다!

![at](https://user-images.githubusercontent.com/59636424/131242477-f4a4277a-e3af-48f4-95a3-9813e1897aac.PNG)


### Feed Forward Neural Network

![ㅋㅌㅊㅋㅌㅊ](https://user-images.githubusercontent.com/59636424/131241905-30119f57-1d79-4c81-9108-c16841a38839.PNG)

Feed Forward(fully connected feed forward layer)는 Encoder와 Decoder에 각각 layer에 존재한다!

이러한 수식이 나오는 이유는 먼저, linear layer를 먼저 두고 그 다음에 ReLU activation, 마지막으로 linear layer를 둠으로 이러한 수식이 표현 가능하다!

(여기서 중간 차원은 2048, 입력과 출력 차원은 512(d_model dimension)이다!)

### Positional Encoding

**단어들의 순서에 대해 고려하지 않고 있으므로 각 단어들의 순서에 대한 정보를 넣기 위해서** 사용된다!

![psotit](https://user-images.githubusercontent.com/59636424/131243054-e4a812b2-9d23-4aca-bc4f-91b16160a843.PNG)

각각의 임베딩된 단어에 positional encoding을 더하면 **모델의 각 단어의 위치**와 **시퀀스 내의 다른 단어 간의 위치 차이**에 대한 정보를 알 수 있다!

=> 추가적으로, 이렇게 positional encoding을 더한 값은 attention하면서 Query/Key/Value 벡터들로 dot-product되면 단어간의 거리를 늘릴 수 있다!!

이러한 poisitional encoding은 각각의 임베딩된 단어에 더하기 전에 **sin과 cos 함수를 이용하여 -1~1 사이의 값으로 mapping** 시킨다!

![aaa](https://user-images.githubusercontent.com/59636424/131243420-2d2ce4bb-438b-414e-8dab-fd126415bdca.PNG)

~~~
내 생각이지만 원래 positional encoding은 Byte를 이용해 000,010 등으로 표현하는데 sin, cos을 통해 실수로 좀 더 풍부하게 표현하기 위함일까라는 생각을 해본다...
~~~

### residual connection & layer normalization

![qqqqq](https://user-images.githubusercontent.com/59636424/131243886-5baa022b-ff0a-403a-bc3e-ed36a4ca58db.PNG)

각 sublayer 마다 residual connection과 그 이후에 바로 layer normalizaion이 존재한다.

-> 보통 residual connection을 사용하는 이유는 인공신경망이 깊어질수록 **기울기 소실문제를 막기** 위해서 사용한다!

그래서 residual connection수행을 위해 더해야 되기 때문에 입출력 차원을 맞추는 이유 중 하나이기도 하다!

LayerNorm(x+sublayer(x))로 표현할 수 있다.


### Decoder

**이전 timestep에서 생성한 결과를 입력으로 현재 timestep의 결과 토큰 생성**

Decoder 또한 6개의 layer를 가지고 있다!

Decoder는 Masked Multi-head Attention, Multi-head Attention, Feed Forward layer로 구성되어있다!!

![wqqq](https://user-images.githubusercontent.com/59636424/131244246-e192c97a-287d-4584-bd52-88a2b9692626.PNG)

위의 사진은 전반적인 흐름인데 encoder와 decoder의 흐름이 유사함을 볼 수 있다.

---

![deocder](https://user-images.githubusercontent.com/59636424/131244311-15760a4b-92d3-4af9-9d82-15bdd9742a45.PNG)

Decoder의 좀 더 자세한 흐름은 Masked Multi-head Attention -> add & Normalization -> Multi head Attention -> add & Normalization -> Feed Forward -> add & Normalization식으로 흘러간다! (이것을 계속 반복!)

* Masked Multi-head attention layer는 첫번째 multi-head attention layer로 사용된다.

~~~
이것이 왜 Masked Multi-head attention인 이유는?!

현재 시점이 t라고 하면 output을 생성하는데 attention을 얻을 경우에 t번째의 이후의 값들을 참고하지 않겠다는 말이다!
(지금까지 출력된 단어에 대해서만 attention을 적용!) -> t 이후에 position에 attention을 주지 않으면 t 이후의 값도 미리 알고 있게 되므로 의존하게 되므로!
~~~

이렇게 현재 decoder의 입력값을 받고 encoder의 최종값인 Key와 Vale로 **Masked Multi head attention을 통해 나온 것은 Query로 사용**한다!

~~~
왜 Masked Multi head attention에서 Query가 output이 되는가?

우선, Query는 지금 decoder에서 이런 값이 나왔는데 무엇이 output이 돼야 할까?라는 질문이 될 수 있다.

그래서 이러한 Query가 t 시점 이후의 정보는 masking 했으므로 t번째 위치까지만의 attention을 얻게 되므로 딱 Query가 적합하다!
~~~

**encoder의 최종값인 Key와 Value와 첫번째 multi-head attention layer의 결과인 Query도 받아 2번째 multi-head attention layer에서 decoder의 다음 단어에 적합한 단어를 찾는다!**

* 아래는 decoder가 다음 단어를 출력하는 과정이다.

![wown](https://user-images.githubusercontent.com/59636424/131245596-5da8e559-9a8a-4d2b-9b58-22749985422c.gif)

![qqqqqqq](https://user-images.githubusercontent.com/59636424/131250627-243f86de-9ccc-4556-b499-99821b0768e4.gif)


**이외에는 encoder와 유사하다!**

### 마지막 linear and Softmax layer

**이제 여러 개의 decoder를 거치고 난 후, 소수로 이뤄진 벡터 1개가 남는데 이것을 단어로 바꿔주는 역할을 한다!**

* linear layer는 벡터를 softmax에 들어갈 logit 변경 시켜준다!(fully-connected layer로 training data가 1000개 영어 단어를 학습했다면 logits vector를 1000의 크기로 만든다.)

* softmax layer는 모든 단어에 대한 확률값으로 만들고 가장 큰 출력값을 가진 단어가 출력된다!

![z](https://user-images.githubusercontent.com/59636424/131245880-8b9e4a6c-2931-4461-a3e9-13d2971cd5d0.PNG)


### Application of Attention in our Model

#### 1. encoder self-attention layer

![zc](https://user-images.githubusercontent.com/59636424/131245916-a465e039-a506-4abb-9b4a-6794518749ca.PNG)

앞서, 모두 말했지만 이 self-attention layer는 이전 layer의 output에서 모두 Key, Value, Query를 가져와 사용한다.

(예를 들면 첫번째 layer는 input 값을 받으니 그 전에 position encoding과 embedding한 input embedding이 될 것이다.)

#### 2. decoder self-attention layer

![wwd](https://user-images.githubusercontent.com/59636424/131246032-1f9fbca7-3d14-4caf-b7e9-c0e534b45879.PNG)

encoder와 유사하다!

그러나 미래에 오는 값들에 대한 attention은 수행이 불가능하다!! (현재 t 시점 이후 값들은 masking out을 통해 접근이 불가능하게 만든다!)

#### 3. encoder-decoder attention layer

![qwqw](https://user-images.githubusercontent.com/59636424/131246059-85ccaf27-fabc-4773-9db8-44492d3e67f8.PNG)

이전 decoder layer에서 만든 Query와 encoder output에서 오는 Key, Value들로 encoder output의 모든 position에 attention을 줄 수 있다.

## 4. Why self-Attention

1. 각 층에서 발생하는 연산 복잡도 감소!(레이어당 전체 연산량이 줄어든다!)

~~~
입력의 길이가 d_model(512차원)의 크기보다 입력의 길이가 작을 경우에 해당하지만 대부분 데이터셋을 구성하는 문장들은 d_model(512차원)보다 짧다!
~~~

2. 병렬적 계산이 가능하다.

~~~
병렬적 계산으로 8 head로 8번의 self-attention을 수행하지만 전체 layer의 1번 self-attention의 계산량과 같다!
~~~

3. Long-term dependency도 잘 학습한다.

~~~
멀리 떨어진 원소들 사이의 거리가 상수배로 매우 작으므로 이러한 상수배로 멀리 떨어진 원소들을 학습한다. (position encoding과 softmax를 사용했기 때문에)
~~~

![qqqq](https://user-images.githubusercontent.com/59636424/131246434-a5b1005c-be9a-4279-bb51-9d1e45cb77fb.PNG)

위의 사진은 head가 늘어날수록 다양한 가중치를 가짐을 볼 수 있다.

![aaa](https://user-images.githubusercontent.com/59636424/131246462-6597dcce-f53d-4109-9cc6-cc99ecb9d390.PNG)

위의 표는 self-attention이 다른 것들과 비교했을 때,모든 면이 뛰어남을 알 수 있다. (n이 d보다 훨씬 작다!)

restricted는 시퀀스 길이 n이 클 때, r크기의 주변만 고려할 때 사용한다.

## 5. Training

### 5-1. Training Data and Batching

논문에서는 WMT 2014 English-German dataset 450만 짝지어진 문장 데이터를 사용했다.

### 5-2. Optimizer

Adam optimizer을 사용했고 learning rate를 training 동안 고정시키지 않고 변화시켰다.

=> 이유는 처음에는 학습이 잘 되지 않은 상태이므로 learning rate를 빠르게 증가시켜 변화주다가 학습이 꽤 됐을 시점에 learning rate를 천천히 감소시켜 변화를 작게 주기 위해서이다!

### 5-3. Regularization

### Residual Dropout

sublayer output을 정규화와 더하기 전에 적용하고 embedding 합을 구할 때, dropout 비율을 0.1 적용했다.


## 6. Conclusion

transformer는 recurrence를 이용하지 않고 빠르고 정확하게 sequential data를 처리할 수 있는 model로 제시

가장 핵심적인 것은 encoder와 decoder에서 attention을 통해 Query와 가장 밀접한 연관성을 가진 Value를 강조할 수 있고 병렬화가 가능해졌다.
