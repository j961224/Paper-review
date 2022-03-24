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

그래서, token i와 j까지의 relative distance를 고려하기 위한 Relative Position을 사용하려 한다.

(논문 사진 3 첨부)

아래는 Contents vector와 Position Vector를 Q_c, K_c, V_c, Q_r, K_r로 표현할 수 있다. 하지만, V는 마지막에 Softmax와 곱할 때 사용하므로 V_r은 필요가 없다.

(논문 사진 4 첨부)

따라서, 위의 식과 같이 **Content to Content, Content to Position, Position to Content**을 고려하여 계산한다. Attention Score 계산이 끝나면 루트(3 x d)로 크게 scaling을 해줘서 모델 학습에 안정화를 시켜준다.




원래 relative position embedding 구할 시에 sequence 길이가 N이라면, O(N^2 * d)이지만, K와 Q가 저장되어 재사용되므로 새로 할당받을 필요가 없으므로 매번 계산하더라도 메모리가 O(kd)로 감소시켰다.






