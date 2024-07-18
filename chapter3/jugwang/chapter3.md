## 확장 함수

- 코틀린의 확장 함수는 기존 클래스에 멤버 함수를 추가하는 것과 같은 효과
- 확장할 대상 타입 : 수신 객체 타입
- 확장 함수를 정의하기 위해서는 함수 이름 앞에 수신 객체 타입을 붙여야 함

```kotlin
// 표현 방식
fun 수신타입.확장함수() { ... }

// 예시
fun String.singleQuote() = "'$this'"
fun String.doubleQuote() = "\"$this\""

fun main() {
	"Hi".singleQuote() eq "'Hi'"
	"Hi".doubleQuote() eq "\"Hi\""
```

- 이 확장 함수를 (확장 함수가 정의되지 않은) 다른 패키지에서 사용하려면 임포트해야한다.
- 클래스 내부에서 this를 생략할 수 있었던 것처럼 확장 함수 안에서도 this를 생략할 수 있다.

```kotlin
class Book(val title: String)

fun Book.categorize(category: String) =
	"""title: "$title", category: $category"""
	
fun main() {
	Book("Dracula").categorize("Vampire") eq
		"""title: "Dracula", category: Vampire"""
}
```

- Book.categorize(String) 은 categorize(Book, String) 으로 바꿔 쓸 수 있다.
- 확장 함수를 사용하는 이유
    
    → this를 사용함으로써(생략함으로써) 구문적 편의를 얻기 때문
    
- 문법 설탕이란?
    - 정의 : 문법 설탕은 프로그래밍 언어에 추가된 문법적 요소로, 기존의 기능을 더 쉽게 읽거나 표현할 수 있게 해줌
    - 목적 :
        - 코드를 더 간결하고 읽기 쉽게 만듦
        - 프로그래머의 생산성을 향상
        - 코드의 가독성을 높임
    - 예시 코드
    
    ```kotlin
    // 확장 함수 사용(문법 설탕)
    fun String.addExclamation() = this + "!"
    val result = "Hello".addExclamation()
    
    // 일반 함수 사용
    fun addExclamationToString(str: String) = str + "!"
    val result = addExclamationToString("Hello")
    ```
    

## 이름 붙은 인자와 디폴트 인자

- 이름 붙은 인자를 사용하면 코드 가독성이 좋아진다.

```kotlin
fun color(red: Int, green: Int, blue: Int) =
	"($red, $green, $blue)"
	
	
fun main() {
	color(blue = 0, red = 99, grenn = 52) eq
		"(99, 52, 0)"
	color(red = 255, 255, 0) eq
		"(255, 255, 0)"
```

- 이름 붙은 인자를 사용하면 색의 순서를 변경할 수 있다.
- 이름 붙은 인자와 일반(위치 기반) 인자를 섞어서 사용할 수 도 있다.
→ 그러나 가독성 뿐만 아니라 이름이 생략된 인자들의 위치를 컴파일러가 알아내지 못할 경우도 있기에 일단 인자 순서를 변경하고 나면, 인자 목록의 나머지 부분에서도 이름 붙은 인자를 사용해야한다.

```kotlin
fun color(
	red: Int = 0,
	green: Int = 0,
	blue: Int = 0,
) = "($red, $green, $blue)"

fun main() {
	color(139) eq "(139, 0, 0)"
	color(blue = 139) eq "(0, 0, 139)"
	color(255, 265) eq "(255, 165, 0)"
```

- 위처럼 이름 붙은 인자는 디폴트 인자와 결합하면 더 유용하다.
- 덧붙은 콤마(railing comma) : 마지막 파라미터에 콤마를 추가로 붙인 것
    - 콤마를 추가하거나 빼지 않아도 새로운 아이템을 추가하거나 아이템의 순서를 바꿀 수 있다.
- 이름 붙은 인자와 디폴트 인자, 그리고 덧붙은 콤마는 생성자에 쓸 수 있다.

- joinToString() 은 디폴트 인자를 사용하는 표준 라이브러리 함수
    - 이터레이션이 가능한 객체(리스트, 집합, 범위 등) 의 내용을 String 으로 합쳐줌
    - 이때, 원소 사이에 들어갈 문자열(구분자), 맨 앞에 붙일 문자열(접두사), 맨 뒤에 붙일 문자열(접미사)을 지정할 수도 있다.

