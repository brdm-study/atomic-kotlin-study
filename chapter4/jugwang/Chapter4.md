## 람다

- 람다에는 이름이 없고, 함수 생성에 필요한 최소한의 코드만 필요하며, 다른 코드에 람다를 직접 삽입할 수 있다.
- 람다를 함수 리터럴이라고 부르기도 한다.

```kotlin
fun main() {
	val list = listOf(1, 2, 3, 4)
	val result = list.map({ n: Int -> "[$n]"})
	result eq listOf("[1]", "[2]", "[3]", "[4]")
}
```

- result를 초기화할 때 중괄호 사이에 쓴 코드가 람다
- 파라미터 목록과 함수 본문 사이에는 - > 가 들어간다.
- 코틀린이 람다의 타입을 추론할 수 있다.
    
    ```kotlin
    list.map({ n: Int -> "[$n]"})
    list.map({ n -> "[$n]"})
    list.map({ "[$it]"})  //파라미터가 하나일 경우 코틀린은 자동으로 파라미터 이름을 it 
    ```
    
- 람다가 특정 인자를 사용하지 않는 경우 밑줄을 사용할 수 있다.
    
    ```kotlin
    fun main() {
    	val list = listOf('a', 'b', 'c')
    	list.mapIndexed { index, _ ->
    		"[$index]"
    	} eq listOf("[0]", "[1]", "[2]")
    }
    ```
    

```kotlin
fun main() {
	val list = listOf('a', 'b', 'c')
	list.indices.map {
		"[$it]"
	} eq listOf("[0]", "[1]", "[2]")
}
```

- mapIndexed()를 항상 indices와 map()으로 대신할 수 있지 않다.
위 경우 mapIndexed()에 전달한 람다가 원소 값은 무시하고 인덱스만 사용했기 때문에 대신할 수 있었다.
- 람다에 파라미터가 없을 수도 있다.
    
    → 이 경우 파라미터가 없다는 사실을 강조하기 위해 화살표를 남겨둘 수는 있지만 권장하지는 않는다.
    
    ```kotlin
    fun main() {
    	run { -> trace("A Lambda") }
    	run { trace("Without args") }
    	trace eq """
    		A Lambda
    		Without args
    	"""
    }
    ```
    
- 일반 함수를 쓸 수 있는 모든 곳에 람다를 쓸 수 있다.
- 람다가 너무 복잡하면 이름 붙은 함수로 정의하는 편이 낫다.
- 람다를 단 한번만 사용하는 경우라도 람다가 너무 크면 이름 붙은 함수로 작성하는 게 더 낫다.

## 람다의 중요성

```kotlin
// 리스트에서 짝수 선택
fun filterEven(num: List<Int>): List<Int> {
	val result = mutableListOf<Int>()
	for (i in nums) {
		if (i % 2 == 0) {
			result += i
		}
	}
	return result
}

// 2보다 큰 수만 선택
fun greaterThan2(nums: List<Int>): List<Int> {
	val result = mutableListOf<Int>()
	for (i in nums) {
		if (i > 2) {
			result += 1
		}
	}
	return result
}

// 람다를 사용하면 두 경우 모두 같은 함수를 쓸 수 있다.
// 표준 라이브러리 함수 filter()는 보존하고 싶은 원소를 선택하는 술어를 인자로 받는다.

fun main() {
	val list = listOf(1, 2, 3, 4)
	val even = list.filter { it % 2 == 0 }
	val greaterThan2 = list.filter { it > 2 }
	even eq listOf(2, 4)
	greaterThan2 eq listOf(3, 4)
}
```

- filter() 가 컬렉션에서 원소를 선택하는 코드를 작성하면 우리가 직접 처리해야 했을 이터레이션을 처리해준다.
- 람다를 var나 val에 담을 수 있다. → 로직을 재사용할 수 있다.
    
    ```kotlin
    fun main() {
    	val list = listOf(1, 2, 3, 4)
    	val isEven = { e: Int -> e % 2 == 0 }
    	list.filter(isEven) eq listOf(2, 4)
    	list.any(isEven) eq true
    }
    ```
    
    - isEven을 정의할 때는 코틀린 타입 추론기가 파라미터의 타입을 결정할 수 있는 문맥이 존재하지 않으므로 람다 파라미터의 타입을 명시해야 한다.
- 람다의 또 다른 특징으로 자신의 영역 밖에 있는 요소를 참조할 수 있는 능력을 들 수 있다.
- 클로저 : 함수가 자신이 속한 환경의 요소를 포획하거나 닫아버리는 것
- 위 두 개념은 완전히 다른 개념
→ 클로저가 없는 람다가 있을 수 있고, 람다가 없는 클로저가 있을 수 있다.
    
    ```kotlin
    // 클로저
    fun main() {
    	val list = listOf(1, 5, 7, 10)
    	val divider = 5
    	list.filter { it % divider == 0 } eq
    		listOf(5, 10)
    } // val divider를 포획함. 람다는 포획한 요소를 읽을 뿐 아니라 변경 가능
    
    // 클로저2
    fun main() {
    	val list = listOf(1, 5, 7, 10)
    	var sum = 0
    	val divider = 5
    	list.filter { it % divider == 0 }
    		.forEach { sum += it }
    	sum eq 15
    } // 람다가 가변 변수 sum을 포획할 수 있지만, 보통은 환경의 상태를 변경하지 않는 형태로
      // 코드를 변경할 수 있다.
      
      
     // 일반 함수도 주변 환경의 요소를 포획
     var x = 100
     
     fun useX() {
    	 x++
    	}
    	
    	fun main() {
    		useX()
    		x eq 101
    	} //useX() 는 주변 환경의 x를 포획해서 변경한다.
    ```
    

