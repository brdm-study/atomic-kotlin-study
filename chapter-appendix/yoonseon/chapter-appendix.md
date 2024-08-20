# ✅ B. 자바 상호 운용성

---

- 코틀린은 자바와 100% 호환 가능하다.
    - 새 코틀린 코드를 기존 자바 코드 위에 작성할 수 있음.
    - 코틀린으로 변환할 필요가 없는 자바 코드를 코틀린 코드로 변환할 필요는 없음.
    - 즉, 자바 프로젝트에 코틀린 코드를 끼워넣을 수 있음
- 자바 코드를 코틀린 쪽에서 호출하는 것도 쉽고, 코틀린 코드를 자바 쪽에서 호출하는 것도 가능하다.

## 코틀린에서 자바 호출하기

- 자바에서 사용하는 것처럼 클래스를 import하고 인스턴스를 만들어 함수를 호출하면 된다.

    ```kotlin
    import java.util.Random // 자바 클래스
    
    fun main() {
        val rand = Random(47)
        println(rand.nextInt(100)) // 58
    }
    ```

    - 자바 라이브러리에서 가져온 클래스여도 코틀린 코드에서 인스턴스를 만들 때 `new` 연산자를 사용하지 않음

- 자바 클래스에 정의된 Getter, Setter는 코틀린에서 프로퍼티가 된다.

    ```java
    // 자바 코드
    public class Chameleon {
        private int size;
        private String color;
    
        public int getSize() {
            return size;
        }
        public String getColor() {
            return color;
        }
        public void setSize(int size) {
            this.size = size;
        }
        public void setColor(String color) {
            this.color = color;
        }
    }
    ```

    ```kotlin
    // 코틀린 코드
    fun main() {
        val chameleon = Chameleon();
        chameleon.size = 1
        chameleon.color = "red"
        println(chameleon.size)
        println(chameleon.color)
    }
    ```

    - import한 자바 클래스는 프로퍼티가 있는 코틀린 클래스처럼 작동함

- **자바 라이브러리에 속한 클래스의 필요한 멤버함수가 없을 때 확장함수를 쓰면 유용하다!**

    ```kotlin
    fun Chameleon.adjustToTemperature(isHot: Boolean) {
        color = if (isHot) "grey" else "black"
    }
    
    fun main() {
        val chameleon = Chameleon()
        chameleon.adjustToTemperature(true)
        println(chameleon.color) // gray
    }
    ```


## 자바에서 코틀린 호출하기

- 자바코드에서도 코틀린코드가 자바 라이브러리인 것 처럼 보임

    ```kotlin
    class Basic {
        var value = 1
        fun increment() = value++
    }
    ```

    ```kotlin
    import practice.atomic.appendix.Basic;
    
    public class UsingKotlinClass {
        public static void main(String[] args) {
            Basic b = new Basic();
            b.setValue(10);
            System.out.println(b.getValue()); // 10
            b.increment();
            System.out.println(b.getValue()); // 2
        }
    }
    ```

    - `value` 는 자바빈 스타일의 Getter, Setter가 있는 private 필드가 된다.
    - `increment()` 함수는 자바의 메서드가 된다.

### data 클래스

- data 클래스는 `equals()`, `hashcode()`, `toString()` 같은 추가 멤버 함수를 생성하므로 자바에서도 사용 가능함

    ```kotlin
    public class UseDataClass {
        public static void main(String[] args) {
            // property
            Staff staff = new Staff("홍석희", "GOLD");
            System.out.println(staff.getName()+" "+staff.getRole());
            
            // toString
            Staff copyStaff = staff.copy("박주광", "Silver");
            System.out.println(copyStaff); // Staff(name=박주광, role=Silver)
            
            // Staff 객체를 해시 키로 사용
            HashMap<Staff, String> map = new HashMap<>();
            map.put(staff, "Cheerful");
            System.out.println(map.get(staff)); // Cheerful
        }
    }
    ```


> 명령줄을 사용해 코틀린 코드와 함께 동작하는 자바 코드를 실행한다면 `kotlin-runrime.jar` 를 의존 관계로 추가해야 하는데 인텔리J IDEA는 `korlin-runtime.jar` 를 (IDE 안에서 코틀린과 함께 동작 하는 자바 코드를 실행할 때) 자동으로 포함시켜준다.(포함하지 않으면 라이브러리의 유틸리티 클래스들을 찾을 수 없다는 실행 시점 오류가 발생함)
>

### 코틀린 최상위 함수를 자바에서 사용

