## 7. 연산자 오버로딩과 기타 관례
>특정 함수 이름과 연관된 언어의 기능을 연결해주는 기법인 관례(convention)에 대해서 알아보자.

### 7.1 산술 연산자 오버로딩
코틀린은 자바와 다르게 다양한 자료형에 대하여 산술 연산자를 지원한다. add() 등과 같은 메서드를 사용하기 보다 +와 같은 산술 연산자를 사용하는 편이 더 편할 수 있다.
이런 부분을 코틀린은 가능하게 한다.

#### 7.1.1 이항 산술 연산 오버로딩
plus라는 이름의 함수를 정의하여 + 연산을 수행할 수 있다.

```kotlin
data class Point(val x: Int, val y: Int) {
    // plus 정의
    operator fun plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }
}


fun main(args: Array<String>) {
    val point1 = Point(10, 20)
    val point2 = Point(30, 40)
    
    // + 연산자 사용 시 내부적으로 plus() 호출
    println(point1 + point2)
}
```

- `operator` 키워드는 어떤 함수가 관례를 따르는 함수인지 명시하기 위하여 반드시 사용해야한다.
- `point1 + point2`는 내부적으로 `point1.plus(point2)`로 동작한다.
- 확장 함수로도 정의가 가능하다.

```kotlin
// 확장 함수
operator fun Point.plus(other: Point): Point {
    return Point(x + other.x, y + other.y)
}
```

**오버로딩 가능한 이항 산술 연산자**
|산술 연산자|함수 이름|
|---|---|
|*|times|
|/|div|
|%|mod(1.1부터 rem)|
|+|plus|
|-|minus|

연산자 우선순위는 동일하게 유지된다.

- 자바에서 정의된 메서드를 코틀린에서 호출할 경우도 관례에 맞아 떨어지면 연산자 식 사용이 가능하다.
- 함수의 이름과 파라미터의 개수만 일치하게 사용하면 된다.

**자바 메서드를 연산 오버로딩으로 호출**
자바 메서드 정의
```java
public class Calculator {
    private int value;

    public Calculator(int value) {
        this.value = value;
    }

    // plus 연산 오버로딩 역할
    // 전달받은 정수를 새로운 객체의 생성자 인자로 전달
    public Calculator plus(int number) {
        return new Calculator(this.value + number);
    }

    @Override
    public String toString() {
        return "Calculator(value=" + value + ")";
    }
}
```
코틀린에서 호출
```kotlin
fun main() {
    val calculator = Calculator(10)

    // 자바의 plus 메서드를 명시적으로 호출
    val sum = calculator.plus(20)
    println(sum) // Calculator(value=30)

    // 코틀린의 연산 오버로딩을 사용
    // Calculator.plus(Int) 호출
    val result = calculator + 20
    println(result) // Calculator(value=30)
}
```

- 연산자를 정의할 때 두 피연산자가 같은 타입일 필요는 없다.
- 자동으로 교환 법칙(commutativity)을 지원하지 않는다.
- 연산자 함수의 반환 타입은 두 피연산자 중 하나가 아니더라도 상관없다.
- 오버로딩이 가능하기 때문에 이름이 같더라도 파라미터 타입이 다르다면 서로 다른 연산자 함수로 동작한다.

```kotlin
// Double 자료형을 매개변수로 정의
// 반환 값을 String으로 정의
operator fun Point.times(scale: Double): String {
    val p = Point((x * scale).toInt(), (y * scale).toInt())
    return p.toString()
}

fun main(args: Array<String>) {
    val p = Point(10, 20)

    // Point 자료형과 Double 자료형을 이용한 연산 수행
    println(p * 1.5)
}

>> Point(x=15, y=30)
```

**비트 연산자에 대한 부분**
표준 숫자 타입에 대해 비트 연산자를 정의하지 않고, 중위 연산자 표기법을 지원하는 일반 함수를 사용해 비트 연산을 수행한다.

