# AWS BeanStalk배포

## 목차

-  [AWS BeanStalk start pack 가져오기](#GrabAWS)
-  [프로젝트 설정](#updaterepo)
-  [배포](#updatefastai)
-  [ PC에서 테스트](#stoppinginstance)

## AWS Elastic BeanStalk 배포
이 문서는 AWS Elastic Beanstalk을 사용하여 학습된 모델을 배포하는 간략한 가이드 입니다. Lesson2에서 Jeremy’s Bear 이미지를 분류하는 배포한 starter app과 함께 제공 됩니다.

## AWS BeanStalk start pack 가져오기

```
wget https://github.com/fastai/course-v3/raw/master/docs/production/aws-beanstalk.zip

unzip aws-beanstalk.zip

cd app
```

## 프로젝트 설정
**학습된 모델 업로드**

Google Drive 또는 Dropbox와 같은 클라우드 서비스에 학습된 모델 파일을 업로드(예: stage-2.pth) 하세요. 그리고 파일의 다운로드 링크를 복사하십시오.  **주의:**  다운로드 링크는 일반적으로 사용되는 뷰를 제공하는 공유링크와는 다른, 파일에 링크가 직접적으로 연결 되어있는 링크([Direct download link](https://en.wikipedia.org/wiki/Direct_download_link))를 말합니다. 필요시 아래의 주소에서 링크를 변경 할 수 있습니다. (공유링크 -> 직접 다운로드 링크로 변경 :  [https://rawdownload.now.sh/](https://rawdownload.now.sh/)) 만약 배포 테스트만을 원한다면, Jeremy’s 곰 분류 모델이 기본적으로 설정 된 모델을 그대로 사용하면 됩니다. 모델의 가중치 URL은 이미 샘플 App에 기입 되어 있으므로 이 단계를 건너뛸 수 있습니다.

  

**App을 나의 모델로 변경하기**

app 폴더에 있는  `server.py`를 편집기를 통하여 열고  `model_file_url`  이 부분을 위에서 복사 해둔 url로 변경해주세요. 같은 파일에서  `classes = ['black', 'grizzly', 'teddys']` 에 부분 클래스 내용을 모델에서 예상하는 것으로 변경하세요.

## 배포
시작을 위해서, AWS Elastic Beanstalk Console을 열고 **_Create New Application_**을 클릭해 주세요. 그 후 우리의 어플리케이션이름과 상세내역을 입력하시고 "create"를 클릭합니다. 그럼 환경에 대한 내용이 스크린에 나타날것 입니다. 
![createl](https://cdn-images-1.medium.com/max/1600/1*quAQHRvOIMAk0Mk65HZFlw.png)
이제 위 화면에서 보이는 **_Create one now_**를 클릭 해주세요, 그럼 아래와 같은 화면이 나타납니다.
![create one now](https://cdn-images-1.medium.com/max/1600/1*lHQAyoAtdvAgVViIPvLikg.png)

**_Web Server Environment”_** 를 선택하세요 그리고 아래와 같은 웹서버 환경에 대한 자세한 내용을 확인 할 수 있습니다.
![create one now](https://cdn-images-1.medium.com/max/1600/1*XdBeWqjKIXi8NR2GRrG5lg.png)
**_Environment name_** 을 입력, **_Domain_** 을 선택, **_Description_** 의 내용을 입력을 해줍니다. 

**중요 : ** Base configuration 부분에서는 **_Preconfigured Platform_** 을 선택하고 **_Docker_**를 선택하세요.
![_Preconfigured ](https://cdn-images-1.medium.com/max/1600/1*vn8LQgQhcmjC4rAwGPPGUA.png)
스크롤을 아래로 내려 좀 더 자세한 내용을 입력해 줘야합니다. 
![_Preconfigured ](https://cdn-images-1.medium.com/max/1600/1*djdzftgYq0GVJTCrZ32SnQ.png)
**_Upload our code_**를 선택하고 Upload버튼을 클릭합니다. 그럼 아래와 같은 화면이 나타납니다. start pack을 업도르 하기 전에 **아래에 있는 지침을 읽어 주세요!**

![uploadpage](https://cdn-images-1.medium.com/max/1600/1*dSCUOfueGR9x1Yvc9wtSiw.png)
몇가지 특이한 이유로 우리는 web app 폴더를 압축하여 업로드를 할 수있습니다. 그러기 위해서는 web app폴더에 있는 파일들을 직접 압축해야합니다.  이유를 추측하자면, AWS Beanstalk는 web app 폴더에서가 아닌 압축된 파일 내에서 Dockerfile을 찾아야 하기 때문입니다.  이렇게 하기 위해서는 PC에서 web app 폴더로 들어가서 거의 있는 파일들을 압축하세요. Mac에서는 위와 같이 시행 했습니다.
![zip ](https://cdn-images-1.medium.com/max/1600/1*gteqrx77ZiN2_931tlcyQQ.png)
**옵션 : ** 만약 PC에서 “server.py ”에서 실행하였다면  여기 경로에 app > static > model > “model.h5” or “model.pth” 와 같은 파은 파일이 있을 것입니다.  압축 파일의 크기를 작게 하기 위해서 이 파일을 삭제하여도 무방합니다. 참고로 AWS BeanStalk는 업로드 가능한 최대 크기가 512MB 입니다.
**중요 :** **_Create Environment_**를 버튼을 누루지마세요! **_Configure More Options_**를 설정해야 합니다.

![severenvironment](https://cdn-images-1.medium.com/max/1600/1*5NFRFo5p3ZOduJHFOkQewA.png)
이제 여기부터 중요한 내용이 나옵니다,  AWS Beanstalk에서 쉽게 우리의 Web app를 배포를 위해서 **_Configure More Options_** 클릭 하세요. 그럼 새로운 페이지가 열립니다. **_Configuration presets_** 섹션에서 **_Custom Configuration_** 를 선택합니다. 그럼 아래 화면과 같이 보일것 입니다.
![confugureationpresets](https://cdn-images-1.medium.com/max/1600/1*eWV0eihm4CusaFz7b0dogw.png)
첫번째 섹션에서 **_Software_**를 찾고 거기에 있는 **_Modify_** 를 클릭하세요.  그럼 아래와 같은 페이지가 열릴것 입니다. **_Proxy server_** 를 "Nginx”에서 **_None_** 으로 변경 해줍니다. 
그리고 SAVE를 누르고 페이지에서 나갑니다.
![_Proxy ](https://cdn-images-1.medium.com/max/1600/1*ONxjzSZGhCq459dkyZROhw.png)
Save를 하고 나면 다시 Configuration page지가 나타 날것입니다.  Section > **_“Instances”_** 를 찾아서 **_“Modify”_** 를 눌러 주세요.  그럼 아래와 같은 페이지가 보입니다. “**_Instance type_**” 에서 “t1.micro” 를 “**_t3.small_**” 로 변경하세요 그리고 save를 눌려 줍니다.
![t3.small](https://cdn-images-1.medium.com/max/1600/1*fOlaVJrC1XN708Cb1zkSWQ.png)
위와 같이 custom configuration를 설정을 했다면 이제 **_Create Environment_** 버튼을 눌려주세요. 이후 화면에서 AWS BeanStalk 배치에 관한 모든 로그을 보여줄 것입니다. 
![log](https://cdn-images-1.medium.com/max/1600/1*LStNH3er7EDPDHeHbElA8g.png)
10-15분 후, 아래와 같은 대쉬보를 볼 수 있습니다. 상태를 "초록색"으로 표시하고 단계별 자세한 로그 또한 보입니다. 
![status](https://cdn-images-1.medium.com/max/1600/1*yylhjLktaLDqGPnVAty0Yg.png)

**APP 작동 테스트 URL**
최종으로 APP을 확인 할려면, http://YOUR_APP_NAME.AWS_REGION_HERE.elasticbeanstalk.com 을 열거나 대쉬보드에 있는 URL을 클릭하세요.

## PC에서 테스트
App Sever를 PC에서 실행하거나 위의 단계를 변경하려는 경우:
```
python app/server.py serve
```
App 테스트를 위해 http://localhost:8080/ 주소로 접속하세요.


>  샘플 코드를 제공한 Simon Willison에게 감사를 표합니다.
