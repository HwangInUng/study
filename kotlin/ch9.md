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

### 9.2 실행 시 제네릭스의 동작: 소거된 타입 파라미터와 실체화된 타입 파라미터
- JVM의 제네릭스는 보통 타입 소거를 사용해 구현
- 실행 시점에 제네릭 클래스 인스턴스에 타입 인자 정보가 들어있지 않다는 의미
- 코틀린에서는 `inline` 함수를 통해 이런 제약을 우회 가능

#### 9.2.1 실행 시점의 제네릭: 타입 검사와  캐스트
- 제네릭 타입 인자 정보는 런타임에 지워짐
- 제네릭 클래스 인스턴스의 타입 인자에 대한 정보를 유지하지 않음을 의미

```kotlin
val list1: List<String> = listOf("a", "b")
val list2: List<Int> = listOf(1, 2, 3)
```

- 컴파일러는 두 리스트를 서로 다른 리스트로 인식
- 실행 시점에는 완전히 같은 타입 객체로 인식
- 컴파일러가 타입을 보장하기 때문에 실행 시점에서 해당 타입의 데이터만 보유하는 것이 가능

```kotlin
if (value is List<String>) {...}

>> ERROR: Cannot check for instance of erased type
```
- 타입 소거는 실행 시점에 타입 인자 검사가 불가능하다는 한계가 있다.
- 다만, 타입 정보의 크기가 줄어들어 메모리 사용량 감소라는 장점도 보유하고 있다.
- 자바의 `<?>`와 같은 스타 프로젝션(`*`)을 사용하여 타입 검사를 할 수 있다.

```kotlin
// 스타 프로젝션 사용
if (value is List<*>) {...}

// 파라미터가 2개 이상
if (value is Map<*, *>) {...}
```

- `as` 또는 `as?`를 사용한 캐스팅에도 제네릭 타입 사용이 가능하다.
- 기저 클래스가 같고, 타입 인자가 다르더라도 캐스팅에 성공한다는 점을 주의해야한다.
- 컴파일러가 `Unchecked cast: Collection<*> to List<Int>`와 같은 경고를 출력한다.

```kotlin
fun printSum(c: Collection<*>) {
    // 기저 클래스 : Collection
    // 타입 인자는 정의하기 나름
    val intList = c as? List<Int> ?: throw IllegalArgumentException("List is expected")

    println(intList.sum())
}

fun main(args: Array<String>) {
    printSum(listOf(1, 2, 3)) // 타입 추론을 통한 정상 동작
}

>>> 6
```

- 잘못된 타입의 원소가 들어있는 리스트를 전달하면 `ClassCastException`이 발생한다.
- 해당 예외가 발생하는 시점은 `intList.sum()`을 실행하는 시점이다.
- 컴파일러가 안전하지 못한 검사와 수행할 수 있는 검사를 알려주기 때문에 힌트를 잘 참고해야한다.

#### 9.2.2 실체화한 타입 파라미터를 사용한 함수 선언
제네릭 함수 에서도 타입 인자 정보를 실행 시점에 알아낼 수는 없다.

```kotlin
fun <T> isSome(value: Any) = value is T
```

- `Error: Cannot check for instance of erased type: T` 예외 발생
- 인라인 함수의 타입 파라미터는 실체화되기 때문에 실행 시점에서 확인 가능
- 컴파일러가 식을 함수 본문으로 바꾸기 때문

```kotlin
inline fun <reified T> isA(value: Any) = value is T

fun main(args: Array<String>) {
    println(isA<String>("abc"))
    println(isA<String>(12))
}

>> true
>> false
```

- `reified` 키워드를 사용하여 타입 파라미터 지정
- 함수 호출 시점에 제네릭 타입을 구체적인 타입으로 치환

```kotlin
inline fun <reified T> isA(value: Any): Boolean {
    return value is T
}

// 호출 시점에서 구체화
println(isA<String>("Hello"))
// 컴파일 후:
println("Hello" is String)
```

- `filterIsInstance<T>()`를 사용하면 실체화한 타입 파라미터를 활용하여 해당 타입의 원소만 반환
- 자바에서는 `reified` 타입 파라미터를 사용하는 inline 함수 호출이 불가능

#### 9.2.3 실체화한 타입 파라미터로 클래스 참조 대신
실체화한 타입 파라미터를 사용해 API 쉽게 호출하는 방법을 알아보자.

