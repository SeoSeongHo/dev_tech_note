AWS CloudFormation
==================

### 도입 배경 
AWS 리소스 사용 수가 점점 늘어나, 새로운 프로젝트 개발 시, 수동으로 생성해야 하는 리소스들이 많아짐.

***

### 문제 발생 및 계획 수립
수동으로 생성해야 하는 리소스들이 많아짐에 따라, 실수가 발생할 수도 있고, 
시간적인 측면에서도 손해가 발생함.

이러한 문제들에 대해 고찰하던 중, 
*Infrastructure as Code* 에 도전해보기로 결심하고, 2가지 방안을 세움. 


* AWS CloudFormation

* Terraform

결국, AWS 리소스를 주로 사용하고 있던 터라, 1번 AWS CloudFormation 을 선택. (IAM Role 등을 통해 조금 더 쉽게 AWS Infra 에 접근 가능)

AWS 리소스에 대한 정보를 템플릿 (Json, Yaml) 파일로 구성하여, 자동 생성 및 관리 가능

***

### 원하는 방향
* 프로젝트 생성 및 코드 업데이트
* Jenkins 를 통해 코드 빌드 및 테스트, ECR 에 docker image push
* CloudFormation 을 통해 새로운 리소스 생성 및 변경된 리소스 업데이트

***

### CloudFormaion 스택 생성 순서

1. Create yaml files that is included all of infrastructure.

2. Sync all files to S3 bucket.
``` aws s3 sync . s3://endpoint ```

3. Create CloudFormation stack.

***

### CloudFormation 스택 업데이트 순서

1. Modify yaml files that are included all of code as infrastructure.

2. Sync all files to S3 bucket.
``` aws s3 sync . s3://endpoint ```

3. Update CloudFormation stack.

***

### Difficult point And Resolution

#### 자신이 사용하는 리소스의 구조를 정확하게 이해해야 한다.
리소스의 구조를 정확하게 이해해야만, 템플릿을 아름답게 분리하고, 
어떤 리소스가 필수적으로 먼저 시작되어야 하는지 파악하여 편리하게 배포할 수 있다.

#### Yaml 파일이 익숙하지 않은 사람들에게 주는 팁
``` aws cloudformation validate-template --template-body file://{filePath} ```  

해당 명령어는 해당 파일의 문법적인 오류를 잡아준다. 로컬에서 yaml 파일 생성 후, 한 번 오류를 잡고 S3 에 sync 하자

#### IAM Role 정책 잘 활용하기
IAM Role 을 통해, 다른 인프라와의 접근이 가능하므로, 이를 잘 이용하여, 템플릿을 구성한다.

***

#### 예제 링크 [ECS CloudFormation Example Template](https://github.com/aws-samples/ecs-refarch-cloudformation/blob/master/infrastructure/ecs-cluster.yaml)

#### 스택 간 의존성

![img](images/cloudformation_designer.png)
