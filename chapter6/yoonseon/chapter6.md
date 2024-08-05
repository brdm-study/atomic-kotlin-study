# ✅ 73. 예외처리

---

## 오류보고

- 표준 라이브러리의 예외로 충분하지 못한 경우가 있어서 더 구체적으로 처리하기 위해 Exception이나 Exception 하위 타입을 상속한 새 예외 타입을 정의할 수 있음
- `throw` 는 `Throwable` 의 하위 타입을 요구함

```kotlin
// Exception 상속받은 예외 타입
class DatabaseException(message: String) : Exception(message)

// Exception 상속받은 예외 타입
class NetworkException(message: String) : Exception(message)

// Exception 하위 타입을 상속받은 새 예외 타입(NetworkException을 상속)
class ApiRequestException(message: String) : NetworkException(message)

fun main() {
    try {
        // 예외를 던짐
        throw ApiRequestException("This is an API request exception")
    } catch (e: ApiRequestException) {
        // ApiRequestException을 처리
        println("Caught ApiRequestException: ${e.message}")
    } catch (e: NetworkException) {
        // NetworkException을 처리
        println("Caught NetworkException: ${e.message}")
    } catch (e: DatabaseException) {
        // DatabaseException을 처리
        println("Caught DatabaseException: ${e.message}")
    } catch (e: Exception) {
        // 다른 모든 Exception을 처리
        println("Caught Exception: ${e.message}")
    }
}
```

## 복구

- `Exception` 을 `throw` 하면 함수 호출 체인을 타고 위쪽으로 이동함
    - 예를 들어 a() 예외 발생 → a()를 호출한 b()로 예외 전달 → 함수 실행 주체인 main()으로 전달
- 예외를 전달하는 과정에서 예외와 일치하는 `Exception Handler`(`try-catch` 구문)가 있으면 예외를 잡음
    - 예외를 잡아낼 핸들러를 찾으면 핸들러 검색이 끝나고 핸들러가 실행
    - 찾지 못하면 콘솔에 스택 트레이스를 출력하면서 종료됨
- `catch` 절이 여러개인 경우, 정의 순서에 따라 부모 클래스 예외가 자식 클래스 예외를 처리할 수 있음
- 같은 예외가 다른 상황에서 발생하여 문제가 생길 수 있는데, 이 문제를 해결하려면 `Exception`을 상속받은 여러 예외 타입을 정의하고, 예외 핸들러에서 각 예외를 구체적으로 처리하면 됨

> 너무 많은 예외 타입을 만들지는 말라. 간단한 규칙으로. 처리 방식이 달리 야한다면 다른 예외 타입을 사용해 이를 구분하라. 그리고 처리 방법이 같은 경우 동일한 예외 타입을 쓰면서 생성자 인자를 다르게 주는 방식으로 구체적인 정보를 전달하라.
>

## 자원 해제

- `finally` 는 예외를 처리하는 과정에서 자원을 해제할 수 있게 보장함
    - `try` 블록이 정상적으로 끝났거나 예외가 발생했는지와 관계없이 항상 `finally` 절이 실행됨
    - 중간에 `catch` 절이 있어도 `finally` 가 실행됨
    - `try` 블록 안에서 `return`을 사용하더라도 `finally` 절은 여전히 실행됨

## 가이드 라인

코틀린 예외의 주목적은 프로그램 버그를 발견하는 것이지 예외 복구가 아님

1. **논리 오류**
    - **정의:** 코드에 있는 버그.
    - **처리:** 예외를 잡지 않거나 최상위 수준에서 예외를 잡고 버그를 보고. 영향을 받은 연산을 다시 시작할 수 있음.