```kotlin
fun main() {
	val list - listOf(1, 2, 3,)
	list.toString() eq "[1, 2, 3]"
	list.joinToString() eq "1, 2, 3"
	list.joinToString(prefix = "(",
		postfix = ")") eq "(1, 2, 3)"
	list.joinToString(separator = ":") eq
		"1:2:3"
}
```

- 객체 인스턴스를 디폴트 인자로 전달하는 경우 같은 인스턴스가 반복해서 전달된다.
- 디폴트 인자로 함수 호출이나 생성자 호출 등을 사용하는 경우, 함수를 호출할 때마다 해당 객체의 새 인스턴스가 생기거나 디폴트 인자에서 호출하는 함수가 호출된다.

```kotlin
class DefaultArg
val da = DefaultArg()

fun g(d: DefaultArg = da) = println(d)

fun h(d: DefaultArg = DefaultArg()) =
	println(d)
	
fun main() {
	g()
	g()
	h()
	h()
}

// 출력 결과
namedanddefault.DefaultArg@7440e464
namedanddefault.DefaultArg@7440e464
namedanddefault.DefaultArg@49476842
namedanddefault.DefaultArg@78308db1
```

- trimMargin() 함수 : 각 줄의 시작 부분을 인식하기 위한 경계를 표현하는 접두사 String을 파라미터로 받아 사용한다.

```kotlin
poem.trimMargin(marginPrefix = "|->")
```

## 오버로딩

- 파라미터 목록이 여러 다른 함수에 같은 이름을 사용하는 것
- 추후 작성……………………………………..

---

## when 식

- 두세 가지 이상의 선택지가 있는 경우 when 식을 사용하면 좋다.

```kotlin
fun ordinal(i: Int): String =
	when (i) {
		1 -> "erste"
		3 -> "dritte"
		7 -> "siebte"
		8 -> "achte"
		20 -> "zwanzigste"
		else -> numbers.getValue(i) + "te"
```

- else 가지를 없애면 “when’ expression must be exhaustive, add necessary ‘else’ branch” 컴파일 타입 오류 발생
- when 식을 문처럼 취급하는 경우(즉, when의 결과를 사용하지 않는 경우)에만 else 가지를 생략할 수 있음

```kotlin
fun processInputs(inputs: List<String>) {
	val coordinates = Coordinates()
	for (input in inputs) {
		when (input) {
		"up", "u" -> coordinates.y--
		"right", "r" -> {
			trace("Moving right")
			coordinates.x++
		}
		"nowhere" -> {}
		"exit" -> return
		else -> trace("bad input: $input")
	}
}
```

- when의 인자(when 다음에 있는 괄호 안에 올 값)로는 임의의 식이 올 수 있다.
- 매치 조건에도 아무 값이나 올 수 있다.(꼭 상수 X)

```kotlin
setOf("red", "blue") -> "purple"
```

- Set 을 사용해서 순서 상관없이 같은 값이 나오게 할 수 있다.

```kotlin
bmi < 18.5 -> "Underweight"
```

- 인자를 취하지 않는 특별한 형태
    
    → 인자가 없는 when에서는 화살표 왼쪽의 식에 항상 Boolean 타입의 식을 넣어야 한다
    

## 이넘

- 이넘은 이름을 모아둔 것

```kotlin
enum class Level {
	Overflow, High, Medium, Low, Empty
}

fun main() {
	Level.Medium eq "Medium"
```

- enum 을 만들면 enum의 이름에 해당하는 문자열을 돌려주는 toString()이 생성된다.
- when 식을 사용해 enum 항목마다 서로 다른 동작을 수행할 수 있다.
- 이넘은 인스톤스 개수가 미리 정해져 있고 클래스 본문 안에 이 모든 인스턴스가 나열되어 있는 특별한 종류의 클래스

```kotlin
enum class Direction(val notation: String) {
	North("N"), South("S"), East("E"), West("W"); //세미클론이 꼭 필요함
	val opposite: Direction
		get() = when (this) {
			North -> South
			South -> North
			West -> East
			East -> West
		}
}

fun main() {
	North.notation eq "N"
	North.opposite eq South
	West.opposite.opposite eq West
	North.opposite.notation eq "S"
```

