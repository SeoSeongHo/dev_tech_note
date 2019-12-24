## Nginx

### nginx 란

트래픽이 많은 웹사이트를 위해 확장성을 위해 설계한 비동기 이벤트 기반구조(event driven)의 웹서버 소프트웨어이다.

웹서버, 프록시 서버로 많이 사용되고 있는 nginx 에 대해 필자는 프록시 서버로 이용하기 위해 nginx 에 대해 공부해보았다.

***

### nginx vs apache

시장 점유율을 보면 크게 이 두 가지 솔루션이 인기를 끌고 있다. 하여, 간단하게 두 솔루션의 차이점을 설명하고자 합니다

#### apache
- 쓰레드 / 프로세스 기반 구조
- 요청이 많으면, 많은 쓰레드를 생성 (1개의 클라이언트 == 1개의 쓰레드) 하므로, CPU 소모가 심함

#### nginx
- 비동기 event driven 기반 구조
- 다수의 연결을 효과적으로 처리가능 (프록시)
- apache 보다 적은 리소스로 빠르게 동작 가능

***

### nginx as proxy server

필자는 nginx 를 WAS 의 앞단에서 요청을 받아, WAS 로 nginx 의 설정(nginx.conf)에 따라 전달해주는 역할로 사용하였다.

nginx server 는 nginx.conf file 을 기반으로 docker container 위에 구동시켰다.

nginx.conf file 은 크게 3가지 파트로 나누어진다.

#### global
- user : nginx 프로세스가 실행되는 권한
- worker_processes : nginx 프로세스 실행 가능 수

```
worker_processes 1;
```

#### events
- worker_connections : 위 하나의 프로세스가 처리 가능한 커넥션 수
- use epoll : 수천개의 file descriptor를 처리할 수 있도록 보다 효율적인 알고리즘을 사용하고 있어, 대량 요청이 발생하는 시스템에 적합

```
events { 
  worker_connections 1024;
  use epoll;
}
```

#### http
- keepalive_timeout : 커넥션을 유지할 설정 값 (초 단위)
- sendfile : 응답을 보낼 때, user-space의 buffuer 메모리를 사용하지 않고, kernel file buffer 를 사용하여 보다 빠른 성능을 보장할지 여부
- upstream
  - server : 업스트림 서버 (WAS)
  - keepalive : 유지시키는 최대 커넥션 개수
- server
  - listen : 맵핑시킬 포트
  - location : 맵핑시킬 endpoint, redirect, header 등등 설정 가능 부분
  
```
http {
    upstream test_server {
        server was_server:5000;
		    keepalive 1024;
    }

    server {
        listen 80;
        listen 443;

        location / {
            proxy_pass         http://test_server;
            proxy_redirect     off;
            proxy_set_header   Host $host;
        }
    }
}
```

***

### 마무리

처음에 AWS Elastic Beanstalk, ECS 에서 배포 시 붙이는 로드밸런서의 역할과 겹친다고 생각하여 많이 헷갈렸었지만,

물론 같은 기능을 하게끔 구현이 가능하지만, 하나의 인스턴스 내에서 커넥션을 조절해야할 때나, 

Request 를 filtering, macro Setting 해야될 때 효과적이라는 것을 새롭게 알게 되었다.
