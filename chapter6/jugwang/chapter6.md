## 예외 처리

- 예외 처리는 세 가지 활동을 합친 것
    1. 오류 보고
    2. 복구
    3. 자원 해제

- 오류 보고
    - 표준 라이브러리의 예외로 충분하지 못한 경우가 자주 있으므로, 예외를 더 구체적으로 처리하기 위해 Exception이나 Exception의 하위 타입을 상속한 새 예외 타입을 정의할 수 있다.
    
    ```kotlin
    class Exception1( 
    	val value: Int
    ): Exception("wrong value: $value")
    
    open class Exception2( 
    	description: String
    ): Exception(description)
    
    class Exception3( 
    	description: String
    ): Exception2(description)
    
    fun main() { 
    	capture {
    		throw Exception1(13)
    	} eq "Exception1: wrong value: 13" 
    	capture {
    		throw Exception3("error") 
    	} eq "Exception3: error"
    }
    ```
    
    - 새 예외 타입을 정의하려면 Exception을 상속하라(Exception은 Throwable을 확장한다).

- 복구
    - 예외 처리의 큰 목표는 복구
        
        (복구 : 문제를 해결하고 프로그램을 안정적인 상태로 되돌린 후 실행을 계속한다는 뜻)
        
    - 예외와 일치하는 예외 핸들러가 있으면 예외를 잡는다.
    - 예외를 잡아낼 핸들러를 찾으면 핸들러 검색이 끝나고 핸들러가 실행된다.
    - 프로그램이 일치하는 핸들러를 찾지 못하면 콘솔에 스택 트레이스를 출력하면서 종료된다.
    - 예외 핸들러에서는 catch 키워드 다음에 처리하려는 예외의 목록을 나열한다.
    그 후 복귀 과정을 구현하는 코드 블록이 온다.
    
    ```kotlin
    fun toss(which: Int) = when (which) { 
    	1 -> throw Exception1(1)
    	2 -> throw Exception2("Exception 2") 
    	3 -> throw Exception3("Exception 3") 
    	else -> "OK"
    }
    
    fun test(which: Int): Any? = 
    	try {
    		toss(which)
    	} catch (e: Exception1) { 
    		e.value
    	} catch (e: Exception3) {
    		e.message
    	} catch (e: Exception2) { 
    		e.message
    	}
    	
    fun main() {
    	test(0) eq "OK"
    	test(1) eq 1
    	test(2) eq "Exception2" 
    	test(3) eq "Exception3"
    }
    ```
    
- 너무 많은 예외 타입을 만들지 않기
- 간단한 규칙으로, 처리 방식이 달라야 한다면 다른 예외 타입을 사용해 이를 구분
- 처리 방법이 같은 경우 동일한 예외 타입을 쓰면서 생성자 인자를 다르게 주는 방식으로 구체적인 정보를 전달

- 자원 해제
    - 실패를 피할 수 없을 때 자원을 자동으로 해제하게 만들면 프로그램의 다른 부분이 계속 안전하게 실행되도록 도움을 줄 수 있다.
    - finally는 예외를 처리하는 과정에서 자원을 해제할 수 있게 보장한다.
    - try 블록이 정상적으로 끝났거나 예외가 발생했는지와 관계없이 항상 finally 절이 실행된다.
    - 중간에 catch 절이 있어도 finally 가 제대로 작동한다.

