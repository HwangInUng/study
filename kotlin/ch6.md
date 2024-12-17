## 6. 코틀린 타입 시스템

### 6.1 널 가능성
>NPE(NullPointException)을 피할 수 있게 돕기 위한 코틀린 타입 시스템의 특성

최신 언어에서 null에 대한 접근 방법은 가능한 한 이 문제를 실행 시점에서 컴파일 시점으로 옮기는 것에 집중한다.
널이 될 수 있는지 여부를 타입 시스템에 추가하여 컴파일러가 여러 가지 오류를 컴파일 시 미리 감지해서 실행 시점에 발생하는 예외 가능성을 줄일 수 있다.

#### 6.1.1 널이 될 수 있는 타입
>코틀린과 자바의 첫 번째이자 가장 중요한 차이는 코틀린이 널이 될 수 있는 타입을 명시적으로 지원한다는 점이다.

**null로부터 안전하지 않은 Java**
```java
int strLen(String s) {
  return s.length();
}

System.out.println(strLen(null));
>>NullPointException 발생
```
코틀린은 함수가 null을 인자로 받을 수 있는지 생각하고, 함수에 적용할 수 있다.
```kotlin
// null 예외
fun strLen(s: String) = s.length


fun main(args: Array<String>) {
    print(strLen(null))
}
```
위와 같이 작성하면 컴파일 단계에서 다음과 같은 예외와 도움말이 나온다.
-  Null can not be a value of a non-null type String
-  Change parameter 's' type of function 'strLen' to 'String?'

그렇다면 어떻게 해야하나?
```kotlin
String?, Int?, MyCustomType?
```
어떤 타입이든 이름 뒤에 물음표를 붙이면 null 참조를 저장할 수 있다는 의미이다.

```kotlin
// 예외 발생
fun strLenSafe(s: String?) = s.length
// 정상 동작
fun strLenSafe(s: String?) = s?.length


fun main(args: Array<String>) {
    // null Safe
    print(strLenSafe(null))
}
```
- `fun strLenSafe(s: String?) = s.length`는 컴파일 단계에서 에러가 발생한다.
- 이유는 null을 허용하는 변수에 대해 `변수.메서드()`와 같은 직접 호출은 불가능하기 때문이다.
- 따라서 `fun strLenSafe(s: String?) = s?.length`와 같이 호출도 null이 아닌 경우에만 진행하도록 수정한다.

**if 검사를 통해 null 값 다루기**
```kotlin
fun strLenByif(s: String?): Int =
    // null 검사를 추가하면 컴파일 가능
    if (s != null) s.length else 0

fun main(args: Array<String>) {
    val x: String? = null
    print(strLenByif(x))
}
```

#### 6.1.2 타입의 의미
타입은 분류이자 어떤 값들이 가능한지와 해당 타입에 대해 수행할 수 있는 연산의 종류를 컴파일러가 결정할 수 있게 하는 것이다.

**자바의 타입 시스템**
- String 타입은 String과 null 두 가지 값이 들어갈 수 있다.
- instanceof 연산자도 null이 String이 아니라고 판단한다.
- 하지만 자바는 추가로 검사를 하기 전에는 그 변수가 어떤 연산을 수행할 수 있는지 알 수 없다.
- null 체크를 지원하는 어노테이션이 있긴 하지만 표준 자바 컴파일 절차의 일부가 아니다.
- 즉, 일관성 있게 적용된다고 보장할 수 없다.

**코틀린의 타입 시스템**
- 모든 검사가 컴파일 시점에 수행된다.
- 널이 될 수 있는 타입을 처리하는 데 별도의 실행 시점 부가 비용이 들지 않는다.

#### 6.1.3 안전한 호출 연산자: ?.
null 검사와 메서드 호출을 한번의 연산으로 수행하는 연산자이다.

```kotlin
// ?. 연산자
s?.toUpperCase();

// 일반적인 검사
if( s != null) s.toUpperCase()
else null
```
- 안전한 호출의 결과 타입도 널이 될 수 있는 타입이라는 사실에 유의
- 즉, `s?.toUpperCase()`의 식의 결과가 `String?`일 수도 있다.

