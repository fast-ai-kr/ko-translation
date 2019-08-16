# SageMaker

## 목차
- [가격](#pricing)
- [설정하기](#setup)
  - [SageMaker 노트북 인스턴스 생성하기](#create_sagemaker_notebook)
  - [인스턴스 종료하기](#shutdown_instance)
- [설치시 발생하는 문제 트러블슈팅 하기](#troubleshooting)
- [기타 필요한 도움에 대해서..](#more_help)

이 글은 SageMaker를 사용해서 fast.ai 코스의 v3인 _"Practical Deep Learning for Coders"_를 빠르게 시작해볼 수 있게하기 위한 가이드 입니다.

아래 나열된 단계를 전에 완료한적이 있거나, 전에 진행하던 작업을 재개하고 싶으면 [SageMaker 작업재개](./sagemaker_update.md) 글을 참조하시기 바랍니다.

[AWS CloudFormation](https://aws.amazon.com/cloudformation/) 을 사용해서 노트북 인스턴스, 노트북의 생명주기에 대한 설정, 그리고 IAM Role를 포함한 모든 SageMaker 자원을 설정하게 됩니다. 디폴트로는 _ml.p2.xlarge_ 종류의 SageMaker 노트북 인스턴스가 설정되게 되는데, 이 종류는 NVIDIA K80 GPU와 50GB의 EBS 디스크공간을 제공합니다.

## 가격 <span id="pricing"></span>

권장되는 인스턴스는 _ml.p2.xlarge_로, 시간당 약 $1.26정도의 금액이 발생합니다. 시간당 과금율은 선택된 인스턴스의 종류에 따라 상이하기 때문에, 가용한 모든 종류를 [여기서](https://aws.amazon.com/sagemaker/pricing/) 확인해 보시기 바랍니다. 해당 인스턴스 또는 _ml.p3.2xlarge_ 인스턴스를 사용하기 위해서, [여기서](https://course.fast.ai/start_aws.html#step-2-request-service-limit) 명시적으로 제한요청(limit request)에 대한 요청을 해야만 합니다. 이 때 인스턴스는 멈춰져 있어야만 합니다.

## 설정하기 <span id="setup"></span>

### SageMaker 노트북 인스턴스 생성하기 <span id="create_sagemaker_notebook"></span>

1. [AWS 웹페이지](https://aws.amazon.com/)를 방문해서 'Sign In to the Console'을 클릭합니다. 그 다음으로 이메일 또는 계정이름과 비밀번호를 사용해서 로그인 하시기 바랍니다. 계정이 없는 경우, 계정 생성(sign up)을 해 주시기 바랍니다.
![로그인](https://course.fast.ai/images/aws/signin.png)
계정이 없는 경우, 'Sign in to the Console' 대신 'Sign up' 버튼을 클릭 하여 가입을 진행 합니다. 가입을 위해서는 신용카드 정보를 설정하는 과정이 필요합니다. 입력되는 신용카드로 과금된 인스턴스의 사용에 대한 비용이 청구됩니다 (무료 크레딧이 있다면, 무료 크레딧이 소진될 때 까지는 과금되지 않습니다). 이 때 신원을 검증하기 위해서 핸드폰 전화번호도 기입되어야 하고, 해당 전화번호로 전화가 걸려오게 됩니다.

2. 생성된 또는 이미 가지고 있는 계정으로 로그인 되었다면, CloudFormation을 사용하여 SageMaker의 모든 자원을 생성할 준비가 된 것입니다. CloudFormation 스택을 실행하기 위해서, 현재 거주하고 계신 지역에 가장 가까운 리전(Region)에 대한 <kbd>Launch Stack</kbd> 버튼을 아래 테이블에서 클릭합니다. 

| 리전 | 이름 | 실행 링크 |
| --- | --- | ------ |
| US West (Oregon) Region |	us-west-2 |	[<img src="https://course.fast.ai/images/aws/cfn-launch-stack.png"/>](https://us-west-2.console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/create/review?filter=active&templateURL=https%3A%2F%2Fs3-eu-west-1.amazonaws.com%2Fmmcclean-public-files%2Fsagemaker-fastai-notebook%2Fsagemaker-cfn.yml&stackName=FastaiSageMakerStack) |
US East (N. Virginia) Region | us-east-1 | [<img src="https://course.fast.ai/images/aws/cfn-launch-stack.png"/>](https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/review?filter=active&templateURL=https%3A%2F%2Fs3-eu-west-1.amazonaws.com%2Fmmcclean-public-files%2Fsagemaker-fastai-notebook%2Fsagemaker-cfn.yml&stackName=FastaiSageMakerStack)
US East (Ohio) Region | us-east-2 | [<img src="https://course.fast.ai/images/aws/cfn-launch-stack.png"/>](https://us-east-2.console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks/create/review?filter=active&templateURL=https%3A%2F%2Fs3-eu-west-1.amazonaws.com%2Fmmcclean-public-files%2Fsagemaker-fastai-notebook%2Fsagemaker-cfn.yml&stackName=FastaiSageMakerStack)
US East (N. California) Region | us-west-1 | [<img src="https://course.fast.ai/images/aws/cfn-launch-stack.png"/>](https://us-west-1.console.aws.amazon.com/cloudformation/home?region=us-west-1#/stacks/create/review?filter=active&templateURL=https%3A%2F%2Fs3-eu-west-1.amazonaws.com%2Fmmcclean-public-files%2Fsagemaker-fastai-notebook%2Fsagemaker-cfn.yml&stackName=FastaiSageMakerStack)
Asia Pacific (Tokyo) Region | ap-northeast-1 | [<img src="https://course.fast.ai/images/aws/cfn-launch-stack.png"/>](https://ap-northeast-1.console.aws.amazon.com/cloudformation/home?region=ap-northeast-1#/stacks/create/review?filter=active&templateURL=https%3A%2F%2Fs3-eu-west-1.amazonaws.com%2Fmmcclean-public-files%2Fsagemaker-fastai-notebook%2Fsagemaker-cfn.yml&stackName=FastaiSageMakerStack)
Asia Pacific (Seoul) Region | ap-northeast-2 | [<img src="https://course.fast.ai/images/aws/cfn-launch-stack.png"/>](https://ap-northeast-2.console.aws.amazon.com/cloudformation/home?region=ap-northeast-2#/stacks/create/review?filter=active&templateURL=https%3A%2F%2Fs3-eu-west-1.amazonaws.com%2Fmmcclean-public-files%2Fsagemaker-fastai-notebook%2Fsagemaker-cfn.yml&stackName=FastaiSageMakerStack)
Asia Pacific (Sydney) Region | ap-southeast-2 | [<img src="https://course.fast.ai/images/aws/cfn-launch-stack.png"/>](https://ap-southeast-2.console.aws.amazon.com/cloudformation/home?region=ap-southeast-2#/stacks/create/review?filter=active&templateURL=https%3A%2F%2Fs3-eu-west-1.amazonaws.com%2Fmmcclean-public-files%2Fsagemaker-fastai-notebook%2Fsagemaker-cfn.yml&stackName=FastaiSageMakerStack)
Asia Pacific (Mumbai) Region | ap-south-1 | [<img src="https://course.fast.ai/images/aws/cfn-launch-stack.png"/>](https://ap-south-1.console.aws.amazon.com/cloudformation/home?region=ap-south-1#/stacks/create/review?filter=active&templateURL=https%3A%2F%2Fs3-eu-west-1.amazonaws.com%2Fmmcclean-public-files%2Fsagemaker-fastai-notebook%2Fsagemaker-cfn.yml&stackName=FastaiSageMakerStack)
Asia Pacific (Singapore) Region | ap-southeast-1 | [<img src="https://course.fast.ai/images/aws/cfn-launch-stack.png"/>](https://ap-southeast-1.console.aws.amazon.com/cloudformation/home?region=ap-southeast-1#/stacks/create/review?filter=active&templateURL=https%3A%2F%2Fs3-eu-west-1.amazonaws.com%2Fmmcclean-public-files%2Fsagemaker-fastai-notebook%2Fsagemaker-cfn.yml&stackName=FastaiSageMakerStack)
Canada (central) Region | ca-central-1 | [<img src="https://course.fast.ai/images/aws/cfn-launch-stack.png"/>](https://ca-central-1.console.aws.amazon.com/cloudformation/home?region=ca-central-1#/stacks/create/review?filter=active&templateURL=https%3A%2F%2Fs3-eu-west-1.amazonaws.com%2Fmmcclean-public-files%2Fsagemaker-fastai-notebook%2Fsagemaker-cfn.yml&stackName=FastaiSageMakerStack)
EU (Ireland) Region | eu-west-1 | [<img src="https://course.fast.ai/images/aws/cfn-launch-stack.png"/>](https://eu-west-1.console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks/create/review?filter=active&templateURL=https%3A%2F%2Fs3-eu-west-1.amazonaws.com%2Fmmcclean-public-files%2Fsagemaker-fastai-notebook%2Fsagemaker-cfn.yml&stackName=FastaiSageMakerStack)
EU (Frankfurt) Region | eu-central-1 | [<img src="https://course.fast.ai/images/aws/cfn-launch-stack.png"/>](https://eu-central-1.console.aws.amazon.com/cloudformation/home?region=eu-central-1#/stacks/create/review?filter=active&templateURL=https%3A%2F%2Fs3-eu-west-1.amazonaws.com%2Fmmcclean-public-files%2Fsagemaker-fastai-notebook%2Fsagemaker-cfn.yml&stackName=FastaiSageMakerStack)
EU (London) Region | eu-west-2 | [<img src="https://course.fast.ai/images/aws/cfn-launch-stack.png"/>](https://eu-west-2.console.aws.amazon.com/cloudformation/home?region=eu-west-2#/stacks/create/review?filter=active&templateURL=https%3A%2F%2Fs3-eu-west-1.amazonaws.com%2Fmmcclean-public-files%2Fsagemaker-fastai-notebook%2Fsagemaker-cfn.yml&stackName=FastaiSageMakerStack)

3. 그러면 아래의 스크린샷과 같이 AWS 자원을 생성하기 위한 템플릿이 나타나는 AWS CloudFormation 웹 콘솔창이 열리게 됩니다. 입력 파라메터들에 대한 내용을 둘러보고난 후, 디폴트 설정을 그대로 사용하거나 필요에 따라서 값을 업데이트 합니다. 그리고나서, **"I acknowledge that AWS CloudFormation might create IAM resources."** 체크박스란을 체크한 후 <kbd>Create</kbd> 버튼을 클랙하여 스택을 생성합니다.

![스택 생성](https://course.fast.ai/images/sagemaker/create_stack.png)

4. 아래와 같은 생성된 스택을 보여주는 CloudFormation 페이지가 나타날 것입니다. 스택이 **CREATE_COMPLETE** 상태가 되고나면, AWS 웹 콘솔을 열고 상단 바에 있는 Services를 클릭하여 'sagemaker'를 타이핑합니다. 그러고나면, Amazon SageMaker를 클릭합니다.

![스택 생성 완료](https://course.fast.ai/images/sagemaker/01.png)

5. 좌측의 네비게이션바에서, 노트북 인스턴스를 선택합니다. 이곳으로부터, 노트북 인스턴스를 생성, 관리, 접근하는것이 가능합니다. fastai 라는 이름의 노트북 인스턴스의 상태가 _InService_ 인것을 아래의 스크린샷과 같이 확인 가능합니다.

![노트북 인스턴스 메뉴](https://course.fast.ai/images/sagemaker/17.png)

6. "Open Jupyter" 링크를 클릭하여 Jupyter 웹 콘솔차을 엽니다.

### 인스턴스 종료하기 <span id="shutdown_instance"></span>
- 인스턴스를 사용한 작업이 완료되면, 노트북이 열려 있는 탭을 닫습니다. 그리곤, Stop 버튼을 을 누르는것을 꼭 기어하시기 바랍니다! 만약 Stop 버튼을 누르지 않는다면, Stop 버튼이 눌러질때 까지 비용이 계속해서 발생하게 됩니다.
![Stop 버튼](https://course.fast.ai/images/sagemaker/23.png)
다시 노트북을 실행하고 싶다면, 코스 또는 fastai 라이브러리를 업데이트한 후 [SageMaker 재개하기](./sagemaker_update.md) 페이지로 가보시기 바랍니다.

## 설치시 발생하는 문제 트러블슈팅 하기 <span id="troubleshooting"></span>

- 15분이 경과했음에도 불구하고 노티 이메일을 수신하지 못했다면, 노트북 인스턴스에 fast.ai 라이브러리 및 디펜던시를 설치하는데 문제가 생긴것일 수 있습니다. 트러블슈팅을 하기 위해서, [AWS 콘솔](https://aws.amazon.com/console/)을 열고 **CloudWatch** 링크를 클릭합니다 (검색창에 cloudwatch를 타이핑 해서 찾을 수 있습니다). CloudWatch 콘솔창이 열리고나면, Logs -> /aws/sagemaker/NotebookInstances -> fastai/LifecycleConfigOnStart 또는 fastai/LifecycleConfigOnCreate 의 뎁스로 들어가서 설치 스크립트의 출력결과를 확인하는 것이 가능합니다.

## 기타 필요한 도움에 대해서.. <span id="more_help"></span>

코스의 컨텐츠에 관련하여 질문이나 이슈사항이 있다면, [fast.ai 포럼](http://forums.fast.ai/)에 이를 포스팅 하는것을 권장합니다.
