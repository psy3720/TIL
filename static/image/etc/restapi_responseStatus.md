rest api 구현 시 상태값 반환 관련 고민(에러)

상황
나이가 20살 이상인 경우 통과하는 로직 -> 나이값이 15살이 들어온 경우 어떤 상태값을 반환해야 할까?

정리
- 5xx를 제외한 400이나 200으로 처리해야 한다.
- 만약 클라이언트와 서버가 API 스펙시 나이값을 20살 이하로 넘기는 것은 안된다고 정의한 경우 400으로 응답값을 반환해야 한다.

추가 고민
클라이언트가 서버가 요구하는 스펙을 다 지켰을 때 발생하는 비즈니스 예외들은 어떻게 처리해야 할까?

예시 상황: 회원가입 신청시 연령과 관계없이 가입신청을 받아야 하는데, 일반적인 경우는 바로 회원가입이 되지만, 20살 이하인 경우 내부 심사를 위해 보류 상태로 반환해야 하는 경우 어떻게 응답해야 할까?

비슷한 시나리오: 보험금을 지급해달라고 서버에 요청했는데 HTTP API 스펙의 요청은 모두 만족했지만, 비즈니스 로직상 이 사람이 요건을 모두 충족하지 못해서 지급이 승인 거절되는 경우에는 어떻게 응답해야 할까?

두 가지 상황의 경우 클라이언트의 요청에서 서로 약속한 HTTP API 스펙은 만족했지만, 비즈니스 로직까지 정상 수행했지만, 비즈니스 로직의 결과가 성공으로 처리되지 않을 수 있다. 이런 경우 400을 반환하는 것은 클라이언트 개발자에게 잘못 개발했다고 말하는 것과 같다(클라이언트 개발자는 서로 약속한 API 스펙을 다 지켰지만)

따라서 복잡한 비즈니스 로직이 들어가는 경우 200 OK로 응답하면서 결과에 내부 비즈니스 응답 코드를 정의해서 함께 전달한다.
다음과 같이 응답 코드를 만들고 데이터도 한번 감싸서 전달하는 것이다. 이것을 봉투 패턴이라고 한다.

**봉투가 포함된 응답**
 ```
 {
	"code": "success" // "fail", "hold", "deny" ...
	"data": {memberId: ... 결과 데이터} ...
 }
 ```

- 200 OK라는 응답은 HTTP API 스펙상 요청이 성공해서 비즈니스 로직을 정상적으로 실행하는 단계까지 전달된 것으로 기준을 잡는다.
- HTTP 200 응답 코드는 비즈니스 로직의 내부 결과와는 무관하게 클라이언트와 서버간에 요청과 응답이 정상 수행되었는지를 기준으로 잡는다.

정리
- 서로 약속한 HTTP API 스펙을 만족한 경우 -> 200, 만족하지 않은 경우 -> 400
- 비즈니스 로직이 정상 수행 되지만, 다양한 결과가 존재(승인, 거절등..) -> 200+ 비즈니스 코드 반환(봉투패턴)
- 비즈니스 로직을 수행하다가 내부에서 시스템 예외나 NullPointerException 등등 비즈니스와 관계없는 시스템 예외가 발생한 경우 -> 500

*다양한 비즈니스 응답이 있는 복잡한 비즈니스 로직이 있는 HTTP API는 봉투패턴을 고려하고, 비즈니스 로직이 거의 없는 단순한 조회에서는 봉투패턴을 고려하지 않는다.

[참고](https://www.inflearn.com/questions/111465/5xx-%EA%B4%80%EB%A0%A8-%EC%A7%88%EB%AC%B8%EC%9E%88%EC%8A%B5%EB%8B%88%EB%8B%A4