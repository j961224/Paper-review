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

* Content to Position: i번째 위치한 Content가 Query로, j position에 대해 Relative position 정도 매핑하여 Attention score를 구한다.
