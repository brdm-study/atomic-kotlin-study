## 객체

- 객체는 프로퍼티(val, var)를 사용해 데이터를 저장하고, 함수를 사용해 이런 데이터에 대한 연산을 수행한다.
- 멤버 함수를 호출하려면 객체를 가리키는 식별자를 지정하고 점(.)과 연산의 이름을 덧붙인다.

## 클래스

- 클래스를 정의함으로써 새로운 타입을 정의할 수 있다.
- 클래스 본문 안에 정의된 함수를
    - 자바 : 메서드
    - 코틀린 : 멤버 함수
- 멤버 함수는 어떤 클래스에 속한 특정 인스턴스에 대해 작용한다.
- 멤버 함수를 호출할 때 코틀린은 조용히 객체를 가리키는  참조를 함수에 전달해서 관심의 대상이 되는 객체를 추적한다.
- 멤버 함수 안에서는 this라는 이름으로 이 참조에 접근할 수 있다.

```kotlin
class Hamster {
	fun speak() = "Squeak! "
	fun exercise() =
		this.speak() + // 'this' 로 한정함
			speak() + // 'this' 없이 호출함
			"Running on wheel"
}

fun main() {
	val hamster = Hamster() //객체 생성
	println(hamster.exercise())
}
```

- 어떤 특성이 불필요한데도 this를 사용한다면, 코드를 읽는 사람은 왜 코드를 이렇게 짰는지 고민하느라 시간을 쓰거나 헷갈릴 수 있다.
→ 불필요한 this는 사용하지 않는 것이 좋다.

## 프로퍼티

- 프로퍼티는 클래스에 속한 var 나 val 이다.
    - var : 변수에 다시 대입이 가능
    - val : 다시 대입할 수 없음
- 프로퍼티를 정의함으로써 클래스 안에서 상태를 유지한다.
- 각각의 객체는 프로퍼티를 저장할 자신만의 공간을 따로 할당받는다.

- 안티패턴 : 가변(var)인 최상위 프로퍼티를 선언하는 일
→ 프로그램이 복잡해질수록 공유된 가변 상태에 대해 제대로 추론하기 어렵기 때문

```kotlin
class Kitchen {
	var table: String = "Roundtable"
}

fun main() {
	val kit chen1 = Kitchen()
	val kit chen2 = kitchen1
	println("kitchen1: ${kitchen1.table}") 
	println("kitchen2: ${kitchen2.table}") 
	kitchen1.table = "Square table"
	println("kitchen1: ${kitchen1.table}") 
	println("kitchen2: ${kitchen2.table}")
}

// 출력
kitchen1: Round table
kitchen2: Round table
kitchen1: Square table
kitchen2: Square table
```

- var와 val이 객체가 아니라 참조를 제한한다.
- var를 사용하면 참조가 가르키는 대상을 다른 대상으로 다시 엮을 수 있지만, val을 사용하면 다른 대상을 엮을 수 없다.
→ 객체에서 가변성은 내부 상태를 바꿀 수 있다는 뜻

## 생성자

- 생성자는 정보를 전달해 새 객체를 초기화할 수 있다.

```kotlin
class Alien(name: String) {
	val greeting = "Poor $name!"
}
	
fun main() {
	val alien = Alien("Mr.MeeSeeks")
	println(alien.greeting)
}
```

- 코틀린에서는 new가 불필요한 중복이기 때문에 제외
- 생성자 밖에서는 name에 접근할 수 없다.
- 클래스 본문 밖에서도 생성자 파라미터에 접근할 수 있게 하려면 파라미터 목록에 var나 val을 지정해야한다.

- 디폴트 toString()
    
    ```kotlin
    // 출력
    AlienSpecies@4d7e1886
    ```
    
- 오버라이드 된 toString()
    
    ```kotlin
    override fun toString(): String {
    	return "Scientist('$name)"
    }
    
    // 출력
    Scientist('Zeep Xanflorp')
    ```
    
    - override를 명시함으로써 코드의 의도를 더 명확히 할 수 있다.

## 가시성 제한

- 가시성을 제어하기 위해 접근 변경자 제공
- 변경자는 클래스, 함수, 프로퍼티 정의 앞에 위치
- 변경자는 변경자가 붙어 있는 그 정의의 가시성만 제한
- 어떤 클래스에서 도우미 함수를 쓰이는 멤버 함수를 private으로 선언하면 클래스 외부에서 이 함수를 실수로 쓰는 경우를 방지
→ 가능한 한 많은 요소를 private 으로 선언
- 에일리어싱 : 한 객체에 대해 여러 개 유지하는 경우
- 모듈 : 코드 기반상에서 논리적으로 독립적인 각 부분을 뜻함
    - 코드를 모듈로 나는 방법은 빌드 시스템(그레이들, 메이븐)에 따라 달라짐
