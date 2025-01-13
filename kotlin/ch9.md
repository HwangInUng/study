## 9. 제네릭스
>데이터의 타입을 코드 작성 시에 고정하지 않고, 실행 시에 구체적인 타입으로 지정할 수 있도록 하는 프로그래밍 기법인 제네릭에 대해 알아보자.

- 실체화한 타입 파라미터 사용 시 인라인 함수 호출에서 구체적인 타입을 실행 시점에 확인 가능
- 일반 클래스 또는 함수는 타입 인자 정보가 실행 시점에 사라짐

**선언 지점 변성**

기저 타입은 같지만 타입 인자가 다른 경우 상/하위 관계에 대한 파악이 가능
```kotlin
Type<A>
Type<B>
```

여기서 기저 타입은 `Type`이고, A와 B는 각각 다른 타입 인자이다.
이런 경우를 `List`로 대입하여 알아보면 다음과 같다.

```kotlin
List<Any> // 상위
List<Int> // 하위
```
`Any` 타입을 인자로 받는 `List`에 `Int` 타입을 전달 할 수 있는지 여부를 선언 지점 변성을 통해 확인 가능하다.

**사용 지점 변성**

제네릭 타입을 선언할 때가 아니라 사용하는 지점에 지정할 수 있는 기능이다.

- out : 공변성을 의미하며 **읽기 전용**으로 사용한다. / Producer
- in : 반공변성을 의미하며 **쓰기 전용**으로 사용한다. / Consumer

```kotlin
class Box<T>(var value: T) // var value = 변경가능하다.

fun copyFrom(source: Box<out Any>, destination: Box<in Any>) {
    val value = source.value // out 키워드를 통해 읽기 가능
    destination.value = value // in 키워드를 통해 쓰기 가능
    println("Copied value: $value to destination")
}

fun main() {
    val source = Box("Hello")
    val destination = Box<Any>("Initial")
    copyFrom(source, destination) // "Copied value: Hello to destination"
    println(destination.value)    // "Hello"
}
```

선언 지점 변성과 사용 지점 변성에 대해서는 뒤에서 자세히 다루도록 하자.

### 9.1 제네릭 타입 파라미터
제네릭스를 사용해 타입 파라미터를 받는 타입을 정의하는 방법을 알아보자.

- 제네릭 타입의 인스턴스 생성 시 구체적인 타입 인자로 치환
- 타입 인자도 추론 가능

```kotlin
// 타입 인자 전달
List<String>
Map<String, SomeClass>

// 타입 추론
val stringList = listOf("String1", "String2")
```

타입 추론이 불가능한 빈 리스트는 타입 인자를 전달해야한다.

```kotlin
val emptystringList: ArrayList<String> = arrayListOf()
val emptyStringList = ArrayList<String>()
```

- 자바는 1.5 버전부터 제네릭이 도입되었기 때문에 `ArrayList<String> list = new ArrayList();` 등의 `(로(raw) 타입)`을 허용
- 코틀린은 애초에 제네릭이 지원되어 `(로(raw) 타입)`을 허용하지 않음

#### 9.1.1 제네릭 함수와 프로퍼티
제네릭 함수 호출 시 반드시 구체적 타입을 인자로 전달해야 한다.

```kotlin
fun <T> List<T>.slice(indices: IntRange): List<T>
```

- 수신 객체와 반환 타입에 `T`가 사용
- 컴파일러가 대부분 타입 인자 추론이 가능

```kotlin
val letters = {'a'..'z'}.toList()
println(letters.slice<Char>(0..2)) // 명시적 선언
println(letters.slice(10..13) // 타입 추론

>> [a,b,c]
>> [k,l,m,n]
```

- 람다에서 사용되는 `it`의 타입은 `T`라는 제네릭 타입
- `T`는 함수 타입 `(T) -> Boolean` 타입의 파라미터 타입

```kotlin
fun <T> List<T>.filter(predicate: (T) -> Boolean): List<T>
val authors = listOf("Dmitry", "Svetlana") // T = String, 타입 추론

authors.filter {it.name == "Dmitry"}
```

- 클래스나 인터페이스 안에 정의된 메서드, 확장 함수 및 최상위 함수에서 타입 파라미터 선언 가능
- 제네릭 확장 프로퍼티의 선언도 가능

