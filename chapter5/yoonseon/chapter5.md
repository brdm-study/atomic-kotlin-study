# ✅ 55. 인터페이스

---

- 인터페이스는 클래스가 무엇을 하는지만 기술하고, 어떻게 하는지는 기술하지 않는다.
- 인터페이스는 존재의 목표나 임무를 기술하며, 클래스는 세부적인 구현 사항을 포함한다.

```kotlin
interface Computer {
    fun prompt(): String
    fun calculateAnswer(): Int
}

class Desktop() : Computer {
    override fun prompt() = "Hello"
    override fun calculateAnswer() = 11
}

class DeepThought() : Computer {
    override fun prompt() = "Thinking..."
    override fun calculateAnswer() = 42
}

class Quantum() : Computer {
    override fun prompt() = "Probably..."
    override fun calculateAnswer() = -1
}

fun main() {
    val computers = listOf(Desktop(), Quantum(), DeepThought())
    val calculateAnswers = computers.map { it.calculateAnswer() } // [11, 42, -1]
    val prompts = computers.map { it.prompt() } // [Hello, Thinking..., Probably...]
}
```

- Computer는 prompt()와 calculateAnswer()를 **선언**하지만 구현하지 않음
- 인터페이스를 구현한 클래스는 인터페이스가 선언한 모든 멤버를 구현하는 본문을 제공해야함
- 인터페이스 멤버를 구현할 때는 반드시 `override` 키워드를 붙여야 함

### 인터페이스가 프로퍼티를 선언할 수 있다.

```kotlin
interface Player {
    val symbol: Char
}

class Food: Player {
    override val symbol: Char = '.'
}

class Robot: Player {
    override val symbol get() = 'R'
}

class Wall(override val symbol: Char): Player

fun main() {
    listOf(Food(), Robot(), Wall('|'))
        .map { it.symbol } // [., R, |]
}
```

- 각 하위 클래스는 symbol 프로퍼티를 다른 방법으로 오버라이드 함
    - Food : symbol 값을 직접 다른 값으로 바꿈
    - Robot : 값을 반환하는 커스텀 게터를 사용
    - Wall : 생성자 인자 목록에서 symbol을 오버라이드 함

### Enum도 인터페이스를 구현할 수 있다.

```kotlin
interface Hotness {
    fun feedback(): String
}

enum class SpiceLevel : Hotness{
    Mild {
        override fun feedback() = "It adds flavor!"
    },
    Medium {
        override fun feedback() = "It it warm in here?"
    },
    Hot {
        override fun feedback() = "I'm in suddenly sweating a lot."
    }
}

fun main() {
    val feedbacks = SpiceLevel.values()
        .map { it.feedback() }
    feedbacks.forEach(::println)
    // 출력
    // It adds flavor!
    // It it warm in here?
    // I'm in suddenly sweating a lot.
}
```

- 컴파일러는 각각의 enum 원소들이 feedback()을 구현하는지 확인함 → 안하면 컴파일에러

## SAM 변환

- 단일 추상 메서드(Single abstract Method, SAM) 인터페이스는 자바 개념
- 코틀린에는 SAM 인터페이스를 `fun interface` 키워드로 선언한다.

    ```kotlin
    fun interface ZeroArg {
        fun f(): Int
    }
    ```

- 두 개 이상의 추상 메서드를 선언하면 컴파일 에러가 발생한다.

    ```kotlin
    fun interface ZeroArg {
        fun f(): Int
        fun two(): Int // 컴파일 에러 Fun interfaces must have exactly one abstract method
    } 
    ```


- SAM 인터페이스를 일반적인 클래스를 통해 구현할 수도 있고, 람다를 넘기는 방식으로 구현할 수도 있다.
    - 람다를 사용하는 경우 SAM 변환(SAM conversion)이라고 함
    - SAM 변환을 쓰면 람다가 인터페이스의 유일한 메서드를 구현하는 함수가 된다.

    ```kotlin
    fun interface ZeroArg {
        fun f(): Int
    }
    
    fun interface OneArg {
        fun g(n: Int): Int
    }
    
    fun interface TwoArg {
        fun h(i: Int, j:Int): Int
    }
    
    class VerboseZero : ZeroArg {
        override fun f() = 11
    }
    
    val verboseZero = VerboseZero()
    
    val samZero = ZeroArg { 2 }
    
    class VerboseOne() : OneArg {
        override fun g(n: Int) = n + 47    
    }
    
    val verboseOne = VerboseOne()
    
    val samOne = OneArg { it + 47 }
    
    class VerboseTwo : TwoArg {
        override fun h(i: Int, j: Int) = i + j
    }
    
    val verboseTwo = VerboseTwo() // 클래스를 통해 구현
    
    val samTwo = TwoArg { i, j -> i + j } // 람다로 바로 구현
    
    fun main() {
        verboseZero.f() // 11
        samZero.f() // 11
        verboseOne.g(92)
        samOne.g(139)
        verboseTwo.h(1,2) // 3
        samTwo.h(1,2) // 3
    }
    ```

    - 자주 쓰이는 구문은 SAM 변환을 사용해 더 간결한 구문을 작성할 수 있음
    - 객체를 한 번만 쓰는 경우, 억지로 클래스를 정의할 필요가 없어짐

- 람다를 SAM 인터페이스가 필요한 곳에 넘길 수도 있는데, 이때 굳이 먼저 객체로 람다를 둘러쌀 필요도 없다.

    ```kotlin
    fun interface Action {
        fun act()
    }
    
    fun delayAction(action: Action) {
        action.act()
    }
    
    fun main() {
        delayAction { println("main") }
    }
    ```

    - main() 에서는 Action 인터페이스를 구현하는 개첼 대신에 람다를 바로 전달하고 코틀린은 이 람다로부터 자동으로 Action 객체를 만들어준다.

# ✅ 56. 복잡한 생성자

---

- init 블록을 쓰면 객체 생성 중에 init 블록 안의 코드가 실행된다.
- 생성자 파라미터에 var, val이 붙어있지 않더라도 init 블록에서 사용 가능함

    ```kotlin
    private var counter = 0
    
    class Message(text: String) {
        private val content: String,
        
        init {
            counter += 10
            content = "[$counter] $text"
        }
        
        override fun toString() = content
    }
    
    fun main() {
        val m1 = Message("1")
        println(counter) // 10
        val m2 = Message("2")    
        println(counter) // 20
    }
    ```

    - content는 val로 정의되어 있지만 초기화되지 않았는데, 이런 경우 코틀린은 생성자 안의 어느 지점서 **오직 한 번만** 초기화가 일어나도록 보장한다. → 두 번 이상 할당하거나, 값을 할당하지 않을 시 컴파일 오류 발생

- 생성자는 생성자 파라미터 목록과 init 블록을 합친 것이며, 이들은 객체를 생성하는 동안 실행됨
- init을 여럿 정의할 수 있으며, init 블록은 클래스 본문에 정의된 순서대로 실행된다.
- 하지만, init을 여기저기 분산시키면, 유지보수하는데에 문제가 생길 수 있음

*객체 생성 시점에 validate 같은 것들을 init 블록을 사용해 해결할 수 있을 것 같습니다.*

# ✅ 57. 부생성자

---

