# 1부 프로그래밍 기초
## 컴파일 언어
코틀린은 해석(interpret)되지 않고 컴파일(compile)된다.  
JVM 기반의 자바나 코틀린 같은 언어는 소스코드를 바이트코드로 이루어진 class 파일로  
컴파일을 통해 바이트코트로 번역된 `.class` 파일로 만들어지고, `.class` 파일은 JVM 에서 실행된다.

## 코틀린의 두 가지 특징
1. **자바와의 상호 운영성** 
   - 코틀린은 별다른 노력 없이 자바 프로젝트에 코틀린을 통합할 수 있다.
   - 작게 코틀린 기능을 작성해서 기존 자바 코드 사이에 끼워 넣을 수 있다.
   - 자바 코드 입장에서는 코틀린 코드가 마치 자바 코드처럼 보인다.
2. **빈 값 표현 방식**
   - 코틀린은 널 오류를 발생시킬 가능성이 있는 연산을 **컴파일 시점**에 금지한다.
   - 따라서 자바의 널 오류를 최소화하거나 없앨 수 있다.

## 함수 표현 방식
- 코틀린은 다른 언어와 달리 식 끝에 세미콜론 (`;`)을 붙이지 않아도 된다.
- 한 줄 안에 여러 식을 넣을 때 사용할 수 있으나 권장하지 않는다.

## var와 val 타입

```java
// 자바
public static void main(String[] args) {
    int mutable = 10; // 가변
    final int immutable = 20; // 불변
}
```
```kotlin
// 코틀린
fun main() {
    var mutable = 10 // 불변
    val immutable = 20 // 가변
}

```

## 변수 할당

```java
// 자바
int variable_1 = 10;
var variable_2 = 20; // 타입추론 -> java 10부터 지원
```

```kotlin
// 코틀린
var variable_1 = 10 // #1
var variable_2: Int = 20 // #2
```

- 코틀린은 기본적으로 타입추론을 사용한다.
- 2번과 같이 타입을 직접 명시할 수 있다.

## 함수 선언

```java
// 자바
public 반환타입 메서드명(타입1 파라미터1, 타입2 파라미터2, ...) {
    return 반환타입에 맞는 변수;
}
```
```kotlin
// 코틀린
fun 함수이름 (파라미터1: 타입1, 파라미터2: 타입2): 반환타입 {
    return 결과값
}
```
- 함수 본문이 한 식으로만 이뤄진 경우(본문 함수) 아래와 같이 축약형으로 정의할 수 있다.
    ```kotlin
    fun 함수이름(파라미터1 : 타입1, 파라미터2: 타입2, ... ): 반환타입 = 결과를_계산하는_식
    
    // 예시
    fun cube(x: Int): Int = x * x * x;
    ```
  
## 문자열 템플릿

```kotlin
val name = "Alice"
val age = 30
val message = "Hello, $name. You are $age years old."
println(message)

// 출력
// Hello, Alice. You are 30 years old.
```
- 코틀린은 `$` 를 사용하여 변수를 문자열에 삽입 할 수 있다.

    ```kotlin
    fun main() {
        val answer = 42
        println("Found$answer!") // #1
        val condition = true
        println("${if (condition) 'a' else 'b'}") // #2
        println("printing a $1") // #3
    }
    ```
  - **1 :** 문자열에 변수 삽입
  - **2 :** 조건문의 결과도 가능
  - **3 :** 프로그램 식별자가 아닌 문자열 앞에 `$`가 붙은 경우에는 아무일 없음.


## 루프와 범위

```java
// 자바
for (int i = 0; i < 10; i++) {
    System.out.print(i + " ");
}

for (int i = 0; i < 10; i=i+2) {
    System.out.print(i + " ");
}

for (int i = 9; i >= 0; i--) {
    System.out.print(i + " ");
}

for (int number : numbers) {
    System.out.print(number + " ");
}
```

```kotlin
// 코틀린
for (c in "Kotlin") {
	print(c)
}

for (i in 0 until 10) { // 0부터 9까지
    print(i + " ")
}

for (i in 0..10) { // 0부터 10까지
    print("$i ")
}

for (i in 0 until 10 step 2) { // 0부터 9까지 2씩 증가
    print(i + " ")
}

for (i in 10 downTo 1 step 2) { // 역순으로 2씩 감소
    print(i + " ") // 10, 8, 6, 4, 2
}

```
- `..` : 양 끝의 값을 포함한 범위
- `until` : 오른쪽 끝 값을 제외한 범위

범위를 변수에 담을 수도 있다.

```java
// java
IntStream range = IntStream.rangeClosed(0, 10);

// Kotlin
var range = 0..10
```

### in 키워드
- 코틀린은 `in` 키워드를 사용해서 어떤 값이 범위에 속하는지를 검사한다.
- 반면 `!in`은 범위에 속하지 않은 경우 `true`를 반환한다.

```kotlin
fun inNumRange(n: Int) = n in 50..100
fun notLowerCase(ch: Char) = ch !in 'a'..'z'
fun main() {
    val i1 = 11
    val i2 = 100
    val c1 = 'K'
    val c2 = 'k'
    println("$ i1 ${inNumRagne(i1)}")
    println("$ i2 ${inNumRagne(i2)}")
    println("$ c1 ${notLowerCase(c1)}")
    println("$ c2 ${notLowerCase(c2)}")
}

// 출력
// 11 false
// 100 true
// K true
// k false
```