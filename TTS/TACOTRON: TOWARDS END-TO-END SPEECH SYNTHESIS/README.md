# TACOTRON: TOWARDS END-TO-END SPEECH SYNTHESIS 논문 리뷰!

**2017년 구글에서 나온 방법으로 이 기술 이후에 TACOTRON2 등이 나왔다.**

## 0. Abstract

- TTS(텍스트 음성 변환)합성 시스템은 여러 단계로 이루어져 있다.

    텍스트 분석 프론트엔드, 음향 모델 및 오디오 합성 모듈
    
    이러한 구성 요소를 구축하려면 종종 광범위한 도메인 전문 지식이 필요하다!
    
- 이 논문에서는 문자에서 직접 음성을 합성하는 end-to-end 생성 텍스트 음성 변환 모델인 **Tacotron**을 설명하려한다.

- 논문에서 sequence to sequence framework가 어려운 일에 잘 작동되기 위해 몇몇 중요한 기술을 설명하려한다.

- Tacotron은 US English set에서 생산 매개변수 시스템(1세대 2013년 기술)을 능가하는 **mean opinion score(음성 품질 측정)에서 5점 만점 중, 3.82점이라는 성과**를 올렸다.

- Tacotron은 **프레임 레벨에서 음성을 생성하기**에 이전 sample 레벨에서의 자기회귀 모델보다 더 빠르다!

## 1. INTRODUCTION

- 현대 TTS 파이프라인은 복잡하다!