- 코틀린에서 예외를 사용하는 경우에 대한 가이드 라인
    - 논리 오류
        - 논리 오류는 코드에 있는 버그
        - 이와 관련한 예외를 전혀 잡지 않거나 애플리케이션의 최상위 수준에서 예외를 잡고 버그를 보고
    - 데이터 오류
        - 프로그래머가 제어할 수 없는 잘못된 데이터에 의해 생긴 오류
        - 코틀린 라이브러리는 예외를 던지는 대신 null을 돌려주는 식으로 나쁜 결과를 처리하도록 설계됨
        - 실패가 예상되는 연산마다 보통 예외를 던지는 버전 대신 쓸 수 있는 ‘OrNull’ 버전이 존재
    - 검사 명령
        - 검사 명령은 논리 오류를 검사
        - 버그를 찾으면 예외를 발생시키지만, 함수 호출처럼 보이기 때문에 코드가 명시적으로 예외를 던질 필요가 없어짐
    - 입출력(I/O) 오류
        - 우리가 제어할 수 없는 외부 조건이자 무시해서는 안되는 오류
        - I/O 오류로부터 복구할 수 있는 경우가 종종 있다 → 입출력 연산을 다시 시행해서 복구를 시도할 수 있다.
        - 코틀린에서 I/O 는 예외를 던지므로 애플리케이션에는 이런 예외를 처리하고 복구를 시도하는 코드가 포함되어야 함

## 검사 명령

> 검사 명령은 만족시켜야 하는 제약 조건을 적은 단언문
보통 함수 인자와 결과를 검증할 때 검사 명령을 사용함
> 
- 검사 명령은 뻔하지 않은 요구 사항을 표현함으로써 프로그래밍 오류를 발견한다.
- 검사 명령은 일반적으로 실패 시 오류를 던진다.
- 감시 명령을 사용하면 코드를 작성하거나 요구 조건을 생각하기도 쉬워지고, 작성된 코드를 이해하기도 더 수월해진다.

- require()
    - 계약에 의한 설계의 사전 조건은 초기화 관련 제약 사항을 보장한다.
    - require()는 보통 함수 인자를 검증하기 위해 사용되며, 함수 본문 맨 앞에 위치하는 경우가 많다.
    - 인자 검증이나 사전 조건 검증은 컴파일 시점에 할 수 없다.
    
    ```kotlin
    data class Month(val monthNumber: Int) {
    	init {
    		require(monthNumber in 1..12) {
    			"Month out of range: $monthNumber"
    		}
    	}
    }
    
    fun main( ) {
    	Month(1) eq "Month(monthNumber=1)"
    	capture { Month(13) } eq
    		"IllegalArgument Exception: " +
    		"Month out of range: 13"
    	}
    ```
    
    - require() 는 조건을 만족하지 못하면 IllegalArguement Exception을 반환
    - 파라미터를 두 개 받는 require() , 디폴트 메시지를 내놓는 파라미터 1개 뿐인 require() 도 있다.
- requireNotNull()
    - 첫 번째 인자가 null인지 검사해 널이 아니면 그 값을 돌려준다.
    - 하지만 널이면 IllegalArgumentException을 발생시킴
    - 성공한 경우 requireNotNull ()의 인자는 자동으로 널이 아닌 타입으로 스마트 캐스트된다.
    → 따라서 보통은 requireNotNull()의 반환값을 사용해야 할 필요가 없다.
    
    ```kotlin
    fun not Null(n: Int?): Int {
    	requireNotNull(n) { // [1]
    		"notNull() argument cannot be null"
    	}
    	return n * 9  // [2]
    }
    
    fun main() {
    	val n: Int? = null
    	capture {
    		notNull(n)
    	} eq "IllegalArgument Exception: " +
    		"notNull() argument cannot be null"
    	capture {
    		requireNotNull(n)  // [3]
    	} eq "IllegalArgumentException: " +
    		"Required value was null"
    	not Null(11) eq 99
    }
    ```
    
    - [2] requireNotNull() 호출이 n을 널이 될 수 없는 값으로 스마트 캐스트해주므로 n에 대해 더 이상 널 검사를 할 필요가 없다.

