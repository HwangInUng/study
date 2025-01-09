## 8. 고차 함수: 파라미터와 반환 값으로 람다 사용
>람다를 인자로 받거나 반환하는 함수인 고차 함수(high order function)에 대해서 알아보자.

- 고차 함수를 이용한 코드 중복을 없애고, 추상화하는 방법을 알아보고,
- 람다 사용에 따라 발생할 수 있는 성능상 부가 비용이 없는 인라인 함수에 대해 알아보자.

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

대부분의 언어에서 사용되는 람다 함수의 `화살표(->)`는 **화살표 좌항의 파라미터를 우항의 실행부로 전달한다**라는 의미로 받아들이면 된다.
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
상황에 따라 수행해야하는 로직이 달라지는 경우 어떤 로직을 선택하여 함수로 반환하는 방법이 사용될 수 있다.

```kotlin
enum class Delivery { STANDARD, EXPEDITED }

class Order(val itemCount: Int)

fun getShippingCostCalculator(
        delivery: Delivery): (Order) -> Double {
    // 조건에 따라 람다를 반환
    if (delivery == Delivery.EXPEDITED) {
        return { order -> 6 + 2.1 * order.itemCount }
    }
    return { order -> 1.2 * order.itemCount }
}

fun main(args: Array<String>) {
    val calculator = getShippingCostCalculator(Delivery.EXPEDITED)
    val order = Order(3)
    println(calculator(order))
}
```

- 반환 타입으로 함수 타입을 지정
- return 식에 람다, 멤버 참조, 함수 타입의 값을 계산하는 식 등을 작성

#### 8.1.6 람다를 활용한 중복 제거
반복되거나 재사용되는 코드의 중복을 제거해보자.

```kotlin
data class SiteVisit(
        val path: String,
        val duration: Double,
        val os: OS
)

enum class OS { WINDOWS, LINUX, MAC, IOS, ANDROID }

fun main(args: Array<String>) {
    val log = listOf(
            SiteVisit("/", 34.0, OS.WINDOWS),
            SiteVisit("/", 22.0, OS.MAC),
            SiteVisit("/loign", 12.0, OS.LINUX),
            SiteVisit("/signup", 8.3, OS.IOS),
            SiteVisit("/", 16.3, OS.ANDROID),
    )
    // filter의 술어로 OS.WINDOWS 사용
    val averageWindowsDuration = log.filter { it.os == OS.WINDOWS }
            .map(SiteVisit::duration)
            .average()

    println(averageWindowsDuration)
}
```
중복을 제거하기 위해서 `OS.WINDOWS` 부분을 파라미터로 전달해보자.

```kotlin
// OS를 파라미터로 전달
fun List<SiteVisit>.averageDuration(os: OS) = filter { it.os == os }.map(SiteVisit::duration).average()

fun main(args: Array<String>) {
    val log = listOf(...)
    println(log.averageDuration(OS.WINDOWS))
}
```

- 플랫폼 비교 표현만으로는 모바일 디바이스 사용자의 평균 방문 시간 등을 구하긴 제한
- 복잡한 질의에 따른 분석이 제한

위 문제를 해결할 수 있는 고차 함수를 정의해보자.

```kotlin
// 고차 함수 선언
// 파라미터로 함수 타입을 전달
fun List<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean) = filter(predicate).map(SiteVisit::duration).average()

fun main(args: Array<String>) {
    val log = listOf(
            SiteVisit("/", 34.0, OS.WINDOWS),
            SiteVisit("/", 22.0, OS.MAC),
            SiteVisit("/loign", 12.0, OS.LINUX),
            SiteVisit("/signup", 8.3, OS.IOS),
            SiteVisit("/", 16.3, OS.ANDROID),
    )
    // 모바일 디바이스 조회
    println(log.averageDurationFor { it.os in setOf(OS.ANDROID, OS.IOS) })
    // 특정 플랫폼의 경로 조회
    println(log.averageDurationFor { it.os == OS.IOS && it.path == "/signup" })
}
```

- 코드의 일부분을 람다로 정의하여 파라미터로 전달
- 함수 타입은 전략 패턴으로 동작할 수 있기 때문에 람다식이 없다면 인터페이스를 전달하고, 있다면 함수 타입을 사용

**고차 함수와 전략 패턴**
|구분|고차 함수|전략 패턴|
|---|---|---|
|정의|다른 함수를 인자로 받거나 반환할 수 있는 함수|객체 지향 디자인 패턴 중 하나로 특정 동작을 동적으로 변경할 수 있도록 하는 패턴|
|접근방식|함수형 프로그래밍|객체 지향 프로그래밍|
|동작 전달 방식|함수를 인자로 전달|객체를 인자로 전달|
|코드 간결성|매우 간결|인터페이스 및 클래스 정의 필요|
|상태 유지|불가능(무상태 함수)|가능(전략 객체에서 상태 보유)|

고차 함수와 전력 패턴은 **행동의 동적 변경**이라는 동일한 목적을 위해 사용되며, 모듈화를 통한 코드의 중복을 줄인다.
하지만 위의 표에 정리된 것처럼 특징이 존재하기 때문에 용도에 따라 사용할 수 있다.

