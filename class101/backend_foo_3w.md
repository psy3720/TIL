### 2023.08.28

이 내용은 크리에이터 푸님의 "현직 대기업 개발자 푸와 함께하는 진짜 백엔드 시스템 실무!" 강의를 기반으로 정리하였습니다.

강의 링크  
(https://class101.net/ko/products/T6HT0bUDKIH1V5i3Ji2M)
---

#### 무중단 배포
> 무중단 배포는 배포를 중단없이 하는 것이다.

배포: 개발환경에서 개발된 코드를 패키징하여 서버에서 새로운 버전의 애플리케이션을 실행하도록 하는것을 의미한다.

이전 버전 애플리케이션을 종료시키고, 새로운 버전 애플리케이션을 실행하고 새로운 버전 애플리케이션이 요청을 받을 준비가 될 때까지 서비스가 중단된다.  
이렇게 서비스가 중단되는 시간을 다운타임이라고 한다.

새로운 버전의 애플리케이션을 시작하기 전에 이전 버전 애플리케이션을 내려야 하는 이유는 같은 포트를 사용하기 때문이다.  
하나의 서버에서 하나의 포트를 동시에 서로 다른 애플리케이션이 사용하는 것은 불가능하다.


#### 서버를 두 개로 늘린다면?  
서버가 두개 가 되면 사용자는 두 서버 모두의 IP 또는 도메인 주소를 알아야 한다.  
그리고 두 서버중 어떤게 배포되고 있는지 알 수 도 없기 때문에 애플리케이션 서버와 클라이언트 사이에 중계 해줄 서버가 필요하다.  

리버스 프록시는 애플리케이션 서버와 사용자 사이에서 요청을 중계해주는 서버이다.  
클라이언트는 서버를 대리하는 존재만 알게 된다.  

- 리버스 프록시를 사용하게 되면 트래픽도 분산할수 있음.(로드밸런싱)  
  반대로 프록시(대리자)는 클라이언트를 숨겨주는 역할을 한다.  
  현재 예제에서는 nginx를 사용한다.

#### Nginx란?
> Nginx는 오픈 소스 웹 서버 소프트웨어 및 리버스 프록시 서버 소프트웨어로, 웹 서버 및 애플리케이션 배포에 널리 사용된다.  
높은 성능, 안정성, 확장성을 가지고 있어 웹 호스팅, 로드 밸런싱 캐싱, SSL/TLS 암호화 및 웹 애플리케이션 서버와 함께 사용되는 다양한 웹 기반 서비스에 적합하다.

특징
- 성능 및 확장성: Nginx는 매우 빠른 속도로 웹 페이지를 제공하며, 많은 동시 접속자를 처리할 수 있는 확장성을 제공한다.  
- 리버스 프록시: Nginx는 웹 요청을 받아서 다른 서버로 전달하는 리버스 프록시로 사용된다. 이를 통해 애플리케이션 서버, 데이터베이스 서버 등과의 연결을 관리하고 부하 분산을 수행할 수 있다.
- 로드밸런싱: Nginx는 여러 서버간의 트래픽을 분산하여 서버의 부하르 고르게 분산시키는 로드 밸런싱 기능을 제공한다.
- 정적 및 동적 콘텐츠 서빙: Nginx는 정적 파일(이미지, CSS, Javascript 등)과 동적 웹페이지 요청 모두를 처리할 수 있다.
- SSL/TLS 암호화: HTTPS를 통한 암호화를 지원하므로 보안 연결을 설정하는 데 사용된다.
- 캐싱: Nginx는 캐시 기능을 제공하여 반복적인 요청에 대한 응답을 캐시하고 웹 서버의 부하를 줄인다.

Nginx는 웹 서버로 사용될 뿐만 아니라, 애플리케이션 서버와 함께 사용하여 웹 애플리케이션을 배포하고 관리하는 데도 널리 사용된다.  
여러가지 배포 방식중 예제에서는 롤링(Rolling) 배포 방식을 사용한다.

Question
1. 트래픽이 매우 많아져도 nginx만으로 충분한가요?
   nginx에도 처리할수 있는 트래픽의 양이 있기 때문에 충분하지 않다.  
   현재 구성의 병목에 따라 nginx가 실행되는 서버 scale-up, 네트워크 장치로 로드밸런싱, DNS 리다이렉션 방식등을 사용할 수 있다.  


2. 롤링으로 배포하다가 배포에 실패하면 어떻게 하나요?  
   서비스 환경에 따라 다르지만 다음 두가지를 고민해보기.  
    - 배포성공여부 체크(배포스크립트 자체나 새로 배포된 서비스에 특정 api요청을 했을 때 응답이오는 제3의 서비스)  
    - 수동롤백 vs 자동롤백(배포가 잦다면 자동롤백을 고려)

---

#### 무중단 배포환경 만들기 실습

머신 이미지: 동일한 인스턴스를 생성해주는 기능(snapshot이라고도 부른다)

#### 머신이미지 만들기
이름: cpu-worker-instance-image  
설명: cpu-worker-instance의 이미지입니다.

이미지로부터 인스턴스를 생성한다(worker 인스턴스 2~3)  
source vm instance -> cpu-worker-instance
```
인스턴스명:cpu-worker-instance-2~3
리전:서울
```
---
#### 젠킨스 설정
새로 생성한 워커인스턴스를 젠킨스에 등록한다.

```
# 젠킨스 시작
> sudo systemctl start jenkins

# 젠킨스 상태 확인
> sudo systemctl status jenkins
```

- 젠킨스 외부ip로 접속하여 확인
- 젠킨스관리 > 시스템설정 > 제일 아래에 cpu-worker-instance-2,3 추가(인스턴스1과 동일 ip,name만 다르게 작성)
- 젠킨스인스턴스의 공개키를 각 worker instance에 복사

```
# 젠킨스 공개키 확인
> cat .ssh/id_rsa.pub 

# 각 워커 인스턴스에 추가
> vi .ssh/authorized_keys 

# 권한 변경
> chmod 600 ~/.ssh/authorized_keys
```

각 인스턴스의 ssh로 접속하여 공개키를 등록하지 않아도 gcp의 기능을 사용하여 등록해두면 관리 가능하다.  
GCP 접속 > 메타데이터 > ssh키 > 수정버튼 선택 > 항목추가 버튼 선택 > jenkins 공개키 복사 > 저장버튼 선택

젠킨스 웹으로 돌아가 test Configuration 확인 > 저장버튼 선택

#### 로그 추가 빌드 결과
애플리케이션의 로그를 남기기 위해 스크립트 명령어를 수정한다.  
/dev/null을 nohup.out으로 수정 > build now 버튼 선택

```
# 각 인스턴스의 콘솔로 들어가 로그 확인해보기
> tail -f nohup.out

# 인스턴스 2,3으로 접속하여 docker 시작한다
> sudo systemctl start docker
> sudo chmod 666 /var/run/docker.sock
```

---
#### nginx 인스턴스 만들기
로드밸런싱을 하기 위한 Nginx 인스턴스를 생성한다.

```
인스턴스명:nginx-instance-1
리전: 타이완
e2-medium
centos 7(20gb, 표준영구디스크)
```

```
# nginx 설치하기
> sudo yum install nginx 

# nginx 실행
> sudo systemctl start nginx 
```
nginx를 실행한 후 외부IP로 접속해 nginx 실행을 확인한다.


#### 로드밸런싱 설정 추가
```
# 설정파일 열기
> sudo vi /etc/nginx/nginx.conf 
```

- include와 server 사이에 upstream 추가됨.
- server에 proxy 설정 추가

```
upstream cpu-bound-app {
  server {instance_1번의_ip}:8080 weight=100 max_fails=3 fail_timeout=3s;
  server {instance_2번의_ip}:8080 weight=100 max_fails=3 fail_timeout=3s;
  server {instance_3번의_ip}:8080 weight=100 max_fails=3 fail_timeout=3s;
}

location / {
  proxy_pass http://cpu-bound-app;
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection 'upgrade';
  proxy_set_header Host $host;
  proxy_cache_bypass $http_upgrade;
}
```

[참고]
Nginx Load Balancing Docs:( https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)

로드밸런싱 여부 확인 외부IP/hash/123 접속시 에러가 발생한다.

#### connect 에러 해결
```
# nginx 리로드
> sudo systemctl reload nginx 

# nginx 로그 확인
> sudo tail -f /var/log/nginx/error.log 

# 보안정책 조정 명령  
> sudo setsebool -P httpd_can_network_connect on
```
---

#### 스트레스 테스트
로드밸런싱 환경을 구성한 후 instance가 1~3개일 때의 스트레스 테스트를 진행한다.
인스턴스를 하나씩 내렸다가 올렸다가 실패하는 요청을 확인한다.
artillery를 사용한다.

nginx 외부ip/hash/123 로드밸런싱 요청 확인하기
visual studio code에서 yaml에 target 외부ip nginx로 지정

```
duration 360
arrivalRate 1
url 변경 > /hash/123
```

```
# 스트레스 테스트 실행
> artillery.cmd run --output report.json cpu-test.yaml  
> artillery.cmd report .\report.json
```
arrivalRate 8,16 등으로 변경하여 테스트를 진행해본다.

#### 500에러와 502에러
두 에러는 웹 서버와 웹 애플리케이션 간의 통신 문제로 인해 발생하는 HTTP 상태코드이다.

1. 500 Internal Server Error(내부 서버 오류):
   웹서버 또는 웹애플리케이션 서버 내부에서 오류가 발생한 경우에 반환된다.
   예를 들면 코드버그, 구성오류, 서버 리소스 부족 등이 이에 해당한다.

2. 502 Bad Gateway(게이트웨이 오류):
   웹 서버가 다른 서버와 통신하려고 시도했지만, 이 서버로부터 잘못된 응답을 받았거나 응답을 받지 못한 경우에 발생한다.
   이 오류는 웹 서버가 요청을 처리하기 위해 다른 서버로 전달하려고 할 때 중간 서버 또는 게이트웨이 서버(Nginx, Apache 등..)에서 발생한다.

nginx에서 애플리케이션 서버가 응답하지 않는 상태가 됐을 때 502에러가 발생한다. (hang 상태 -> nginx는 요청을 받을수없다고 생각하고 연결을 끊는다)

---
#### arrivalRate 8로 변경후 instance 1만 살려두기
테스트를 진행한 후 1번 인스턴스를 제외하고 나머지는 종료시킨 후 스트레스 테스트를 진행한다.
종료시킨 인스턴스는 외부ip로 접속해서 죽어있는지 확인한다.

```
> docker ps

# 특정 애플리케이션 찾기
> docker ps | grep spring-boot-cpu-bound  

# 스스로 애플리케이션이 하던일을 마저하고 종료하도록 kill
> docker container kill -s 15 컨테이너ID 
```

로드밸런싱 되는 서버가 죽어서 응답이 없는 경우 그 요청을 버리지 않고, 다른 서버에 요청하여 응답을 받아온다.
이처럼 실패한 요청에 대해 정상적으로 처리될 수 있도록 자동으로 응답 있는 서버로 요청을 포워딩 해주는 것을 페일오버라고 한다.