- 추가 멤버를 정의하고 싶다면, 마지막 이넘 값 다음에 세미클론을 추가한 후 정의를 포함시켜야한다.
- 일반 생성자를 호출할 때처럼 괄호를 사용해 생성자 인자를 전달하면(North(”N”)) notation 의 값을 전달할 수 있다.
- opposite 프로퍼티 게터에 접근할 때마다 게터가 동적으로 결과를 계산한다.

## 데이터 클래스

- 데이터 저장만 담당하는 클래스가 필요하면 data 클래스를 사용해 코드양을 줄이면서 여러 가지 공통 작업을 편하게 수행할 수 있다.
- data 클래스를 정의할 때는 data 라는 키워드를 사용한다.
- 이때 모든 생성자 파라미터를 var나 val로 선언해야 한다.

```kotlin
class Person(val name: String)

data class Contact(
	val name: String,
	val number: String
)

fun main() {
	Person("Cleo") neq Person("Cleo") //둘은 다르다
	// 데이터 클래스는 타당한 동등성 검사 제공
	Contact("Miffy", "1-234-567890") eq
		Contact("Miffy", "1-234-567890")
}

// 출력 결과
dataclasses.Person@54bedef2
Contact(naem=Miffy, number=1-234-567890)
```

- Person 클래스 data 키워드 없이 정의됐으므로 같은 name을 담은 두 인스턴스가 서로 동등하지 않다.
- data 클래스의 유용한 함수 - copy()
    - 현재 객체의 모든 데이터를 포함하는 새 객체를 생성
    - copy() 의 파라미터 이름은 생성자 파라미터의 이름과 같다.

- data 클래스를 만들면 객체를 HashMap이나 HashSet에 넣을 때 키로 사용할 수 있는 해시 함수를 자동으로 생성해준다.

```kotlin
data class Key(val name: String, val id: Int)

fun main() {
	val korvo: Key = Key("Korvo", 19)
	korvo.hashCode() eq -2041757108
	val map = HassMap<Key, String>()
	map[korvo] = "Alien"
	map[korvo] eq "Alien"
	val set = HashSet<Key>()
	set.add(korvo)
	set.contains(korvo) eq true
```

## 구조 분해 선언

- 표준 라이브러리에 있는 Pair 클래스

```kotlin
fun compute(input: Int): Pair<Int, String> =
	if (input > 5)
		Pair(input * 2, "High")
	else
		Pair(input * 2, "Low")
		
fun main() {
	compute(7) eq Pair(14, "High")
	compute(4) eq Pair(8, "Low")
	val result = compute(5)
	result.first eq 10
	result.second eq "Low"
}
```

- Pair는 List나 Set처럼 파라미터화된 타입
- 구조 분해 선언을 통해서 여러 식별자를 동시에 선언하면서 초기화할 수 있다.

<aside>
💡 val (a, b, c) = 여러_값이_들어있는_값

</aside>

- 여러 값이 들어 있는 값(객체)을 여러 컴포넌트로 분해해서 각 컴포넌트를 순서대로 대입해준다.
- 구조 분해 구문에서는 (등호 왼쪽에 있는) 식별자 이름을 괄호 안에 넣어야 한다.

```kotlin
fun main() {
	val (value, description) = compute(7)
	value eq 14
	description eq "High"
}
```

위 코드는 compute() 가 반환하는 Pair를 구조 분해하는 선언

- Triple 클래스는 정확히 세 가지 값을 묶는다.
- 코틀린이 Pair 와 Triple 만 제공하는 이유
→ 더 많은 값을 저장하고 싶거나 코드에서 Pair와 Triple을 많이 사용한다면 각 상황에 맞는 특별한 클래스를 작성하라는 뜻

```kotlin
data class Computation(
	val data: Int,
	val info: String
)
```

- Pair<Int, String> 반환하는 것보다 위처럼 Computation 을 만들어 반환하는 게 낫다. → 자신의 역할을 잘 설명하는 이름을 붙이는 것이 좋다

 

```kotlin
val (_, _, animal) = tuple
animal eq "Mouse"
```

- 구조 분해 선언으로 선언할 식별자 중 일부가 필요하지 않은 경우,
이름 대신 밑줄(_)을 사용할 수 있고, 맨 뒤쪽의 이름들은 아예 생략 가능

