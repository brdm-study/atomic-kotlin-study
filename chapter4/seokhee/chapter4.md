## 람다

---

- ** 람다(Lambda)**: 부가적인 장식이 덜 들어간 함수
  
- 함수 생성에 필요한 최소한의 코드만 필요, 다른 코드에 직접 삽입 가능
```kotlin
val list = listOf(1, 2, 3, 4)
val result = list.map({ n: Int -> "[$n]" })
```

- 코틀린은 람다의 타입을 추론 가능
```kotlin
val result = list.map({ n -> "[$n]" })
```

- 람다의 파라미터가 하나라면 it로 자동 지정
```kotlin
val result = list.map({ "[$it]" })
```

- 모든 타입의 List에 map() 사용 가능
```kotlin
val list = listOf('a', 'b', 'c', 'd')
val result = list.map({ "[${it.uppercase()}]" })
```

- 람다가 유일한 파라미터인 경우 괄호 제거 가능
```kotlin
val result = list.map { "[${it.uppercase()}]" }
```

- 람다가 마지막 파라미터인 경우 람다를 괄호 밖에 위치 가능
```kotlin
list.joinToString(" ") { "[$it]" }
```

- 파라미터가 두 개 이상인 경우
```kotlin
list.mapIndexed { index, element -> "[$index: $element]" }
```

- 사용하지 않는 인자는 밑줄로 표시
```kotlin
list.mapIndexed {  index, _ -> "[$index]} }
```

- 파라미터가 없는 람다
```kotlin
run { trace("Without args") }
```

- 람다는 var과 val에 담아 재사용 가능
```kotlin
val isEven: (Int) -> Boolean = { it % 2 == 0 }
val result = list.filter(isEven)
```

- 람다는 자신의 영역 밖 요소를 참조 가능 (클로저)
```kotlin
val divider = 5
val result = list.filter { id % divider == 0 }
```

## 람다의 중요성
---

- 람다는 프로그래밍에 중요한 능력을 부여
- 예시: 짝수만 걸러내는 함수
```kotlin
fun filterEven(nums: List<Int>): List<Int> {
    val result = mutableListOf<Int>()
    for (i in nums) {
        if (i % 2 == 0) result += i
    }
    return result
}
```

- 람다와 표준 라이브러리 함수 사용
```kotlin
val result = list.filter { it % 2 == 0 }
```

- 람다는 함수의 인자로 전달 가능
```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
val isEven: (Int) -> Boolean = { it % 2 == 0 }
val result = list.filter(isEven)
```

## 컬렉션에 대한 연산
---

- List 생성 방법
```kotlin
val list1 = List(10) { it }
val list2 = List(10) { 0 }
val list3 = List(10) { 'a' + it }
val list4 = List(10) { list3[it % 3] }
val mutableList1 = MutableList(5) { 10 * (it + 1) }
```

- 다양한 컬렉션 함수
```kotlin
val list = listOf(1, 2, 3, 4)
val filtered = list.filter { it % 2 == 0 }
val anyEven = list.any { it == 3 }
val allEven = list.all { it == 3 }
val noneEven = list.none { it == 5 }
val firstEven = list.find { it % 2 == 0 }
val firstOrNullEven = list.firstOrNull { it == 5 }
val lastOrNullEven = list.lastOrNull { it % 2 == 0 }
val countEven = list.count { it % 2 == 0 }
```

- filter(), map(), find() 등의 연산은 중간 연산이면서 지연 계산을 수행
```kotlin
val filteredList = listOf(1, 2, 3, 4).asSequence()
  .filter { it % 2 == 0 }
  .map { it * it }
  .toList()
```

## 시퀀스(Sequence)
---

- 시퀀스는 컬렉션에 비해 지연 계산을 사용하여 최적화 가능
- generateSequence()로 무한 시퀀스 생성
```kotlin
val naturalNumbers = generateSequence(1) { it + 1 }
val firstThree = naturalNumbers.take(3).toList()
```

## 멤버 참조
---

- 멤버 참조는 함수, 프로퍼티, 생성자에 대한 참조를 가능하게 함
```kotlin
val animalList = listOf(Animal(true), Animal(false))
animalList.filterNot(Animal::isTall)
```

- 객체의 정렬 순서 지정 시 유용
```kotlin
animalList.sortedWith(compareBy(Animal::age, Animal::name))
```