```kotlin
fun printAllCaps(s: String?) {
    val allCaps: String? = s?.toUpperCase() // String? 일수도
    println(allCaps)
}

fun main(args: Array<String>) {
    printAllCaps("abc")
    printAllCaps(null)
}
```

- 프로퍼티를 읽거나 쓸 떄도 안전한 호출 사용 가능
```kotlin
class Employee (val name:String, val manager: Employee?)

fun managerName(employee: Employee): String? = employee.manager?.name

fun main(args: Array<String>) {
    val ceo = Employee("Boss", null)
    val developer = Employee("Bob", ceo)

    println(managerName(developer))
    println(managerName(ceo))
}
```
- 객체 그래프의 중간 객체가 여럿 있는 경우 한 식 안에서 안전한 호출을 연쇄해서 사용 가능
- 별도의 추가적인 검사 없이 ?. 연산자를 통해 간결하게 작성 가능

```kotlin
class Address (val streetAddress: String, val zipCode: Int, val city: String, val country: String)
class Company (val name: String, val address: Address?)
class Customer (val name: String, val company: Company?)

// 확장함수
fun Customer.contryName (): String {
    // 연쇄적으로 확인
    val country = this.company?.address?.country
    return if (country != null) country else "Unkown"
}

fun main(args: Array<String>) {
    val customer = Customer("Dmitry", null)
    println(customer.contryName())
}
```

#### 6.1.4 엘비스 연산자: ?:
null 대신 사용할 디폴트 값을 지정할 때 편리하게 사용할 수 있는 연산자이다.

- ?: 모양이 시계방향으로 90도 눕혔을 때 엘비스를 닮았다고해서 이름이 붙여졌다.
- 널 복합 연산자라고도 부른다.

```kotiln
fun foo (s: String?) {
  val t: String = s ?: ""
}
```

- 이항 연산자로 좌항을 계산한 값이 널인지 검사
- 널이 아니면 좌항 값을 결과, 널이면 우항 값을 결과로 반환
- 객체가 널인 경우 널을 반환하는 안전한 호출 연산자와 함께 사용
- 객체가 널인 경우에 대비한 값을 지정

```kotlin
fun strLenSafe(s: String?) = s?.length ?: 0

fun main(args: Array<String>) {
    println(strLenSafe("abc"))
    println(strLenSafe(null))
}

// countryName 코드 간소화
fun Customer.contryName (): String = this.company?.address?.country ?: "Unkown"
```

- return, throw 등의 연산도 식으로 작동한다.
- 엘비스 연산자 우항에 return, throw를 넣을 수도 있다.
- 이런 패턴은 전제 조건을 검사하는 경우 특히 유용하다.

```kotlin
fun printShoppingLabel (customer: Customer) {
    val address = customer.company?.address
        ?: throw IllegalArgumentException("No Address")

    // 수신 객체 지정 람다 사용
    // 위에서 검사가 수행되었기 때문에 null이 아니라고 확신
    with (address) {
        println(streetAddress)
        println("$zipCode $city, $country")
    }
}

fun main(args: Array<String>) {
    val address = Address("Elsestr. 47", 90676, "Munich", "Germany")
    val jetbrains = Company("JetBrains", address)
    val customer = Customer("Dmitry", jetbrains)

    printShoppingLabel(customer)
    // exception 반환
    printShoppingLabel(Customer("ET", null))
}
```

#### 6.1.5 안전한 캐스트: as?
어떤 값을 지정한 타입으로 캐스트하는 연산자이다.

- 값을 대상 타입으로 변환할 수 없으면 null을 반환한다.
- 일반적인 패턴은 캐스트를 수행한 뒤 엘비스 연산자를 사용하는 것이다.

```kotlin
class Ch6Person(val firstName: String, val lastName: String) {
    override fun equals(other: Any?): Boolean {
        // 안전한 캐스트와 엘비스 연산자를 활용
        val otherPerson = other as? Ch6Person ?: return false

        return otherPerson.firstName == firstName && otherPerson.lastName == lastName
    }

    override fun hashCode(): Int = firstName.hashCode() * 37 + lastName.hashCode()
}

fun main(args: Array<String>) {
    val p1 = Ch6Person("Dmitry", "Jemerov")
    val p2 = Ch6Person("Dmitry", "Jemerov")

    // ==는 equals 호출
    println(p1 == p2)
    println(p1.equals(p2))
}
```