- 코틀린 최상위 함수는 코틀린 파일로부터 이름을 딴 자바 클래스의 static 메서드로 컴파일됨

    ```kotlin
    // TopLevelFunction.kt
    
    package practice.atomic.appendix
    
    fun hi() = "hello!"
    ```

    ```java
    // CallTopLevelFunction.java
    
    public class CallTopLevelFunction {
        public static void main(String[] args) {
            String hi = TopLevelFunctionKt.hi(); // static
            System.out.println(hi); // hello!
        }
    }
    ```

    - `<최상위 함수의 파일명>.<최상위 함수명>` 으로 static 처럼 사용함
    - 최상위 함수는 static으로 취급되므로 static import도 가능함

### @JvmName

- 코틀린 최상위 함수를 자바에서 사용할 때 `@JvmName` 를 사용해서 클래스 이름을 변경할 수 있음

    ```java
    @file:JvmName("Utils") // 클래스명 지정
    package practice.atomic.appendix
    
    fun hi() = "hello!"
    ```

    ```kotlin
    public class CallTopLevelFunction {
        public static void main(String[] args) {
            String hi = Utils.hi(); // 변경한 클래스명으로 접근
            System.out.println(hi); 
        }
    }
    ```


## 자바의 Checked Exceptipn과 코틀린

- 자바에서는 Checked Exception이 발생할여지가 있다면 컴파일 시점에 예외를 처리해야한다.

    ```java
    public class JavaChecked {
        public static void main(String[] args) {
            BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
            try {
                String str = br.readLine(); // IOException 처리 필요
            } catch (IOException e) {
                // 예외 복구 로직 작성 필요
                throw new RuntimeException(e);
            }
        }
    }
    ```


- 코틀린은 예외 처리를 강제하지 않는다.

    ```kotlin
    fun main() {
        val br = BufferedReader(System.`in`.bufferedReader())
        val str = br.readLine()
        println(str)
    }
    ```

    - 자바에서는 복구가 불가능한 상황에서 Checked Exception을 사용하지만, 이 경우 최상위에서 예외를 잡아내고, 가능하면 전체 과정을 다시 시작하는게 최선임.
    - 모든 중간 단계가 예외를 그냥 넘기게 두면 코드를 이해할 때 더해지는 인지적인 부담이 줄어듦

### @Throws

- 자바에서 호출해야 하는 코틀린 코드를 작성하고 Checked Exception을 지정해야 한다면, 코틀린의 `@Throws` 어노테이션을 사용해서 자바 쪽 호출자에 검사 예외 정보를 전달할 수 있음

    ```kotlin
    // AnnotateThrows.kt
    
    import java.io.IOException
    
    @Throws(IOException::class)
    fun hasCheckedException() {
        throw IOException()
    }
    ```

    ```java
    // CatchChecked.java
    
    public class CatchChecked {
        public static void main(String[] args) {
            try {
                AnnotateThrowsKt.hasCheckedException();
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        }
    }
    ```

    - 위와 같이 코틀린이 예외 처리를 위한 언어 지원을 제공하지만, 일반적으로 예외처리는 실제 문제를 해결할 수 있는 경우(대부분 I/O 연산에 해당)에만 사용하는 것을 권장함

## 널이 될 수 있는 타입과 자바

- 코틀린은 nullable 타입과 non-nullable 타입을 구분하지만, 자바는 모든 참조 타입이 nullable 타입임

    ```java
    // JTool.java
    
    public class JTool {
        public static JTool get(String s) {
            if (s == null) return null;
            return new JTool();
        }
        public String method() {
            return "success";
        }
    }
    ```

    ```kotlin
    // PlatformTypes.kt
    
    object KotlinCode {
        val a: JTool? = JTool.get("") // 널이 될 수 있는 타입으로 지정
        val b: JTool = JTool.get("") // 널이 될 수 없는 타입으로 지정
        val c = JTool.get("") // 타입 추론으로 지정
    }
    
    fun main() {
        with(KotlinCode) {
            println(a?.method()) // a가 nullable 타입으로 ?.를 사용해야함
            println(b.method()) // success
            println(b.method()) // non-nullable 타입 처럼 동작하기에 검사가 필요하지 않음
    
            println(::a.returnType) // interoperability.JTool?
            println(::b.returnType) // interoperability.JTool
            println(::c.returnType) // interoperability.JTool!
        }
    }
    ```

    - `c` 의 반환 타입이 `JTool!`

### 플랫폼 타입