## 컬렉션에 대한 연산

- 함수형 언어는 컬렉션을 다룰 수 있는 수단을 제공
    - map(), filter(), any(), forEach() 등
- List를 만들어내는 여러 가지 방법

```kotlin
// 람다는 인자로 추가할 원소의 인덱스를 받는다
val list1 - :ist(10) { it }
list1 eq "[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]"

// 한 값으로만 이뤄진 리스트
val list2 = List(10) { 0 }
list2 eq "[0, 0, 0, 0, 0, 0, 0, 0, 0, 0]"

// 글자로 이뤄진 리스트
val list3 = List(10) { 'a' + it }
list3 eq "[a, b, c, d, e, f, g ,h, i, j]"

// 정해진 순서를 반복
val list4 = List(10) { list3[it % 3] }
list4 eq "[a, b, c, a, b, c, a, b, c, a]"
```

- mutableList 초기화

```kotlin
val mutableList1 =
	MutableList(5, { 10 * (it + 1) })
mutableList1 eq "[10, 20 ,30 ,40 ,50]"
val mutableList2 =
	MutableList(5) { 10 * (it + 1) }
mutableList2 eq "[10, 20 ,30 ,40 ,50]"
```

- List() 와 MutavleList()는 생성자가 아니라 함수
    - 대문자로 시작해서 마치 생성자인 것처럼 보일 뿐
- 다양한 컬렉션 함수
    - fliter() 는 주어진 술어와 일치하는(술어가 true를 반환하는) 모든 원소가 들어 있는 새 리스트를 만든다.
    - any() 는 원소 중 어느 하나에 대해 술어가 true를 반환하면 true를 반환한다.
    - all() 은 모든 원소가 술어와 일치하는지 검사한다.
    - none() 은 술어와 일치하는 원소가 하나도 없는지 검사한다.
    - find() 와 firstOrNull()은 모두 술어와 일치하는 첫 번째 원소를 반환한다. 원소가 없을 때 find()는 예외를 던지고, findOrNull()은 null을 반환한다.
    - lastOrNull()은 술어와 일치하는 마지막 원소를 반환하며, 일치하는 원소가 없으면 null을 반환한다.
    - count()는 술어와 일치하는 원소의 개수를 반환한다.
    
    ```kotlin
    val list - listOf(-3, -1, 5, 7, 10)
    
    list.filter { it > 0 } eq listOf(5, 7, 10)
    list.count { it > 0 } eq 3
    
    list.find { it > 0 } eq 5
    list.firstIrNull { it > 0 } eq 5
    lsit.lastOrNull { it < 0 } eq -1
    
    list.any { it > 0 } eq true
    list.any { it != 0 } eq true
    
    list.all { it > 0 } eq false
    list.all { it != 0 } eq true
    
    list.none { it > 0 } eq false
    list.none { it != 0 } eq true
    ```
    
    - filter()와 count() 는 모든 원소에 술어를 적용하지만 any() 나 find()는 결과를 찾자마자 이터레이션을 중단한다.
    
    ```kotlin
    val (pos, neg) = list.partition { it > 0 }
    pos eq "[5, 7, 10]"
    neg eq "[-3, -1]"
    ```
    

```kotlin
data class Product(
	val description: String,
	val price: Double
)

fun main() {
	val products = listOf(
		Product("bread", 2.0),
		Product("wine", 5.0)
	)
	products.sumByDouble { it.price } eq 7.0
	
	products.sortedByDesending { it.price } eq
		...
		...
	products.minByOrNull { it.price } eq
		Product("bread", 2.0)
	}
```

- sorted() 와 sortedBy() 는 오름차순
sortedDesending() 과 sortedByDesending() 은 내림차순
- MinByOrNull()은 주어진 대소 비교 기준에 따라 찾은 최솟값을 돌려주며, 리스트가 비어 있는 경우 null을 반환
- sumBy() 와 sumByDouble() 은 컬렉션에 들어 있는 값을 모두 더해서 넘침이 발생하지 않는다고 확신할 때만 사용해야한다.
→ sumOf() 사용을 권장

```kotlin
// 넘칠 것 같으면
products.sumByDouble { it.price }

// 아래 처럼 사용
products.sumOf { it.price.toBigDecimal() }
```

- take() 와 drop()
    - 각각 첫 번째 원소를 취하고, 첫 번째 원소를 제거
- takeLast() 와 dropLast()
    - 각각 마지막 원소를 취하고, 마지막 원소를 제거