```kotlin
// 일반 호출
val serviceImpl = ServiceLoader.load(Service::class.java)
// 실체화한 타입 파라미터 사용
val serviceImpl = loadErvice<Service>()

// 함수 정의
inline fun <reified T> loadService() {
    // 타입 파라미터의 클래스 호출
    return ServiceLoader.load(T::class.java)
}
```

#### 9.2.4 실체화한 타입 파라미터의 제약
실체화한 타입 파라미터 사용이 가능한 경우는 다음과 같다.

- 타입 검사와 캐스팅 : `is`, `!is`, `as`, `as?`
- 코틀린 리플렉션 API : `::class`
- 코틀린 타입에 대응하는 `java.lang.Class` 얻기 : `::class`
- 다른 함수를 호출할 때 타입 인자로 사용

제약 사항은 다음과 같다.
- 인스턴스 생성
- 동반 객체 메서드 호출
- 실체화한 타입 파라미터를 요구하는 함수에 실체화하지 않은 타입 파라미터를 넘기기
- 클래스, 프로퍼티, 인라인 함수가 아닌 함수의 타입 파라미터를 `reified`로 지정

기본적으로 실체화한 타입 파라미터는 인라인 함수에서만 사용이 가능하기 때문에 해당 함수에 전달되는 모든 람다와 함께 인라이닝이 된다.
이런 경우 `noinline` 변경자를 이용해 함수 타입 파라미터의 인라이닝을 금지할 수 있다.

---

### 9.3 변셩: 제네릭과 하위 타입
변성은 기저 타입은 같지만 타입 인자가 다른 여러 타입의 관계가 어떤지 설명하는 개념이다.

#### 9.3.1 변성이 있는 이유: 인자를 함수에 넘기기
읽기 전용과 변경 가능한 컬렉션에서 변성이 필요한 이유를 알아보자.

**읽기 전용**
```kotlin
fun printContents(list: List<Any>) {
    println(list.joinToString())
}

fun main(args: Array<String>) {
    printContents(listOf("abc", "bac"))
}

>> abc, bac
```
Any 타입의 하위 타입이 String이므로 완전히 안전하기 때문에 정상적으로 동작한다.
하지만 변경이 가능한 경우에는 어떤 문제가 발생할까?

**변경 가능**
```kotlin
fun addAnswer(list: MutableList<Any>) {
    list.add(42)
}

fun main(args: Array<String>) {
    val strings = mutableListOf("abc", "bac")
    addAnswer(strings) // 컴파일 에러
}
```
Any 타입의 하위 타입 중 무작위로 추가가 된다면 안정성을 보장할 수 없기 때문에 컴파일 에러가 발생한다.

#### 9.3.2 클래스, 타입, 하위 타입

- 클래스 : 타입을 정의하기 위한 틀이지만 클래스가 곧 타입이라고 할 수 없으며, 널을 포함하는 타입까지 고려하면 최소 2개 이상의 타입 구성 가능
- 타입 : 인터페이스, 클래스, 제네릭, 함수 등으로 정의될 수 있음
- 하위 타입 : 어떤 타입 A의 값이 필요한 모든 장소에 어떤 타입 B의 값을 넣어도 문제가 없으면 B는 A의 하위 타입으로 간주

```kotlin
fun test(i: Int) {
    val n: Number = i // Int는 Number의 하위 타입으로 컴파일 성공
    fun f (s: String) {...}
    f(i) // 컴파일 에러
}
```

- 컴파일러는 변수 대입, 인자 전달 시 하위 타입 검사를 매번 수행
- 변수 타입의 하위 타입인 경우에만 값을 변수에 대입하게 허용
- 간단한 경우 하위 타입은 하위 클래스와 근본적으로 같음
- 널이 될 수 있는 타입은 널이 될 수 없는 동일 클래스의 상위 타입

인스턴스 타입 사이의 하위 타입 관계가 성립하지 않으면 그 제네릭 타입을 무공변(invariant)라고 말하고,
반대로 하위 타입 관계가 성립하면 공변적(covariant)라고 말한다.
자바에서는 모든 클래스가 무공변이다.

#### 9.3.3 공변성: 하위 타입 관계를 유지
`SomeClass<A>`가 `SomeClass<B>`의 하위 타입이면 `SomeClass`는 공변적이다.

- 제네릭 클래스가 타입 파라미터에 대해 공변적일 경우 `<out T>`로 표시
- 타입 파라미터를 공변적으로 만들면 타입이 정확히 일치하지 않아도 함수 인자나 반환 값으로 사용 가능

