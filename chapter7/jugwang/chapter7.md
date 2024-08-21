# 코틀린 정리 - 7장

## 확장 람다

> 확장 람다는 확장 함수와 비슷하다. 차이가 있다면 함수가 아니라 람다라는 점
> 

- 코틀린 문서에서는 확장 람다를 수신 객체가 지정된 함수 리터럴이라고 말한다.
    
    (함수 리터럴이라는 말은 람다와 익명 함수를 모두 포함)
    
- 수신 객체가 암시적 파라미터로 추가 지정된 람다라는 사실을 강조하고 싶을 대는 수신 객체가 지정된 람다를 확장 람다의 동의어로 더 자주 사용한다.
- 확장 함수와 마찬가지로 확장 람다도 여러 파라미터를 받을 수 있다.

```kotlin
val zero: Int.() -> Boolean = {
	this = 0
}

val one: Int.(Int) -> Boolean = {
	this % it = 0
}

val two: Int.(Int, Int) -> Boolean = {
	arg1, arg2 ->
	this % (arg1 + arg2) == 0
}

val three: Int.Ont, Int, Int) -> Boolean = {
	arg1, arg2, arg3 ->
	this % (arg1 + arg2 + arg3) == 0
}

fun main() {
	0.zero() eq true
	10.one(10) eq true
	20.two(10, 10) eq true
	30.three(10, 10, 10) eq true
}

// 지금까지는 val 선언해서 확장 람다를 소개
```

```kotlin
class A {
	fun af() = 1
}

class B {
	fun bf() = 2
}

fun f1(lambda: (A, B) -> Int) =
	lambda(A(), B())
	
fun f2(lambda: A.(B) -> Int) =
	A().lambda(B())
	
fun main() {
	f1 { aa, bb -> aa.af() + bb.bf() }
	f2 { af() + it.bf() }
}

// f2() 처럼 함수의 파라미터로 확장 람다가 사용되는 경우가 더 일반적
```

- 확장 람다의 반환 타입이 Unit이면, 람다 본문이 만들어낸 결과는 무시된다.
(람다 본문의 마지막 식의 값을 무시한다는 뜻. return으로 Unit이 아닌 값을 반환하면 타입 오류가 발생한다.)
- 일반 람다를 파라미터로 받는 위치에 확장 람다를 전달할 수도 있다.
    
    (이때 두 람다의 파라미터 목록이 서로 호환되어야 한다.)
    
- :: 을 사용하면 확장 람다가 필요한 곳에 참조를 넘길 수 있다.
- 확장 함수에 대한 참조는 확장 람다와 타입이 같다.
    - Int::f2 는 Int.() → Int다.
- 확장 람다 대신 익명 함수 구문을 사용할 수 있다.
    
    ```kotlin
    fun exec(
    	arg1: Int, arg2: Int,
    	f: Int.(Int) -> Boolean
    ) = arg1.f(arg2)
    
    fun main() {
    	exec(10, 2, fun Int.(d: Int): Boolean {
    		return this % d == 0
    	}) eq true
    }
    ```
    
    - main()에서 exec() 호출은 익명 확장 함수를 익명 람다 위치에 써도 받아들여진다는 사실을 보여준다.
- 코틀린 표준 라이브러리에는 확장 람다와 함께 사용하는 함수가 많이 들어 있다.
    - StringBuilder는 toString()을 적용해 불변 String을 만들어낼 수 있는 가변 객체
    - 반대로 더 현대적인 buildString()은 확장 람다를 인자로 받는다.
    - buildString()은 자체적으로 StringBuilder 객체를 생성하고, 확장 람다를 생성한 StringBuilder 객체에 적용한 다음, toString()을 호출해 문자열을 얻는다.
    
    ```kotlin
    private fun messy(): String {
    	val built = StringBuilder()
    	built.append("ABCs: ")
    	('a'..'x').forEach { built.append(it) }
    	return built.toString()
    }
    
    private fun clean() = buildString {
    	append("ABCs: ")
    	('a'..'x').forEach { append(it) }
    }
    
    private fun cleaner() =
    	('a'..'x').joinToString("", "ABCs: ")
    
    fun main() {
    	messy() eq "ABCs: abcdefghijklmnopqrstuvwx"
    	messy() eq clean()
    	clean() eq cleaner()
    }
    ```
    
    - clean()에서 buildString()을 사용할 때는 append() 호출의 수신 객체를 직접 만들고 관리할 필요가 없다.
    → 모든 코드가 더 간결해진다.

- 확장 람다를 사용해 빌더 작성하기
    - 빌더 패턴의 장점
        - 여러 단계를 거쳐 객체를 생성한다. 객체 생성을 여러 단계로 나누면 객체 생성이 복잡할 때 유용할 수 있다.
        - 동일한 기본 생성 코드를 사용해 다양한 조합의 객체를 생성할 수 있다.
        - 공통 생성 코드와 특화한 코드를 분리할 수 있다. 이를 통해 객체들의 변종 유형에 따른 코드를 더 쉽게 작성할 수 있고, 작성한 코드를 더 쉽게 읽을 수 있다.
    - 확장 람다를 사용해 빌더를 구현하면 도메인 특화 언어(Domain Specific LAnguage, DSL)를 만들 수 있다는 장점도 있다.
        - DSL의 목표는 전문 프로그래머가 아닌 도메인 전문가에게 더 편하고 이해하기 쉬운 문법을 제공하는 것
        - 이를 통해 DSL을 둘러싼 주변 (프로그래밍) 언어의 아주 일부분만 알아도 (동시에 주변 언어가 제공하는 구조와 안전성이라는 이점을 살리면서) 도메인 전문가가 직접 잘 작동하는 해법을 만들어낼 수 있다.
    
    ```kotlin
    // 예시.. 여러 종류의 샌드위치를 조리하기 위한 재료와 절차를 담는 시스템
    // 조리법은 Recipe 클래스로 표현
    
    open class Recipe : ArrayList<RecipeUnit>()
    
    open class RecipeUnit {
    	override fun toString() =
    		"${this::class.simpleName}"
    }
    
    open class Operation : RecipeUnit()
    class Toast : Operation()
    class Grill : Operation()
    class Cut : Operation()
    
    open class Ingredient : RecipeUnit()
    class Bread : Ingredient()
    class PeanutButter : Ingredient()
    class GrapeJelly : Ingredient()
    class Ham : Ingredient()
    class Swiss : Ingredient()
    class Mustard : Ingredient()
    
    open class Sandwich : Recipe( ) {
    	fun action(op: Operation): Sandwich {
    		add(op)
    		return this
    	}
    	fun grill() = action(Grill())
    	fun toast() = action(Toast())
    	fun cut() = action(Cut())
    }
    
    fun sandwich(
    	fillings: Sandwich.() -> Unit
    ): Sandwich {
    	val sandwich = Sandwich()
    	sandwich.add(Bread())
    	sandwich.toast()
    	sandwich.fillings()
    	sandwich.cut()
    	return sandwich
    }
    
    fun main() {
    	val pbj = sandwich {
    		add(PeanutButter())
    		add(GrapeJelly())
    	}
    	val hamAndSiwss = sandwich {
    		add(Ham())
    		add(Swiss())
    		add(Mustard())
    		grill()
    	}
    	pbj eq "[Bread, Toast, PeanutButter, " +
    		"GrapeJelly, Cut]"
    	hamAndSiwss eq "[Bread, Toast, Ham, " +
    		"Swiss, Mustard, Grill, Cut]"
    }
    ```
    
    - fillings 확장 람다는 호출자가 Sandwich를 여러 가지 설정으로 준비할 수 있게 해준다.
    하지만 각각의 설정을 만들어내는 생성자에 대해 사용자가 알 필요는 없다.
    - 사용자는 sandwich()를 사용해 Sandwich를 만드는 문법만 이해함으로써, 중괄호 속에서 재료를 준비하고 필요한 절차를 기술하기만 하면 된다.

