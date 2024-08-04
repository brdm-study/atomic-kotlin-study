## 인터페이스

> 인터페이스는 타입이라는 개념을 기술한다. 인터페이스는 그 인터페이스를 구현하는 모든 클래스의 프로토타입이다.
> 
- 인터페이스의 특징
    - 인터페이스는 클래스가 무엇을 하는지는 기술하지만, 그 일을 어떻게 하는지는 기술하지 않는다.
    - 인터페이스는 (클래스의) 형태를 제시하지만, 일반적으로 구현은 포함하지 않는다.
    - 인터페이스는 객체의 동작을 지정하지만, 그 동작을 어떻게 수행하는지에 대한 세부 사항은 제시하지 않는다.
    - 인터페이스는 존재의 목표나 임무를 기술하며, 클래스는 세부적인 구현 사항을 포함한다.

- 인터페이스의 정의
    - 독립적이고 관련이 없는 여러 시스템이 만나서 서로 작용하거나 의사소통하는 장소
    - 시스템의 여러 부품이 서로 의사소통하는 수단

```kotlin
interface Player {
	val symbol: Char
}

class Food : Player {
	override val symbol = '.'
}
```

- 인터페이스 멤버를 구현할 때는 반드시 override 변경자를 붙여야 한다.
- override는 코틀린에게 일부러, 인터페이스에 있는 이름과 똑같은 이름을 의도적으로 사용한다는 사실, 즉 실수로 함수를 오버라이드하지 않는다는 점을 알려준다.

```kotlin
interface Hotness {
	fun feedback(): String
}

enum class SpiceLevel : Hotness {
	Mild {
		override fun feedback() =
			"It adds flavor!"
	},
	...
	...
}
```

- 이넘도 인터페이스를 구현할 수 있다.
    - 자바도 가능할까? YES

- SAM 변환
    - 단일 추상 메서드(Single Abstract Method, SAM) 인터페이스는 자바 개념으로, 자바에서는 멤버 함수를 ‘메서드’ 라고 부른다.
    - 코틀린에는 SAM 인터페이스를 정의하는 fun interface 라는 특별한 문법이 있다.
    
    ```kotlin
    fun interface ZeroArg {
    	fun f(): Int
    }
    
    fun interface OneArg { 
    	fun g(n: Int) : Int
    }
    
    fun interface TwoArg { 
    	fun h(i: Int, j: Int): Int
    }
    ```
    
    - fun interface 라고 쓰면 컴파일러는 그 안에 멤버 함수가 하나만 들어 있는지 확인
    - 람다를 사용하는 경우 SAM 변환이라고 한다.
    - SAM 변환을 쓰면 람다가 인터페이스의 유일한 메서드를 구현하는 함수가 된다.
    
    ```kotlin
    // SAM 인터페이스 예제
    // 위 인터페이스 코드와 연결되어있음
    
    class VerboseZero : ZeroArg { 
    	override fun f() = 11
    }
    
    val verboseZero = VerboseZero()
    
    val samZero = ZeroArg { 11 }
    
    class VerboseOne : OneArg {
    	override fun g(n: Int) = n + 47
    }
    
    val verboseOne = VerboseOne()
    
    val samOne = OneArg { it + 47 }
    
    class VerboseTwo : TwoArg {
    	override fun h(i: Int , j: Int ) = i + j
    }
    
    val verboseTwo = VerboseTwo()
    
    val samTwo = TwoArg { i, j -> i + j }
    
    fun main() {
    	verboseZero.f() eq 11 
    	samZero.f() eq 11 
    	verbose0ne.g(92) eq 139 
    	samOne.g(92) eq 139 
    	verboseTwo.h(11, 47) eq 58 
    	samTwo.h(11, 47) eq 58
    }
    
    // SAM 변환 예제
    
    fun interface Action {
    	fun act()
    }
    
    fun delayAction(action: Action) { 
    	trace("Delaying... ")
    	action.act()
    }
    
    fun main() { // Action 인터페이스를 구현하는 객체 대신 람다를 바로 전달
    	delayAction { trace("Hey!") } 
    	trace eq "Delaying... Hey!"
    } // 코틀린은 이 람다로부터 자동으로 Action 객체를 만들어준다.
    ```
    
    - 자주 쓰이는 구문을 SAM 변환을 사용해 더 간결한 구문으로 작성할 수 있다.
    - 객체가 한 번만 쓰는 경우, 겨우 한 객체를 만들기 위해 클래스를 억지로 정의할 필요가 없다.
    - 람다를 SAM 인터페이스가 필요한 곳에 넘길 수 있다.
    → 굳이 먼저 객체로 람다를 둘러쌀 필요도 없다.

## 복잡한 생성자

```kotlin
Class Alien(val name: String)

fun main() {
	val alien = Alien("Pencilvester")
	alien.name eq "Pencilvester"
}
```

- 위 코드는 생성자 코드를 쓰지 않고 코틀린이 생성자 코드를 만들어줌.

```kotlin
private var counter = 0

class Message(text: String) { 
	private val content: String 
	init {
		counter += 10
		content = "[$counter] $text"
	}
	override fun toString() = content
}

fun main() {
	val m1 = Message("Big ba-da boom!") 
	m1 eq "[10] Big ba-da boom!"
	val m2 = Message("Bzzzzt!")
	m2 eq "[20] Bzzzzt!"
}
```

- 생성자 파라미터에 var나 val이 붙어 있지 않더라도 init 블록에서 사용할 수 있다.
- content는 val로 정의되어 있지만 정의 시점에 초기화시키지 않음
→ 코틀린은 생성자 안의 어느 지점에서 한번(그리고 오직 한 번만) 초기화가 일어나도록 보장
→ content에 값을 다시 할당하거나 content를 초기화하는 일을 잊어버리면 오류가 발생
- 생성자는 (클래스 본문에 들어오기 전에 초기화된) 생성자 파라미터 목록과 init 블록(또는 블록들)을 합친 것이며, 객체를 생성하는 동안 실행된다.
- 코틀린에서는 init 절을 여럿 정의할 수 있고, 각 init 블록은 클래스 본문에 정의된 순서대로 실행

## 부생성자

- 같은 클래스에 속한 객체를 만들어내는 방법을 여러 가지 원할 경우 생성자를 ‘오버로드’ 해야 한다.
→ 코틀린에서는 오버로드한 생성자를 **부생성자**라고 부른다.
- 생성자 파라미터 목록(클래스 이름 뒤에 옴)과 프로퍼티 초기화, init 블록들을 모두 합한 생성자는 **주생성자**라고 부른다.

- 부생성자 만드는 방법
    - constructor 키워드 다음에 주생성자나 다른 부생성자의 파라미터 리스트와 구별되는 파라미터 목록을 넣는다.
    
    ```kotlin
    class WithSecondary(i: Int) { 
    	init {
    		trace("Primary: $i")
    	}
    	constructor(c: Char) : this(c - 'A') { 
    		trace("Secondary: '$c'")
    	}
    	constructor(s: String) : 
    		this(s.first() ) { // [1] 
    		trace("Secondary: \"$s\"")
    	}
    	/* 주생성자를 호출하지 않으면 
    		컴파일이 되지 않는다
    	constructor(f: Float) { // [2] 
    		trace("Secondary: $f")
    	}
    	*/
    }
    
    fun main() {
    	fun sep() = trace("-".repeat(10))
    	WithSecondary(1) // 주생성자 호출
    	sep()
    	WithSecondary('D') // 첫 번째 부생성자 호출
    	sep()
    	WithSecondary("Last Constructor") // 두 번째 부생성자 호출
    	trace eq """
    		Primary: 1
    		----------
    		Primary: 3 
    		Secondary: 'D'
    		----------
    		Primary: 11
    		Secondary: 'L'
    		Secondary : " Last Constructor"
    	"""
    }
    ```
    
    - 부생성자 안에서는 this 키워드를 통해 주생성자나 다른 부생성자를 호출
    - 부생성자에서 다른 생성자를 허출(this 사용)하는 부분은 생성자 로직 앞에 위치해야 한다.
    → 생성자 본문이 다른 초기화의 결과에 영향을 받을 수 있기 때문
    - 주생성자는 언제나 부생성자에 의해 직접 호출되거나 다른 부생성자 호출을 통해 간접적으로 호출되어야 한다.
    - 생성자 사이에 공유되어야 하는 모든 초기화 로직은 반드시 주생성자에 위치해야 한다.
    - 부생성자를 쓸 때 init 블록을 꼭 쓸 필요는 없다.
    - 주생성자의 파라미터만 val이나 var를 덧붙여 프로퍼티로 선언할 수 있다.
    - 부생성자에 반환 타입을 지정할 수 없다.
    - 부생성자 본문을 적지 않아도 된다(하지만 this() 호출은 반드시 포함해야 한다).

- 추가적인 부생성자 활용 예시

