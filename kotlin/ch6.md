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

#### 6.1.5 안전한 캐스트: as?
#### 6.1.6 널 아님 단언: !!
#### 6.1.7 let 함수
#### 6.1.8 나중에 초기화할 프로퍼티
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