#### 6.1.6 널 아님 단언: !!
null이 될 수 있는 타입의 값을 다룰 때 사용할 수 있는 도구 중 가장 단순하면서도 무딘 도구이다.

- 어떤 값이든 널이 될 수 없는 타입으로 강제
- 실제 널에 !!를 적용하면 NPE 발생

```kotlin
fun ignoreNulls (s: String?) {
    // 널일 수 있는 값에 !! 적용
    val sNotNull: String = s!! // 예외는 여기를 가리킨다
    println(sNotNull.length)
}

fun main(args: Array<String>) {
    // null 전달
    ignoreNulls(null)
}

>>Exception in thread "main" java.lang.NullPointerException
```

- 예외가 발생한 지점은 null 값을 사용하는 코드가 아니라 단언문이 위치한 곳을 가리키는 점을 유의해야한다.
- !!이 쓰인 곳의 값이 null이 아님을 잘못알고 있다는 차원에서 예외를 해당 위치에서 가리키는 것 같다.
- !!를 사용해서 발생하는 예외의 스택 트레이스는 어떤 식에서 예외가 발생했는지에 대한 정보가 들어있지 않다는 점을 유의해야한다.

```kotlin
person.company!!.address!!.country // 이렇게 작성하면 어떤 부분에서 예외가 발생했는지 알기 어렵다
```

!!는 호출된 함수가 언제나 다른 함수에서 널이 아닌 값을 전달받는다는 사실이 분명하다면 굳이 널 검사를 할 필요 없이 널 아님 단언문을 사용하면된다.

#### 6.1.7 let 함수
null이 될 수 있는 식을 더 쉽게 다룰 수 있는 함수이다.

- 안전한 호출 연산자와 함께 사용하면 결과를 변수에 넣는 작업을 간단한 식으로 구현이 가능하다.
- 널이 될 수 있는 값을 널이 아닌 값만 인자로 받는 함수에 넘기는 경우 자주 사용한다.

```kotlin
// 예외 케이스
fun sendEmailTo(email: String) = email

fun main(args: Array<String>) {
    val email: String? = ""
    sendEmailTo(email)
}

>>Type mismatch.
  Required:
  String
  Found:
  String?

// 정상 처리
fun sendEmailTo(email: String) = email

fun main(args: Array<String>) {
    val email: String? = ""
    // null 체크
    if (email != null) sendEmailTo(email)
}
```

- let 함수는 자신의 수신 객체를 인자로 전달받은 람다에게 넘긴다.
- 안전한 호출 구문을 사용해 널이 될 수 있는 값에 대해 let을 호출하고,
- 널이 될 수 없는 타입을 인자로 받는 람다를 let에 전달한다.
- 널이 될 수 있는 타입의 값을 널이 될 수 없는 타입의 값으로 바꿔서 람다에 전달하게 된다.

```kotlin
fun sendEmailTo(email: String) = email

fun main(args: Array<String>) {
    val email: String? = null
    // 기본사용 문법
    email?.let { email -> sendEmailTo(email) }
    // it 사용
    email?.let { sendEmailTo(it) }
}

// 조금 더 긴 패턴
fun sendEmailTo(email: String) {
    println("Sending email to $email")
}

fun main(args: Array<String>) {
    var email: String? = "yole@example.com"
    email?.let { sendEmailTo(it) }

    email = null
    email?.let { sendEmailTo(it) }
}
```

- 아주 긴 식이 있고 그 값이 널이 아닐 때 수행해야 하는 로직이 있을 때 쓰면 더 편하다.
- 긴 식의 결과를 저장하는 변수를 따로 만들 필요가 없다.

```kotlin
// 긴 식의 결과를 담을 변수 선언
val person: Person? = getTheBestPersonInTheWorld()
// 변수 null 여부 확인
if (person != null) sendEmailTo(person.email)

// let 사용 간소화
getTheBestPersonInTheWorld().let { sendEmailTo{it.email} }
```

복수 개의 값이 널인지 검사해야하는 경우 다음과 같이 처리 가능하지만 중첩되기 때문에 가독성이 떨어진다.
이런 경우는 if를 사용해 모든 값을 한꺼번에 검사하는 편이 나을 수도 있다.