```kotlin
class DatabaseConnection {
    private var host: String
    private var port: Int
    private var username: String
    private var password: String
    private var database: String
    private var useSSL: Boolean

    // 주 생성자
    constructor(host: String, port: Int, username: String, password: String, database: String, useSSL: Boolean) {
        this.host = host
        this.port = port
        this.username = username
        this.password = password
        this.database = database
        this.useSSL = useSSL
        println("Full configuration: Connecting to $database on $host:$port")
    }

    // 부생성자 1: 기본 포트(3306)를 사용하는 경우
    constructor(host: String, username: String, password: String, database: String) : this(host, 3306, username, password, database, true) {
        println("Using default port 3306")
    }

    // 부생성자 2: 연결 문자열을 사용하는 경우
    constructor(connectionString: String) : this("", 0, "", "", "", false) {
        // 연결 문자열 파싱 로직
        val parts = connectionString.split(";")
        for (part in parts) {
            val (key, value) = part.split("=")
            when (key.toLowerCase()) {
                "host" -> this.host = value
                "port" -> this.port = value.toInt()
                "username" -> this.username = value
                "password" -> this.password = value
                "database" -> this.database = value
                "usessl" -> this.useSSL = value.toBoolean()
            }
        }
        println("Parsed connection string for database: $database")
    }

    // 부생성자 3: 환경 변수를 사용하는 경우
    constructor() : this("", 0, "", "", "", false) {
        this.host = System.getenv("DB_HOST") ?: "localhost"
        this.port = System.getenv("DB_PORT")?.toIntOrNull() ?: 3306
        this.username = System.getenv("DB_USERNAME") ?: "root"
        this.password = System.getenv("DB_PASSWORD") ?: ""
        this.database = System.getenv("DB_NAME") ?: "mydatabase"
        this.useSSL = System.getenv("DB_USE_SSL")?.toBoolean() ?: true
        println("Using environment variables for database connection")
    }

    fun connect() {
        // 실제 데이터베이스 연결 로직
        println("Connecting to database $database on $host:$port with user $username")
    }
}

fun main() {
    // 모든 매개변수를 지정하여 연결
    val conn1 = DatabaseConnection("localhost", 3306, "user", "pass", "mydb", true)
    conn1.connect()

    // 기본 포트를 사용하여 연결
    val conn2 = DatabaseConnection("localhost", "user", "pass", "mydb")
    conn2.connect()

    // 연결 문자열을 사용하여 연결
    val conn3 = DatabaseConnection("host=myserver;port=3307;username=user;password=pass;database=mydb;usessl=true")
    conn3.connect()

    // 환경 변수를 사용하여 연결
    val conn4 = DatabaseConnection()
    conn4.connect()
}
```

## 상속

> 상속은 기존 클래스를 재사용하면서 변경해 새로운 클래스를 만드는 메커니즘
> 

```kotlin
open class Base

class Derived : Base()
```

- 상속 관계를 표현할 때는 기반 클래스와 파생 클래스라는 용어를 자주 사용
    - 부모 클래스와 자식 클래스
    - 상위 클래스와 하위 클래스
- 기반 클래스는 open 이어야 한다.
- open 으로 지정하지 않은 클래스는 상속을 허용하지 않는다.
→ 클래스는 기본적으로 상속에 대해 닫혀 있다.
    - 자바에서는 final을 사용해 클래스의 상속을 명시적으로 금지하지 않는 한 클래스는 자동으로 상속 가능
    - 코틀린에서도 final을 쓸 수 있지만, 모든 클래스가 기본적으로 final이기 때문에 굳이 final을 지정할 필요가 없다.
    - 코틀린에서는 open 키워드를 사용해 해당 클래스가 상속을 고려해 설계됐다는 사실을 명시적으로 드러낸다.

```kotlin
open class GreatApe { 
	val weight = 100.0 
	val age = 12
}

open class Bonobo : GreatApe() 
class Chimpanzee : GreatApe() 
class BonoboB : Bonobo()

fun GreatApe.info() = "wt: $weight age: $age" //확장 함수

fun main() {
	GreatApe().info() eq "wt: 100.0 age: 12" 
	Bonobo().info() eq "wt: 100.0 age: 12" 
	Chimpanzee().info() eq "wt: 100.0 age: 12" 
	BonoboB().info() eq "wt: 100.0 age: 12"
}
```

- 코틀린은 Bonobo, Chimpanzee, BonoboB 모두 GreatApe 와 같은 타입인 것처럼 받아들인다.
- 이런 식으로 하위 클래스를 상위 클래스와 같은 타입으로 취급하는 동작은 상속의 단계가 아무리 깊어도 제대로 동작한다.
- 상수를 사용하면 어떤 클래스뿐 아니라 그 클래스를 상속하는 모든 클래스에서 사용할 수 있는 코드를 작성할 수 있다.
→ 상속은 코드를 단순화하고 재사용할 수 있는 기회를 제공

```kotlin
open class Great Ape { 
	protected var energy = 0 
	open fun call() = "Hoo!" 
	open fun eat() {
		energy += 10
	}
	fun climb(x: Int) { 
		energy -= x
	}
	fun energylevel() = "Energy: $energy"
}

class Bonobo : GreatApe( ) { 
	override fun call() = "Eep!" //오버라이딩
	override fun eat() {
		// 기반 클래스의 프로퍼티를 변경한다 
		energy += 10
		// 기반 클래스의 함수를 호출한다 
		super.eat()
		}
		// 함수를 추가한다
		fun run() = "Bonobo run"
	}
	
	class Chimpanzee : GreatApe() { 
		// 새 프로퍼티
		val additionalEnergy = 20 
		override fun call() = "Yawp!" //오버라이딩
		override fun eat() {
			energy += additionalEnergy 
			super.eat()
		}
		// 함수를 추가한다
		fun jump() = "Chimp jump"
	}
	
	fun talk(ape: GreatApe): String {
		// ape.run() // ape의 함수가 아니다
		// ape.jump() // 역시 ape의 함수가 아니다 
		ape.eat()
		ape.climb(10)
		return "${ape.call()} ${ape.energylevel()}"
	}
	
	fun main() {
		// 'energy’에 접근할 수 없다
		// GreatApe().energy
		talk(GreatApe()) eq "Hoo! Energy: 0"
		talk(Bonobo()) eq "Eep! Energy: 10"
		talk(Chimpanzee()) eq "Yawp! Energy: 20"
	}
```

- 파생 클래스에서 기반 클래스와 똑같은 시그니처를 갖는 함수를 정의하면 기반 클래스에 정의됐던 함수가 수행하던 동작을 새로 정의한 함수의 동작으로 대체
→ 오버라이딩
- 코틀린의 오버라이드 제약
    - 기반 클래스의 함수 시그니처와 똑같은 시그니처의 함수를 파생 클래스에서 사용할 때 override 키워드를 적지 않으면 우연히 오버라이드를 하는 실수를 저질렀다고 가정하여 오류 메시지를 표시한다.
    - 기반 클래스의 함수가 open으로 지정되어 있지 않으면 하위 클래스에서 이 함수를 오버라이드 할 수 없다.
- talk() 안에서 call()은 각 타입에 따라 다른 동작을 수행
→ 다형성

## 기반 클래스 초기화

> 클래스가 다른 클래스를 상속할 때, 코틀린은 두 클래스가 모두 제대로 초기화되도록 보장
> 
- 코틀린은 다음 생성자가 호출되도록 보장함으로써 올바른 객체를 생성한다.
    - 멤버 객체들의 생성자
    - 파생 클래스에 추가된 객체의 생성자
    - 기반 클래스의 생성자
- 클래스를 상속할 때는 기반 클래스 생성자의 인자 목록을 기반 클래스 이름 뒤에 부여야 한다.
→ 하위 클래스 객체를 생성하는 중에 기반 클래스 생성자를 호출
- 기반 클래스 생성자 파라미터가 없어도 코틀린은 기반 클래스의 생성자를 인자 없이 호출하기 위해 기반 클래스 이름 뒤에 괄호를 붙이도록 강제한다.
- 파생 클래스의 파라미터가 꼭 기반 클래스 생성자의 파라미터의 수, 타입, 순서에 의해 제약 받을 필요는 없다.
→ 파생 클래스의 책임은 기반 클래스 생성자를 호출할 때 제대로 인자를 제공하는 것뿐이다.

## 추상 클래스

> 추상 클래스는 하나 이상의 프로퍼티나 함수가 불완전하다는 점을 제외하면 일반 클래스와 같다.
본문이 없는 함수 정의나 초깃값 대입을 하지 않은 프로퍼티 정의가 불완전한 정의다.
인터페이스는 추상 클래스와 비슷하지만, 인터페이스에는 추상 클래스와 달리 상태가 없다.
> 
- 클래스 멤버에서 본문이나 초기화를 제거하려면 abstract 변경자를 해당 멤버 앞에 붙여야 한다.
- abstract가 붙은 멤버가 있는 클래스에는 반드시 abstract를 붙여야 한다.

```kotlin
abstract class WithProperty { 
	abstract val x: Int
}

abstract class WithFunctions { 
	abstract fun f(): Int
	abstract fun g(n: Double)
}
```

- g() 처럼 abstract 함수의 반환 타입을 적지 않으면 코틀린은 반환 타입을 Unit이라고 간주
- 추상 클래스를 상속하는 상속 관계를 따라가다 보면, 어딘가에 궁극적으로 추상 함수와 프로퍼티의 정의가 있는 클래스가 존재해야 한다.
- 인터페이스에 정의된 함수나 프로퍼티는 모두 기본적으로 추상 멤버다.
- 인터페이스에 함수나 프로퍼티 정의가 있을 때는 abstract가 불필요한 중복이므로 생략할 수 있다.
    
    ```kotlin
    interface Redundant {
    	abstract val x: Int
    	abstract fun f(): Int
    	abstract fun g(n: Double)
    }
    
    interface Removed {
    	val x: Int
    	fun f(): Int
    	fun g(n: Double)
    }
    ```
    

