## 8. 고차 함수: 파라미터와 반환 값으로 람다 사용
>람다를 인자로 받거나 반환하는 함수인 고차 함수(high order function)에 대해서 알아보자.

- 고차 함수를 이용한 코드 중복을 없애고, 추상화하는 방법을 알아보고,
- 람다 사용에 따라 발생할 수 있는 성능상 부가비용을 없는 인라인 함수에 대해 알아보자.

### 8.1 고차 함수 정의
코틀린에서의 고차 함수는 다음과 같다.

- 람다나 함수 참조를 인자로 넘길 수 있는 함수
- 람다나 함수 참조를 반환하는 함수
- 두 가지를 모두 충족하는 함수

```kotlin
// 술어 함수를 인자로 전달 받는 filter
list.filter {x > 0}
```

#### 8.1.1 함수 타입
람다 인자의 타입을 어떻게 선언하는지부터 살펴본다.

**람다를 변수에 대입**
```kotlin
val sum = {x:Int, y:Int -> x + y}
val action = {println(sum(42)}
```

- 변수의 타입을 지정하지 않음
- 람다의 파라미터 타입만 지정
- 컴파일러가 `sum`과 `action`을 함수 타입으로 추론

**람다 타입 선언 추가**
```kotlin
val sum: (Int, Int) -> Int = {x, y -> x + y}
// 반환 값 없음
val action: () -> Unit = {println(42)}
```

- 함수 파라미터의 타입은 괄호 안에 순서대로 작성
- 화살표 추가
- 함수를 수행한 반환 타입을 지정
- 함수 타입을 선언할 때는 반환 타입을 반드시 명시

대부분의 언어에서 사용되는 람다 함수의 `화살표(->)`는 **'화살표 좌항의 파라미터를 우항의 실행부로 전달한다'**라는 의미로 받아들이면 된다.
또한 우항에 단일 표현식이 있는 경우 결과를 암시적으로 반환한다.

```kotlin
// 단일 표현식 : 암시적 반환
val sum = {x:Int, y:Int -> x + y}

// 여러 문장 : 명시적 반환
// 반드시 레이블을 사용
val sum2: (Int, Int) -> Int = sum2@{ x, y ->
    val result = x + y
    return@sum2 result
}
```

- Kotlin에서 람다의 결과를 명시적으로 반환하기 위해서는 `@sum2`와 같은 레이블을 반드시 사용
- Kotlin의 람다에서의 return은 기본적으로 상위 함수 전체를 종료하려는 시도로 동작
- 이를 방지하기 위해 특정 람다를 대상으로 명시적 변환을 하려면 레이블을 반드시 지정

```kotlin
val sum = (Int, Int) -> = sum@{ // 레이블 지정
  ...실행문
  return@sum 결과 // 지정된 레이블을 호출하여 명시적 반환
}
```

**널로 반환 타입 지정**
```kotlin
// 함수의 실행 결과가 널
var canReturnNull: (Int, Int) -> Int? = {x, y -> null}

// 함수 자체가 널
var funOrNull: ((Int, Int) -> Int)? = null
```

- 함수의 실행 결과가 널인 타입
- 함수 타입 자체가 널인 타입

**파라미터 이름과 함수 타입**
```kotlin
fun performRequest (
  url: String,
  callback: (code: Int, content: String) -> Unit
) {
  ...실행부
}
```

- `callback`은 파라미터로 전달된 함수 타입의 이름
- 함수 타입의 파라미터 이름은 지정된 것을 반드시 사용해야하는 것은 아님

#### 8.1.2 인자로 받은 함수 호출
고차 함수를 어떻게 구현하는지 알아보자.

```kotlin
// 함수 타입인 파라미터를 선언
fun twoAndThree(operation: (Int, Int) -> Int) {
    println(operation(2, 3))
}

fun main(args: Array<String>) {
    twoAndThree { x, y -> x + y }
}
```

- 고차 함수의 파라미터로 람다를 전달
- 고차 함수 내부에서는 함수 타입의 파라미터를 일반 함수와 동일하게 호출하여 로직을 수행

**자바에서의 구현**
```java
// 함수형 인터페이스 선언
@FunctionalInterface
interface Operation {
    int apply(int x, int y);
}

class HigherOrderFunctionExample {
    // 고차 함수 구현
    public static void twoAndThree(Operation operation) {
        System.out.println(operation.apply(2, 3));
    }

    public static void main(String[] args) {
        // 람다를 사용하여 고차 함수 호출
        twoAndThree((x, y) -> x + y);
    }
}
```

- 자바는 람다를 직접 전달할 수 없기에 함수형 인터페이스 구현이 필수적
- 람다식으로 익명 클래스 없이 간단하게 전달 가능

**filter 구현**
```kotlin
fun String.filter(predicate: (Char) -> Boolean): String
```

- `String.filter` : 수신 객체 타입
- `predicate` : 파라미터 이름(함수 타입)
- `(Char) -> Boolean` : 파라미터 함수 타입
- `:String` : 반환 타입

```kotlin
fun String.filter(predicate: (Char) -> Boolean) : String {
    val sb = StringBuilder()

    // indices : 수신 객체(String)의 길이 범위
    for(index in indices) {
        val element = get(index)
        if(predicate(element)) sb.append(element)
    }

    return sb.toString()
}

fun main(args: Array<String>) {
    // 알파벳 범위 내의 문자 반환
    println("ab1c".filter{it in 'a'..'z'})
}
```

- 문자열의 각 문자가 술어 조건에 해당하면 true
- 그렇지 않으면 false를 반환하여
- true에 해당하는 조건만 StringBuilder에 추가하여 반환