```kotlin
// 모든 List 타입에 확장 프로퍼티 사용 가능
val <T> List<T>.penultimate:T
  get() = this[size - 2] // size - 2를 한 인덱스의 값을 get

println(listOf(1,2,3).penultimate) // Int로 타입 추론

>> 3
```

- 일반 프로퍼티는 타입 파라미터를 가질 수 없음
- `Type parameter of a property must be used in its receiver type` 컴파일 에러 발생

#### 9.1.2 제네릭 클래스 선언

- 꺽쇠 기호(<>)를 클래스 이름 뒤에 붙이면 클래스를 제네릭하게 만들 수 있다.
- 클래스 본문 안에서 타입 파라미터를 다른 일반 타입처럼 사용 가능하다.

```kotlin
interface List<T> { // 타입 파라미터 정의
  operator fun get(index: Int): T // 타입 호출
  //...
}
```

- 구체적인 타입 인자 또는 하위 클래스의 타입 파라미터를 전달 가능
- `T`는 다른 이름을 사용해도 무방

```kotlin
// Number 타입을 인자로 전달하는 제네릭 인터페이스
interface Operator<Number> {
    fun getValue(): Number
}
// 하위 클래스를 전달
class IntOperator(private val value: Number) : Operator<Int> {
    override fun getValue(): Int {
        return value.toInt();
    }
}
class DoubleOperator(private val value: Number) : Operator<Double> {
    override fun getValue(): Double {
        return value.toDouble();
    }
}


fun main(args: Array<String>) {
    // Number 타입의 파라미터 전달
    val intOp = IntOperator(4.55)
    val doubleOp = DoubleOperator(4.55)

    println(intOp.getValue())
    println(doubleOp.getValue())
}

>> 4
>> 4.55
```

#### 9.1.3 타입 파라미터 제약
클래스나 함수에 사용할 수 있는 타입 인자를 제한하는 기능에 대해 알아보자.

- `콜론(:)`을 표시하여 상한 타입을 작성
- 자바에서의 `<T extends Number>`와 유사

```kotlin
fun <T : Number> List<T>.sum(): T {...}
```

- 타입 상한을 정하면 `T` 타입의 값을 그 상한 타입의 값으로 취급

```kotlin
fun <T : Number> oneHalf(value: T): Double {
  return value.toDouble() / 2.0
}
```

- 두 개 이상의 파리미터 타입을 전달 할 때 서로 비교할 수 없는 값을 전달하는 경우 컴파일 오류가 발생

```kotlin
fun <T: Comparable<T>> max(first: T, second: T): T {
  return if (first > second) first else second
}

println(max("kotlin", "java")) // 비교 가능
println(max("kotlin", 42)) // 비교 불가

>> kotlin
>> ERROR : inferred type Any is not a subtype of Comparable<Any>?
```

- `T`의 상한 타입은 `Comparable<T>`
- String이 먼저 입력된 경우 뒤의 파라미터도 동일한 String으로 지정해야 정상 동작 수행

**타입 파라미터 복수 제약 설정**
타입 상한 선을 여러 타입으로 지정해야하는 경우 `where` 등을 사용할 수 있다.

```kotlin
fun <T> ensureTrailingPeriod(seq: T)
  where T : CharSequence, T : Appendable {
  if (!seq.endsWith('.')) {
    seq.append('.')
  }
}
```

- `where`을 활용해 타입 파라미터 제약 목록 정의
- 데이터 접근 연산(endsWith)과 데이터 변환 연산(append)을 `T` 타입의 값에게 수행 가능하다는 의미

#### 9.1.4 타입 파라미터를 널이 될 수 없는 타입으로 한정

- 아무런 상한이 없는 타입 파라미터 : `Any?`와 같음
- 널이 될 수 있는 타입을 포함하는 타입 인자를 지정해도 치환 가능

```kotlin
// 널을 허용하는 타입 파라미터
class Processor<T> { // Any?와 동일
  fun process(value: T) {
    value?.hashCode()
  }
}
```

- 항상 널이 될 수 없는 타입만 인자로 받으려면 `Any`를 사용
- 또는 `?`가 붙지 않은 타입으로 명시
- `?`가 붙은 타입을 지정하면 컴파일 에러가 발생

---