- 인터페이스와 추상 클래스의 차이점
    - 추상 클래스는 상태가 있지만 인터페이스는 상태가 없다.
        
        (상태 : 프로퍼티 안에 저장된 데이터를 뜻함)
        
- 인터페이스도 프로퍼티를 선언할 수 있지만, 데이터는 실제로 해당 인터페이스를 구현하는 클래스 안에만 저장될 수 있다.
- 인터페이스 안에서 프로퍼티에 값을 저장하는 것은 금지되어 있다.

```kotlin
interface IntList {
	val name: String
	//컴파일 되지 않는다
	// val list = listOg(0)
}
```

- 인터페이스와 추상 클래스는 해당 타입의 객체가 생성되기 전에 모든 추상 프로퍼티와 함수가 구현되도록 보장한다.
- 객체를 생성하기 전에는 클래스의 멤버를 호출할 수 없다.

- 코틀린에서는 클래스가 오직 한 기반 클래스만 상속할 수 있다.

```kotlin
open class Animal
open class Mammal: Animal()
open class AquaticAnimal : Animal()

// 기반 클래스가 둘 이상이면 컴파일이 되지 않는다 
// class Dolphin : Mammal(), AquaticAnimal()
```

- 코틀린도 자바와 마찬가지로 다중 상태 상속을 금지하는 대신 다중 인터페이스 상속은 허용
- 여러 인터페이스를 상속하다 보면 시그니처(함수 이름과 파라미터 목록을 합친 정보)가 같은 함수를 둘 이상 동시에 상속할 때가 있다.
→ 프로그래머가 직접 충돌 해결
    
    ```kotlin
    interface A { 
    	fun f() = 1
    	fun g() = "A.g" 
    	val n: Double
    		get() = 1.1
    }
    
    interface B { 
    	fun f() = 2
    	fun g() = "B.g" 
    	val n: Double
    		get() = 2.2
    }
    
    class C : A, B {
    	override fun f() = 0
    	override fun g() = super<A>.g() 
    	override val n: Double
    		get() = super<A>.n + super<B>.n
    }
    
    fun main() { 
    	val C = C()
    	c.f() eq 0
    	c.g() eq "A. g"
    	c.n eq 3.3
    }
    ```
    
    - 코틀린은 식별자가 같은데 타입이 다른 식으로 충돌이 일어나는 경우를 허용하지 않는다.

## 업캐스트

> 업캐스트 : 객체 참조를 받아서 그 객체의 기반 타입에 대한 참조처럼 취급하는 것
> 
- 코틀린은 단일 상속 계층 내 여러 클래스에서 코드를 재사용할 수 있는 방식으로만 상속을 사용하게 한다.

<img width="216" alt="스크린샷 2024-07-30 오후 11 27 39" src="https://github.com/user-attachments/assets/82ec89ef-0ba7-43d7-96b5-31714c56f255">


- 업캐스트가 이뤄지면서 각 객체가 Circle, Square, Triangle 중 어느 타입인지에 대한 구체적인 정보는 사라진다.
→ 모두 그냥 Shape 객체로 취급
- 구체적인 타입을 더 일반적인 타입으로 다루는 게 상속의 전부
- 상속 메커니즘은 오직 기반 타입으로 업캐스트한다는 목적을 달성하기 위해 존재
- 이런 추상화로 인해 구체적인 타입에 따라 매번 show() 함수를 작성하지 않고 단 한번만 작성해도 된다.
- 객체를 위해 작성된 코드를 재사용하는 방법이 업캐스트
- 실제로 업캐스트를 사용하지 않는데 상속을 사용하는 거의 모든 경우는 상속을 잘못 사용하는 것
→ 격언 : 상속보다 합성을 택하라

- 치환 가능성은 리스코프 치환 원칙이라고도 부르는데, 업캐스트를 한 다음에는 파생 타입이 정확히 기반 타입과 똑같이 (그 이상도 그 이하도 아닌) 취급될 수 있다고 말한다.
→ 업캐스트가 파생 클래스에 추가된 멤버 함수를 ‘잘라버리는’ 효과가 있다는 뜻

```kotlin
// shape에는 color(), rotate()가 없다.

fun trim(shape: Shape) { 
	trace(shape.draw())
	trace(shape.erase())
	// 컴파일되지 않는다
	// shape.color()
	// shape.rotate() 
}

fun main() {
	trim(Square())
	trim(Triangle()) 
	trace eq """
		Square.draw 
		Square.erase 
		Triangle.draw 
		Triangle.erase
	"""
}
```

- trim() 안에서 사용할 수 있는 멤버 함수는 모든 Shape에 공통으로 들어 있는 멤버들, 즉 기반 타입 Shape에 정의된 멤버들 뿐이다.
- 업캐스트를 한 다음에는 기반 타입의 멤버만 호출할 수 있다.

## 다형성

> 다형성 : 객체나 멤버의 여러 구현이 있는 경우를 뜻함
> 

```kotlin
open class Pet {
	open fun speak( ) = "Pet"
}

class Dog : Pet() {
	override fun speak() = "Bark!"
}

class Cat : Pet() {
	override fun speak() = "Meow"
}

fun talk(pet: Pet) = pet.speak()

fun main() {
	talk(Dog()) eq "Bark!"
	talk(Cat()) eq "Meow" 
}
```

- Dog이나 Cat을 talk()에 넘길 때 파라미터 타입이 Pet이기 때문에 구체적인 타입은 잊혀진다.
→ 즉, Dog나 Cat은 모두 Pet으로 업캐스트된다.
- 다형성은 부모 클래스 참조가 자식 클래스의 인스턴스를 가리키는 경우 발생
- 부모 클래스 참조에 대해 멤버를 호출하면 다형성에 의해 자식 클래스에서 오버라이드한 올바른 멤버가 호출
- 함수 호출을 함수 본문과 연결 짓는 작업을 **바인딩**이라고 한다.
    - 바인딩은 컴파일 시점에 일어난다.
- 다형성이 사용되는 경우에는 같은 연산이 타입에 따라 다르게 작동해야 한다.
    - 컴파일러는 어떤 함수 본문을 사용해야할지 미리 알 수 없기에 함수 본문을 동적 바인딩을 사용해 실행 시점에 동적으로 결정한다.
    (동적 바인딩을 늦은 바인딩이나 동적 디스패치라고 부르기도 한다.)

- 동적 바인딩은 정적 바인딩을 사용할 때와 비교하면,
실행 시점에 타입을 결정하는 추가 로직이 성능에 약간 부정적인 영향을 끼침

## 합성

> 객체 지향을 사용해야 하는 가장 큰 이유는 코드 재사용이다.
> 
- 합성 : 기존 클래스의 객체를 새 클래스 안에 생성하는 직접적인 접근 방식(새 클래스가 기존 클래스들을 합성한 객체로 이뤄짐)
- 합성을 쓸 경우는 기본 코드의 (형태가 아니라) 기능을 재사용하는 것
- 합성은 포함 관계
    - 집은 건물이며(is a), 부엌을 포함(has-a)한다.
    
    ```kotlin
    interface Building
    interface Kitchen
    
    interface House: Building {
    	val kitchen: Kitchen
    }
    ```
    
    - 상속은 ~이다(is-a) 관계를 표현
    - ‘~이다’ 관계가 타당해 보이면 상속이 잘 들어맞는 경우가 많다
    - 집에 부엌이 둘 있다면 합성을 사용해 더 쉽게 집을 표현할 수 있다.
    
    ```kotlin
    interface Building
    interface Kitchen
    
    interface House: Building {
    	val kitchen1: Kitchen
    	val kitchen2: Kitchen
    	// val kitchens: List<Kitchen>
    }
    ```
    

- 합성과 상속 중 선택하기
    - 합성과 상속 모두 새 클래스에 하위 객체를 넣는다.
    - 합성은 명시적으로 하위 객체를 선언하지만, 상속은 암시적으로 하위 객체가 생긴다.
    - 합성은 기존 클래스의 기능을 제공하지만 인터페이스를 제공하지는 않는다.
    - 합성한 객체를 완전히 감추고 싶다면 private 로 포함시키면 된다.
    
    ```kotlin
    class Features {
    	fun f1() = "feature1"
    	fun f2() = "feature2"
    }
    
    class Form {
    	private val features = Features()   //실제 사용자는 어떻게 구현됐는지 알 수 없다.
    	fun operation1() =
    		features.f2() + features.f1()
    	fun operation2() =
    		features.f1() + features.f2()
    }
    ```
    
    - Form이 Feature를 상속한다면, 클라이언트 프로그래머가 Form을 Feature로 업캐스트하리라 예상할 수 있다.
    → 연결 관계가 명확해지므로 상속 관계가 Form의 일부가 된다.
    → 관계를 수정하면 해당 연결 관계에 의존하는 모든 코드가 망가진다.

```kotlin
class Engine {
	fun start() = trace("Enginestart")
	fun stop() = trace("Enginestop")
}

class Wheel {
	fun inflate(psi : Int) =
		trace("Wheel inflate($psi)")
}

class Window(val side: String) {
	fun rollUp( ) =
		trace("$side Window roll up" )
	fun rollDown() =
		trace("$side Window roll down")
}

class Door(val side: String) {
	val window = Window(side)
	fun open() = trace("$side Door open")
	fun close() = trace("$side Door close")
}

class Car {
	val engine = Engine()
	val wheel = List(4) { Wheel() }
	// 두 개의 문
	val leftDoor = Door("left")
	val rightDoor = Door("right")
}

fun main() {
	val car = Car()
	car.leftDoor.open()
	car.rightDoor.window.rollUp()
	car.wheel[0].inflate(72)
	car.engine.start()
	trace eq """
		left Door open
		right Window roll up
		Wheel inflate(72)
		Engine start
	"""
}
```

