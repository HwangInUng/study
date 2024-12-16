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
class MyService {
    fun performAction(): String = "foo"
}

class MyTest {
    // lateinit을 사용할 때 var로 프로퍼티 선언
    private lateinit var myService: MyService? = null
    @Before
    fun setUp() {
        myService = MyService()
    }

    @Test
    fun testAction() {
        Assert.assertEquals(
            "foo",
            myService.performAction()
        )
    }
}
```

- lateinit 프로퍼티는 DI 프레임워크와 사용하는 경우가 많다.
- lateinit 프로퍼티의 값을 DI 프레임워크가 외부에서 설정해준다.
- 자바 프레임워크와의 호환성을 위해 lateinit가 지정된 프로퍼티와 가시성이 똑같은 필드를 생성한다.

#### 6.1.9 널이 될 수 있는 타입 확장
#### 6.1.10 타입 파라미터의 널 가능성
#### 6.1.11 널 가능성과 자바

---

### 6.2 코틀린의 원시 타입
#### 6.2.1 원시 타입: Int, Boolean 등
#### 6.2.2 널이 될 수 있는 원시 타입: Int?, Boolean? 등
#### 6.2.3 숫자 변환
#### 6.2.4 Any, Any?: 최상위 타입
#### 6.2.5 Unit 타입: 코틀린의 void
#### 6.2.6 Nothing 타입: 이 함수는 결코 정상적으로 끝나지 않는다