## 멤버 참조

- 함수, 프로퍼티, 생성자에 대해 만들 수 있는 멤버 참조는 해당 함수, 프로퍼티, 생성자를 호출하는 뻔한 람다를 대신할 수 있다.
- 멤버 함수나 프로퍼티 이름 앞에 그들이 속한 클래스 이름과 2중 콜론(::)을 위치시켜서 멤버 참조를 만들 수 있다.
ex) Message::isRead
    
    ```kotlin
    // 술어를 받아 필터하는 filterNot
    message.filterNot(Message::isRead)
    
    // 비교기(comparator)를 사용해 리스트를 정렬하는 sortedWith
    messages.sortedWith(compareBy(
    	Message::isRead, Message::sender))
    	
    	// 읽지 않은 메시지가 보낸 사람 순서로 정렬 된 후 읽은 메시지가
    	// 보낸 사람 순서로 정렬됨
    ```
    
    - 객체의 기본적인 대소 비교를 따르지 않도록 정렬 순서를 지정해야 할 때는 프로퍼티 참조가 유용하다.

- 함수 참조
    - List에 읽지 않은 메시지뿐만 아니라 중요한 메시지가 들어 있는지도 검사하고 싶다고 가정
    → 이 로직을 람다에 넣을 수도 있지만, 이런 경우 별도의 함수로 추출하면 코드를 더 이해하기 쉽다.
    → 코틀린에서는 함수 타입이 필요한 곳에 함수를 바로 넘길 수 없지만, 그 대신 함수에 대한 참조는 넘길 수 있다.
        
        ```kotlin
        data class Message(
        	val sender: String,
        	val text: String,
        	val isRead: Boolean,
        	val attachments: List<Attachment>
        )
        
        data class Attachment(
        	val type: String,
        	val name: String
        )
        
        fun Message.isImportant(): Boolean =
        	text.contains("Salary increase") ||
        		attachments.any {
        			it.type == "image" &&
        				it.name.contains("cat")
        		}
        		
        fun main() {
        	val messages = listOf(Message(
        		"Boss", "Let's discuss goals " + 
        		"for next year", false,
        		listOf(Attachment("image", "cute cats"))))
        	messages.any(Message::isImportant) eq true
        }
        ```
        
    - 새 Message 클래스에는 attachments 라는 프로퍼티와 Message.isImportant() 라는 확장 함수가 추가됨
    - 참조를 만들 수 있는 대상이 멤버 함수로만 제한되어 있지는 않다.
    - Message를 유일한 파라미터로 받는 최상위 수준 함수가 있으면 이 함수를 참조로 전달할 수 있다.
    - 최상위 수준 함수에 대한 참조를 만들 때에는 클래스 이름이 없으므로 ::function 처럼 쓴다.
    
    ```kotlin
    fun ignore(message: Message) = 
    	...
    	
    fun main() {
    	...
    	msgs.filter(::ignore).size eq 1
    	msgs.filterNot(::ignore).size eq 1
    ```
    

- 생성자 참조
    - 클래스 이름을 사용해 생성자에 대한 참조를 만들 수 있다.
    
    ```kotlin
    data class Student(
    	val id: Int,
    	val name: String
    )
    
    fun main() {
    	val names = listOf("Alice", "Bob")
    	val students =
    		names.mapIndexed { index, name ->
    			Student(index, name)
    		}
    	students eq listOf(Student(0, "Alice"),
    		Student(1, "Bob"))
    	names.mapIndexed(::Student) eq students
    }
    ```
    
    - mapIndexed() 함수는 names의 각 원소의 인덱스를 원소와 함께 람다에 전달
    - students 정의에서는 이 두 값(인덱스와 원소)을 명시적으로 생성자에 넘김
    - names.mapIndexed(::Student) 를 통해 똑같은 효과를 얻음
    - 함수와 생성자 참조를 사용하면 단순히 람다로 전달되기만 하는 긴 파라미터 리스트를 지정하는 수고를 하지 않아도 됨
    → 람다를 사용할 때보다 가독성이 좋아짐

- 확장 함수 참조
    - 참조 앞에 확장 대상 타입 이름을 붙이면 됨
  
```kotlin
fun Int.time47() = times(47)

class Frog
fun Frog.speak() = "Ribbit!"

fun goInt(n: Int, g: (Frog) -> String) =
	g(frog)
	
fun main() {
	goInt(12, Int::times47) eq 564
	goFrog(Frog(), Frog::speak) eq "Ribbit!"
```

## 고차 함수

<aside>
💡 프로그래밍 언어에서 함수를 다른 함수의 인자로 넘길 수 있거나 함수가 반환값으로 함수를 돌려줄 수 있으면, 언어가 고차 함수를 지원한다고 말한다.

</aside>

- 표준 라이브러리 any() 직접 구현
    
    ```kotlin
    fun <T> List<T>.any(
    	predicate: (T) -> Boolean
    ): Boolean {
    	for (element in this) {
    		if (predicate(element)) //선택 기준을 element 가 만족하는지
    		return true
    	}
    	return false
    }
    
    fun main() {
    	val ints = listOf(1, 2, -3)
    	ints.any { it > 0 } eq true
    	val strings = listOf("abc", " ")
    	strings.any { it.isBlank() } eq true
    	strings.any(String::isNotBlank) eq // 함수 참조를 전달하는 또 다른 방법
    		true
    }
    ```
    

