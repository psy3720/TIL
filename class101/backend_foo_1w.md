### 2023.08.26

이 내용은 크리에이터 푸님의 "현직 대기업 개발자 푸와 함께하는 진짜 백엔드 시스템 실무!" 강의를 기반으로 정리하였습니다.  

강의 링크  
(https://class101.net/ko/products/T6HT0bUDKIH1V5i3Ji2M)
---

#### 강의 환경 세팅
- jdk 1.8
- intelliJ(community)
- visual studio code

#### JDK란?
> Java Development Kit의 약자로, JRE(Java Runtime Environment)와 Java 컴파일러를 포함하는 소프트웨어 개발 키트(SDK)

#### IDE란?
> Integeration Development Environment의 약자로, 통합 개발 환경이라는 의미이며 개발에 필요한 요소들을 통합하여 제공해주는 환경

---

#### Google Cloud Platform 가입 후 접속

VM이란?
: virtual machine의 약자로 가상 장비라는 의미이다.  

- vm은 host 운영체제 위에 가상화를 하여 가상의 장비를 만들어 내어 사용하는 것
- 이때 vm이 사용하는 운영체제를 guest 운영체제라고 한다.
- 기본적인 서버는 물리장비(physical hardware)위에 host 운영체제가 올라가고 그 위에서 애플리케이션이 실행되는 형태이다.   
*이때, 인스턴스는 vm하나하나를 의미한다. (인스턴스=생성된 vm을 지칭하는 용어)

--- 

#### 인스턴스 생성하기
- E2, e2-micro
- 부팅디스크 centos7로 변경(표준 영구  디스크)
- http(80), https(443) 트래픽 허용 체크

리전,영역
: 리젼이란 해당 인스턴스가 위치할 지역을 의미한다. 국가/도시 단위로 구분한다. 영역은 특정 리젼에서 별도의 영역을 나눠놓은 것을 의미한다.  
*동일 리젼이라고 하더라도 영역은 물리적으로 독립된 공간이다. 리젼a영역에 문제가 발생했을 때 a영역에 있는 인스턴스는 서비스가 불가능해질 수 있다.  
그러나 같은 리젼의 b영역은 물리적으로 독립된 공간이기 때문에 영향을 받지 않는다.  

#### 용어
- 내부IP: 인스턴스 구분, GCP내의 가상머신간 통신 및 기타서비스간의 통신에 사용된다.  
- 외부IP: 외부에서 접근할 수 있는 IP

ssh로 접속 후 명령어 입력해보기
```
# 네트워크 인터페이스의 현재 상태를 표시하는 명령
> ifconfig  
```

[참고]
https://opencourse.tistory.com/591?category=354408
---

#### Docker Desktop 설치
#### 장점
- 독립적인 운영체제로 동작하는것이 아니라 프로세스 개념으로 동작하기 때문에 vm에 비해 성능적 이점
- 컨테이너를 스크립트로 관리할 수 있기 때문에 운영에 용이하다.

#### vm과의 차이점
vm은 host 운영체제 위에서 독립적인 machine을 만들어서 그 위에 guest 운영체제를 동작시키므로 가상화 단계를 거치면서 오버헤드가 발생한다.

docker는 애플리케이션을 마치 프로세스처럼 간주하여 vm에 비해 오버헤드가 적어 성능상의 이점이 있다.

```
# docker 설치, 옵션은 y
> sudo yum install docker

# docker 실행
> sudo systemctl start docker

# docker sample container start
> sudo docker run -d -p 80:80 docker/getting-started
```
해당 인스턴스의 외부ip/tutorial url로 접속해보면 접속할 수 있다.

```
# docker 실행중인 컨테이너 확인
> sudo docker ps 

# docker 실행 중지
> sudo docker stop [container id] 
```
---

#### 프로세스와 프로그램?
> 하드디스크에 저장된 프로그램을 실행하면 메모리에 올라가고, 메모리에 올라간 프로그램을 프로세스라 한다. 메모리에 올라간 프로세스 중 실행시키기 적당한 프로세스를 선택하여 CPU가 실행시킨다.

> CPU가 현재 시점에 실행시키기 적당한 프로세스를 고르는 방법을 스케줄링이라고 한다.

I/O를 하는 시간을 IO Burst라고 하고, CPU를 사용하는 시간을 CPU Burst라고 한다.
그리고 I/O가 많은 애플리케이션을 I/O Burst/Bound 애플리케이션, CPU Burst가 많은 애플리케이션을 CPU Burst/Bound이라 한다.

CPU를 많이 사용하는 애플리케이션을 만들기 위해 Hash 연산을 많이 수행하는 애플리케이션을 사용한다.

---

#### sample project 다운로드
https://github.com/lleellee0/cpu-bound-application
다운받은 jdk 1.8로 세팅

```
# wget 다운로드
> sudo yum install wget

# java 다운로드
> sudo yum install java

# sample 코드 다운로드
> wget https://github.com/lleellee0/class101-files/raw/main/cpu-0.0.1-SNAPSHOT.jar
```

*permalink로 다운받을 경우 invalid or corrupt 에러 발생한다. raw로 받아야 함.
[참고] (https://12340zszs.tistory.com/97)

---


#### 스트레스 테스트 툴로 성능 측정하기
nodejs, visual studio 설치

artillery 사용
> artillery는 node.js로 작성된 스트레스 테스트 도구이다. 사용하기 편하고 쉽게 설정이 가능해 다양한 상황에서 스트레스 테스트를 할 수 있다.  


#### 특징
 - HTTP(S), Socket.io, Websocket등 다양한 프로토콜을 지원한다.
 - 시나리오 테스트를 할 수 있다.
 - Javascript로 로직을 작성해서 추가할 수 있다.

```
# artillery 설치
> npm install -g artillery
```

#### yaml 파일 작성
```
config:
  target: "http://123.123.123" 
  phases:
    - duration: 60
      arrivalRate: 1
      name: Warm up
scenarios:
  - name: "just get hash"
    flow:
      - get:
          url: "/hash/123"
```
- duration: 측정시간
- arrivalRate: 초당 요청개수
- url: 요청url
- target: 서버의 host를 지정한다.

```
# 성능측정(cpu-test.yaml의 요청을 report.json으로 출력한다)
> artillery.cmd run --output report.json .\cpu-test.yaml

# json형식의 report를 html 형식으로 보기
> artillery.cmd report .\report.json 
```

[참고]
node artillery (https://www.artillery.io/)
---

#### CPU bound 애플리케이션을 도커 이미지로 만들고, GCP 인스턴스에 배포해보기

#### [참조]
- spring boot docker (https://spring.io/guides/topicals/spring-boot-docker/)
- dockerhub (https://hub.docker.com/)

#### docker hub repository 생성
docker hub 접속 > 로그인 > repository > create repository > name:spring-boot-cpu-bound로 생성

에러참고
(https://chaelin1211.github.io/study/2021/04/01/docker-error.html)


#### [프로세스]
1. dockerfile을 빌드해서 이미지 생성
2. 이미지를 docker hub 저장소에 푸시하여 이미지 업로드
3. 저장소에 푸시한 이미지를 풀받기
4. 다운로드된 이미지를 실행(run)

```
# docker run
docker run -p {HOST_PORT}:{CONTAINER_PORT} 사용자_이름/저장소_이름
```
외부에서 HOST_PORT로 요청이 들어오면 그 요청이 CONTAINER_PORT로 포워딩된다.

```
#docker hub에 push
docker push user_name/repository_name
```
docker login을 통해 docker hub에 로그인 된 상태여야 한다.

---

#### GCP에 pull받고 컨테이너 실행하기
```
# docker 설치
> sudo yum install docker 

# docker 실행
> sudo systemctl start docker

# dockerhub로부터 pull 받기
> sudo docker pull user_name/respository_name

# docker container run
> sudo docker run -p 80:80 user_name/repository_name
```

[에러 참고]  
(https://rainbound.tistory.com/entry/windows-%EC%97%90%EB%9F%AC-The-system-cannot-find-the-file-specified)