- 같은 클래스에 속한 객체를 생성하는 방법을 여러 가지 원할 경우 생성자를 ‘오버로드’ 해야함
- 코틀린에서 오버로드한 생성자를 **부생성자**(secondary constructor)라고 함
- 생성자 파라미터 목록과 프로퍼티 초기화, init 블록들을 모두 합한 생성자는 **주생성자**(primary constructor)라고 함
- 부생성자를 만들려면 `constructor` 키워드 다음에 주생성자나 다른 부생성자의 파라미터 리스트와 구별되는 다른 파라미터 목록을 넣어야한다.
- 부 생성자 안에서는 `this` 키워드를 통해 주생성자나 다른 부생성자를 호출함

```kotlin
class WithSecondary(i: Int) {
    init {
        println("주 생성자: ${i}")
    }
    constructor(c: Char) : this(c.code) { // this로 다른 생성자 호출 후 본문 실행
        println("char 부 생성자: ${c}")
    }

    constructor(s: String) : this(s.first()) { // this로 다른 생성자 호출 후 본문 실행
        println("string 부 생성자: ${s}")
    }
}

fun main() {
    WithSecondary("Hello")
    println("----------")
    WithSecondary('C')
}
```

```kotlin
// 출력
주 생성자: 72
char 부 생성자: H
string 부 생성자: Hello
----------
주 생성자: 67
char 부 생성자: C
```

- 부생성자에서 다른 생성자를 호출(this 사용)하는 부분은 생성자 로직 앞에 위치해야 하는데, 생성자 본문이 다른 초기화의 결과에 영향을 받을 수 있기 때문임

### 부 생성자 주의사항

- 주생성자는 언제나 부생성자에 의해 직접 호출되거나, 다른 부생성자 호출을 통해 간접적으로 호출되어야함
- 그렇지 않으면 컴파일 오류가 발생하며, **생성자 사이에 공유되어야 하는 모든 초기화 로직은 반드시 주생성자에 위치해야함**
- 부생성자를 쓸 때 init 블록을 꼭 쓸 필요는 없음. 단 사용하지 않는다면 주생성자 초기화 로직은 동작하지 않음
- 주생성자가 아닌 부생성자는 파라미터에 val이나 var를 덧붙여 프로퍼티로 선언 불가능함 → 주생성자만 가능
- 부생성자에 반환타입을 지정할 수 없음
- 멤버 프로퍼티와 생성자 파라미터가가 동일할경우 this를 사용해서 모호함을 없애야한다.
- 부생성자 본문을 적지 않아도 되지만, this() 호출은 반드시 포함해야한다.

# ✅ 58. 상속

---

코틀린에서의 상속은 콜론을 통해 지정한다.

```kotlin
open class Base

class Derived : Base()
```

- 부모 클래스에는 `open` 키워드를 붙여야하며, `open` 으로 지정하지 않은 클래스는 기본적으로 상속을 허용하지 않음. → 클래스는 기본적으로 상속에 닫혀있음

```kotlin
open class GreatApe {
    val weight = 100.0
    val age = 12
}

open class Bonobo : GreatApe()

class Chimpanzee : GreatApe()

class BonoboB : Bonobo()

fun GreatApe.info() = "wt: ${this.weight} age: ${age}"

fun main() {
    GreatApe().info() // "wt: 100.0 age: 12"
    Bonobo().info() // "wt: 100.0 age: 12"
    Chimpanzee().info() // "wt: 100.0 age: 12"
    BonoboB().info() // "wt: 100.0 age: 12"
}
```

- 💡 info()는 GreatApe의 확장함수로 당연히 GreatApe 타입 객체에서 이 함수를 호출할 수 있다.
    - GreatApe를 상속받는 Bonobo, Chimpanzee, BonoboB 객체에서도 info()를 호출할 수 있다.
    - 3가지 타입은 모두 서로 다른 타입이지만, 코틀린은 이들은 GreatApe와 같은 타입으로 취급한다.
    - 상속은 GreatApe를 상속한 모든 존재가 항상 GreatApe라고 보장하기 때문에  GreatApe의 함수와 프로퍼티를 자손 클래스에서도 여전히 사용할 수 있음

# ✅ 59. 기반 클래스 초기화

---

- 코틀린은 다음 생성자가 호출되도록 보장함으로써 올바른 객체를 생성함.
    - 멤버 객체들의 생성자
    - 파생 클래스에 추가된 객체 생성자
    - 기반 클래스의 생성자

- 부모 클래스에 생성자 파라미터가 있다면, 반드시 자식 클래스의 생성자 인자를 제공해야하며 제공하지 않을 경우 컴파일 에러 발생함

```kotlin
open class GreatApe(
    val weight: Double,
    val age: Int,
)

open class Bonobo(weight: Double, age: Int) 
    : GreatApe(weight, age) // 부모 클래스 생성자

class Chimpanzee(weight: Double, age: Int)
    : GreatApe(weight, age)

class BonoboB(weight: Double, age: Int)
    : GreatApe(weight, age)

fun GreatApe.info() = "wt: ${this.weight} age: ${age}"

fun main() {
    GreatApe(100.0, 12).info() // "wt: 100.0 age: 12"
    Bonobo(110.0, 13).info() // "wt: 110.0 age: 13"
    Chimpanzee(120.0, 14).info() // "wt: 120.0 age: 14"
    BonoboB(130.0, 15).info() // "wt: 130.0 age: 15"
}
```

- 코틀린은 객체에 사용할 메모리를 확보한 후 부모 클래스의 생성자를 먼저 호출하고, 다음으로 자식 클래스의 생성자를 호출하며 맨 마지막에 파생된 클래스의 생성자를 호출한다.

    <aside>
    💡 부모 클래스 생성자 → 파생 클래스 생성자 → … → 맨 마지막 파생 클래스 생성자

    </aside>

- 이런 식으로 모든 생성자 호출은 자신 이전에 생성되는 모든 객체의 올바름에 의존한다.
- 부모 클래스에 부생성자가 있으면 부모 클래스의 주생성자 대신 부생성자를 호출할 수도 있다.

```kotlin
open class Base(
    val i: Int,
)

class Derived: Base {
    constructor(i: Int): super(i) // 부모 클래스 생성자 호출
    constructor(): this(9)
}

fun main() {
    val d1 = Derived(11)
    println(d1.i) // 11
    val d2 = Derived()
    println(d2.i) // 9
}
```

- 부모 클래스 생성자를 호출하려면 `super` 키워드를 사용, 자신의 다른 생성자를 호출할 때는 `this` 키워드 사용

### 인터페이스와 달리 클래스를 상속받을 때 ()를 붙이는 이유

- 자식 클래스 객체를 생성하는 중에 부모 클래스 생성자를 호출하기 위해서임
- 기반 클래스 생성자 파라미터가 없어도 코틀린은 부모 클래스의 생성자를 인자 없이 호출하기 위해 부모 클래스 이름 뒤에 괄호를 붙이도록 강제함

# ✅ 60. 추상 클래스

---

- 추상 클래스는 하나 이상의 프로퍼티나 함수가 불완전하다는 점을 제외하면 일반 클래스와 같음
    - 불완전한 정의? : 본문이 없는 함수의 정의나 초깃값 대입을 하지 않은 프로퍼티 정의
- 인터페이스는 추상 클래스와 비슷하지만, 인터페이스에는 추상 클래스와 달리 상태가 없음
- 클래스 멤버에서 본문이나 초기화를 제거하려면 `abstract` 변경자를 해당 멤버 앞에 붙여야 함

```kotlin
abstract class WithProperty {
    abstract val x: Int
}

abstract class WithFunctions {
    abstract fun f() : Int
    abstract fun g(n: Double)
}
```