- 이런 식으로 내부를 노출시킨 설계는 클라이언트가 클래스를 사용하는 방법을 이해할 때 도움이 되고, 클래스를 만든 사람의 코드 복잡도를 줄여준다.

- 상속을 하면 기존 클래스를 커스텀화한 버전을 만든다.
→ 즉, 일반적인 목적의 클래스를 갖고 특정 필요에 맞는 특별한 클래스를 만든다.
- Car는 Vehicle을 포함하지 않으며, Vehicle이다. ‘~이다’ 관계는 상속으로 표현하고, 포함 관계는 합성으로 표현한다.
- 기존 클래스를 사용해 새 클래스를 만들 때 상속을 우선적으로 선택하면 모든 것이 불필요하게 복잡해진다.
→ 합성을 먼저 시도해보라.

## 상속과 확장

- 때로는 기존 클래스를 새로운 목적으로 활용하기 위해 새로운 함수를 추가해야 할 때가 있다.
- 이때 기존 클래스를 변경할 수 없으면 새 함수를 추가하기 위해 상속을 사용해야 한다.
→ 이로 인해 코드를 이해하고 유지 보수하기 어려워진다.

```kotlin
open class Heater {
	fun heat(temperature: Int) =
		"heating to $temperature"
}

fun warm(heater: Heater) {
	heater.heat(70) eq "heating to 70"
}
```

- 가정 : 이 라이브러리를 변경하고 싶지 않고, 재사용하고 싶다. + 실제로 원하는 기능은 냉난방 및 공조 시스템(HVAC)
- 상속을 사용하면 기존 warm() 함수와 다른 모든 함수는 Heater에 작용할 수 있으므로 우리가 만든 새 HVAC 타입에 대해서도 작동
- 만약 합성을 사용한다면 새 HVAC 타입을 기존 함수에 적용할 수 없다.

```kotlin
class HVAC : Heater() {
	fun cool(temperature: Int) =
		"cooling to $temperature"
}

fun warmAndCool(hvac: HVAC) {
	hvac.heat(70) eq "heating to 70"
	hvac.cool(60) eq "cooling to 60"
}

fun main() {
	val heater = Heater()
	val hvac = HVAC()
	warm(heater)
	warm(hvac)
	warmAndCool(hvac)
}
```

- 위 방식은 Heater가 원하는 기능을 전부 제공하지 못하므로, Heater를 상속해 HVAC를 만들고 다른 함수를 추가한다.
- 추가된 함수는 업캐스트를 하면 잘려나가므로 기반 클래스에서는 쓸 수 없다.
    - 리스코프 치환 원칙(치환 가능성) : 기반 클래스를 받아들이는 함수는 반드시 하위 클래스의 객체를 (하위 클래스라는 사실을 알지 못하고) 받아도 아무 문제가 없어야 한다.

- 상속을 하면서 함수를 추가하는 건, 클래스에 기반 클래스가 있다는 사실을 무시하고 시스템 전반에서 파생 클래스를 엄격하게 식별해 취급할 때 유용하다.
- 기반 클래스 타입의 참조를 통해서만 파생 클래스 인스턴스에 접근한다면, 파생 클래스에 추가된 함수를 호출할 방법이 없으므로 쓸데없이 함수를 추가한 셈이 된다.

```kotlin
fun Heater.cool(temperature: Int) = 
	"cooling to $temperature"
	
fun warmAndCool(heater: Heater) { 
	heater.heat(70) eq "heating to 70" 
	heater.cool(60) eq "cooling to 60"
}

fun main() {
	val heater = Heater() 
	warmAndCool(heater)
}
```

- HVAC 클래스를 만든 이유는 Heater 클래스에 cool() 함수를 추가해 warmAndCool() 함수 안에서 warm() 과 cool()을 모두 쓰기 위함
→ 확장 함수를 쓰면 상속이 필요 없다.
- 기반 클래스 인터페이스를 확장하기 위해 상속하는 대신, 확장 함수를 사용하면 상속을 사용하지 않고 기반 클래스의 인스턴스를 직접 확장할 수 있다.

```kotlin
// Heater 라이브러리의 소스 코드를 바꿀 수 있다면 다른 방식으로 설계 예제

class TemperatureDelta(
	val current: Double, 
	val target: Double
}

fun TemperatureDelta.heat() { 
	if (current < target)
		trace("heating to $target")
}

fun TemperatureDelta.cool() { 
	if (current > target)
		trace("cooling to $target")
}

fun adjust(deltaT: TemperatureDelta) { 
	deltaT.heat()
	deltaT.cool()
}

fun main() {
	adjust(TemperatureDelta(60.0, 70.0)) 
	adjust(TemperatureDelta(80.0, 60.0))
	trace eq """
		heating to 70.0 
		cooling to 60.0
	"""
}
```

- 코틀린 표준 라이브러리의 Sequence 인터페이스에는 멤버 함수가 하나만 들어 있다.
    
    (다른 모든 Sequence 함수는 모두 확장이다.)
    
    - 초기에는 이런 접근 방법이 자바와 호환성을 유지하기 위한 목적으로 쓰였지만, 이제는 ‘필수적인 메서드만 정의해 포함하는 간단한 인터페이스를 만들고, 모든 부가 함수를 확장으로 정의하라’ 라는 코틀린 철학이 됨

- 어댑터 패턴
    
    ```kotlin
    interface LibType {
    	fun f1()
    	fun f2()
    }
    
    fun utility(lt: Libtype) {
    	lt.f1()
    	lt.f2()
    }
    
    fun utility(lt: LibType) {
    	lt.f2()
    	lt.f1()
    }
    
    // 이 라이브러리를 사용하려면 우리가 만든 기존 클래스를 LibType으로 
    // 변환할 방법이 필요하다.
    ```
    
    ```kotlin
    open class MyClass { 
    	fun g() = trace("g()") 
    	fun h() = trace("h()")
    }
    
    fun useMyClass(mc: MyClass) 
    	mc.g()
    	me.h() 
    }
    
    class MyClassAdaptedForLib
    	MyClass(), LibType { 
    	override fun f1() = h() 
    	override fun f2() = g()
    }
    
    fun main() {
    	val mc = MyClassAdaptedForlib()
    	utility1(mc)
    	utility2(mc)
    	useMyClass(mc)
    	trace eq "h() g() g() h() g() h()"
    }
    
    // 상속을 하는 과정에서 클래스를 확장하긴 하지만,
    // 새 멤버 함수는 오직 UsefullLibrary 에 연결하기 위해서만 쓰인다.
    // 다른 곳에서는 MyClass 객체로 취급할 수 있다.
    // 위 코드는 MyClass가 상속에 대해 열린 open 클래스라는 점에 의존
    
    // MyClass 를 수정할 수 없고 open도 아니라면? >> Adapter 사용
    ```
    

```kotlin
class MyClass { // open된 클래스가 아님 
	fun g() = trace("g()")
	fun h() = trace("h()")
}

fun useMyClass(mc: MyClass) { 
	mc.g()
	me.h() 
}

class MyClassAdaptedForlib : LibType { 
	val field = MyClass() 
	override fun f1() = field.h()
	override fun f2() = field.g()
}

fun main() {
	val me = MyClassAdaptedForLib() 
	utility1(mc)
	utility2(mc)
	useMyClass(mc.field)
	trace eq "h() g() g() h() g() h()"
}

// 이 코드는 useMyClass(mc.field) 호출처럼 명시적으로 MyClass 객체에
// 접근해야 하지만 기존 라이브러리를 새로운 인터페이스에 맞게 전환해 연결하는
// 문제를 쉽게 해결해준다.
```

- 확장 함수를 모아서 인터페이스를 구현할 수는 없다.

- 멤버 함수와 확장 함수 비교
    - 확장 함수의 가장 큰 한계는 오버라이드할 수 없다는 점
    - 멤버 함수는 타입의 핵심을 반영
    (그 함수 없이는 그 타입을 상상할 수 없어야 한다.)
    - 확장 함수는 타입의 존재에 필수적이지 않은, 대상 타입을 지원하고 활용하기 위한 외부 연산이나 편리를 위한 연산이다.
    - 타입 내부에 외부 함수를 포함하면 타입을 이해하기가 더 어려워지지만, 일부 함수를 확장 함수로 정의하면 대상 타입을 깔끔하고 단순하게 유지할 수 있다.
        
        ```kotlin
        // 멤버 함수로 정의한 코드
        
        interface Device {
        	val model: String
        	val productionYear: Int
        	fun overpriced() = model.startsWith("i") 
        	fun outdated() = productionYear < 2050
        }
        
        class MyDevice(
        	override val model: String,
        	override val productionYear: Int
        ): Device
        
        fun main() {
        	val gadget: Device =
        		MyDevice("my first phone", 2000)
        	gadget.outdated() eq true 
        	gadget.overpriced() eq false
        }
        ```
        
        ```kotlin
        // overpriced() 와 outdated()를 하위 클래스에서 오버라이드할
        // 가능성이 없다고 가정하고 확장으로 정의한 코드
        
        interface Device {
        	val model: String
        	val productionYear: Int
        }
        
        fun Device.overpriced() = 
        	model.startsWith("i")
        	
        fun Device.outdated() = 
        	productionYear < 2050
        
        class MyDevice(
        	override val model: String, 
        	override val productionYear: Int
        ): Device
        
        fun main() {
        	val gadget: Device =
        		MyDevice("my first phone", 2000)
        	gadget.outdated() eq true 
        	gadget.overpriced() eq false
        }
        ```
        
        - 두 방법은 궁극적으로 설계상의 선택일 뿐