## 영역 함수

> 영역 함수는 객체의 이름을 사용하지 않아도 그 객체에 접근할 수 있는 임시 영역을 만들어주는 함수다.
> 
- 영역 함수는 오로지 코드를 더 간결하고 읽기 좋게 만들기 위해 존재한다.
- let(), run(), with(), apply(), also()라는 다섯 가지 영역 함수가 있다.
각각은 람다와 함께 쓰이며, 따로 임포트할 필요가 없다.
- 각 영역 함수는 문맥 객체를 it으로 다루는지 혹은 this로 다루는지와 각 함수가 어떤 값을 반환하는지에 따라 달라진다.
- with만 나머지 네 함수와 다른 호출 문법을 사용한다.

```kotlin
data class Tag(var n: Int = 0) {
	var s: String = ""
	fun increment() = ++n
}

fun main() {
	// let(): 객체를 'it'으로 접근하고
	// 람다의 마지막 식의 값을 반환한다
	Tag(1).let {
		it.s = "let: ${it.n}"
		it.increment()
	} eq 2
	// let()을 사용하면서 람다 인자에 이름을 붙인 경우다
	Tag(2).let { tag ->
		tag.s = "let: ${tag.n}"
		tag.increment()
	} eq 3
	// run(): 객체를 'this' 로 접근하고
	// 람다의 마지막 식의 값을 반환한다
	Tag(3).run {
		s = "run: $n" // 암시적 'this'
		increment()  // 암시적 'this'
	} eq 4
	// with() : 객체를 'this' 로 접근하고
	// 람다의 마지막 식을 반환한다
	with(Tag(4)) {
		s = "with: $n"
		increment()
	} eq 5
	// apply(): 객체를 'this' 로 접근하고
	// 변경된 객체를 다시 반환한다
	Tag(5).apply {
		s = "apply: $n"
		increment()
	} eq "Tag(n=6)"
	// also() : 객체를 'it' 으로 접근하고
	// 변경된 객체를 다시 반환한다
	Tag(6).also {
		it.s = "also: ${it.n}"
		it.increment()
	} eq "Tag(n=7) "
	// also() 에서도 람다의 인자에 이름을 붙일 수 있다
	Tag(7).also { tag->
		tag.s = "also: ${tag.n}"
		tag.increment()
	} eq "Tag(n=8)"
}
```

- 필요한 기능의 여러 조합을 만족시켜야 하므로 여러 가지 영역 함수가 존재
    - 문맥 객체를 this로 접근할 수 있는 영역 함수(run(), with(), apply())를 쓰면 영역 블록 안에서 가장 깔끔한 구문을 쓸 수 있다.
    - 문맥 객체를 it으로 접근할 수 있는 영역 함수(let(), also())에서는 람다 인자에 이름을 붙일 수 있다.
    - 결과를 만들어야 하는 경우, 람다의 마지막 식의 값을 돌려주는 영역 함수(let(), run(), whit())를 쓸 수 있다.
    - (객체에 대한 호출) 식을 연쇄적으로 사용해야 하는 경우, 변경한 객체를 돌려주는 영역 함수 (apply() 와 alsi())를 쓴다.
- run()은 확장 함수, with()는 일반 함수. 이 특징을 제외하면 두 함수는 같은 일을 한다.
- 수신 객체가 널이 될 수 있거나 연쇄 호출이 필요한 경우에는 run()을 사용

- 영역 함수의 특성

|  | this 문맥 객체 | it 문맥 객체 |
| --- | --- | --- |
| 마지막 식의 값을 돌려줌 | with, run | let |
| 수신 객체를 돌려줌 | apply | also |
- 안전한 접근 연산자 ?. 을 사용하면 영역 함수가 널이 될 수 있는 수신 객체에도 적용할 수 있다. 수신 객체가 null이 아닌 경우에만 영역 함수가 호출
- let(), run(), apply(), also() 에 대해 안전한 접근 연산자를 쓰면, 수신 객체가 null인 경우 전체 영역이 무시된다.
- 영역 함수는 연쇄 호출에서 널이 될 수 있는 타입과 함께 쓸 수 있다.
- 어떤 영역 함수도 use()와 비슷한 자원 해제를 제공하지는 못한다.
- use()는 let(), also()와 비슷하지만 람다에서 반환을 허용하지 않는다.
- 영역 함수를 사용하면서 자원 해제를 보장하고 싶다면 영역 함수를 use() 람다 안에서 쓰면 된다.
    
    ```kotlin
    Blob(7).use { it.run { show() } }
    ```
    

- 영역 함수는 인라인된다.
    - 일반적으로 람다를 인자로 전달하면, 람다 코드를 외부 객체에 넣기 때문에 일반 함수 호출에 비해 실행 시점의 부가 비용이 좀 더 발생한다.
    - 람다가 주는 이점(신뢰성과 코드 구조 개선)에 비하면 이런 부가 비용은 큰 문제가 되지 않는다.
    - JVM에는 부가 비용을 상쇄해줄 만한 여러가지 최적화 기능이 들어 있다.
    - 영역 함수를 inline으로 만들면 모든 실행 시점 부가 비용을 없앨 수 있다.
    → 영역 함수를 주저하지 않고 원하는 대로 쓸 수 있다.
    - 컴파일러는 inline 함수 호출을 보면 함수 호출 식을 함수의 본문으로 치환하며, 이때 함수의 모든 파라미터를 실제 제공된 인자로 바꿔준다.
    - 인라인 함수가 람다를 인자로 받으면, 컴파일러는 인라인 함수의 본문과 함께 람다 본문을 인라인 해준다.
    → 인라인 함수에 람다를 전달하는 경우 클래스나 객체가 추가로 생기지 않는다.
    (이런 동작은 인라인 함수에 람다 리터럴을 바로 전달하는 경우에만 성립한다. 람다를 변수에 담아서 전달하거나 다른 함수가 반환하는 람다를 전달하면 람다를 저장하기 위한 객체가 생긴다.)
    - 원하는 함수에는 언제나  inline을 적용할 수 있지만, 일반적으로 inline의 목적은 함수 인자로 전달되는 람다를 인라이닝하거나 실체화한 제네릭스를 정의하는 것이다.

