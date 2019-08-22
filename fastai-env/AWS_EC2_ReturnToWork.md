# AWS EC2 작업재개

## 목차

- [단계별 가이드 : 작업재개](#stepbystep)
-  [1. instance 시작하기](#startinginstance)
-  [2. 코스 저장소 업데이트](#updaterepo)
-  [3. fastai 라이브어리 업데이트](#updatefastai)
-  [4. instance 정지](#stoppinginstance)


## 단계별 가이드 : 작업재개<span id="stepbystep"></span>

### 단계1 : instance 시작하기<span id="startinginstance"></span>
[AWS console](https://aws.amazon.com/console/)에 로그인 하여 EC2 링크를 클릭하세요(이는 이전 작업 내용 또는 창의 왼쪽에서 'Service'에서 찾거나 검색창에 EC2를 검색 가능합니다).  이후 페이지에서 왼쪽 메뉴의 ‘Instances’ 또는 ‘Running Instances’  클릭하세요.

![AWS instancel](https://course.fast.ai/images/aws/instance.png)

시작할 instance의 체크박스를 체크하고 'Actions' -> 'Instance State'를 'Start'를 눌려 주세요.

![AWS instance start](https://course.fast.ai/images/aws/start.png)

'Instance Type'이 정지('stopped')되어있는 동안에는 'Instance Settings'에서 머신의 타입을 변경 가능 합니다. 이는 사양이 낮은 머신(저렴)에서 테스트를 마친 후 우수한 GPU(비쌈)가 있는 환경으로 바꾸어 작업 할 수 있도록하는 장점을 가지고 있습니다.

'Instance State'가 주황색인 경우 초록색으로 변경 될때까지 약간의 시간이 소요 됩니다.

![AWS instance pending](https://course.fast.ai/images/aws/pending.png)


상태가 초록색으로 바뀌면, IPv4 칼럼에 있는 IP를 복사하세요.

![running status](https://course.fast.ai/images/aws/pubdns.png)

이제 자신의 컴퓨터에서 터미널을 열고 아래의 명령어를 입력하세요(IP_ADDRESS 부분은 위에서 복사한 IP로 변경).
```
ssh -L8888:localhost:8888 ubuntu@IP_ADDRESS
```
만약 코스의 레파지토리 또는 라이브러리를 업데이트 하고 싶다면 지금 단계에서 하시면 됩니다. 준비가 되었다면 아래의 명령어를 입력하세요. 
```
jupyter notebook
```
그 다음 [localhost:8888](http://localhost:8888/)를 통하여 nodebook에 접속 가능 합니다.

### 단계2 : 코스 레파지토리 업데이트<span id="updaterepo"></span>
강의를 듣다가 보면 git의 레파지토리가 최신으로 업데이트 되는 경우가 있습니다(업데이트가 빈번히 이러나고 있습니다). 그렇기 때문에 우리가 clone 했던 레파지토리도 업데이트를 해주어야 합니다. 그러기 위해서는 PC의 터미널을 여시고 아래의 두 명령어를 입력하세요.
 ```
cd course-v3
git pull
```

![repo update](https://course.fast.ai/images/gradient/update.png)

위 명령어를 실행 후에 코스에서 필요한 최신의 notebook들로 변경 될것입니다. 만약 'course-v3/nbs'에 수정을 직접하였다면 GitHub는 에러를 발생 시킵니다. 이를 해결하기 위해서는 ``git stash `` 명령어를 입력하여 PC에서 변경한 사항을 제거해야합니다. `` git pull`` 하기 전에 수업에서 수정하거나 개인적으로 유지하고 싶은것이 있다면 따로 저장 해야합니다. 그렇지 않으면 삭제가 될 수 있습니다. 

### 단계3 : fastai 라이브어리 업데이트<span id="updatefastai"></span>
fastai 라이브어리를 최신으로 업데이트 하고 싶다면 아래의 명령어를 터미널에 입력하세요.
```
conda update conda
conda install -c fastai fastai
```



### 단계4 : instance 중지<span id="stoppinginstance"></span>

추가 요금이 청구되는 사항을 피하고 싶다면 작업이 끝난 후에는 [AWS console](https://us-west-2.console.aws.amazon.com/ec2)로 돌아가서 instance를 중지 시키는것을 항상 해야만 합니다.  좋은 방법으로는 컴퓨터를 종료하거나 로그오프 할때에 알람을 설정하여 잊지 않도록 하는것은 좋은 습관입니다.

instance를 중지 하기 위해서 원하는 instance의 체크 박스에 체크를 하고 'Actions' -> 'Instance State' -> 'Stop'을 선택하세요. 

![stop status](https://course.fast.ai/images/aws/stop.png)

**주의 : instance의 'Instance state'가 'running' 중에는 요금이 부과 됩니다.**
