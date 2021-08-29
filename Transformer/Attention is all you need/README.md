# Attention is all you need

* 본 논문 링크: https://papers.nips.cc/paper/2017/file/3f5ee243547dee91fbd053c1c4a845aa-Paper.pdf

## Abstract

이때까지 Recurrent model은 순차적으로 처리하는 특징으로 여러 작업을 동시에 수행하지 못 했다!

attention을 통한 encoder와 decoder를 연결하는 모델이 최고의 모델이다!! (이 논문이 2017년에 나왔는데 이때는 맞는 듯 하다!)

이 논문에서는 앞서 말한 문제가 있던 Recurrence를 제거하고 attention mechanism을 이용한 network 구조를 가진 **Transformer**를 제안하려고 한다!

이러한 mechanism은 **병렬 처리 기능**과 **학습 시간이 덜 소요**되므로 성능이 좋다!!
