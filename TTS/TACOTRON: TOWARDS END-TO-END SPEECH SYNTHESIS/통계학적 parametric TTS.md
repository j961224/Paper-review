# 통계학적 parametric TTS

**음성 데이터로부터 음성 특징 추출(음성 특징이 파라미터) -> 파라미터로 통계 모델링 훈련 -> 텍스트 입력 시, 모델로부터 파라미터 생성 -> 음성 합성 과정을 통해 적절한 음성 재구성**

![tttttt](https://user-images.githubusercontent.com/59636424/132984075-22b4ca90-10da-4581-ab34-f5099c5eaeb0.PNG)

* 통계적: 언어적인 feature에서 음성적 feature를 통계적 음성 모델 기반으로 예측

* 파라메트릭: 음성 신호에서 음성적 feature를 추출 및 음성 신호 복원(합성)

* 훈련 단계: 언어적/음성적 feature 추출, 모델 훈련

* 합성 단계: 음성적 feautre 생성, 음성 신호 합성 