- repeat() 직접 구현

```kotlin
fun repeat(
	times: Int,
	action: (Int) -> Unit // (Int) -> Unit 타입의 함수를 action 파라미터로 받는다.
) {
	for (index in 0 until times) {
		action(index)
	}
}
fun main() {
	repeat(3) { trace("#$it") }
	trace eq "#0 #1 #2"
}
```

```kotlin
fun main() { 
	val returnTypeNullable: (String) -> Int? =
		{ null }
	val mightBeNull: ((String) -> Int)? = null
	returnTypeNullable("abc") eq null
	// 널 검사를 하지 않으면 컴파일 되지 않음
	// mightBeNull("abc")
	if (mightBeNull != null_ {
		mightBeNull("abc")
	}
}
```

- 반환 타입이 널이 될 수 있는 타입으로 만드는 것과 함수 전체의 타입을 널이 될 수 있는 타입으로 만드는 것의 차이점에 주의
- mightBeNull에 저장된 함수를 호출하기 전에 함수 참조 자체가 null이 아닌지 반드시 검사해야함

## 리스트 조작하기

- 묶기
    - zip() 은 두 List의 원소를 하나씩 짝짓는 방식으로 묶는다.
    
    ```kotlin
    fun main() {
    	val left = listOf("a", "b", "c", "d")
    	val right = listOf("q", "r", "s", "t")
    	
    	left.zip(right) eq
    		"[(a, q), (b, r), (c, s), (d, t)]"
    		
    	left.zip(0..4) eq
    		"[(a, 0), (b, 1), (c, 2), (d, 3)]"
    		
    	(10..100).zip(right) eq
    		"[(10, q), (11, r), (12, s), (13, t)]"
    }
    ```
    
    - 두 시퀀스 중 어느 한쪽이 끝나면 묶기 연산도 끝난다.
    - zip() 함수는 만들어진 Pair에 대해 연산을 적용하는 기능도 있다.
    
    ```kotlin
    data class Person(
    	val name: String,
    	val id: Int
    )
    
    fun main() {
    	val names = listOf("Bob", "Jill", "Jim")
    	val ids = listOf(1731, 9274, 8378)
    	names.zip(ids) { name, id ->
    		Person(name, id)
    	} eq "[Person(name=Bob, id=1731), " +
    		...
    		...
    	}
    ```
    
    - 한 List에서 어떤 원소와 그 원소에 인접한 다음 원소를 묶으려면 zipWithNext() 사용
    
    ```kotlin
    list.zipWithNext { a, b -> "$a$b" } eq
    	"[ab, bc, cd]"
    ```
    
- 평평하게 하기
    - flatten() 은 각 원소가 List인 List를 인자로 받아서 원소가 따로따로 들어 있는 List를 반환한다.
    
    ```kotlin
    fun main() {
    	val list = listOf(
    		listOf(1, 2),
    		listOf(4, 5),
    		listOf(7, 8),
    	)
    	list.flatten() eq "[1, 2, 4, 5, 7, 8]"
    }
    ```
    
    ```kotlin
    fun main() {
    	val inRange = 1..3
    	
    	intRange.map { a -> // 세 List가 생겼다는 정보를 유지
    		intRange.map { b -> a to b }
    	} eq "[" +
    		"[(1, 1), (1, 2), (1, 3)], " +
    		"[(2, 1), (2, 2), (2, 3)], " +
    		"[(3, 1), (3, 2), (3, 3)]" +
    		"]"
    		
    	// 평평한 List를 얻는 두 가지 방법
    	
    	intRange.map { a ->  // 단일 List로 만든다.
    		intRange.map { b -> a to b }
    	}.flatten() eq "[" +
    		"(1, 1), (1, 2), (1, 3), " +
    		"(2, 1), (2, 2), (2, 3), " +
    		"(3, 1), (3, 2), (3, 3)" +
    		"]"
    		
    		
    	intRange.flatMap { a ->  // 바로 위 작업을 해야 하는 경우가 흔해서 한번 호출하면 map()과 flatten()을 모두 수행해주는 flatMap()이라는 합성 연산을 제공
    		intRange.map { b -> a to b }
    	} eq "[" +
    		"(1, 1 ), (1, 2), (1, 3), " +
    		"(2, 1), (2, 2), (2, 3), " +
    		"(3, 1), (3, 2), (3, 3)" +
    		"]"
    }
    ```
    
    ```kotlin
    class Book(
    	val title: String,
    	val authors: List<String>
    )
    
    fun main() {
    	val books = listOf(
    		Book("1984", listOf("George Orwell")),
    		Book("Ulysses", listOf("James Joyce"))
    	)
    	books.map { it.authors }.flatten() eq
    		listOf( "George Orwell" , "James Joyce" )
    	books.flatMap { it.authors } eq
    		listOf( "George Orwell" , "James Joyce" )
    }
    ```
    