- `타입!` 은 코틀린의 **플랫폼 타입**이며, 코틀린에서 이를 표현할 수 있는 방법은 없다.
- 플랫폼 타입은 코틀린이 코틀린 타입을 벗어나는 타입을 추론했을 때 사용한다.

```kotlin
fun main() {
    val xn: JTool? = JTool.get(null) // 1
    println(xn) // null
    val yn = JTool.get(null) // 2
    println(yn) // 3
    yn.method() // 4
    val zn: JTool = JTool.get(null) // 5
}
```

- 각 단계에서 발생하는 동작과 결과는 다음과 같다.
    - **1. Nullable 타입 (`JTool?`)의 사용**

        ```kotlin
        val xn: JTool? = JTool.get(null)
        println(xn) // null
        ```

        - `xn`은 `JTool?`로 선언된 널이 될 수 있는 타입임
        - `JTool.get(null)`의 결과로 `null`을 할당받아도 xn에 할당 가능
        - 따라서, `xn`의 값은 `null`이 되고 `println(xn)`은 `null`을 출력
    - **2. 플랫폼 타입 (`JTool!`)의 사용**

        ```kotlin
        val yn = JTool.get(null)
        println(yn) // null
        yn.method() // NullPointerException 발생
        ```

        - `yn`은 타입 추론에 의해 `JTool!`로 선언된 플랫폼 타입임
        - `yn`은 `null`을 가지고 있으며, `yn.method()`를 호출할 때 `NPE` 가 발생하는데 이 예외는 `yn`이 `null`이었기 때문에 발생하지만, 예외 메시지는 이와 관련된 정보를 제공하지 않음
        - 코틀린에서는 플랫폼 타입에서 `null`을 받을 수 있지만, 이를 역참조할 때 안전한 호출 연산자 `?.`을 사용하여 NPE를 방지할 수 있음
    - **3. Non-null 타입 (`JTool`)의 사용**

        ```kotlin
        val zn: JTool = JTool.get(null)
        ```

        - `zn`은 `JTool` 타입으로 선언된 널이 될 수 없는 타입임
        - 그러나 `JTool.get(null)`이 반환하는 값이 `null`이기 때문에, 이 값을 `zn`에 할당하려고 하면 코틀린 컴파일러는 널 가능성을 검사하여 `NPE` 를 발생시키는데. 이 경우,  예외 메시지에는 null이 할당되었다는 정보가 포함됩니다.
    - **요약**
        - **Nullable 타입 (`JTool?`)**: 안전하게 `null`을 할당받을 수 있고, 안전한 호출 연산자(`?.`)를 사용하여 `null`을 처리해야함
        - **플랫폼 타입 (`JTool!`)**: `null`을 할당받을 수 있지만, 이를 역참조할 때 `NPE` 가 발생할 수 있으며, 예외 메시지는 제한적임
        - **Non-null 타입 (`JTool`)**: `null`을 할당받으면 즉시 `NPE` 가 발생하며, 이 예외는 적절한 메시지를 포함함

## 널 가능성 어노테이션

- 자바 코드에 @Nullable 이나 @NonNullable 어노테이션을 사용하면 코틀린이 자바 타입을 nullable 타입과 non-nullable 타입으로 처리하도록 지정할 수 있음

    ```java
    public class JTool {
        @Nullable // 추가
        public static JTool get(String s) {
            if (s == null) return null;
            return new JTool();
        }
        @NotNull // 추가
        public String method() {
            return "success";
        }
    }
    ```

    ```kotlin
    fun main() {
        val xn: JTool? = JTool.get(null) // 1
        println(xn) // null
        val yn = JTool.get(null) // 2
        println(yn) // 3
        yn.method() // 4
        val zn: JTool = JTool.get(null) // 5
    }
    ```


### `JTool` 클래스에 `@Nullable` , `@NonNullable` 어노테이션을 사용하여 개선된 점

1. **널 가능한 타입으로 안전하게 처리**

    ```kotlin
    val xn: JTool? = JTool.get(null) // 1
    println(xn) // null
    ```

    - `@Nullable` 어노테이션이 `get` 메서드에 추가되어, 코틀린은 이 메서드의 반환 값이 널이 될 수 있음을 인식함
    - 따라서 `xn`을 `JTool?` 타입으로 선언하고 널 값을 안전하게 처리할 수 있음
