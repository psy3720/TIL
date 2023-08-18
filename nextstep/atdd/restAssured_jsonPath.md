(주)넥스트스텝에서 류성현 강사님이 진행하는 ATDD, 클린 코드 with Spring 7기로 참여하며 배웠던 것들을 정리한다.

교육일정은 2023.06.26~ 2023.08.09까지 진행되었다.  

과정 진행 방식은 미션수행 -> 온라인 강의 수강 -> 피드백/코드리뷰를 반복하며 진행하고 각 주차별 미션을 수행하면 된다.

ATDD, 클린 코드 with Spring은 스프링 웹 애플리케이션을 개발하는 과정에서 ATDD(인수 테스트 주도 개발)프로세스를 경험하고 클린코드 작성과 인수 테스트 기반 리팩터링에 대해 고민해보는 과정이다.

**강의링크**
(https://edu.nextstep.camp/c/R89PYi5H)

---
#### ATDD란?
> ATDD는 테스트 가능한 요구사항으로 소프트웨어를 개발하는 방법 중 하나이다.  
요구사항을 시나리오 형태로 정리하여 이를 검증하는 인수 테스트를 만들고, 인수 테스트를 만족시키는 코드를 구현하는 방식으로 진행하는 개발 방법론이다.

> 테스트 작성을 한 뒤 기능을 구현 한다는 점에서 TDD와 비슷한 점이 있지만 기능 단위의 테스트를 작성하는 TDD와 달리 시나리오 단위의 테스트, 인수 테스트를 작성한다는 점에서 차이가 있다.  
이러한 차이를 이용하여 TDD와 ATDD를 함께 진행한다면 훨씬 효과적으로 소프트웨어 개발을 할 수 있다.

---

#### 1주차 정리

**RestAssured**

RestAssured 사용하기
: RestAssured는 자바에서 RESTful 웹 서비스의 테스트를 쉽게 수행할 수 있도록 도와주는 자동화된 테스트 라이브러리이다.  
이 라이브러리는 REST API의 요청과 응답을 검증하고 테스트하는데 사용된다. 주로 API 테스트 자동화 작업에서 많이 활용되며, API의 동작을 검증하고 문제를 찾아내는 데 도움을 준다.

[**참고**]  
(https://rest-assured.io/)

**주요 기능과 특징**
1. 읽기쉬운 DSL(Domain-Specific Language)
: RestAssured는 간결하고 읽기 쉬운 DSL을 제공하여 HTTP 요청 및 응답을 작성하고 검증하는 과정을 쉽게 표현할 수 있다.

2. 자동화된 검증
: JSON, XML 등 다양한 응답 형식의 데이터를 자동으로 파싱하여 검증할 수 있다. 특정 필드의 값을 검증하거나 상태 코드, 헤더 등의 정보를 검증하는 것이 가능하다.

3. 요청과 응답 설정
: 다양한 요청 방식(GET,POST,PUT,DELETE 등..)과 파라미터 설정, 헤더 조작 등이 가능하다. 응답의 다양한 측면을 확인할 수 있다

4. 코드와 테스트의 분리
: 테스트 코드를 작성할 때, HTTP 요청 및 응답 설정을 명확하게 분리하여 테스트 코드를 관리하기 쉽게한다.

5. 자동 로그 기능
: 요청과 응답의 로그를 자동으로 출력하여 테스트 시 어떤 요청과 응답이 발생했는지 확인할 수 있다.

6. JUnit 또는 TestNG와 통합 가능
: 주로 JUnit 또는 TestNG와 통합하여 테스트 슈트를 작성하고 실행하는데 사용된다.


**(예시)**
```
import io.restassured.RestAssured;
import io.restassured.response.ExtractableResponse;
import io.restassured.response.Response;
import org.junit.jupiter.api.Test;
import org.springframework.http.HttpStatus;

public class ApiTest {
	@Test
	public void testGetRequest() {
        ExtractableResponse<Response> response = RestAssured
                .given()
                .when()
                .get("/users/1")
                .then().extract();
			
		assertThat(response.statusCode()).isEqualTo(HttpStatus.OK.value());	
	}
}
```
- get("url")에 요청할 url을 적는다. 예시에서는 /users/1로 요청을 보낸다.
- junit을 사용하여 요청에 대한 응답 상태코드를 검증한다.

---

#### JsonPath
JsonPath는 json 데이터 구조에서 특정 데이터를 추출하거나 쿼리하기 위한 표현식 언어이다.
jsonpath를 사용하면 json응답에서 원하는 데이터를 선택하고 추출할 수 있다.

자세한 문법은 다음 사이트를 참고한다.
(https://github.com/json-path/JsonPath)

다음은 간단한 예시이다. 예를 들어, json 형식의 응답이 다음과 같다고 가정해본다.

```
{
  "id": 123,
  "name": "John Doe",
  "email": "john.doe@example.com",
  "address": {
    "street": "123 Main St",
    "city": "New York",
    "zipcode": "10001"
  },
  "phoneNumbers": ["123-456-7890", "987-654-3210"],
  "active": true
}
```

그러면 다음과 같이 검증할 수 있다.
```
import io.restassured.response.Response;
import org.junit.jupiter.api.Test;

import static io.restassured.RestAssured.given;
import static org.assertj.core.api.Assertions.assertThat;

public class ExampleAPITest {

    @Test
    public void testGetUserInfo() {
        // GET 메서드 요청을 보내고 응답을 받는다
        Response response = given()
                .when()
                .get("https://api.example.com/users/123");

        // 응답 상태 코드 검증
        response.then().statusCode(200);

        // JsonPath를 사용하여 데이터 추출
        String name = response.jsonPath().getString("name");
        String city = response.jsonPath().getString("address.city");
        String firstPhoneNumber = response.jsonPath().getString("phoneNumbers[0]");
        boolean isActive = response.jsonPath().getBoolean("active");

        // 데이터 검증
        assertThat(name).isEqualTo("John Doe");
        assertThat(city).isEqualTo("New York");
        assertThat(city).isEqualTo("123-456-7890");
        assertThat(isActive).isTrue();
    }
}

```
위의 예제에서는 RestAssured의 given() 및 when() 메서드를 사용하여 GET 요청을 보내고, jsonPath()를 사용하여 응답 데이터를 추출한다. 이후에 추출한 데이터를 검증하는 코드이다.

---

기본 데이터 타입 뿐만 아니라 사용자 정의 클래스에도 적용이 가능하다.  
예를 들어 다음과 같은 JSON 응답이 있고

```
{
  "data": [
    {
      "id": 1,
      "name": "Alice"
    },
    {
      "id": 2,
      "name": "Bob"
    }
  ]
}
```

User Class가 다음과 같이 정의되어 있다고 할 때
```
public class User {
    private int id;
    private String name;

    // Getters and setters
}
```

응답에서 JsonPath를 사용하여 List<User> 형태로 추출할 수 있다.
```
@Test
void testUsersSize() {
    Response response = RestAssured.get("https://api.example.com/data");
	
    List<User> users = response.jsonPath().getList("data", User.class);
	
    assertThat(users.size()).isEqualTo(2);
}
```
 

#### [참고]
  - (https://www.baeldung.com/guide-to-jayway-jsonpath)  
  - (https://jsonpath.com/)