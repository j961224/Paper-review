# DEBERTA: DECODING-ENHANCED BERT WITH DISENTANGLED ATTENTION

## 1. Abstract

Deberta는 Decoding-enhanced BERT with disentangled Attention이다.

2개의 저명한 기술을 사용!!

> 1. 각 단어는 **content embedding과 relative position embedding**으로 표현!! -> Token vector
> 2. MLM pretrain 시, mask token을 예측하는데 decoding layer에서 **absolute position**을 함께 사용! -> enhanced mask decoder
> 3. 추가적으로, fine-tuning 시에 **virtaul adversarial training**을 사용!

## 2. Deberta Architecture

### Disentangled Attention
