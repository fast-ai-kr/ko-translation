# AWS EC2

## 목차
- [가격](#price)
- [단계 1: 로그인 또는 가입](#login)
- [단계 2: 서비스 제한 설정](#RequestServicelimit)
- [단계 3: ssh 키 생성 및 업로드](#sshkey)
- [단계 4: instance 시작](#launchinstance)
- [단계 5: instance에 접속](#connectioninstance)
- [단계 6: fast.ai  설치](#installfastai)
- [단계 7: instance 종료](#doneinstance)
- [References](#refer)

## AWS EC2에 오신것을 환영 합니다! 
AWS EC2에서는 [DLAMI](https://aws.amazon.com/machine-learning/amis/)라고하는 사전구상된 이미지가 제공하며, 이는 아마존에서 제공하는 딥러닝을 위한것 입니다. DLAMI를 사용한다고하더라도 AWS EC2 instance를 세팅한다는것은 부담스러울수 있습니다. 하지만 아래에 내용이 있으니 걱정하지마세요. 사실 아마존에서 instance를 세팅하는 자세한 [단계별 가이드](https://aws.amazon.com/getting-started/tutorials/get-started-dlami/)가 있으며,  그것으로 부터 많은 것을 참고 하였습니다. 
만일 작업으로 복귀하고 싶거나 아래의 단계를 완료하였다면, [복귀 작업](https://course.fast.ai/update_aws.html) 단계로 이동하십시오.

## 가격 <span id="price"></span>
우리가 제안하는 p2.xlarg instance의 경우 [시간당 $0.9 비용](https://aws.amazon.com/ec2/instance-types/p2/)이 청구 됩니다.

## 단계 1 : 로그인 또는 가입<span id="login"></span>
[AWS 웹페이지](https://aws.amazon.com/)에 방문하여 'Sign In to the Console'을 클릭하세요

![AWS sing in to the consol](https://course.fast.ai/images/aws/signin.png)

만약 계정이 없으시다면, 'Sign In to the Console' 대신에 'Sign up' 버튼이 보입니다.

![AWS sing ip](https://course.fast.ai/images/aws/signup.png)

다음으로 로그인 하고자 한다면 당신의 ID(mail)과 비밀번호를 입력하시고, 만약 가입하고 있다면 신용카드에 대한 내용을 입력해야 합니다. instace를 사용에 대한 요금이 등록한 신용카드로 청구가 됩니다.( 만약 free credit이 있다면 그것을 넘기전에는 요금이 청구되지 않습니다).  또한 당신의 신원을 확인하기 위해 통화가능한 전화번호를 제공해야 한다는 점에 유의하십시오.

## 단계 2 : 서비스 제한 설정<span id="RequestServicelimit"></span>
방금 계정을 생성한 경우 본 코스에 필요한 instance 유형의 제한을 요청하십시오.(기본값 : 0)
먼저 'Services'를 클릭한 다음 'EC2'를 클릭하십시오.

![ClickEC2](https://course.fast.ai/images/aws/ec2.png)

그런 다음 왼쪽 창에서 Limits를 선택한 다음 p2.xlarge를 찾을 때까지 목록을 찾아 내려가세요. limit가 한개 또는 그이상이라면 이번단계를 건너 뛰어도 됩니다. 그것이 아니라면 그렇지 않으면 'Request limit increase'을 클릭 하세요.

![setlimit](https://course.fast.ai/images/aws/increase_limit.png)

'use case description box'에는 ‘[FastAI] Limit Increase Request’라고 입력하고 선호하는 언어와 연락방법를 선택해주세요. 그리고 'Submit'버튼을 눌려 체출합니다. 그럼 요청한 내용을 검토한다는 응답을 받고 대략 두시간 내에 승인 통지를 받습니다.

 ![submit](https://course.fast.ai/images/aws/increase_limit2.png)
 
승인을 기다리는동안 단계3을 진행하세요.


## 단계 3 : ssh 키 생성 및 업로드 <span id="sshkey"></span>
이번 단계에서는 터미널을 필요로 합니다. 그렇기에 윈도우 환경에서는 추가적인 설치가 필요합니다. [여기](https://course.fast.ai/terminal_tutorial.html)를 참고하세요.

터미널에 'ssh-keygen'을 입력한 다음 return을 세 번 누르십시오.  그럼 ./ssh 폴더 속에 'id_rsa'와 'id_rsa.pub'라는 두개의 파일이 생성되어 있을 것입니다. 'id_rsa'는 안전하게 보관해야하는 private key이고 'id_rsa.pub'는 public key로서 통신할 누군가에게 전달해야하는 key입니다. (이번 경우에는 전달해야할 대상은 AWS입니다.)

윈도우 환경에서는 public key를 윈도우 폴더로 복사하여 쉽게 접근할수 있습니다. ( WSL home 폴더에 기본적으로 생성)  아래에 명령어가 public key를 temp의 폴더로 복사하는 명령어입니다. temp대신 원하는 경로를 변경하여 사용할수 있습니다.
```console
cp .ssh/id_rsa.pub /mnt/c/Temp/
```
만약 ssh key를 만들었다면, AWS Console로 돌아가서 서비스 제한 증가를 요청한 Region에 있는지 확인하세요.  Console의 웹 주소를 보면 자신위치를 확인 할 수 있습니다. 예를 들어 https://us-west-2.console.aw.amazon.com 는 오리건 지역이고  https://ap-south-1.console.aws.amazon.com/ 는 뭄바이 지역입니다.  화면의 상단 오른쪽 모서리에 username 옆에 있는 드롭박스에서 원하시는 Region을 선택 가능 합니다.

다시 'Services'를 클릭한 다음 'EC2'를 클릭하세요.

![AWS EC2](https://course.fast.ai/images/aws/ec2.png)

쿼리 바를 통하여 검색도 가능합니다. 왼쪽 메뉴에서 'Key pairs'을 찾은 다음 클릭하세요.

![key pair](https://course.fast.ai/images/aws/key_pair.png)

새로운 창에서:

 1.  ‘Import Key Pair’ 버튼을 클릭하세요.
 2.  ' id_rsa.pub'를 선택해 주세요 . ('.ssh' 폴더 또는 복사된 폴더 중 하나에서 찾을 수 있습니다.)
 3. 원하는 경우 key의 이름을 변경하고  ‘Import’를 클릭하세요.
 
## 단계 4: instance 시작 <span id="launchinstance"></span>

만약 p2 instance를 승인 받지 못하였다면 마지막 단계에서 진행이 불가능하므로 시작하기전에 조금 기다려야 함을 유의하세요.

 AWS Console에 로그인한 다음 쿼리 바에서 'EC2'를 검색을 하거나 'Service'에서 'EC2'를 클릭하세요. 그다음  EC2화면에서 'Launch instance' 버튼을 클릭하세요.

 ![Launch instance](https://course.fast.ai/images/aws/launch_instance.png)
 
‘deep learning’을 검색하여 첫번째 옵션( Deep Learning AMI (Ubuntu) Version 16.0)을 선택하세요. 

![Deep Learning AMI ](https://course.fast.ai/images/aws/amiubuntu.png)

p2.xlarge'를 찾을 때까지 아래로 스크롤 이동하여 선택하세요. 그리고 ‘Review and Launch’ 버튼을 누르세요.

![Review and Launch ](https://course.fast.ai/images/aws/p2.png)

마지막으로 'Review' 탭에서 'Launch'버튼을 누르세요.

![Review tab Launch ](https://course.fast.ai/images/aws/launch.png)

팝업 창의 첫 번째 드롭다운 메뉴에서 2단계에서 생성한 key를  선택한 다음 선택한 private key로 액섹스 할수 있음을 확인하는 체크박스에 체크 해줍니다. 그런 다음 ‘Launch Instance’ 버튼을 클릭 합니다.

![keypair ](https://course.fast.ai/images/aws/key.png)


## 단계 5: instance에 접속<span id="connectioninstance"></span>

다음 창에서 스크롤을 아래로 내려보면 ‘View Instances’버튼을 확일 할 수 있습니다. 클릭해주세요. 'Instance State'에 상태가 'running'이라 보이는 instance가 있음을 확인 할 수 있을 것입니다. Amazon은 instance가 구동되는 시간만큼 요금을 부과하므로 추가 요금을 내지 않으려면 **instance의 사용을 마치면 항상 중지**해야 합니다. 단계 7에서 좀 더 자세히 설명 할것 입니다.

아래에서와 같이 instance의 상태가 주황생으로 나타나는 경우 instance가 준비될 때까지 조금 기다려야 합니다.

![pending](https://course.fast.ai/images/aws/pending.png)

상태가 초록색으로 바뀌면, IPv4 칼럼에 있는 IP를 복사하세요.

 ![running status](https://course.fast.ai/images/aws/pubdns.png)

이제 연결을 해볼 시간입니다! OS의 command line [terminal](https://course.fast.ai/terminal_tutorial_)을 열고 아래의 명령어를 입력하세요.
```console
ssh -i ~/.ssh/<your_private_key_pair> -L localhost:8888:localhost:8888 ubuntu@<your instance IP>
```
(< your instance IP> 부분을 위에서 복사 해둔 IP로 변경해주세요. 또한   'your_private_key_pair'가 아닌 'your_private_key_pair.pub'를 사용 하고자 함에 유의하세요.)

IP 주소에 대해서 신뢰성에 대한 묻는 질문을 받게 될것입니다. 그때 'yes'라고 답변해야합니다.

## 단계 6: fast.ai 설치<span id="installfastai"></span>

아래 명령어를 실행하세요.
```
git clone https://github.com/fastai/course-v3
```
fast.ai에 대한 도구들이 있는 폴더를 다운 받습니다.
그런 다음 아래의 명령어를 실행하여  fast.ai와 PyTorch를 사용하는데 필수적인 패키지를 설치합니다. 
```
conda update conda
conda install -c pytorch -c fastai fastai pytorch torchvision cuda92
```
그런 다음 아래의 명령을 실행하여 코스에서 필요한 도구가 있는 폴더로 이동을 해줍니다.
```
cd course-v3/nbs/dl1
```
마지막으로 아래의 명령어를 실행하세요.
```
jupyter notebook
```
 이제 터미널에서 localhost:8888의 주소로 새로운 웹 창이 뜨면서 접속이 되었을 것입니다 이로 노트북에 접속 할수 있게 된것입니다. .   
만약 localhost:8888이 작동하지 않는다면, 터미널 창으로 돌아가서 ‘jupyter notebook’을 입력한 후 아래와 같은 메세지를 찾아보세요 “Copy/paste this URL into your browser when you connect for the first time, to login with a token:”

해당 URL을 복사하여 브라우저에 붙여넣고 해당 주소로 이동을 하면 jupyter notebook에 연결이 됩니다. 

jupyte notebook에 대해서 자세히 알고 싶다면 [여기](https://course.fast.ai/index.html)로 이동하여 jupyter notebook 튜토리얼을 실행하세요.  작업이 끝나면 instance를 중지하는것을 잊지마세요.

fastai 라이브러리를 사용하는 동안 문제가 있으면 아래 명령어를 실행해 보세요.
```
conda install -c fastai fastai
```

## 단계 7: instance 종료<span id="doneinstance"></span>

추가 요금이 청구되는 사항을 피하고 싶다면 작업이 끝난 후에는 [AWS console](https://us-west-2.console.aws.amazon.com/ec2)로 돌아가서 instance를 중지 시키는것을 항상 해야만 합니다.  좋은 방법으로는 컴퓨터를 종료하거나 로그오프 할때에 알람을 설정하여 잊지 않도록 하는것은 좋은 습관입니다.

instance를 중지 하기 위해서 원하는 instance의 체크 박스에 체크를 하고 'Actions' -> 'Instance State' -> 'Stop'을 선택하세요. 

![stop status](https://course.fast.ai/images/aws/stop.png)

다시 instance에서 작업이 필요하거나 코스의 내용이나 fastai를 업데이터 하고 싶은 경우 [여기(작업재개)](https://course.fast.ai/update_aws.html)를 참고하세요.

여기서 주의해야 할점으로는 Terminate을 누르지 말고  stop버튼을 눌려야합니다. Terminate버튼은 instance가 완전히 제거 되기에 하던 작업 또한 같이 제거 됩니다.

## 참고<span id="refer"></span>
https://aws.amazon.com/getting-started/tutorials/get-started-dlami/통신하고자하는 사람에게 건내줘야하는 public key입니다.(우리의 경우에는 AWS에게 건내주어야 합니다.)



