## 확장 함수

---

코틀린의 확장 함수는 클래스나 인터페이스를 수정하지 않고 새로운 함수를 추가할 수 있게하는 기능이다. 이는 특히 기존 라이브러리의 클래스를 변경할 수 없는 경우나 코드의 가독성을 높이기 위해 사용하면 좋다.

```kotlin
fun String.lastchar(): Char = this[this.length - 1]
```

확장함수는 일반 함수와 비슷하게 정의하면 된다. 함수 이름 앞에 해당 함수가 확장될 클래스의 타입을 명시하면 된다.

위 코드를 보자. `String` 클래스에 `lastChar`라는 확장 함수를 임의로 추가했다. 이제 모든 String 객체에 대해 `lastChar` 메소드를 사용할 수 있다.

```kotlin
val str = "Kotlin"
println(str.lastChar()) // output: n
```

확장 함수를 사용하면 어떠한 이점이 있을까?

1. 클래스 코드 수정 불필요: 기존 클래스의 코드를 수정할 필요 없이 확장이 가능하다.
2. 컴파일 시 정적으로 결정: 확장 함수는 실제 클래스의 메소드로 추가되는 것이 아니라 컴파일 시 확장 함수가 호출된 곳에 실제 함수 코드가 삽입된다.
3. 가시성: 확장 함수는 정의된 파일 내에서만 가시성을 가진다. 확장 함수를 다른 곳에서 사용하기 위해서는 해당 파일에 임포트하여 사용할 수 있다.

확장 함수를 사용하면 코드의 가독성이 향상된다. 예를 들어 여러 클래스에서 공통으로 사용되는 기능을 확장 함수로 정의하면 코드가 더 직관적이고 간결해진다.

```kotlin
fun List<Int>.sumOfSquares(): Int = this.sumBy { it * it }

val numbers = listOf(1, 2, 3)
println(numbers.sumOfSquares()) // output: 14
```

코틀린의 확장함수는 DSL을 구축할 때 매우 유용하다. `DSL(Domain Specific Language)` 은 특정 도메인에 특화된 언어로, 코틀린의 유연한 문법을 활용해 직관적이고 간결한 코드를 작성할 수 있게 된다.

```kotlin
fun StringBuilder.buildHtml(block: StringBuilder.() -> Unit): String {
    val sb = StringBuilder()
    sb.block()
    return sb.toString()
}

val html = StringBuilder().buildHtml {
    append("<html>")
    append("<body>")
    append("<h1>Hello, World!</h1>")
    append("</body>")
    append("</html>")
}
println(html)
```

또 확장 함수는 null 안정성과 타입 변환에 유용하게 사용될 수 있다.

```kotlin
fun <T> List<T>.secondOrNull(): T? = if (this.size > 1) this[1] else null

val list = listOf(1, 2, 3)
println(list.secondOrNull()) // Output: 2
```

## 이름 붙은 인자와 디폴트 인자

---

코틀린의 이름 붙은 인자(named arguments)와 디폴트 인자(default arguments)는 함수 호출 시 인자의 순서와 생략을 유연하게 관리할 수 있는 기능이다. 이를 통해 코드를 더 명확하고 간결하게 작성할 수 있다.

### 이름 붙은 인자

이름 붙은 인자는 함수 호출 시 인자의 이름을 명시하여 전달하는 방식이다. 이를 통해 인자의 순서에 의존하지 않고 함수 호출을 할 수 있다.

```kotlin
fun createUser(name: String, age: Int, email: String) {
    println("Name: $name, Age: $age, Email: $email")
}

// 이름 붙은 인자를 사용한 함수 호출
createUser(name = "Alice", age = 30, email = "alice@example.com")

// 인자의 순서와 상관없이 호출 가능
createUser(email = "bob@example.com", name = "Bob", age = 25)
```

### 디폴트 인자

디폴트 인자는 함수 정의 시 인자에 기본값을 지정하는 기능이다. 이를 통해 함수 호출 시 일부 인자를 생략할 수 있다.

```kotlin
fun greetUser(name: String, greeting: String = "Hello") {
    println("$greeting, $name!")
}

// 디폴트 인자를 사용한 함수 호출
greetUser("Alice") // Output: Hello, Alice!
greetUser("Bob", "Hi") // Output: Hi, Bob!
```