```kotlin
// 중첩 let
val name: String? = "Alice"
val age: Int? = 30
val city: String? = null

name?.let { nonNullName ->
    age?.let { nonNullAge ->
        city?.let { nonNullCity ->
            // name, age, city 모두 널이 아닐 때만 이 블록이 실행됨
            println("Name: $nonNullName, Age: $nonNullAge, City: $nonNullCity")
        }
    }
}

// if 사용
val info = if (name != null && age != null && city != null) {
    "Name: $name, Age: $age, City: $city"
} else {
    null
}

info?.let(::println)
```

#### 6.1.8 나중에 초기화할 프로퍼티
코틀린에서는 클래스 안의 널이 될 수 없는 프로퍼티는 생성자 안에서 초기화해야한다.
또한, 프로퍼티 타입이 널이 될 수 없는 타입이라면 반드시 널이 아닌 값으로 초기화해야한다.

이런 문제를 해결해 주는 것이 lateinit 변경자이다.

```kotlin
class MyTest {
    // lateinit을 사용할 때 var로 프로퍼티 선언
    private lateinit var myService: MyService
    fun setUp() {
        myService = MyService()
    }

    fun testAction():String {
        return myService.performAction()
    }
}

// 예외 케이스
// lateinit property myService has not been initialized 예외 발생
fun main(args: Array<String>) {
    val test = MyTest()

    println(test.testAction())
}

// 정상 실행
fun main(args: Array<String>) {
    val test = MyTest()
    test.setUp()
    println(test.testAction())
}

```

- lateinit 프로퍼티는 DI 프레임워크와 사용하는 경우가 많다.
- lateinit 프로퍼티의 값을 DI 프레임워크가 외부에서 설정해준다.
- 자바 프레임워크와의 호환성을 위해 lateinit가 지정된 프로퍼티와 가시성이 똑같은 필드를 생성한다.

#### 6.1.9 널이 될 수 있는 타입 확장
확장 함수를 정희하면 null 값을 다루는 강력한 도구로 활용할 수 있다.
직접 변수에 대해 메서드를 호출해도 확장 함수인 메서드가 알아서 널을 처리해주며, 이런 처리는 확장 함수에서만 가능하다.

```kotlin
fun verifyUserInput (input: String?) {
    // null 검사를 수행하지 않아도 함수 내부적으로 수행
    if(input.isNullOrBlank()) {
        println("Please fill in the required fields")
    }
}

fun main(args: Array<String>) {
    verifyUserInput(" ")
    verifyUserInput(null)
}
```
isNullOrBlank() 함수의 코드를 들여다 보면 return 시 null 체크를 수행한다.

```kotlin
public inline fun CharSequence?.isNullOrBlank(): Boolean {
    contract {
        returns(false) implies (this@isNullOrBlank != null)
    }

    // null 또는 isBlank()인 경우를 확인하여 결과를 반환
    return this == null || this.isBlank()
}
```

- 자바에서는 메서드 안의 `this`가 메서드가 호출된 수신 객체를 가리키므로 항상 널이 아니다.
- 코틀린에서는 널이 될 수 있는 타입의 확장 함수 안에서는 `this`가 널이 될 수 있다는 것이 자바와 다르다.
- let 사용 시에는 반드시 안전한 호출 연산인 ?. 연산자를 사용해야한다.
- 또한, 처음에 확장 함수를 작성할 때 널이 될 수 없는 타입으로 작성하고, 널 가능성이 생기면 그때 체크하는 로직을 추가하자.

#### 6.1.10 타입 파라미터의 널 가능성
코틀린의 함수나 클래스의 모든 타입 파라미터는 기본적으로 널이 될 수 있다.

```kotlin
fun <T> printHashCode (t: T) {
    // t가 널일 가능성을 내포하고 있음
    println(t?.hashCode())
}

fun main(args: Array<String>) {
    printHashCode(null)
}
```

- 타입 파라미터 T에 대해 추론한 타입은 널이 될 수 있는 Any? 타입이다.
- 타입 파라미터는 타입 상한을 통해 널이 될 수 없음을 확실히 할 수 있다.