- C++이나 자바 같은 언어는 특별히 금지하지 않는 한 상속을 허용
- 코틀린은 기본적으로 상속을 사용하지 않을 거라고 가정

## 클래스 위임

> 합성과 상속은 모두 새 클래스 안에 하위 객체를 삼는다.
합성에서는 하위 객체가 명시적으로 존재하고, 상속에서는 암시적으로 존재한다.
> 
- 합성은 내포된 객체의 기능을 사용하지만 인터페이스를 노출하지 않는다.
- 클래스가 기존의 구현을 재사용하면서 동시에 인터페이스를 구현해야 하는 경우, 
상속과 클래스 위임 두 가지 선택지가 있다.
- 클리스 위임 : 상속과 합성의 중간 지점
    - 합성과 마찬가지로 새 클래스 안에 멤버 객체를 심고, 상속과 마찬가지로 심겨진 하위 객체의 인터페이스를 노출 + 새 클래스를 하위 객체의 타입으로 업캐스트 가능

```kotlin

interface Controls {
	fun up(velocity : Int): String
	fun down(velocity: Int): String 
	fun left(velocity: Int): String 
	fun right(velocity: Int): String 
	fun forward(velocity: Int): String 
	fun back(velocity: Int): String 
	fun turboBoost(): String
}

class SpaceShipControls : Controls { 
	override fun up(velocity: Int) =
		"up $velocity"
	override fun down(velocity: Int) =
		"down $velocity"
	override fun left(velocity : Int) =
		"left $velocity"
	override fun right(velocity: Int) =
		"right $velocity"
	override fun forward(velocity: Int) =
		"forward $velocity"
	override fun back(velocity: Int) =
		"back $velocity"
	override fun turboBoost() = "turbo boost"
}
```

- 제어 장치의 기능을 확장하거나 명령을 일부 조정하고 싶다면 SpaceShipControls 를 상속해야 하지만 open 아니므로 상속할 수 없다.
→ Controls의 멤버 함수를 노출하려면 SpaceShipControls의 인스턴스를 프로퍼티하고 Controls의 모든 멤버 함수를 명시적으로 SpaceShipControls 인스턴스에 위힘해야함

```kotlin
class ExplicitControls : Controls {
    private val controls = SpaceShipControls()
    // 수동으로 위임 구현하기
    override fun up(velocity: Int) = 
	    controls.up(velocity)
    override fun back(velocity: Int) = 
	    controls.back(velocity)
    override fun down(velocity: Int) = 
	    controls.down(velocity)
    override fun forward(velocity: Int) = 
	    controls.forward(velocity)
    override fun left(velocity: Int) = 
	    controls.left(velocity)
    override fun right(velocity: Int) = 
	    controls.right(velocity)
    // 변형한 구현
    override fun turboBoost(): String = 
	    controls.turboBoost() + "... boooooost!"
}

fun main() {
    val controls = ExplicitControls()
    controls.forward(100) eq "forward 100"
    controls.turboBoost() eq 
	    "turbo boost... boooooost!"
}
```

- 이 클래스가 제공하는 인터페이스는 일반적인 상속에서 사용해야 하는 인터페이스와 동일하다.
- 코틀린은 이런 클래스 위임 과정을 자동화해준다.
    
    ```kotlin
    interface AI
    class A : AI
    
    class B(val a: A) : AI by a
    
    // 이 코드를 '클래스 B는 AI 인터페이스를 a 멤버 객체를 사용해(by) 구현한다'
    // 라고 읽는다.
    // 인터페이스에만 위임을 적용할 수 있다.
    // -> A() by a 라고 by 앞에 클래스 이름을 쓸 수 없다.
    // 위임 객체(a) 는 생성자 인자로 지정한 프로퍼티여야 한다.
    ```
    

```kotlin
// by를 사용한 코드

class DelegatedControls( 
	private val controls: SpaceShipControls =
		SpaceShipControls()
): Controls by controls {
	override fun turboBoost(): String =
		"${controls.turboBoost()} ... boooooost!"
}
```

- 위임을 하면 별도로 코드를 작성하지 않아도 멤버 객체의 함수를 (위임을 사용해 새로 정의한) 외부 객체를 통해 접근할 수 있다.
- 코틀린은 다중 클래스 상속을 허용하지 않지만, 클래스 위임을 사용해 다중 클래스 상속을 흉내낼 수는 있다.
- 일반적으로 다중 상속은 전혀 다른 기능을 가진 여러 클래스를 하나로 묶기 위해 쓰인다.

```kotlin
// 화면에 직사각형을 그리는 클래스와 마우스 이벤트를 관리하는 클래스를 하나로 합쳐서 버튼 만들기 예시

interface Rectangle {
    fun paint(): String
}

class ButtonImage(
    val width: Int,
    val height: Int,
): Rectangle {
    override fun paint() = 
	    "painting ButtonImage($width, $height)"
}

interface MouseManager {
    fun clicked(): Boolean
    fun hovering(): Boolean
}

class UserInput: MouseManager {
    override fun clicked() = true
    override fun hovering() = true
}

// 앞의 두 클래스를 open 으로 정의한다고 해도 하위 타입을
// 정의할 때는 상위 타입 목록에 클래스를 하나만 넣을 수 있기 때문에
// class Button : ButtonImage(), UserInput() 처럼 쓸 수 없다.

class Button(
    val width: Int,
    val height: Int,
    var image: Rectangle = 
	    ButtonImage(width, height),
    private val input: MouseManager = UserInput()
): Rectangle by image, MouseManager by input

fun main() {
    val button = Button(10, 5)
    button.paint() eq 
	    "painting ButtonImage(10, 5)"
    button.clicked() eq true
    button.hovering() eq true
    // 위임한 두 타입으로 업캐스트가 모두 가능하다.
    val rectangle: Rectangle = button
    val mouseManager: MouseManager = button
}
```

- Button 클래스는 두 인터페이스 Rectangle, MouseManager를 구현한다.
- Button이 ButtonImage와 UserInput 구현을 모두 상속할 수는 없지만, 두 클래스를 모두 위임할 수는 있다.
- 생성자 인자 목록의 image 정의가 public 인 동시에 var 라는 점에 유의
    
    → 이로 인해 클라이언트 프로그래머가 동적으로 ButtonImage를 변경할 수 있다.
    
- main() 의 마지막 두 줄은 Button을 자신이 위임한 두 가지 타입으로 업캐스트할 수 있음을 보여준다. (다중 상속의 목표)
→ 결과적으로 위임은 다중 상속의 필요성을 해결해준다.
- 상속은 제약이 될 수 있다.
    - 예를 들어 상위 클래스가 open이 아니거나, 새 클래스가 다른 클래스를 이미 상속하고 있는 경우에는 다른 클래스를 상속할 수 없다.
    - 클래스 위임을 사용하면 이런 제약을 포함한 여러 제약을 피할 수 있다.
- 클래스 위임을 조심히 사용하라.
    - 상속, 합성, 클래스 위임이라는 세 가지 방법 중에 합성을 먼저 시도하라.
    - 타입 계층과 이 계층에 속한 타입 사이의 관계가 필요할 때는 상속이 필요하다.
    - 이 두 가지 선택이 모두 적합하지 않을 떄 위임을 쓸 수 있다.

## 다운캐스트

> 다운캐스트는 이전에 업캐스트했던 객체의 구체적인 타입을 발견한다.
> 
- 기반 클래스가 파생 클래스보다 더 큰 인터페이스를 가질 수 없으므로 업캐스트는 항상 안전하다.
- 모든 기반 클래스 멤버가 존재한다고 보장할 수 있으며 멤버를 호출해도 안전한다.
- 다운캐스트는 실행 시점에 일어나며 실행 시점 타입 식별(Run-Time Type Identification, RTTI) 이라고도 한다.

- 스마트 캐스트
    - 코틀린의 스마트 캐스트는 자동 다운캐스트이다.
    - is 키워드는 어떤 객체가 특정 타입인지 검사한다.
    (이 검사 영역 안에서는 해당 객체를 검사에 성공한 타입이라고 간주)
    
    ```kotlin
    fun main() {
    	val b1: Base = Derived1() // 업캐스트
    	if(b1 is Dervived1)
    		b1.g() // 'is' 검사의 영역 내부
    	// val b2: Base = Derived2() // 업캐스트
    	if(b2 is Dervived2)
    		b2.h() // 'is' 검사의 영역 내부
    }
    ```
    
    - 스마트 캐스트는 is를 통해 when의 인자가 어떤 타입인지 검색하는 when 식 내부에서 아주 유용하다.

```kotlin
fun what(c: Creature): String = 
	when (c) {
    is Human -> c.greeting()
    is Dog -> c.bark()
    is Alien -> c.mobility()
    else -> "Something else"
}
```

- what()은 이미 업캐스트된 Creature를 받아서 정확한 타입을 찾는다.
- Creature 객체를 상속 계층에서 정확한 타입, 더 일반적인 기반 클래스에서 더 구체적인 파생 클래스로 다운캐스트한다.