#### 8.1.3 자바에서 코틀린 함수 타입 사용
- 컴파일된 코드 안에서 함수 타입은 일반 인터페이스로 변환
- `FunctionN` 인터페이스를 구현하는 객체로 저장
- 각 인터페이스에 `invoke` 메서드 정의가 하나 포함
- 인자 개수에 따라 적당한 `FunctionN` 인터페이스를 구현하는 클래스의 인스턴스를 저장
- `invoke` 메서드 본문에는 람다의 본문이 들어감

**함수 타입 변환**
```kotlin
val example: (Int, Int) -> Int = {x, y -> x + y}

// FunctionN : 인자 N개 (최대 22개까지 지원)
Function2<T1, T2, R>
```

- `example`에 대입한 함수 타입은 JVM에서 직접 지원되지 않음
- `FunctionN`이라는 인터페이스 계층으로 변환
- `invoke` 메서드에 람다의 본문인 `x + y`가 포함

**FunctionN 인터페이스**
```kotlin
public interface Function2<T1, T2, R> : Function<R> {
  public operator fun invoke(p1: T1, p2:T2): R
}

val sum: (Int, Int) -> Int = {x, y -> x + y}
println(sum(2,3))

// 실제 호출
sum(2, 3) -> sum.invoke(2, 3)
```

- 단일 메서드로 `invoke` 메서드를 정의하여 함수 타입 호출 시 실행
- 제네릭 타입으로 파라미터의 유형 및 개수와 반환 타입을 결정
- `invoke`는 함수 호출을 처리하는 메서드로 동작

**자바로 컴파일**
```java
Function2<Integer, Integer, Integer> sum = (x, y) -> x + y;
System.out.println(sum.invoke(3, 5)); // invoke 메서드 호출
```

**자바에서 호출**
```kotlin
// 코틀린 선언
fun processTheAnswer(f: (Int) -> Int) {
  println(f(42))
}
```

```java
// 자바 호출
processTheAnswer(number -> number + 1);
```

`Unit`이 반환 타입인 함수나 람다를 자바로 작성할 때에는 `Unit.INSTANCE`를 명시적으로 작성하여 반환값을 지정해줘야 한다.
코틀린에서 `Unit`은 값이 존재하기 때문에 자바의 `void`로 대체 할 수 없다.

#### 8.1.4 디폴트 값을 지정한 함수 타입 파라미터나 널이 될 수 있는 함수 타입 파라미터
디폴트 값을 지정하면 다음과 같은 이점이 있다.

- 기본 동작이 보장되므로 함수 호출에 대한 불편함을 감소
- 필요한 경우에는 기본 동작 이외의 동작을 전달하여 처리가 가능

**하드 코딩**
```kotlin
fun <T> Collection<T>.joinToString(
        separator: String = ", ",
        prefix: String = "",
        postfix: String = ""
):String {
    val result = StringBuilder(prefix)

    for ((index, element) in this.withIndex()) {
        if(index > 0) result.append(separator)

        // toString() 메서드를 사용하여 객체를 문자열로 변환
        result.append(element)
    }

    result.append(postfix)

    return result.toString()
}
```

- 컬렉션의 각 원소를 문자열로 변환하는 방법에 대한 제어가 부족
- 람다를 통해 원소를 문자열로 바꾸는 방법으로 해결이 가능

```kotlin
fun <T> Collection<T>.joinToString(
        separator: String = ", ",
        prefix: String = "",
        postfix: String = "",
        // 람다 default 값 지정
        transform: (T) -> String = { it.toString() }
): String {
    val result = StringBuilder(prefix)

    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        // transform 함수를 호출
        result.append(transform(element))
    }

    result.append(postfix)

    return result.toString()
}

fun main(args: Array<String>) {
    val letters = listOf("Alpha", "Beta")

    // 기본 동작 : toString()
    println(letters.joinToString())
    // 람다를 인자로 전달 : lowercase()
    println(letters.joinToString { it.lowercase() })
    // 여러 인자 전달
    println(letters.joinToString(separator = "! ", postfix = "! ") {
        it.uppercase()
    })
}
```

- 제네릭 함수이기에 컬렉션의 원소 타입을 T로 받음
- 디폴트 값에 의한 특별한 구문이 필요하지 않음
- `toString`이 디플트로 동작하며 다른 동작을 원할 시 `lowercase`, `uppercase` 등을 호출 가능

**널이 될 수 있는 함수 타입**
```kotlin
fun foo(callback: (() -> Unit)? {
  //...생략

  if(callback != null) {
    callback()
  }
}
```

- 널이 될 수 있는 함수 타입을 받으면 그 함수를 직접 호출 할 수 없음
- 반드시 타입 가드를 사용하여 널 안정성 검사를 수행한 이후에 스마트 캐스팅을 통해 사용 가능

```kotlin
fun <T> Collection<T>.joinToString(
        separator: String = ", ",
        prefix: String = "",
        postfix: String = "",
        // 람다 default 값 지정
        transform: ((T) -> String)? = null
): String {
    val result = StringBuilder(prefix)

    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        // null이 아닌 경우 실행, null인 경우 디폴트 설정과 동일
        result.append(transform?.invoke(element) ?: element.toString())
    }

    result.append(postfix)

    return result.toString()
}
```

엘비스 연산자를 이용하여 함수 타입이 null이 아닌 경우 실행하고, null인 경우 `toString`을 기본으로 실행하도록 지정한다.

#### 8.1.5 함수를 함수에서 반환
