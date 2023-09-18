### 2023.08.15

이 내용은 김영한님의 "스프링 부트 - 핵심 원리와 활용" 강의를 기반으로 정리하였습니다.

---

#### 메트릭 등록1 - 카운터
마이크로미터를 사용해서 메트릭을 직접 등록한다. 주문수, 취소수를 대상으로 카운터 메트릭을 등록한다.


**MeterRegistry**
:  마이크로미터 기능을 제공하는 핵심 컴포넌트

**Counter**:
- 단조롭게 증가하는 단일 누적 측정항목
  - 단일값
  - 보통하나씩 증가
  - 누적이므로 전체값을 포함(total)
  - 프로메테우스에서는 일반적으로 카운터의 이름 마지막에 _total을 붙여서 my_order_total과 같이 표현한다.
- 값을 증가하거나 0으로 초기화 하는 것만 가능하다.
- 마이크로미터에서 값을 감소하는 기능도 지원하지만, 목적에 맞지 않는다.

Counter 생성 예시
```
 Counter.builder("my.order")
                .tag("class", this.getClass().getName())
                .tag("method", "cancel")
                .description("order")
                .register(registry).increment();
```
- Counter.builder("name"): name에는 메트릭 이름을 지정한다.
- tag는 프로메테우스에서 필터할 수 있는 레이블로 사용된다.

*주문과 취소를 각각 한번씩 실행한 후에 메트릭을 확인한다.
(http://localhost:8080/actuator/metrics/my.order)

*프로메테우스 포멧 메트릭
(http://localhost:8080/actuator/prometheus)
- 메트릭 이름이 my.order -> my_order_total로 변경된 것을 확인할 수 있다.
- 프로메테우슨느 . -> _로 변경한다.
- 카운터는 마지막에 _total을 붙인다. 프로메테우스는 관례상 카운터 이름의 끝에 _total을 붙인다.

#### 그라파나 등록 - 주문수,취소수
카운터는 계속 증가하기 때문에 특정 시간에 얼마나 증가했는지 확인하기 위해 increase() 함수를 사용해서 등록한다.
- increase(my_order_total{method="order"}[1m])
- increase(my_order_total{method="cancel"}[1m])

---

**메트릭 등록2 - @Counted**
: 앞서 만든 방식은 메트릭을 등록하고 관리하는 로직이 핵심 비즈니스 개발 로직에 포함되어 있기 때문에 분리해서 관리하는 것이 좋다.
스프링 AOP를 사용하여 해결할 수 있다. 마이크로미터는 필요한 AOP 구성요소를 이미 만들어두었다.

```
@Counted("my.order")
@Override
public void order() {
	log.info("주문");
	stock.decrementAndGet();
}

@Counted("my.order")
@Override
public void cancel() {
	log.info("취소");
	stock.incrementAndGet();
}
```
@Counted 어노테이션을 사용하려면 CountedAspect를 등록해야 한다.  

!!주의 CountedApsect를 빈으로 등록하지 않으면 @Counted 관련 AOP가 동작하지 않는다.

```
import io.micrometer.core.aop.CountedAspect;

@Bean
public CountedAspect countedAspect(MeterRegistry registry) {
	return new CountedAspect(registry);
}
```

액츄에이터 메트릭을 확인
(http://localhost:8080/actuator/metrics/my.order)

프로메테우스 포멧 메트릭 확인
(http://localhost:8080/actuator/prometheus)

그라파나 대시보드 확인
메트릭 이름과 tag가 기존과 같으므로 같은 대시보드에서 확인 가능하다.

---
#### 메트릭 등록3 - Timer
**Timer**  
Timer는 특별한 메트릭 측정 도구인데, 시간을 측정하는데 사용된다.
카운터와 유사한데, Timer를 사용하면 실행 시간도 함께 측정할 수 있다.
Timer는 다음과 같은 내용을 한번에 측정해준다.
- seconds_count: 누적 실행 수 - 카운터
- seconds_sum: 실행 시간의 합 - sum
- seconds_max: 최대 실행 시간(가장 오래걸린 실행시간) - 게이지
- 내부에 타임 윈도우라는 개념이 있어서 1~3분 마다 최대 실행 시간이 다시 계산된다.

```
Timer timer = Timer.builder("my.order")
                .tag("class", this.getClass().getName())
                .tag("method", "order")
                .description("order")
                .register(registry);

        timer.record(() -> {
            log.info("취소");
            stock.incrementAndGet();
            sleep(200);
        });
```
- Timer.builder(name)을 통해서 타이머를 생성한다. name에는 메트릭 이름을 지정한다.
- tag를 사용할때는, 프로메테우스에서 필터할 수 있는 레이블로 사용된다.
- 타이머를 사용할 때는 timer.record()를 사용하면 된다. 그안에 시간을 측정할 내용을 함수로 포함하면 된다.

주문과 취소를 각각 한번씩 실행한 다음 메트릭을 확인한다.
- 주문:(http://localhost:8080/order)
- 취소:(http://localhost:8080/cancel)
- 메트릭: (http://localhost:8080/actuator/metrics/my.order)

#### 프로메테우스 포멧 메트릭 확인
(http://localhost:8080/actuator/prometheus)

프로메테우스로 다음 접두사가 붙으면서 3가지 메트릭을 제공한다.
- seconds_count: 누적 실행 수
- seconds_sum: 실행 시간의 합
- seconds_max: 최대 실행 시간(가장 오래걸린 실행 시간), 프로메테우스 gauge  
 *참고: 내부에 타임 윈도우라는 개념이 있어서 1~3분 마다 최대 실행시간이 다시 계산된다.

평균 실행 시간도 계산할 수 있다.  
seconds_sum / seconds_count = 평균 실행시간

---

#### 메트릭 등록4 - @Timed
타이머는 @Timed라는 애노테이션을 통해 AOP를 적용할 수 있다.

```
@Timed("my.order")
@Slf4j
public class OrderServiceV4 implements OrderService {
```

```
@Bean
public TimedAspect timedAspect(MeterRegistry registry) {
	return new TimedAspect(registry);
}
```

주문과 취소를 각각 한번씩 실행한 다음에 메트릭을 확인한다.  
(http://localhost:8080/order)
(http://localhost:8080/cancel)
(http://localhost:8080/actuator/metrics/my.order)
(http://localhost:8080/actuator/prometheus)

---
**메트릭 등록5 - 게이지**
Gauge(게이지)
: 게이지는 임의로 오르내릴 수 있는 단일 숫자 값을 나타내는 메트릭
값의 현재 상태를 보는데 사용한다.
값이 증가하거나 감소할 수 있다.
예)차량의 속도, CPU 사용량, 메모리 사용량

참고: 카운터와 게이지를 구분할 때는 값이 감소할 수 있는가를 고민해보면 도움이 된다.

```
@Bean
public MeterBinder stockSize(OrderService orderService) {
	return registry -> Gauge.builder("my.stock", orderService, service -> {
		log.info("stock gauge call");
		return service.getStock().get();
	}).register(registry);
}
```
**!!메트릭은 100% 정확한 숫자를 보는데 사용하는 것이 아니다. 약간의 오차를 감안하고 실시간으로 대략의 데이터를 보는 목적으로 사용한다.**