## 맵 만들기

```kotlin
data class Person(
	val name: String,
	val age: Int
)

val names = listOf( "Alice" , "Arthricia" ,
	"Bob", "Bill", "Birdperson", "Charlie",
	"Crocubot", "Franz", "Revolio" )
val ages = list0f (21, 15, 25, 25, 42, 21,
	42, 21, 33)
fun people(): List<Person> =
	names.zip(ages) { name, age ->
		Person(name, age)
}

fun main() {
	val map: Map<Int, List<Person>> =
		people().groupBy(Person::age)
	map[15] eq listOf(Person("Arthricia", 15))
	map[21] eq listOf(
		Person("Alice", 21),
		Person("Charlie", 21),
		Person("Franz" , 21))
	map[22] eq null
	map[25] eq listOf(
		Person("Bob", 25),
		Person("Bill", 25))
	map[33] eq listOf(Person("Revolio", 33))
	map[42] eq listOf(
		Person("Birdperson", 42),
		Person("Crocubot", 42))
}
```

- Map을 사용하면 키를 사용해 값에 빠르게 접근할 수 있다.
- 라이브러리 함수 groupBy()는 이런 Map을 만드는 방법 중 하나
    - groupBy()의 파라미터는 원본 컬렉션의 원소를 분류하는 기준이 되는 키를 반환하는 람다
    - 원본 컬렉션의 각 원소에 이 람다를 적용해 키 값을 얻은 후 맵에 넣어준다.
- filter() 를 사용하면 새로운 키가 나타날 때마다 그룹을 만드는 작업을 반복해야 한다.
- 그룹이 두 개만 필요할 경우에는 술어에 의해 컬렉션 내용을 두 그룹으로 나누는 partition() 함수가 더 직접적
- groupBy()는 결과 그룹이 세 개 이상인 경우 적합

```kotlin
fun main() {
	val map: Map<Person, String> =
		people().associateWith { it.name }
	map eq mapOf(
		Person("Alice", 21) to "Alice",
		Person("Arthricia", 15) to "Arthricia",
		Person("Bob", 25) to "Bob",
		Person("Bill", 25) to "Bi ll",
		Person("Birdperson", 42) to "Birdperson",
		Person("Charlie" , 21 ) to "Charlie" ,
		Person("Crocubot" , 42) to "Crocubot" ,
		Person("Franz", 21) to "Franz",
		Person("Revolio", 33) to "Revolio")
}
```

- 리스트에 대해 associateWith() 사용하면, 리스트 원소를 키로 하고 associateWith()에 전달된 함수를 리스트 원소에 적용한 반환값을 값으로 하는 Map을 만들 수 있음

- associateBy() 는 associateWith()가 만들어내는 연관 관계를 반대 방향으로 뒤집는다.
    
    ```kotlin
    "Alice" to Person("Alice", 21 ),
    ...
    ```
    
    - associateBy()의 셀렉터가 유일한 키 값을 만들어내야 한다.
    - 키 값이 유일하면 associateBy()가 반환하는 맵이 키와 값을 하나씩 연결시켜줄 수 있다.
    (이전 예제 기준, name은 겹치지 않아서 다 나왔지만 age를 사용하면 맵의 키가 줄면서 Person 객체 일부가 제외됨)
        - age처럼 셀렉터가 돌려주는 키와 연관된 값이 여럿 존재하는 경우, 같은 키를 내놓는 값 중 컬렉션에서 맨 나중에 나타나는 원소가 생성되는 Map에 포함
    
- getOrElse() 는 Map에서 값을 찾는다.
- getOrElse() 는 키가 없을 때 디폴트 값을 계산하는 방법이 담긴 람다를 인자로 받는다.
- getOrPut() 은 MutableMap에만 적용할 수 있다.
- getOrPut() 은 키가 있으면 그냥 연관된 값을 반환한다.
    - 맵에서 키를 찾을 수 없으면, 값을 계산한 후 그 값을 키와 연관시켜 맵에 저장하고 저장한 값을 반환

```kotlin
fun main() {
// getOrElse() 예제
    val map = mapOf("a" to 1, "b" to 2)

    val valueA = map.getOrElse("a") { -1 }
    println("Value for key 'a': $valueA")// 출력: Value for key 'a': 1

    val valueC = map.getOrElse("c") { -1 }
    println("Value for key 'c': $valueC")// 출력: Value for key 'c': -1

// getOrPut() 예제
    val mutableMap = mutableMapOf("x" to 10, "y" to 20)

    val valueX = mutableMap.getOrPut("x") { 30 }
    println("Value for key 'x': $valueX")// 출력: Value for key 'x': 10
    println("Map after getting 'x': $mutableMap")// 출력: Map after getting 'x': {x=10, y=20}

    val valueZ = mutableMap.getOrPut("z") { 30 }
    println("Value for key 'z': $valueZ")// 출력: Value for key 'z': 30
    println("Map after putting 'z': $mutableMap")// 출력: Map after putting 'z': {x=10, y=20, z=30}
}
```

이 예제 코드는 다음과 같은 내용을 보여줍니다:

1. `getOrElse()` 사용:
    - 존재하는 키 "a"에 대해 값을 가져옵니다.
    - 존재하지 않는 키 "c"에 대해 기본값 -1을 반환합니다.
2. `getOrPut()` 사용:
    - 존재하는 키 "x"에 대해 기존 값을 반환하고 맵을 변경하지 않습니다.
    - 존재하지 않는 키 "z"에 대해 새 값 30을 계산하고, 이를 맵에 추가한 후 반환합니다.

이 코드를 실행하면 각 함수의 동작을 명확히 볼 수 있습니다. `getOrElse()`는 읽기 전용 동작을 수행하며, `getOrPut()`은 필요한 경우 맵을 수정합니다.

- 여러 Map 연산이 List가 제공하는 연산과 겹친다. → filter() 나 map() 을 적용할 수 있다.
- map() 을 Map에 적용하는 행위 : 맵을 변환한다
    
    ```kotlin
    fun main() {
    	val even = map0f(2 to "two", 4 to "four")
    	even.map {
    		"${it.key}=${it.value}"
    	} eq list0f( "2=two" ,"4=four" )
    	
    	even.map { (key, value) ->
    		"$key=$value"
    	} eq list0f( "2=two", "4=four" )
    	
    	even.mapKeys { (num, _) -> _num }  // 밑줄을 이용해 컴파일러 경고를 막음. 새 맵 반환
    		.mapValues { (_, str) _> "minus $str" } eq
    	mapOf(-2 to "minus two",
    		-4 to "minus four")
    		
    	even.map { (key, value) ->
    		-key to "minus $value"
    	}.toMap() eq mapOf(-2 to "minus two",  // 여기서 새로운 Map을 생성하려면 명시적으로 toMap() 호출
    		-4 to "minus four")
    }
    ```
    
- any() 나 all() 같은 함수도 Map 적용 가능

## 시퀀스

<aside>
💡 코틀린 Sequence는 List와 비슷하지만, Sequence를 대상으로 해서는 이터레이션만 수행 가능 → 인덱스를 써서 Sequence의 원소에 접근할 수 없다.
이 제약으로 인해 시퀀스에서 연산을 아주 효울적으로 연쇄시킬 수 있다.

</aside>


- 코틀린 Sequence를 다른 함수형 언어에서는 스트림이라고 부른다.
- List에 대한 연산은 즉시 계산됨 → List 연산을 연쇄시키면 첫 번째 연산의 결과가 나온 후에야 다음 연산을 적용할 수 있음
- 즉시 계산은 직관적이고 단순하지만 최적은 아니다.
  