이 두 기능을 함께 사용해보자. 함수 호출이 더욱 유연해지고 가독성이 높아짐을 느낄 수 있다.

```kotlin
fun printUserInfo(name: String, age: Int = 25, country: String = "USA") {
    println("Name: $name, Age: $age, Country: $country")
}

// 디폴트 인자와 이름 붙은 인자를 함께 사용
printUserInfo(name = "Alice") // Output: Name: Alice, Age: 25, Country: USA
printUserInfo(name = "Bob", country = "Canada") // Output: Name: Bob, Age: 25, Country: Canada
```

해당 기능들을 활용하면 어떠한 이점이 있을까?

1. 코드 가독성 향상: 이름 붙은 인자를 사용하면 함수 호출 시 어떤 값이 어떤 인자에 대응되는지 명확하게 표현할 수 있게 된다.
2. 유연한 함수 호출: 디폴트 인자를 사용하면 호출할 때 불필요한 인자를 생략할 수 있어 코드가 간결해진다. 특히, 많은 인자를 받는 함수에서 유용하다.
3. 오버로딩 감소: 디폴트 인자를 사용하면 동일 함수의 여러 오버로드를 줄일 수 있다.

```kotlin
// 오버로딩이 필요한 경우
fun log(message: String) = log(message, "INFO")
fun log(message: String, level: String) = println("[$level] $message")

// 디폴트 인자를 사용하는 경우
fun log(message: String, level: String = "INFO") = println("[$level] $message")

log("System started") // Output: [INFO] System started
log("System error", "ERROR") // Output: [ERROR] System error
```

실제 사용될 여지도 굉장히 많아 보인다.

REST API 요청을 처리하는 부분에서 디폴트 인자를 통해 기본값을 설정하므로 기존 자바코드에서는 찌꺼기 처럼 남아있던 코드를 삭제할 수 있다.

```kotlin
fun getUserInfo(userId: String, detailLevel: String = "basic"): UserInfo {
    // detailLevel에 따라 기본 정보 또는 상세 정보를 반환
    ...
}

// 기본값 사용
val userInfoBasic = getUserInfo("user123")

// 상세 정보 요청
val userInfoDetailed = getUserInfo("user123", detailLevel = "detailed")
```

또 설정 클래스를 초기화 할때 디폴트 인자를 사용하는 것도 좋은 방법같아 보인다.

```kotlin
data class Config(
    val host: String = "localhost",
    val port: Int = 8080,
    val useSsl: Boolean = false
)

val defaultConfig = Config()
val customConfig = Config(host = "example.com", useSsl = true)
```

## 함수 오버로딩

---

코틀린의 함수 오버로딩은 같은 이름의 함수를 여러개 정의할 수 있게 하는 기능이다. 각 함수는 서로 다른 파라미터 목록을 가져야 하며, 이를 통해 같은 동작을 하는 함수지만 다른 입력값을 처리할 수 있다. 이는 코드의 유연성과 재사용성을 높이는데 유용하게 사용된다.

```kotlin
fun display(value: Int) {
    println("Displaying an integer: $value")
}

fun display(value: String) {
    println("Displaying a string: $value")
}

fun display(value: Int, prefix: String) {
    println("$prefix: $value")
}

// 함수 호출
display(42) // Output: Displaying an integer: 42
display("Hello, Kotlin") // Output: Displaying a string: Hello, Kotlin
display(42, "Value") // Output: Value: 42
```

기존 Java 기반의 코드에서도 비슷하게 설계가 가능한 점이 있어 간단하게 짚고 넘어가겠다.

## When 표현식

---

코틀린의 when 표현식은 여러 조건을 처리할 때 사용하는 강력한 제어구조이다. when은 자바의 switch문과 유사한데, 다만 더욱 유연하고 기능이 풍부하다. 조건에 따라 값을 반환하거나 특정 블록을 실행할 수 있게 한다.

when은 다양한 방식으로 사용되지만 가장 기본적인 형태는 값을 검사하는 형태이다.

```kotlin
fun describe(obj: Any): String =
    when (obj) {
        1          -> "One"
        "Hello"    -> "Greeting"
        is Long    -> "Long number"
        !is String -> "Not a string"
        else       -> "Unknown"
    }

// 함수 호출
println(describe(1))        // Output: One
println(describe("Hello"))  // Output: Greeting
println(describe(1000L))    // Output: Long number
println(describe(2.5))      // Output: Not a string
println(describe("Other"))  // Output: Unknown
```