- 고차 함수 : 전략이 간단하고, 간결하게 동작을 표현하거나 인터페이스 및 구현체 관리가 불필요한 경우에 사용
- 전략 패턴 : 상태를 유지해야하거나, 명시적인 인터페이스로 여러 구현체를 관리해야하는 경우

---

### 8.2 인라인 함수 : 람다의 부가 비용 없애기
람다를 활용한 코드의 성능이 어떨지 생각해보자.

- 람다 식을 사용할 때마다 새로운 클래스가 만들어지지 않음
- 변수를 포획하면 람다가 생성되는 시점마다 새로운 무명 클래스 객체가 생성
- 실행 시점에 무명 클래스 생성에 따른 부가 비용 발생

결론적으로 일반 함수를 사용한 구현보다 덜 효율적이다.

#### 8.2.1 인라이닝이 작동하는 방식
`inline` 변경자를 어떤 함수에 붙이면 컴파일러가 해당 함수를 호출하는 모든 문장을 함수 본문에 해당하는 바이트코드로 바꿔치기 한다.
즉, 함수를 호출하는 바이트코드 대신 함수 본문을 번역한 바이트 코드로 컴파일한다는 의미다.

- 람다 사용 시 함수 호출 오버헤드를 제거하여 성능 최적화 달성
- 람다 캡처 비용 감소를 위해 클로저 객체 생성을 회피

```kotlin
inline fun <T> synchronized(lock: Lock, action: () -> T): T {
    lock.lock()
    try {
        return action()
    }
    finally {
        lock.unlock()
    }
}
```

- 자바에서는 임의의 객체에 대해 `synchronized`를 사용할 수 있다.
- 코틀린에서도 모든 타입을 인자로 받는 `synchronized` 함수를 제공한다.
- 동기화에 명시적인 락을 사용하기 위해서는 `withLock` 함수를 사용한다.
- `withLock`은 락을 건 상태에서 코드를 실행해야 할 때 먼저 고려해볼만 하다.

```kotlin
fun foo (l: Lock) {
    println("Before)

    // 인라이닝된 코드
    synchronized(l) {
        // 람다의 본문이 인라이닝된 코드
        println("Action")
    }

    println("After")
}
```

위 코드에서 `{println("Action")}`은 람다이지만 해당 람다를 호출하는 코드 정의의 일부분으로 간주되기 때문에 함수 인터페이스를 구현하는 무명 클래스로 감싸지 않는다.

```kotlin
class LockOnwer(val lock: Lock) {
    fun runUnderLock (body: () -> Unit) {
        // 변수에 저장된 람다 코드를 알 수 없는 상태
        synchronized(l, body)
    }
}
```

이런 상황에서는 람다가 일반적인 경우(무명 클래스로 감싸지는)와 동일하게 호출된다.

- 한 인라인 함수를 두 곳에서 각각 다른 람다를 사용해 호출한다면 각각 인라이닝된다.
- 각 람다의 본문이 인라인 함수의 본문 코드에서 람다를 사용하는 위치에 복사된다.

#### 8.2.2 인라인 함수의 한계

- 람다를 사용하는 모든 함수를 인라이닝할 수는 없음
- 해당 함수에 인자로 절달된 람다 식의 본문은 결과 코드에 직접 들어갈 수 있음
- 직접 펼쳐지기 때문에 함수가 파라미터로 전달받은 람다를 본문에 사용하는 방식이 한정적임
- 단, 파라미터로 받은 람다를 변수에 저장하고 해당 변수를 사용한다면 어딘가에 객체가 존재해야하기 때문에 인라이닝이 불가능

```kotlin
fun <T, R> Sequence<T>.map(transform: (T) -> R): Sequence<R> {
    // 새로운 객체에 주 생성자 파라미터로 함수 값을 전달
    return TransformingSequence(this, transform)
}
```

- 전달받은 람다를 프로퍼티로 저장
- `transform` 인자를 일반적인 함수 표현으로 만들 수 밖에 없으며
- 함수 인터페이스를 구현하는 무명 클래스 인스턴스로 만들어야만 함

이유는 map 내부에서 `TransformingSequence`의 주 생성자로 전달된 `transform`이 프로퍼티로 저장되기 때문에
어딘가에는 객체가 존재해야한다는 특성으로 인해 인라인이 불가능한 것이다.

**복수 개의 람다 중 일부 인라인 금지**
```kotlin
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) {...}
```

`noinline` 키워드를 사용해 인라이닝에서 제외할 수 있다.

#### 8.2.3 컬렉션 연산 인라이닝
인라인 함수의 한계에서는 `Sequence`의 표준 라이브러리 함수는 인라이닝 되지 않고 동작한다고 배웠다.
반면 컬렉션의 표준 라이브러리 함수는 대부분 인라인을 지원한다.

그렇다면 컬렉션과 `Sequence`의 표준 라이브러리 함수 지원에 차이가 발생하는 이유가 뭘까?