예시로 [통계학적 parametric TTS](https://github.com/j961224/Paper-review/blob/main/TTS/TACOTRON:%20TOWARDS%20END-TO-END%20SPEECH%20SYNTHESIS/%ED%86%B5%EA%B3%84%ED%95%99%EC%A0%81%20parametric%20TTS.md)를 들자!

    1. 다양한 언어학적 특성들을 추출하는 text frontend
    
    2. 음성의 길이를 예측하는 모델
    
    3. 음섣 특성을 예측하는 모델
    
    4. 음성 신호 합성을 위한 vocoder(spectrogram(음성을 숫자로 표현하는 여러가지 방법 중 하나)을 소리로 변환)
    
    이렇게 4가지로 이루어져 있다.

![tts](https://user-images.githubusercontent.com/59636424/132998191-995df7b7-73ff-470c-8825-99a0a700b0ff.PNG)

- 이러한 통계학적 parametric TTS의 단점은?

    1. 각각 광범위한 전문적인 도메인 지식을 기반하고 디자인하기 매우 어렵다.

    2. 각 요소가 독립적으로 학습해 각 요소에서 발생한 에러가 축적될 수 있다.

    3. 새로운 시스템 구축 시, 상당한 엔지니어적인 노력이 필요로 한다.

![ccc](https://user-images.githubusercontent.com/59636424/132998243-4d889fa7-9d45-436b-b42c-afc42ceb61c6.PNG)


![ent-to](https://user-images.githubusercontent.com/59636424/132998189-43cbf9b1-ed24-425e-bd11-78c41c8de448.PNG)


- **그래서 최소한의 사람이 라벨링한 <text,audio> pair로만 학습될 수 있는 end-to-end TTS 시스템을 사용할 시, 장점은?**

    1. 힘든 feautre engineering의 필요를 경감시켜준다.

    2. 화자, 언어, 감정과 같이 다양한 속성에 대해 많은 조건을 쉽게 조절 가능하다.

    3. 새로운 데이터에 대한 적응도 쉽다.

    4. 하나의 모델은 각 요소의 오류를 축적할 수 있는 multi stage model보다 더 좋을 것이다.

---

**따라서, 이러한 이점을 통해 end-to-end모델로 현실에서 얻을 수 있는 다양하고 noisy한 많은 데이터를 학습할 수 있도록 한다. -> 이러한 측면에서 모델을 만들었다!**

- end-to-end 모델은 어려운 학습 과제가 있다!

    같은 text라도 다른 발음이나 말하는 스타일로도 대응할 수 있어 학습이 어렵다.(그래서 주어진 입력을 다양한 signal level로 대체된다.)
    
    TTS output은 연속적이고 output sequence는 보통 입력보다 많이 길어 prediction error가 빠르게 축적된다.

**그래서 논문에서는 Tacotron을 제안!**

: attention과 sequence to sequence 모델을 기반한 end-to-end TTS 모델이다.

    입력은 characters, 출력은 raw spectrogram이다.
    
    <text, audio>로 주어진 쌍에서, random 초기화로 완전히 훈련될 수 있다.
    
    음소(언어의 낱말을 구분시켜주는 이론적인 낱낱의 소리 / 쉽게 말해 영어사전의 발음기호를 생각) 수준의 할당이 필요없으니 text만 있는 많은 양의 데이터로 학습 가능하다.
    
    US English evaluation st에서 3.82의 mean opinion score를 기록했다.

## 2. Model Architecture

![wwww](https://user-images.githubusercontent.com/59636424/132988754-5631abd4-d7f7-40c6-ae99-865b29664844.png)

Tacotron은 attention 기반 sequence to sequence 모델이다. (구성은 크게 encoder, attention 기반의 decoder, post-processing net이 존재한다.)

### 2-1. CBHG Module

![cbhg논문](https://user-images.githubusercontent.com/59636424/132989187-634c43d0-4f13-4c67-99c4-85f99e4b9b2d.PNG)

CBHG는 encoder와 decoder 둘 다 사용하고 **1-D convolutional filters의 bank, Highway network, Bidirectional GRU**를 사용한다. (앞글자를 따서 CBHG)

CBHG는 sequence로부터 특성을 추출하는 강력한 모듈이다!

![cbhg 종류](https://user-images.githubusercontent.com/59636424/132989701-394988e9-f2bc-4879-8902-737b489ac43a.PNG)

위의 그림은 encoder의 CBHG 구조로 자세히 뜯어보자!

  1. input sequence를 **k개의 1-D convolutional filters를 가진 bank를 통과**한다. **kth filter는 width k**를 가지는 convolution filter이다. (k=1,2,....k)
    
    1D convolution Bank는 k개의 필터(각각 k의 길이는 1~k)를 가지고 있다.
    
    각 filter는 k개의 Sequence를 보고 특정길이(k)를 고려하여 정보를 추출하는 역할
    
    이 filter는 local 정보와 문백 정보를 추출한다! (unigram, bigram,.. k-gram까지 filter를 이용해서 모델링하는 것과 비슷하다!)
    
    이렇게 k개의 convolution의 출력은 stacking 된다. (쌓인다)
    
  2. 시간에 따라 **max pooling**을 하여 Sequence에 따라 변하지 않는 부분(local invariance)를 추출한다. (local invariance(변하지 않는 부분)을 증가시키기 위해)
  
    -> 이렇게 local invariance를 증가시키는 것은 문맥이 달라져도 변하지 않는 부분들을 강조한다는 뜻이다!
  
  2-1. stride 1을 사용하여 time resolution(시간 축 상의 해상도)을 보존한다!
    
    -> 최대한 들어온 sequence 순서를 지켜주기 위해서!!

  3. **고정된 폭을 가지는 1D convolution**을 통과 -> Sequence 데이터의 벡터 사이즈와 일치하는 벡터를 생성
  
  4. **residual connection**으로 앞서 통과해서 생성된 벡터와 input Sequence 벡터와 더한다.
  
    residual connection으로 모델의 깊게 쌓을 수 있게 되고 학습할 때 빠르게 수렴이 가능하다.

  5. **Highway 네트워크**를 통과하여 high level 특성을 추출한다.
  
  6. 정방향 문맥과 역방향 문맥에서 Sequential 특성을 추출하기 위해서 **bidirectional GRU** 사용한다.

**모든 1-D convolution Network는 Batch Normalization을 포함(정규화 작용)**

---

* Highway 네트워크

![ㄱㄱㄱㄱㄱ](https://user-images.githubusercontent.com/59636424/132992224-cbcbbd7c-1fa0-42bd-918f-76af7487e7b3.PNG)

Hightway 네트워크는 Gate 구조를 추가한 Residual Connection이다. **입력값 x와 함수 H(x)를 어느정도의 비율로 섞을지를 학습하여 결정한다.**

0~1의 값을 갖는 T(x)를 만들어 x와 H(x)에 곱해준다!

=> 이러한 것이 층이 깊어지더라도 속성을 유지할 수 있으므로 high level 특성을 추출할 수 있다!

### 2-2. Encoder

![인코더](https://user-images.githubusercontent.com/59636424/132992445-9eb99f22-fd01-4c93-9cd2-f30d96c85d51.PNG)

**인코더의 목표는 text의 강력한 시퀀셜 표현을 추출하는 것이다.**

  1. input을 character Sequence로 받고 각 문자들을 one-hot벡터로 만든 후, embedding vector로 변환시켜준다.

  2. non-linear transform인 **prenet**의 dropout과 bottleneck layer로 수렴을 돕고 일반화 효과를 낸다.
  
        Channel 의 차원을 축소하는 개념이 bottleneck layer(FC layer -> RELU -> Dropout(0.5) -> FC layer -> RELU -> Dropout(0.5))
        
        Dropout으로 과적합을 방지하려한다.
  
  3. CBHG module로 최종 인코더의 output으로 바꾼다.

### 2-3. Decoder

decoder는 **content-based tanh attention decoder**를 사용한다! (그림에서 2번째 line에서 Attention RNN이라고 되어 있는데 Decoder RNN이다!)

content-based Function은 dot product, general, concat이 있는데(Seq2Seq with attention 강의에서 attention mechanisms 3가지) 이중에서 content-based tanh는 아래와 같다.

![ㅈㅈㅈㅈ](https://user-images.githubusercontent.com/59636424/132995137-8e2b3d4a-e9cf-4a5d-9a8e-1253aba17188.PNG)

그리고 decoder RNN과 attention RNN을 사용한다!

input으로 context vector + attention RNN cell output을 사용한다!

![디코더](https://user-images.githubusercontent.com/59636424/132995163-41289fa1-ed01-44cc-a8c9-d409982842f0.PNG)

* 전반적인 Decoder 흐름

**decoder는 인코더에서 생성된 Sequence 벡터(context vector)와 t-1 시점까지 생성된 decoder의 mel-scale spectrogram을 input으로 사용해 t 시점의 mel-scale spectrogram을 생성!**

        * mel-scale spectrogram이란?
        
        원래 peach(음의 높낮이)가 주파수가 증가함에 따라 exponential 상승하니 log 적용하여 linear하게 만든 spectrogram이다.
        
   1. decoder의 **input은 t-1 시점까지 decoder에서 생성된 mel spectrogram**이다. 처음 시점에는 생성된 spectrogram이 없으므로 all-zero frame <Go>를 input으로 사용한다.
    
   2. input을 pre-net을 통과시켜 벡터 생성 후, Attention-RNN의 input으로 사용한다.
   
        encoder와 마찬가지로 decoder의 pre-net도 과적합을 막기 위해 사용한다! -> dropout이 그 역할을 한다!
    
   3. Attention-RNN에서 추출된 Sequence hidden vector를 Query로 attention에 넣어 encoder의 vector의 각 시점과 관련된 vector의 가중합인 **context vector**를 추출한다.
    
   4. 이렇게 구한 **Sequence hidden vector와 context vector를 concat**해서 Decoder-RNN의 Input으로 사용한다!
   
   5. Decoder-RNN에서 추출된 결과가 Decoder output인 t시점의 mel spectrogram이다!
   
        각 decoder step에서 한번에 r개의 겹치지 않은 output frame을 예측
        
        -> r개의 예측값 전체를 input으로 사용할 수도 있다!
    
        이렇게 한 시점에서 r개의 frame(r개의 mel spectrogram)을 한꺼번에 뽑으면 decoder step의 전체 수를 r만큼 줄일 수 있다!
    
        -> 그러므로 학습 및 수렴 속도를 상승시킨다!

이렇게 나온 매 시점별 생성된 r개씩의 mel spectrogram을 post-processing net의 input으로 사용한다!
    
### 2-4. POST-PROCESSING NET AND WAVEFORM SYNTHESIS

![ㅌㅌㅌ](https://user-images.githubusercontent.com/59636424/132996670-ed4acef7-d102-4a01-a8cf-3462b8c007d1.PNG)

**post-processing net의 역할은 seq2seq target을 waveform(파형)으로 합성할 수 있는 target으로 변환시켜주기**
    
* post-processing net에서 **decoder에서 mel spectrogram 전체를 보고 linear spectrogram을 생성**한다! 
    
    -> decoder에서 바로 linear spectrogram을 생성한 tacotron모델보다 좋은 품질의 음성을 생성한다!
    
    Seq2Seq 모델은 decoder는 시점별로 한개씩 mel spectrogram을 생성
    
* post-processing net에서 CBHG module에서 Bidirectional GRU를 이용하므로 각 frame에 예측 에러를 바로잡기 위해 정방향, 역방향 정보를 모두 가진다!
    
    -> Seq2Seq모델은ㅊ항상 왼쪽에서 오른쪽으로 동작한다.

* Griffin-Lim 알고리즘
    
    **linear spectrogram을 음성 신호(waveform)로 합성하는데 사용되는 알고리즘이다!**
    
    미분을 이용하고 학습 가능한 weight가 없기 때문에 이 과정에서 어떠한 loss도 없다.
    
    
## 3. EXPERIMENTS

Tacotron을 North American English dataset(전문 여성 화자의 24.6시간 정도의 발화)에 대해 학습
    
### 3-1. Ablation Analysis
    
![ㄱㄱㄱㄱㄱㄱ](https://user-images.githubusercontent.com/59636424/132997824-40a63049-b853-45ea-a719-737414bf6983.PNG)

    
* vanilla Seq2Seq 모델(a)과 attention alignment(sequence) 비교

-> vanilla Seq2Seq 모델의 output의 그래프를 우선 보면 좀 늘어져 있고 실제로 음성도 이상하게 늘어지는 소리가 나는 결과가 나왔다.

대조적으로 Tacotron은 깨끗하고 스무스한 alignment를 갖는다. -> 실제로 Tacotron은 깔끔한 소리가 나는 결과가 나왔다.
    
* (b)는 CBHG encoder 대신, 2-layer의 residual GRU 인코더를 사용하여 비교
    
-> GRU encoder의 alignment가 더 noisy함을 알 수 있다. -> 실제로 음성도 약간의 noisy가 껴있다!

---
    
![ㅂㅂ](https://user-images.githubusercontent.com/59636424/132997924-c17097ae-5991-4333-b7e8-e0348060af84.PNG)

* post process net을 사용한 모델과 사용하지 않은 모델 비교
    
-> post-processing net의 예측이 더 나은 하모닉(여러 배음)을 포함한다.(사진에서 post process net은 아래에서 주름이 좀 더 이어지는 것을 확인할 수 있다.)
    
-> post process net을 사용하지 않는 것은 실제 음성에서는 인조적 소리가 나고 사용하는 것은 깔끔하고 자연스러운 소리가 난다.
    
### 3-2. Mean Opinion Score Tests

![ㅈㅈㅈㅈ](https://user-images.githubusercontent.com/59636424/132997993-6250e8f5-fc3c-4243-8a4f-a0c2953803f7.PNG)

* 본 논문에서는 mean opinion score test를 진행했고, 자연스러움에 대해 5점 만점으로 평가
    
* prarmetric 시스템과 concatenative 시스템과 비교했을 때, **3.82로 좋은 결과**를 보여줬다.

## 4. Discussions

* 논문에서 **Tacotron을 제안**했고 이는 character 시퀀스를 입력으로 받아 spectrogram을 출력하는 **end-to-end TTS 모델**

* mean opinion score로 실제 자연스러움의 부분에 있어 **parametric system의 성능을 능가**했다.
    