2. **데이터 오류**
    - **정의:** 프로그래머가 제어할 수 없는 잘못된 데이터에 의해 발생.
    - **처리:** 애플리케이션이 프로그램 로직을 탓하지 않고 처리해야 함. 예를 들어, `String.toInt()`는 예외를 발생시키지만, `String.toIntOrNull()`은 실패 시 null을 반환.
    - **특징:** 코틀린 라이브러리는 예외 대신 null을 반환하는 방식을 선호. 실패가 예상되는 연산마다 ‘OrNull’ 버전이 존재.
3. **검사 명령**
    - **정의:** 논리 오류를 검사.
    - **처리:** 버그를 찾으면 예외를 발생시키지만, 함수 호출처럼 보임. 코드가 명시적으로 예외를 던질 필요가 없어짐.
4. **입출력(I/O) 오류**
    - **정의:** 제어할 수 없는 외부 조건에 의해 발생.
    - **처리:** ‘OrNull’ 방식을 사용하면 코드가 더 쉽게 이해됨. I/O 오류는 종종 복구 가능. 코틀린에서는 예외를 던지므로, 애플리케이션에 예외를 처리하고 복구를 시도하는 코드가 필요.



# ✅ 74. 검사 명령

---

## require()

- 매개변수의 값이 참인지 체크, 거짓이라면 `IllegalArgumentException` 발생
- 인자는 조건식과, 반환할 메시지(`()-> Any`)이며, 조건식만 전달할 경우 디폴트 메시지가 사용됨

**사용 전**

```kotlin
fun calculatePercentage(value: Int): Int {
    // 조건을 수동으로 확인
    if (value !in 0..100) {
        throw IllegalArgumentException("Value must be between 0 and 100")
    }
    return value
}
```

**사용 후**

```kotlin
fun calculatePercentage(value: Int): Int {
    // require()를 사용하여 조건을 확인
    require(value in 0..100) { "Value must be between 0 and 100" }
    return value
}

```

## requireNotNull()

- 매개변수의 값이 null이 아니라면 value를 반환, null이면 `IllegalArgumentException` 발생
- 인자는 null인지 확인할 값과, 반환할 메시지(`()-> Any`)이며, 조건식만 전달할 경우 디폴트 메시지가 사용됨

**사용 전**

```kotlin
fun printName(name: String?) {
    // null 여부를 수동으로 확인
    if (name == null) {
        throw IllegalArgumentException("Name cannot be null")
    }
    println(name)
}
```

**사용 후**

```kotlin
fun printName(name: String?) {
    // requireNotNull()을 사용하여 null 여부를 확인
    requireNotNull(name) { "Name cannot be null" }
    println(name)
}
```

## check()

- 매개변수의 값이 참인지 체크, 거짓이라면 `IllegalStateException` 발생
- 인자는 조건식과, 반환할 메시지(`()-> Any`)이며, 조건식만 전달할 경우 디폴트 메시지가 사용됨

**사용 전**

```kotlin
fun validateState(state: Int) {
    // 조건을 수동으로 확인
    if (state < 0) {
        throw IllegalStateException("State must not be negative")
    }
}
```

**사용 후**

```kotlin
fun validateState(state: Int) {
    // check()를 사용하여 조건을 확인
    check(state >= 0) { "State must not be negative" }
}
```

## checkNotNull()

- 매개변수의 값이 null이 아니라면 value를 반환, null이면 `IllegalStateException` 발생
- 인자는 null인지 확인할 값과, 반환할 메시지(`()-> Any`)이며, 조건식만 전달할 경우 디폴트 메시지가 사용됨

**사용 전**

```kotlin
fun printNonNullName(name: String?) {
    // null 여부를 수동으로 확인
    if (name == null) {
        throw IllegalStateException("Name cannot be null")
    }
    println(name)
}
```

**사용 후**

```kotlin
fun printNonNullName(name: String?) {
    // checkNotNull()을 사용하여 null 여부를 확인
    val nonNullName = checkNotNull(name) { "Name cannot be null" }
    println(nonNullName)
}
```