- WithProperty는 초깃값이 없는 `x`를 선언하는데, 코틀린에서는 초기화 코드가 없으면 해당 참조를 `abstract`로 선언해야 하며, 그 참조가 속한 클래스에도 `abstract`를 붙여야 함
- WithFunctions은 `f()`와 `g()`를 선언하지만 정의를 하지 않는데, 이 경우에도  `abstract` 를 붙여야하고 클래스 앞에도 `abstract` 를 붙여야 함
    - `g()` 처럼 반환 타입이 없으면 코틀린은 `Unit` 타입을 반환한다고 간주한다.

> 인터페이스에 정의된 함수나 프로퍼티는 기본적으로 추상 멤버이기에 추상 클래스와 비슷하지만, 추상 클래스는 상태를 갖고 **인터페이스는 상태를 갖지 않는다.**(인터페이스에서 값을 저장하는 것은 금지되있음)
>

### 인터페이스와 추상클래스 모두 구현이 있는 함수를 포함할 수 있다.

```kotlin
interface PropertyAccessor {
    val a: Int
        get() = 11
}

class Impl : PropertyAccessor

fun main() {
    println(Impl().a) // 11
}
```

### 추상 클래스가 있는데 왜 인터페이스가 필요한가?

코틀린에서는 클래스가 오직 한  부모 클래스만 상속 받을 수 있으며 인터페이스만 다중 상속을 허용함

```kotlin
interface A {
    fun f() = 1
    fun g() = "A.g"
    val n: Double
        get() = 1.1
}

interface B{
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
    val c = C()
    println( c.f() ) //  0
    println( c.g() ) //  "A.g"
    println( c.n ) //  3.3
}
```

- 인터페이스 `A` 와 `B` 는 함수 `f()` , `g()` 와 프로퍼티 `n` 의 시그니처가 같으므로, 이 문제를 해결하지 않으면 컴파일 오류가 발생함
- `f()` 처럼 멤버 함수나 프로퍼티를 오버라이드할 수도 있고, 함수에서는 `super` 키워드를 사용해 부모 클래스의 함수를 호출할 수도 있음
- 이때 `C.g()` 나 `C.n` 의 정의처럼 어떤 기반 클래스의 멤버를 호출할지 표시하기 위해 `super` 뒤에 다이아몬드 연산자로 클래스 이름을 지정해야함
- 코틀린은 식별자가 같은데 타입이 다른 식으로 충돌이 일어나는 경우를 허용하지 않으며, 이를 해결해주지 않음

# ✅ 업캐스트

---

객체 참조를 받아서 그 객체의 기반 타입에 대한 참조처럼 취급하는 것을 ‘업캐스트(upcast)한다’ 라 말한다.

```kotlin
interface Shape {
    fun draw(): String
    fun erase(): String
}

class Circle : Shape {
    override fun draw() = "Circle.draw"
    override fun erase() = "Circle.erase"
}

class Square : Shape {
    override fun draw() = "Square.draw"
    override fun erase() = "Square.erase"
}

class Triangle : Shape {
    override fun draw() = "Triangle.draw"
    override fun erase() = "Triangle.erase"
}

fun show(shape: Shape) =
    println( "show: ${shape.draw()}" )

fun main() {
    show(Circle())   // show: Circle.draw
    show(Square())   // show: Square.draw
    show(Triangle()) // show: Triangle.draw
}
```

- show()의 파라미터는 부모 클래스인 Shape이므로, show()는 Circle, Square, Triangle 타입을 모두 허용
- 각 타입은 모두 기반 Shape 클래스의 객체처럼 취급되고, 이를 일컬어 구체적인 타입이 기반 타입으로 **업캐스트**됐다라고 함(구현체 객체가 어떤 타입인지는 사라지고 Shape로 취급됨)
- 실제로 업캐스트를 사용하지 않는데 상속을 사용하는 모든 경우는 상속을 잘못 사용하고 있는 것임

# ✅ 62. 다형성

---

프로그래밍에서 다형성(polymorphism)은 객체나 멤버의 여러 구현이 있는 경우를 뜻함

```kotlin
open class Pet {
    open fun speak() = "Pet"
}

class Dog : Pet() {
    override fun speak() = "멍"
}

class Cat : Pet() {
    override fun speak() = "야옹"
}

fun talk(pet: Pet) = pet.speak()

fun main() {
    println( talk(Dog()) ) // 멍
    println( talk(Cat()) ) // 야옹
}
```

- Dog이나 Cat을 talk()에 넘길 때 모두 Pet으로 업캐스팅 되고 각 객체는 Pet으로 취급된다.
- 다형성은 부모 클래스 참조가 자식 클래스에 인스턴스를 가리키는 경우 발생함
- 부모 클래스 참조에 대해 멤버를 호출하면 다형성에 의해 자식 클래스에서 오버라이드한 올바른 멤버가 호출됨
- 다형성이 사용되는경우 같은 연산이 타입에 따라 다르게 작동하지만 컴파일러는 어떤 함수 본문을 사용해야할지 미리 알 수 없어서 함수 본문을 **동적 바인딩**을 통해 실행 시점에 동적으로 결정해야함

# ✅ 63. 합성

---

- 객체 지향 프로그래밍에서 새 클래스를 만듦으로써 코드를 재사용함(변경 시 한 곳만 수정)
- 하지만 새로 클래스를 만드는 대신 누군가 만들고 디버깅해둔 기존 클래스를 사용함
- 여기서 핵심은 기존 코드를 더럽히지 않고 클래스를 재사용하는 것임
- 이를 달성하는 방법중 하나는 **상속**인데, 상속을하면 기존 클래스 타입에 속하는 새 클래스를 만들고 기존 클래스를 변경없이 기존 클래스의 형식대로 새 클래스에 코드를 추가하게됨
- 또는 기존 클래스 객체를 새 클래스 안에 생성하는 좀 더 직접적인 접근 빙법을 택할 수 있으며, 새 클래스가 기존 클래스들을 합성한 객체로 이뤄지기 때문에 이런 방식을 **합성**이라고 부름

- 합성은 포함관계며 ‘집은 건물이며, 부얶을 포함한다’라는 관계를 아래와 같이 표현할 수 있음

```kotlin
interface Building
interface Kitchen

interface House: Building {
    val kitchen: Kitchen
}
```

### 상속보다는 합성을 선택해라

- 합성이 상속보다 구현이 더 간단하다.
    - 합성을 사용하면 해당 설계를 더 단순하게 만들 수 있는지 검토해봐야함
- 합성은 뻔해 보이지만 강력하다.
    - 클래스가 성장하면 여러 가지 관련이 없는 요소를 책임져야 하는데 합성은 각 요소를 서로 분리할 때 도움을 준다.
    - 합성을 사용하면 클래스의 복잡한 로직을 단순화할 수 있다.

## 합성과 상속 중 선택하기

- 합성은 명시적으로 하위 객체를 선언하지만, 상속은 암시적으로 하위 객체가 생긴다는 점이 다르다.
- 합성은 기존 클래스의 기능을 제공하지만 인터페이스를 제공하지는 않음
- 여러분은 새 클래스에서 객체의 특징을 사용하고 싶어 객체를 포함시키지만, 사용자는 합성으로 포함된 객체의 인터페이스가 아니라 여러분이 새 클래스에서 정의한 인터페이스를 보게됨
- 합성한 객체를 완전히 감추고 싶다면 비공개(private)로 포함시키면 됨