## 제네릭스 만들기

> 제네릭스는 ‘나중에 지정할’ 타입에 대해 작동하는 코드를 말한다.
> 
- 코드가 ‘미리 정하지 않은 어떤 타입’ 에 대해 작동할 수 있다면, 더욱더 일반적인 코드가 될 수 있을 것이다. (여기서 말하는 일반적인 코드는 제약이 심하지 않은 코드)
→ 이런 ‘정해지지 않은 타입’이 바로 제네릭 타입 파라미터

- Any
    - Any는 코틀린 클래스 계층의 루트다.
    - 모든 코틀린 클래스는 Any를 상위 클래스로 가진다.
    - 미리 정해지지 않은 타입을 다루는 방법 중 하나로 Any 타입의 인자를 전달하는 방법이 있고, 때로 이를 제네릭스를 사용해야 하는 경우와 혼동하기도 한다.
    - Any를 사용하는 방법은 두 가지다.
        - 가장 간단한 첫 번째 접근 방법은 Any에 대해서만 연산을 수행하고, 디른 어느 타입도 요구하지 않는 경우다. (이런 경우는 극히 제한적)
        - Any에는 멤버 함수가 equals(), hashCode(), toString() 세 가지뿐
        - 두 번째로 확장 함수도 있지만, 이런 확장 함수는 Any 타입 객체에 대해 직접 연산을 적용할 수는 없다.
            - 예를 들어 Any.apply()는 함수 인자를 Any에 적용할 뿐이고, Any 타입 객체의 내부 연산을 직접 호출할 수는 없다.
    
    ```kotlin
    class Person {
    	fun speak() = "Hi!"
    }
    
    class Dog {
    	fun bark() = "Ruff!"
    }
    
    class Robot {
    	fun communicate() = "Beep!"
    }
    
    fun talk(speaker: Any) = when (speaker) {
    	is Person -> speaker.speak()
    	is Dog -> speaker.bark()
    	is Robot -> speaker.communicate()
    	else -> "Not a talker" // 또는 예외 발생
    }
    
    fun main() {
    	talk(Person()) eq "Hi!"
    	talk(Dog()) eq "Ruff!"
    	talk(Robot()) eq "Beep!"
    	talk(11) eq "Not a talker"
    }
    ```
    
    - when 식은 speaker의 타입을 찾고 적절한 함수를 호출한다.
    - 앞으로 talk()가 다른 타입의 값을 처리할 일이 전혀 없다고 예상한다면 이런 해법도 참을 만하다.
    - 그렇지 않다면 새로운 타입을 추가할 때마다 talk() 함수를 변경해야 하고, 추가한 타입에 맞춰 talk()를 변경하는 것을 놓치면 실행 시점의 정보에 의존해야 문제를 찾을 수 있다.

- 제네릭스 정의하기
    - 중복된 코드는 제네릭 함수나 타입으로 변환하는 것을 고려해볼 만하다.
    
    ```kotlin
    fun <T> gFunction(arg: T): T = arg
    
    class GClass(T>(val x: T) {
    	f un f() : T = x
    }
    
    class GMemberFunction {
    	fun <T> f(arg: T) : T = arg
    }
    
    interface GInterface<T> {
    	val x: T
    	fun f(): T
    }
    
    class GImplementation<T>(
    	override val x: T
    ) : GInterface<T> {
    	override fun f() : T = x
    }
    
    class ConcreteImplementation
    	: GInterface<String> {
    	override val x: String
    		get() = "x"
    	override fun f () = "f() "
    }
    
    fun basicGenerics() {
    	gFunction("Yellow")
    	gFunction(1)
    	gFunction(Dog()).bark() // [1]
    	gFunction<Dog>(Dog()).bark()
    	GClass("Cyan").f()
    	GClass(11).f()
    	GClass(Dog()).f().bark() // [2]
    	GClass<Dog>(Dog()).f().bark()
    	GMemberFunticon().f("Amber")
    	GMemberFunticon().f(111)
    	GMemberFunticon ().f(Dog()).bark() // [3]
    	GMemberFunticon().f<Dog>(Dog()).bark()
    	GImplementation("Cyan").f()
    	GImplementation(11).f()
    	GImplementation(Dog()).f().bark()
    	ConcreteImplementation().f()
    	ConcreteImplementation().x
    }
    ```
    
    - basicGenerics()는 각 제네릭 클래스나 함수가 여러 타입을 처리하는 모습을 보여준다.
        - gFunction()은 타입 파라미터를 T로 받고, 결과를 T로 돌려준다.
        - GClass는 T를 저장한다. GClass의 멤버 함수인 f()는 T를 반환한다.
        - GMemberFunction은 클래스 안에서 멤버 함수를 파라미터화한다. 단 전체 클래스를 파라미터화하지는 않는다.
        - GInterface처럼 interface가 제네릭 파라미터를 받게 할 수도 있다. GInterface를 구현한 클래스는 GImplementation처럼 타입 파라미터를 재정의하거나, ConcreteImplementation처럼 타입 파라미터에 구체적인 타입 인자를 제공해야 한다.
    - [1], [2], [3] 에서 결과에 대해 결과 타입이 Dog이라는 타입으로 결정되기 때문에 bark()를 호출할 수 있다.
    - [1], [2], [3] 의 경우 타입 추론에 의해 타입 파라미터의 구체적 타입이 결정된다. 하지만 제네릭이나 제네릭을 호출하는 코드가 너무 복잡해서 컴파일러가 타입을 추론하지 못할 수도 있다.
    → 이런 경우에는 [1], [2], [3] 의 바로 다음 줄에서 한 것처럼 직접 구체적인 타입을 지정해야 한다.

- 타입 정보 보존하기
    - 제네릭 클래스나 제네릭 함수의 내부 코드는 T타입에 대해 알 수 없다.
    → 이를 타입 소거라고 한다.
    - 제네릭 코드를 사용하는 일반적인 예로 다른 객체를 담는 컨테이너를 들 수 있다.
        
        ```kotlin
        // Car 타입의 객체를 하나 담을 수 있는 CarCrate라는 단순한 컬렉션
        
        class Car {
        	override fun toString() = "Car"
        }
        class CarCrate(private var c: Car) {
        	fun put(car: Car) { c = car }
        	fun get(): Car = c
        }
        fun main( ) {
        	val cc = CarCrate(Car())
        	val car: Car = cc.get()
        	car eq "Car"
        }
        
        //cc.get()을 호출하면 Car 타입 결과값이 나온다. 
        // 이 도구를 Car뿐 아니라 다른 타입에 대해서도 활용하기 위해 Crate<T>로 일반화한다.
        
        open class Crate<T>(private var contents: T) {
        	fun put(item: T) { contents = item }
        	fun get(): T = contents
        }
        
        fun main() {
        	val cc = Crate(Car())
        	val car: Car = cc.get()
        	car eq "Car"
        }
        ```
        
        - Crate<T>는 T 타입의 값만 Crate에 put()으로 넣을 수 있도록 보장한다.

