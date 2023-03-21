# Strategic Design

* 전략적 설계(Strategic Design)
  * 도메인 주도 설계(DDD)는 크게 전략적 설계와 전술적 설계로 나눌 수 있음.
  * 전략적 설계 절차
    1. 프로젝트에 참여하는 사람들 모두가 공통적으로 사용할 보편언어를 정의하고, 이를 가지고 프로젝트를 분석하여 핵심 개념을 정리하여 설계가 이루어짐.
    2. 정리가 이루어진 개념들을 분석하여 제한된 컨텍스트로 잘 표현해야 함. 이들을 가지고 매핑하여 하나의 컨텍스트 맵(Context Map)으로 작성함.
    3. 작성한 컨텍스트 맵을 가지고 서비스를 어떻게 분할 및 통합하여 설계할 것인지 분석 후 그에 따른 결과를 도출함.

<br>

* 보편언어(Ubiqutious Language)
  * 프로젝트 관계자들 모두가 사용할 수 있는 하나의 언어를 의미함.
  * 여기에서 정의한 언어는 설계 및 구현, 산출물 작성 시에 동일하게 사용됨.

<br>

* 제한된 컨텍스트(Bounded Context)
  * 하나의 거대한 시스템에서, 여러 단위로 나누기 위해 맥락을 제한하여 하나의 모델 속에 범위를 정하는 것을 의미함.
  * 각 컨텍스트 안에서 하나의 모델이 구현되며, 그에 맞춰 산출물이 나뉘어져 나오게 됨.

<br>

* 서브도메인(Sub-Domain)
  * 말 그대로 하나의 거대한 비즈니스 도메인을 나누기 위한 단위를 의미함.
    * 핵심 서브도메인(Core Sub-Domain): 다른 경쟁 도메인과 차별화할 수 있는 핵심적인 영역.
    * 지원 서브도메인(Supporting Sub-Domain): 비즈니스에 필수적인 역할을 수행하지만, 핵심적인 영역이 아닌 부분을 의미함. 그렇지만, 탄탄한 지원 서브도메인 없이 핵심 서브도메인이 올바르게 구축되지 못하므로 중요하게 관리가 이루어져야 함.
    * 일반 서브도메인(Generic Sub-Domain): 특화된 영역은 아니지만, 전체적인 비즈니스 도메인 구성에 필요한 일반적인 부분을 의미함.

<br>

* 참고
  * https://engineering-skcc.github.io/msa/DDD-StrategicDesign/
  * https://velog.io/@dvmflstm/도메인-주도-설계-14.-모델의-무결성-유지