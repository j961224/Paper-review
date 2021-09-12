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

