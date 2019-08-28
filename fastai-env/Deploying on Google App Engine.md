# Google App Engine에서 배포 하기

## 목차

-  [배포를 위한 start pack 가져오기](#downloadstarerpack)
-  [프로젝트 설정](#per-project_setup)
-  [변경한 App GitHub로 업로드](#uploadapp)
-  [배포](#deploy)
-  [PC에서 테스트](#localTest)

## Google App Engine 배포
이 문서는 Google App Engine Custom runtime을 사용하여 학습된 모델을 배포하는 간략한 가이드 입니다. Lesson2에서 Jeremy’s Bear 이미지를 분류하는 배포한 starter app과 함께 제공 됩니다.

## 배포를 위한 start pack 가져오기<span id="downloadstarerpack"></span>
```
wget https://github.com/fastai/course-v3/raw/master/docs/production/google-app-engine.zip

unzip google-app-engine.zip

cd google-app-engine/app
```

## 프로젝트 별 설정<span id="per-project_setup"></span>

**학습된 모델 업로드** 

  Google Drive 또는 Dropbox와 같은 클라우드 서비스에 학습된 모델 파일을 업로드(예:  stage-2.pth) 하세요. 그리고 파일의 다운로드 링크를 복사하십시오.
**주의:**  다운로드 링크는 파일로 직접 있어야 합니다. 일반적으로 사용되는 뷰를 제공하는 공유링크와는 다른것 입니다.  필요시 아래의 주소에서 링크를 변경 할 수 있습니다.
(공유링크 -> 직접 다운로드 링크로 변경  : https://rawdownload.now.sh/)
만약  배포 테스트만을 원한다면, Jeremy’s 곰 분류 모델이 기본적으로 설정 된 모델을 그대로 사용하면 됩니다.  모델의 가중치 URL은 이미 샘플 App에 기입 되어 있으므로 이 단계를 건너뛸 수 있습니다.


<br>

**App을 나의 모델로 변경하기** 
 
 
  app 폴더에 있는 ``server.py``를 편집기를 통하여 열고 ``model_file_url`` 이 부분을 위에서 복사 해둔 url로 변경해주세요.   같은 파일에서 ``classes = ['black', 'grizzly', 'teddys'] `` 에 부분 클래스 내용을 모델에서 예상하는 것으로 변경하세요.



## 변경한 App GitHub로 업로드<span id="uploadapp"></span>

변경 완료한 App의 폴더를 GitHub로 Push하고 repo의 URL을 복사 합니다.

## 배포<span id="deploy"></span>
우선 Google Cloud 대쉬보드를 열어주세요, 그리고 **_Create Project_** 버튼을 클릭한 다음 새 GCP 프로젝트의 이름을 정해 줍니다.  새로운 비용청구를 위한 계정을 생성하거나 기존의 청구 계정을 이용 하여 설정 가능 합니다.  화면은 아래와 같을 것 입니다. 
![new project](https://cdn-images-1.medium.com/max/1440/1*J_JfUCxs-WAfsNJsW_gXjQ.png)
GCP 프로젝트를 생성 후 GCP 대시보드가 나타날 거에요. 그럼 오른쪽 상단에 있는 [>_] 와 같이 생긴 작은 Activate Cloud Shell 버튼을 클릭하세요.
 ![cloud shell](https://cdn-images-1.medium.com/max/1440/1*X9XC4D-zQLXDTrWPw9csYw.png)
 **참고** 만약 구글 클라우드 계정을 생성한 후 Cloud Shell을 처음 시작 할때에는 몇분의 시간이 걸립니다.

터미널 창은 아래와 같이 같은 페이지에 열립니다.
![cloud shell2](https://cdn-images-1.medium.com/max/1440/1*zswXHm5sxmmy5sIj5x60BQ.png)
터미널에서 아래와 명령어를 실행하여 google app engine application을 생성 합니다.
```
gcloud app create
```
그런 다음 지리적을 가까운 지역(region)을 선택하시고 몇분 후에 "_“Success! the app is now created_" 메세지를 확인하면 성공적으로 생성 된것 입니다.  처음 app을 배포 하기 위해서 ‘gcloud app deploy’를 사용해 주세요.
![cloud shell3](https://cdn-images-1.medium.com/max/1440/1*mjRaAbLgGbPxcv2Fzu8YVA.png)
위에서 업로드 하였던 repo에서 다운로드 받습니다.  예로 여기서는 fast.ai google cloud engine starter pack 입니다 :
```
git clone https://github.com/pankymathur/google-app-engine
```
다운 받은 폴더로 이동 해주세요 :
```
cd google-app-engine
```
자신의 app을 Google App Engine에 배포합니다 :
```
gcloud app deploy
```
“Services to deploy” 메세지가 표시되면 "Y"를 입력하세요
![deploy app](https://cdn-images-1.medium.com/max/1440/1*V2drMPZjBsHHh73wctN1cA.png)
app engine을 도커 기반 app을 배포 하고 app의 URL을 제공하는데 까지는 8-10분의 시간의 소요 됩니다.
 
### app의 URL 테스트
배포한 app이 잘 작동하는지 확인 하기위해서  http://YOUR_PROJECT_ID.appspot.com 아래의 주소로 접속하거나 브라우져 내의 쉘(GCP Cloud Shell)에서 아래에 명령을 입력하세요. 그럼 새로운 브라우져 창과 함게 App 열립니다. 
```
gcloud app browse
```

## PC에서 테스트하기 <span id="localTest"></span>
App 서버를 로컬에서 실행하거나 위의 단계를 변경하려는 경우 :
```
python app/server.py serve
```
그리고 테스트를 위해 http://localhost:8080/ 여기 주소로 이동하세요.

>  샘플 코드를 제공한 Simon Willison에게 감사를 표합니다.
