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