```kotlin
fun <T: Any> printHashCodeNotNull(t: T) {
    println(t.hashCode())
}

fun main(args: Array<String>) {
    // Null can not be a value of a non-null type TypeVariable(T) 컴파일 에러
    printHashCodeNotNull(null)
    // 정상 동작
    printHashCodeNotNull(42)
}
```
타입 파라미터는 널을 허용하는 타입을 표시할 때 물음표를 타입 이름 뒤에 붙여야 하는 **규칙의 유일한 예외**다.

#### 6.1.11 널 가능성과 자바
자바 타입 시스템은 널 가능성을 지원하지 않는다. 이런 문제를 어떻게 해결하는지 알아보자.
- 애너테이션(@NotNull 등)을 이용하여 널 가능성 정보를 표시한다.
- 코틀린은 널 가능성 애너테이션을 인식한다.
  - JSR-305 표준
  - 안드로이드
  - 젯브레인스 도구 애너테이션 등
- 널 가능성 애너테이션이 없는 경우 플랫폼 타입이 된다.

플랫폼 타입은 코틀린이 널 관련 정보를 알 수 없는 타입을 의미한다.
즉, 사용자에 의해 플랫폼 타입에 수행되는 모든 연산에 대한 책임이 주어진다.

```java
public class Person {
  // 널 가능성을 알지 못함
  private final String name;

  public Person (String name) {
    this.name = name;
  }
  // name이 널인지 아닌지 확인 할 수 없다.
  public String getName() {
    return name;
  }
}
```
위 자바 코드에서는 getName()을 호출한 시점에 name 프로퍼티가 널인지 아닌지 에측하기가 어렵다. 그래서 다음과 같이 검사를 해야한다.

```java
...생략
public String getName() {
  if(name != null) return name;
  return "Unkown";
}
```

- 코틀린 컴파일러는 공개(public) 가시성인 코틀린 함수의 널이 아닌 타입인 파라미터와 수신 객체에 대한 널 검사를 추가해준다.
- 즉, 공개 가시성 함수에 널 값을 사용하면 즉시 예외가 발생한다.

```kotlin
// 여기서 Person은 자바 코드로 작성된 클래스이다.
fun yellAtSafe (person: Person) {
    // 널 값을 처리하여 실행 시점에 예외가 발생하지 않음
    println((person.name ?: "Anyone").uppercase() + "!!!")
}

fun main(args: Array<String>) {
    yellAtSafe(Person(null))
}
```

- 자바 API의 대부분 라이브러리는 널 관련 애너테이션을 사용하지 않기 때문에 조심해야한다.
- 코틀린이 플랫폼 타입을 도입한 것은 널 안정성으로 얻는 이익보다 검사에 드는 비용이 훨씬 더 큰 점을 고려하여 프로그래머에게 제대로 처리할 책임을 부여하는 방법을 택한 것이다.
- 또한, 코틀린에서는 플랫폼 타입을 선언할 수 없다.

```kotlin
fun main(args: Array<String>) {
    val person = Ch6Person("first", "last")
    val i: Int = person.firstName // Type mismatch. Required: Int Found: String
}
```
책의 예제와는 다른 예외가 발생한다.

추가로 코틀린에서 자바 메서드를 오버라이드할 때 메서드의 파라미터의 널 여부를 결정해야 한다.

```java
interface StringProcessor {
  void process(String value);
}
```
위와 같이 String 타입을 파라미터로 받는 추상 메서드를 보유한 인터페이스가 있다고 가정했을 때

```kotlin
class StringPrinter: StringProcessor {
  override fun process(value: String) {
    print(value)
  }
}


class NullableStringPrinter: StringProcessor {
  override fun process(value: String?) {
    if(value != null) {
      print(value)
    }
  }
}
```

- 자바 클래스 또는 인터페이스를 코틀린에서 구현하는 경우 널 가능성을 제대로 처리하는 것이 중요하다.
- 코틀린 컴파일러는 널이 될 수 없는 타입으로 선언한 모든 파라미터에 대해 널이 아님을 검사하는 단언문을 만들어 준다.

---

### 6.2 코틀린의 원시 타입
코틀린은 원시 타입과 래퍼 타입을 구분하지 않고 동작한다.

