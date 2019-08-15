# Kaggle.com

## 목차
- [fast.ai v3 코스를 위한 Kaggle 커널을 사용하는것](#fastai)
- [커널의 색인](#index)
- [시작하기 위한 방법](#getting_started)
  - [단계 1: Kaggle 계정의 생성](#step1)
  - [단계 2: fast.ai 강의의 각각에 대한 Notebook 둘러보기](#step2)
  - [단계 3: 모든것이 설정 되었고, 작업을 시작해 보기!](#step2)
- [리소스 & 한계점](#resources)

## Kaggle 커널에 오신것을 환영 합니다! 
![Kaggle 커널에 오신것을 환영 합니다](https://course.fast.ai/images/kaggle/landing_page.png)

[Kaggle](https://kaggle.com/)은 Google, Inc가 소유한 데이터 과학자와, 머신 러닝을 연구하는자들을 위한 온라인 커뮤니티 입니다. Kaggle은 사용자들이 데이터셋을 출판하고, 검색할 수 있게 헤 줘서 웹-기반의 데이터과학 환경에서 모델을 만들어 볼 수 있게 해 줍니다. 또한, 다른 데이터 과학자들과 머신러닝 엔지니어들간에 협업하고, 데이터과학 문제에 대한 경연에 참가할 수 있도록 해 줍니다. Kaggle은 머신러닝 경연을 제공하면서 시작되었고, 지금은 데이터과학을 위한 클라우드 기반의 워크벤치인 공용 데이터 플랫폼 또한 제공합니다 ([더 읽어보기](https://en.wikipedia.org/wiki/Kaggle)).

하지만, Kaggle 커널은 몇 가지 한계점이 있으므로, 이에 대한 내용은 하단부의 [리소스 & 한계점](#resources) 섹션에서 확인해 보시기 바랍니다.

아래 나열된 단계를 이미 완료 하였고, 하던 작업을 재개하려는 분은 [작업 재개하기 섹션]((https://course.fast.ai/update_kaggle.html))을 참고하기 바랍니다.

## fast.ai v3 코스를 위한 Kaggle 커널을 사용하는것 <span id="fastai"></span>

Kaggle 커널은 fastai 라이브러리가 미리 설치된 상태로 제공됩니다. William Horton @wdhorton과 Sanyam Bhutani @init_27이 강의에서 사용된 Notebook을 커널로 포팅해 두었습니다. Sanyam Bhutani가 Kaggle 커널을 관리하기 때문에, 관련된 질문과 문제점은 [이곳에서](https://forums.fast.ai/t/platform-kaggle-kernels/32569) 이야기 되어야 합니다.

설정 단계는 필요치 않고, 단순히 "Copy and Edit" 버튼을 눌러서 fork 한 후 실행하면 됩니다.

## 커널의 색인 <span id="index"></span>
- [Lesson 1 Pets](https://www.kaggle.com/hortonhearsafoo/fast-ai-v3-lesson-1)
- [Lesson 2 Download](https://www.kaggle.com/init27/fastai-v3-lesson-2)
- [Lesson 2 SGD](https://www.kaggle.com/init27/fastai-v3-lesson-2-sgd)
- [Lesson 3 Camvid-tiramisu](https://www.kaggle.com/hortonhearsafoo/fast-ai-v3-lesson-3-camvid-tiramisu)
- [Lesson 3 Camvid](https://www.kaggle.com/hortonhearsafoo/fast-ai-v3-lesson-3-camvid)
- [Lesson 3 Head-Pose](https://www.kaggle.com/hortonhearsafoo/fast-ai-v3-lesson-3-head-pose)
- [Lesson 3 Planet](https://www.kaggle.com/hortonhearsafoo/fast-ai-v3-lesson-3-planet)
- [Lesson 3 Tabular](https://www.kaggle.com/hortonhearsafoo/fast-ai-v3-lesson-3-imdb)
- [Lesson 4 Collab](https://www.kaggle.com/init27/fastai-v3-lesson4-collab)
- [Lesson 4 Tabular](https://www.kaggle.com/init27/fastai-v3-lesson-4-tabular)
- [Lesson 5 SGD-MNIST](https://www.kaggle.com/hortonhearsafoo/fast-ai-v3-lesson-5-sgd-mnist)
- [Lesson 6 Pets-more](https://www.kaggle.com/init27/fastai-v3-lesson-6-pets)
- [Rossmann data clean](https://www.kaggle.com/init27/fastai-v3-rossman-data-clean)
- [Lesson 6 Rossmann](https://www.kaggle.com/init27/fastai-v3-lesson-6-rossman)
- [Lesson 7 Human-numbers](https://www.kaggle.com/init27/fastai-v3-lesson-7-human-numbers)
- [Lesson 7 Resnet MNIST](https://www.kaggle.com/init27/fastai-v3-lesson-7-resnet-mnist)

## 시작하기 위한 방법 <span id="getting_started"></span>

### 단계 1: Kaggle 계정의 생성 <span id="step1"></span>

[여기](https://www.kaggle.com/)를 접속하여 Kaggle 에 가입합니다. email 을 검증합니다. 검증이 완료되면, 계정정보를 사용해서 로그인이 가능합니다.

![단계1](https://course.fast.ai/images/kaggle/sign_up.png)

### 단계 2: fast.ai 강의의 각각에 대한 Notebook 둘러보기 <span id="step2"></span>

위의 색엔 섹션에 나열된 lesson중 하나를 클릭합니다. 그러면, 그 lesson에 대한 커널이 열리게 됩니다. 이 때 fork 버튼을 클릭합니다.

![단계2](https://course.fast.ai/images/kaggle/fork.png)

### 단계 3: 모든것이 설정 되었고, 작업을 시작해 보기! <span id="step3"></span>

lesson에 필요한 모든 데이터셋과 사전요구사항은 이미 설치되어 있습니다. Jupyter Notebook으로 작업하는것과 동일한 방식으로 커널 내부에서 작업하시면 됩니다.

![단계3](https://course.fast.ai/images/kaggle/start_working.png)

데이터셋에 최초로 접근하는 경우 알아두어야 할 사항으로, Kaggle은 텍스트 메시지를 통핸 전화번호의 검증을 요구하게 됩니다.

## 리소스 & 한계점 <span id="resources"></span>

- Kaggle 커널은 무료로 실행이 가능하다.
- Notebook은 [fastai repository](https://github.com/fastai/course-v3) 동일한 빈도로 업데이트 되지는 않는다.
- fastai 팀에 의해서 공식적으로 지원되는 것은 아니다 (Sanyam Bhutani에 의해서 관리된다. [관련된 토론 쓰레드 링크](https://forums.fast.ai/t/platform-kaggle-kernels/32569))
- (K-80 인스턴스) GPU 사용시간은 세션당 6시간으로 제한되어 있다.
- 디스크 사용은 커널 당 5GB로 제한되어 있다.
- RAM은 커널당 14GB로 제한되어 있다.
