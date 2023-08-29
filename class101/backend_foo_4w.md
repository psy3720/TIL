### 2023.08.29

이 내용은 크리에이터 푸님의 "현직 대기업 개발자 푸와 함께하는 진짜 백엔드 시스템 실무!" 강의를 기반으로 정리하였습니다.

강의 링크  
(https://class101.net/ko/products/T6HT0bUDKIH1V5i3Ji2M)
---

#### 배포자동화
로컬에서 Github로 푸쉬가 이루어지면, Github에서는 Jenkins로 알려준다.  
그 후 jenkins에서는 새로운 코드를 빌드하고 빌드가 완료되면 jar파일로 패키징 후 GCP인스턴스로 배포한다.

#### 샘플코드
cpu-bound-application repository를 fork한다.

젠킨스 웹 접속 > cpu-worker-instance deploy 아이템 > 구성 > 소스코드 관리 git선택 > Repository URL 입력 (fork한 repository url) > 빌드 유발 - github hook trigger for GITScm polling 체크

빌드에서 해야할 일은 소스코드를 저장소에서 다운로드(pull)받고, 그 소스코드의 의존성을 다운받아 애플리케이션으로 바로 실행할 수 있도록 jar파일로 묶는 행위가 포함된다.

#### script 작성하기 전에 로컬에서 확인
spring boot application 접속 후 maven 명령어 확인해보기

```
# clean과 package 명령어 실행 필요
> ./mvnw clean package
```

젠킨스에서 build now 버튼 누르면 권한 에러가 발생한다.  
mvnw 실행 권한이 없는 이유는 windows와 리눅스의 권한 관리 체계가 다르기 때문에 실행권한을 추가해야 한다.

```
# 실행권한 추가
> chmod 544 ./mvnw 
```

젠킨스 설정
- Build Steps > Add build step > Execute shell 체크
- jar파일 target 부터 경로 복사하여 작성한다.
```
remove prefix : target
execute command :
sudo java -jar cpu-**.jar
```  
[참고]
- javac 1.8로 버전 변경(https://mchch.tistory.com/226)

```
# 인스턴스 1,2,3에 자바를 설치
> sudo yum install -y java
```

build now 버튼 클릭시 백그라운드로 실행하지 않으면 빌드가 안끝난걸로 판단하게 된다. 따라서 백그라운드로 실행하는 스크립트로 수정한다.
> nohup 기존스크립트 > nohup.out 2>&1 &

```
# 프로세스 확인
> ps -aux | grep java 

# 프로세스 종료
> sudo kill -9 프로세스ID
```
---

#### github repository로 접속 후 webhook 추가
- jenkins 인스턴스 외부ip/github-webhook
- sourcetree에 git clone받은 후 port 80 -> 8080으로 commit 후 push
  젠킨스 웹에서 push 이벤트가 발생할 경우 build 되는 것 확인
  *기본적으로 /github-webhook으로 전송된다.

---
#### 모든 인스턴스에 lsof 설치(1,2,3)
기존 애플리케이션을 종료시키지 않으면 포트가 이미 사용중이기 때문에 기존 실행중인 애플리케이션을 종료한 후 실행해야 한다.

```
# losf 설치 
> sudo yum install -y lsof 

# 8080 포트를 사용하는 프로세스를 죽이는 명령
> sudo kill -15 ${sudo lsof -t -i:8080}  
```

#### 젠킨스 스크립트에 추가
sudo kill -15 $(sudo lsof -t -i:8080)

여태까지 한 방식은 동시에 모든 인스턴스에 deploy를 진행하므로 무중단 배포라고 하기 힘들다.  
sleep 명령어를 사용하여 동시에 진행하지 않고, 순차적으로 인스턴스에 배포를 진행하면 된다.  
하지만 sleep만으로는 충분하지 않기 때문에 배포 성공시 다음 인스턴스에 배포를 진행하는 배포스크립트를 고민해보기.  
(배포 성공과 실패를 구분하고, 실패시 어떻게 대처할 것인지..)