2. **플랫폼 타입과 널 안전성**

    ```kotlin
    val yn = JTool.get(null)
    println(yn) 
    yn.method() // 컴파일 에러 발생
    ```

    - `@Nullable` 어노테이션이 추가되어 타입추론으로 `yn`은 `JTool?` 타입으로 추론됨
    - `yn` 이 null이 될 수 있으므로, 코틀린은 `yn`의 멤버 함수인 `method()`를 호출할 때 컴파일 에러가 발생시킴. 이 에러는 `yn`이 널일 수 있기 때문에 널 안전성을 보장하기 위해 `?.` 연산자 또는 `!!` 연산자를 사용해야 한다는 것을 의미
3. **널 체크로 인한 컴파일 에러로 NPE 방지**

    ```kotlin
    val zn: JTool = JTool.get(null) // 5
    ```

    - `@Nullable` 어노테이션이 추가되어, `JTool.get(null)`의 반환 타입은 `JTool?` 로 추론됨
    - `zn`은 null이 아닌 `JTool` 타입으로 선언되었으므로, `JTool.get(null)`의 결과가 `null`일 수 있기에 컴파일 에러가 발생함
    - 따라서, 널이 될 수 있는 값을 널이 아닌 타입에 할당하여 발생하는 `NPE`를 것을 방지할 수 있음

### 코틀린은 여러가지 널 가능성 어노테이션을 지원

- JSR-305 표준에 지정된 @Nullable, @CheckForNull
- 안드로이드 @Nullable과 @NotNull
- 젯브레인즈 도구가 지원히는@Nullable과 @NotNull (찾아보니 이게 제일 간편하고 자주 쓰인다고 함)
- 그 밖에 다른 널 가능성 애너테이션도 지원하며 코틀린 문서에서 확인 가능함

## 컬렉션과 자바

- 코틀린은 자바 표준 컬렉션 라이브러리를 사용해 더 개선된 코틀린 컬렉션을 제공한다.
    - 예를 들어 mutableList를 생성하면 실제로 자바 ArrayList가 생성된다.
- 자바 코드와 매끄럽게 통합하기 위해 코틀린은 자바 표준 라이브러리의 인터페이스를 사용하며, 자바와 똑같은 구현을 사용함.
    - 코틀린은 자바와 쉽게 혼합될 수 있어서 코틀린 컬렉션을 자바에 넘길 때 별도로 변환할 필요가 없음
    - 코틀린 개발자가 자바 표준 라이브러리에서 오랫동안 성능 튜닝을 거친 코드를 자동으로 활용할 수 있음
    - 코틀린 자체 라이브러리를 정의하지 않고 자바 컬렉션을 사용하므로 코틀린 애플리케이션에 포함된 표준 라이브러리 크기가 작은데, **코틀린 표준 라이브러리는 주로 자바 컬렉션을 향상시켜주는 확장 함수로 구성됨**

### 자바와 코틀린의 컬렉션

1. **자바의 컬렉션**
    - 자바에서는 `java.util.List`, `java.util.Set`, `java.util.Map` 등 모든 컬렉션이 기본적으로 변경 가능함
    - 컬렉션의 불변성을 보장하기 위해, 자바는 컬렉션을 감싸서 내부를 변경할 수 없도록 하는 래퍼를 제공하지만, 정적 타입 검사를 통해 완전한 안전성을 보장하지는 않음
2. **코틀린의 컬렉션**
    - 코틀린에서는 컬렉션의 가변성과 읽기 전용 성격을 명확히 구분하여 제공함
    - **읽기 전용 컬렉션**: 기본적으로 코틀린에서 제공하는 `Collection`, `List`, `Set`, `Map` 인터페이스는 읽기 전용으로, 이들은 컬렉션의 내용을 변경할 수 있는 메서드를 제공하지 않음
    - **가변 컬렉션**: `MutableCollection`, `MutableList`, `MutableSet`, `MutableMap` 인터페이스를 통해 가변 컬렉션을 명시적으로 정의하며, 이들은 컬렉션의 내용을 수정할 수 있는 메서드를 제공함

### 자바와 코틀린의 컬렉션 인터페이스

- 코틀린은 자바의 컬렉션 인터페이스를 그대로 따르지만, **읽기 전용** 인터페이스와 **가변** 인터페이스를 명확히 분리함
    - **읽기 전용**: `Collection`, `List`, `Set`, `Map`
    - **가변**: `MutableCollection`, `MutableList`, `MutableSet`, `MutableMap`