> check( )는 require( )와 동일하지만 IllegalStateException을 던진다는 차이가 있다.
일반적으로 check( )를 함수의 맨 끝에서 함수 결과(또는 함수가 반환하는 객체의 필드)가 올바른지 검증하 기 위해 사용하는데, 그래서 check( )는 무엇인가가 잘못된 상태에 들어가지 않았는지를 검사한다.
>

## assert()

- `assert()`는 주로 디버깅 목적으로 사용되어 특정 조건이 참인지 확인, 조건이 거짓일 경우 `AssertionError` 발생
- `assert` 문은 JVM의 `-ea` (enable assertions) 옵션이 활성화된 경우에만 실행됨

**사용 전**

```kotlin
fun calculateFactorial(n: Int): Int {
    // 수동으로 조건 확인
    if (n < 0) {
        throw AssertionError("n should be non-negative")
    }
    // 계산 로직 생략
    return 1
}
```

**사용 후**

```kotlin
fun calculateFactorial(n: Int): Int {
    // assert()를 사용하여 조건을 확인
    assert(n >= 0) { "n should be non-negative" }
    // 계산 로직 생략
    return 1
}
```

> 특별한 설정이 없어도 항상 사용할 수 있는 require( )와 check( )를 사용하는 것이 좋다.
>

# ✅ 75. Nothing 타입

---

- Noting은 함수가 결코 정상적으로 끝나지 않는다는 사실을 표현하는 반환 타입임
- 무조건 예외를 반환하는 함수 / 무한 루프 함수에 적용함.

```kotlin
fun infinite(): Nothing { 
    while (true) {}
}
```

- Nothing 타입의 대표 적인 예로 `TODO()` 함수가 있음
    - **TODO()**
        - 코틀린 내장 함수로 반환타입이 Nothing이고, 항상 `NotImplementedError`를 던짐
        - 주로 구현 중인 코드나 아직 작성되지 않은 코드 블록을 명확히 표시하기 위해 사용됨

        ```kotlin
        fun calculateSum(a: Int, b: Int): Int {
            // TODO: 나중에 구현할 예정
            return TODO()
        }
        ```


- 만약 리스트가 오로지 null 값만을 포함하고 있다면, 컴파일러는 이 리스트의 타입을 `List<Nothing?>`으로 추론할 수 있음

    ```kotlin
    fun main() {
        // 리스트를 null 값만으로 초기화
        val listNone: List<Nothing?> = listOf(null, null, null)
    
        // 출력
        println(listNone) // [null, null, null]
    }
    ```


# ✅ 76. 자원 해제

---

- 코틀린의 `use()` 함수는 자원 해제를 간편하게 수행할 수 있도록 도와줌
    - 객체 사용 후 자동으로 자원을 닫아주어 자원 누수를 방지하고 코드의 안정성을 높이는 데 유용함
- `use()`는 `AutoCloseable` 인터페이스를 구현하는 모든 객체에 작용할 수 있음
    - `AutoCloseable` 안에는 `close()` 함수만 들어있는데 use()가 블록이 끝날 때 이를 자동으로 호출함

```kotlin
fun main() {
    // use()를 사용하여 자원 자동 해제
    BufferedReader(InputStreamReader(System.`in`)).use { br ->
        var str = br.readLine()
        println(str)
    } // 블록 종료 후 BufferedReader가 AutoClosed 됨
}
```

# ✅ 77. 로깅

---