- check()
    - 계약에 의한 설계의 사후 조건은 함수의 결과를 검사한다.
    - 결과를 확신할 수 없는 복잡하고 긴 함수에서 사후 조건이 유용하다.
    - 함수의 결과에 대한 제약 사항을 묘사해야 하는 경우라면, 이를 사후 조건으로 표현하는 게 현명하다.
    - check()는 require()와 동일하지만 IllegalStateException을 던진다는 차이가 있다.
    - 일반적으로 check()를 함수의 맨 끝에서 함수 결과(또는 함수가 반환하는 객체의 필드)가 올바른지 검증하기 위해 사용한다.
    - check() 는 무엇인가가 잘못된 상태에 들어가지 않았는지를 검사한다.
    
    ```kotlin
    fun createResultFile(create: Boolean) {
    	if (create)
    		resultFile.writeText("Results\n# ok")
    	// ... 다른 실행 경로들
    	check(resultFile.exists()) {
    		"${resultFile.name} doesn't exist!"
    	}
    }
    
    fun main( ) {
    	resultFile.erase()
    	capture {
    		createResultFile(false)
    	} eq "IllegalStateException: " +
    		"Results.txt doesn't exist!"
    	createResultFile(true)
    }
    ```
    
    - 사전 조건이 인자가 제대로 들어왔음을 검증했다고 가정하면, 사후 조건이 실패한다는 건 거의 항상 프로그래밍 실수가 있다는 의미

- assert()
    - check() 문을 주석 처리했다가 해제하는 수고를 덜기 위해 assert()를 사용한다.
    - assert()의 경우 프로그래머가 assert() 검사를 활성화하거나 비활성화할 수 있다.
    - assert()는 자바에서 온 명령이며, 기본적으로 비활성화되어 있지만, 명령줄 플래그로 명시적으로 assert()를 활성화할 수 있다. 코틀린에서는 -ea라는 플래그를 사용한다.
    - 그러나 특별한 설정이 없어도 항상 사용할 수 있는 require()와 check()를 사용하는 것이 좋다.

## Nothing 타입

> Nothing은 함수가 결코 반환되지 않는다는 사실을 표현하는 반환 타입
> 
- 항상 예외를 던지는 함수의 반환 타입이 Nothing이다.

```kotlin
fun infinite(): Nothing {
	while (true) {}
} // 이 함수는 결코 반환되지 않으므로 반환 타입이 Nothing
```

- Nothing은 아무 인스턴스도 없는 코틀린 내장 타입
- 실용적인 예로는 내장 함수인 TODO()
    - 이 함수는 반환 타입이 Nothing 이고, 항상 NotImplementedError를 던진다.
- Nothing은 모든 타입과 호환 가능하다.
→ 즉, 모든 다른 타입의 하위 타입으로 취급된다.

## 자원 해제

> try-finally 블록을 써서 자원을 정리하는 과정은 지루하고 실수를 저지르기 쉽다.
코틀린 라이브러리 함수는 우리를 대신해 자원을 정리해준다.
> 
- use()
    - 닫을 수 있는 자원을 제대로 해제해주고, 자원 해제 코드를 직접 작성하는 수고로부터 해방시켜준다.
    - 자바의 AutiCloseable 인터페이스를 구현하는 모든 객체에 작용할 수 있다.
    - use()는 인자로 받은 코드 블록을 실행하고, 그 블록을 어떻게 빠져나왔는지(return을 포함하는 정상적인 경로나 예외)와 관계없이 객체의 close()를 호출한다.
    - use()는 모든 예외를 다시 던져주기 때문에 프로그램에서는 여전히 예외를 처리해야 한다.
    
    ```kotlin
    fun main() {
    DataFile("Results.txt")
    	.bufferedReader()
    	.use{ it.readlines().first() } eq
    	"Results"
    }
    ```
    
    - AutoCloseable 인터페이스를 구현하면 use()에 사용할 수 있는 여러분 자신만의 클래스를 만들 수 있다.
    - AutoCloseable 인터페이스 안에는 close() 함수만 들어 있다.
    
    ```kotlin
    class Usable() : AutoCloseable {
    	fun func() = trace("func()")
    	override fun close() = trace("close()")
    }
    
    fun main() {
    	Usable().use{ it.func() }
    	trace eq "func() close()"
    }
    ```
    

## 로깅

