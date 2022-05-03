# StarGAN v2: Diverse Image Synthesis for Multiple Domains

[paper link](https://arxiv.org/pdf/1912.01865.pdf)

## Abstract

1) 다양한 이미지 생성
2) 많은 도메인 확장성

## Introduction

* Domain: 시각적으로 구분되는 것(성별 등)
* Style: 독특한 특징으로 각각의 이미지를 뜻함(메이크업, 금발, 헤어스타일 등)

기존에는 Generator에 one-hot, multi-hot attribute vector를 input으로 합치는 방식을 사용

-> **StarGAN v2에서는 domain specific style code를 사용하려 한다!(domain label)**

> * mapping network: style code로 변형시켜주는 것
> * style encoder: reference image로부터 style code 추출

위에 것으로 여러 domain에 대해 translate 가능하다.(특정 domain에 대해 여러 가지 style에 대해 translate 가능하다.)

## Proposed framework

* image x, domain y

![4가지 framework](https://user-images.githubusercontent.com/59636424/166462858-888ba342-6b8a-45ff-8660-d2b82b839a1f.PNG)

### Generator

> * input: image x & style code s(style을 반영한 code -> mapping network나 style encoder로 만들어진 것)
> * output: G(x,s) = output image

AdalN(adaptive instance normalization)을 아용해 x와 s를 합친다. -> 이것으로 output image 생성

### Mapping Network F

> * input: latent code z & domain y
> * output: style code s(도메인마다 다른 style code)

domain y를 보여주는 latent code와 style code랑 mapping 시켜준다!

### Style encoder

> * input: image x & domain y
> * output: style code s

이 style code는 Generator에서 이미지 생성 때 사용된다.

### Discriminator

> * input: image x
> * output: 진짜인지, 가짜인지 T/F 분류

Generator에서 만들어진 이미지에 대해서 이미지가 도메인에 속해있는지 확인

## Training Objective

### Adversial loss

![adversial loss](https://user-images.githubusercontent.com/59636424/166464199-ccf26382-0b58-4c8b-b294-95c2294db818.PNG)

제일 중요한 loss로 **Generator는 input x, s~로 output image를 생성하게 한다.** (output image = G(x, s~))

그리고, **F(Mapping Network)는 target domain(y~)에 맞는 s~를 생성하도록 한다.**

### Style reconstruction

![style reconstruction](https://user-images.githubusercontent.com/59636424/166464712-5c613c82-3401-4e23-a98e-70eeff8547e4.PNG)

Generator가 이미지를 생성할 때, style code s~로 스타일에 잘 맞게 변환시킬 때 사용한다.

### Style diversification

![style diversification](https://user-images.githubusercontent.com/59636424/166464864-6d6a43c3-d441-44d3-8b83-d56077028bc6.PNG)

Generator가 다양한 스타일의 이미지를 생성할 수 있도록 한다.

수식을 보면, **각각 다른 두 이미지를 만들고 그 사이의 거리를 손실함수로 넣었다.**

논문 상에는 **목적함수 최적점이 없어서 loss weight를 0까지 linearly decay로 학습시켰다고 한다.**

### Preserving source characteristics

![preserving source characteristics](https://user-images.githubusercontent.com/59636424/166465320-2a4f91b8-5fd7-4738-9529-1bda9e32af2b.PNG)

> * s^: image x와 domain y에 대해 추출한 style code(Encoder 사용)
> * s~: target domain에 대한 style code(Mapping Network 사용)

target domain(y~)로 바꾼 이미지를 원래 domain(y)으로 바뀌도록 돌린다.

-> 이는, 처음 image x와 유사하도록 한다.

**도메인과 무관한 특성을 보존하려 한다! -> 이미지 합성이 목표이므로**(생성물이 어떤 결과물인지 보존)

어느정도 원본 이미지의 특징을 갖고 있어야하므로 이렇게 image x를 보존하는 방식으로 학습!

### Full objective

![full objective](https://user-images.githubusercontent.com/59636424/166466496-cf124cbc-3f1a-4b37-8faa-7d50bd6d4bd8.PNG)

앞에서 말한 loss function을 하이퍼파라미터로 조정하여 각 loss 중요도를 반영한다.

중간에 마이너스는 0으로 loss weight를 줄이므로 마이너스로 한 것인가?(최적점?)

## Experiments

* Dataset

> * CelebA-HQ(남자, 여자 2개 domain)
> * AFHQ(cat, dog, wildlife 3개 domain)

* 평가지표

> * FID: 실제 이미지와 생성된 이미지 특정 벡터간 거리 차이값(작을수록 좋다.)
> * LPIPS: 이미지 매치 유사성

### latent-guided synthesis

![latent1](https://user-images.githubusercontent.com/59636424/166467647-5cc75dd5-6c9d-40b3-8271-8e946e88284d.PNG)

![latent2](https://user-images.githubusercontent.com/59636424/166467656-56e0c25a-3605-4aa4-a65a-8692d44a32b0.PNG)

latent-guided는 noise로부터 얼마나 이미지를 잘 생성시켰는지를 확인한다.

### reference-guided synthesis

![reference1](https://user-images.githubusercontent.com/59636424/166467892-1e38c4a9-dd0a-4234-88fd-9d6943ccf1a8.PNG)

![reference2](https://user-images.githubusercontent.com/59636424/166467910-eceb97be-01db-4e04-9c0f-2c2dac7bacc2.PNG)

reference-guided는 reference image로부터 얼마나 feature를 잘 뽑아서 생성했는지를 확인한다.

## Discussion

1. multi-head mapping network와 style encoder로 domain 별 style code 생성
2. 가우시안 분포의 비선형 변환으로 style space 생성(이로써, 모델의 유연성을 주었다.) -> StyleGAN에서도 활용
3. 많은 domain 데이터를 완벽히 활용 -> **도메인과 무관한 특징들을 학습해 더 낫게 일반화!**

* 3번 예시

이전에는 흑발 -> 금발 변형 모델이 있다면, 갈색 머리 사람 이미지는 사용 불가했음

**그러나, StarGAN에서는 도메인과 무관한(포즈, 얼굴 형태 등) 특징들을 자연스러운 사람 이미지 생성에 도움을 준다!** -> 이로써, 데이터를 완벽히 활용했다고 할 수 있다.

## Conclusion

하나의 도메인 이미지를 target 도메인의 다양한 이미지로 변환가능하다.

다양한 도메인에 걸쳐 풍부한 스타일 사진을 만드는 것이 가능하다.