- **자바와의 호환성**:
    - 코틀린의 `List`와 `MutableList`는 자바의 `java.util.List`와 호환됩니다. 그러나 코틀린에서는 `java.util.List` 대신 코틀린의 `List`를 사용하는 것이 좋다.
    - 코틀린에서 `List`를 자바의 `java.util.List`로 변환할 수 있지만, 코틀린의 컬렉션을 자바로 전달할 때는 주의가 필요함. 코틀린의 읽기 전용 참조는 내부적으로 가변 컬렉션을 참조할 수 있으므로, 자바 코드에서 이를 변경할 수 있기 때문

    ```kotlin
    fun main() {
        // 가변 리스트 생성
        val mutable = mutableListOf(1, 2, 3)
        // 읽기 전용 참조 생성
        val list: List<Int> = mutable
        // 가변 리스트에 요소 추가
        mutable += 4
        // 읽기 전용 참조가 가리키는 리스트 내용이 변경됨
        println(list) // 출력: [1, 2, 3, 4]
    }
    
    ```

    - 이 예제에서 `list`는 읽기 전용 참조이지만, `mutable`이 변경되면서 `list`의 내용도 변경된다.
      (`list`가 가리키는 컬렉션이 여전히 가변적이기에)

## 자바 원시 타입

### 자바의 타입 처리

1. **참조 타입과 원시 타입**
    - **참조 타입** :
        - 객체를 생성할 때 `new` 키워드를 사용하여 힙 메모리에 객체를 생성함
    - **원시 타입** :
        - 자바에서는 `int`, `boolean`, `long`, `char`, `byte`, `short`, `float`, `double` 이 원시 타입으로 정의됨
        - 원시 타입은 객체가 아니기에 null을 가질 수 없어, 제네릭 타입 인자로 사용할 수 없음.
        - 원시 타입을 제네릭 인자로 사용하거나 null을 대입하려면, 자바 표준 라이브러리에서 제공하는 참조 타입 (`java.lang.Integer`, `java.lang.Boolean` 등)을 사용해야 함
2. **래퍼 타입**
    - 원시 타입 값을 힙에 저장할 때는 `Integer`, `Boolean` 등과 같은 래퍼 타입을 사용함.
    - 이런 타입을 래퍼 타입(Boxed type)이라고 하며, 원시 타입을 감싸기만 하여 힙에 저장

### 코틀린의 타입 처리

1. **단일 타입 사용**
    - 코틀린에서는 자바와 달리 원시 타입과 래퍼 타입을 구분하지 않음
    - `Int`, `Boolean` 등은 원시 타입과 래퍼 타입을 구분하지 않고 단일 타입으로 사용됨
2. **타입 변환**
    - 코틀린은 JVM에서 자바와 동일한 원시 타입 지원을 채택하기에 따라서 코틀린 코드가 바이트코드로 컴파일될 때 `Int` 타입은 JVM의 `int`로 변환되며, 널이 될 수 있는 `Int?` 타입이나 제네릭 타입 인자로 사용되는 `Int`는 래퍼 타입으로 표현된다.

    ```kotlin
    fun main() {
        // 원시 타입 int가 JVM에서 int로 만들어진다
        val i = 10
    
        // 래퍼 타입이 JVM에서 Integer로 만들어진다
        val iOrNull: Int? = null
    
        // 리스트는 원시 타입을 래퍼 타입으로 사용하여 List<Integer>로 변환된다
        val list: List<Int> = listOf(1, 2, 3)
    }
    ```


### 요약

- **자바**: 원시 타입과 참조 타입을 명확히 구분하고, 원시 타입을 사용할 때는 힙 메모리 대신 스택 메모리를 사용하여 성능을 향상시킨다. 원시 타입의 값을 래퍼 타입으로 감싸서 힙에 저장할 수 있다.
- **코틀린**: 자바와 달리 원시 타입과 래퍼 타입을 구분하지 않으며, 모든 타입을 단일 형태로 사용한다. 코틀린은 가능한 경우 바이트코드에서 원시 타입을 사용하려고 하며, 널이 될 수 있는 타입이나 제네릭 타입 인자는 래퍼 타입으로 변환된다.

> ***코틀린은 원시타입, 래퍼타입을 구분하지 않으니까 원시타입도 힙에 저장되나?***
>
>코틀린에서는 원시 타입과 래퍼 타입을 구분하지 않지만, JVM에서 알아서 이를 다르게 처리하는데,
`Int` 타입의 지역 변수나 `val`, `var`로 선언된 변수는 일반적으로 스택 메모리에 저장된다.
반면, `Int?`와 같은 널을 허용하는 타입은 힙 메모리에서 박싱 타입(`Integer`)으로 저장된다.