```kotlin
class Features {
    fun f1() = "feature1"
    fun f2() = "feature2"
}

class Form {
    private val features = Features() // 합성 객체 비공개
    fun operation1() = 
        features.f2() + features.f1()
    fun operation2() = 
        features.f1() + features.f2()
}
```

- Features 클래스는 Form의 연산에 대한 구현을 제공하지만 Form을 사용하는 개발자는 features에 접근할 수 없음
- 즉, Form을 구현하는 더 나은 방법을 찾아내면 feature를 제거하고 새 접근 방법을 택해도 Form을 사용하는 코드에는 영향이 미치지 않는다는 뜻임
- 때로는 클래스 사용자가 여러분이 만든 새 클래스의 합성에 직접 접근하는게 합리적인 경우가 있는데 이런 경우 멤버 객체를 공개(public)로 만들 수 있음

```kotlin
class Engine {
    fun start() = println("엔진 start")
    fun stop() = println("엔진 stop")
}

class Wheel {
    fun inflate(psi: Int) = println("타이어 공기 주입 (${psi})")
}

class Window(
    val side: String,
) {
    fun rollUp() = println("${side} 창문 올림")
    fun rollDown() = println("${side} 창문 내림")
}

class Door(
    val side: String,
) {
    val window = Window(side)
    fun open() = println("${side} 문 열림")
    fun close() = println("${side} 문 닫힘")
}

class Car {
    val engine = Engine()
    val wheel = List(4) { Wheel() }
    val leftDoor = Door("왼쪽")
    val sideDoor = Door("오른쪽")
}

fun main() {
    val car = Car()
    car.leftDoor.open()
    car.leftDoor.window.rollUp()
    car.wheel[0].inflate(40)
    car.engine.start()
}
```

- Car 합성은 단순히 하부 구현에 속하는 문제가 아니라 문제 분석에 일부분임
- 이런 식으로 내부를 노출시킨 설계는 클라이언트가 클래스를 사용하는 방법을 이해할 때 도움이 되고, 클래스를 만든 사람의 코드 복잡도를 줄여줌

> 다형성의 영리함으로 인해 모든 것을 상속으로 처리해야 할 것 처럼 느끼기 쉽지만, 이 느낌은 설계에 짐이 됨.
실제로 기존 클래스를 사용해 새 클래스를 만들 때 상속을 우선적으로 선택하면 모든것이 불필요하게 복잡해짐.
더 나은 접근 방법은 합성을 먼저 시도하는 것이며, 특히 상속과 합성 중 어느쪽이 더 잘 적용될지 분명히 알 수 없는 경우 합성을 먼저 시도해야 함
>

# ✅ 64. 상속과 확장

---

- 때로는 기존 클래스를 새로운 목적으로 활용하기 위해 새로운 함수를 추가해야할 때가 있음
- 이때 기존 클래스를 변경할 수 없으면 새 함수를 추가하기 위해 상속을 사용해야하는데, 이로 인해 코드를 이해하고 유지보수하기 어려워짐
- 부모 클래스 인터페이스를 확장하기 위해 상속 대신 확장 함수를 사용하면 상속을 사용하지 않고 부모 클래스의 인스턴스를 직접 확장할 수 있음
    - 코틀린 표준 라이브러리의 Sequence 인터페이스는 멤버 함수가 1개만 들어있고 다른 모든 Sequence 함수는 모두 확장임

## 어댑터 패턴

라이브러리에서 타입을 정의하고 그 타입의 객체를 파라미터로 받는 함수를 제공하는 경우가 종종 있음

```kotlin
interface LibType {
    fun f1()
    fun f2()
}

fun utility1(lt: LibType) {
    lt.f1()
    lt.f2()
}

fun utility2(lt: LibType) {
    lt.f2()
    lt.f1()
}
```

- 위 라이브러리를 사용하려면 함수 파라미터가 LibType이기에 기존 클래스를 LibType으로 변환할 방법이 필요하다.

```kotlin
interface LibType {
    fun f1()
    fun f2()
}

fun utility1(lt: LibType) {
    lt.f1()
    lt.f2()
}

fun utility2(lt: LibType) {
    lt.f2()
    lt.f1()
}

open class MyClass {
    fun g() = println( "g(n)" )
    fun h() = println( "h(n)" )
}

fun useMyClass(mc: MyClass) {
    mc.g()
    mc.h()
}

class MyClassAdaptedForLib : MyClass(), LibType {
    override fun f1() = h()
    override fun f2() = g()
}

fun main() {
    val mc = MyClassAdaptedForLib()
    utility1(mc)
    utility2(mc)
    useMyClass(mc)
}
```

- MyClassAdaptedForLib를 만들기 위해 기존 MyClass를 상속받고 MyClassAdaptedForLib는 LibType을 구현하므로 utility1, utility2 함수의 인자로 전달될 수 있다.
    - 이런 방식은 상속을 하는 과정에서 클래스를 확장시키기는 하지만, 새 멤버 함수는 오직 UsefulLibrary에 연결하기 위해서만 쓰임
    - 부모 클래스 사용자가 자식 클래스에 대해 꼭 알아야 하는 방식으로 MyClassAdaptedForLibrary 클래스를 사용하는 코드는 없음

**Myclass가 open이 아니고 수정할 수 없다면?**

```kotlin
interface LibType {
    fun f1()
    fun f2()
}

fun utility1(lt: LibType) {
    lt.f1()
    lt.f2()
}

fun utility2(lt: LibType) {
    lt.f2()
    lt.f1()
}

class MyClass { // open 제거 
    fun g() = println( "g(n)" )
    fun h() = println( "h(n)" )
}

fun useMyClass(mc: MyClass) {
    mc.g()
    mc.h()
}

class MyClassAdaptedForLib : LibType {
    val field = MyClass() // MyClass 필드 추가
    override fun f1() = field.h()
    override fun f2() = field.g()
}

fun main() {
    val mc = MyClassAdaptedForLib()
    utility1(mc)
    utility2(mc)
    useMyClass(mc.field)
}
```

- 이전 코드 만큼 깔끔하지 않으며, useMyClass(mc.field) 호출처럼 명시적으로 MyClass 객체에 접근해야함
- 하지만 이런 방법은 여전히 기존 라이브러리를 새로운 인터페이스에 맞게 전환해 연결하는 문제를 쉽게 해결함
- 확장 함수는 어댑터를 생성할 때 아주 유용할 것 같지만, 불행히도 확장 함수를 모아서 인터페이스를 구현할 수는 없음

## 확장 함수와 멤버 함수 비교

- 함수가 private 멤버에 접근해야 한다면 멤버 함수를 정의할 수 밖에 없다.

    ```kotlin
    class Z(var i: Int = 0) {
        private var j = 0
        fun increment() {
            i++
            j++
        }
    }
    
    fun Z.decrement() {
        i--
        j-- // 접근할 수 없음
    }
    ```


- 확장 함수의 가장 큰 한계는 오버라이드 할 수 없다는 점임
- 함수를 오버라이드할 필요가 없고 클래스의 공개 맴버만으로 충분할 때는 이를 멤버 함수로 구현할 수도 있고, 확장 함수로 구현할 수도 있음. → 스타일의 문제일 뿐, 코드의 명확성을 가장 크게 높힐 수 있는 방법을 선택해야함
- 멤버 함수는 타입의 핵심을 반영하는데, 확장 함수는 대상 타입을 지원하고 활용하기 위한 외부 연산이나 편리를 위한 연산임.
- 타입 내부에 외부 함수를 포함하면 타입을 이해하기가 더 어려워지지만, 일부 함수를 확장함수로 정의하면 대상 타입을 깔끔하고 단순하게 유지할 수 있음