- 변경 가능한 참조
    - 자동 다운캐스트는 특히 대상이 상수어야만 제대로 작동
    - 대상 객체를 가리키는 기반 클래스 타입의 참조가 변경 가능(var)하다면 타입 검사와 사용 지점 사이에 객체의 구체적인 타입이 달라질 수 있다는 뜻

- as 키워드
    - as 키워드는 일반적인 타입을 구체적인 타입으로 강제 변환한다.
    - as가 실패하면 ClassCastException을 던진다.
    - 안전한 캐스트인 as?는 실패해도 예외를 던지지 않는 대신 null을 반환한다.
    
    ```kotlin
    fun dogBarkSafe(c: Creature) =
    	(c as? Dog)?.bark() ?: "Not a Dog"
    	
    fun main() {
    	dogBarkSafe(Dog()) eq "Yip!" 
    	dogBarkSafe(Human()) eq "Not a Dog"
    }
    ```
    

- 리스트 원소의 타입 알아내기
    - 술어에서 is를 사용하면 List나 다른 이터러블(이터레이션을 할 수 있는 대상 타입)의 원소가 주어진 타입의 객체인지 알 수 있다.
    
    ```kotlin
    val group: List<Creature> = listOf(
    	Human(), Human(), Dog(), Alien(), Dog()
    )
    
    fun main() {
    	val dog = group
    		.find { it is Dog } as? Dog // [1]
      dog?.bark() eq "Yip!"
    }
    ```
    
    - 객체를 Dog으로 다루고 싶어서 [1] 명시적으로 타입을 변환
    - group 안에 Dog이 하나도 없을 수도 있으며, find()가 null을 반환하므로 Dog?로 변환해야 한다.
    - 보통은 지정한 타입에 속하는 모든 원소를 돌려주는 filterInstance()를 써서 [1]과 같은 코드를 피한다.
        
        ```kotlin
        fun main() {
        	val humans1: List<Creature> =
        		group.filter { it is Human } 
        	humans1.size eq 2
        	val humans2: List<Human> =
        		group.filterIsInstance<Human>() 
        		humans2 eq humans1
        }
        ```
        
        - filterIsInstance()는 filter()와 is를 사용한 경우와 같은 결과를 내놓지만 가독성이 더 좋다. 다만 결과 타입은 다르다.
        - filter() (반환값의 모든 원소가 Human임에도 불구하고) Creature의 List를 내놓는 반면, filterIsInstance()는 대상 타입인 Human의 리스트를 반환

## 봉인된 클래스

> 클래스 계층을 제한하려면 상위 클래스를 sealed로 선언하라.
> 

```kotlin
// 여행자들이 여러 교통 수단을 이용해 여행을 다니는 경우를 생각해보자

open class Transport

data class Train(
	val line: String
): Transport()

data class Bus(
	val number: String,
  val capacity: Int,
): Transport()

fun travel(transport: Transport) =
	when (transport) {
		is Train -> 
			"Train ${transport.line}"
    is Bus -> 
	    "Bus ${transport.number}: " + 
		    "size ${transport.capacity}"
    else -> "${transport} is in limbo!"
	}

fun main() {
	listOf(Train("S1"), Bus("11", 90))
		.map(::travel) eq
		"[Train S1, Bus 11: size 90]"
}
```

- Train과 Bus는 Transport 유형에 따라 다른 세부 사항을 저장한다.
- travel()은 다운캐스트가 근본적인 문제가 될 수 있는 지점이라는 사실을 보여준다.
    - Tram이라는 클래스를 새로 정의했다고 하면, travel()이 여전히 컴파일되고 실행도 되기 때문에 Tram 추가에 맞춰 when을 바꿔야 한다는 아무런 단서가 없다.
    → 다운캐스트가 여기저기 흩어져 있다면 이런 변경으로 인해 유지 보수가 힘들어진다.
- 이 상황은 sealed 키워드를 사용해 개선할 수 있다.

```kotlin
sealed class Transport

data class Train(
	val line: String
): Transport()

data class Bus(
	val number: String,
  val capacity: Int,
): Transport()

fun travel(transport: Transport) =
	when (transport) {
		is Train -> 
			"Train ${transport.line}"
    is Bus -> 
	    "Bus ${transport.number}: " + 
		    "size ${transport.capacity}"
	}

fun main() {
	listOf(Train("S1"), Bus("11", 90))
		.map(::travel) eq
		"[Train S1, Bus 11: size 90]"
}
```

- sealed 클래스를 직접 상속한 하위 클래스는 반드시 기반 클래스와 같은 패키지와 모듈 안에 있어야 한다.
- Transport가 sealed라서 코틀린이 다른 Transport의 하위 클래스가 존재할 수 없다는 확신을 할 수 있기 때문에 else 가지가 필요 없다.
- Sealed 클래스는 Tram 같은 하위 클래스를 도입했을 때 변경해야 하는 모든 지점을 표시해준다.
- 다운캐스트를 과도하게 사용하는 설계를 본다면 미심쩍게 생각해야 한다.
→ 보통은 다형성을 써서 코드를 더 깔끔하게 잘 작성할 수 있는 방법이 있다.

- sealed 와 abstract 비교
    
    ```kotlin
    // abstract 와 sealed 클래스가 타입이 똑같은 함수, 프로퍼티, 생성자를 제공하는 경우
    
    abstract class Abstract(val av: String) {
    	open fun concreteFunction() {}
    	open val concreteProperty = ""
    	abstract fun abstractFunction(): String
    	abstract val abstractProperty: String
    	init {}
    	constructor(c: Char) : this(c.toString())
    }
    
    open class Concrete() : Abstract("") {
    	override fun concreteFunction() {}
    	override val concreteProperty = ""
    	override fun abstractFunction() = ""
    	override val abstractProperty = ""
    }
    
    sealed class Sealed(val av: String) {
    	open fun concreteFunction() {}
    	open val concreteProperty = ""
    	abstract fun abstractFunction(): String
    	abstract val abstractProperty: String
    	init {}
    	constructor(c: Char) : this(c.toString())
    }
    
    open class SealedSubclass : Sealed("") {
    	override fun concreteFunction() {}
    	override val concreteProperty = ""
    	override fun abstractFunction() = ""
    	override val abstractProperty = ""
    }
    
    fun main() {
    	Concrete()
    	SealedSubclass()
    }
    ```
    
    - sealed 클래스는 기본적으로 하위 클래스가 모두 같은 파일 안에 정의되어야 한다는 제약이 가해진 abstract 클래스다.
    - sealed 클래스의 간접적인 하위 클래스를 별도의 파일에 정의할 수 있다.
    
    ```kotlin
    class ThirdLevel : SealedSubclass()
    
    // 직접 Sealed를 상속하지 않으므로 SealedVsAbstract.kt 파일에 있을 필요가 없다.
    ```
    

- 하위 클래스 열거하기
    - 어떤 클래스가 sealed인 경우 모든 하위 클래스를 쉽게 이터레이션할 수 있다.
    
    ```kotlin
    sealed class Top
    class Middle1 : Top() 
    class Middle2 : Top()
    open class Middle3 : Top() 
    class Bottom3 : Middle3()
    
    fun main() {
    	Top::class.sealedSubclasses
    		.map { it.simpleName} eq 
    		"[Middle1, Middle2, Middle3]"
    }
    ```
    
    - 클래스를 생성하면 클래스 객체가 생성된다.
    - ::class가 클래스 객체를 돌려주므로 Top::class는 Top에 대한 클래스 객체를 만들어준다.
    - 클래스 객체의 프로퍼티 중에는 sealedSubclasses가 있다.
    - Top::class로 얻은 클래스 객체에서 이 프로퍼티는 Top이 sealed 클래스이길 기대한다.(그렇지 않다면 빈 리스트를 돌려준다.)
    - sealedSubClasses는 이런 봉인된 클래스의 모든 하위 클래스를 돌려준다.
    - sealedSubclasses는 리플렉션을 사용한다.
        - kotlin-reflection.jar라는 의존 관계가 클래스 경로에 있어야 리플렉션을 쓸 수 있다.
    - sea;edSubclasses는 하위 클래스를 실행 시점에 찾아내므로 시스템의 성능에 영향을 끼칠 수 있다.

## 타입 검사

- 예를 들어 곤충은 대부분은 날 수 있지만, 날 수 없는 곤충이 극소수 존재한다. 이때 Inspect 인터페이스가 날 수 없는 극소수 곤충의 영향을 받지 않아야 하므로 basic()에 타입 검사를 통해 날 수 없는 곤충에 제약을 가한다.
- 극소수 타입에만 적합한 특별한 행동 방식을 기반 클래스에 넣는 것은 타당하지 않다.
- 특별한 처리를 위해 별도의 소수 타입을 선택하는 것이 타입 검사의 전형적인 유스케이스다.

- 외부 함수에서 타입 검사하기
    - 음료를 저장하고 배달하는 것(BeverageContainer)
    - 재활용은 외부 함수로 다룬다.
    → 캔인지 유리인지 점점 추가가 되는데 까먹고 추가안해도 모른다.
    - 이 때 sealed 클래스를 선언하면 매번 새로 추가된 타입을 검사해야 한다는 뜻
    - 인터페이스는 sealed가 될 수 없으므로 반드시 클래스로 정의해야 한다.
    - 클래스가 BeverageContainer의 직접적인 하위 클래스이면 컴파일러는 recycle()에 있는 when의 모든 하위 타입을 검사하도록 보장한다.
    - 하지만 GlassBottle과 AluminumCan 같은 하위 타입은 검사할 수 없다.
    → 모든 타입을 검사하려면 recycle2() 같이 when 안에 다른 when을 내포시켜야한다.
    
    ```kotlin
    fun BeverageContainer.recycle() = 
    	when(this) {
    		is Can-> "Recycle Can"
    		is Bottle -> "Recycle Bottle"
    	}
    	
    fun BeverageContainer.recycle2() = 
    	when(this) {
    		is Can -> when(this) {
    			is SteelCan -> "Recycle Steel"
    			is AluminumCan-> "Recycle Aluminum"
    		is Bottle -> when(this) {
    		...
    		}
    	}
    ```
    