- data 클래스의 프로퍼티는 이름에 의해 대입되는 것이 아니라 순서대로 대입
- 유의 사항
    - 어떤 객체를 구조 분해에 사용했는데 이후 그 data 클래스에서 맨 마지막이 아닌 위치에 프로퍼티를 추가하는 경우,
    새 프로퍼티가 기존에 다른 값을 대입받던 식별자에 대입되면서 예상과 다른 결과를 낳을 수 있음 → 컴파일러가 잡아주지 못함
- Pair나 Triple 같은 라이브러리가 제공하는 data 클래스는 프로퍼티 순서가 바뀌지 않으므로 구조 분해해도 안전
- withIndex() 는 표준 라이브러리가 List에 대해 제공하는 확장 함수
    - 컬렉션의 값을 IndexedValue라는 타입의 객체에 담아서 반환하며, 구조 분해 가능하다

## 널이 될 수 있는 타입

- 자바에서는 null 또는 의미 있는 값이 결과가 되도록 허용한다.
- 코틀린에서 모든 타입은 기본적으로 널이 될 수 없는 타입
- 하지만 무언가 null 결과를 내놓을 수 있다면, 타입 이름 뒤에 물음표(?)를 붙여서 결과가 null이 될 수도 있음을 표시

```kotlin
val s3: String? = null
val s4: String? = s1
```

- 타입 이름 끝에 ?을 붙여서 기존 타입을 다른 타입으로 지정한다.
    - String 과 String? 는 서로 다은 타입
- 코틀린에서는 null이 될 수 있는 타입을 단순히 역참조(즉, 멤버 프로퍼티나 멤버 함수에 접근)할 수 없다.

```kotlin
fun main() {
	val s1: String = "abc"
	val s2: String? = s1
	
	s1.length eq 3 // [1]
	// 컴파일 불가
	// s2.length  // [2]
```

- [1] 널이 될 수 없는 타입의 멤버에 접근할 수 없다.
- [2] 널이 될 수 있는 타입의 멤버를 참조하는 경우, 코틀린은 오류를 발생시킨다.
- 대부분 타입의 값은 메모리에 있는 객체에 대한 참조로 저장된다.
→ 역참조의 의미

## 안전한 호출과 엘비스 연산자

- 안전한 호출 : 일반 호출에 사용하는 점(.)을 물음표와 점(?.) 으로 바꾼 것
- 안전한 호출을 사용하면 널이 될 수 있는 타입의 멤버에 젭근하면서 아무 예외도 발생하지 않게 해줌
- 수신 객체가 null이 아닐 때만 연산을 수행

- 엘비스 연산자 : ?.결과로 null을 만들어내는 것 이상의 일이 필요할 때 사용
    - ?: 의 왼쪽 식의 값이 null이 아니면 왼쪽 식의 값이 전체 엘비스 식의 결과값
    왼쪽 식이 null이면 ?:의 오른쪽 식의 값이 전체 결과값

```kotlin
fun main() {
	val s1: String? = "abc"
	(s1 ?: "---") eq "abc"
	val s2: String? = null
	(s2 ?: "---") eq "---"
}
```

## 널 아님 단언

- null이 될 수 없다고 주장하기 위해 느낌표 두 개(!!) 쓴다
- x!! 는 ‘x 가 null일 수도 있다는 사실을 무시하라. 내가 x가 null이 아니라는 점을 보증한다’ 라는 뜻
- x!! 는 x 가 null이 아니면 x를 내놓고, null 이면 오류를 발생시킨다.

```kotlin
fun main() {
	var x: String? = "abc"
	x!! eq "abc"
	capture {
		val s: String = x!!
	} eq "NullPointerException"
}
```

## 확장 함수와 널이 될 수 있는 타입

- isNullOrEmpty() : 수신 String이 null이거나 빈 문자열인지 검사한다.
- isNullOrBlank() : isNullOrEmpty() 와 같은 검사를 수행한다. 그리고 수신 객체 String이 온전히 공백 문자(\t , \n 포함) 로만 구성되어 있는지 검사한다.

```kotlin
fun main() {
	val s1: String? = null
	s1.isNullOrEmpty() eq true
	s1.isNullOrBlank() eq true
	
	val s2 = ""
	s2.isNullOrEmpty() eq true
	s2.isNullOrBlank()eq true
```

## 제네릭스 소개

## 확장 프로퍼티

## break와 continue