코틀린은 open 키워드를 사용하지 않으면, 상속과 다형성을 의도적으로 막는다. 이는 코틀린이 나아갈 방향에 대한 통찰을 제공한다.
구체적인 특정 상황에서 상속을 어떻게 사용할지 심사숙고하는 중이라면, 진짜 상속이 필요할지를 고려하고 **상속보다는 확장 함수와 합성을 택해라** 라는 격언을 적용해라.

# ✅ 65. 클래스 위임

---

합성과 상속은 모두 새 클래스 안에 하위 객체를 심는데, 합성에서는 하위 객체가 명시적으로 존재하고, 상속에서는 암시적으로 존재함.

- 클래스가 기존 구현을 재사용하면서 동시에 인터페이스를 구현해야 하는 경우, 상속과 클래스 위임 두 가지 선택지가 있음
    - 클래스 위임은 상속과 합성 중간 지점으로 합성과 마찬가지로 새 클래스 안에 멤버 객체를 심고, 상속과 마찬가지로 심겨진 하위 객체의 인터페이스를 노출시킴
    - 게다가 새 클래스를 하위 객체의 타입에 업캐스트할 수 있음
    - 코드를 재사용하기 위해 클래스 위임은 합성을 상속만큼 강력하게 만듬

```kotlin
interface Controls {
    fun up(velocity: Int): String
    fun down(velocity: Int): String
    fun left(velocity: Int): String
    fun right(velocity: Int): String
    fun forward(velocity: Int): String
    fun back(velocity: Int): String
    fun turboBoost(): String
}

class SpaceShipControls : Controls {
    override fun up(velocity: Int) = "up $velocity"
    override fun down(velocity: Int) = "down $velocity"
    override fun left(velocity: Int) = "left $velocity"
    override fun right(velocity: Int) = "right $velocity"
    override fun forward(velocity: Int) = "forward $velocity"
    override fun back(velocity: Int) = "back $velocity"
    override fun turboBoost() = "turbo boost"
}

```

- 제어 장치의 기능을 확장하거나 명령을 일부 조정하고 싶다면 SpaceShipControls를 상속하려 하겠지만 SpaceShipControls는 open이 아니므로 상속할 수 없음.
- Controls의 멤버 함수를 노출하려면 SpaceShipControls의 인스턴스를 프로퍼티화 하고 Controls의 모든 멤버 함수를 명시적으로 SpaceShipControls에 위임 해야함

    ```kotlin
    class ExplicitControls : Controls {
        private val controls = SpaceShipControls()
        // 수동으로 위임 구현하기
        override fun up(velocity: Int) = controls.up(velocity)
        override fun back(velocity: Int) = controls.back(velocity)
        override fun down(velocity: Int) = controls.down(velocity)
        override fun forward(velocity: Int) = controls.forward(velocity)
        override fun left(velocity: Int) = controls.left(velocity)
        override fun right(velocity: Int) = controls.right(velocity)
        // 변형한 구현
        override fun turboBoost(): String = controls.turboBoost() + "... boooooost!"
    }
    
    fun main() {
        val controls = ExplicitControls()
        println(controls.forward(100) == "forward 100")
        println(controls.turboBoost() == "turbo boost... boooooost!")
    }
    ```


- 코틀린은 위와 같은 클래스 위임 과정을 자동화해줌
- 클래스를 위임하려면 `by` 키워드를 인터페이스 이름 뒤에 넣고 by 뒤에 위임할 멤버 프로퍼티의 이름을 넣음

```kotlin
class ExplicitControls(
    private val controls: Controls = SpaceShipControls()
) : Controls by controls {
    override fun turboBoost(): String = controls.turboBoost() + "... boooooost!"
}

fun main() {
    val controls = ExplicitControls()
    println(controls.forward(100) == "forward 100")
    println(controls.turboBoost() == "turbo boost... boooooost!")
}

```

- 위 코드를 ‘클래스 ExplicitControls는 Controls 인터페이스를 controls 멤버 객체를 사용해(by) 구현한다’ 라고 읽음
    - ExplicitControls() by controls라고 by 앞에 클래스 이름을 쓸 수는 없음
    - 위임 객체(controls)는 생성자 인자로 지정한 프로퍼티여야 함
- 위임을 하면 별도로 코드를 작성하지 않아도 멤버 객체의 함수를 외부 객체를 통해 접근할 수 있음

- 코틀린에서 다중 클래스 상속을 허용하지 않지만, 클래스 위임을 사용해 다중 클래스 상속을 흉내낼 수는 있음
    - 일반적으로 다중 상속은 전혀 다른 기능을 가진 여러 클래스를 하나로 묶기 위해 쓰임

```kotlin
interface Rectangle {
    fun paint(): String
}

class ButtonImage(
    val width: Int,
    val height: Int,
): Rectangle {
    override fun paint() = "painting ButtonImage(${width}, ${height})"
}

interface MouseManager {
    fun clicked(): Boolean
    fun hovering(): Boolean
}

class UserInput: MouseManager {
    override fun clicked() = true
    override fun hovering() = true
}

class Button(
    val width: Int,
    val height: Int,
    var image: Rectangle = ButtonImage(width, height),
    private val input: MouseManager = UserInput()
): Rectangle by image, MouseManager by input

fun main() {
    val button = Button(10, 5)
    println( button.paint() ) // painting ButtonImage(10, 5)
    println( button.clicked() ) // true
    println( button.hovering() ) // true

    // 위임한 두 타입으로 업캐스트가 모두 가능하다.
    val rectangle: Rectangle = button
    val mouseManager: MouseManager = button
}
```

- Button 클래스는 두 인터페이스 Rectangle, MouseManager를 구현하는데, Button이 ButtonImage와 UserInput 구현을 모두 상속할 수는 없지만, 두 클래스를 모두 위임할 수는 있음
- 생성자 인자 목록의 image 정의가 public인 동시에 var라는 점에 유의해야하는데, 이로 인해 클라이언트 개발자가 동적으로 ButtonImage를 변경할 수 있음
- main()의 마지막 두 줄은 Button을 자신이 위임한 2가지 타입으로 업캐스트 할 수 있음을 보여주며 이것이 다중 상속의 목표임. 결과적으로 이임은 다중 상속의 필요성을 해결해줌

> 상속은 상위클래스가 open이 아니거나 새 클래스가 다른 클래스를 이미 상속하고 있는 경우에 다른 클래스를 상속할 수 없지만 클래스 위임을 사용하면 이런 제약을 포함한 여러 제약을 피할 수 있음
>

> 클래스 위임을 조심히 사용하고, 상속, 합성, 클래스 위임 중에서 합성을 제일 먼저 시도해야함.
합성은 가장 단순한 방법이며 대부분의 유스케이스를 해결해줌
타입 계층과 이 계층에 속한 타입 사이의 관계가 필요할 때는 상속이 필요함.
이 두가지 방법이 적합하지 않을 때 위임을 쓸 수 있음.
>

# ✅ 66. 다운캐스트

---

다운캐스트는 이전에 업캐스트 했던 객체의 구체적인 타입으로 형변경하는 것임

- 부모 클래스가 자식 클래스보다 더 큰 인터페이스를 가질 수 없으므로 업캐스트는 항상 안전함
- 다운캐스트는 자식 클래스에 특정한 추가적인 함수나 속성에 접근하려는 경우에만 의미가 있으며, 런타임 시에 실제 객체가 해당 자식 클래스의 인스턴스인지 확인할 필요가 있음