```kotlin
// recycle() 을 BeverageContainer 안에 넣는 방식

interface BeverageContainer { 
	fun open(): String
	fun pour() = "$name: Pour" 
	fun recycle(): String
}
...
...

class HDPEBtotle : PlasticBottle() {
	override fun recycle() = "Recycle HDPE"
}
...
```

- 모든 하위 클래스가 recycle()을 오버라이드하도록 강제한다.
- 이렇게 하면 recycle()의 행동 방식이 여러 클래스에 분산된다는 점이 있다.

## 내포된 클래스

> 내포된 클래스를 사용하면 객체 안에 더 세분화된 구조를 정의할 수 있다.
> 
- 내포된 클래스는 단순히 외부 클래스의 이름 공간 안에 정의된 클래스일 뿐이다.
- 외부 클래스의 구현이 내포된 클래스를 ‘소유’ 한다.
- 내포된 클래스가 필수적인 기능은 아니지만 코드를 깔끔하게 해줄 때가 있다.

```kotlin
class Airport(private val code: String) {
	open class Plane {
		// 자신을 둘러싼 클래스의 private 프로퍼티에 접근할 수 있다.
		fun contact(airport: Airport) =
			"Contacting ${airport.code}"
	}
	private class PrivatePlane : Plane() // Airport 밖에서 볼 수 없다.
	fun privatePlane(): Plane = PrivatePlane()
}

fun main() {
	val denver = Airport("DEN")
	var plane = Plane()   // [1]
	plane.contact(denver) eq "Contacting DEN"
	// 다음과 같이 할 수 없다.
	// val privatePlane = Airport.PrivatePlane()
    
	val frankfurt = Airport("FRA")
	plane = frankfurt.privatePlane()
	// 다음과 같이 할 수 없다.
	// val p = plane as PrivatePlane   // [2]
	plane.contact(frankfurt) eq "Contacting FRA"
}
```

- Plane 객체를 생성할 때 Airport 객체가 필요하지는 않다.
- 하지만 Airport 클래스 본문 밖에서 Plane 객체를 생성하려고 한다면 일반적으로 생성자 호출을 한정시켜야 한다.[1]
- Airport 밖에서 이렇게 받은 public 타입의 객체 참조를 다시 private 타입으로 다운캐스트할 수 없다.[2]

- 지역 클래스 : 함수 안에 내포된 클래스
    
    ```kotlin
    fun localClasses() {
    	open class Amphibian // interface여야 할 것 같지만 지역 interface 허용되지 않는다.
    	class Frog : Amphibian()
    	val amphibian: Amphibian = Frog()
    }
    ```
    
    - 지역 open 클래스는 거의 정의하지 말아야 한다.
    - 지역 클래스로 open 클래스를 정의해야 하는 경우라면 아마도 일반적인 클래스로 정의할 만큼 그 클래스가 중요한 경우일 것이다.
    - 지역 클래스의 객체를 반환하려면 그 객체를 함수 밖에서 정의한 인터페이스나 클래스로 업캐스트해야 한다.

- 인터페이스에 포함된 클래스
    - 인터페이스 안에 클래스를 내포시킬 수도 있다.
    
    ```kotlin
    interface Item {
    	val type: Type
    	data class Type(val type: String)
    }
    
    class Bolt(type: String) : Item {
    	override val type = Item.Type(type)
    }
    
    fun main() {
    	val items = listOf(
    		Bolt("Slotted"), Bolt("Hex")
    	)
    	items.map (Item::type) eq
    		"[Type(type=Slotted), Type(type=Hex)]"
    }
    ```
    
    - Bolt 안에서는 val type을 반드시 오버라이드하고 Item.Type이라는 한정시킨 클래스 이름을 써서 값을 대입해야 한다.

- 내포된 이넘
    - 이넘도 클래스이므로 다른 클래스 안에 내포될 수 있다.
    
    ```kotlin
    class Ticket(
    	val name: String,
    	val seat: Seat = Coach
    ) {
    	enum class Seat {
    		Coach,
    		Premium,
    		Business,
    		First
    	}
    ```
    
    - 이넘을 함수에 내포시킬 수는 없고, 이넘이 다른 클래스(다른 이넘 클래스도 마찬가지)를 상속할 수도 없다.
    - 인터페이스는 안에 이넘을 내포시킬 수 있다.
    
    ```kotlin
    interface Game {
    	enum class State { Playing, Finished }
    	enum class Mark { Blank, X, O}
    }
    ```
    

## 객체

- object 는 여러 인스턴스가 필요하지 않거나 명시적으로 인스턴스를 여러 개 생성하는 것을 막고 싶은 경우 논리적으로 한 개체 안에 속한 함수와 프로퍼티를 함께 엮는 방법
- object의 인스턴스를 직접 생성하는 경우는 결코 없다.
- object를 정의하면 그 object의 인스턴스가 오직 하나만 생긴다.

```kotlin
object JustOne {
	val n = 2
	fun f() = n * 10
	fun g() = this.n * 20
}

fun main() {
	// val x = JustOne() // 오류
	JustOne.n eq 2
	JustOne.f() eq 20
	JustOne.g() eq 40
}
```

- object 키워드가 객체의 구조를 정의하는 동시에 객체를 생성해버린다.
- object 키워드는 내부 원소들을 object로 정의한 객체의 이름 공간 안에 넣는다.
- object에 대해 파라미터 목록을 지정할 수 없다.
- object는 다른 일반 클래스나 인터페이스를 상속할 수 있다.
    
    ```kotlin
    open class Paint(val color: String) { 
    	open fun apply() = "Applying $color"
    }
    
    object Acrylic: Paint("Blue") { 
    	override fun apply() =
    		"Aer yl i e, ${su per. appl y()}"
    }
    
    interface PaintPreparation { 
    	fun prepare(): String
    }
    
    object Prepare: PaintPreparation { 
    	override fun prepare( ) = "Scrape"
    }
    
    fun main() {
    	Prepare.prepare() eq "Scrape"
    	Paint("Green").apply() eq "Applying Green" 
    	Acrylic.apply() eq "Acrylic, ApplyingBlue"
    }
    ```
    

- object의 인스턴스는 단 하나뿐이므로 이 인스턴스가 object를 사용하는 모든 코드에서 공유된다.
- object는 함수 안에 넣을 수 없지만, 다른 object나 클래스 안에 object를 내포시킬 수는 있다.
- 클래스 안에 객체를 넣는 또 다른 방법으로 compaion object가 있다.

## 내부 클래스

> 내부 클래스는 내포된 클래스와 비슷하지만, 내부 클래스의 객체는 자신을 둘러싼 클래스의 인스턴스에 대한 참조를 유지한다.
> 
- inner 클래스는 자신을 둘러싼 클래스의 객체에 대한 암시적 링크를 갖고 있다.

```kotlin
class Hotel(private val reception: String) {
	open inner class Room(val id: Int = 0) {
		// Room욜 둘러싼 클래스의 'reception’ 을 사용한다 
		fun callReception() =
			"Room $id Calling $reception"
	}
	private inner class Closet : Room() 
	fun closet(): Room = Closet()
}

fun main() {
	val nycHotel = Hotel( "311")
	// 내부 클래스의 인스턴스를 생성하려면
	// 그 내부 클래스를 둘러싼 클래스의 인스턴스가 필요하다 
	val room = nycHotel.Room(319)
	room.callRece pti on() eq
		"Room 319 Calling 311"
	val sfHotel = Hotel("0")
	val closet = sfHotel.closet()
	closet.callReception() eq "Room 0 Calling 0"
}
```

- 내포된 클래스는 inner 클래스를 상속할 수 없다.
- inner 클래스의 객체는 자신과 연관된 외부 객체에 대한 참조를 유지한다.
- 따라서 inner 클래스의 객체를 생성하려면 외부 객체를 제공해야 한다.
- 코틀린은 inner data 클래스를 허용하지 않는다.

