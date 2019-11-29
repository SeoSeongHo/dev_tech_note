AWS CDK (Cloud Development Kit)
================================

### 배경
저번 시간에 AWS CloudFormation 의 적용 건에 대해 간단하게 설명하였다. 

그런데 !!!! 어제 AWS 세미나를 통해 알게 된..


AWS CDK 라는 것이, 나름 힘들게 작성했었던 cloudformation 템플릿을 간단한 코드로 구현이 가능하고, 추상화까지 가능하여,

조금 더 깔끔하고 편리하게 IAC 를 구성할 수 있다는 것을 알았다.

이에 세미나를 통해, 문서를 통해 알게 된 정보들을 적어두려 한다.

***

### 설명
저번에도 간단하게 말했듯, 백엔드 자동화는 크게 2가지로 분리된다고 생각한다.
* CI / CD
: 코드 빌드, 테스트 및 배포 자동화

* IAC (Infrastructure as code)
: 인프라 서비스, 리소스 프로비저닝 및 배포를 코드로 관리 (자동화)

여기서 인프라 서비스와, 리소스를 잘 구분못하는 사람들을 위해 알고 있는 간단한 정의로 설명하자면,

인프라 서비스는 쉽게 말해, AWS EC2, AWS SQS, AWS VPC 와 같이, AWS 내 크게 크게 자리잡고 있는 서비스들을 칭한다.

인프라 리소스는 AWS VPC 내를 예로 들자면, 갖가지 Network 설정, API GateWay 등 서비스를 구성하는 리소스를 의미한다.

AWS CDK 는 아래 예제와 같이 코드 몇 줄로, 몇 백줄 짜리의 CloudFormation 템플릿을 만들고, 배포해준다.

***

### 그렇다면 왜 IAC 를 적용하는가 ?
물론, 콘솔에서 직접 생성하는 것보다 시간이 줄어드는 큰 장점이 있을 것이다.

하지만 필자가 생각하는 가장 중요한 장점은, 수동으로 콘솔에서 생성하였을 때, 발생할 수 있는 실수를 줄이는 것이다.

수동으로 작업이 진행되다보면, 사람이 하는 일이다보니, 실수로 다른 VPC 를 사용한다거나, 잘못된 설정을 주는 경우가 생긴다.

이 때, 바로 알아차리기도 힘들 뿐더러, 수동으로 다시 인프라 서비스, 리소스들을 롤백해야 하는 경우가 생긴다.

하지만 IAC 를 사용한다면, 이러한 변경점에 대한 이슈를 미리 알아차릴 수 있고, 버그가 생겼을 때에도 자동으로 롤백해주는 기능이 있어, 큰 위험에 노출될 확률을 줄여준다.

이러한 의미에서 AWS CDK 는 인프라 배포 작업에 있어, 큰 도움을 줄 것이라 생각한다.

***

### 실행 방법
AWS CDK 는, AWS SDK 를 받아서, 혹은 AWS CLI 를 통해 사용한다..

#### installing cdk

```npm install -g aws-cdk```

#### checking cdk version. 

아직 cdk 가 완전히 정착되지 못하여, 한달 내에 많은 버전업이 생기기 때문에, 수시로 버전업을 해주는 것이 좋다 !

```cdk --version```

#### initializing cdk

cdk 를 시작하는 단계, 이 때, LANGUAGE 를 선택할 수 있는데, java, python, c# 등 유명한 언어는 대부분 존재한다.

```cdk init --language LANGUAGE [TEMPLATE]```

```cdk init --language csharp```

#### Compiling the Application

```dotnet build src```

#### Downloading AWS CDK SDK

```dotnet add package Amazon.CDK.AWS.S3```

#### Example C# code

AWS CDK SDK 를 통하여, S3 Bucket 을 생성하는 코드이다. 기본적으로 AWS Base 값들이 들어가져 있고, 그 안에 수정할 값들만 변경하여 주면 된다.

```using Amazon.CDK;
using Amazon.CDK.AWS.S3;

namespace HelloCdk
{
    public class HelloStack : Stack
    {
        public HelloStack(Construct scope, string id, IStackProps props) : base(scope, id, props)
        {
            new Bucket(this, "MyFirstBucket", new BucketProps
            {
                Versioned = true
            });
        }
    }
}
```

#### making cloudformation template

다음 명령어는, 해당 코드를, cloudformation template 으로 합성해주는 명렁어이다.

```cdk synth```

아래와 같이 실제 cloudformation template 이 만들어진 것을 볼 수 있다.

```Resources:
  MyFirstBucketB8884501:
    Type: AWS::S3::Bucket
    Properties:
      VersioningConfiguration:
        Status: Enabled
    Metadata:
      aws:cdk:path: HelloCdkStack/MyFirstBucket/Resource
  CDKMetadata:
    Type: AWS::CDK::Metadata
    Properties:
      Modules: "@aws-cdk/aws-codepipeline-api=VERSION,@aws-cdk/aws-events=VERSION,@aws-c\
        dk/aws-iam=VERSION,@aws-cdk/aws-kms=VERSION,@aws-cdk/aws-s3=VERSION,@aws-c\
        dk/aws-s3-notifications=VERSION,@aws-cdk/cdk=VERSION,@aws-cdk/cx-api=VERSION\
        .0,hello-cdk=0.1.0"
```

#### differences from previous resources

이 전, 리소스와 어떤 부분들이 변하였는지 보여주는 명령어이다.

```cdk diff```

아래와 같이 변경점들을 미리 보아, 수동으로 생성하였을 때, 놓칠 수 있는 실수들을 줄여준다.

```Stack HelloCdkStack
Resources
[~] AWS::S3::Bucket MyFirstBucket MyFirstBucketB8884501
 |- [+] BucketEncryption
     |- {"ServerSideEncryptionConfiguration":[{"ServerSideEncryptionByDefault":{"SSEAlgorithm":"aws:kms"}}]}
```

#### deploying cdk

cdk 로 만들어진 cloudformation template 을 deploy 하는 명령어다.

```cdk deploy```

아래와 같이 리소스가 업데이트 되는 모습을 볼 수 있다.

```HelloCdkStack: deploying...
HelloCdkStack: creating CloudFormation changeset...
 0/2 | 10:55:30 AM | UPDATE_IN_PROGRESS   | AWS::S3::Bucket    | MyFirstBucket (MyFirstBucketID) 
 1/2 | 10:55:50 AM | UPDATE_COMPLETE      | AWS::S3::Bucket    | MyFirstBucket (MyFirstBucketID) 

HelloCdkStack

Stack ARN:
arn:aws:cloudformation:REGION:ACCOUNT-ID:stack/HelloCdkStack/ID
```

#### destroying cdk 

아래 명렁어를 통해, template 을 통하여, 만들어진 리소스를 제거할 수 있다.

```cdk destroy```

***

### 정리

* AWS CDK 는 인프라 리소스 프로비저닝 변경점을 미리 보아 실수를 예방할 수 있다.

* 코드 몇 줄로, cloudformation 의 몇 백줄 코드를 대신할 수 있다.

* yaml, json file 로 cloudformation 을 구성하는 끔찍함을 없앨 수 있다. (yaml - 들여쓰기..... , json : 따옴표......)