- internal 정의는 그 정의가 포함된 모듈 내부에서만 접근 가능
- internal은 private 과 public 사이에 위치

## 패키지

- 패키지는 연관 있는 코드를 모아둔 것

```kotlin
import kotlin.math.PI as circleRatio // 정적 팩토리 메서드에 별칭
import kotlin.math.cos as cosine

fun main() {
    println(circleRatio)
    println(cosine(circleRatio))
    println(cosine(2 * circleRatio))
}
```

- as는 라이브러리에서 이름을 잘못 선택했거나 이름이 너무 길 때 유용

- 파일 이름이 항상 클래스 이름과 같아야 하는 자바와 달리, 코틀린에서는 소스 코드 파일 이름으로 아무 이름이나 붙여도 좋다.
- 패키지 이름도 아무 이름이나 선택할 수 있다.
    
    (패키지 이름과 패키지 파일이 들어 있는 경로를 똑같이 하는게 좋은 스타일로 여겨짐)
    

## 테스트

- JUnit : 자바에서 가장 널리 쓰이는 테스트 프레임워크이며, 코틀린에서도 유용하게 쓸 수 있다.
- 코테스트(Kotest) : 코틀린 전용으로 설계됐으며, 코틀린 언어의 여러 기능을 살려서 작성됐다.
- 스펙(Spek) 프레임워크 : 명세 테스트라는 다른 형태의 테스트를 제공한다.

- eq(equals, 같다)
- neq(not equals, 같지 않다)

- trace 객체는 출력을 저장해서 나중에 사용할 수 있게 해준다.
- trace에 결과를 추가하는 것은 함수 호출처럼 보이기 때문에 println()을 trace()로 대치할 수 있다.

## 예외

- 예외는 오류가 발생한 지점에서 ‘던져지는’ 객체
- 예외를 잡아내지(catch) 않으면 프로그램이 중단되면서 예외에 대한 상세 정보가 들어 있는 스택 트레이스가 출력된다.

```kotlin
Exception in thread "main" java.lang.NumberFormat Exception: For input string: "1$"
at java.lang.NumberFormat Exception. forlnputString(NumberFormat Exception. java:65)
at java.lang.Integer. parselnt (Integer. java:580)
at java.lang.Integer. parselnt (Integer. java:615)
at Tolnt ExceptionKt .erroneousCode(at Tolnt Exception.kt :6)
at Tolnt ExceptionKt .main(at Tolnt Exception.kt :10)
```

- 스택 트레이스는 예외가 발생한 파일과 위치 등과 같은 상세 정보를 표시한다.

```kotlin
fun averageIncome(income: Int, months: Int) = 
	if (months == 0)
    	throw IllegalArgumentException("Months can't be zero")
    else
    	income / months
```

- 예외를 던질 때는 throw 키워드 다음에 던질 예외의 이름을 넣고 그 뒤에 예외에 필요한 인자들을 추가한다.

## 리스트

- 리스트는 컨테이너, 즉 다른 객체를 담는 객체에 속한다.
    - 컨테이너는 컬렉션이라고도 한다.
- List는 표준 코틀린 패키지에 들어 있기 때문에 import를 할 필요가 없다.

```kotlin
fun main() {
	val ints = listOf(99,3,5,7,11, 13)
}
```

- List는 자기 자신을 표시할 때 각 괄호([])를 쓴다.

- sort 와 sorted
    - sort : 함수 이름을 ‘sort’라고 붙이면 원본 List를 직접(이를 인플레이스라고 말하기도 한다) 바꾼다는 의미가 들어 있다.
    - sorted : 호출하면 원본의 원소들은 정렬한 새로운 List를 돌려준다. 따라서 원래의 List는 그대로 남아 있다.

```kotlin
fun main() {
	//타입을 추론한다.
    val numbers = listOf(1, 2, 3)
    val strings = listOf("one", "two", "three")
    
    //타입을 명시한다.
    val numbers2: List<Int> = listOf(1, 2, 3)
    val strings2: List<String> = listOf("one", "two", "three")
}
```

- List의 타입은 추론할 수도, 타입 파라미터를 통해 명시할 수도 있다.

```kotlin
//반환할 타입을 추론한다.
fun inferred(p: Char, q: Char) = 
	listOf(p, q)

//반환할 타입을 명시한다.
fun explicit(p: Char, q: Char): List<Char> = 
	listOf(p, q)
```

