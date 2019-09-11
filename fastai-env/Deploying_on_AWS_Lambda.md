# AWS Lambda 배포

## 목차
- [요금](#price)
- [요구조건](#requirements)
- [준비 단계](#preparation)
- [단계 1: S3 Bucket 생성](#create)
- [단계 2: 학습 된 모델 적용](#export)
- [단계 3: SAM 예제 프로젝트 가지고 오기](#grab)
- [Application 개요](#application)
- [PC에서 개발](#local)
- [패키징과 배포](#packaging)
- [log 확인, 추적 및 필터링 ](#fetch)
- [삭제](#cleanup)

## AWS Lambda 배포
이 문서는 [Amazon API Gateway](https://aws.amazon.com/api-gateway/) & [AWS Lambda](https://aws.amazon.com/lambda/)을 사용하여 fastai 모델을 배포하는 간략한 가이드 입니다. 이 가이드에서는 Lambda 와 API Gateway AWS와 상호작용하는 어플리케이션을 만들기위해 [Serverless Application Model (SAM)](https://aws.amazon.com/serverless/sam/) 프레임워크를 사용합니다.


[AWS Lambda](https://aws.amazon.com/lambda/)는 프로지져닝과 서버의 관리 없이 코드 실행이 가능한 서비스 입니다. Lambda를 사용할 시간 만큼 계산 되어 청되 되며, 코드가 실행 되고 있지 않을때에는 요금이 부과 되지 않습니다. 

[Amazon API Gateway](https://aws.amazon.com/api-gateway/)는 규모와 상관없이 REST 및 WebSocket API를 생성, 게시, 유지하고 모니터링 및 보안하기 위한 AWS 서비스입니다.

## 가격<span id="price"></span>
AWS Lambda를 사용하면 사용한 만큼만 비용을 지불합니다. 함수 **요청** 수와 **기간**, 코드를 실행하는 데 걸리는 시간에 따라 요금이 청구됩니다. AWS Lambda 요금에 대한 자세한 내용은 [여기](https://aws.amazon.com/lambda/pricing/)를 참조하세요.

Amazon API Gateway통하여 자신의 API가 구동 중일 경우만 요금을 지불하면 됩니다. HTTP/REST API의 경우, 수신한 API 호출과 전송한 데이터 양에 대해서만 요금을 지불하면 됩니다. 요금에 대한 자세한 내용은 [여기](https://aws.amazon.com/api-gateway/pricing/)에서 확인 하세요 .


## 요구조건<span id="requirements"></span>
로컬 컴퓨터에 다음과 같이 응용 프로그램을 설치해야 한다.
 * [AWS CLI](https://aws.amazon.com/cli/)가 Administrato 권한으로 설정 되어야 합니다.
 * [Python3 설치](https://www.python.org/downloads/)
 * [Docker 설치](https://www.docker.com/community-edition)
 * [AWS SAM CLI](https://aws.amazon.com/serverless/sam/)가 설지 되어야합니다. 자세한 내용은 가이드를 참조 하세요.

## 준비 단계<span id="preparation"></span>
### 단계 1: S3 Bucket 생성<span id="create"></span>
먼저, 배포를 하기 전에 Lambda functions/layer과 함께 ZIP파일로 압축이 되어있는 우리의 모델을 업로드하기 위해 `S3 bucket`을 필요로 합니다.  만약 S3 Bucket가 없다면 아래의 명령어를 실행시켜 생성해 주세요.
```
aws s3 mb s3://REPLACE_WITH_YOUR_BUCKET_NAME
```

### 단계 2: 학습 된 모델 적용(학습된 모델 S3에 업로드)<span id="export"></span>
우선 Jupyter notebook와 같은곳에서 같이 프로그램 되고 학습된 모델을 필요로 합니다. [SAM(Serverless Application Model )](https://aws.amazon.com/serverless/sam/) 어플리케이션은    Output Name을 가진 클래스 텍스트 파일을 가지고 TorchScript 형식으로 개발된 PyTorch 모델이 S3에 저장 되어있다고 생각합니다.

Fastai 비젼 모델을 추출하는 Python 코드는 아래의 예제와 같습니다. 
```
# export model to TorchScript format
trace_input = torch.ones(1,3,299,299).cuda()
jit_model = torch.jit.trace(learn.model.float(), trace_input)
model_file='resnet50_jit.pth'
output_path = str(path_img/f'models/{model_file}')
torch.jit.save(jit_model, output_path)
# export classes text file
save_texts(path_img/'models/classes.txt', data.classes)
tar_file=path_img/'models/model.tar.gz'
classes_file='classes.txt'
# create a tarfile with the exported model and classes text file
with tarfile.open(tar_file, 'w:gz') as f:
    f.add(path_img/f'models/{model_file}', arcname=model_file)
    f.add(path_img/f'models/{classes_file}', arcname=classes_file)
```
이제 우리의 모델을 S3에 업로드 할 준비가 되었습니다. 
```
import boto3
s3 = boto3.resource('s3')
# replace 'mybucket' with the name of your S3 bucket
s3.meta.client.upload_file(tar_file, 'REPLACE_WITH_YOUR_BUCKET_NAME', 'fastai-models/lesson1/model.tar.gz')
```

Lesson1에 근거한 모델을 학습, JIT 포멧으로 추출 및 모델의 업로드의 대한 내용은 [여기서](https://github.com/fastai/course-v3/blob/master/docs/production/lesson-1-export-jit.ipynb) 확인 가능합니다.

### 단계 3: SAM 예제 프로젝트 가지고 오기<span id="grab"></span>
우리는 우리의 애플리케이션을 배치하기 위해 SAM을 사용할 것이다. 그러기 위해서 먼저 아래 명령을 사용하여 예시 프로젝트를 다운로드해야 한다.
 ```
wget https://github.com/fastai/course-v3/raw/master/docs/production/aws-lambda.zip

unzip aws-lambda.zip
```

## Application 개요<span id="application"></span>
예시의 어플리케이션은 비전 모델에 인터페이스를 호출을 합니다.  Lambda function이 로드 되면 S3에서 PyTorch 모델을 다운로드하고 모델을 메모리에 로드합니다.

입력값으로 인터넷 어디가에 있는 이미지의 URL을 가지고 있는 JSON 객체를 사용합니다. 어플리케이션은 이미지를 다운로드하고 이미지의 픽셀을 PyTorch Tensor객체로 변환한 다음 PyTorch 모델에게 전달합니다. 결과물로 모델에서 가장 높은 score, class name 그리고 confidence level(신뢰수준)을 반환합니다.

이 어플리케이션의 구조는 다음과 같습니다.
```
.
├── event.json          <-- Event payload for local testing
├── pytorch             <-- Folder for source code
│   ├── __init__.py
│   ├── app.py          <-- Lambda function code
└──template.yaml        <-- SAM Template
```
SAM 템플릿에 대한 자세한 내용은 [여기](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-template-basics.html)를 참조하십시오.

Python 프로그래밍 언어를 사용하여 Lambda function을 프로그래밍하는 방법에 대한 자세한 내용은 [여기](https://docs.aws.amazon.com/lambda/latest/dg/python-programming-model-handler-types.html)를 참조하십시오.

**Request Body format**
Lambda function의 입력값으로 분류할 이미지의 URL이 포함된 JSON string body를 넣어 줍니다.
예시:
```
{
    "url": "REPLACE_THIS_WITH_AN_IMAGE_URL"
}
```

**Response format**
Lambda function은 상태코드(성공시 코드 : 200)와 예측한 Class와 신뢰 점수(confidence score)를 JSON객체로 반환을 합니다.
예시:
```
{
    "statusCode": 200,
    "body": {
        "class": "english_cocker_spaniel",
        "confidence": 0.99
    }
}
```
다른 입력/출력 형식을 취하도록 어플리케이션에 따라 이 파일을 수정할 수 있습니다.<br/>
**Lambda Layer**
추가적인 코드와 콘텐츠를 [Lambda layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html)의 형태로 가지고와서 Lambda function으로 구성 가능합니다. 하나의 계층(Layer)은 Zip 파일이며 이는 라이브러리, 사용자 지정 런타임,  기타 dependencies로 구성된다. 계층(Layer)를 이용하면, 배포 패키지를 포함하지 않고도 function에서 라이브러리를 사용 할 수 있습니다. 

이 프로젝트에서는 어플리케이션을을 실행하는데 있어 필수적인  PyTorch 라이브어리가 포함된 공개 액세스 가능한 Lambda layer을 사용할 것입니다. 이러한 layer는 다음과 같은 regions에 있습니다.  [us-west-2, us-east-1, us-e-west-1, ap-southeast-1, ap-ss-southeast-2,ap-north-1,eu-central-1]. 기본으로 설정된 region은 us-east-1 입니다. 아래에 표시된 표는 다른 버전의 PyTorch를 가지는 PyTorch Lamdba layer가 나타나 있습니다.

| Layer ARN| PyTorch Version| 
| :------------------- | -------------------: |
| arn:aws:lambda:AWS_REGION:934676248949:layer:pytorchv1-py36:1 | PyTorch 1.0.1
| arn:aws:lambda:AWS_REGION:934676248949:layer:pytorchv1-py36:2 |  PyTorch 1.1.0 |

만약 기본으로 설정 된 region(즉  us-east-1)이 아닌 region에서 모델을 실행한다면  `template.yaml`의 `AWS_REGION`라고 적혀잇는 부분을 알맞은 region으로 변경을 해야합니다. (예: us-west-2).
```
...
  LambdaLayerArn:
    Type: String
    Default: "arn:aws:lambda:AWS_REGION:934676248949:layer:pytorchv1-py36:2"
        ...
```

## PC에서 개발<span id="local"></span>

### Lambda Test 환경 구축
우선 이전에 단계에서서 S3로 업로드한 PyTorch model에서 S3 Bucket과 Key값을 대체하여  `env.json`파일을 생성해 줍니다.
```
{
    "PyTorchFunction": {
      "MODEL_BUCKET": "REPLACE_WITH_YOUR_BUCKET_NAME",  
      "MODEL_KEY": "fastai-models/lesson1/model.tar.gz"      
    }
}
```

### local sample payload를 이용하여 함수 호출
`event.json`이라는 파일을 편집하고 분류할 이미지 `URL` 값(json)을 입력하십시오.
함수를 PC 테스트하려면 다음 sam 명령을 호출하십시오.
```
sam local invoke PyTorchFunction -n env.json -e event.json
```
만약 어플리케이션에서 자신만의 결과를 보고 싶은경우  `event.json`파일을 입력 형식과 일치하야 합니다.
다음은 컴퓨터 비전에서 사용했을 경우 예제입니다.
```
curl -d "{\"url\":\"REPLACE_THIS_WITH_AN_IMAGE_URL\"}" \
    -H "Content-Type: application/json" \
    -X POST http://localhost:3000/invocations
```
**SAM CLI**는 Lambda와 API Gateway을 Local(PC)에서 에뮬레이트 하도록 하고 template.yaml를 사용하여 시스템환경에서 어플리케이션이 어떻게 동작할지 파악합니다(실행시간, 소스코드 위치 등). - 아래에  내용은 API를 초기화와 라우팅 하는 내용이며 향후 CLI가 읽을것 입니다.
 ```
...
Events:
    PyTorch:
        Type: Api # More info about API Event Source: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#api
        Properties:
            Path: /invocations
            Method: post
```

## 패키징과 배포<span id="packaging"></span>
AWS Lambda Python 구동에서는 어플리케이션을 포함한 모든 dependencies가 있는 플랫 폴더가 필요하다. SAM은 `CodeUri` 속성을 사용하여 애플리케이션과 dependencies을 위치를 파악합니다.
 ```
...
    PyTorchFunction:
        Type: AWS::Serverless::Function
        Properties:
            CodeUri: pytorch/
            ...
```
그 다음 Lambda의 function을 S3로 패키지하기 위해 아래의 명령어를 입력합니다.
```
sam package \
    --output-template-file packaged.yaml \
    --s3-bucket REPLACE_THIS_WITH_YOUR_S3_BUCKET_NAME
```
아래의 명령어로 Cloudform Stackdmf 생성하고 우리의 SAM resources를 배포합니다. 버킷 이름과 개체 키에 대한 기본 매개 변수를 다시 지정해야 합니다. 아래에 --parameter-overrides 옵션을 이용하여 값들을 deploy시에 전달 할 수 있습니다. 
```
sam deploy \
    --template-file packaged.yaml \
    --stack-name pytorch-sam-app \
    --capabilities CAPABILITY_IAM \
    --parameter-overrides BucketName=REPLACE_WITH_YOUR_BUCKET_NAME ObjectKey=fastai-models/lesson1/model.tar.gz
```
SAM의 자세한 사용방법은 [HOWTO가이드](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-quick-start.html)를 참조하세요.
배포가 완료 후 다음 명령을 통하서 API Gateway Endpoint URL을 알수 있습니다.
```
aws cloudformation describe-stacks \
    --stack-name pytorch-sam-app \
    --query 'Stacks[].Outputs[?OutputKey==`PyTorchApi`]' \
    --output table
```

## log 확인, 추적 및 필터링<span id="fetch"></span>

트러블 슈팅을 단순화를 위해 SAM CLI는 sam log라는 명령어를 가지고 있습니다. sam logs라는 명령어를 이용하여 Lambda function에서 생성된 log를 가지와서 확일 할 수있습니다. 더하여 이 명령에는 터미널의 log를 인쇄하는것 외에도 버그를 빠르게 찾는 등의 여러 가지의 할용도 높은 기능들이 있습니다. 

`참고`: sam log 명령어는 sam에서 뿐만 아니라 AWS Lambda에서도 작동합니다.
```
sam logs -n PyTorchFunction --stack-name pytorch-sam-app --tail
```
Lambda function의 log 필터링에 대한 자세한 내용은 [SAM CLI 설명서](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-logging.html)에서 확일 할 수 있습니다.

## 삭제<span id="cleanup"></span>

최근에 배포된 Serverless Application를 삭제 하기 위해서 다음의 AWS CLI 명령을 사용하십시오.
```
aws cloudformation delete-stack --stack-name pytorch-sam-app
```