- 타입 파라미터 제약
    - 타입 파라미터 제약은 제네릭 타입 인자가 다른 클래스를 상속해야 한다고 지정한다.
    - <T: Base>는 T가 Base 타입이나 Base에서 파생된 타입이어야 한다는 뜻이다.
    - 타입 파라미터 제약으로 Base를 지정하는 경우와 Base를 상속하는 제네릭하지 않은 타입(일반 타입)의 차이
        
        ```kotlin
        // 여러 가지 물건과 각각을 처리하는 방법을 모델링하는 타입 계층
        
        interface Disposable {
        	val name: String
        	fun action(): String
        }
        
        class Compost (override val name: String) :
        	Disposable {
        	override fun action() = "Add to composter"
        }
        
        interface Transport : Disposable
        
        class Donation(override val name: String) :
        	Transport {
        	override fun action() = "Call for pickup"
        }
        
        class Recyclable(override val name: String) :
        	Transport {
        	override fun action() = "Put in bin"
        }
        
        class Landfill(override val name: String)
        	Transport {
        	override fun action() = "Put in dumpster"
        }
        
        val items = listOf(
        	Compost("Orange Peel"),
        	Compost("Apple Core"),
        	Donation("Couch"),
        	Donation("Clothing"),
        	Recyclable("Plastic"),
        	Recyclable("Metal"),
        	Recyclable("Cardboard"),
        	Landfill("Trash"),
        )
        val recyclables =
        	items.filterisInstance<Recyclable>()
        ```
        
        - 제약을 사용하면 제네릭 함수 안에서 제약이 이뤄진 타입의 프로퍼티와 함수에 접근할 수 있다.
    - 왜 일반 다형성 대신 제약을 써야 할까?
        
        → 답은 반환 타입에 있다. 다형성을 쓰는 경우 반환 타입을 기반 타입으로 업캐스트해 반환해야 하지만, 제네릭스를 쓰면 정확한 타입을 지정할 수 있다.
        
        ```kotlin
        private val rnd = Random(47)
        
        fun List<Disposable>.aRandom(): Disposable =
        	this[rnd.nextInt(size)]
        	
        fun <T: Disposable> List<T>.bRandom(): T =
        	this[rnd.nextint(size)]
        	
        fun <T> List<T>.cRandom(): T =
        	this[rnd.nextInt(size)]
        	
        fun sameReturnType() {
        	val a: Disposable = recyclables.aRandom()
        	val b: Recyclable = recyclables.bRandom()
        	val c: Recyclable = recyclables.cRandom()
        }
        ```
        
        - 제네릭스를 쓰지 않은 aRandom()은 기반 클래스인 Disposable만 만들어낼 수 있다. 반면 main()에서 bRandom과 cRandom은 Recyclable을 만들어낸다.
        - bRandom()은 Disposable의 멤버를 전혀 사용하지 않아서 T에 걸린 :Disposable 제약은 의미가 없고, 그렇기 때문에 제약을 걸지 않은 cRandom()과 같아진다.
    - 타입 파라미터 제약이 필요한 경우는 다음 두 가지가 모두 필요할 때뿐이다.
        - 타입 파라미터 안에 선언된 함수나 프로퍼티에 접근해야 한다.
        - 결과를 반환할 때 타입을 유지해야 한다.

- 타입 소거
    - 최조의 자바에는 제네릭이 포함되어 있지 않았고, 몇 년 후 제네릭스가 추가됐을 때는 이미 엄청나게 많은 코드가 제네릭스 없이 작성된 상태였다.
    - 기존 코드를 깨지 않는 절충점으로 제네릭 타입은 컴파일 시점에만 사용할 수 있고 런타임 바이트코드에서는 제네릭 타입 정보가 보존되지 않는다
    (제네릭 타입의 파라미터 타입이 지워져버린다.)
    → 자바와의 호환성 때문에 이런 타입 소거는 코틀린에도 영향을 끼친다.
    - 코틀린 설계자들이 자바의 결정을 따라 타입 소거를 사용하게 된 이유
        - 자바 호환성을 유지한다.
        - 타입 정보를 유지하려면 부가 비용이 든다. 제네릭 타입 정보를 저장하면 제네릭 List나 Map이 차지하는 메모리가 상당히 늘어난다.

- 함수의 타입 인자에 대한 실체화
    - 함수 인자의 타입 정보를 보존하려면 reified 키워드 추가
    
    ```kotlin
    // 함수 a()가 작업을 수행하기 위해서 클래스 정보가 필요하다고 가정
    
    import kotlin.reflect.KClass // 코틀린 클래스를 표현하는 클래스
    
    fun <T: Any> a(kClass: KClass<T>) {
    	// kClass<T>를 사용한다
    }
    ```
    
    - a()를 두 번째 제네릭 함수인 b()에서 호출한다면 제네릭 인자의 타입 정보를 전달하고 싶을 것이다.
    
    ```kotlin
    // 타입 소거로 인해 컴파일되지 않음
    // fun <T: Any> b() = a(T::class)
    ```
    
    - 위 코드를 실행할 때는 타입 정보 T가 지워지기 때문에 b()가 컴파일되지 않는다.
    - 함수 본문에서 함수의 제네릭 타입 파라미터의 클래스를 사용할 수는 없다.
    - 자바에서는 타입 정보를 전달하는 것으로 해결한다.
    
    ```kotlin
    fun <T: Any> c(kClass: KClass<T>) = a(kClass)
    
    class K
    
    val kc = c(K::class)
    ```
    
    - 컴파일러가 이미 T의 타입을 알고 조용히 타입을 넘겨줄 수 있는데, 이런 식으로 명시적으로 타입 정보를 전달하는 것은 불필요한 중복
    → 실제 refied 키워드가 해주는 일이 바로 이런 일
    
    ```kotlin
    inline fun <reified T: Any> d() = a(T::class)
    
    val kd = d<K>()
    ```
    
    - d()는 c()와 같은 효과를 내지만, 클래스 참조를 인자로 요구할 필요가 없다.
    - reified는 reified가 붙은 타입 인자의 타입 정보를 유지시키라고 컴파일러에 명령
    → 이제 실행 시점에도 타입 정보를 사용할 수 있기 때문에 함수 본문 안에서 이를 쓸 수 있다.
    
    ```kotlin
    inline fun <reified T> check(t: Any) = t is T
    // fun <T> check1(t: Any) = t is T // [1]
    
    fun main() {
    	check<String>("1") eq true
    	check<Int>("1") eq false
    }
    
    // [1] reified 가 없으면 타입 정보가 지워지므로 실행 시점에 어떤 객체가 T의 인스턴스인지
    // 검사할 수 없다.
    ```
    