![4장1](https://github.com/user-attachments/assets/f571b0f8-8a7f-4fc4-a780-b19dc10c25d4)

![4장2](https://github.com/user-attachments/assets/94f18659-39a0-4438-ae0e-b5ae59a59f87)


- 지연 계산은 필요할 때만 계산을 수행한다.
- filter나 map()을 Sequence에 대해 호출하면 다른 Sequence가 생기며, 계산 결과를 요청할 때까지는 아무 일도 벌어지지 않는다. 대신 새 Sequence는 지연된 모든 연산에 대한 정보를 저장해두고, 필요할 때만 저장해둔 연산을 실행한다.

- Sequence 연산에는 중간 연산과 최종 연산이 있다.
    - 중간 연산 : 결과로 다른 Sequence를 내놓는다. filter() 와 map()은 중간 연산
    - 최종 연산 : Sequence가 아닌 값을 내놓는다. 결과값을 얻기 위해 최종 연산은 저장된 모든 계산을 수행

```kotlin
fun main() {
	val list = list0f(1, 2, 3, 4)
	trace(list.asSequence()
		.filter(lnt::isEven)
		.map(lnt::square)
		.toList())
	trace eq """
		1.isEven()
		2.isEven()
		2.square()
		3.isEven()
		4.isEven()
		4.square()
		[4, 16]
	"""
}
```

- Sequence는 (중간) 연산을 저장해두기 때문에 각 연산을 원하는 순서로 호출할 수 있고, 그에 따라 지연 계산이 발생한다.

- generateSeqeunce()의 첫 번째 인자는 시퀀스의 첫 번째 원소이고, 두 번째 인자는 이전 원소로부터 다음 원소를 만들어내는 방법을 정의하는 람다

```kotlin
fun main() {
	val naturalNumbers =
		generateSequence(1) { it + 1 }
	natura1Numbers.take(3).toList() eq
		listOf(1, 2, 3)
	naturalNumbers.take(10).sum() eq 55
}
```

- 컬렉션인 경우 size 프로퍼티를 통해 미리 크기를 알 수 있었다.
- Sequence는 마치 무한인 것처럼 취급된다.
    - 위 예제에서는 take()를 통해 원하는 개수만큼 원소를 얻은 다음, 최종 연산을 수행

- 첫 번째 인자를 요구하지 않고 Sequence의 다음 원소를 반환하는 람다만 받는 generateSequence() 오버로딩 버전
- 람다는 더 이상 원소가 없으면 null을 내놓는다.
- 밑 예제는 입력에 XXX 가 나타날 때까지 Sequence 원소를 생성

```kotlin
fun main() {
	val items = mutableListOf(
		"first", "second", "third", "XXX", "4th"
	)
	val seq = generateSequence {
		items.removeAt (0). takeIf { it != "XXX" }
	}
	seq.toList() eq "[first, second, third]"
	capture {
		seq.toList()
	} eq "IllegalStateException: This " +
		"sequence can be consumed only once."
}
```

- takeIf()는 수신 객체(위에서는 removeAt(0)이 반환한 String)가 주어진 술어를 만족하면 수신 객체를 반환하고, 술어를 만족하지 않으면(즉, XXX) null을 반환한다.

- Sequence는 한 번만 이터레이션 할 수 있다.
    - (시퀀스에 따라서는 반복 이터레이션이 가능한 경우도 있다. 그러나 처음부터 모든 연산을 다시 실행할 가능성이 있으므로 반복 계산을 피하기 위해서 컬렉션으로 변환해 저장)
- 여러 번 처리하고 싶다면 우선 시퀀스를 Collection 타입 중 하나로 변환해야 한다.

```kotlin
fun <T> T.takeIf(
	predicate : (T) -> Boolean
): T? {
	return if (predicate(this)) this else null
}

fun main() {
	"abc".takeIf { it != "XXX" } eq "abc"
	"XXX".takeIf { it != "XXX" } eq null
}

// 감소 수열 예제
fun main() {
	generateSequence(6) {
		( it - 1 ).takeIf { it > 0 }
	} .toList() eq listOf(6, 5, 4, 3, 2, 1 )
}
```

## 지역 함수

- 다른 함수 안에 정의된 이름 붙은 함수를 지역 함수라고 한다.
- 지역 함수는 클로저다.
    
    → 즉, 지역 함수는 자신을 둘러싼 환경의 var나 val을 포획한다.
    포획을 쓰지 않으면 주변 환경의 값을 별도로 파라미터로 전달받아야 한다.
    

```kotlin
fun main() {
	val logMsg = StringBuilder()
	fun log(message: String) = 
		logMsg.appendLine(message)
	log("Starting computation)
	val x = 42
	log("Computation result: $x")
```

- 지역 확장 함수로 정의할 수도 있다.

```kotlin
fun main() {
	fun String.exClaim() = "$this!"
	"Hello".exClaim() eq "Hello!"
```

- main() 안에서만 exClaim을 사용할 수 있다.

```kotlin
package localfunctions

class Session(
	val title: String,
	val speaker: String
)

val sessions = listOf(Session(
	"Kotlin Coroutines", "Roman Elizarov"))
	
val favoriteSpeakers = setOf("Roman Elizarov")
```

```kotlin
import localfunctions.*;
import atomictest.eq

fun main() {
	fun interesting(session: Session): Boolean {
		if (session.title.contains("Kotlin") &&
			session.speaker in favoriteSpeakers) {
			return true
		}
		// ... 추가 검사
		return false
	}
	sessions.any(::interesting) eq true
}
```

- interesing() 이 단 한번만 사용되지만 안에 쓰인 return 식 때문에 람다로 정의할 수 없음

```kotlin
// 익명 함수 사용
fun main() {
	sessions.any(
		fun(session: Session): Boolean {
			if (session.title.contains("Kotlin") &&
			session.speaker in favoriteSpeakers) {
			return true
		}
		// ... 추가 검사
		return false
	}) eq true
}
```

- 람다가 너무 복잡해서 읽기 어렵다면 지역 함수나 익명 함수로 대신하자

- 레이블

```kotlin
fun main() {
	val list = listOf(1, 2, 3, 4, 5)
	val value = 3
	var result = ""
	list.forEach {
		result += "$it"
		if (it == value) {
			result eq "123"
			return
		}
	}
```

- 위 코드에서 return 은 main() 함수를 끝낸다는 뜻
- 람다를 둘러싼 함수가 아니라 람다에서만 반환해야 한다면 레이블이 붙은 return 을 사용해여 한다
    
    ```kotlin
    return@forEach
    
    // 람다 앞에 레이블@을 넣으면 새 레이블을 만들 수도 있다.
    list.forEach tag@{
    	...
    	return@tag
    }
    ```
    

```kotlin
// 위에서 익명 함수를 람다로 바꿔보자
fun main() {
	sessions.any { session ->
		if (session.title.contains("Kotlin") &&
			session.speaker in favoriteSpeakers) {
			return@any true
		}
		// ... 추가 검사
		false
	} eq true
}
```

- 예제 살펴보기

```kotlin
// first() 익명함수
// second() 람다
// third() 지역 함수에 대한 참조 반환
// forth() 세번째와 같은 효과를 내지만 식 본문 써서 더 간결하게 처리
// fifth() 람다를 식 본문 함수에 사용해 같은 효과

fun fisrt(): (Int) -> Int {
	val func = fun(i: Int) = i + 1
	func(1) eq 2
	return func
}

fun second(): (String) -> String {
	val func2 = { s: String -> "$s!" }
	func2("abc") eq "abc!"
	return func2
}

fun third(): () -> String {
	fun greet() = "Hi!"
	return ::greet
}

fun forth() = fun() = "Hi!"

fun fifth() = { "Hi!" }

fun main() {
	val funRef1: (Int) -> Int = first()
	val funRef2: (String) -> String = second()
	val funRef3: () -> String = third()
	val funRef4: () -> String = fourth()
	val funRef5: () -> String = fifth()
	
	funRef1(42) eq 43
	funRef2("xyz") eq "xyz!"
	funRef3() eq "Hi!"
	funRef4() eq "Hi!"
	funRef5() eq "Hi!"
	fisrt()(42) eq 43
	second()("xyz") eq "xyz!"
	third()() eq "Hi!"
	fourth()() eq "Hi!"
	fifth()() eq "Hi!"
}
```

## 리스트 접기

- fold()는 리스트의 모든 원소를 순서대로 조합해 결과값을 하나 만들어낸다.

```kotlin
fun main() {
	val list - listOf(1, 10, 100, 1000)
	list.fold(0) { sum, n ->
		sum + n
	} eq 1111
}
```

- fold() 는 자신이 sum 을 구하는 중이라는 걸 알지 못한다.
    
    → sum 은 예제를 더 쉽게 이해할 수 있도록 선택한 이름일뿐

```kotlin
fun main() {
	val listOf(11, 13, 17, 19)
	list.fold(7) { sum, n ->
		sum + n
	} eq 67
	list.runningFold(7) { sum, n ->
		sum + n
	} eq "[7, 18, 31, 48, 67]"
	list.reduce { sum, n ->
		sum + n
	} eq 60
	list.runningReduce { sum, n ->
		sum + n
	} eq "[11, 24, 41, 60]"
}
```

## 재귀

<aside>
💡 재귀(recursion)는 함수 안에서 함수 자신을 호출하는 프로그래밍 기법
꼬리 재귀는 일부 재귀 함수에 명시적으로 적용할 수 있는 최적화 방법

</aside>

- 재귀 함수를 호출하면 매 재귀 호출이 호출 스택에 프레임을 추가
    
    → 이로 인해 StackOverflowError 가 발생하기 쉬움
    
    - StackOverflowError : 호출 스택을 너무 많이 써서 더 이상 스택에 쓸 수 있는 메모리가 없다는 뜻
- 프로그래머가 재귀 호출 연쇄를 제때 끝내지 않아서 StackOverflowError를 야기하는 경우를 무한 재귀라 한다.

- 무한 재귀 없이 그냥 재귀 호출을 아주 많이 해도 똑같은 에러를 낼 수 있다.
ex) sum(n) = n + sum(n-1)
- StackOverflowError 를 막기 위해서는 재귀 대신 이터레이션을 택해야 한다.

```kotlin
//재귀

fun sum(n: Long): Long {
	if (n == 0L) return 0
	return n + sum(n - 1)
}

fun main() {
	sum(2) eq 3
	sum(1000) eq 500500
	(1..100_000L).sum() eq 50005000
}

// 이터레이션

fun sum(n: Long): Long {
	var accumulator - 0L
	for (i in 1..n) {
		accumulator += i
	}
	return accumulator
}

fun main() {
	sum(10000) eq 50005000
	sum(100000) eq 5000050000
}
```

- 이터레이션 방식을 보면 sum() 호출이 단 한번만 이뤄지고 결과는 for 루프를 통해 계산된다.
- 이터레이션을 사용한 해법이 단순하긴 하지만 가변 상태 변수 ccumulator를 사용해 변하는 값을 저장해야 하는데, 함수형 프로그래밍에서는 가변 상태를 가능하면 피하려고 한다.
- 호출 스택 넘침을 막기 위해 함수형 언어들은 꼬리 재귀라는 기법을 사용한다.
→ 꼬리 재귀 목표 : 호출 스택의 크기 줄이기

- 꼬리 재귀
    - tailrec 키워드 사용
    - 컴파일러가 수행하는 최적화
    - 모든 재귀 함수에 적용할 수 있는 것은 아님
    - tailrec을 성공적으로 사용하려면 재귀가 마지막 연산이여야 함
        
        → 재귀 함수가 자기 자신을 호출해 얻은 결과값을 아무 연산도 적용하지 않고 즉시 반환해야 한다는 뜻
        
    
    ```kotlin
    private tailrec fun sum(
    	n: Long,
    	accumulator: Long
    ): Long =
    	if (n == 0L) accumulator
    	else sum(n - 1, accumulator + n)
    	
    fun sum(n: Long) = sum(n, 0)
    
    fun main() {
    	sum(2) eq 3
    	sum(10000) eq 500050000
    	sum(100000) eq 5000050000
    }
    ```
    
    - accumulator 파라미터를 추가하면 재귀 호출 중에 뎃셈을 할 수 있고, 결과를 받으면 그냥 반환하는 일 말고는 할 일이 없다.
    - accumulator 이 불변값이 된다.

- 피보나치 수 계산도 꼬리 재귀로 효율적이게 계산할 수 있다.