when 표현식은 단순한 값 비교 외에도 복잡한 조건을 처리할 수 있다.

```kotlin
fun numberDescription(x: Int): String =
    when (x) {
        in 1..10    -> "Between 1 and 10"
        !in 10..20  -> "Outside 10 to 20"
        in listOf(21, 22, 23) -> "In the list"
        else -> "None of the above"
    }

// 함수 호출
println(numberDescription(5))   // Output: Between 1 and 10
println(numberDescription(15))  // Output: Outside 10 to 20
println(numberDescription(22))  // Output: In the list
println(numberDescription(30))  // Output: None of the above
```

여러 조건을 쉼표로 구분하여 한 번에 처리할 수도 있다.

```kotlin
fun getColor(value: String): String =
    when (value) {
        "Red", "Green", "Blue" -> "Primary color"
        "Black", "White"       -> "Neutral color"
        else                   -> "Unknown color"
    }

// 함수 호출
println(getColor("Red"))    // Output: Primary color
println(getColor("Black"))  // Output: Neutral color
println(getColor("Yellow")) // Output: Unknown color
```

각 분기에 블록을 사용하여 블록 안에서 여러 줄의 코드를 실행하는 것도 가능하다.

```kotlin
fun evaluateExpression(expression: Any): String =
    when (expression) {
        is Int -> {
            val doubleValue = expression * 2
            "Double of $expression is $doubleValue"
        }
        is String -> {
            val length = expression.length
            "Length of \"$expression\" is $length"
        }
        else -> "Unknown type"
    }

// 함수 호출
println(evaluateExpression(10))     // Output: Double of 10 is 20
println(evaluateExpression("Kotlin")) // Output: Length of "Kotlin" is 6
```

when 표현식은 복잡한 조건문을 간결하게 작성할 수 있어 코드의 가독성을 높일 수 있게 된다. 다양한 타입과 조건을 처리할 수 있어 여러 상황에 유용한데, 특히 예외 처리와 결합되었을 때 더 우아하게 사용이 가능한 것 같다.

```kotlin
fun parseInput(input: Any): Int =
    when (input) {
        is String -> input.toIntOrNull() ?: throw IllegalArgumentException("Invalid number format")
        is Int -> input
        else -> throw IllegalArgumentException("Unsupported type")
    }

// 함수 호출
println(parseInput("123"))  // Output: 123
println(parseInput(456))    // Output: 456
try {
    println(parseInput("abc"))
} catch (e: IllegalArgumentException) {
    println(e.message)      // Output: Invalid number format
}
```

데이터 클래스의 상태를 처리하기 위해 사용되기도 한다고 한다.

```kotlin
sealed class Result
data class Success(val data: String) : Result()
data class Failure(val error: String) : Result()

fun handleResult(result: Result): String =
    when (result) {
        is Success -> "Success with data: ${result.data}"
        is Failure -> "Failure with error: ${result.error}"
    }

// 함수 호출
println(handleResult(Success("Data loaded"))) // Output: Success with data: Data loaded
println(handleResult(Failure("Network error"))) // Output: Failure with error: Network error
```

## Enum

---

코틀린의 enum 클래스는 열거형 타입을 정의하는데 사용된다. 열거형 타입이란 상수의 집합을 나타내며 상수들은 열거형 클래스의 인스턴스이다.

코틀린에서 Enum을 정의하는 방법은 다음과 같다.

```kotlin
enum class Direction {
    NORTH, SOUTH, EAST, WEST
}

// 사용 예시
val direction = Direction.NORTH
println(direction) // Output: NORTH
```

자바에서의 사용과 매우 비슷하다.

enum은 단순히 상수를 정의하는 것 외에도 프로퍼티와 메소드를 가질 수 있다.

```kotlin
enum class Color(val rgb: Int) {
    RED(0xFF0000),
    GREEN(0x00FF00),
    BLUE(0x0000FF);

    fun containsRed() = (rgb and 0xFF0000 != 0)
}

// 사용 예시
val color = Color.RED
println(color.containsRed()) // Output: true
println(Color.GREEN.containsRed()) // Output: false
```

열거형 클래스와 위에서 배운 When 표현식을 함께 사용해보자. 더 간결하고 가독성 높은 코드를 작성할 수 있다.