- 타입 변성
    - T와  U 사이에 상속 관계가 있을 때 Container<T> 라는 제네릭 타입 객체를 Container<U>라는 제네릭 타입 컨테이너 객체에 대입하고 싶은 경우, Container 타입을 어떤 식으로 쓸지에 따라 Container의 타입 파라미터에 in이나 out 변성 애너테이션을 붙여서 타입 파라미터를 적용한 Container 타입의 상하위 타입 관계를 제한해야 한다.
        
        ```kotlin
        // 세 가지 버전의 Box 컨테이너
        
        class Box<T>(private var contents: T) {
        	fun put(item: T) { contents = item }
        	fun get() : T = contents
        }
        
        class InBox<in T>(private var contents: T) {
        	fun put(item: T) { contents = item }
        }
        
        class OutBox<out T>(private var contents: T) {
        	fun get() : T = contents
        }
        ```
        
        - in T 는 이 클래스의 멤버 함수가 T 타입 값을 인자로만 받고, T 타입 값을 반환하지는 않는다는 뜻
        → 즉, T 객체를 InBox 안으로(into) 집어넣을 수는 있어도, InBox에서 T 객체가 나올 수는 없다.
        - out T 는 이 클래스의 멤버 함수가 T 타입 값을 반환하기만 하고 T 타입 값을 인자로 받지는 않는다는 뜻
        → 즉, T 객체가 OutBox 밖으로(out) 나올 수는 있어도 OutBox 안으로 T 객체를 넣을 수는 없다.
    - 변성
 
      
  ![image](https://github.com/user-attachments/assets/7dca5d95-0364-4538-bb2c-0e00050b0302)

   
    - Box<T>는 무공변(invariant)
    - 즉, Box<Cat>과 Box<Pet> 사이에 아무런 하위 타입 관계가 없다. 따라서 둘 중 어느 쪽도 반대쪽에 대입될 수 없다.
    - OutBox<out T>는 공변(convariant)
    - OutBox<Cat>을 OutBox<Pet>으로 업캐스트하는 방향이 Cat을 Pet으로 업캐스트하는 방향과 같은 방향으로 변한다.
    - InBox<in T>는 반공변(contravariant)
    - 즉, InBox<Pet>이 InBox<Cat>의 하위 타입이다.
    - InBox<Pet>을 InBox<Cat>으로 업캐스트하는 방향이 Cat을 Pet으로 업캐스트하는 방향과 반대 방향으로 변한다.

- 코틀린 표준 라이브러리의 읽기 전용 List는 공변이다.
- 따라서 List<Cat>을 List<Pet>에 대입할 수 있다.
- 반면 MutableList는 읽기 전용 리스트의 기능에 add()를 추가하기 때문에 무공변이다.
- 함수는 공변적인 반환 타입을 가진다.
→ 이 말은 오버라이드하는 함수가 오버라이드 대상 함수보다 더 구체적인 반환 타입을 돌려줘도 된다는 뜻

## 연산자 오버로딩

> 컴퓨터 프로그래밍에서 오버로딩은 ‘이미 존재하는 어떤 대상에 추가로 의미를 더하는 것’을 뜻한다.
> 
- 연산자 오버로딩(operator overloading)을 사용하면, 새로 만든 타입에 대해 + 같은 연산자에 의미를 부여하거나 기존 타입에 대해 작용하는 연산자에 추가로 의미를 부여할 수 있다.
- 연산자를 오버로딩하려면 fun 앞에 operator 키워드를 붙여야 한다.
- 함수 이름으로는 연산자에 따라 미리 정해진 특별한 이름만 쓸 수 있다.
(예를 들어 + 연산자에 대한 특별 함수는 plus()이다.)
    
    ```kotlin
    data class Num(val n: Int)
    
    operator fun Num.plus(rval: Num) =
    	Num(n + rval.n)
    	
    fun main() {
    	Num(4) + Num(5) eq Num(9)
    	Bum(4).plus(Num(5)) eq Num(9)
    }
    ```
    
- 두 피연산자 사이에 사용하기 위해 일반(연산자가 아닌) 함수를 정의하고 싶다면 infix 키워드를 사용하면 된다.
- 연산자들은 대부분 이미 infix이므로 infix를 붙이지 않아도 된다.
- 어떤 연산자를 확장 함수로 정의하면 클래스의 private 멤버를 볼 수 없지만, 멤버 함수로 정의하면 private 멤버에 접근할 수 있다.

- 동등성
    - ==(동등성) 과 ! = (비동동성)은 equals() 멤버 함수를 호출한다.
    - data 클래스는 자동으로 저장된 모든 필드를 서로 비교하는 equals()를 오버라이드해준다.
    - equals()는 확장 함수로 정의할 수 없는 유일한 연산자. 이 연산자는 반드시 멤버 함수로 오버라이드되어야 한다.
    → 이 연산자를 오버라이드할 때는 반드시 비교 대상 타입을 선택해야 한다.
    - ===(삼중 등호) 기호는 참조 동등성을 검사한다.
    - equals()를 오버라이드할 때 항상 hashCode()도 오버라이드해야 한다.
    기본적인 규칙은 두 객체가 같다면 그 두 객체의 hashCode()도 같은 값을 내놓아야 한다는 것이다.
    → 이 규칙을 지키지 않으면 Map 이나 Set 같은 표준 데이터 구조가 제대로 작동하지 않는다.

- 산술 연산자
    - 연산자 우선순위는 고정되어 있고 내장 타입이나 커스텀 타입에서 모두 동일하다.
    - 때로 산술 연산자와 프로그래밍 연산자를 섞어 쓰면 결과가 분명하지 않아 보이는 경우도 있다.
        
        ```kotlin
        fun main() {
        	val x: Int? = 1
        	val y: Int = 2
        	val sum = x ?: 0 + y
        	sum eq 1
        	(x ?: 0) + y eq 3 // [1]
        	x ?: (0 + y) eq 1 // [2]
        }
        ```
        
        - sum 초기화 식에서 + 가 엘비스 연산자 ?: 보다 우선순위가 높기 때문에 결과는 1 ?: (0+2) 의 결과인 1이다.
        - 우선순위를 분명히 알 수 없는 여러 연산자를 섞어 쓸 때는 [1] 이나 [2] 처럼 괄호를 사용하는 것이 좋다.

- 비교 연산자
    - compareTo()를 정의하면 모든 비교 연산자(<. >. ≤, ≥) 를 쓸 수 있다.
    - compareTo() 는 다음을 알려주는 Int를 반환해야 한다.
        - 두 피연산자가 동등한 경우 0을 반환한다.
        - 첫 번째 피연산자(수신 객체)가 두 번째 피연산자(함수의 인자) 보다 크면 양수를 반환한다.
        - 첫 번째 피연산자가 두 번째 피연산자보다 작으면 음수를 반환한다.

- 범위와 컨테이너
    - rangeTo()는 범위를 생성하는 .. 연산자를 오버로드하고, contains()는 값이 범위 안에 들어가는지 여부를 알려주는 in 연산을 오버로드한다.
    
    ```kotlin
    data class R(val r: IntRange) { // Range
    	override fun toString() = "R($r)"
    }
    
    operator fun E.rangeTo(e: E) = R(v..e.v)
    
    operator fun R.contains(e: E): Boolean =
    	e.v in r
    	
    fun main() {
    	val a = E(2)
    	val b = E(3)
    	val r = a..b // a.rangeTo(b)
    	(a in r) eq true // r.contains(a)
    	(a !in r) eq false // !r.contains(a)
    	r eq R(2..3)
    }
    ```
    

- 컨테이너 원소 접근
    - get()과 set()은 각괄호([])를 사용해 컨테이너의 원소를 읽고 쓰는 연산을 정의한다.
    
    ```kotlin
    data class C(val c: Mutablelist<Int>) {
    	override fun toString() = "C($c) "
    }
    
    operator fun C.contains(e: E) = e.v in c
    
    operator fun C.get(i: Int) : E = E(c[i])
    
    operator fun C.set(i : Int, e: E) {
    	c[i] = e.v
    }
    
    fun main() {
    	val c = C(mutableListOf(2, 3))
    	(E(2) in c) eq true // c.contains(E(2))
    	(E(4) in c) eq false // c.contains(E(4))
    	c[1] eq E(3) // c.get(1)
    	c[1] = E(4) // c.set(2, E(4))
    	c eq C(mutableListOf(2, 4))
    }
    ```
    
    - 함수나 클래스를 사용(호출)한 부분에서 함수 정의로 바로 이동하는 기능은 연산자에도 적용할 수 있다.
        - 예를 들어 커서를 ..에 놓고 정의로 이동하면 이 연산자에 의해 어떤 rangeTo() 함수가 실행될지 그 정의를 볼 수 있다.

- 호출 연산자
    - 객체 참조 뒤에 괄호를 넣으면 invoke()를 호출한다.
    - invoke() 연산자는 객체가 함수처럼 작동하게 만든다.
    - invoke()가 받을 수 있는 파라미터 개수는 원하는 대로 지정할 수 있다.
    
    ```kotlin
    class Func {
    	operator fun invoke() = "invoke() "
    	operator fun invoke(i: Int) = "invoke($i)"
    	operator fun invoke(i: Int, j: String) =
    		"invoke($i, $j)"
    	operator fun invoke(
    		i: Int, j: String, k: Double
    	) = "invoke($i, $j, $k) "
    }
    
    fun main() {
    	val f = Func()
    	f() eq "invoke()"
    	f(22) eq "invoke(22)"
    	f(22, "Hi") eq "invoke(22, Hi)"
    	f(22, "Three", 3.1416) eq
    		"invoke(22, Three, 3.1416)"
    }
    ```
    
    - invoke()가 vararg를 파라미터로 받으면, 같은 타입에 속한 파라미터를 임의의 개수만큼 받을 수도 있다.
    - invoke() 를 확장 함수로 정의할 수 도 있다.

- 역작은따옴표로 감싼 함수 이름
    - 역작은따옴표로 정의한 이름은 자바 코드와의 상호 작용을 더 쉽게 해준다.
    (하지만 이 기능을 활용해 이해할 수 없는 코드를 쉽게 만들어낼 수 있으니 조심해야 한다.)
    
    ```kotlin
    infix fun String. `#!%`(s: String) =
    	"$this Rowzafrazaca $s"
    	
    fun main( ) {
    	"howdy" `#!%` "Ma'am!" eq
    		"howdy Rowzafrazaca Ma'am!"
    }
    ```
    

## 연산자 사용하기 6

- 가변 컬렉션에 +=을 호출하면 컬렉션 내용을 변경하지만, +를 호출하면 예전 원소에 새 원소가 추가된 새로운 컬렉션을 반환한다.

```kotlin
fun main() {
	var list = listOf(1, 2)
	val initial = list
	list += 3
	list eq "[1, 2, 3]"
	list = list.plus(4)
	list eq "[1, 2, 3, 4]"
	initial eq "[1, 2]"
}
```

- 마지막 줄은 initial 컬렉션이 변경되지 않고 그대로 있는 모습을 보여준다,
→ 원소를 추가할 때마다 새 컬렉션을 만드는 걸 원하지 않았을 것
- var list 를 val list 로 바꾸면( 읽기 전용 리스트에 대해) 이런 문제가 발생하지 않는다.
+= 호출이 컴파일되지 못하기 때문
    
    → 그렇기 때문에 디폴트로 val을 사용해야 하는 이유
    
- 구조 분해 연산자
    - 보통 직접 정의할 일이 거의 없는 또 다른 연산자로, 구조 분해에 사용되는 componentN() 함수를 말한다.
    
    ```kotlin
    // 구조 분해 대입을 위해 코틀린이 조용히 component1() 과 component2() 함수를 호춢
    
    class Duo(val x: Int, val y: Int) {
    	operator fun component1(): Int {
    		trace("component1()")
    		retrun x
    	}
    	operator fun component2(): Int {
    		trace("component2()")
    		retrun y
    	}
    }
    
    fun main() {
    	val (a, b) = Duo(1, 2)
    	a eq 1
    	b eq 2
    	trace eq "component1() component2()"
    }
    ```
    
    - 같은 접근 방법을 Map에도 적용 가능
    - Map 의 Entry 타입에는 component1() 과 component2() 멤버 함수가 정의되어 있다.
    - 코틀린은 data 클래스의 각 프로퍼티에 대해(data 클래스 생성자에 프로퍼티가 나타난 순서대로) componentN()을 생성해준다.

## 프로퍼티 위임

> 프로퍼티는 접근자 로직을 위임할 수 있다.
> 
- by 키워드를 사용하면 프로퍼티를 위임과 연결할 수 있다.

<aside>
💡 val (또는 var) 프로퍼티이름 by 위임객체이름

</aside>

- 프로퍼티가 val(읽기 전용)인 경우 위임 객체의 클래스에는 getValue() 함수 정의가 있어야 하며, 프로퍼티가 var(읽고 쓸 수 있음)인 경우 위임 객체의 클래스에는 getValue()와 setValue() 함수 정의가 있어야 한다.

```kotlin
// 읽기 전용인 경우

class Readable(val i: Int) {
	val value : String by BasicRead()
}

class BasicRead {
	operator fun getValue(
		r: Readable,
		property: KProperty<*>
	) = "getValue: ${r.i}"
}

fun main() {
	val x = Readable(11)
	val y = Readable(17)
	x.value eq "getValue: 11"
	y.value eq "getValue: 17"
}

// Readable의 Value는 BasicRead 객체에 의해 위임된다.
// getValue()는 Readable에 대한 접근을 가능하게 하는 Readable 파라미터를 얻는다.
// 프로퍼티 뒤에 by라고 지정하면 BasicRead 객체를 by 앞의 프로퍼티와 연결한다.
// 이때, BasicRead의 getValue()는 Readable의 i에 접근할 수 있다.
// getValue()가 String을 반환하기 때문에 value 프로퍼티의 타입도 String이어야 한다.
// getValue()의 두 번째 파라미터는 KProperty라는 특별한 타입이며, 이 타입의 객체는 위임 프로퍼티에 대한 리플렉션 정보를 제공한다.
// 위임 프로퍼티가 var이면 위임 객체가 읽기와 쓰기를 모두 처리해야 하므로, 위임 클래스에는 getValue()와 SetValue()가 모두 있어야 한다.
```

```kotlin
// 읽고 쓸 수 있음

class ReadWriteable(var i: Int) {
	var msg = ""
	var value: String by BasicReadWrite()
}

class BasicReadWrite {
	operator fun getValue(
		rw: ReadWriteable,
		property: KProperty<*>
	) = "getValue: ${rw. i}"
	operator fun setValue(
		rw: ReadWriteable,
		property: KProperty<*>,
		s: String
	) {
		rw.i = s.tolntOrNull() ?: 0
		rw.msg = "setValue to ${rw.i}"
	}
}
	
fun main() {
	val x = ReadWriteable(11)
	x.value eq "getValue: 11"
	x.value = "99"
	x.msg eq "setValue to 99"
	x.value eq "getValue: 99"
}

// setValue() 함수 맨 앞의 두 가지 파라미터는 getValue()와 똑같으며, 세 번째 값은
// 프로퍼티 초기화 식에서 = 오른쪽에 있던 식의 결과값으로, 프로퍼티에 설정하려는 값
// getValue()의 반환 타입과 setValue()의 세 번째 파라미터 값의 타입은 해당 위임 객체가 적용된 프로퍼티의 타입과 일치해야 한다.
// 이 예제에서는 이 타입이 ReadWriteable의 value 프로퍼티 타입인 String이어야 한다.
```

- 위 두 예제 코드 모두 어떤 인터페이스도 구현할 필요가 없다.
- 클래스는 단순히 필요한 함수 이름과 시그니처만 만족하면 위임 역할을 수행할 수 있다.
- 원한다면 인터페이스를 상속할 수도 있다.
- 정리하면, 위임 클래스는 다음 두 함수를 모두 포함하거나 어느 하나를 포함할 수 있다. 위임 프로퍼티에 접근하면 읽기와 쓰기에 따라 다음에 열거한 두 함수가 호출된다.
    - 읽기의 경우: `operator fun getValue(thisRef: T, property: KProperty<*>): V`
    - 쓰기의 경우: `operator fun setValue(thisRef: T, property: KProperty<*>, value: V)`
- 위임 프로퍼티가 val 이면 첫 번째 함수만 필요하며, ReadOnlyProperty 인터페이스(예제 코드 안적었음) 와 SAM 변환을 통해 이를 구현할 수도 있다.
    - thisRef: T는 위임자(다른 객체에 처리를 맡기는) 개체를 가리킨다. 여기서 T는 위임자 개체의 클래스. 이 함수에서 thisRef를 쓰지 않고 싶다면 Any?를 사용해 위임자 객체의 내부를 보기 어렵게 만들 수 도 있다.
    - property: KProperty<*>는 위임 프로퍼티에 대한 정보를 제공한다. 가장 일반적으로 사용하는 정보는 name. name은 위임 프로퍼티의 필드 이름을 돌려준다.
    - value는 setValue()로 위임 프로퍼티에 저장할 값이다. V는 위임 프로퍼티의 타입이다.
- 위임자 객체의 private 멤버에 대한 접근을 가능하게 하려면 위임 클래스를 내포시켜야 한다.
    
    ```kotlin
    class Person(
    	private val first: String,
    	private val last: String
    ) {
    	val name by // SAM 변환
    	ReadOnlyProperty<Person, String) { _, _ ->
    		"$first $last"
    	}
    }
    
    fun main() {
    	val alien = Person("Floopy", "Noopers")
    	alien.name eq "Floopy Noopers"
    }
    ```
    
- 위임자 객체의 멤버에 대한 접근이 충분하다면 getValue()와 setValue()를 확장 함수로 만들 수 도 있다.

- 지금까지 살펴본 대부분의 예에서 getValue()와 setValue()의 첫 번째 파라미터의 타입은 구체적인 타입이었다.
→ 이런 식으로 정의한 위임은 그 구체적인 타입에 얽매이게 된다.
- 때로는 첫 번째 타입을 Any?로 지정해 무시함으로써 더 일반적인 목적의 위임을 만들 수도 있다.

## 프로퍼티 위임 도구

- Map은 위임 프로퍼티의 위임 객체로 쓰일 수 있도록 미리 설정된 코틀린 표준 라이브러리 타입 중 하나
    
    ```kotlin
    class Driver(
    	map: MutableMap<String, Any?>
    ) {
    	var name: String by map
    	var age: Int by map
    	var id: String by map
    	var available: Boolean by map
    	var coord: Pair <Double, Double> by map
    }
    
    fun main() {
    	val info = mutableMapOf<String, Any?>(
    		"name" to "BrunoFiat" ,
    		"age" to 22,
    		"id" to "X97C111",
    		"available" to false,
    		"coord" to Pair(111.93, 1231.12)
    	)
    	val driver = Driver(info)
    	driver.available eq false
    	driver.available = true
    	info eq "{name=Bruno Fiat, age=22, " +
    		"id=X97C111, available= true, " +
    		"coord=(111.93, 1231.12)}"
    }
    ```
    
    - driver.available = true라고 설정할 때 원본 맵 info가 변경된다.
    - 이런 동작이 작동하는 이유는 코틀린 표준 라이브러리에서 Map의 확장 함수로 프로퍼티 위임을 가능하게 해주는 getValue()와 setValue()를 제공하기 때문
- Delegates.observable()은 가변 프로퍼티의 값이 변경되는지 관찰한다.
    
    ```kotlin
    class Team {
    	var msg =
    	var captain : String by observable("<O>") {
    		prop, old, new ->
    		msg += "${prop.name} $old to $new"
    	}
    }
    
    fun main() {
    	val team = Team()
    	team.captain = "Adam"
    	team.captain = "Amanda"
    	team.msg eq "captain <0> to Adam " +
    		"captain Adam to Amanda"
    }
    ```
    
    - observable()은 인자를 두 개 받는다.
        - 프로퍼티의 최초 값: 여기서는 “<0>”
        - 프로퍼티가 변경될 때 실행할 동작을 지정하는 함수: 여기서는 람다를 사용한다. 함수의 인자는 변경 중인 프로퍼티, 그 프로퍼티의 현재 값, 그 프로퍼티에 저장될 새로운 값이다.
- Delegates.vetoable()을 사용하면 새 프로퍼티 값이 어떤 술어를 만족하지 않을 때 프로퍼티가 변경되는 것을 방지할 수 있다.
    
    ```kotlin
    fun aName(
    	property: KProperty<*>,
    	old: String,
    	new: String
    ) = if (new.startsWith("A")) {
    	trace("$old -> $new")
    	true
    } else {
    	trace("Name must start with 'A'")
    	false
    }
    
    interface Captain {
    	var captain: String
    }
    
    class TeamWithTraditions : Captain {
    	override var captain: String
    		by Delegates.vetoable("Adam", ::aName)
    }
    
    class TeamWithTraditions2 : Captain {
    	override var captain: String
    		by Delegates.vetoable("Adam") {
    			_, old, new ->
    			if (new.startsWith("A")) {
    				trace("$old -> $new")
    				true
    			} else {
    				trace("Name must start with 'A'")
    				false
    			}
    		}
    }
    
    fun main() {
    	listOf(
    		TeamWithTraditions(),
    		TeamWithTraditions2()
    	).forEach {
    		it.captain = "Amanda"
    		it.captain = "Bill"
    		it.captain eq "Amanda"
    	}
    	trace eq """
    		Adam -> Amanda
    		Name must start with 'A'
    		Adam -> Amanda
    		Name must start with 'A'
    	"""
    }
    ```
    
    - Delegates.vetoable()은 인자를 두 개 받는다.
        - 첫 번째 파라미터는 프로퍼티의 초깃값
        - 두 번째 파라미터는 onChange() 함수
    - 이 함수는 변경해도 되면 true, 변경을 막아야하면 false를 반환

- notNull()
    - 이 함수는 읽기 전에 꼭 초기화해야 하는 프로퍼티를 정의한다.

## 지연 계산 초기화

> 프로퍼티를 초기화하는 두 가지 방법
1. 프로퍼티를 정의하는 시점이나 생성자 안에서 초깃값을 저장
2. 프로퍼티에 접근할 때마다 값을 계산하는 커스텀 게터를 정의
> 
- 프로퍼티를 초기화하는 세 번째 경우
    - 초깃값을 계산하는 비용이 많이 들지만 프로퍼티를 선언하는 시점에 즉시 필요하지 않거나 아예 전혀 필요하지 않을 수도 있는 경우
    - 예시
        - 복잡하고 시간이 오래 걸리는 계산
        - 네트워크 요청
        - 데이터베이스 접근
    - 위와 같은 프로퍼티를 생성 시점에 즉시 초기화하는 경우
        - 애플리케이션 초기 시작 시간이 길어질 수 있다.
        - 전혀 사용하지 않거나 나중에 계산해도 될 프로퍼티 값을 계산하기 위해 불필요한 작업을 수행할 수 있다.
- 지연 계산 프로퍼티는 생성 시점이 아니라 처음 사용할 때 초기화된다.
    
    ```kotlin
    val lazyProperty by lazy { 초기화코드 }
    ```
    

```kotlin
// 지연 계산 프로퍼티의 초기화가 어떻게 작동하는지에 대한 예제
// Int 타입인 지연 계산 프로퍼티의 동작을 lazy() 위임 없이 구현