## 스마트 캐스트

- 코틀린 스마트 캐스트는 자동 다운캐스트로 `is` 키워드를 사용하여 어떤 객체가 특정 타입인지 검사함
- 이 검사 영역 안에서는 해당 객체를 검사에 성공한 타입이라고 간주함

```kotlin
fun main() {
    val b1: Base = Derived1() // 업캐스트
    if (b1 is Derived1) {
        b1.g() // b1이 Derived1 타입이면 Derived1타입의 g()를 다운캐스팅없이 바로 호출 가능
    }
    if (b1 is Derived2) {
        b2.h() // b2가 Derived2 타입이면 Derived2타입의 h()를 다운캐스팅없이 바로 호출 가능
    } 
}
```

- 스마일 캐스트는 is를 통해 when의 인자가 어떤 타입인지 검색하는 when 식 내부에서 유용하다.

```kotlin
interface Creature

class Human : Creature {
    fun greeting() = "I'm Human"
}

class Dog : Creature {
    fun bark() = "Yip!"
}

class Alien : Creature {
    fun mobility() = "Three legs"
}

fun what(c: Creature): String = when (c) {
    is Human -> c.greeting()
    is Dog -> c.bark()
    is Alien -> c.mobility()
    else -> "Something else"
}

fun main() {
    val c: Creature = Human()
    println(what(c) == "I'm Human")
    println(what(Dog()) == "Yip!")
    println(what(Alien()) == "Three legs")

    class Who : Creature
    println(what(Who()) == "Something else")
}

```

## 변경 가능한 참조

- 스마일캐스트는 대상이 상수여야만 자동으로 작동한다.

## as 키워드

- `as` 키워드는 일반적인 타입을 구체적인 타입으로 강제 변환함
- `as` 가 실패하면 ClassCastException이 발생함
- 일반 `as` 를 **안전하지 않은 캐스트**라 부름

```kotlin
open class Animal

class Cat : Animal() {
    fun meow() = "Meow!"
}

class Dog : Animal() {
    fun bark() = "Bark!"
}

fun main() {
    val animal: Animal = Cat() // 업캐스트
    
    // 일반 `as` 캐스팅
    try {
        val cat: Cat = animal as Cat // 성공적인 캐스팅
        println(cat.meow()) // "Meow!"
        
        val dog: Dog = animal as Dog // 실패할 캐스팅
        println(dog.bark()) // 이 줄은 실행되지 않음
    } catch (e: ClassCastException) {
        println("Failed to cast: ${e.message}") // "Failed to cast: Animal cannot be cast to Dog"
    }
}

```

- **안전한 캐스트**인 `as?` 는 실패해도 예외를 던지지 않는 대신 null을 반환한다.
- `as?` 에서 NPE를 방지하려면 엘비스 연산자를 사용하여 직접적으로 처리할 수 있다.

```kotlin
open class Animal

class Cat : Animal() {
    fun meow() = "Meow!"
}

class Dog : Animal() {
    fun bark() = "Bark!"
}

fun main() {
    val animal: Animal = Cat() // 업캐스트

    // 안전한 캐스팅 `as?`
    val cat: Cat? = animal as? Cat
    println(cat?.meow() ?: "Not a Cat") // "Meow!" - 안전하게 호출 가능

    val dog: Dog? = animal as? Dog
    println(dog?.bark() ?: "Not a Dog") // "Not a Dog" - 안전하게 처리

    // 엘비스 연산자를 사용하여 `null`을 다른 값으로 대체
    val defaultMessage = dog?.bark() ?: "No bark available"
    println(defaultMessage) // "No bark available"
}

```

## 리스트의 원소타입 알아내기

- 술어에서 is를 사용하면 List나 다른 이터러블의 원소가 주어진 타입의 객체인지 알 수 있음

```kotlin
fun main() {
    val group: List<Creature> = listOf(Human(), Human(), Dog(), Alien(), Dog())
    
    val dog = group
        .find { it is Dog } as? Dog // [1]
    
    println(dog?.bark())
}
```

- 위 코드의 경우 보통은 지정한 타입에 속하는 모든 원소를 돌려주는 filterIsInstance()를 써서 [1]과 같은 코드를 피할 수 있음

```kotlin
fun main() {
    val group: List<Creature> = listOf(Human(), Human(), Dog(), Alien(), Dog())
    
    val humans1: List<Creature> = group.filter { it is Human }
    println(humans1.size == 2) // true
    
    // Human 타입의 모든 인스턴스를 필터링
    val humans2: List<Human> = group.filterIsInstance<Human>()
    println(humans2 == humans1) // true
}

```

# ✅ 67. 봉인된 클래스

---

- 클래스 계층을 제한하려면 상위 클래스를 `sealed` 로 선언해라
- `sealed` 클래스를 직접 상속한 하위 클래스는 반드시 상위 클래스와 같은 패키지와 모듈안에 있어야 함

```kotlin
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
        is Train -> "Train ${transport.line}"
        is Bus -> "Bus ${transport.number}: size ${transport.capacity}"
        else -> "${transport} is in limbo!"
    }

fun main() {
    val result = listOf(Train("S1"), Bus("11", 90))
        .map(::travel)
    println(result.toString()) // [Train S1, Bus 11: size 90]
}
```

- Train과 Bus는 Transport 유형에 따라 다른 세부 사항을 저장함
- travel() 에는 정확한 transport 타입을 when을 통해 찾아내며, Transport 클래스에 다른 하위 타입이 있을 수도 있으므로 코틀린은 else 가지를 디폴트로 요구함
- travel()은 다운캐스트가 근본적인 문제가 될 수 있는 지점이라는 사실을 보여줌
    - Transport를 상속한 Tram이라는 클래스를 새로 만들면 travel()은 여전히 컴파일되고 실행도 되기에 Tram 추가에 맞춰 travel()의 when을 바꿔야 한다는 아무런 단서가 없음
    - 코드에서 다운캐스트가 여기저기 흩어져 있다면 이런 변경으로 인해 유지보수가 힘들어짐
- 이 상황은 `sealed` 키워드를 사용해 개선할 수 있으며, `sealed` 키워드로 상속을 제한한 클래스를 봉인된 클래스라고 부름

```kotlin
sealed class Transport // << sealed class로 변경

data class Train(
    val line: String
): Transport()

data class Bus(
    val number: String,
    val capacity: Int,
): Transport()

fun travel(transport: Transport) =
    when (transport) {
        is Train -> "Train ${transport.line}"
        is Bus -> "Bus ${transport.number}: size ${transport.capacity}"
        // else 가지 제거
    }

fun main() {
    val result = listOf(Train("S1"), Bus("11", 90))
        .map(::travel)
    println(result.toString()) // [Train S1, Bus 11: size 90]
}
```

- 코틀린은 when 식이 모든 경우를 검사하도록 강제하지만 travel()의 when은 더 이상 else 가지를 요구하지 않음
    - Transport가 `sealed` 라서 코틀린이 다른 Transport의 하위 클래스가 존재할 수 없다는 사실을 확신할 수 있기 때문
    - 이제 when 문이 가능한 모든 경우를 다 처리하므로 else 가지가 필요 없음