```kotlin
enum class Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

fun isWeekend(day: Day): Boolean {
    return when (day) {
        Day.SATURDAY, Day.SUNDAY -> true
        else -> false
    }
}

// 사용 예시
println(isWeekend(Day.MONDAY)) // Output: false
println(isWeekend(Day.SUNDAY)) // Output: true
```

서버의 상태 혹은 작업의 상태를 사용할 때 enum을 사용하면 안전하게 관리가 가능하다. 그 외에도 특정 설정 값들을 열거형으로 정의하여 관리하는 것이 코드의 가독성과 유지보수성을 높일 수 있다.

enum은 자바에서의 사용과 유사하여 간단하게 짚고 넘어간다.

## 데이터 클래스

---

코틀린의 데이터 클래스는 말 그대로 데이터를 저장하는데 사용하는 클래스를 간결하게 정의할 수 있는 기능이다. 데이터 클래스는 데이터 저장 및 조작을 위한 표준 메소드를 자동으로 생성해준다. 이러한 메소드는 `toString()` `equals()` `hashCode()` `copy()` `componentN()` 등의 함수가 포함된다.

데이터 클래스는 Data 키워드를 사용하여 정의한다. 주 생성자에는 최소한 하나 이상의 프로퍼티가 포함되어야 한다.

```kotlin
data class User(val name: String, val age: Int)

// 데이터 클래스 사용 예시
val user1 = User("Alice", 30)
val user2 = User("Bob", 25)

println(user1) // Output: User(name=Alice, age=30)
println(user2) // Output: User(name=Bob, age=25)
```

데이터 클래스를 사용하면 자동으로 데이터 관리를 위한 메소드가 생성된다.

데이터 저장을 위한 클래스를 간결하게 정의할 수 있고, 데이터 클래스는 프로퍼티를 val로 정의하여 불변한 객체를 만들 때 주로 사용된다.

익숙하지 않은 데이터 클래스의 메서드들도 살펴보자.

데이터 클래스에는 객체를 복사하면서 일부 프로퍼티만 변경할 수 있는 `copy()` 메서드가 자동으로 생성된다.

```kotlin
val user3 = user1.copy(age = 31)
println(user3) // Output: User(name=Alice, age=31)
```

또 데이터 클래스는 주 생성자에 정의된 프로퍼티에 대해 자동으로 `componentN()` 메서드를 생성하여, 구조 분해 선언을 지원한다.

```kotlin
val (name, age) = user1
println("Name: $name, Age: $age") // Output: Name: Alice, Age: 30
```

데이터 클래스는 간결하게 DTO를 정의할 때 매우 유용하게 사용된다. 복잡한 클래스도 간결하게 표현이 가능하기 때문이다.

## 구조 분해 선언

---

코틀린의 구조 분해 선언(destructing declaration)은 복합 객체의 프로퍼티를 한 번에 여러개의 변수에 할당할 수 있는 기능이다. 이 기능을 사용해서 코드를 더욱 간결하고 가독성 좋게 작성할 수 있다. 구조 분해 선언은 주로 데이터 클래스와 함께 사용되지만, 다양한 컬렉션이나 커스텀 객체에도 적용할 수 있다.

데이터 클래스는 자동으로 `componentN()` 메서드를 생성하여 구조 분해 선언을 지원한다.

```kotlin
data class User(val name: String, val age: Int)

val user = User("Alice", 30)

// 구조 분해 선언 사용
val (name, age) = user
println("Name: $name, Age: $age") // Output: Name: Alice, Age: 30
```

이렇게 구조 분해 선언을 사용하면 객체의 프로퍼티를 간단히 분해하여 각각의 변수에 할당할 수 있다. 이를 통해 코드가 더 명확해지고 직관적이게 된다.

데이터 클래스 외에서 사용할 때는 어떻게 하면 될까?

리스트나 배열에서도 구조 분해 선언을 사용할 수 있다.

```kotlin
val list = listOf("Alice", "Bob", "Charlie")

val (first, second, third) = list
println("$first, $second, $third") // Output: Alice, Bob, Charlie
```

Map 에 대해서도 맵의 요소를 반복문을 통해 구조 분해 선언을 사용할 수 있다.

```kotlin
val map = mapOf("name" to "Alice", "age" to 30)

for ((key, value) in map) {
    println("$key: $value")
}
// Output:
// name: Alice
// age: 30
```

