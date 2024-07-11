# 코틀린 공부 정리

## 01 소개
- 작은 단계를 거치며 작은 성취 맛보기
- 전방 참조 없음
- 다른 언어를 참조하지 않음
- 설명보다는 예제
- 이론보다는 실습

## 02 왜 코틀린인가
프로그래밍 언어 설계는 프로그래머를 더 생산적으로 만들기 위해 필요한 기능이 무엇인지 계속 추측하고 가정하는 것

- **가독성**: 간결한 문법
- **도구**: 젯브레인즈
- **다중 패러다임**: 명령형, 함수형, 객체 지향 프로그래밍
- **다중 플랫폼**: JVM, Android, JS
- **자바 상호 운용성**
- **NULL 값 표현 방식**: 널 오류를 발생시킬 가능성이 있는 연산을 컴파일 시점에, 프로그램이 실행되기도 전에 금지함으로써 이런 문제를 해결

## 03 Hello, World!
- **주석**: `//`(연속된 슬래시 두 개), 여러 줄 주석: `/* 내용 */` - 코드만 보고 분명히 알 수 없는 정보를 덧붙이기 위해 쓰임
- **fun**: function(함수)을 뜻하는 키워드
- **main**: 코틀린 프로그램의 'Entry Point(진입점)'를 뜻하는 특별한 함수 -> 코틀린 실행 시 자동으로 호출 됨

```kotlin
fun main() {
    println("Hello, World!")
}
```

## 04 var와 val
- **var**: 변할 수 있는 수(variable). 내용 재대입 가능 -> mutable
- **val**: 값(value). 단 한 번만 초기화 가능. 초기화 이후 내용 변경 불가 -> immutable
- **val or var 식별자: 타입 = 초기화 식**

```kotlin
var mutableVariable: Int = 10
val immutableValue: Int = 10
```

## 05 데이터 타입
타입을 지정하지 않아고 보통은 코틀린이 타입을 추론해 줌

```kotlin
val myInt = 5
val myDouble = 5.99
val myBoolean = true
val myString = "Hello"
val myChar = 'A'
val myRawString = """
    This is a raw string.
    You can write multiple lines.
"""
```

## 06 함수
- **fun 함수이름(파라미터명: 타입, ...)**: 반환타입 {} -> 반환타입은 생략 가능
- **fun 함수이름(파라미터명: 타입, ...)**: 반환타입 = 결과를 계산하는 식 -> 식 본문 함수
- **함수 시그니처** = 함수 이름 + 파라미터 + 반환타입
- **전통적인 명령형 언어의 관점**: 함수 = 여러 명령어를 묶어서 이름 붙인 축약
- **함수형 언어의 관점**: 함수 = 치환 가능한 값이 있는 식
- 의미 있는 결과를 제공하지 않는 함수의 반환타입은 Unit
- 함수를 작성할 때는 서술적인 이름을 사용

```kotlin
fun add(a: Int, b: Int): Int {
    return a + b
}

fun greet(name: String) = "Hello, $name"
```

## 07 if 식

```kotlin
fun main() {
    val a = 5
    val b = 10
    
    if (a > b) {
        println("a는 b보다 크다")
    } else if (a < b) {
        println("a는 b보다 작다")
    } else {
        println("a와 b는 같다")
    }
}
```

## 08 문자열 템플릿
- **문자열 템플릿**: String을 프로그램으로 만드는 방법
- **$**: 식별자의 내용을 String에 넣어줌. $ 다음에 식별자가 오지 않으면 아무일도 없음
- **+**: 문자열 연결
- **${}**: 식을 평가하여 그 결과를 String으로 변환해 삽입
- **\(역슬래시) or 큰따옴표 연속 3개**: String안에 특수 문자를 넣을 경우

```kotlin
val name = "John"
val greeting = "Hello, $name!"
val age = 30
val ageInfo = "Next year, I will be ${age + 1} years old."
```

## 09 수 타입
- 타입에 따라 서로 다른 방식으로 저장됨
- 코틀린은 숫자 사이에 밑줄을 넣도록 허용
- 정수를 다른 정수로 나누면 코틀린은 정수를 반환 -> 나머지 버림
- 나눗셈이 들어간 계산 식에서는 순서에 유의해야 함
- Int의 범위를 벗어나면 overflow -> Long 사용

```kotlin
val num1 = 100_000
val num2 = 2_000
val division = num1 / num2 // 결과는 50
val largeNumber = 1_000_000_000L
```

## 10 불리언
- **&&(논리곱)**: and 연산
- **||(논리합)**: or 연산
- 조건식은 헷갈리기 쉬우므로 항상 가독성을 유지하고 의도를 명시해야 함
- 괄호가 없는 경우 우선순위는 and > or

```kotlin
val isAdult = true
val hasPermission = false
val canEnter = isAdult && hasPermission // 결과는 false
```

## 11 while로 반복하기
while은 주어진 boolea식이 true인 동안 반복을 수행

```kotlin
var i = 0
while (i < 5) {
    println(i)
    i++
}
```

```kotlin
var i = 0
do {
    println(i)
    i++
} while (i < 5)
```

## 12 루프와 범위
- for 키워드는 주어진 순열 속에 속한 각 값에 대해 코드블록을 실행
- 주어진 값이란? String, Collection 등
- 주로 in 키워드와 함께 사용

```kotlin
for (i in 1..5) {
    println(i)
}
```

- **..**: 양 끝값을 포함한 범위
- **until**: 마지막 값은 제외 -> ‘..<’로 사용하는 것으로 최근 업데이트
- **downTo**: 감소하는 범위 (폐구간)
- **String.lastIndex**: 문자열의 마지막 인덱스 값
- **repeat**: 표준 라이브러리 함수 - 정해진 횟수만큼 반복해줌

```kotlin
for (i in 5 downTo 1) {
    println(i)
}

repeat(3) {
    println("Hello")
}
```

## 13 in 키워드
- for 루프 제어식에 있는 in = 이터레이션
- 나머지 in은 모두 원소 여부 검사 용도
- hasChar() 대신 in 사용 가능
- 코틀린 1.8버전 부터는 반 열린 범위를 부동 소수점 범위에 적용 가능 (1.0 ..< 2.0)
- String이 어떤 String 범위 안에 속하는지 검사 가능 -> 알파벳 순으로 비교

```kotlin
val range = 1..10
val isInRange = 5 in range // 결과는 true

val str = "Kotlin"
val hasCharK = 'K' in str // 결과는 true
```

## 14 식과 문
- **문(statement)**: 효과를 발생시키지만 결과를 내놓지 않음 (ex. for문)
- **식(expression)**: 결과를 만듦 (ex. 함수식)
- if는 식을 만들 수 있음 -> if의 결과를 변수에 대입 가능
- **전위 연산자(prefix operator)**: 변수값을 먼저 변경 -> 값을 반환
- **후위 연산자(postfix operator)**: 값을 반환 -> 변수값을 나중에 변경

```kotlin
val max = if (a > b) a else b

var x = 5
val y = ++x // y는 6, x는 6

var p = 5
val q = p++ // q는 5, p는 6
```