|함수|설명|자바 연산자|
|:-:|---|:-:|
|shl|왼쪽 시프트|<<|
|shr|오른쪽 시프트(부호 비트 유지)|>>|
|ushr|오른쪽 시프트(0으로 부호 비트 설정|>>>|
|and|비트 곱|&|
|or|비트 합|\||
|xor|비트 배타 합|^|
|inv|비트 반전|~|

#### 7.1.2 복합 대입 연산자 오버로딩
`+=`와 같은 복합 대입 연산자에 대한 오버로딩을 알아보자.

- 변수가 변경 가능한 경우에만 복합 대입 연산자 사용이 가능하다.
- 복합 대입 연산자의 경우 원래 객체의 내부 상태를 변경하게 만들고 싶은 경우 사용한다.
- 연산 오버로딩에 사용한 함수 이름에 `Assign`을 붙여주면 된다.

**오버로딩 가능한 이항 복합 대입 연산자**
|산술 연산자|함수 이름|
|---|---|
|*=|timesAssign|
|/=|divAssign|
|%=|modAssign(1.1부터 remAssign)|
|+=|plusAssign|
|-=|minusAssign|

```kotlin
// 복합 대입 연산자 정의
operator fun <T> MutableCollection<T>.plusAssign(element: T) {
    // 현재 객체에 요소 추가
    this.add(element)
}

fun main(args: Array<String>) {
    val list = arrayListOf(1, 2)

    list += 5

    println(list.get(2))
}

>> 5
```

- 산술 연산자와 복합 대입 연산자를 동시에 정의하지 말아야 한다.
- 변경 불가능한 경우 새로운 값을 반환하는 산술 연산을 사용한다.
- 변경이 가능한 클래스를 설계할 때는 복합 대입 연산을 사용한다.

컬렉션의 경우 산술 연산 오버로딩은 항상 새로운 컬렉션을 반환하고, 복합 대입 연산 오버로딩은 항상 변경 가능한 컬렉션에 작용하여 객체 상태를 변화 시킨다.

```kotlin
fun main(args: Array<String>) {
    // 객체의 상태만 변경 : 5를 추가
    val list = arrayListOf(1, 2)
    list += 5

    // 새로운 객체 반환
    val newList = list + listOf(4, 5)

    println(list)
    println(newList)
}
```

#### 7.1.3 단항 연산자 오버로딩
단항 연산자 오버로딩의 특징은 `unary`를 접두사로 사용하며, 함수 인자를 취하지 않는다.

```kotlin
operator fun Point.unaryMinus(): Point {
    return Point(-x, -y)
}
```

**오버로딩 가능한 단항 산술 연산자**
|연산자|함수 이름|
|---|---|
|+a|unaryPlus|
|-a|unaryMinus|
|!a|not|
|++a, a++|inc|
|--a, a--|dec|

- inc, dec는 일반적인 전위와 후위 증가/감소 연산자와 같은 의미를 제공한다.
- 전위와 후위 연산을 처리하기 위해 별다른 처리를 할 필요가 없다.

```kotlin
// x에 1을 더하여 새로운 객체를 반환
operator fun Point.inc(): Point {
    return Point(x + 1, y)
}

fun main(args: Array<String>) {
    var p = Point(10, 20)
    // 후위 증가 연산 수행
    println(p++)
    // 적용된 값이 출력
    println(p)
}

>>Point(x=10, y=20)
>>Point(x=11, y=20)
```

---

### 7.2 비교 연산자 오버로딩
자바의 equals나 compareTo를 == 비교 연산자를 직접사용 하여 비교 연산을 수행해보자.

#### 7.2.1 동등성 연산자 : equals
코틀린에서는 `==` 연산자 호출 시 `equals` 호출로 컴파일한다.

- `==`와 `!=`는 내부에서 인자가 널인지 검사
- 널이 될 수 있는 값에도 적용이 가능

**equals 정의**
```kotlin
// 직접 정의
override fun equals(obj: Any?): Boolean { // Any에 정의된 메서드 오버라이딩
    // 객체의 동일성 비교
    if (obj === this) return true
    // 파라미터 타입 검사
    if (obj !is Point) return false

    // 스마트 캐스트 적용
    return obj.x == x && obj.y == y
}
```

위 코드를 자바로 표현하면 다음과 같다.
```java
@Override
public boolean equals(Object obj) {
    // 동일 객체 비교
    if (this == obj) {
        return true;
    }

    // null 체크와 타입 검사
    if (!(obj instanceof Point)) {
        return false;
    }

    // 타입 변환 후 비교
    Point other = (Point) obj;
    return this.x == other.x && this.y == other.y;
}
```

- `==`를 이용해 객체의 동일성을 비교
- 널 여부와 타입 검사를 수행
- 스마트캐스트 적용이 불가능하기 때문에 타입 변환 수행

코틀린에서 `==`를 사용하여 equals를 대신할 때 주의사항은 다음과 같다.

- `===`을 이용해 수신 객체와 동일성 검사를 수행하며, `===`는 오버로딩이 불가능
- `Any`에서 상속 받은 `equals`가 확장 함수보다 우선순위가 높게 적용되어 확장 함수로 정의 불가능
- `Any`의 equals에는 `operator`가 붙어있지만 상속하는 클래스들에게는 필요 없음

#### 7.2.2 순서 연산자 : compareTo
자바에서는 `Comparable` 객체를 호출하거나 길게 메서드를 호출하여 한 객체와 다른 객체의 크기를 비교할 수 있다. 코틀린에서는 어떻게할까?

- `Comparable` 인터페이스를 지원
- compareTo 메서드 호출을 `<`, `>`, `<=`, `>=`와 같은 비교 연산자를 사용
- 반환하는 값은 Int

```kotlin
// Comparable 인터페이스 구현
class Human(val firstName: String, val lastName: String) : Comparable<Human> {
    override fun compareTo(other: Human): Int {
        return compareValuesBy(this, other, Human::lastName, Human::firstName)
    }
}

fun main(args: Array<String>) {
    val p1 = Human("Alice", "Smith")
    val p2 = Human("Bob", "Johnson")

    // Human 클래스의 first, last name을 각각 비교
    println(p1 < p2)
}
```

위에서 정의된 `Comparable` 인터페이스는 자바 쪽의 컬렉션 정렬 메서드 등에 사용이 가능하다.

**compareValuesBy()**
```kotlin
public fun <T> compareValuesBy(a: T, b: T, vararg selectors: (T) -> Comparable<*>?): Int {
    require(selectors.size > 0)
    return compareValuesByImpl(a, b, selectors)
}

// 실제 구현
private fun <T> compareValuesByImpl(a: T, b: T, selectors: Array<out (T) -> Comparable<*>?>): Int {
    for (fn in selectors) {
        val v1 = fn(a)
        val v2 = fn(b)
        val diff = compareValues(v1, v2)
        if (diff != 0) return diff
    }
    return 0
}
```

- 첫 번째, 두 번째 인자를 `compareValues()`로 전달해 `===` 연산자를 통해 동일성 체크
- 같지 않다면 즉시 결과를 반환하고, 같다면 `selectors`로 전달된 비교 함수를 통해 두 객체를 비교
- 각 비교 함수는 람다 또는 프로터티/메서드 일 수 있음

마지막으로 `Comparable` 인터페이스를 구현하는 모든 자바 클래스를 간결한 연산자 구문으로 비교가 가능하다.

---

### 7.3 컬렉션과 범위에 대해 쓸 수 있는 관례
컬렉션을 다룰 때 가장 많이 사용하는 다음 두 가지를 더욱 간단하게 하는 연산자 구문을 알아보자.

- 인덱스를 통한 원소 검색
- 컬렉션에 특정 원소 포함 여부

#### 7.3.1 인덱스로 원소에 접근 : get과 set
`somMap[key] = value` 처럼 인덱스 연산자를 사용해 원소를 읽고, 쓰는 방법에 대해 알아보자.

**원소를 읽는 연산 get**
```kotlin
// get, set 사용 인덱스 연산
operator fun Point.get (index: Int): Int {
    // 인덱스를 수신 객체로 하는 when 표현식
    return when(index) {
        0 -> x
        1 -> y
        else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
}

fun main(args: Array<String>) {
    val p = Point(10, 20)

    // get을 호출
    println(p[0])
    println(p[1])
}
```

**원소를 쓰는 연산 set**
```kotlin
// 변경 가능한 Point 클래스 정의
data class MutablePoint(var x: Int, var y: Int)

operator fun MutablePoint.set(index: Int, value: Int) {
    when (index) {
        0 -> x = value
        1 -> y = value
        else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
}

fun main(args: Array<String>) {
    val p = MutablePoint(10, 20)

    p[1] = 100
    println(p)
}
```

- map의 경우 key와 동일한 타입으로 연산자의 파라미터를 정의
- 2차원 행렬 또는 배열의 경우 `[row, col]` 형태로 메서드 호출 가능
- 다양한 파라미터 타입에 대해 오버로딩을 수행해야 할 수 있음

#### 7.3.2 in 관례
객체가 컬렉션에 포함되어 있는지 멤버십 검사(membership tet)를 한다. 이 때 in과 대응하는 함수는 `contains`다.

```kotlin
// in 사용 케이스
data class Rectangle(val upperLeft: Point, val lowerRight: Point)

operator fun Rectangle.contains(p: Point): Boolean {
    // in 연산자를 사용해 좌표가 사각형 안에 있는지 확인
    // until은 <=와 <를 사용해 범위를 정의
    return p.x in upperLeft.x until lowerRight.x &&
            p.y in upperLeft.y until lowerRight.y
}

fun main(args: Array<String>) {
    val rect = Rectangle(Point(10, 20), Point(50, 50))
    println(Point(20, 30) in rect)
    println(Point(5 ,5) in rect)
}

>> true
>> false
```

- 우항 객체는 `contains` 메서드의 수신 객체로 지정
- 좌항 객체는 `contains` 메서드의 인자로 전달
- until의 열린 범위는 끝 값을 포함하지 않는 범위를 의미(10~20일 경우 끝 값 20을 제외한 10~19)

`contains` 부분을 자바 코드로 표현하면 다음과 같다.

```java
// contains 메서드 구현
public boolean contains(Point p) {
    // until 대신 >=, < 연산자 사용
    // and 연산자를 x, y 각 두 번씩 사용
    return p.getX() >= upperLeft.getX() && p.getX() < lowerRight.getX()
            && p.getY() >= upperLeft.getY() && p.getY() < lowerRight.getY();
}
```

#### 7.3.3 rangeTo 관례
`..` 연산자를 통해 rangeTo 함수를 간략하게 표현하는 방법에 대해 알아보자.

- rangeTo는 아무 클래스에나 정의 가능
- 다만 `Comparable` 인터페이스를 구현한 클래스는 rangeTo 불필요
- 코틀린 표준 라이브러리에 모든 `Comparable` 객체에 대해 적용 가능한 rangeTo 함수가 들어있음

```kotlin
fun main(args: Array<String>) {
    val now = LocalDate.now() // 현재 날짜
    val vacation = now..now.plusDays(10) // 현재로부터 10일 후
    
    // 현재 날짜 기준으로 다음주에 휴가 기간이 표함되어 있는지 확인
    println(now.plusWeeks(1) in vacation)
}
```

- `now..now.plusDays(10)`은 `now.rangeTo(now.plusDays(10))`으로 컴파일러에 의해 변환
- `Comparable`에 대한 확장 함수로서 rangeTo()가 동작
- rangeTo 연산자는 다른 산술 연산자보다 우선순위가 낮음
- `()`를 이용해 인자를 감싸주면 명시적으로 표현되어 가독성 향상

#### 7.3.4 for 루프를 위한 iterator 관례
범위 검사와 똑같이 in 연산자를 사용하는 for 루프는 어떻게 관례를 적용하는지 알아보자.

- iterator 메서드를 확장 함수로 정의한다.
- 클래스 안에 직접 iterator 메서드를 구현하거나 표준 라이브러리에 확장 함수로 정의되어져 있다.

```kotlin
// iterator 정의
operator fun ClosedRange<LocalDate>.iterator(): Iterator<LocalDate> =
    // 날짜 범위를 반복하는 이터레이터 정의
    object : Iterator<LocalDate> {
        var current = start

        // compareTo 관례를 이용
        override fun hasNext() = current <= endInclusive

        // 현재 날짜를 먼저 반환하고, 다음 날짜로 이동
        // 해당 부분은 클로저를 통해 next 함수 외부에 정의된 변수 상태를 지속적으로 참조
        override fun next() = current.apply {
            current = plusDays(1)
        }
    }

fun main(args: Array<String>) {
    val newYear = LocalDate.ofYearDay(2020, 1)
    val daysOff = newYear.minusDays(1)..newYear

    for(dayoff in daysOff) {
        println(dayoff)
    }
}
```

- 범위 객체에 전달된 타입에 대한 이터레이터를 반환하는 확장 함수 정의
- LocalDate 객체가 반복되면서 수행될 작업을 내부에 정의
- 호출 시 범위를 지정하고, for 루프를 통해 해당 동작 수행

**컬렉션과 범위 객체**
|특성|컬렉션|범위 객체|
|---|---|---|
|in 연산자 동작|contains|contains|
|for 루프|iterator|iterator|
|범위 지원 여부|x|o|
|타입|List, Set, Map 등|IntRange, ClosedRange 등|

위 표에서 볼 수 있듯이 컬렉션과 범위 객체에서의 in 사용 관례 부분은 유사한 부분이 많다.
그렇다면 멤버십 검사와 for 루프 관점에서는 어떻게 적용되는지 알아보자.

|기능|멤버십 검사|for 루프 반복|
|---|---|---|
|문맥|if (element in collection)|for (element in collection)|
|메서드|fun contains(element:T)|fun iterator()|
|결과|Boolean|Iterator|
|용도|특정 요소가 컬렉션에 존재하는지 여부|컬렉션 요소를 순회|


---

### 7.4 구조 분해 선언과 component 함수
복합적인 값을 분해해서 여러 다른 변수를 한꺼번에 초기화 할 수 있는 구조 분해 선언에 대해 알아보자.

- 괄호를 이용해 여러 변수를 묶어서 선언한다.
- 구조 분해 선언의 각 변수는 `componentN` 이라는 함수를 통해 초기화한다.
- data class의 주 생성자에 들어있는 프로퍼티는 컴파일러가 자동으로 `componentN` 함수를 만들어준다.

구조 분해 선언은 함수가 반환하는 값을 쉽게 풀어서 여러 변수에 넣을 수 있다.

```kotlin
// 구조 분해 선언
data class NameComponents(val name: String, val extension: String)

fun splitFilename(fullName: String): NameComponents {
    // 파일명과 확장자를 분리
    val result = fullName.split('.', limit = 2)

    // 데이터 클래스 인스턴스 반환
    return NameComponents(result[0], result[1])
}

fun main(args: Array<String>) {
    val (name, ext) = splitFilename("example.kt")
    println("$name.$ext")
}
```

크기가 정해진 배열 또는 컬렉션을 이용할 경우 다음과 같이 사용 가능하다.

```kotlin
fun splitFilename(fullName: String): NameComponents {
    // 파일명과 확장자를 분리
    val (name, ext) = fullName.split('.', limit = 2)

    // 데이터 클래스 인스턴스 반환
    return NameComponents(name, ext)
}
```

- `componentN`은 맨 앞의 다섯 원소에 대하여 제공된다.
- 원소가 정의된 인덱스와 호출하는 인덱스가 다른 경우 예외가 발생한다.

```kotlin
fun main(args: Array<String>) {
    val (one, two, three, four, five) = listOf(1, 2, 3)
    println(four)
}

>>Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: Index 3 out of bounds for length 3
```

#### 7.4.1 구조 분해 선언과 루프
변수 선언이 들어갈 수 있는 장소에는 어디든 구조 분해 선언을 사용할 수 있다.

```kotlin
// map 구조 분해
fun printEntries(map:Map<String, String>) {
    // key와 value를 구조 분해와 객체 이터레이션
    for ((key, value) in map) {
        println("$key -> $value")
    }
}

fun main(args: Array<String>) {
    val map = mapOf("oracle" to "java", "jetbrains" to "kotlin")
    printEntries(map)
}
```

- 코틀린에서는 Map에 대한 직접적인 for 루프가 가능하다.
- Map.Entry에 대한 확장 함수로 `component1`과 `component1`를 제공한다.

---

### 7.5 프로퍼티 접근자 로직 재활용 : 위임 프로퍼티

- 값을 뒷받침하는 필드에 단순히 저장하는 것보다 더 복잡한 방식으로 작동하는 프로퍼티를 쉽게 구현 가능하다.
- 접근자 로직을 매번 재구현할 필요가 없다.
- 위임 객체(delegate)를 이용해 작업을 처리하게 맡긴다.

#### 7.5.1 위임 프로퍼티 소개
**기본 문법**
```kotlin
class Foo {
    var p: Type by Delegate()
}
```

- p의 접근자 로직을 다른 객체에게 위임
- by 뒤에 있는 식을 계산하여 위임에 쓰일 객체를 얻음

```kotlin
class Foo {
    // 컴파일러가 생성한 도우미 프로퍼티
    private val delegate = Delegate()
    var p: Type

    // 접근자들
    set(value: Type) = delegate.setValue(..., value)
    get() = delegate.getValue(...)
}
```

- 접근자는 프로퍼티 P와 연관된 동작을 정의
- Type은 프로퍼티 P의 데이터 타입으로, get 접근자의 반환 타입과 set 접근자의 매개변수 타입을 결정

```kotlin
class Delegate {
    // 백킹 필드 정의
    private var backingField: String = "default"

    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return backingField
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        backingField = value
    }
}

class Foo {
    private val delegate = Delegate()
    var p: String
        // 접근자
        get() = delegate.getValue(this, ::p)
        set(value) = delegate.setValue(this, ::p, value)
}

fun main(args: Array<String>) {
    val foo = Foo()
    println(foo.p)
    // set
    foo.p = "new value"
    println(foo.p)
}
```

- `thisRef` : 위임 프로퍼티를 소유한 객체를 나타내며(여기서는 Foo), null 포함 가능 타입으로 정의되어 다양한 컨텍스트에서 사용
- `property` : 위임 프로퍼티에 대한 메타데이터 제공하며, 여기서 사용된 `KProperty`는 리플렉션 기능을 지원

간소화하여 사용하면 아래와 같이 사용 가능하다.
```kotlin
class Foo {
    // 위임 프로퍼티의 접근자를 사용 가능
    var p: String by Delegate()
}
```

#### 7.5.2 위임 프로퍼티 사용 : by lazy()를 사용한 프로퍼티 초기화 지연

- 지연 초기화는 객체의 일부분이 실제로 필요할 때 초기화하는 패턴이다.
- 초기화 과정에 자원을 많이 사용하거나 반드시 초기화하지 않아도 되는 프로퍼티에 대해 적용 가능하다.

```kotlin
class Email{}

fun loadEmails(person: Ch7Person): List<Email> {
    println("Load email for ${person.name}")
    return listOf()
}

class Ch7Person (val name: String) {
    // 이메일 리스트 : 초기값 null
    private var _emails: List<Email>? = null
    val emails: List<Email>
        get() {
            // 이메일 리스트가 null이면 로드
            if (_emails == null) {
                _emails = loadEmails(this)
            }
            // 이메일 리스트 반환
            return _emails!!
        }
}

fun main(args: Array<String>) {
    val p = Ch7Person("Alice")
    // _emails == null 이기 때문에 로드
    p.emails
    // 데이터가 있으므로 반환
    p.emails
}
```

- 백킹 프로퍼티(_emails) 기법을 사용
- 널이 될 수 있는 타입과 널이 될 수 없는 타입으로 총 두개의 프로퍼티 사용
- `by lazy {}`를 사용해 간소화 가능


**lazy 함수**
```kotlin
class Ch7Person (val name: String) {
    val emails by lazy {loadEmails(this)}
}
```

- getValue() 메서드가 들어있는 객체를 반환
- lazy와 by를 함께 사용해 위임 프로퍼티 생성
- 기본적으로 스레드 안전하게 동작
- 다중 스레드 환경에서 사용하지 않을 프로퍼티를 위해 막을 수 있음

#### 7.5.3 위임 프로퍼티 구현
어떤 객체의 프로퍼티가 바뀔 때마다 리스너에게 변경 통지를 보내고 싶은 경우 등에 대한 기능을 구현하고 위임 프로퍼티를 사용해 리팩토링을 해보자.

- PropertyChangeSupport 인스턴스를 저장 할 changeSupport 필드를 보유하는 도우미 클래스를 생성

```kotlin
open class PropertyChangeAware {
    // PropertyChangeSuppeort 인스턴스 생성
    protected val changeSupport = PropertyChangeSupport(this)

    fun addPropertyChangeListener(listener: PropertyChangeListener) {
        changeSupport.addPropertyChangeListener(listener) {
            changeSupport.addPropertyChangeListener(listener)
        }
    }

    fun removePropertyChangeListener(listener: PropertyChangeListener) {
        changeSupport.removePropertyChangeListener(listener)
    }
}

class PersonByPropertyTest(val name: String, age: Int, salary: Int) : PropertyChangeAware() {
    var age: Int = age
        set(newValue) {
            // field는 프로퍼티의 백킹 필드를 참조
            val oldValue = field
            field = newValue
            // 변경사항을 리스너에게 통지
            changeSupport.firePropertyChange("age", oldValue, newValue)
        }

    var salary: Int = salary
        set(newValue) {
            val oldValue = field
            field = newValue
            changeSupport.firePropertyChange("salary", oldValue, newValue)
        }
}

fun main(args: Array<String>) {
    val p = PersonByPropertyTest("Dmitry", 33, 2000)
    p.addPropertyChangeListener(
            // 변경 리스너를 인자로 전달
            PropertyChangeListener { event ->
                println("Property ${event.propertyName} changed " +
                        "from ${event.oldValue} to ${event.newValue}")
            }
    )

    p.age = 40
    p.salary = 3000
}
```
위 코드의 경우 세터 코드에서의 중복이 발생한다. 이 부분에 대하여 리팩토링을 수행해보자.

```kotlin
class ObservableProperty(
        var propName: String, var propValue: Int,
        val changeSupport: PropertyChangeSupport
) {
    fun getValue(): Int = propValue
    // 세터를 호출하여 사용할 수 있도록 중복 로직을 정의
    fun setValue(newValue: Int) {
        val oldValue = propValue
        propValue = newValue
        changeSupport.firePropertyChange(propName, oldValue, newValue)
    }
}

class PersonByPropertyTest(val name: String, age: Int, salary: Int) : PropertyChangeAware() {
    var _age = ObservableProperty("age", age, changeSupport)
    var age: Int
        get() = _age.getValue()
        set(value) {
            _age.setValue(value)
        }

    val _salary = ObservableProperty("salary", salary, changeSupport)
    var salary: Int
        get() = _salary.getValue()
        set(value) {
            _salary.setValue(value)
        }
}
```

- `addPropertyChangeListener()`를 별도로 호출할 필요가 없어짐
- `changeSupport.firePropertyChange()`를 각 프로퍼티의 세터에서 호출할 필요가 없어짐
- 다만, `ObservableProperty`의 게터와 세터는 `Int` 타입만 다룰 수 있는 상태로 제네릭을 추가하여 타입을 유연하게 확장하도록 처리 필요

```kotlin
// 제네릭으로 호출 시 타입을 지정하면 T로 해당 타입 파라미터가 전달
class ObservableProperty<T> (...){
    fun getValue():T = propValue
    fun setValue(newValue: T) {
        ...
    }
]
```

위의 두 가지의 코드에서 최종적으로 프로퍼티 위임을 사용하여 관례에 맞게 수정한 결과를 보자.

```kotlin
class ObservableProperty(
        var propValue: Int, val changeSupport: PropertyChangeSupport
) {
    operator fun getValue(p: PersonByPropertyTest, prop: KProperty<*>): Int = propValue
    operator fun setValue(p: PersonByPropertyTest, prop: KProperty<*>, newValue: Int) {
        val oldValue = propValue
        propValue = newValue
        changeSupport.firePropertyChange(prop.name, oldValue, newValue)
    }
}

class PersonByPropertyTest(val name: String, age: Int, salary: Int) : PropertyChangeAware() {
    // 생성자 매개변수로 전달된 age, salary의 정보를 내부적으로 획득
    var age: Int by ObservableProperty(age, changeSupport)
    var salary: Int by ObservableProperty(salary, changeSupport)
}
```

- `operator` 변경자를 붙여서 정의
- 프로퍼티를 표헌하는 객체 `PersonByPropertyTest`와 프로퍼티를 표현하는 객체 `KProperty`를 파라피터로 전달
- 리플렉션을 통해 `.name`을 이용하여 프로퍼티 이름 획득
- `.name`을 통해 프로퍼티 이름 획득이 가능하므로 주 생성자에서 name을 제거

코틀린은 위임 객체를 감춰진 프로퍼티에 저장하고, 주 객체의 프로퍼티를 읽거나 쓸 때마다 위임 객체의 `getValue`와 `setValue`를 호출해준다.

마지막으로 by의 오른쪽에 있는 식이 반드시 새 인스턴스일 필요는 없다.

- 함수 호출
- 다른 프로퍼티
- 다른 식 등

다만, 우항의 식을 계산한 결과가 컴파일러가 호출할 수 있는 올바른 타입을 제공해야만한다.

#### 7.5.4 위임 프로퍼티 컴파일 규칙
```kotlin
var prop: Type by MyDelegate()
```

- 위임 객체의 인스턴스는 감춰진 프로퍼티에 저장하고, `delegate`라고 부름
- 프로퍼티 표현을 위해 `KProperty` 타입의 객체를 사용하고, `property`라고 부름