그럼 사용자가 직접 정의한 클래스에 구조 분해를 사용하려면 어떻게 해야할까? `componentN()` 메서드를 직접 정의하면 된다.

```kotlin
class Person(val name: String, val age: Int) {
    operator fun component1() = name
    operator fun component2() = age
}

val person = Person("Alice", 30)
val (name, age) = person
println("Name: $name, Age: $age") // Output: Name: Alice, Age: 30
```

## 코틀린의 옵셔널

---

코틀린의 Nullable Type(optional)은 null 안정성을 보장하기 위해 도입된 개념이다. 코틀린에서는 기본적으로 모든 변수는 null 값을 가질 수 없다. 그러나 특정 변수나 객체가 null 값을 가질 수 있는 경우 이를 명시적으로 처리할 수 있다.

Nullable Type은 타입 뒤에 ?를 붙여 정의한다. 이는 해당 변수가 null을 가질 수 있음을 나타낸다. 

```kotlin
var name: String = "Alice"
name = null // 컴파일 오류: null을 허용하지 않음

var nullableName: String? = "Bob"
nullableName = null // 허용: nullableName은 Nullable Type
```

Nullable Type을 사용함으로써 null 참조 오류를 예방하고 명시적으로 null 처리를 할 수 있게 된다.

자바의 옵셔널과 다르게 코틀린 옵셔널 전용 안전한 연산자를 제공한다. 이를 통해 null 처리를 더욱 효과적으로 할 수 있다.

?.연산자는 변수나 객체가 null인지 확인한 후 null이 아닌 경우에만 해당 프로퍼티나 메소드를 호출합니다.

```kotlin
val length: Int? = nullableName?.length
println(length) // nullableName이 null이면 null을 반환
```

엘비스 연산자 (?:)는 변수나 객체가 null인 경우 기본 값을 대체하여 제공한다.

```kotlin
val length: Int = nullableName?.length ?: 0
println(length) // nullableName이 null이면 0을 반환
```

non-null 단정 연산자 (!!)는 변수나 객체가 null이 아님을 확신할 때 사용합니다. 만약 해당 연산자가 사용된 곳에서 null이 발견된 경우 NullPointerException이 발행됩니다.

```kotlin
val length: Int = nullableName!!.length
println(length) // nullableName이 null이면 예외 발생
```

코틀린의 옵셔널이 특별한 이유는 Null 체크를 하는 과정에서 해당 변수가 null이 아님을 확인하면 자동으로 non-null 타입으로 캐스팅 된다. 이를 스마트 캐스트라고 한다.

```kotlin
fun printLength(text: String?) {
    if (text != null) {
        println("Length: ${text.length}") // 스마트 캐스트로 text를 non-null로 처리
    } else {
        println("Text is null")
    }
}

printLength(nullableName)
```

## 제네릭

---

코틀린의 제네릭(Generic)은 타입을 매개변수로 사용할 수 있게 하여 코드의 재사용성을 높이고 타입 안정성을 보장하는 기능이다. 제네릭은 클래스, 인터페이스, 함수 등 다양한 요소에 적용가능하다.

기본적으로 제네릭 타입은 타입 파라미터를 사용하여 정의한다. 타입 파라미터는 일반적으로 대문자 T를 사용하여 나타낸다.

```kotlin
class Box<T>(val value: T)

val intBox = Box(1)
val stringBox = Box("Hello")

println(intBox.value) // Output: 1
println(stringBox.value) // Output: Hello
```

제네릭을 사용하면 컴파일 시 타입을 체크하여 타입 안전성을 보장할 수 있다. 또 다양한 타입에 대해 동일한 코드를 재사용할 수 있게 되어 유연한 코드 작성이 가능하다.

제네릭은 또한 함수에도 적용이 가능하다. 함수의 제네릭 타입은 함수 이름 앞에 타입 파라미터를 추가하여 정의하면 된다.

```kotlin
fun <T> printValue(value: T) {
    println(value)
}

printValue(123)       // Output: 123
printValue("Kotlin")  // Output: Kotlin
printValue(true)      // Output: true
```

제네릭 타입에 제약 조건을 추가하여 특정 타입이나 인터페이스를 상속받는 타입만 허용할 수도 있다.

