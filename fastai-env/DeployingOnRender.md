## Render 배포

## 목차
[일회성 설정](#One-time_setup)
* [GitHub로 부터 starter app fork하기](#forkstarterapp)
* [Render 계정 생성](#createaccount)

[프로젝트 별 설정](#Per-project_setup)
* [학습된 모델 파일 올리기](#uploadmodel)
* [app을 모델에 맞게 변경하기](#customizemodel)
* [변경된 내용을 GitHub에 push와 commit하기](#pushgithub)

[배포](#deploy)
[테스트](#test)
[개인PC에서 테스트](#localtest)
<br>


![render-logol](https://course.fast.ai/images/render/render-logo.svg  =150x50)

이것은 몇번의 클릭만으로 학습된 모델을 [Render](https://render.com/)에 배포하는 간단한 가이드입니다.  제 2장에서  
Jeremy’s Bear 이미지를 분류할 때 사용하는 [start repo](https://github.com/render-examples/fastai-v3)에서 나옵니다. 

starter app은  https://fastai-v3.onrender.com/에 배포 되어져 있습니다.


## 일회성 설정<span id="One-time_setup"></span>

### GitHub로 부터 starter app fork하기<span id="forkstarterapp"></span>
https://github.com/render-examples/fastai-v3를 자신의 GitHub로 fork 해주세요.

### Render 계정 생성<span id="createaccount"></span>
Render에 [가입](https://render.com/i/fastai-v3)하세요.  가입하는데 신용카드는 요구하지 않습니다.

## 프로젝트 별 설정<span id="Per-project_setup"></span>
만약 Render에 초기 배포 테스트를 원한다면, starter repo는 Jeremy’s 곰 분류 모델이 기본적으로 설정 되어있습니다. 그게 아니고 자신의 모델로 테스트를 원하면 계속 읽으세요.

### 학습된 모델 파일 올리기<span id="uploadmodel"></span>
``learner.export``(예: export.pkl)를 사용하여 만든 교육받은 모델 파일을 Google Drive 또는 Dropbox와 같은 클라우드 서비스에 업로드하십시오. 그리고 파일의 다운로드 링크를 복사하십시오.

**주의:**  다운로드 링크는 직접 파일이 다운로드를 시작해야합니다. 일반적으로 사용되는 뷰를 제공하는 공유링크와는 다른것 입니다. 
* Google Drive : [링크 생성 법](https://www.wonderplugin.com/online-tools/google-drive-direct-link-generator/)
* Dropbox : [링크 생성 법](https://syncwithtech.blogspot.com/p/direct-download-link-generator.html)

### app을 모델에 맞게 변경하기<span id="customizemodel"></span>

 1. app 폴더에 있는 ``server.py``를 편집기를 통하여 열고 ``model_file_url`` 이 부분을 위에서 복사 해둔 url로 변경해주세요. 
 2. 같은 파일에서 ``classes = ['black', 'grizzly', 'teddys'] `` 에 부분 클래스 내용을 모델에서 예상하는 것으로 변경하세요.


### 변경된 내용을 GitHub에 push와 commit하기<span id="pushgithub"></span>
GitHub repo 상태를 위에서 생성한 현재의 상태로 유지하여주세요. Rendor는 GitHub repo와 통합을 하고 변경이 있어 사용자가 push 할때 마다 자동으로 빌드하고 배포합니다.


## 배포<span id="deploy"></span>

 1. Render에서 새로운 웹 서비스를 생성하고  위에서 생성한 repo를 사용하세요. 이 단계에서 사용자는 Render에게 repo에 접근할수있는 권한을 부여 해줘야 합니다. 
 2. 배포 화면에서 서비스 이름을 선택하고 도커를 사용하세요. URL은 서비스이름을 토대로 생성 될 것입니다. 필요에 의해서 서비스 이름은 변경 가능하지만 URL은 사용자가 변경 하지 못합니다. 
 3. **Save Web Service**을 클릭하세요. 그럼 다 된거에요. 그럼 사용자의 서비스가 구축 될 것입니다. Render 대쉬보드에 표시된 URL은 몇분안에 살아나야 합니다. 이것에 되한 진행과정을 배포 로그에서 확인 가능합니다.


## 테스트<span id="test"></span>
App의 URL은 https://service-name.onrender.com.와 비슷할 것 입니다. App을 테스트 할때 서비스 로그를 모니터링 가능 합니다. 

## 개인 PC에서 테스트<span id="localtest"></span>
App 서버를 로컬에서 실행하려면 터미널에서 아래의 명령어를 실행 하세요.
```
python app/server.py serve
```
만약 Doker를 설치 하였다면,  아래 명령어를 실행하면 Render와 동일한 환경에서 테스트 할 수 있습니다. 
```
docker build -t fastai-v3 . && docker run --rm -it -p 5000:5000 fastai-v3
```
테스트를 위해 [http://localhost:5000/](http://localhost:5000/)로 이동하세요 

 

>  샘플 코드를 제공한 Simon Willison에게 감사를 표합니다.