#### 6.2.1 원시 타입: Int, Boolean 등
코틀린은 아래와 같이 원시타입과 래퍼타입을 구분하지 않고, 항상 같은 타입으로 사용한다.

```kotlin
val i: Int = 1
val list: List<Int> = listOf(1, 2, 3)
```

또한, 숫자 타입 등 원시 카입의 값에 대해 메서드 호출이 가능하다.
```kotlin
fun showProgress (progress: Int) {
    // 숫자 타입의 함수 호출
    val percent = progress.coerceIn(0, 100)
    println("We're ${percent}% done!")
}

fun main(args: Array<String>) {
    showProgress(146)
}
```

- 코틀린은 항상 객체로 표현하는 것이 아니다.
- 대부분 코틀린의 Int 타입은 자바의 int 타입으로 컴파일된다.
- 제네릭 클래스를 사용하는 경우는 불가능하다.

|구분|타입|
|---|---|
|정수 타입|Byte, Short, Int, Long|
|부동소수점 타입|Float, Double|
|문자 타입|Char|
|불리언 타입|Boolean|

Int와 같은 코틀린 타입에는 널 참조가 들어갈 수 없기 때문에 자바 원시 타입으로 컴파일이 가능하다.

#### 6.2.2 널이 될 수 있는 원시 타입: Int?, Boolean? 등
널이 될 수 있는 코틀린 타입은 자바 원시 타입으로 표현이 불가능하며, 래퍼 타입으로 컴파일된다.

```kotlin
// age의 초기값을 null로 부여
data class Pserson (val name: String, val age: Int? = null) {
    fun isOlderThan (other: Pserson): Boolean? {
        if (age == null || other.age == null) {
            return null
        }

        return age > other.age
    }
}

fun main(args: Array<String>) {
    println(Pserson("Sam", 35).isOlderThan(Pserson("Amy", 42)))
    // age가 null이기 때문에 이 경우 Integer형으로 컴파일
    println(Pserson("Sam", 35).isOlderThan(Pserson("Jane")))
}
```

- Int? 타입의 두 값을 직접 비교할 수 없다.
- 그렇기 때문에 null 체크를 두 값 모두 수행해야한다.

```kotlin
val list = listOf(1, 2, 3)
```

위 코드에서 사용된 값들은 모두 Integer 타입이며 이유는 아래와 같다.
- List는 기본적으로 제네릭을 통해 타입을 받는다.
- JVM은 제네릭을 구현할 때 타입 인자로 원시 타입을 허용하지 않는다.
- 따라서 자바와 코틀린은 제네릭 클래스에 항상 박스 타입을 사용해야 한다.

#### 6.2.3 숫자 변환
코틀린과 자바의 가장 큰 차이점 중 하나는 숫자를 변환하는 방식이다.

- 자바는 한 타입의 숫자를 다른 타입의 숫자로 자동 변환한다.
- 코틀린은 자동 변환하지 않으며, 심지어 원래 타입의 범위보다 넓은 경우조차도 자동 변환이 불가능하다.

```kotlin
val intNumber = 1
val longNumber: Long = i.toLong()
```

위와 같이 직접 변환 메서드를 호출해야 한다. 이유는 다음과 같다.
- 개발자의 혼란을 피하기 위해 타입 변환을 명시하기로 결정
- 박스 타입을 비교하는 경우 문제 발생
- 두 박스 타입 간의 equals 메서드는 그 안에 값이 아닌 박스 타입 객체를 비교

```kotlin
val x = 1
val list = listOf(1L, 2L, 3L)

// 명시적으로 타입을 변환
println(x.toLong() in list)
```

**숫자 리터럴에 _ 사용**
긴 숫자 리터럴을 다룰 때 가독성 높게 표기가 가능하여 금액, 바이트 크기, 긴 상수 값 등을 표현할 때 유용하다.

```kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val bytes = 0b11010010_01101001_10010100_10010010
```

#### 6.2.4 Any, Any?: 최상위 타입
자바의 Object 클래스와 유사한 역할을 하는 타입으로 모든 널이 될 수 없는 타입의 조상 타입이다.

