# [Reference and Document Aware Semantic Evaluation Methods for Korean Language Summarization](https://aclanthology.org/2020.coling-main.491.pdf)



## 1. Introduction

* 생성된 요약과 참조 요약 사이의 의미론적 의미를 반영하지 않기 때문에 ROUGE는 제한적이다.
* ROUGE 점수는 n-gram 중첩을 기반으로 계산되므로, 두 단어가 동일한 의미를 가지더라도 점수가 낮을 수 있다.
* 영어와 달리 **다양한 형태소를 하나의 단어로 묶어 여러 가지 의미와 문법적 기능을 표현하는 응집어인 한국어**에서 특히 두드러진다.

![Untitled](https://user-images.githubusercontent.com/59636424/180901104-160eb113-86a1-4c05-95bb-20e305cfee9b.png)

<div align="center"><한국어 Rouge Score 한계></div>
  
  
**생성된 요약 및 참조 요약과 함께 원본 문서도 고려해야 한다.**

**이제 광범위한 평가를 통해, 우리는 제안된 평가 지표에서 인간 판단과의 상관 관계가 ROUGE 점수보다 훨씬 더 높다는 것을 보여주려고 한다.**
  
## 2. Related Work
  
### 텍스트 요약의 평가 방법
  
* 자동 평가 방법
  
1. extrinsic automatic method: 문서 관련성 판단을 구성하는 작업의 완료에 어떤 영향을 미치는지에 기초하여 요약 모델을 평가
2. intrinsic automatic method: 속성 분석을 통해 또는 수동으로 생성된 요약과의 유사성을 계산하여 품질을 평가
  
* pyramid method: 인간이 만든 다양한 요약을 검사하고 각각 점수 가중치를 갖는 요약 콘텐츠 단위를 만든다.
* Rouge: 후보와 참조 요약 사이의 어휘 겹침의 유사성을 평가한다. → **동의어 또는 구문을 설명하지 않는다.**
  
위의 Metric을 보완하고자 나온 Metric 존재

- ParaEval: 구문 테이블을 기반으로 하는 일치 방법을 사용
- ROUGE-WE: 의미론적 유사성 측정과 토큰 사이의 코사인 거리를 가진 어휘 매칭 방법을 사용
- ROUGE 2.0: WordNet을 동의어 사전으로 사용하고 계산 토큰은 일치하는 단어의 모든 동의어와 겹친다.
- ROUGE-G: WordNet의 어휘 및 의미 매칭을 사용 → 어려운 수작업으로 만든 사전과 동의어 사전을 필요로 하기 때문에 한계
  
**위 Metric은 동의어 구조를 지원하기 위해 ROUGE를 확장하는 데 사용**
  
모델로는 최근에 텍스트 요약 모델이 제안되어 텍스트 범위를 요약에 포함해야 하는지 여부를 예측하기 위한 모델을 훈련 → 강화 학습 기반 요약 모델도 제안
  
하이브리드 접근법은 추상적 방법과 추출적 방법을 모두 사용
  
## 3. Methodology
  
깊은 의미론적 의미를 반영하기 위해 참조 요약을 사용하여 생성된 요약을 평가하는 방법을 제안
  
**생성된 요약을 원본 문서와 참조 요약과 함께 평가하는 방법을 제안**
  
### 3.1 Reference and Document Aware Semantic Evaluation
  
$$
y_p = [w_1, ..., w_n] $$
  
: generated summary from the summarization model
  
$$
y_r = [w_1, ..., w_m] $$

: reference summary
  
y_p와 y_r는 constructed using sentence-embedding methods
  
SBERT는 의미론적 유사성 검색에 적합하며 BERT, RoBERTa 및 범용 문장 인코더를 포함한 이전의 최첨단 접근 방식보다 빠른 추론 속도를 보여주었다. → SBERT 이용
  
### 계산 과정
  
* 1단계. 사전 훈련된 SBERT를 활용하여 각 e를 SBERT 임베딩으로부터 얻어진다.

![Untitled (1)](https://user-images.githubusercontent.com/59636424/180903786-77f0d3b9-3d9a-4883-918a-decec5c33425.png)
  
* 2단계. v_p를 구성하기 위해 mean pooling을 수행 (j는 word embedding dimension 지수 / n은 E 길이)
  
![dd PNG](https://user-images.githubusercontent.com/59636424/180903864-0b7b16a9-7ac9-4110-90d4-906a1fd8b4a3.png)

* 3단계. v_p와 v_r 사이의 의미 유사성 점수를 얻기
  
![dd PNG (1)](https://user-images.githubusercontent.com/59636424/180903899-2fc95aa5-84ec-454d-889f-cd98774f3779.png)
  
원본 문서와의 사실적 일관성을 고려하는 것이 중요하다는 점을 상기
  
동일한 문서에 따라 중요한 정보를 요약하는 방법은 사람마다 다르다. **따라서 요약 모델을 평가할 때 생성된 요약과 함께 원본 문서도 고려**
  
* 4단계. D = [w1, ..., wk], the document representation, vd, can be obtained using Eqs. (1) and (2). Thus, the similarity score between vp and vd can be defined a

![dd PNG (2)](https://user-images.githubusercontent.com/59636424/180905486-58706420-c4b5-4e60-ae77-2aa8ffe94142.png)
  
* 5단계. 참조 및 원본 문서가 주어진 경우 생성된 요약본의 참조 문서 인식 의미 점수(RDASS)는 s(p, r)와 s(p, d)를 평균하여 정의

![dd PNG (3)](https://user-images.githubusercontent.com/59636424/180905535-9f3f13e2-4656-485f-93aa-215076dc7b97.png)
  
### 3.2 Fine-tuning SBERT with the Abstractive Summarization Model
  
참조 요약 및 원본 문서에 대한 보다 상황화된 정보를 포착하도록 추가 교육을 실시할 수 있다. **abstractive 요약 모델을 이용한 SBERT의 미세 조정 방법을 제안**
  

D = [w1, ..., wk](문서) → hidden represention(hp = [h1, ..., hn]) → summary 생성 (yp = [w1, ..., wn])

hidden representation은 decoder의 output vector! → 이 hidden representation으로 SBERT를 미세조정 → 이를 **FWA-SBERT**이라고 표현