- 최상위 수준 함수에 대한 참조
```kotlin
animalList.filter(::isCat)
```

- 생성자에 대한 참조
```kotlin
val students = names.mapIndexed(::Student)
```

- 확장 함수에 대한 참조
```kotlin
goInt(12, Int::times47)
```

## 고차함수
---

- 고차 함수는 함수의 인자로 다른 함수를 받거나, 함수를 반환할 수 있음
```kotlin
val isPlus: (Int) -> Boolean = { it > 0 }
val result = listOf(1, 2, -3).any(isPlus)
```

- 함수 참조를 통해 호출 가능
```kotlin
val helloWorld: () -> String = { "Hello world!" }
helloWorld()
```

- 함수가 함수 파라미터를 받는 경우
```kotlin
val ints = listOf(1, 2, -3)
ints.any { it > 0 }
```

- nullable한 함수 타입
```kotlin
val transform: (String) -> Int? = { s: String -> s.toIntOrNull() }
val mightBeNull: ((String) -> Int)? = null
```

## List 조작하기
---

- zip()과 zipWithNext()
```kotlin
val left = listOf("a", "b", "c", "d")
val right = listOf("q", "r", "s", "t")
val zipped = left.zip(right)
val zippedWithNext = list.zipWithNext()
```

- flatten()과 flatMap()
```kotlin
val nestedList = listOf(listOf(1, 2), listOf(3, 4))
val flattened = nestedList.flatten()
val books = listOf(Book("1984", listOf("George Orwell")), Book("Ulysses", listOf("James Joyce")))
val authors = books.flatMap { it.authors }
```

## Map 만들기
---

- Map 생성 및 그룹핑
```kotlin
val peopleByAge: Map<Int, List<Person>> = people().groupBy(Person::age)
val map = mapOf(1 to "one", 2 to "two")
val mutableMap = map.toMutableMap()
mutableMap.getOrPut(0) { "zero" }
```

- filterKeys()와 filterValues()
```kotlin
val map = mapOf(1 to "one", 2 to "two", 3 to "three")
val filteredKeys = map.filterKeys { it % 2 == 1 }
val filteredValues = map.filterValues { it.contains('o') }
```

## 지역함수
---

- 함수 안에서 함수 정의 가능
```kotlin
fun main() {
  val logMsg = StringBuilder()
  fun log(message: String) = logMsg.appendLine(message)
}
```

- 클로저로서의 지역 함수
```kotlin
fun main() {
  fun String.exclaim() = "$this!"
}
```

- 지역 함수 참조
```kotlin
fun main() {
  fun isEven(number: Int) = number % 2 == 0
  val numbers = listOf(1, 2, 3, 4)
  numbers.any(::isEven)
}
```

- 익명 함수
```kotlin
fun main() {
   listOf(1, 2, 3, 4).any(fun(number: Int): Boolean {
     return number % 2 == 0
   })
}
```

- 함수가 함수를 반환하는 경우
```kotlin
fun first(): (Int) -> Int {
  return { it + 1 }
}

fun main() {
  first()(1)
}
```

## List 접기
---

- fold()와 reduce()
```kotlin
val list = listOf(1,10, 100, 1000)
val sum = list.fold(0) { sum, n -> sum + n }
val chars = “A B C D E”.split(” “)
val reduced = chars.reduce { acc, e -> “$acc $e” }
```

- runningFold()와 runningReduce()
```kotlin
val list = listOf(11, 13, 17, 19)
val runningFolded = list.runningFold(7) { sum, n -> sum + n }
val runningReduced = list.runningReduce { sum, n -> sum + n }
```

## 재귀(Recursion)
---

- 재귀 함수 예시
```kotlin
fun factorial(n: Long): Long {
  if (n <= 1) return 1
  return n * factorial(n - 1)
}
```

- 꼬리 재귀
```kotlin
tailrec fun factorial(n: Long, acc: Long = 1): Long {
  return if (n <= 1) acc else factorial(n - 1, acc * n)
}
```

- 피보나치 수열 예제
```kotlin
fun fibonacci(n: Int): Long {
  tailrec fun fibonacci(n: Int, a: Long, b: Long): Long {
    return if (n == 0) a else fibonacci(n - 1, b, a + b)
  }
  return fibonacci(n, 0, 1)
}
```
