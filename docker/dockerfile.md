## DockerFile

DockerFile 이란, Docker 이미지를 만들기 위한 설정들이 적혀 있는 파일입니다.
호스트와 컨테이너 간 관계, 컨테이너 내부 실행 명령 등과 같은 커맨드 라인으로 구성됩니다.

***

### DockerFile 문법

* FROM : 도커 이미지는 기본적으로 layer 를 가지고 있고, 여기에 수정을 할 시, 추가적인 layer 를 생성하는 방식으로 구성되어 있습니다.
하여, DockerFile 의 상단에는 어떤 이미지를 기반으로 할지 설정하는 FROM 명령어가 존재합니다.
```
FROM microsoft/dotnet:2.2-aspnetcore-runtime
```


* RUN : 컨테이너 내 실행 명령어를 쓸 때 사용합니다.
주의할 점으로는 yes/no 를 선택해야 하는 경우에, 이미지 생성 중에는 사용자 입력을 통해 선택할 수 없으므로, 미리 -y 와 같은 옵션 값을 넣어주어야 합니다.
미리 CI / CD 파이프라인을 통해, 빌드와 같은 단계를 거치지 않았다면, 해당 RUN 명령어를 통해 각 언어별 빌드 및 이후 과정을 진행할 수 있습니다.
```
RUN dotnet build {project_name} -c Release -o /app
```


* EXPOSE : 호스트와 연결할 포트 번호입니다.
docker run -p port:port 와 같이 컨테이너 구동 시, 포트 맵핑을 시키려면 (WAS 의 경우), 해당 명령어를 통해 컨테이너의 포트를 열어주어야 합니다.
```
EXPOSE 80
EXPOSE 443
```


* VOLUME : 컨테이너 저장소 대신, 호스트에 저장하는 명령어입니다.
docker run -v 명령어를 사용하면, 호스트와 컨테이너간 볼륨 마운트가 가능합니다.
```
VOLUME {path}
VOLUME [{path1}, {path2}]
```


* CMD / ENTRYPOINT : 컨테이너가 구동되었을 때, 실행되는 실행파일입니다. (c# 에선 dll 파일)
두 가지의 차이는 CMD 는 docker run 시, 추가 인자를 통해서 실행할 수 있는 반면, ENTRYPOINT 는 지정한 실행파일 실행만이 가능하다.
```
ENTRYPOINT ["dotnet", {dll file}]
```


* WORKDIR : CMD 명령어에서 사용할 실행 파일이 존재하는 디렉토리입니다.
```
WORKDIR /app
```

***

### 마무리

두서 없이 필자가 나중에 보기 편한대로 적은 글이라 혼동될 수 있습니다.
DockerFile 의 경우, docker 공식 페이지에서 친절하게 설명되어 있으니 참고하시면 좋을 것 같습니다.
DockerFile 에서 그치는 것이 아니라, 이를 기반으로 하여, CI / CD 파이프라인을 자동으로 구축하는 것까지
이해하고, 설계가 가능해야 실제 프로덕션 환경에서 편리함을 몇 배로 가져다 줄 것 입니다.