```kotlin
fun <T : Number> add(a: T, b: T): Double {
    return a.toDouble() + b.toDouble()
}

println(add(1, 2))         // Output: 3.0
println(add(1.5, 2.3))     // Output: 3.8
// println(add("1", "2"))  // 컴파일 오류: String은 Number의 서브타입이 아님
```

코틀린의 제네릭은 공변성(convariance)과 반공변성(contravariance)을 지원하여 타입 간 상속 관계를 표현할 수 있다.

공변성은 특정 타입을 그 타입의 상위 타입으로 대체할 수 있게 한다. Out 키워드는 타입 파라미터가 생산자 역할을 할 때 사용한다.

```kotlin
interface Producer<out T> {
    fun produce(): T
}

class StringProducer : Producer<String> {
    override fun produce() = "Hello"
}

val producer: Producer<Any> = StringProducer()
println(producer.produce()) // Output: Hello
```

반공변성은 특정 타입을 그 타입의 하위 타입으로 대체할 수 있게 한다. in 키워드는 타입 파라미터가 컨슈머 역할을 할 때 사용한다.

```kotlin
interface Consumer<in T> {
    fun consume(item: T)
}

class AnyConsumer : Consumer<Any> {
    override fun consume(item: Any) {
        println(item)
    }
}

val consumer: Consumer<String> = AnyConsumer()
consumer.consume("Kotlin") // Output: Kotlin
```

## 확장 프로퍼티

---

코틀린의 확장 프로퍼티(Extension Property)는 기존 클래스에 새로운 프로퍼티를 추가할 수 있는 기능이다. 확장 프로퍼티를 사용하면 클래스의 코드 변경 없이도 원하는 프로퍼티를 확장할 수 있으며, 이는 특히 라이브러리에 내장되어 있는 클래스나 불변 클래스를 확장할 때 유용하다.

확장 프로퍼티는 기존 클래스에 새로운 프로퍼티를 추가하는 방법이다. 함수와 마찬가지로 클래스 이름 앞에 .을 붙여 정의한다.

```kotlin
val String.lastChar: Char
    get() = this[this.length - 1]

val str = "Kotlin"
println(str.lastChar) // Output: n
```

확장 프로퍼티는 확장 함수와 같이 기존 클래스를 수정하지 않고 새로운 프로퍼티를 만들 수 있다. 하지만 초기화가 불가능하므로 field처럼 생각하면 안된다.

확장 프로퍼티는 val 뿐만 아니라 var로도 정의하여 가변 확장 프로퍼티로 만들 수 있다. 이 경우에는 getter와 setter를 모두 명시적으로 제공해야 한다.

```kotlin
var StringBuilder.lastChar: Char
    get() = this[this.length - 1]
    set(value: Char) {
        this.setCharAt(this.length - 1, value)
    }

val sb = StringBuilder("Kotlin?")
sb.lastChar = '!'
println(sb) // Output: Kotlin!
```

## break & continue

---

코틀린의 break과 continue는 반복문 내에서 반복을 제어하기 위해 사용되는 제어 흐름문이다. 이 두 키워드를 사용해서 반복문을 유연하게 제어할 수 있다.

- break: 현재 반복을 중단하고 반복문을 빠져나옴
- continue: 현재 반복을 건너뛰고 다음 반복으로 넘어감

실전 코틀린 코드에서는 사실 잘 안쓰이긴 한다.

코틀린에서는 특별히 레이블(Label)을 활용해서 다중 반복문에서 특정 반복문을 제어할 수 있다. 레이블은 반복문 앞에 @를 붙여 정의하며 break과 continue문에서 사용된다.

```kotlin
outer@ for (i in 1..5) {
    for (j in 1..5) {
        if (i * j > 6) {
            break@outer
        }
        println("$i * $j = ${i * j}")
    }
}

// Output:
// 1 * 1 = 1
// 1 * 2 = 2
// 1 * 3 = 3
// 1 * 4 = 4
// 1 * 5 = 5
// 2 * 1 = 2
// 2 * 2 = 4
// 2 * 3 = 6
```

```kotlin
outer@ for (i in 1..3) {
    for (j in 1..3) {
        if (i == j) {
            continue@outer
        }
        println("$i != $j")
    }
}

// Output:
// 1 != 2
// 1 != 3
// 2 != 1
// 2 != 3
// 3 != 1
// 3 != 2
```