- 반환값의 타입이 타입 파라미터를 포함할 수도 있다.

- 가변 List는 필요하다고 명시적으로 표시해야만 얻을 수 있다.
- 읽기 전용과 가변 List
    - listOf() 는 상태를 변화시키는 함수가 들어 있지 않은 읽기 전용 리스트를 만들어낸다.
    - mutableListOf() 는 변경할 수 있는 MutableList 를 돌려준다.
        - add() 나 addAll()을 사용해 MutableList에 원소나 다른 컬렉션의 원소들을 추가할 수 있다.
        
- +=의 비밀
- += 연산자를 쓰면 불변 리스트가 마치 가변 리스트인 것처럼 보인다.

```kotlin
fun main() {
	var list = listOf('X')	//불변리스트
    list += 'Y'	          //가변 리스트처럼 보임
    list eq "[X, Y]"
}
```

- list는 var이라서 가능하다.

## 가변 인자 목록

- vrarag 키워드는 길이가 변할 수 있는 인자 목록을 만든다.
- vararg 키워드를 사용하면 listOf처럼 임의의 길이로 인자를 받을 수 있는 함수를 정의할 수 있다.
    - vararg = variable argument list
- 함수 정의에는 vararg로 선언된 인자는 최대 하나만 있어야 한다.
- 일반적으로 마지막 파라미터를 vararg로 선언하는 게 간편하다.
- vararg를 사용하면 함수에 임의의 개수만큼(0포함) 인자를 전달할 수 있다.
- 함수 본문에서는 파라미터 이름을 통해 vararg 인자에 접근할 수 있다.
    - 이 때 파라미터는 Array로 취급된다.

```kotlin
fun sun(vararg numbers: Int): Int {
	var total = 0
    for (n in numbers) {
    	total += n
    }
    return total
}
```

- Array는 항상 가변 객체라는 점에 유의하라.
- Array를 (Array 타입의 인자 하나로 넘기지 않고) 인자 목록으로 변환하고 싶으면 스프레드 연산자(*)를 사용하라.

```kotlin
val list = listOf(9, 10, 11)
sum(*list.toIntArray()) eq 30
```

- 스프레드 연산자는 배열에만 적용할 수 있다.
- List를 인자 목록에 전달하고 싶을 때에는 먼저 Array로 변환한 다음에 스프레드 연산자를 사용해야 한다.

## 집합

- Set은 각각의 값이 오직 하나만 존재할 수 있는 컬렉션
- Set 의 특징
    - 중복 불가능
    - 순서 중요하지 않음
    - 원소인지 여부를 검사하기 위해 in 과 contains()를 모두 쓸 수 있음
    - 여러 가지 일반적인 벤 다이어그램 연산을 수행할 수 있음
    
- List 에서 중복을 제거하려면
    - Set 으로 변환
    - List를 반환하는 distinct() 사용

- 읽기 전용 - setOf()
- 가변 set - mutableSetOf()

## 맵

- Map은 키와 값을 연결하고, 키가 주어지면 그 키와 연결된 값을 찾아준다.
- 키-값 쌍을 mapOf()에 전달해 Map을 만들 수 있다.
- 키와 값을 분리하려면 to를 사용한다.
- 읽기 전용 - mapOf()
    - 상태 변경을 허용하지 않는다.
- 가변 - mutableMapOf()
- mapOf() 와 mutableMapOf()는 원소가 Map에 전달된 순서를 유지해준다.
- 키와 값을 연관(연결) 시켜주기 때문에 때로 Map을 연관 배열이라고 부르기도 한다.

```kotlin
fun main() {
	val constants = mapOf(
		"Pi" to 3.141,
		"e" to 2.718,
		"phi" to 1.618
	)
	constants eq
		"{Pi=3.141, e=2.718, phi=1.618}"
		
	// 키에 해당하는 값을 찾는다
	constants["e"] eq 2.718
	constants.keys eq setOf("Pi", "e", "phi")
	contstants.values eq "[3.141, 2.718, 1.618]"
	
	var s = ""
	// 키-값 쌍을 이터레이션한다
	for (entry in constants) {
		s += "${entry.key} = ${entry.value}, "
		}
		s eq "Pi=3.141, e=2.718, phi=1.618,"
		
		s = ""
		// 이터레이션을 하면서 키와 값을 분리한다
		for ((key, value) in constants)
			s += "$key=$value, "
		s eq "Pi=3.141, e=2.718, phi=1.618, "
	}
```