|구분|Sequence|컬렉션|
|---|---|---|
|동작 방식|지연 실행 방식(lazy evaluation)|즉시 실행 방식(eager evaluation)|
|인라인 여부|X|O|
|특징|실행 계획이 저장되어 최종 연산 시 동작|함수 호출 오버헤드를 감소|
|클로저 객체 생성|O|X|

컬렉션과 `Sequence`의 가장 큰 차이는 동작 방식이다.

즉시 실행을 위해 클로저 객체 생성 및 함수 호출 오버헤드를 감소시켜야하는 것이 컬렉션의 동작 방식이고,
지연 실행을 위해 실행 계획을 저장해놓고, 중간 결과 없이 최종 연산을 한번에 수행하는 것이 시퀀스의 동작 방식이다.

위와 같이 이유 떄문에 모든 컬렉션을 시퀀스로 변환하는 `asSequece`를 호출해서는 안된다.
컬렉션의 크기가 큰 경우에만 성능적인 이점을 달성 할 수 있다.

#### 8.2.4 함수를 인라인으로 선언해야 하는 경우
inline 키워드를 사용한다고 무조건 함수 성능을 높일 수 없는데 그 이유는 다음과 같다.

- 일반 함수는 JVM의 JIT 과정에서 코드 실행을 분석하여 인라이닝을 지원한다.
- JVM 최적화를 활용하면 각 함수 구현이 정확히 한 번만 있으면 된다.
- 반면, 코틀린 인라인 함수는 바이트 코드에서 각 함수 호출 지점을 함수 본문으로 대치한다.
- 이런 이유로 코드 중복이 발생한다.

그렇다면 람다를 인자로 받는 함수의 인라이닝을 어떨까?

- 함수 호출 비용, 람다 표현 클래스 및 인스턴스에 해당하는 객체를 만들지 않아도된다.
- 일반 람다에서 사용할 수 없는 넌로컬 반환 등의 기능을 사용할 수 있다.

#### 8.2.5 자원 관리를 위해 인라인된 람다 사용

```kotlin
fun <T>Lock.withLock(action: () -> T): T {
    lock()
    try {
        return action()
    } finally {
        unlock()
    }
}
```
위 코드는 락을 획득한 후 작업하는 과정을 `action`이라는 별도 함수로 분리한다.

**use**
- 닫을 수 있는 자원에 대한 확장 함수
- 람다를 인자로 받으며 람다를 호출한 다음 자원을 닫아줌
- 예외가 발생하더라도 자원을 확실히 닫음

```kotlin
fun readFirstLineFromFile(path: String): String {
    BufferedReader(FIleReader(path)).use {br ->
        return br.readLine()
    }
}
```

- `BufferedReader` 객체를 만들고 `use` 함수를 호출하여 연산을 수행할 람다를 넘긴다.
- 람다에서 결과를 반환하는 것이 아닌 외부 함수에서 결과를 반환한다.

---

### 8.3 고차 함수 안에서 흐름 제어
루프와 같은 명령형 코드를 인자로 전달되는 람다 안에서 사용할 떄 벌어지는 일들을 알아보자.

#### 8.3.1 람다 안의 return문 : 람다를 둘러싼 함수로부터 반환

**for 루프**
```kotlin
data class Ch8Person (val name: String, val age: Int)

fun lookForAlice (people: List<Ch8Person>) {
    for(person in people) {
        if(person.name == "Alice") {
            println("Ok")
            return
        }
    }
    println("No")
}

fun main(args: Array<String>) {
    val people = listOf(Ch8Person("Alice", 29), Ch8Person("Bob", 33))
    println(lookForAlice(people))
}
```

위 코드를 다양한 컬렉션 함수를 통해 실행시켜보면서 return 사용에 대하여 알아보자.

**forEach**
```kotlin
fun lookForAlice(people: List<Ch8Person>) {
    people.forEach {
        if (it.name == "Alice") {
            println("Ok")
            return
        }
    }
    println("No")
}
```

`forEach`를 사용하더라도 return에는 문제가 없는 모습을 볼 수 있다.
람다에서 return은 람다를 호출하는 함수가 실행을 끝내고 반환되며 이런 return 형태를 `non-local return`이라 부른다.

- 자바에서는 for, synchronized 함수 등이 함수 블록을 중단하고 반환한다.
- 코틀린에서도 람다를 함수로 받을 경우 자바의 return과 동일하게 동작한다.
- 이런 경우는 인라인 함수인 경우에만 해당한다.

#### 8.3.2 람다로부터 반환: 레이블을 사용한 return

- 람다 식에서 레이블을 이용하여 로컬 return 사용 가능
- `break`와 유사한 역할을 수행
- 이 경우 람다의 실행을 끝내고, 람다를 호출한 코드의 실행을 이어감

```kotlin
fun lookForAlice(people: List<Ch8Person>) {
    people.forEach lable@{
        if (it.name == "Alice") return@lable
        // if (it.name == "Alice") return@forEach 식으로 함수 이름으로 레이블 가능
    }
    println("No")
}
```
이 경우 항상 "No"가 출력된다.
