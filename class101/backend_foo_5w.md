### 2023.08.30

이 내용은 크리에이터 푸님의 "현직 대기업 개발자 푸와 함께하는 진짜 백엔드 시스템 실무!" 강의를 기반으로 정리하였습니다.

강의 링크  
(https://class101.net/ko/products/T6HT0bUDKIH1V5i3Ji2M)
---

#### I/O bound 애플리케이션도 서버를 늘리면 성능을 올릴 수 있을까?
- CPU bound 애플리케이션과 I/O bound 애플리케이션의 차이에 대해 이해하기
- I/O의 종류별로 서버를 늘려 애플리케이션의 성능을 올릴 수 있을지 알아보기

하드디스크를 많이 사용하는 I/O bound 애플리케이션이라면 서버를 늘려서 성능향상이 가능할 수 있다.

DB가 병목이라면서버를 늘려도 DB의 성능에 의존적이기 때문에 성능향상이 어렵다.

---
#### 한줄게시판 실습
- GitHub에서 포크받아 DB를 이용한 한줄 게시판 기능을 이해하기
- Postman으로 한줄 게시판의 API를 테스트
- GCP 인스턴스에 io-bound-application을 배포하기

아래의 주소에서 샘플코드를 fork한다. 그리고 sourcetree에 clone한다.
(https://github.com/lleellee0/io-bound-application)

```
# postgresql 실행(docker 사용)
> docker run --name pgsql -d -p 5432:5432 -e POSTGRES_USER=postgresql -e POSTGRES_PASSWORD=postgrespassword -e POSTGRES_INITDB_ARGS="--data-checksums -E utf8 --no-locale" postgres
```

- postman을 다운로드하고 아래의 api를 테스트해본다.

```
http method: post
url: localhost:8080/post
body: {
	"content": "my post."
}
```

요청을 보내고 조회되는지 확인한다. (http://localhost:8080/posts) (get)

---
#### DB용 인스턴스를 생성

```
인스턴스명: postgresql-instance-1
리전: 타이완
e2 medium/표준영구디스크 centos7
```

GCP인스턴스에서는 DB와 애플리케이션 코드가 다른곳에 올라가기 때문에 postgresql에 해당하는 GCP 내부 IP로 application.yaml을 수정한다.

```
# docker 설치
> sudo yum install -y docker

# docker 실행
> sudo systemctl start docker

# 권한 변경
> sudo chmod 666 /var/run/docker.sock

# docker로 postgresql 실행
> docker run --name pgsql -d -p 5432:5432 -e POSTGRES_USER=postgresql -e POSTGRES_PASSWORD=postgrespassword postgres:11
```
*버전차이로 인해 에러가 발생해 11버전 명시함.

도커 볼륨이란?
> 볼륨이란 postgresql과 공유하는 내부 저장소 개념
*예제에서는 편의상 cpu 워커 인스턴스를 재사용
*실무에서 사용하는 경우 볼륨을 잡아두고 로그를 남기거나 DB있는 데이터를 저장해야한다.

#### 젠킨스 설정
1. 젠킨스 웹접속 > 기존 item cpu-bound-application deploy로 변경
2. 새로운 item 생성 copy -> cpu-bound-application deploy
3. io-bound-application deploy로 수정
4. github repository 수정 -> fork받은 io 레포지토리로
5. 로컬에서 maven jar파일 생성해보고 같은 이름으로 source files, execute command 수정
6.  postgresql instance 내부아이피로 수정한 yaml 커밋 and push
7. 젠킨스 웹에서 Build now눌러 수동으로 배포하여 확인해보기
8. nginx인스턴스 외부ip/posts 접속하여 확인한다.

---

#### 기능 추가
- 페이징
- 글번호/글내용으로 검색기능 추가

깃플로우 전략을 사용하여 각 기능을 개발한다.
개발 후 병합시 pull request를 사용해본다.

수정코드는 아래의 레포지토리를 참고한다.
(https://github.com/lleellee0/io-bound-application)

* pull request시 Diff view를 Split모드로 하면 변경내용을 보기 편하다.
---

#### 스트레스 테스트 하기
artillery를 사용하여 스트레스 테스트를 진행한다.

1. 아래의 링크에서 데이터 셋 링크를 다운 받는다.
   한줄 게시판에 작성될 법한 데이터 셋 링크 : https://github.com/lleellee0/class101-files
   (데이터셋은 구글링을 통해 구할 수 있다.)

데이터 가공을 위해 google docs 스프레드시트 사용

Google Docs 스프레드시트 링크 : https://docs.google.com/spreadsheets/u/0/

2. 데이터셋을 업로드하고 가공한다.
- A,B열 삭제
- column name을 content로 변경

3. 가공된 데이터를 다시 저장한다.
   파일 > 다운로드 > csv로 저장


#### io용 테스트 yaml 작성
1. cpu-test.yaml 생성

2. 테스트 스크립트 작성
```
# io-test.yaml 파일 (초기 형태)
config:
  target: "http://{nginx-ip}"
  phases:
    - duration: 333
      arravalRate: 3
      name: Warm up
  payload:
    path: "ratings_test_1k.csv"
    fields:
      - "content"
scenarios:
  - name: "just post content"
    flow:
      - post:
          url: "/post"
          json:
            content:  "{{ content }}"

# io-test.yaml 파일 (7분 12초 고도화된 형태)
config:
  target: "http://{nginx-ip}"
  phases:
    - duration: 60
      arrivalRate: 3
      name: Warm up
    - duration: 120
      arrivalRate: 3
      rampTo: 100
      name: Ramp up load
    - duration: 600
      arrivalRate: 100
      name: Sustained load
  payload:
    path: "ratings_test_10k.csv"
    fields:
      - "content"
scenarios:
  - name: "just post content"
    flow:
      - post:
          url: "/post"
          json:
            content:  "{{ content }}"
```




