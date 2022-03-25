# DEBERTA: DECODING-ENHANCED BERT WITH DISENTANGLED ATTENTION

## 1. Abstract

Deberta는 Decoding-enhanced BERT with disentangled Attention이다.

2개의 저명한 기술을 사용!!

> 1. 각 단어는 **content embedding과 relative position embedding**으로 표현!! -> Token vector
> 2. MLM pretrain 시, mask token을 예측하는데 decoding layer에서 **absolute position**을 함께 사용! -> enhanced mask decoder
> 3. 추가적으로, fine-tuning 시에 **virtual adversarial training**을 사용!

## 2. Deberta Architecture

### Disentangled Attention

H_i: Sequence i번째에 대한 context vector

P_{i|j}: i번째의 token에서 j번째에 대한 relative position vector

![tntlr](https://user-images.githubusercontent.com/59636424/159870581-91e910e5-c857-4f13-9f37-5ec6f3c3c97d.PNG)

위의 수식은 Content Vector와 Relative position Vector와의 Cross Attention식이다.

각 수식은 **Content to Content, Content to Position, Position to Content, Position to Position**으로 이루어져 더해진다. 하지만, position to position은 relative position embedding을 사용하기에, 계산을 하지 않는다!!

* Content to Position: i번째 위치한 Content가 있다면, 전체 시퀀스 내 각 j 위치의 Relative Position에서 해당 Content가 어떤 의미인지 Attention Score 구하여 모델링
* Position to Content: 모든 시퀀스 내 각 Content에서 해당 i 위치의 Position이 어떤 의미를 지니는지 Attention Score를 구하여 모델링


기존 연구에는 Content to Position, Content to Content로 Attention weight를 구했지만, 한 쪽 방향으로만 모델링 할 수 없으니 **Position to Content**도 고려한다.

---

그래서, token i와 j까지의 relative distance를 고려하기 위한 Relative Position을 사용하려 한다.

![33](https://user-images.githubusercontent.com/59636424/159897449-607cff4f-2994-4b64-974d-91dc69110d75.PNG)

아래는 Contents vector와 Position Vector를 Q_c, K_c, V_c, Q_r, K_r로 표현할 수 있다. 하지만, V는 마지막에 Softmax와 곱할 때 사용하므로 V_r은 필요가 없다.

![44](https://user-images.githubusercontent.com/59636424/159897467-5a78870a-9432-42c8-9cb7-04ccb819bbe8.PNG)

따라서, 위의 식과 같이 **Content to Content, Content to Position, Position to Content**을 고려하여 계산한다. Attention Score 계산이 끝나면 루트(3 x d)로 크게 scaling을 해줘서 모델 학습에 안정화를 시켜준다.

---

![temp](https://user-images.githubusercontent.com/59636424/160038726-e4fb2f64-f2af-47b4-8263-2d244b7f6518.png)

원래 relative position embedding 구할 시에 sequence 길이가 N이라면, O(N^2 * d)이지만, K와 Q가 저장되어 재사용되므로 새로 할당받을 필요가 없으므로 매번 계산하더라도 메모리가 O(kd)로 감소시켰다.

---

### Enhanced Mask Decoder

Deberta는 Bert와 같이 MLM으로 학습하는 모델이지만, Disentangled Attention은 content와 relative position만을 고려하고 absolute position을 고려하지 않는다. 그래서, 결정적인 이슈들이 많고 absolute position은 문법적으로 필요한 요소이므로 absolute position을 사용하려 한다.


* Absolute position이 필요한 이유

```
A new store opened beside the new mall
```

-> store과 mall 주변 단어에 new라는 같은 단어가 있어 context와 relative position만 이용하면 구분이 힘듦 -> 그래서 서로 다른 단어인 것을 Absolute position으로 고려해줌!!

**방법은 MLM softmax 취하기 전에 absolute position을 넣어 합치려고 한다!**

추가적으로, EMD는 pretraining 때, position 뿐만 아니라 유용한 정보를 넣을 수 있다!!

아래 그림은 Deberta mlm pretrain 과정이다!

![2](https://user-images.githubusercontent.com/59636424/160087779-f7331c7c-496f-46a3-876c-9b0c27d03cc2.png)

![1](https://user-images.githubusercontent.com/59636424/160087308-33b3bfcb-1e9b-4aea-b58c-ab5a8540cd4f.png)


---

### SCALE INVARIANT FINE-TUNING (SIFT)

안정적으로 fihne tuning을 하기 위한 알고리즘이다!!

* Virtaul adversarial training

**모델의 성능을 일반화**하는데 좋고 Input에 small perturbation 줌으로써, adversarial attachk에도 모델 robustness를 증진시킨다! -> 최대한 같은 output을 낼 수 있도록 한다!!

NLP task에는 word sequence 대신, **word embedding에 perturbation을 적용**시킨다!

하지만!, 임베딩 값은 단어나 모델마다 매우 다르고 큰 모델일수록, variance가 커지며 adversarial training의 불안정성을 키운다!! => 그래서 **normalized word embedding에 perturbation을 추가하는 Adversarial Fine-Tuning**

**그리고 모델의 크기가 클수록 성능 개선이 뚜렷하다!!**