> 로깅은 실행 중인 프로그램에서 정보를 얻은 행위
> 
- 코틀린 로깅 : 오픈 소스 로깅 패키지
- 로깅을 하려면 사용할 로거를 만들어야 한다.
- 거의 대부분의 경우, 로거를 파일 영역에 정의해서 같은 파일에 있는 모든 컴포넌트가 로거를 사용할 수 있게 한다.
- 시작 설정에서는 실제로 보고에 사용할 로깅 수준을 결정
- 실행 중에도 로깅 수준을 변경할 수 있음
- 다양한 로깅을 통합하기 위해 공통 로깅 인터페이스를 개발
    - 여러 가지 로깅 라이브러리를 지원하는 퍼사드로 개발
- 코틀린-로깅 라이브러리는 SLF4J 위에 만든 퍼사드
    - SLF4J 자체는 여러 가지 로깅 프레임워크 위에 만들어진 추상화

## 단위 테스트

> 단위 테스트는 함수의 여러 측면에 대해 올바른지 검증하는 테스트를 작성하는 방법
> 
- 자바에서는 Junit이 가장 유명하다.
- 코틀린 표준 라이브러리에는 여러 테스트 라이브러리에 대한 퍼사드를 제공하는 kotlin.test가 들어 있다.
→ 이로 인해 어느 한 라이브러리에 구속될 필요가 없다.
- kotlin.test는 기본 단언 함수를 감싸는 래퍼도 제공
- 단언문 함수
    - 실제 값과 예상값을 비교하는 assertEquals()
    - 첫 번째 파라미터로 들어온 Boolean 식이 참인지 검증하는 assertTrue()
- assert로 시작하는 여러 가지 함수
    - assertEquals() , assertNotEquals()
    - assertTrue(), assertFalse()
    - assertNull() , assertNotNull()
    - assertFails() . assertFailsWith()
- expect()
    - 코드 블록을 실행하고 그 결과를 예상값과 비교
    
    ```kotlin
    fun <T> expect(
    	expected: T,
    	message: String?,
    	block: () -> T
    ) {
    	assertEquals(expected, block(), message)
    }
    ```
    

- 테스트 프레임워크
    - 전형적인 테스트 프레임워크는 단언문 함수와 테스트를 실행하고 결과를 표시하기 위한 메커니즘으로 이뤄진다.
    - 코틀린은 정의와 식에 애너테이션을 허용
    (애너테이션: @뒤에 애너테이션 이름을 붙인 것으로, 애너테이션이 붙은 요소를 특별히 처리할 수 도 있다는 뜻)
    - @Test 애너테이션은 일반 함수를 테스트 함수로 바꿔준다.
    - @Test 함수를 실행하는 방법
        - 독립적으로 실행
        - 클래스의 일부분으로 실행
        - 애플리케이션에 정의된 모든 테스트와 함께 실행
    - 일반적으로 코틀린은 함수 이름에 글자와 숫자만 허용
    - 하지만 함수 이름을 역작은따옴표로 감싸면 이름에 아무 문자(공백 포함)나 사용할 수 있다.
        
        ```kotlin
        @Test
        fun ' pause and resume ' ( ) {
        ```
        
    - 단위 테스트의 본질적인 목표는 복잡한 소프트웨어의 점진적인 개발 과정을 단순화하는 것

- 모킹과 통합 테스트
    - 다른 요소에 의존하는 시스템으로 인해 격리된 테스트를 만들기 어려워짐
    → 실제 구성 요소를 의존 관계에 추가하는 대신 모킹에 의존
    - 목(mock) 은 테스트를 실행하는 동안 실물을 대신하는 가짜이며, 저장된 데이터의 무결성을 유지하기 위해 데이터베이스를 모킹하는 경우가 많다.
    - 기능을 각 부분을 독립적으로 테스트하는 것이 중요하며, 바로 단위 텐스트가 그런 일을 한다.
    - 시스템의 여러 부분을 하나로 합쳤을 때 잘 작동하는지 검증하는 것 → 통합테스트
    - 단위 테스트는 ‘내부로 향하는’ 테스트
    - 통합 테스트는 ‘밖으로 향하는’ 테스트