- 코틀린 로깅(Kortlin-logging)이라는 오픈 로깅 패키지를 사용할 수 있음
    - 오픈 소스 : https://github.com/oshai/kotlin-logging
    - 사용법 : [https://velog.io/@glencode/kotlin-logging을-사용하여-효과적으로-로깅하기](https://velog.io/@glencode/kotlin-logging%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EC%97%AC-%ED%9A%A8%EA%B3%BC%EC%A0%81%EC%9C%BC%EB%A1%9C-%EB%A1%9C%EA%B9%85%ED%95%98%EA%B8%B0)
- 코틀린-로깅 라이브러리는 SLF4J(Simple Logging Facade for Java : 자바 단순 로깅 파사드)위에 만든 파사드이다.

- 로깅을 하려면 사용할 logger를 만들어야 하는데, 대부분의 경우 로거를 파일 영역에 정의해서 같은 파일에 있는 모든 컴포넌트가 로거를 사용할 수 있게 함

```kotlin
private val log = KLoggin().logger

fun main() {
    val msg = "Hello!"
    log.trace(msg)
    log.debug(msg)
    log.info(msg)
    log.warn(msg)
    log.error(msg)
}
```

- trace(), debug(), info()는 동작에 대한 정보를 기록하고, warn(), error()는 문제가 발생한 사실을 표시함

# ✅ 78. 단위 테스트

---

- 단위테스트는 함수의 여러 측면에 대해 올바른지 검증하는 테스트를 작성하는 방법임
- 자바에서는 수많은 단위 테스트 프레임워크 중 JUnit이 가장 유명함
- 코틀린 표준 라이브러리에는 여러 테스트 라이브러리에 대한 퍼사드를 제공하는 `kotlin.test`가 들어있음
- `kotlin.test` 를 사용하려면 build.gradle.kt에 dependencies 부분에 다음을 추가하면 코틀린 플러그인이 자동으로 코틀린 테스트 관련 의존관계를 처리해줌

    ```kotlin
    implementation(kotlin("test"))
    ```


- kotlin.test는 assert로 시작하는 여러 가지 함수를 제공함
    - assertTrue(), assertFalse()
    - assertEquals(), assertNotEquals()
    - assertSame(), assertNotSame()
    - assertIs(), assertIsNot()
    - assertIsOfType(), assertIsNotOfType()
    - assertNull(), assertNotNull()
    - assertContains()
    - assertRangeContains()
    - assertContentEquals()

## 테스트 프레임워크

- `kotlin.test` 는 가장 일반적으로 쓰이는 함수에 대한 퍼사드를 제공한다.
- 단언문은 기반 데스트 프레임워크의 적절한 함수에 처리를 위임한다.
- 예를 들어 `kotlin.test` 의 `assertEquals()` 함수 는 `org.junit.jupiter.api.Assertions` 클래스의 `assertEquals()` 를 사용한다.
- `@Test` 어노테이션을 써서 테스트할 수 있고 다음 방법으로 실행할 수 있다.
    - 독립적으로 실행
    - 클래스의 일부분으로 실행
    - 애플리케이션에 정의된 모든 테스트와 함께 실행
- 일반적으로 코틀린은 함수 이름에 글자와 숫자만 허용하지만 함수 이름을 역작은따옴표 `(`)` 로 감싸면 이름에 아무 문자(공백 포함)나 사용할 수 있다. 이 기능을 통해 pause and resume' 처럼 데스트를 기술하는 문장을 함수 이름으로 쓸 수 있다.

## 모킹과 통합테스트

- **목(Mock):** 실제 구성 요소를 대신하는 가짜 객체
    - **용도:** 시스템의 다른 부분에 대한 의존성을 제거하고 격리된 테스트를 수행하기 위해 사용
    - 실제와 똑같은 인터페이스를 구현한 목을 만들 수도 있고. Mock판 같은 모킹 라이브러리를 시용해 목을 만들 수도 있음
- **단위 테스트(Unit Test):** 시스템의 각 기능이나 모듈을 독립적으로 테스트
    - **특징:** 내부 구현에 중점을 두고 테스트하며, 기능이 예상대로 작동하는지 확인함
- **통합 테스트(Integration Test):** 시스템의 여러 부분이 함께 작동하는지 테스트
    - **특징:** 외부 시스템과의 통합을 포함하여 전체 시스템의 동작을 테스트함