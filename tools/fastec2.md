# fastec2: 평범한 사람들을 위한 AWS 컴퓨터 관리
> 작성일: 2019년 2월 15일, 작성자: Jeremy Howard

_이 글은 fastec2 에 대한 시리즈 중 파트1에 해당하는 것이다. fastec2를 사용하여 오랫동안-수행되는 작업을 실행하고 모니터링하는 법을 배우고 싶다면 파트2를 확인해 보기 바란다._

AWS EC2는 매우 훌륭한 시스템이다. 누구든지 시간당 약간의 돈으로 컴퓨터를 빌릴 수 있게 해주는데, 이때 빠른 네트워크 연결뿐만 아니라 많은 디스크 공간도 함께 포함된다. 나는 특히 AWS에 감사하는데, 그 이유는 fast.ai에서의 연구/개발을 위해 사용할 수 있는 compute 크레딧을 많이 공급해준 [Activate](https://aws.amazon.com/activate/) 프로그램이 있었기 때문이다. 

하지만, AWS EC2를 이용해서 작업해본 적이 있다면, 설정 과정에서 느리고 복잡한 AWS 콘솔 GUI와 장황하면서도 투박한 명령어 인터페이스(CLI)에서 꽉 막힘을 느꼈을지도 모른다. AWS 관리를 간소화하기 위한 여러 가지 툴이 있기는 하지만, 파워유저들을 위한 경우가 많으며 복잡한 아키텍처에서 여러 개의 컴퓨터를 배포하기 위한 사람들을 위해 작성되어 있다.

평범한 사용자를 위한 툴은 어디에 있는가? 평범한 사용자라는 것은 하나 또는 두 개 정도의 컴퓨터를 실행시켜서 몇 작업을 수행하고, 작업 완료 후 이를 종료하길 원하는 사람들이다. 또한, 평범한 사용자라는 것은 여러 가지 AWS에 특화된 VPC와 Security Group, IAM Role 등과 같은 전문용어까지 배우고 싶지 않은 사람들을 의미한다.

![ec2_template](https://www.fast.ai/images/ec2_template.png)

## 목차
- [오버뷰](#overview)
- [설치 및 설정](#installation-and-configuration)
- [첫 번째 요청에 의한 인스턴스를 생성 해보기](#creating-your-initial-on-demand-instance)
- [아마존 머신 인스턴스(AMI)를 생성 해보기](#creating-your-amazon-machine-instance-ami)
- [인스턴스의 실행 및 연결 해보기](#launching-and-connecting-to-your-instance)
- [Spot 인스턴스를 실행 해보기](#launching-a-spot-instance)
- [대화형 REPL과 ssh API를 사용 해보기](#using-the-interactive-repl-and-ssh-api)

나 자신도 매우 평범한 사람이기 때문에, 이러한 툴을 작성하는 것이 좋을 것이라고 생각했다. 그리고 그 툴은 [fastec2](https://github.com/fastai/fastec2/) 이다. 이 툴이 여러분들을 위한 것인가? 다음은 이 툴이 간소화하고자 하는것에 대한 요약을 보여준다 (여기에서 `instance`란, 단순히 ‘AWS상의 컴퓨터’를 의미한다):

- 요청에 의한, 또는 임시성(spot) 인스턴스를 실행
- 어떤 인스턴스들이 실행 중인지 확인
- 인스턴스를 실행
- ssh를 이용하여 이름이 부여된 인스턴스로 접속
- 오랫동안-실행하는 스크립트를 spot 인스턴스에서 실행, 모니터링, 그리고 그 결과를 저장
- 자동 포매팅(구성)/마운팅이 포함된 볼륨과 스냅삿을 생성하고 사용
- 인스턴스의 종류를 바꾸기 (e.g. GPU를 추가하거나 삭제하기)
- 요청에 의한, 또는 spot 인스턴스 종류에 대한 가격정책 확인
- 보통의 명령어창 또는 Jupyter를 통한 접근
- Jupyter Notebook API
- `Tab` 키를 이용한 자동완성
- 더 많은 내용을 탐색하기 위한 대화식의 REPL IPython 명령어

이 툴은 데이터분석, 데이터수집, 그리고 머신러닝 모델의 학습을 하고자 하는 사람들에게 가장 유용할 것이라고 예상한다. fastec2는 복잡한 네트워크 아키텍처에서의 거대한 서버 함대의 관리를 쉽게 하려고 디자인 된 것이 아니고, 애플리케이션 개발을 돕기 위한 것도 아니다. 이러한 것을 원한다면, [Terraform](https://www.terraform.io/) 또는 [Cloud Formation](https://aws.amazon.com/cloudformation/) 을 확인해 보길 권장한다.

이 툴이 동작하는 방식을 확인하기 위해서 새로운 아마존 머신 이미지(AMI)를 생성하고, 그 인스턴스로부터 AMI를 실행하고, 연결하는 완전한 방법을 알아보도록 한다. Spot 인스턴스를 실행하기 위한 방법과 그 인스턴스에서 오랫동안-수행되는 스크립트를 실행하고, 스크립트의 결과를 수집하는 방법 또한 알아보게 될 것이다. 나는 여러분들이 이미 AWS 계정을 가지고 있고, ssh를 이용하여 인스턴스로 연결하기 위한 기본내용을 알고 있다고 가정하고 진행할 것이다. fastec2의 여러 가지 멋진 기능들은 [Fire](https://github.com/google/python-fire/), [Paramiko](http://www.paramiko.org/), [boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html) 와 같은 라이브러리에서 제공되고 있다. 이를 가능하게 해준 모든 분께 감사를 드리는 바이다.

<div id="overview"></div>

## 오버뷰

fastec2이 주로 사용되었으면 하는 사용의 예는 다음과 같다: 다양한 종류의 머신을 대화식으로 시작하고 멈추길 원하지만, 매번 동일한 프로그램, 데이터, 그리고 설정이 자동으로 적용되어 있기를 원한다. 가끔은 요청에의한 이미지를 생성, 실행, 그리고 필요에 따라 멈출 수도 있을 것이다. 또한, 인스턴스의 종류를, GPU를 추가하거나 RAM 용량을 늘리는 등 변경하길 가끔씩 원할 수도 있을 것이다. (단일 명령어로, 즉각적으로 이뤄질 수 있는 부분이다!) 그리고, 가끔은 (머신러닝 모델의 학습이나 웹 스크래핑과 같은 작업을 위한) 스크립트 실행 및 결과를 저장하기 위한 spot 인스턴스를 실행할 수도 있을 것이다.

이러한 작업을 잘 해내기위한 주요 요소는 여러분이 딱 필요한 만큼만 AMI를 설정하는 것이다. AMI란 아마존에 있는 시스템 운영자(sysadmin)가 여러분을 위해 만들어 주는것 쯤으로 생각해 볼 수 있는데, 실제로 매우 쉽고 빠르게 가능하다는 것을 나중에 알게될 것이다. AMIs의 생성과 사용을 쉽게 만들어 놓음으로써, 필요할 때 여러분이 필요한 머신을 손쉽게 생성할 수 있는 것이다.

fastec2에 있는 모든것들은 AWS 콘솔을 통해서, 그리고 공식 AWS CLI를 통해서도 가능한 것이다. 덧붙여서, fastec2이 할 수 없는 많은 일들도 존재한다. 하지만, 이 일들에 대한 작업이 완료될 것이라는 것은 아니다. 다만, fastec2는 공통적으로 사용되는 대부분의 기능들을 위해 편리성을 제공하기 위한 것이다. fastec2이 제공하는것엔 무엇이 있는지 발견해보는 기회를 가져보고, 그것들이 현존하는 다른 무엇들보다 손쉽고 빠른 환경을 제공하길 바란다.

<div id="installation-and-configuration"></div>

## 설치 및 설정

Python3.6 또는 이후의 버전이 필요하다. 아직까지 Python3.6을 사용하고 있지 않다면, [Anaconda](https://www.anaconda.com/distribution/)를 설치하기를 강하게 권장한다. 그렇게 함으로써, 원하는 만큼의 여러가지 Python 버전에 대한 환경의 설치가 가능하고, 필요에 따라서 각기 다른 환경간 변경을 할 수 있다. fastec2을 설치하기 위해서:

```
pip install git+https://github.com/fastai/fastec2.git
```

쉘에서 `Tab`키-자동완성을 설치하면 약간의 시간이 절약될 수도 있다. [README](https://github.com/fastai/fastec2/blob/master/README.md)를 확인해 보면, `Tab` 키 자동완성 설치에 대한 방법을 알 수 있을 것이다. 설치가 완료되면, `Tab` 키를 언제든지 눌러서 명령어를 완성하거나, 한번 더 `Tab` 키를 눌러서 가능한 다른 명령들을 확인해 볼 수도 있다.

fastec2은 작업의 수행을 위해서 [AWS CLI](https://aws.amazon.com/cli/)와의 인터페이스로, Python을 사용한다. 따라서, 이 부분을 설정해 줘야한다. CLI는 리전(Region)에 대한 이름 대신, 코드를 사용한다. 사용하고자하는 리전에 대한 리전코드를 알아내는 부분을 fastec2이 도와줄 수 있다. fastec2 어플리케이션을 실행하기 위해서, fe2 를 타이핑 하고, 이어서 명령어 이름과 필요한 인자값을 나열한다. `region` 이라는 명령어는 사용자가 입력한 (대소문자가 구분되는)문자열에 일치하는 가장 첫번째 리전코드를 출력해 준다. 예를 들어서 (`$`를 사용하여 여러분이 타이핑하는 명령줄을 표현하고 있고, `$`이 없는 줄은 응답을 표시하고 있다):

```
$ fe2 region Ohio
us-east-2
```

이제 리전코드를 가지고 있으니, 이를 이용해서 AWS CLI를 설정할 수 있다:

```
$ aws configure
AWS Access Key ID: XXX
AWS Secret Access Key: XXX
Default region name: us-east-2
```

AWS에 대한 Access Key를 얻어오는 내용을 포함한, 더 많은 정보가 필요하다면 [AWS CLI 설정](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)을 참조하기 바란다.

<div id="creating-your-initial-on-demand-instance"></div>

## 첫 번째 요청에의한 인스턴스를 생성 해보기

적절한 소프트웨어가 설치되어 있고, 필요한 데이터파일이 다운로드 되어 있으며, 이를 사용하기 위한 설정이 원하는대로 이뤄진 새로운 인스턴스를 신속하게 만들 수 있다면, 삶이 훨씬 쉬워질 것이다. 이는 AMI라는 것을 생성하여 가능한데, AMI는 일종의 여러분이 미리 설정해둔 "결빙된" 버전의 컴퓨터로, 원하는 횟수만큼 재생성이 즉각적으로 가능하다.

따라서, 우선은 EC2 인스턴스를 필요한 만큼으로 만들게 될 것이다 (이것을 베이스 인스턴스라고 부를 것이다). (이미 설정해둔 인스턴스가 있다면, 이 부분을 건너뛰어도 좋다).

한가지 알아두면 일을 좀 더 쉽게 만들어줄 것이 있는데, AWS에서의 "default"라는 이름의 키페어(key pair)가 있음을 확인해 두는 것이다. 만약 키페어가 없다면, "default" 라는 이름으로 업로드 하거나, 하나 만들어 두어야 한다. 비록 fastec2은 다른 이름의 키를 사용할 수 있지만, 그 키에 대한 이름을 매번 지정해 주어야 한다. 나중에 AMI를 사용하여 새로운 인스턴스를 실행할 때 디스크 크기를 더 크게 바꿀 수 있기 때문에, 베이스 인스턴스에 대한 디스크 크기를 매우 크게 잡아둘 필요는 없다. 일반적으로 **60GB** 정도면 적당한 크기라고 볼 수 있다.

베이스 이미지를 생성하기 위해서, 리눅스 배포본을 포함하는 이미 존재하는 AMI를 가지고 시작해야 할 것이다. 여러분이 이미 선호하는 AMI를 가지고 있다면, 그것을 사용해도 좋다. 그렇지 않다면, 가장 최신의 안정화된 버전의 Ubuntu 이미지의 사용을 권장한다. 최신의 Ubuntu 에 대한 AMI ID를 알고 싶다면, 다음과 같이 타이핑해볼 수 있다:

```
$ fe2 get-ami - id
ami-0c55b159cbfafe1f0
```

위의 명령은 fastec2의 강력한 기능을 보여준다: "`get-`" 으로 시작하는 모든 명령어는 AWS 객체를 반환하는데, 이 객체에 대한 어떠한 속성(property)나 메소드(method)를 호출하는 것도 가능하다 (이 각 명령어들은 `get-` 접두어를 사용하지 않는 버전으로도 존재하는데, 이 버전은 객체를 반환하는것 대신 객체에대한 간략한 요약정보를 출력해 준다). 위에서 보여진것 처럼 메소드나 속성의 이름을 하이픈(-) 다음에 타이핑 해보자. 이 경우에, `get-ami` 에 의해 반환된 AMI 객체의 `id` 라는 속성을 가져오고 있다 (id의 값은 디폴트로 가장 최신의 안정화된 버전의 Ubuntu 이미지에 대한 것이다; 다른 AMI 에 대한 예제는 아래 부분을 참고하길 바란다). 속성과 메소드의 목록을 확인하고 싶으면, 단순히 속성이나 메소드의 이름을 입력하지 않으면 된다:

```
$ fe2 get-ami -

Usage:           fe2 get-ami
                 fe2 get-ami architecture
                 fe2 get-ami block-device-mappings
                 fe2 get-ami create-tags
                 fe2 get-ami creation-date
                 ...
```

그러면 이제는 인스턴스를 실행할 수 있다. 아래의 명령어를 사용하면 새로운 "요청에 의한" Linux 인스턴스를 생성하고, (약 2분 가량 소요된 후)생성이 완료되면 이름, ID, 상태, IP 주소를 출력한다. 이 명령어는 반환하기 전, ssh 로 새로운 인스턴스게 접근 가능할 때 까지 기다린다.

```
$ fe2 launch base ami-0c55b159cbfafe1f0 50 m5.xlarge
base (i-00c7f2f81a841b525 running): 18.216.25.57
```

fe2의 `launch` 명령어는 최소한 4개의 파라메터를 요구한다: 생성하고자 하는 인스턴스의 이름, 사용하고자 하는 AMI (이름이나 ID 모두 가능하다, 위 예제에서는 앞서 얻어와진 AMI ID를 사용하였다), 생성하고자하는 디스크의 크기(GB), 인스턴스의 종류. AWS 페이지를 통해서, 가능한 다른 종류의 인스턴스와 가격을 확인할 수 있다, 서로다른 인스턴스의 가격을 확인하기 위해서는 아래의 `price-demand` 명령을 사용할 수도 있다 (m5 를 관심있는 인스턴스 시리즈로 교체해 보자; 현재로선, US 가격만이 표시되어 있다. 그러나, 그 가격들은 정확하지 않거나 최신의 정보가 아닐 수 있다. 따라서, 전체 가격 리스트를 위해서는 AWS 웹사이트를 방문하는것이 바람직하다).

```
$ fe2 price-demand m5
["m5.large", 0.096]
["m5.metal", 4.608]
["m5.xlarge", 0.192]
["m5.2xlarge", 0.384]
["m5.4xlarge", 0.768]
["m5.12xlarge", 2.304]
["m5.24xlarge", 4.608]
```

인스턴스가 실행중 이라면, 아래의 명령으로 해당 인스턴스로 ssh 접속이 가능하다:

```
$ fe2 connect base
Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.0-1032-aws x86_64)

Last login: Fri Feb 15 22:10:28 2019 from 4.78.240.2

ubuntu@ip-172-31-13-138:~$ |
```

이 시점부터, 베이스 인스턴스를 필요에 맞게 설정할 수 있다. 따라서, 원하는 소프트웨어를 apt install 명령어로 설치하고, 필요한 데이터 파일을 복사하는 등의 작업을 수행해야 한다. 아래에서 다뤄질 fastec2의 몇 기능들을 사용하기 위해서 AMI 에는 [tmux](https://hackernoon.com/a-gentle-introduction-to-tmux-8d784c404340)와 [lsyncd](http://axkibe.github.io/lsyncd/) 가 설치되어 있어야 한다. 따라서 이것들의 설치가 다음 명령어로 수행 되어야 한다 (`sudo apt install -y tmux lsyncd`). 또한, fastec2의 [오랫동안-수행되는 스크립트](https://www.fast.ai/2019/02/15/fastec2/#script) 기능을 사용하고자 한다면, `~/.ssh` 디렉토리에 에 비공개키(private key)가 존재해야만 한다. 해당 비공개키는 스크립트의 결과물을 저장하기 위해 또 다른 인스턴스로 접속하는 권한에 대한 것이다. 이를 위해서, (너무 민감하지 않다면) 보통의 비공개키를 복사하거나, 또는 (ssh-keygen을 타이핑하여) 새로운 비공개키를 생성하여 `~/.ssh/id_dsa.pub` 으로 옮겨놔야 한다.

> **확인사항:** 인스턴스를 AMI로 만들기 전, 다음의 내용을 인스턴스에서 수행했는지를 확인하기 바란다: `lsyncd`와 `tmux`의 설치, 비공개키의 복사.

Jupyter Notebook으로 접속하길 원하거나, 인스턴스에서 실행중인 다른 서비스로 접속하고 싶다면, [ssh 터널링(tunneling)](https://solitum.net/an-illustrated-guide-to-ssh-tunnels/)을 사용할 수 있다. ssh 터널을 생성하기 위해서, `fe2 connect` 명령어에 추가적인 인자값이 필요하다. 이 인자값은 아래와 같이 단일 정수값(단일 포트번호) 또는 (여러 포트번호들)에 대한 배열이 될 수 있다:

```
# Jupyter Notebook으로만 터널링 (포트 8888에서 실행중)
fe2 connect od1 8888

# 두 개의 터널: Jupyter Notebook, 8008에서 실행중인 서버
fe2 connect od1 [8888,8008]
```

이 명령어는 네트워크상에 존재하는 서로다른 머신들 사이의 포워딩에 대한 어떤 화려한 작업을 하는것이 아니다. 이 명령어는 단지 fe2를 실행하는 컴퓨터에서 ssh 로 접속하는 인스턴스로 직접적으로 연결하기 위한 것이다. 따라서, 일반적으로 이 명령어는 여러분의 로컬 PC에서 실행되어야 하고, (Jupyter의 경우) 로컬 PC의 브라우져의 `http://localhost:8888` 주소를 통해서 접근되어야 한다.

<div id="creating-your-amazon-machine-instance-ami"></div>

## 아마존 머신 인스턴스(AMI)를 생성 해보기

베이스 인스턴스에 대한 설정이 완료 되었다면, 이에 대한 여러분만의 AMI를 생성할 수 있다:

```
$ fe2 freeze base
ami-01b7ceef9767a163a
```

여기서 등장하는 `freeze`는 명령어이고, `base`는 인자값이다. `base`라는 이름을 AMI로써 "결빙" 하고자 하는, 여러분이 원하는 이름으로 교체할 수 있다. 이 과정에서, 인스턴스는 재시작 될 것이기 때문에, 열려있는 문서등이 저장되어 있음을 확인하는 등 인스턴스가 재시작되어도 OK인지를 체크해야 한다. 작업 완료까지 약 15분 정도의 시간이 소요된다 (수백 GB급의 큰 디스크의 경우는 수 시간이 소요된다). 작업 현황을 확인해보기 위해서, AWS 콘솔의 AMIs 섹션을 살펴보거나, 다음의 명령어를 수행해 볼 수 있다 (아직 이미지를 생성중인 것에 대하여 `pending`을 출력해서 보여준다):

```
$ fe2 get-ami base - state
pending
```

(나중에 보게되겠지만, 우리가 앞서본 fastec2의 기능을 호출하는 메소드를 사용하고 있다.)

<div id="launching-and-connecting-to-your-instance"></div>

## 인스턴스의 실행 및 연결 해보기

AMI를 생성하는것 까지 성공하였다. 그러면 그 AMI 템플릿을 사용해서 새로운 인스턴스를 실행해 볼 수 있다. 아래처럼 새로운 인스턴스가 생성되기까지 약 2분 정도의 시간이 소요된다:

```
$ fe2 launch inst1 base 80 m5.large
inst1 (i-0f5a3b544274c645f running): 18.191.111.211
```

새로운 인스턴스를 `inst1` 이라는 이름을 부여하고, 앞서 생성한 `base` AMI가 사용된다. 보면 알 수 있듯이 디스크의 크기와 인스턴스의 종류는 AMI를 생성하던 때와 동일할 필요가 없다 (다만, 디스크의 크기는 생성 당시의것 보다 작아질 수는 없다). `launch` 명령어에 대한 모든 가능한 옵션을 확인해볼 수 있다. `iops`와 `spot` 파라메터를 어떻게 사용해야 하는지는 다음 섹션을 통해서 알게될 것이다:

```
$ fe2 launch -- --help

Usage: fe2 launch NAME AMI DISKSIZE INSTANCETYPE [KEYNAME] [SECGROUPNAME] [IOPS] [SPOT]
       fe2 launch --name NAME --ami AMI --disksize DISKSIZE --instancetype INSTANCETYPE
         [--keyname KEYNAME] [--secgroupname SECGROUPNAME] [--iops IOPS] [--spot SPOT]
```

여러분만의 AMI로부터 첫 번째 인스턴스를 생성 가능 했음을 축하한다! 앞서 보여준 `fe2 launch` 명령어를 반복해서 사용하되, 다른 이름으로 여러개의 인스턴스를 생성할 수 있을 것이다. 그렇게 해서, 각 인스턴스에 대해서 `fe2 connect <인스턴스_이름>` 명령으로 ssh 접속이 가능하다. 인스턴스를 멈추기 위해서는, 인스턴스내 터미널에서 아래의 명령을 입력할 수 있다:

```
sudo shutdown -h now
```

또는 다른 대안으로, 로컬 PC의 터미널에 아래의 명령을 입력할 수도 있다 (inst1을 실제 인스턴스의 이름으로 바꿔야한다): 

```
fe2 stop inst1
```

위 명령어에서 `stop`을 `terminate`로 교체하면, 인스턴스를 종료시키는 것이 가능하다 (예를들어, 종료시킨다는 것은 인스턴스를 파괴하고, 인스턴스내 모든 데이터를 삭제하는 행동을 의미한다(디폴트). fastec2 또한 `name` 태그를 삭제해서, 즉시 재사용 가능하도록 조치하게 된다). 인스턴스가 멈출때 까지 fastec2가 기다려주길 원한다면, 다음의 명령을 사용해 볼 수 있다 (그렇지 않다면, 모든 일은 백그라운드에서 자동으로 수행된다):

```
$ fe2 get-instance inst1 - wait-until-stopped
```

다음의 명령어는 매우 편리한 기능을 보여준다. 인스턴스를 멈추고 나서, 다른 종류로 변경하는것이 가능하다! 즉, 초기 프로토타이핑을 저렴한 인스턴스에 수행하고, 준비되었을 때 매우 빠른 머신에서 거대한 데이터에대한 분석을 수행하는 것이 가능하다는 것이다.

```
$ fe2 change-type inst1 p3.8xlarge
```

그리고서, 인스턴스를 재시작하고, 전과 같은 방법으로 접속해볼 수 있다:

```
$ fe2 start inst1
inst1 (i-0f5a3b544274c645f running): 52.14.245.85

$ fe2 connect inst1
Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.0-1032-aws x86_64)
```

이 명령을 사용해서 이것저것 시도하다 보면, 무엇을 생성했었고, 현재 실제 실행 중인 인스턴스는 무엇이 있는지 파악이 어려울 수도 있다. 이때, 인스턴스의 리스트를 알아내기 위해서, 간편하게 `instances` 라는 명령을 수행해볼 수 있다.

```
$ fe2 instances
spot1 (i-0b39947b710d05337 running): 3.17.155.171
inst1 (i-0f5a3b544274c645f stopped): No public IP
base (i-00c7f2f81a841b525 running): 18.216.25.57
od1 (i-0a1b47f88993b2bba stopped): No public IP
```

"No public IP"라고 표시된 인스턴스는 인스턴스가 실행될 때 자동으로 공용 IP를 취득하게 된다. 일반적으로, 인스턴스의 이름으로 `fe2 connect` 명령어를 이용하여 접속하기 때문에, IP가 무엇인지 딱히 신경 쓸 필요는 없다. 다만, 필요하다면 fastec2를 사용해서 IP를 알아내는 것이 항상 가능하다는 점을 알아두길 바란다:

```
$ fe2 get-instance base - public-ip-address
18.216.25.57
```

<div id="launching-a-spot-instance"></div>

## Spot 인스턴스를 실행 해보기

Spot 인스턴스는 요청에 의한 인스턴스보다 약 70% 이상 저렴하다. 하지만, Spot 인스턴스는 언제든지 멈춰질 수 있으며, 항상 가용상태가 아닐 수도 있으며, root 볼륨에 있는 모든 데이터는 인스턴스가 멈춰지는 순간 삭제된다 (사실, 이 인스턴스의 종류는 종료만 될 수 있다. 멈추고, 나중에 재시작할 수 없다). Spot 인스턴스의 가격은 시간대, 인스턴스의 종류, 리전의 종류에 따라서 다르다. 특정 그룹 (여기서는 `p3` 종류를 예로 든다)의 인스턴스들에 대한 지난 3일간의 가격을 확인하기 위해서, 아래 명령어의 사용이 가능하다:

```
$ fe2 price-hist p3
Timestamp      2019-02-13  2019-02-14  2019-02-15
InstanceType
p3.2xlarge         1.1166      1.1384      1.1547
p3.8xlarge         3.9462      3.8884      3.8699
p3.16xlarge        7.3440      7.4300      8.0867
p3dn.24xlarge         NaN         NaN         NaN
```

이를 요청에 의한 인스턴스 가격과 비교해 보자:

```
$ fe2 price-demand p3
["p3.2xlarge", 3.06]
["p3.8xlarge", 12.24]
["p3.16xlarge", 24.48]
["p3dn.24xlarge", 31.212]
```

매우 좋아 보인다! 가격 그래프에 대한 좀 더 상세한 내용을 원한다면, AWS 콘솔의 Spot 갸격 툴을 확인하거나, fastec2의 Jupyter Notebook API를 사용해 볼 수 있다. 이 API는 `EC2` 클래스의 인스턴스를 생성 가능(선택적으로 리전정보를 생성자에게 전달 가능하다)하고, 그 클래스에 대한 메소드 호출이 가능한 것을 제외하면 `fe2` 명령어와 동일하다. (아직까지 Jupyter Notebook을 사용해본적이 없다면, 매우 뛰어난 툴이기 때문에 꼭 확인해보길 바란다! DataQuest에서 도움이 될만한 [튜토리얼](https://www.dataquest.io/blog/jupyter-notebook-tutorial/)이 있으니 이를 확인해 보길 바란다.) `price-demand` 메소드에는 Notebook에서 사용될 때 추가적인 기능이 있는데, 그래프 형태로 지난 몇 주간의 가격을 출력해 준다 (하이픈(-)은 Notebook에서 사용될 때, 언더스코어(_)로 교체되어야 함을 잊지 말자). 

![Example of spot pricing in the notebook API](https://www.fast.ai/images/spot_hist.png)

Spot 인스턴스를 실행하기 위해서, 단순히 `--spot` 을 `launch` 명령어에 추가하면 된다:

```
$ fe2 launch spot1 base 80 m5.large --spot
spot1 (i-0b39947b710d05337 running): 3.17.155.171
```

단 하나의 spot 인스턴스를 요청하고 있다. 요청에 대해서, 현재 수용이 불가한 상태일 수도 있다. 이 경우에, 수 분 정도 시간이 지난 후, fastec2으로부터 요청이 실패했음에 대한 에러 메시지가 출력됨을 확인할 수 있다. 위의 예제 요청의 경우는 성공했음을 알 수 있는데, 그 이유는 새로운 인스턴스가 "running" 이라는 메시지를 출력하고 있기 때문이다.

기억해 두어야 할 것: spot 인스턴스를 멈춘다면, 이는 사실상 종료되는 것으로 모든 데이터가 유실된다! 그리고, AWS는 이 종류의 인스턴스를 언제든지 마음대로 종료시킬 수 있다.

<div id="using-the-interactive-repl-and-ssh-api"></div>

## 대화형 REPL과 ssh API를 사용 해보기

어떤 메소드와 속성이 가용한지를 어떻게 알 수 있는가? 그리고, 그들을 어떻게 더 편리하게 접근할 수 있을까? 이에 대한 대답은: 대화형 [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop)을 사용하라는 것이다! 아래의 그림이 그 의미를 설명해 줄 수 있을 것이다:

![The fastec2 REPL](https://www.fast.ai/images/fe2_repl.png)

`-- -i`를 객체를 반환하는 명령어(현재까지 `instance`, `get-ami`, `ssh`가 있음) 의 마지막에 추가하게 되면, 해당 객체를 `result`라는 특별한 이름으로 접근/사용할 수 있는 [IPython 세션](https://ipython.org/ipython-doc/3/interactive/tutorial.html)으로 이동하게 된다. 따라서, 단순히 `result`를 타이핑 해보자. 그리고, `Tab` 키를 눌러서, 가용한 모든 메소드와 속성을 확인해 보자. 이것은 완전한 Python 인터프리터로, 해당 객체와 상호작용하기 위해서 Python 의 모든 기능을 사용할 수 있다. `result`와의 상호작용을 충분히 하고나서, `Ctrl-d`를 두 번 누르면 IPython 세션을 종료할 수 있다.

이 것의 사용에 대한 한가지 흥미로운점은 ssh 를 통해서 원격지 인스턴스로 명령어를 보낼 수 있는 API를 제공하는 ssh 명령어에 대한 실험이다. `ssh` 명령어에 의해 반환된 객체는 몇 가지 추가적인 기능이 포함된 표준 [Paramiko SSHClient](http://docs.paramiko.org/en/2.4/api/client.html)이다. 그 추가적인 기능 중 하나는 `send(cmd)`로, `cmd`를 해당 인스턴스의 `tmux` 세션으로 전송한다. 이 기능은 주로 스크립트에서 사용되도록 디자인 되었지만, 아래처럼 REPL을 통해서도 실험해 볼 수 있다:

![Communicating with remote tmux session via the REPL](https://www.fast.ai/images/fe2_tmux.gif)

fastec2 API를 대화형으로 샆펴보고 싶다면, 가장 쉬운 방법은 `fe2 i`를 사용해서 REPL을 실행하는 것이다 (추가적으로 리전이름의 일부, 또는 리전 ID를 붙여넣을 수도 있다). `e`라고 불려지게 되는 fastec2.EC2 객체가 자동으로 생성된다. `e`를 타이핑 해보자. 그리고 `Tab` 키를 눌러서, 가능한 모든 옵션을 확인해 보자. IPython은 smart autocall mode에서 시작되게 된다. 즉, 메소드 실행을 위해서 괄호를 타이핑 하는것 조차 필요 없게 될 수 있다는 것이다. 다음은 그 예를 보여준다:

```
$ fe2 i Ohio
IPython 6.1.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: e.instances
inst1 (i-0f5a3b544274c645f m5.large running): 18.222.175.103
base (i-00c7f2f81a841b525 m5.xlarge stopped): No public IP
od1 (i-0a1b47f88993b2bba t3.micro running): 18.188.162.203

In [2]: i=e.get_instance('od1')

In [3]: i.block_device_mappings
Out[3]:
[{'DeviceName': '/dev/sda1',
  'Ebs': {'AttachTime': datetime.datetime(2019, 2, 14, 9, 30, 16),
   'DeleteOnTermination': True,
   'Status': 'attached',
   'VolumeId': 'vol-0d1b1a47539d5bcaf'}}]
```

fastec2는 많은 편리한 AWS EC2를 관리하기 위한 메소드를 제공하고, SSH 및 SFTP의 사용을 쉽게 하기위한 추가적인 기능을 제공한다. 향후 작성될 글을 통해서, fastec2 API의 이런 기능들을 좀 더 자세히 다뤄볼 것이다.

fastec2를 이용하여 오랫동안-수행되는 작업을 실행하고 모니터링하는 방법을 알고 싶다면, 이 글 시리즈의 [파트2](https://www.fast.ai/2019/02/15/fastec2-script/)를 확인해 보길 바란다. 파트2에서, 자동 포맷팅/마운팅을 포함하여 fastec2가 볼륨과 스냅샷을 생성하기 위해 어떠한 도움을 주는지에 대한 내용이 다뤄진다.
