### 2023.09.01

이 내용은 크리에이터 푸님의 "현직 대기업 개발자 푸와 함께하는 진짜 백엔드 시스템 실무!" 강의를 기반으로 정리하였습니다.

강의 링크  
(https://class101.net/ko/products/T6HT0bUDKIH1V5i3Ji2M)
---

#### 서버가 죽는 이유와 Message Queue를 도입하여 데이터 유실 방지
- 서버가 죽는 이유와 Message Queue를 도입하여 어떻게 서버가 죽는 상황을 줄일 수 있는지 알아보기.

서버가 죽었다라는건 증상으로 나눠보면 일부 요청이 실패하는가, 모든 요청이 실패하는가로 나눠볼 수 있고, 원인은 네트워크 장애, DB 장애등 여러가지로 나눠볼 수 있다.  

CPU 바운드 애플리케이션에서는 CPU를 과도하게 사용하는 Hash 연산을 다량 요청했을 때 요청이 실패했고, DB I/O 바운드 애플리케이션에서는 DB로 요청하는 쿼리를 다량 요청했을 때 실패하는 요청이 생겼다.  
톰캣에서 사용자의 요청은 우선 큐에 들어가고, 큐에 들어간 요청이 놀고 있는 쓰레드가 있따면 그 쓰레드에 할당되어 처리된다.  
모든 쓰레드가 사용중인데 새로운 요청이 들어오면 그 요청은 큐에서 대기하게 된다.  

큐 사이즈를 모두 채우고 나서도 계속 요청이 들어오면 그 요청들은 버려진다. 그리고 큐에 들어온 요청도 30초를 넘으면 타임아웃 처리가 된다.(default)

그럼 큐 사이즈, 쓰레드 사이즈, 타임아웃 시간 모두 늘리면 요청에 실패하는 시간은 늦출 수 있겠지만, 결과적으로 해결 방법이 되진 않는다. 실제 처리 속도를 올리지 않으면 결국 요청은 큐에 쌓이고 언젠간 실패한다.

예제에서는 DB 앞단에 큐를 두어 사용자의 글 작성 요청을 모두 RabbitMQ라는 메시지 큐를 사용하여 Queue에 넣었다가 처리한다.  

Tomcat 큐에 넣는것과 Message 큐를 따로 두는것의 차이  
Tomcat 큐에 넣는건 메모리에 저장된 데이터로 애플리케이션 강제 종료시 전부 날아가버릴 수 있다.  
반면 Message 큐를 별도로 사용하면 디스크에 저장하는 등 여러가지 옵션을 줄수있다.  

메시지큐의 장점
- 비동기성:
  요청이 몰릴 때에도 저장했다가 처리할 수 있다. 앞쪽 애플리케이션은 실제 로직이 수행되는 것과 무관하게 단순히 큐에 넣고 다음요청을 받을 수 있는 상태가 된다.

- 애플리케이션간 의존성 제거:
  API를 직접 호출하는 것과 중간에 큐가 있는 것중 뒷쪽에 있는 애플리케이션이 중단되었을 때에도 메시지가 유실되지 않는다.

- 이중화:
  큐도 애플리케이션이기 때문에 이중화 하는 방법을 제공한다.  
  따라서 큐끼리 서로 동기화하기 때문에 하나의 큐인것처럼 사용하지만 실제 이중화된 큐를 사용하는 것이다.

- 신뢰성:
  실패한 메시지는 큐로 Ack를 하지 않기 때문에 그 메시지는 큐에서 빠져나가지 않는다.  
  (절대 유실되지 않는다고 보장할수는 없다, 따라서 유실되면 안되는 메시지의 경우 로깅을 철저히 하여 복구할수 있게 준비해야 한다.)

- 확장성:
  애플리케이션이 스케일 아웃하더라도 메시지큐에서 따로 처리해줄 필요는 없다.  
  사용하던 큐를 그대로 사용하면 된다.

---

#### RabbitMQ 도입 글 목록 캐싱
로컬환경에서 docker로 rabbitmq 띄우고 메시지 발행 및 소비해보기..

```
# rabbitmq 실행
> docker run -d --hostname my-rabbit --name some-rabbit -p 5672:5672 -p 15672:15672 rabbitmq:3-management
```
먼저 선언한 포트(5672)는 레빗MQ, 나중에 명시한 포트(15672)는 모니터링시 사용한다.

(localhost:15672) 접속
기본계정으로 로그인한다. (guest/guest)

#### 메시지 발행 및 소비해보기
1. Queue 생성
   Queues and Streams 탭에서 > Add a new queue > Name: CREATE_POST_QUEUE > Add queue 버튼 선택

2. 메시지 발행
   생성된 queue를 누르고(name) > publish message > payload에 작성
   {
   "content": "my post"
   }
   -> popup 확인(Message published)

3. 메시지 확인
   get messages 탭에 > Get Message(s) 버튼눌러서 확인해보기

---

#### io-bound-application에 적용
1. 설정추가
- maven spring boot amqp 의존성 추가
- application에 rabbitmq 설정 추가

2. Producer와 Consumer 작성
   (https://github.com/psy3720/io-bound-application/commit/47ed4b7c15c253d56a11d402e1da47cc2e0f9bbd)

3. RabbitMQ 인스턴스 생성
```
인스턴스명: rabbitmq-instance-1
리전:도쿄
E2 Medium Centos7 20gb 표준영구디스크 
```

RabbitMQ에서 사용할 방화벽 규칙을 추가한다.  
방화벽 정책 -> 방화벽 규칙 만들기 -> 방화벽 규칙 추가(name:rabitmq)  
5672, 15672 tcp

```
# docker 설치
> sudo yum install docker

# docker 실행
> sudo systemctl start docker

# 권한 수정
> sudo chmod 666 /var/run/docker.sock

# rabbitmq 실행
> docker run -d --hostname my-rabbit --name some-rabbit -p 5672:5672 -p 15672:15672 rabbitmq:3-management // 컨테이너 실행
```
*예제의 rabbitmq 버전이 안맞아 다음과 같이 사용함(centos7 docker 1.13.1, rabbitmq:3.8.9)

레빗MQ 웹 접속하여 QUEUE를 생성한다.  
Queues > Add a new queue > name(CREATE_POST_QUEUE)

4. 수정사항 반영
application.yaml에 rabbitmq 내부ip로 수정한다.  
insert-message-queue으로 브랜치를 만든 후 변경된 내용을 pull request를 사용하여 추가한다.

*cache-first-page 생략  
(https://github.com/psy3720/io-bound-application/commit/33fe797502b917878e48131cbffde00eacdaedd5)

---

#### 스트레스 테스트 - 글 작성 요청은 실패하지 않을까?
RabbitMQ로 글쓰기 기능을 개선했으나 요청이 모두 성공하지 않는다.

실습에서 만든 인스턴스는 다음과 같이 위치한다.
각각의 인스턴스간에 ping을 날려서 확인해보면 한국-> 대만(60~70ms), 한국->일본(30~40ms)정도 지연시간이 걸린다.

같은 리전에 있는 인스턴스끼리는 1ms 정도밖에 걸리지 않는다.

Nginx 인스턴스를 서울에 새로 만들고, Nginx 인스턴스에 RabbitMQ도 같이 올린다.
그러면 모든 인스턴스가 서울 리젼에 있어 인스턴스끼리 Latency가 매우 낮아진다.

이렇게 인스턴스를 위치시키면 요청 한건을 처리하는데 드는 시간이 줄어든다.

(실무에서는 같은 인스턴스에 두 개 이상의 컨테이너를 띄우는게 좋지 않다. 테스트 환경이라면 괜찮으나, 운영환경이라면 한쪽 장애가 다른쪽에 영향을 미칠수 있다.)

```
# 3번인스턴스를 제거한다.
> sudo vi /etc/nginx/nginx.conf

# nginx 실행
> sudo systemctl start nginx

# docker 설치 및 실행
> sudo yum install -y docker
> sudo systemctl start docker

# 권한 변경
> sudo chmod 666 /var/run/docker.sock
```

새로 변경된 rabbitmq host ip를 수정한다.(feat: rabbitmq host 변경)
스트레스 테스트 진행

### nofile errors 해결
```
# nginx pid 확인
> ps aux | grep nginx

# nofile limit 확인 및 변경
> sudo prlimit --nofile --output RESOURCE,SOFT,HARD --pid {PID}
> sudo cat /proc/sys/fs/file-max
> sudo prlimit --nofile=100000 --pid={PID}
```

스트레스 테스트를 하면서 Latency를 줄이고, Error 로그를 확인하여 서버 설정을 변경하여 해결하였다.

---