- Transport를 상속한 Tram이라는 클래스를 새로 만들면 travel()은 `sealed class`로 인해 컴파일 오류가 발생하기에 컴파일시점에 문제를 해결할 수 있다.
- `sealed` 키워드는 다운캐스트를 더 쓸만하게 만들어주지만, 다운캐스트를 과도하게 사용하지말고 다형성을 사용해서 코드를 더 깔끔하게 작성할 수 있는 방법이 있음

## sealed와 abstract 비교

```kotlin
package practice.atomic.chapter5

abstract class Abstract(val av: String) {
    open fun concreteFunction() {}
    open val concreteProperty = ""
    abstract fun abstractFunction(): String
    abstract val abstractProperty: String
    constructor(c: Char) : this(c.toString())
    init {}
}

class Concrete : Abstract("") {
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
    constructor(c: Char) : this(c.toString())
    init {}
}

open class SealedSubclass : Sealed(" ") {
    override fun concreteFunction() {}
    override val concreteProperty = ""
    override fun abstractFunction() = ""
    override val abstractProperty = ""
}

fun main() {
    val concrete = Concrete()
    val sealedSubclass = SealedSubclass()
}
```

- sealed 클래스는 기본적으로 하위 클래스가 모두 같은 파일 안에 정의되어야 한다는 제약이 가해진 abstract 클래스임
- sealed 클래스의 간접적인 하위 클래스를 별도의 파일에 정의할 수 있음

    ```kotlin
    class ThirdLevel : SealedSubclass()
    ```


## 하위 클래스 열거하기

- 어떤 클래스가 sealed인 경우 모든 하위클래스를 쉽게 이터레이션할 수 있음

```kotlin
sealed class Top
class Middle1 : Top()
class Middle2 : Top()
open class Middle3 : Top()
class Bottom3 : Middle3

fun main() {
    val result = Top::class.sealedSubclasses
        .map { it.simpleName }
    println( result.toString() ) // [Middle1, Middle2, Middle3]
}
```

- 클래스를 생성하면 클래스 객체가 생성되는데, 이 클래스 객체의 프로퍼티와 멤버 함수에 접근해서 클래스에 대한 정보를 얻고 클래스에 속한 객체를 생성하거나 조작할 수 있음
    - `::class` 가 클래스 객체를 돌려주므로 `Top::class` 에 대한 클래스 객체를 만듬
- 클래스 객체의 프로퍼티에는 sealedSubclasses가 있는데 Top::class로 얻는 클래스 객체에서 이 프로퍼티Top이 sealed 클래스이길 기대하고(그렇지 않다면 빈 리스트를 반환함) sealed 클래스의 모든 하위 클래스를 반환함
- sealedSubclasses는 리플랙션을 사용함

# ✅ 68. 타입 검사

---

- `sealed class`를 이용해 `when`사용시 모든 타입을 검사하도록 보장할 수 있다.
- 필요에 따라 확장 함수와 타입 검사를 이용하거나, `abstract`멤버를 이용해 설계할 수 있다.

# ✅ 69. 내포된 클래스

---

- 내포된 클래스는 단순히 외부 클래스 이름 공간 안에 정의된 클래스일 뿐이며, 외부 클래스의 구현이 내포된 클래스를 ‘소유’함
- 내포된 클래스가 필수적인 기능은 아니지만 코드를 깔끔하게 해줄 때가 있음

```kotlin
class Airport(
    private val code: String
) {
    open class Plane {
        // 자신을 둘러싼 클래스의 private 프로퍼티에 접근할 수 있다.
        fun contact(airport: Airport) =
            "Contacting ${airport.code}"
    }
    private class PrivatePlane : Plane()
    fun privatePlane(): Plane = PrivatePlane()
}

fun main() {
    val denver = Airport("DEN")
    var plane = Plane() // [1]
    println( plane.contact(denver) ) // "Contacting DEN"
    // 다음과 같이 할 수 없다.
    // val privatePlane = Airport.PrivatePlane()
    
    val frankfurt = Airport("FRA")
    plane = frankfurt.privatePlane()
    // 다음과 같이 할 수 없다.
    // val p = plane as PrivatePlane
    println( plane.contact(frankfurt) ) // "Contacting FRA"
}
```

- Plane 객체를 생성할 때 Airport 객체가 필요하지 않지만, Airport 클래스 본문 밖에서 Plane 객체를 생성하려고 한다면 일반적으로 생성자 호출을 한정 시켜야 함(`[1]`)
- PrivatePlane 처럼 내포된 클래스를 private으로 설정 가능하다.

## 지역 클래스

- 함수 안에 내포된 클래스를 지역 클래스라고 함
- 지역 open 클래스는 거의 정의하지 말아야하는데, 지역 클래스로 open 클래스를 정의해야 한다면 아마 일반적인 클래스로 정의할 만큼 그 클래스가 중요한 경우일 것임

## 인터페이스에 포함된 클래스

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
        Bolt("Slotted"),
        Bolt("Hex")
    )
    val result = items.map { it.type }
    println(result == listOf(Item.Type("Slotted"), Item.Type("Hex")))
}

```

- Bolt 안에서는 val type을 반드시 오버라이드하고 Item.Type 이라는 한정시킨 클래스 이름을 써서 값을 대입해야함

## 내포된 이넘

- 이넘도 클래스, 인터페이스 안에 내포될 수 있다.
- 이넘은 함수에 내포시킬수는 없다.
- 이넘이 다른 클래스(다른 이넘 클래스도 마찬가지)를 상속할 수 없다.

```kotlin
interface Game{
    enum class State {Playing, Finished}
    enum class Mark {Blank, X, O}
}
```

# ✅ 70. 객체

---

- `object` 는 여러 인스턴스가 필요하지 않거나 명시적으로 싱글톤 객체를 만들 때 논리적으로 한 개체 안에 속한 함수와 프로퍼티를 함께 엮는 방법임
- `object` 의 인스턴스를 직접 생성하는 경우는 결코 없으며, `object` 를 정의하면 그 `object` 의 인스턴스가 오직 하나만 생김

```kotlin
object JustOne {
    val n = 2
    fun f() = n * 10
    fun g() = this.n * 20
}

fun main() {
    println( JustOne.n ) // 2
    println( JustOne.f() ) // 20
    println( JustOne.g() ) // 40
}
```

- JustOne() 으로 인스턴스를 만들 수 없음
    - `object` 키워드가 객체 주고를 정의하는 동시에 객체를 생성해버리기 때문
- `object` 키워드는 내부 원소를 `object` 로 정의한 객체 이름 공간 안에 넣음.

- `object` 는 다른 일반 클래스나 인터페이스를 상속받을 수 있다.

```kotlin
open class Paint(val color: String) {
    open fun apply() = "Applying ${color}"
}

object Acrylic: Paint("Blue") {
    override fun apply() = "Acrylic, ${super.apply()}"
}

interface PaintPreparation {
    fun prepare(): String
}

object Prepare: PaintPreparation {
    override fun prepare() = "Scrape"
}

