## Jenkinsfile

이번에는 Jenkinsfile 작성법에 대해 간단하게 정리하여 합니다.

Jenkinsfile 은 groovy 형식으로, 파이프라인을 형성한 github repository 에 존재하면, jenkins 측에서 설정된 경로의 jenkinsfile 을 scan 하여, 작업합니다.

### Component

- node : 쉽게 말해 실행 단위입니다. 파이프라인의 실행 단위를 2개로 나누고 싶으면 
``` 
node { 

} 
node {

} 
``` 
와 같이 분리하여 파일을 구성하면 됩니다.

***

- stage : 쉽게 말해 파이프라인의 한 단계입니다. 예를 들어 파이프라인의 단계를

``` prepare -> build -> test -> deploy -> clean ``` 으로 정했다면 jenkinsfile 을

``` 
node {
  stage('prepare'){
    ~~~
  }
  stage('build'){
    ~~~
  }
  stage('test'){
    ~~~
  }
  stage('deploy'){
    ~~~
  }
  stage('clean'){
    ~~~
  }
}
```

다음과 같이 구성하면 됩니다. ~~~ 대신 각 단계마다의 수행 command 가 들어가면 됩니다.

***

- parameters : Jenkins Pipeline 을 실행시킬 때, UI 상에서 파라미터를 입력한 후, 파일에 반영합니다.

예를 들어 docker image 를 빌드하고 푸쉬할 것인지, cloud 환경에 deploy 까지 진행할 것인지, version tagging 을 무엇으로 달 것인지에 대한 값을 파라미터로 입력받습니다.

```
node{
  properties([
    parameters([
      booleanParam(
        name: 'ENABLE_BUILD_IMAGE',
        defaultValue: false,
        description: 'Do you want to build your application image and push ??'
      ),
      booleanParam(
        name: 'ENABLE_DEPLOYMENT',
        defaultValue: false,
        description: 'Do you want to deploy your application on cloud env ??'
      ),
      string(
        name: 'DEPLOY_VERSION',
        defaultValue: 'latest',
        description: 'version or commit hash'
      ),
      string(
        name: 'BRANCH_NAME',
        defaultValue: 'dev',
        description: 'branch name or hash'
      )
    ])
  ])
  
  def ~~~
  def ~~~
  
  stage('prepare'){
    ~~~
  }
  ~~~
}
```

***

### 결론

필자는 deploy layer 에서 dockerrun.aws.json file 에 parameter 를 넘겨주고, 해당 파일을 실행시키는 방식으로 파이프라인을 구축하였습니다.

다음 포스팅에선 dockerrun.aws.json 에 대한 간단한 정의와, 예시를 보고 조금 더 완성된 CI / CD 에 다가가 보겠습니다.
