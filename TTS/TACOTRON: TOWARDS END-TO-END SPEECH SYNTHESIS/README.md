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
    
    4. 음성 신호 합성을 위한 vocoder(스펙트로그램(음성을 숫자로 표현하는 여러가지 방법 중 하나)을 소리로 변환)
    
    이렇게 4가지로 이루어져 있다.

- 이러한 통계학적 parametric TTS의 단점은?

    1. 각각 광범위한 전문적인 도메인 지식을 기반하고 디자인하기 매우 어렵다.

    2. 각 요소가 독립적으로 학습해 각 요소에서 발생한 에러가 축적될 수 있다.

    3. 새로운 시스템 구축 시, 상당한 엔지니어적인 노력이 필요로 한다.

- **그래서 최소한의 사람이 라벨링한 <text,audio> pair로만 학습될 수 있는 end-to-end TTS 시스템을 사용할 시, 장점은?**

    1. 힘든 feautre engineering의 필요를 경감시켜준다.

    2. 화자, 언어, 감정과 같이 다양한 속성에 대해 많은 조건을 쉽게 조절 가능하다.

    3. 새로운 데이터에 대한 적응도 쉽다.

    4. 하나의 모델은 각 요소의 오류를 축적할 수 있는 multi stage model보다 더 좋을 것이다.

---

**따라서, 이러한 이점을 통해 end-to-end모델로 현실에서 얻을 수 있는 다양하고 noisy한 많은 데이터를 학습할 수 있도록 한다. -> 이러한 측면에서 모델을 만들었다!**