- 한정된 this
    - 멤버 함수나 프로퍼티에 접근할 때는 ‘현재 객체’ 를 명시할 필요가 없다.
    - inner 클래스에서 this는 inner 객체나 외부 객체를 가리킬 수 있다.
    → 이 문제를 해결하기 위해 코틀린에서는 한정된 this 구문을 사용한다.
    (한정된 this는 this 뒤에 @을 붙이고 대상 클래스 이름을 덧붙인 것)
    
    ```kotlin
    class Fruit { // @Fruit라는 레이블이 암시적으로 붙는다 
    	fun changeColor(color: String) =
    		"Fruit $color"
    	fun absorbWater(amount: Int) {}
    	inner class Seed {  // @Seed라는 레이블이 암시적으로 붙는다
    		fun changeColor(color: String) = 
    			"Seed $color"
    		fun germinate() {} 
    		fun whichThis() {
    			// 디폴트로 (가장 안쪽의) 현재 클래스를 가리킨다
    		this.name eq "Seed"
    		// 명확히 하기 위해 디폴트 this를
    		// 한정시킬 수 있다
    		this@Seed.name eq "Seed"
    		// name이 Fruit와 Seed에 다 있기 때문에 
    		// Fruit를 명시해 겁근해야 한다
    		this@Fruit.name eq "Fruit"
    		// 현재 클래스의 내부 클래스에 @레이블을 써서 접근할 수는 없다
    		// this@DNA.name
    	}
    	inner class DNA {   // @DNA라는 레이블이 암시적으로 붙는다 
    		fun changeColor(color: String) {
    			// changeColor(color) // 재귀 호출 
    			this@Seed.chagneColor(color)
    			this@Fruit.changeColor(color)
    		}
    		fun plant() {
    			// 한정을 시키지 않고 외부 클래스의 
    			// 함수를 호출할 수 있다
    			germinate()
    			absorbWater(10)
    		}
    		// 확장 함수
    		fun Int.grow() { // @grow라는 암시적 레이블이 붙는다
    			// 디폴트는 Int.grow()로, 'Int’를 수신 객체로 받는다 
    			this.name eq "Int"
    			// @grow 한정은 없어도 된다
    			this@grow.name eq "Int"
    			// 여기서도 여전히 모든 프로퍼티에 접근할 수 있다 
    			this@DNA.name eq "DNA"
    			this@Seed.name eq "Seed"
    			this@Fruit.name eq "Fruit"
    		}
    		// 외부 클래스에 대한 확장 함수들
    		fun Seed.plant() {} 
    		fun Fruit.plant() {} 
    		fun whichThis() {
    			// 디폴트는 현재 클래스이다
    			this.name eq "DNA"
    			// @DNA 한징은 없어도 된다
    			this@DNA.name eq "DNA"
    			// 다른 클래스 한정은 꼭 명시해야 한다 
    			this@Seed.name eq "Seed"
    			this@Fruit.name eq "Fruit"
    		}
    	}
    }
    
    // 확장 함수
    fun Fruit.grow(amount: Int) {
    	absorbWater(amount)
    	// Fruit의 'changeColor( )'를 호출한다 
    	changeColor("Red") eq "Fruit Red"
    }
    // 내부 클래스를 확장한 함수
    fun Fruit.Seed.grow(n: Int) {
    	germinate()
    	// Seed의 changeColor를 호출한다 
    	changeColor("Green") eq "Seed Green"
    }
    // 내부 클래스의 확장 함수
    fun Fruit.Seed.DNA.grow(n: Int) = n.grow()
    
    fun main() {
    	val fruit = Fruit()
    	fruit.grow(4)
    	val seed = fruit.Seed() 
    	seed.grow(9)
    	seed.whichThis()
    	val dna = seed.DNA() 
    	dna.plant()
    	dna.grow(S)
    	dna.whichThis()
    	dna.changeColor("Purple")
    }
    ```
    
    - Fruit, Seed, DNA는 모두 changeColor() 라는 함수를 제공하고 오버라이드 하지 않는다. 세 클래스 사이에는 아무 상속 관계가 없다.
    - plant() 안에서는 이름 충돌이 없는 경우 두 외부 클래스에 속한 함수를 아무런 한정 없이 호출할 수 있다.

- 내부 클래스 상속
    - 내부 클래스는 다른 외부 클래스에 있는 내부 클래스를 상속할 수 있다.
    
    ```kotlin
    open class Egg {
    	private var yolk = Yolk() 
    	open inner class Yolk {
    		init {trace("Egg.Yolk()") }
    		open fun f() { trace("Egg.Yolk.f()") } }
    	}
    	init { trace("New Egg()") }
    	fun insertYolk(y: Yolk) { yolk = y } 
    	fun g() { yolk. f() }
    }
    
    class BigEgg : Egg() {
    	inner class Yolk : Egg.Yolk() {
    		init { trace("Big Egg.Yolk()") 
    		override fun f() {
    			trace("BigEgg.Yolk.f()")
    		}
    	}
    	init { insert Yolk(Yolk())
    }
    
    fun main() { 
    	BigEgg().g() 
    	trace eq """
    		Egg.Yolk() 
    		NewEgg()
    		Egg.Yolk()
    		BigEgg.Yolk()
    		BigEgg.Yolk.f()
    	"""
    }
    ```
    
    - BiggEgg.Yolk는 명시적으로 Egg.Yolk를 자신의 기반 클래스를 정의하고, Egg.Yolk의 f() 멤버 함수를 오버라이드한다.
    - insertYolk()는 BigEgg가 자신의 Yolk 객체를 Egg 에 있는 yolk 참조로 업캐스트하게 허용
    → g() 가 yolk.f() 를 호출하면 오버라이드된 f() 가 호출된다.
    - Egg.Yolk()에 대한 두 번째 호출은 BigEgg.Yolk 생성자에서 호출한 기반 클래스 생성자다.

- 지역 내부 클래스와 익명 내부 클래스
    - 멤버 함수 안에 정의된 클래스를 지역 내부 클래스라고 한다.
    - 이런 클래스를 객체 식이나 SAM 변환을 사용해 익명으로 생성할 수도 있다.
    - 모든 경우에 inner 키워드를 사용하지는 않지만, 이들은 암시적으로 내부 클래스가 된다.
    
    ```kotlin
    fun interface Pet { 
    	fun speak(): String
    }
    
    object CreatePet {
    	fun home() = " home!" 
    	fun dog(): Pet {
    		val say = "Bark" 
    		// 지역 내부 클래스 
    		class Dog : Pet {
    			override fun speak() = say + home() 
    		}
    		return Dog()
    	}
    	fun cat(): Pet{
    		val emit = "Meow"
    		// 익명 내부 클래스 
    		return object: Pet {
    			override fun speak() = emit + home() 
    		}
    	}
    	fun hamster(): Pet {
    		val squeak = "Squeak"
    		// SAM변환
    		return Pet { squeak + home() }
    	} 
    }
    
    fun main() {
    	CreatePet.dog().speak() eq "Bark home!"
    	CreatePet.cat().speak() eq "Meow home! " 
    	CreatePet.hamster().speak() eq "Squeak home!"
    }
    ```
    
    - 지역 내부 클래스는 함수에 정의된 다른 원소와 함수 정의를 포함하는 외부 클래스 객체의 원소에 모두 접근할 수 있다.
    - say, emit, squeak과 home()을 speak() 안에서 쓸 수 있다.
    - 내부 클래스는 외부 클래스 객체에 대한 참조를 저장하므로 지역 내부 클래스도 자신을 둘러싼 클래스에 속한 객체의 모든 멤버에 접근할 수 있다.
    - SAM 변환은 한계가 있다.
        - 예를 들어 SAM 변환으로 선언하는 객체 내부에는 init 블록이 들어갈 수 없다.
- 코틀린에서는 한 파일 안에 여러 최상위 클래스나 함수를 정의할 수 있다.
- 이로 인해 지역 클래스를 사용할 필요가 거의 없다.
- 따라서 지역 클래스로는 아주 기본적이고 단순한 클래스만 사용해야 한다.
    - 예를 들어 함수 내부에서 간단한 data 클래스를 정의해 쓰는 것은 합리적이며, 지역 클래스가 복잡해지면 이 클래스를 함수에서 꺼내 일반 클래스로 격상시켜야 한다.

## 동반 객체

> 멤버 함수는 클래스의 특정 인스턴스에 작용한다.
일부 함수는 어떤 객체에 ‘대한’ 함수가 아닐 수 있다.
이런 함수는 특정 객체에 매에 있을 필요가 없다.
> 
- companion object(동반 객체) 안에 있는 함수와 필드는 클래스에 대한 함수와 필드다.
- 일반 클래스의 원소는 동반 객체의 원소에 접근할 수 있지만, 동반 객체의 원소는 일반 클래스의 원소에 접근할 수 없다.
    - 동반 클래스의 멤버는 동반 클래스의 인스턴스에 대해 작용하므로 동반 객체의 함수나 프로퍼티가 동반 클래스의 멤버에 접근하려면 반드시 동반 클래스의 인스턴스를 함수 파라미터로 받거나 해야 한다.
- 클래스 안에 object를 정의할 수 있다.
- 하지만 일반 내포 객체 정의는 내포 object와 그 객체를 둘러싼 클래스 사이의 연관 관계를 제공하지 않는다.
- 특히 내포된 object의 멤버를 클래스 멤버에서 참조해야 할 때 내포된 object의 이름을 항상 명시해야 한다.

```kotlin
class WithCompanion{ 
	companion object {
		val i = 3
		fun f() = i * 3 
	}
	fun g() = i + f()
}

fun WithCompanion.Companion.h() = f() * i

fun main() {
	val wc = WithCompanion() 
	we.g() eq 12 
	WithCompanion.i eq 3 
	WithCompanion.f() eq 9 
	WithCompanion.h() eq 27
}	
```

- 동반 객체는 클래스당 하나만 허용된다.
- 명확성을 위해 동반 객체에 이름을 부여할 수도 있다.
- 동반 객체 안에서 프로퍼티를 생성하면 해당 필드는 메모리상에 단 하남나 존재하게 되고, 동반 객체와 연관된 클래스의 모든 인스턴스가 이 필드를 공유한다.
- 동반 객체의 사용법 중 흔한 예로 객체 생성을 제어하는 경우를 들 수 있다.
    - 이 방식은 팩토리 메서드 패턴에 해당
- 동반 객체의 생성자는 동반 객체를 둘러싼 클래스가 최초로 프로그램에 적재될 때 이뤄진다.
