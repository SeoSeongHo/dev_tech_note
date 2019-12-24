## Dockerrun.aws.json file

요즘, 대부분의 어플리케이션을 컨테이너 위에서 구동시키는 것이 추세이다.

이 어플리케이션을 AWS 와 같은 Cloud 환경에 배포할 때, 단순 인스턴스에 접속하여, docker image 를 pull 받고, run 시키는가..??

파일 한 개로 컨테이너 속성, 볼륨 등 다양한 환경들을 조정하고, 구동시킬 수 있는 세련된 방법이 있다.

***

### Dockerrun.aws.json file 이란

Docker image 위치, 컨테이너 설정 등 컨테이너 환경 어플리케이션 종속성과 설정 등을 표현한 json file 이다.

컨테이너로 구성된 어플리케이션을 AWS Elastic Beanstalk 나 Elastic Container Service 에 배포할 때, Dockerfile 이나 Dockerrun.aws.json 파일이 필요하다.

여기서 멀티 컨테이너로 구성되어 있다면, Dockerrun.aws.json file 을 써야하는데, 요즘 대부분 어플리케이션들은 로그 모듈, nginx 와 같은 프록시 서버를 같이 올리기 때문에,

Dockerrun.aws.json file 을 많이 사용한다.

Dockerrun.aws.json file 에는 기본적으로 다음과 같은 일들을 서술한다.

- 인스턴스 볼륨 할당

```
{
  "volumes": [
    {
      "name": "test",
      "host": {
        "sourcePath": "/var/log/test"
      }
    },
    {
      "name": "test2",
      "host": {
        "sourcePath": "/var/log/test2"
      }
    }
  ]
}
```

- 작업 정의
  - Docker image 위치
  - 환경 변수
  - 메모리 예약
  - 인스턴스와 볼륨 마운트
  - 컨테이너 간 종속성
  - etc...
  
```
{
  "volumes": [
    ~~~
  ],
  "containerDefinitions": [
    {
      "name": "testContainer1",
      "image": "%wasDockerImageUrl",
      "environment": [
        {
          "name": "ENV",
          "value": "prod"
        },
        {
          "name": "WAS_TYPE",
          "value": "test"
        }
      ],
      "essential": true,
      "memoryReservation": 256,
      "mountPoints": [
        {
          "sourceVolume": "test",
          "containerPath": "/app/test"
        },
        {
          "sourceVolume": "test2",
          "containerPath": "/app/test2"
        }
      ]
    },
    {
      "name": "testLogContainer",
      "image": "%logDockerImageUrl",
      "environment": [
        {
          "name": "ENV",
          "value": "prod"
        }
      ],
      "essential": true,
      "memoryReservation": 64,
      "mountPoints": [
        {
          "sourceVolume": "test ",
          "containerPath": "/app/test"
        }
      ]
    }
  ]
}
```
 
[Dockerrun.aws.json 예시](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/create_deploy_docker_v2config.html)
 
위 예시를 보면, dockerfile 과 container 환경에 대한 이해도가 있는 사람이라면 쉽게 이해될 것이다 !!
 
***
 
### Dockerrun.aws.json & Jenkinsfile
 
저번 포스팅에서 Jenkinsfile / Deploy layer 에서 Dockerrun.aws.json 에 parameters 를 넘겨 세팅하고, Deploy 한다고 언급했었다.

위에서 작성한 예제 코드에서 %wasDockerImageUrl 와 같은 값을 볼 수 있는데, 이러한 값들을 jenkins 파일에서 넘겨주고, dockerrun.aws.json 파일을 기반으로 어플리케이션을 배포하면 된다.
 
