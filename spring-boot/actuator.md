### 2023.08.10 (수)

이 내용은 김영한님의 "스프링 부트 - 핵심 원리와 활용" 강의를 기반으로 정리하였습니다.

----

### 액츄에이터

스프링 부트에서 제공하는 액츄에이터는 애플리케이션의 운영 환경에서 유용한 기능들을 제공하는 라이브러리이다.
이를 통해 애플리케이션의 상태를 모니터링하고, 관리하며, 디버깅하는 데 도움을 준다.

액츄에이터는 RESTful 엔드포인트를 통해 다양한 정보를 노출하며, 이를 통해 애플리케이션을 모니터링하고 관리할 수 있다.

----

#### 시작하기

build.gradle 추가

```
implementation 'org.springframework.boot:spring-boot-starter-actuator' 
```

동작확인

- 메인 클래스 실행(ActuatorApplication.main())
- http://localhost:8080/actuator 접속

액츄에이터 기능을 웹에 노출하려면 application.yml에 다음과 같이 추가한다. 그러면 액츄에이터가 제공하는 수 많은 기능을 확인할 수 있다.

```
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

액츄에이터가 제공하는 기능은 엔드포인트를 통해 접근할 수 있다.
(http://localhost:8080/actuator/{엔드포인트명})
과 같은 형식으로 접근할 수 있다.

엔드포인트를 사용하려면 엔드포인트 활성화 및 노출을 해야한다.
- 엔드포인트 활성화: 기능 on/off
- 엔드포인트 노출: http/jmx 노출여부

>엔드포인트는 기본으로 활성화되어 있지만 shutdown 엔드포인트는 엔드포인트 활성화와 노출이 둘다 적용되어야 사용가능하다.

---
#### 자주 사용하는 엔드포인트들..

### 1. 헬스 정보(/actuator/health)
> 이 엔드포인트를 통해 애플리케이션의 기본적인 동작 여부를 확인하는데 사용한다.
기본 동작에는 애플리케이션이 요청에 응답할 수 있는 것 뿐만 아니라, 사용하는 데이터베이스의 응답, 디스크 사용량에 대한 정보를 포함한다.

\
헬스 정보를 자세히 보거나 간략히 보려면 다음과 같은 옵션을 지정한다.
```
management.endpoint.health.show-details=always  // detail
management.endpoint.health.show-components=always // simple
```
헬스 컴포넌트(db, diskSpace, ping) 중 하나라도 문제가 생기면 전체 상태는 down 상태가 된다.

*****

### 2. 애플리케이션 정보(/actuator/info)
>  애플리케이션의 기본 정보를 노출한다.

다음과 같은 기능을 제공한다.
 - java: 자바 런타임 정보
 - os: OS 정보
 - env: Environment에서 info.로 시작하는 정보
 - build: 빌드정보(META-INF/build-info.properties 파일 필요)
 - git: git 정보(git.properties 파일 필요)


####  application.yml - 내용 추가
```
management:
  info:
    java:
      enabled: true
    os:
      enabled: true
```

build

빌드 정보를 노출하려면 빌드시점에 META-INF/build-info.properties 파일을 만들어야 한다.
```
 springBoot {
    buildInfo()
 }
```

git

git 정보를 노출하려면 git.properties 파일이 필요하다.

```
plugins {
    ...
    id "com.gorylenko.gradle-git-properties" version "2.4.1" //git info
}
```
*****
### 3. 로거(/actuator/loggers)
> 로깅과 관련된 정보 확인 및 실시간 변경 가능

확인하려면 
(http://localhost:8080/actuator/loggers) 또는
(http://localhost:8080/actuator/loggers/{로거이름}) 과 같이 요청하면 더 자세히 볼 수 있다.

결과
```
{
    "configuredLevel": "DEBUG",
    "effectiveLevel": "DEBUG"
}
```

실시간 로그 레벨 변경하기 위해서는 POST 요청을 보낸다.
post 요청 (http://localhost:8080/actuator/loggers/hello.controler)
content/type: application/json
```
{
    "configuredLevel": "TRACE"
}
```
---
### 4. HTTP 요청 응답 기록(/actuator/httpexchanges)
> HTTP 요청과 응답의 과거 기록을 확인하고 싶을 때 사용한다. HttpExchangeRepository 인터페이스의 구현체를 빈으로
> 등록하면 사용할 수 있다.

스프링 부트는 기본으로 InMemoryHttpExchangeRepository 구현체를 제공한다.

``` 
 import org.springframework.boot.actuate.web.exchanges.InMemoryHttpExchangeRepository;
 import org.springframework.context.annotation.Bean;

 @Bean
 public InMemoryHttpExchangeRepository httpExchangeRepository() {
    return new InMemoryHttpExchangeRepository();
 }
```
이 구현체는 최대 100개의 HTTP 요청을 제공한다. 최대 요청이 넘어가면 과거 요청을 삭제한다.

---

### !! 액츄에이터와 보안

#### 보안 주의
액츄에이터가 제공하는 기능들은 애플리케이션의 내부정보를 너무 많이 노출하기 때문에 엔드포인트를 공개하는것은
좋은 방법이 아니다.
따라서 내부에서만 접근 가능하도록 내부망을 사용하는 것이 안전하다.

#### 액츄에이터 포트 설정방법
```
 management.server.port=1111
```
#### 액츄에이터 URL 경로에 인증 설정
포트를 분리하는 것이 어려운 경우 인증된 사용자만 접근가능하도록 추가 개발 필요
(서블릿 필터, 스프링 인터셉터 or 스프링 시큐리티 사용)


엔드포인트 기본 경로 변경 옵션
```
management:
  endpoints:
    web:
      base-path: "/manage"
```
/actuator/{endpoint} 에서 /manage/{endpoint}로 변경된다.