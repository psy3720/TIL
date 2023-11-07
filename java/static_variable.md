
#### 상수를 static final로 선언하는 이유는?

1. `Static` 변수: 클래스의 정적 변수는 해당 클래스의 모든 인스턴스가 공유하는 변수이다.   
클래스 내의 모든 인스턴스가 `동일한 값`을 가지며 한 번 생성되면 클래스가 로드되는 동안 유지된다. 인스턴스를 생성하지 않고도 클래스 이름으로 직접 접근할 수 있다.


2. `Final` 변수: 변수가 변경될 수 없음을 나타낸다. 한 번 값을 할당하면 해당 값을 변경할 수 없다. 이로써 값의 `불변성`을 보장할 수 있다.

따라서 `static final`로 선언된 상수는 아래와 같은 특징을 가진다.
- 모든 인스턴스가 같은 값을 공유한다.
- 값을 변경할 수 없으므로 `불변성`을 유지한다.
- 인스턴스 생성 없이도 클래스 이름을 통해 접근할 수 있다.
- 메모리 내에 한 번만 할당되므로 메모리 효율적이다.
- `공통된 상수`를 표현할 때 유용하다.


**예시**
```
public class Constants {
	public static final int MAX_VALUE = 100;
	public static final String DEFAULT_NAME = "John";
}
```
위의 예에서 `MAX_VALUE`와 `DEFAULT_NAME`은 모든 인스턴스에서 `동일한 값`을 가지며, 그 값은 변경될 수 없다. 이러한 `상수`는 클래스 내에서 `공통된 설정 값`이나 `상수`를 표현하는데 사용된다.

---
#### static 키워드 없이 final만 사용하면 안되나요?

만약 static 키워드 없이 final만 사용하여 상수를 선언한다면, 해당 상수는 인스턴스마다 독립된 값을 가지게 된다.  
즉, 인스턴스 마다 값이 다를 수 있으며, 값을 변경할 수 없는 불변 상태를 나타낸다.
이러한 상수는 객체의 특정한 속성을 나타내거나 설정값을 나타내는데 사용될 수 있다.

**다음은 static을 사용하지 않고 final만 사용하여 상수를 정의한 예시이다.**
```
public class Person {
	public final String DEFAULT_NAME = "John";
	
	public static void main(String[] args) {
		Person person1 = new Person();
		Person person2 = new Person();
	}
}
```
위의 예시에서는 인스턴스 간에는 같은 값을 가지지만 `static`을 사용한 상수와 달리 모든 인스턴스가 공유하는 것은 아니다.

`static`을 사용하지 않고 `final`만 사용하여 상수를 정의할 때 주의해야 할 몇가지 문제점이 있다.

1. 메모리 사용: `static final` 상수는 메모리 내에 한 번만 할당 되므로 메모리 사용을 최적화할 수 있다.  
그러나 `final`만 사용한 상수는 각 인스턴스마다 별도의 메모리 공간을 할당받아야 한다. 따라서 인스턴스가 많은 경우 메모리 사용이 늘어날 수 있다.


2. 유지 보수 및 가독성: `static`을 사용한 상수는 클래스 레벨에서 바로 접근할 수 있기 때문에 코드의 가독성이 높아질 수 있다.  
반면에 `final`만 사용한 상수는 인스턴스마다 독립된 이름으로 접근해야 한다. 이는 코드의 유지 보수 및 가독성을 저하시킬 수 있다.


3. 상속과 관련된 문제: `final`로만 선언된 인스턴스 변수는 하위 클래스에서 오버라이드할 수 없다.  
이로 인해 확장 가능성이 제한될 수 있다. `static final` 변수는 하위 클래스에서도 동일한 값을 공유하게 된다.

---

**[참고]**
- (https://velog.io/@tjddus0302/Java-%EC%83%81%EC%88%98%EB%8A%94-%EC%99%9C-static-final%EB%A1%9C-%EC%84%A0%EC%96%B8%ED%95%A0%EA%B9%8C)
- (https://djkeh.github.io/articles/Why-should-final-member-variables-be-conventionally-static-in-Java-kor/)

 