- 자바에서는 Object가 참조 타입들의 정점으로 역할을 수행하며 원시 타입은 적용되지 못한다.
- 코틀린의 Any는 원시 타입을 포함한 모든 타입의 조상 타입으로 동작한다.
- 단, Any도 null은 들어갈 수 없다.
- toString, equals, hashCode는 Any에 정의된 메서드를 상속한 것이다.

#### 6.2.5 Unit 타입: 코틀린의 void
자바의 void와 같은 기능을 하는 타입이다. 즉, 반환 값이 없는 경우 사용한다고 이해하면 편하다.

- 코틀린 함수의 반환 타입이 Unit이고, 그 함수가 제네릭 함수를 오버라이드 하지 않는다면 자바 void 함수로 컴파일된다.
- Unit은 모든 기능을 갖는 일반적 타입이며, void와 달리 인자로 사용 가능하다.

```kotlin
interface Processor<T> {
  fun process(): T
}

class NoResultProcessor: Processor<Unit> {
  override fun process () {
    println("")
  }
}
```
자바로 위 코드를 표현하면 다음과 같다.
```java
interface Processor<T> {
    T process();
}

class NoResultProcessor implements Processor<Void> {

    @Override
    public Void process() {
        System.out.println("");
        // Void를 반환 타입으로 사용하면 반드시 null을 반환해야한다.
        // 코틀린의 Unit에 대응하는 값이 없기 때문이다.
        return null; 
    }
}
```
제네릭에 T를 전달하는 구조를 유지하기 위해 자바에서는 Unit에 대응되는 값이 없기 때문에 Void를 제네릭으로 전달해주고, null을 반환한다.

#### 6.2.6 Nothing 타입: 이 함수는 결코 정상적으로 끝나지 않는다
'반환 값'이라는 개념 자체가 의미 없는 함수가 일부 존재하는데 그런 경우를 표현하기 위해 Nothing을 사용한다.

```kotlin
fun fail (message: String): Nothing {
    throw IllegalStateException(message)
}

fun main(args: Array<String>) {
    fail("Error")
}
```

- Nothing은 아무 값도 포함하지 않는다.
- 함수의 반환 타입이나 반환타입으로 쓰일 타입 파라미터로만 쓸 수 있다.
- 컴파일러는 Nothing이 반환 타입인 함수가 결코 정상 종료되지 않음을 알고 그 함수를 호출하는 코드를 분석할 때 사용한다.

```kotlin
val address = company.address ?: fail("No address")
```

컴파일러는 위 코드에서 엘비스 연산자의 우항에서 예외가 발생할 수 있다는 사실을 파악하고, address 값이 널이 아님을 추론할 수 있다.

---

### 6.3 컬렉션과 배열

#### 6.3.1 널 가능성과 컬렉션
컬렉션 안에 널 값을 넣을 수 있는 여부는 어떤 변수의 값이 널이 될 수 있는지와 마찬가지로 중요하다.

```kotlin
// 일반적 사용
fun readNumbers (reader: BufferedReader): List<Int?> {
    // 널이 될 수 있는 제네릭
    val result = ArrayList<Int?>()

    // lineSequence()는 지연평가를 지원한다
    // 코틀린의 Sequnece 형태로 감싸서 처리
    for(line in reader.lineSequence()) {
        try {
            val number = line.toInt()
            result.add(number)
        } catch (e: NumberFormatException) {
            result.add(null)
        }
    }

    return result
}

// 축약 : String.toIntOrNull()
fun readNumbers(reader: BufferedReader): List<Int?> {
    val result = ArrayList<Int?>()

    for (line in reader.lineSequence()) {
        result.add(line.toIntOrNull())
    }

    return result
}
```

`String.toIntOrNull()`은 코틀린 1.1부터 나온 것이 아니라 코틀린 1.0부터 이미 존재했었다.

```kotlin
List<Int?>
List<Int>?
List<Int?>?
```

- List<Int?> : 리스트의 각 원소가 널이 될 수 있는 타입이라고 지정
- List<Int>? : 리스트가 가리키는 변수에 널이 들어갈 수 있지만 List 각 원소에 널이 들어갈 순 없음
- List<Int?>? : 리스트를 가리키는 변수에 널이 들어갈 수도 있고, 각 원소에도 들어갈 수 있음