```kotlin
// 공변적
class Herd<out T: Animal> {
    ...
}

fun feedAll (animals: Herd<Animal>) {
    for (i in 0 until animals.size) {
        animals[i].feed()
    }
}

fun takeCareOfCats (cats: Herd<Cat>) {
    for (i in 0 until cats.size) {
        cats[i].cleanLitter()
    }
    feedAll(cats) // 캐스팅 불필요
}
```

- 모든 클래스를 공변적으로 만들 수 없음
- 공변적으로 선언 시 파라미터 사용 방법을 제한
- 공변적 파라미터는 항상 아웃 위치에만 있어야 함
- 클래스가 T 타입의 값을 생산할 수 있지만 소비는 불가능

```kotlin
interface Transformer<T> {
    // 괄호 안의 T : in
    // 괄호 밖의 T : out
    fun transform(t: T): T
}
```

- in
  - 입력 위치에서만 사용
  - 쓰기 전용으로 값을 소비하는 역할 수행
  - 반환 타입으로 사용 불가
- out
  - 출력 위치에서만 사용
  - 읽기 전용으로 값을 제공하는 역할 수행
  - 타입 매개변수를 메서드의 입력으로 사용 불가

공변성은 상위 타입 관계를 유지해야 할 때 유용하게 사용되며, 하위 타입을 수용할 수 있도록 하기 위하여 사용한다.

```kotlin
// out (공변성)
interface List<out T>: Collection<T> {
    fun subList(fromIndex: Int, toIndex: Int): List<T>
}

// in (반공변성)
interface MutableList<T>: List<T>, MutableCollection<T> {
    override fun add(element: T) Boolean
}
```

in & out의 영향을 받지 않는 예외는 다음과 같다.
- 생성자는 인스턴스 생성 시 한 번만 호출되는 함수로 in & out의 영향을 받지 않는다.
- `private` 메서드는 클래스 내부 구현이기 때문에 영향을 받지 않는다.

#### 9.3.4 반공변성: 뒤집힌 하위 타입 관계
반공변 클래스의 하위 타입 관계는 공변 클래스의 경우와 반대이다.

```kotlin
interface Comparator<in T> {
    // T를 in 위치에 사용
    fun compare(e1: T, e2: T): Int {...}
}
```
- T 타입의 값을 소비
- in 위치에만 쓰여야함

```kotlin
fun main(args: Array<String>) {
    val anyComparator = Comparator<Any> {
        e1, e2 -> e1.hashCode() - e2.hashCode()
    }

    val strings: List<String> = listOf("a", "b")
    strings.sortedWith(anyComparator)
}
```

- Comparator<Any>가 Comparator<String> 보다 더 일반적인 타입을 비교 가능
- Comparator<Any>는 Comparator<String>에 전달되었을 때 String의 조상까지 비교가능
- 즉 반공변에서는 이런 상황에서 Comparator<Any>를 Comparator<Stirng>의 하위 타입이라 부름

```kotlin
Cat -> Animal

// 공변성
Producer<Cat> -> Producer<Animal>
// 반공변성
Consumer<Animal> -> Cumsumer<Cat>
```

in은 해당 클래스의 메서드 안으로 전달돼 메서드에 의해 소비된다는 의미이다.

특정 파타입 파라미터에 대해서는 공변적이면서 다른 타입 파라미터에서는 반공변적일 수도 있는 예시를 알아보자.
```kotlin
interface Finction1<in P, out R> {
    operator fun invoke(p:P):R
}
```

- Function1의 하위 타입 관계는 첫 번째 타입 인자의 하위 타입 관계와 정반대
- 두 번째 타입 인자의 하위 타입 관계와는 같음을 의미

```kotlin
fun enumerateCats(f: (Cat) -> Number) {...}
fun Animal.getIndex(): Int = ...

// Animal은 Cat의 상위 타입
// Int는 Number의 하위 타입
enumerateCats (Animal::getIndex)
```
- 클래스 정의에 변성을 직접 기술하면 그 클래스를 사용하는 모든 장소에 변성이 적용
- 자바는 이를 지원하지 않음

위 예시는 인자의 타입에 대해서는 반공변적이고, 반환 타입에 대해서는 공변적이다.
즉, in의 위치에는 정반대의 하위 타입 관계가 성립하고, out 위치에는 일반적인 하위 타입 관계가 성립한다.


