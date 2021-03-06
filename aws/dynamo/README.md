## AWS Dynamo DB

### 도입 배경
기존 RDS 의 경우, 서버 위에 띄워서 관리하다 보니, 인스턴스 비용이 추가되고, 서버를 관리하는 데에도 노고와 비용이 소모됨
하여, AWS Dynamo DB 에 대해 공부하고, 새롭게 알게 된 내용들을 정리함

기존 DDB 는 처리량의 기준을 잡기 애매하고, 이 기준을 넘을 시, 요청이 거부되어 큰 리스크를 안겨주었었음.
하지만 2018 aws re-invent 이 후, 온디맨드 형식의 dynamo db 를 지원하면서, 오토 스케일링을 통해 처리량 스케일 업 다운이 가능해지면서,
조금 더 안정성을 가지게 됨.

***

### DDB 장점

* 단순하고 신속한 배포

* 데이터 자동 백업

* 빠르고 일관된 응답시간

* 보조 인덱스를 통한 빠른 조회

* 사용한만큼 사용료 지불

***

### DDB 단점

필자가 느끼기에는 러닝커브가 다른 RDS 나 몽고 DB 보다 높은 편인 것 같다.

* 테이블 구성 시, partition key, sort key, gsi, lsi 등을 잘 활용하여야 하고, 지속적인 피드백을 통해 재구성하여 최적화해야함.

* 쓰기, 읽기 처리 용량 구축을 잘하거나, auto scailing 을 통해 최적화해야함.

* 지원되는 ORM 이 많지 않기에, sdk 를 받아서, ORM 을 직접 구축하여 사용해야하는 번거로움이 있음.

***

### Table 생성

Table 은 크게 Partion Key, Sort Key 와 나머지 컬럼들로 구성되어 있음

- Partition Key 는 필수 값으로, 주로 카디널리티가 높은 키 값으로 설정해놓음.

- Sort Key 는 선택 값으로, 주로 범위 서칭을 할 때, 혹은 1:n, m:n 맵핑되는 컬럼 값으로 설정해놓음.

***

### GSI, LSI

DDB 에서 중요한 개념들 중 하나로, 이들을 보조인덱스로 잘 이용하여 테이블을 구성해야만, 쿼리로 데이터접근하기 편해짐.

- GSI (Global Secondary Index) : 테이블과 별도로 읽기 & 쓰기 용량을 책정, 테이블 생성 이후에도 중간에 인덱스 추가 가능

- LSI (Local Secondary Index) : 테이블과 같은 읽기 & 쓰기 용량을 책정, 테이블 생성 시에만 추가 가능

마지막에 설명하겠지만, GSI, LSI 를 잘 이용하여 RDS 의 관계도를 구축할 수 있음. (복잡하고, 어려울 수도......)
복잡한 관계가 얽혀 있는 여러 DB 들을 구성할 때에는 RDS 를 사용하는 게 어떨지......

***

### RCU, WCU

Nosql 기반이라 read, write 성능 자체는 좋은 편이고, 이에 대한 처리량을 최적화하여 설정해두는 것이 최종 유저의 목표 !!

- RCU (Read Capacity Unit) : 읽기 용량 유닛 / 1 RCU 는 초당 4KB 처리 단위로 측정

eventual consistent read 는 읽기 2회 제공, strong consistent read 는 1회 제공

예 ) item 의 크기가 7.5KB 인 경우, 8KB로 올림 측정으로 읽음. (ecr 은 1RCU 소모, scr 은 2RCU 소모)

- WCU (Write Capacity Unit) : 쓰기 용량 유닛 / 1 WCU 는 초당 1kb 처리 단위로 측정

***

### DDB 를 쓰기 전에 고려할 점
무작정 Nosql 이니, I/O 작업이 빠르겠지 하고 덤벼들면, 높은 러닝커브를 겪을 것이다. 아래와 같은 점들을 고려하고, 도입해보자

* Use Case 이해
  * 데이터 성격
  * 엔티티 간 관계
  * 동시 접속 패턴
  * 시계열 데이터인지 아닌지
  * 데이터 보관기간은 어느정도인지
  
* Access Pattern 이해
  * 대상 데이터 분석
  * 1건 읽기 vs 여러 건 읽기 (키와 인덱스 값들을 통하여 범위로 읽을지, 1:1 맵핑 키를 이용하여 1건만 읽을지)
  * 쿼리 집계 및 KPI
  
* 데이터 모델링
  * 1:1, 1:m
  * 1 애플리케이션 == 1 테이블 쓰기를 권고함 (AWS 추천)
  * primary key 정의 (partition key 와 sort key 를 통하여)
  * LSI, GSI 를 활용한 쿼리 정의
  
***

### 비용 최적화 하기
- 쓰기가 가장 비싸므로, 잘 생각 1WCU = 1KB
- TTL 시간이 지나면 삭제해주는 기능은 무료이므로 잘 활용하자
- 테이블 삭제도 무료 !
- eventtual consistent read 는 50 % 저렴하다
- 다른 aws 서비스들은 프리티어가 1년까지만 유지되는 경우가 있지만, DDB 프리티어는 영구적
- Json Data Type 을 지원하여, 여러 데이터를 모아서 하나의 item 으로 저장가능
- 속성 값이 큰 경우 압축하여 binary 형태로 저장 가능
- 속성이름도 item 크기에 포함되므로, 최대한 짧게 설정하기 !!
- 크기가 많이 큰 속성은 S3 에 저장하고, 식별값만 DDB 에 저장하기
- Auto Scailing 을 이용하면, 처리용량이 실제 트래픽이 맞추어 자동 조정됨

***

### 더 나아갈 부분

DDB 테이블에 수정이 있을 시, amazon dynamodb streams 를 이용하여, lambda 를 trigger 할 수 있다는 것을 새롭게 알게 됨.

이를 이용하여, 많은 부분들을 효과적으로 처리할 수 있을 듯..... (실제 프로덕션에 적용하기 전에 조금 더 찾아보고 포스팅하는 것으로..!)