fun main() {
    println( Prepare.prepare() ) // Scrape
    println( Paint("Green").apply() ) // Applying Green
    println( Acrylic.apply() ) // Acrylic, Applying Blue
}
```

- `object` 의 인스턴스는 단 하나뿐이므로 이 인스턴스가 object를 사용하는 모든 코드에서 공유됨
    - 다른 패키지에서도 사용할 수 있으며 모두 같은 객체 참조를 사용함
- `object` 는 함수 안에 넣을 수 없지만, 다른 `object`나 클래스안에 `object`를 내포 시킬 수 있음

    ```kotlin
    object Outer {
        object Nested {
            val a = "Outer.Nested.a"
        }
    }
    
    class HasObject {
        object Nested {
            val a = "HasObject.Nested.a"
        }
    }
    
    fun main() {
        println( Outer.Nested.a ) // 
        println( HasObject.Nested.a ) // 
    }
    ```


# ✅ 71. 내부 클래스

---

- 내부 클래스는 내포된 클래스와 비슷하지만, 내부 클래스의 객체는 자신을 둘러싼 클래스 인스턴스에 대한 참조를 유지함

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
    val nycHotel = Hotel("311")
    // 내부 클래스의 인스턴스를 생성하려면
    // 그 내부 클래스를 둘러싼 클래스의 인스턴스가 필요함
    val room = nycHotel.Room(319)
    println( room.callReception() ) // Room 319 Calling 311

    val sfHotel = Hotel("0")
    val closet = sfHotel.closet()
    println( closet.callReception() ) //Room 0 Calling 0
}
```

- inner class의 객체는 자신과 연관된 외부 객체에 대한 참조를 유지하기에 inner 클래스의 객체를 생성하려면 외부 객체를 제공해야함
- 코틀린은 inner data 클래스를 허용하지 않음

## 한정된 this

- inner 클래스의 장점 중 하나는 this 참조를 사용할 수 있다는 점임
- inner 클래스에서 this는 inner 객체나 외부 객체를 가르킬 수 있는데 이 문제를 해결하기 위해 코틀린은 **한정된 this** 구문을 사용함
    - **한정된 this :** this 뒤에 @을 붙이고 대상 클래스 이름을 덧붙인 것

```kotlin
class Outer {
    val outerName = "Outer"

    inner class Inner {
        val innerName = "Inner"

        fun printNames() {
            println(this.innerName) // Inner 클래스의 innerName 참조
            println(this@Outer.outerName) // Outer 클래스의 outerName 참조
        }
    }
}

fun main() {
    val outer = Outer()
    val inner = outer.Inner()
    inner.printNames()
}
```

## 내부 클래스 상속

- 내부 클래스는 다른 외부 클래스에 있는 내부 클래스를 상속할 수 있다.

```kotlin
open class Egg {
    private var yolk = Yolk()
    open inner class Yolk {
        init { println("Egg.Yolk()") }
        open fun f() { println("Egg.Yolk.f()") }
    }
    init { println("New Egg()") }
    fun insertYolk(y: Yolk) { yolk = y }
    fun g() { yolk.f() }
}

class BigEgg: Egg() {
    inner class Yolk: Egg.Yolk() {
        init { println("BigEgg.Yolk()") }
        override fun f() {
            println("BigEgg.Yolk.f()")
        }
    }
    init { insertYolk(Yolk()) }
}

fun main() {
    BigEgg().g()
}
```

```kotlin
// 출력
Egg.Yolk()
New Egg()
Egg.Yolk()
BigEgg.Yolk()
BigEgg.Yolk.f()
```

- BigEgg.Yolk는 명시적으로 Egg.Yolk를 자신의 기반 클래스로 정의하고, Egg.Yolk와 f() 멤버 함수를 오버라이드함
- insertYolk()는 BigEgg가 자신의 Yoklk 객체를 Egg에 있는 yolk 참조로 업캐스트하게 허용하고, 따라서 g()가 yolk.f()를 호출하면 오버라이드된 f()가 호출됨
- Egg.Yolk()에 대한 두 번째 호출은 BigEgg.Yolk 생성자에서 호출한 기반 클래스 생성자임
- g()를 호출하면 오버라이드한 f()가 쓰인다는 점을 알 수 있음

## 지역 내부 클래스와 익명 내부 클래스

- 멤버 함수 안에 정의된 클래스를 지역 내부 클래스라고 하고 이런 클래스를 객체식이나 SAM 변환을 사용해 익명으로 생성할 수도 있음
- 모든 경우에 inner 키워드를 사용하지는 않지만, 이들은 암시적으로 내부 클래스가 된다.

```kotlin
fun interface Pet {
    fun speak(): String
}

object CreatePet {
    fun home() = " home"

    fun dog(): Pet {
        val say = "Bark"
        // 지역 내부 클래스
        class Dog : Pet {
            override fun speak() = say + home()
        }
        return Dog()
    }

    fun cat(): Pet {
        val emit = "Meow"
        // 익명 내부 클래스
        return object : Pet {
            override fun speak() = emit + home()
        }
    }

    fun hamster(): Pet {
        val squeak = "Squeak"
        // SAM 변환
        return Pet { squeak + home() }
    }
}

fun main() {
    val dog = CreatePet.dog()
    println(dog.speak())  // 출력: Bark home

    val cat = CreatePet.cat()
    println(cat.speak())  // 출력: Meow home

    val hamster = CreatePet.hamster()
    println(hamster.speak())  // 출력: Squeak home
}

```

- 지역 내부 클래스는 함수에 정의된 다른 원소와 함수 정의를 포함하는 외부 클래스 객체의 원소에 모두 접근할 수 있음(say, emit, squeak, home()을 speak()안에서 쓸 수 있음)
- `object : 타입이름` 를 사용하여 익명클래스를 구현할 수 있음

> 코틀린에서는 한 파일 안에 여러 최상위 클래스나 함수를 정의할 수 있는데, 이로 인해 지역 클래스를 사용할 일이 거의 없음.
따라서 지역 클래스로는 아주 기본적이고 단순한 클래스만 사용해야 함.
예를 들어 함수 내부에서 간단한 data 클래스를 정의해 쓰는 것은 합리적이며, 지역 클래스가 복잡해지면 이 클래스를 함수에서 꺼내 일반 클래스로 격상 시켜야함
>

# ✅ 72. 동반 객체

---

- 동반 객체는 companion object 라고 부르며 companion object 안에 있는 함수와 필드는 클래스에 대한 함수와 필드임
- 일반 클래스의 원소는 동반 객체에 접근할 수 있지만, 동반 객체 원소는 일반 클래스 원소에 접근할 수 없음
    - *자바의 static이라고 생각하면 됨*

```kotlin
class WithCompanion {
    companion object {
        val i = 3
        fun f() = i * 3
    }
    fun g() = i + f()
}

fun WithCompanion.Companion.h() = f() * i

fun main() {
    val wc = WithCompanion()
    println( wc.g() ) // 12
    println( WithCompanion.i ) // 3
    println( WithCompanion.f() ) // 9
    println( WithCompanion.h() ) // 27
}
```

- `fun WithCompanion.Companion.h()` 와 같이 동반 객체에 대한 확장 함수도 만들 수 있음
- 동반 객체는 클래스당 하나만 허용되며, 명확성을 위해 동반 객체에 이름을 부여할 수 있음(이름을 부여하지 않으면 이름이 Companion으로 취급됨)

    ```kotlin
    class WithCompanion {
        companion object Named { // 동반 객체 이름 부여
            val i = 3
            fun f() = i * 3
        }
        fun g() = i + f()
    }
    
    fun WithCompanion.Named.h() = f() * i // 동반 객체 이름으로 확장 함수 생성
    
    ```


- 동반 객체 안에서 프로퍼티를 생성하면 해당 필드는 메모리상에 단 하나만 존재하게 되고, 동반 객체와 연관된 클래스의 모든 인스턴스가 이 필드를 공유함(*상수 처리*)
- 동반 객체를 사용하는 가장 흔한 방법은 팩토리 메서드 패턴을 사용하는 것임