class LazyInt(val init: () -> Int) {
	private var helper: Int? = null
	val value : Int
		get() {
			if (helper = null)
				helper = init()
			return helper!!
		}
}

fun main() {
	val later = LazyInt {
		trace("Initializing 'later'")
		5
	}
	trace("First 'value' access:")
	trace(later.value)
	trace("Second 'value' access:")
	trace(later.value)
	trace eq """
		First 'value' access:
		Initializing 'later'
		5
		Second 'value' access:
		5
	"""
}
```

- value 프로퍼티는 값을 저장하지 않지만, helper 프로퍼티에 저장된 값을 읽는 게터가 있다.
- 위 코드는 코틀린이 by lazy()에 대해 생성하는 코드와 비슷하다.

```kotlin
// 프로퍼티를 초기화하는 세 가지 방법 비교
// 정의 시점, 게터, 지연 계산

fun compute(i: Int): Int {
	trace("Compute $i")
	return i
}

object Properties {
	val atDefinition = compute(1)
	val getter
		get() = compute(2)
	val lazylnit by lazy { compute(3) }
	val never by lazy { compute(4) }
}

fun main() {
	listOf(
		Properties::atDefinition,
		Properties::getter,
		Properties::lazyInit
	).forEach {
		trace("${it.name}:")
		trace("${it.get()}")
		trace("${it.get()}")
	}
	trace eq """
		Comput e 1
		at Defi niti on:
		1
		1
		getter:
		Compute 2
		2
		Compute 2
		2
		lazyInit:
		Compute 3
		3
		3
	"""
}
```

- atDefinition은 Properties의 인스턴스를 생성할 때 초기화된다.
- ‘Compute 1’은 ‘atDefinition’ 보다 앞에 나타난다. 이는 초기화가 프로퍼티 접근 이전에 발생한다는 사실을 보여줌
- getter 프로퍼티에 접근할 때마다 getter가 계산된다. 그래서 ‘Compute 2’ 가 프로퍼티에 접근할 때마다 한 번씩, 모두 두 번 나타난다.
- lazyInit의 초기화 값은 never 프로퍼티에 처음 접근할 때 한 번만 계산된다. 프로퍼티에 접근하지 않으면 초기화가 일어나지 않는다. ‘Compute 4’ 가 트레이스에 전혀 나오지 않는다는 사실을 확인하라.

## 늦은 초기화

> 때로는 by lazy()를 사용하지 않고 별도의 멤버 함수에서 클래스의 인스턴스가 생성된 다음에 프로퍼티를 초기화하고 싶은 경우가 있다.
> 
- 특별한 함수 안에서 초기화를 해야 할 때 lateinit을 사용한다.
- 클래스 본문과 최상위 영역이나 지역에 정의된 var에 대해 lateinit을 적용할 수 있다.
- lateinit의 제약 사항
    - lateinit은 var 프로퍼티에만 적용할 수 있고 val에는 적용할 수 없다.
    - 프로퍼티의 타입은 널이 아닌 타입이어야 한다.
    - 프로퍼티가 원시 타입의 값이 아니어야 한다.
    - 추상 클래스의 추상 프로퍼티나 인스턴스의 프로퍼티에 lateinit을 적용할 수 없다.
    - 커스텀 게터 및 세터를 지원하는 프로퍼티에 lateinit을 적용할 수 없다.

```kotlin
class FaultySuitcase : Bag {
	lateinit var items: String
	override fun setUp() {}
	fun checkSocks() = "socks" in items
}

fun main() {
	val suitcase = FaultySuitcase()
	suitcase.setUp()
	capture {
		suitcase.checkSocks()
	} eq
		"UninitializedPropertyAccessException" +
		": lateinitproperty items" +
		"has not been initialized"
	}
```

- .isInitialized를 사용하면 lateinit 프로퍼티가 초기화됐는지 판단할 수 있다.
- 프로퍼티가 현재 영역 안에 있어야 하며, :: 연산자를 사용해 프로퍼티에 접근할 수 있어야만 .isInitialized에 접근할 수 있다.
    
    ```kotlin
    class Withlate {
    	lateinit var x: String
    	fun status() = "${::x.isinitialized}"
    }
    ```
    
    - 지역 lateinit var를 정의할 수도 있지만, 지역 var나 val에 대한 참조를 허용하지 않기 때문에 .isInitialized를 호출할 수 없다.