```kotlin
fun addValidNumbers (numbers: List<Int?>) {
    var sumOfValidNumbers = 0
    var invalidNumbers = 0

    for (number in numbers) {
        if(number != null) {
            sumOfValidNumbers += number
        } else {
            invalidNumbers++
        }
    }

    println("Valid numbers :: $sumOfValidNumbers")
    println("Invalid numbers :: $invalidNumbers")
}

fun main(args: Array<String>) {
    val list = listOf(1, null, 42)
    addValidNumbers(list)
}
```

위 코드처럼 컬렉션에서 널 값을 걸러내는 경우가 자주 발생하기 때문에 `filterNotNull`이라는 표준 라이브러리 함수를 제공한다.
```kotlin
fun addValidNumbers (numbers: List<Int?>) {
    // null이 아님을 보장하므로 List<Int> 타입
    val validNumbers = numbers.filterNotNull()

    println("Valid numbers :: ${validNumbers.sum()}")
    println("Invalid numbers :: ${numbers.size - validNumbers.size}")
}

fun main(args: Array<String>) {
    val list = listOf(1, null, 42)
    addValidNumbers(list)
}
```

#### 6.3.2 읽기 전용과 변경 가능한 컬렉션
자바와 코틀린 컬렉션의 가장 큰 차이는 코틀린에는 읽기 전용과 변경 가능한 컬렉션이 있다는 점이다.

- kotlin.collections.Collection : size, iterator(), contains(), isEmpty() 등 수정에 대한 메서드가 없음
- kotlin.collections.MutableCollection : add(), remove(), clear() 등 수정에 대한 메서드 보유
  - Collection 인터페이스를 확장
  - 원소의 추가, 삭제, 모두 지우기 등을 지원

컬렉션은 가능하면 항상 읽기 전용 인터페이스를 사용하여 데이터에 어떤 일이 벌어지는지를 더 쉽게 이해하자.

원본의 변경을 막기 위해 컬렉션을 복사하는(방어적 복사, defensive copy) 부분에 대한 코드를 보자.
```kotlin
// 읽기 전용과 변경 가능한 컬렉션을 인자로 전달
fun <T>copyElements(source: Collection<T>, target: MutableCollection<T>) {
    // 원본을 변경 가능한 컬렉션으로 이동
    for (item in source) {
        target.add(item)
    }
}

fun main(args: Array<String>) {
    val source: Collection<Int> = arrayListOf(3, 4, 2)
    val target: MutableCollection<Int> = arrayListOf(1)

    copyElements(source, target)
    println(target)
}
```
만약 여기서 target이 읽기 전용이라면 아래와 같은 에러가 발생한다.

```kotlin
fun <T>copyElements(source: Collection<T>, target: Collection<T>) {
    for (item in source) {
        // Unresolved reference: add
        target.add(item)
    }
}
```

읽기 전용 인터페이스 타입 변수를 사용할 때 그 변수는 수많은 참조 중 하나일 수 있다.
이런 경우에는 변경 가능한 인터페이스의 참조가 있을 수도 있기 때문에 읽기 전용이라고 해서 반드시 변경 불가능한 컬렉션을 필요는 없다.

```kotlin
fun main() {
    val mutableList: MutableList<Int> = mutableListOf(1, 2, 3)
    // 같은 리스트를 읽기 전용 인터페이스로 참조
    val readOnlyList: List<Int> = mutableList

    println("Before modification:")
    println("mutableList: $mutableList")
    println("readOnlyList: $readOnlyList")

    // mutableList를 통해 컬렉션을 변경
    mutableList.add(4)

    // readOnlyList도 변경된 내용을 반영
    println("After modification:")
    println("mutableList: $mutableList")
    println("readOnlyList: $readOnlyList")
}
```

위 코드에서 주의 사항은 MutableList를 List의 참조로 받을 순 있지만 List를 MutableList의 참조로 받을 수는 없다.

- List는 읽기 전용이기 때문에 MutableList의 메서드 중 일부가 수행될 수 없다.
- MutableList는 변경 가능하기 때문에 List의 모든 메서드 수행이 가능하다.

또한, 이런 경우에 병렬 실행한다면 동시성 오류 발생을 야기할 수 있기 때문에 일긱 전용 컬렉션이 항상 스레드 안전하지 않다는 것을 인식해야한다.
