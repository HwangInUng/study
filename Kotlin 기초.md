## 함수와 변수
### 코틀린 문법 특징
- `fun` 키워드를 이용해 함수 선언
- 파라미터 이름 뒤에 타입을 지정
- 반드시 **클래스 안에 함수**를 넣을 필요는 없음 -> **최상위 수준**에 함수 정의 가능
- 배열 처리 문법이 따로 존재하지 않는다
### 함수
#### 기본 문법
```kotlin
// 일반함수
[함수 키워드] [함수 이름] ([파라미터 목록]) : [반환 타입] {
	[함수 본문]
} 
// 익명함수
val function: (String, Int) -> String = {
	...실행부
}

// 예시 : 반환 값이 있는 함수
fun getMax(a: Int, b: Int): Int {
	return if(a > b) a else b
}
// 예시 : 반환 값이 없는 함수
fun main(args: Array<String>) {
	println("hi")
}
```
위 코드에서 if는 문장이 아닌 결과를 만들어내는 식이라는 점이 흥미롭다.
- 자바의 모든 제어 구조는 문으로 이루어져 있음
- 코틀린은 루프를 제외한 대부분의 제어 구조가 식으로 이루어져 있음
- 식은 값을 만들어내고, 문은 블록 최상위 요소로 존재하며 값을 만들어내지 않음
#### 간결한 함수 표현
```kotlin
fun getMax(a: Int, b: Int) => if(a > b) a else b
```
- 블록이 본문인 함수 : 본문이 중괄호로 둘러싸인 함수
- **식이 본문인 함수** : 등호와 식으로 이뤄진 함수

코틀린에서는 `식이 본문인 함수`가 더 자주 사용된다. 추가로 식이 본문인 함수는 굳이 사용자가 반환 타입을 적지 않아도 컴파일러가 함수 본문 식을 분석하기 때문에 반환 타입을 작성하지 않아도 된다.
#### 타입 추론 비교 - 코틀린 vs 타입스크립트

| 구 분              | 코틀린                                                                           | 타입스크립트                                                                                   |
| :--------------- | :---------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------- |
| 타입 시스템의 차이       | - 명목적 타입 시스템 사용<br>- 타입의 이름과 선언에 기반하여 타입을 구분<br>- 상속이나 구현 관계를 통해 타입 간의 관계를 정의 | - 구조적 타입 시스템 사용<br>- 객체의 실제 구조나 프로퍼티에 따라 타입 호환성 결정<br>- 동일한 형태를 가지면 다른 타입이라도 호환되는 경우가 발생 |
| 범위와 강도           | - 로컬 변수와 함수의 반환 타입에 대해 강력하게 적용<br>- 클래스 프로퍼티나 함수 매개변수의 타입은 명시적으로 선언해야하는 경우 많음 | - 변수, 함수 반환 타입, 매개변수, 제네릭 등 다양한 곳에서 타입 추론 발생<br>함수의 매개변수나 제네릭 타입 인자도 추론하여 코드 간결성 향상      |
| 제네릭 타입 추론        | - 타입 인자를 명시적으로 제공                                                             | - 전달된 인수를 기반으로 제네릭 타입 인자를 효과적으로 추론                                                       |
| Null 안정성 처리      | - Null 안정성을 지원하여 nullalble 타입과 non-null 타입 구분                                 | - Null과 Undefined에 대한 처리가 엄격하지 않음                                                        |
| 타입 어노테이션의 필요     | - 명시적으로 선언을 권장                                                                | - 타입 추론에 더 의존하여 생략                                                                       |
| 함수형 프로그래밍과 고차 함수 | - 람다 표현식과 고차 함수 적극 활용<br>- 타입 추론은 제한적이므로 타입을 명시적으로 지정                         | - 함수 타입에 대한 추론이 비교적 강력<br>- 고차 함수에서도 타입을 명시하지 않고 사용                                      |
### 변수
키워드로 변수 선언을 시작하는 대신 변수 이름 뒤에 타입을 명시하거나 생략하게 허용한다.
#### 기본 문법
```kotlin
// 타입 표기 생략
val question = "질문"
val answer = 11

// 타입 표기
val question: String = "질문"
val answer: Int = 11
```
타입 표기 생략 시 컴파일러가 초기화 식을 분석해서 초기화 식의 타입을 변수 타입으로 지정하며, 초기화 식이 없는 경우는 아래와 같이 반드시 타입 지정
```kotlin
// 초기화 식이 없으면 정보가 없어 타입 추론 불가
val question: String
question = "test"
```
#### 변수 종류
- val : 변경 불가능한 참조를 저장하는 변수 -> 자바의 final
- var : 변경 가능한 참조를 저장하는 변수 -> 자바의 일반 변수
#### 코틀린과 자바 변수 비교
**선언 방식**
Java
```java
int number = 10;
String name = "Alice";
final int MAX_SIZE = 100; // 불변 변수
```

Kotlin
```kotlin
val number = 10
var name = "Alice"
val MAX_SIZE = 100
```
`val`과 `var`를 이용해 간편하게 불변 변수를 식별이 가능하다.

**타입 추론**
Java
```java
// Java 10 이상 버전부터 var 사용 가능
var number = 10;
```
클래스 변수 및 메소드의 매개변수에서는 사용이 불가능

Kotlin
```kotlin
fun sum(a: Int, b: Int) = a + b
```
변수, 함수 반환 타입, 람다식 등 다양한 곳에서 타입 추론 지원

**Null 안정성**
Java
```java
String name = null;
int length = name.length(); // NullPointException 발생
```

Kotlin
```kotlin
var name: String = null // 컴파일 에러
var name: String? = null // null 허용
```

**컬렉션 초기화**
Java
```java
List<String> names = new ArrayList<>();
names.add("Alice");
names.add("Tom");
```

Kotlin
```kotlin
val names = listOf("Alice", "Tom")
val mutableNames = mutableListOf("Alice", "Tom")
```
#### 불변 객체의 내부 값 변경
`val` 참조 자체는 불변이지만 객체의 내부 값은 변경이 가능하다.
```kotlin
val names = arrayListOf("Tom")
names.add("Kai")
```
---
## 클래스와 프로퍼티
먼저 자바와 코틀린이 클래스를 정의할 때 보이는 차이점을 보고가자.

Java
```Java
public class Person() {
	private String name;
	
	Person (String name) {
		this.name = name;
	}

	public String getName() {
		return name;
	}
}
```

**Kotlin**
```kotlin
class Person(val name: String)
```
위와 같은 클래스 유형을 `값 객체`라고 부른다.
### 프로퍼티
자바는 데이터를 필드에 저장하며, 멤버 필드의 가시성은 보통 `private`로 비공개 처리한다. 이와 같은 이유로 접근자 메서드를 제공하며 필드와 접근자를 묶어 프로퍼티라고 부른다.

코틀린에서는 `val`과 `var`로 프로퍼티를 선언하며, `val`은 읽기 전용이다.
- val : 읽기 전용으로 `getter`만 선언
- var : 변경 가능으로 `getter`와 `setter` 모두 선언

#### getter와 setter 규칙
이름이 `is`로 시작하는 Boolean 타입 프로퍼티의 게터에는 get이 붙지 않고 원래 이름을 그대로 사용하며, 사용자 정의 게터와 세터에도 동일하게 적용된다.
```kotlin
val isMarried: Boolean = true

// getter
isMarried()
// setter
setMarried(false)
```
위와 같이 처리하는 이유는 다음과 같다.
- Java와의 호환성
- **Java Bean 규약**에서는 Boolean 타입의 게터는 is로 시작하고, 세터는 set으로 시작

>Java Bean 규약을 따라야하는 이유
>- 스프링 또는 하이버네이트 등 많은 자바 프레임워크가 JavaBean 규약을 기반으로 동작
>- 리플렉션 또는 직렬화 과정에서 프로퍼티를 자동으로 인식하고 처리가 가능
>- 명명 규칙이 통일되면 코드의 이해와 유지 보수가 쉬워짐
>  
>일반 프로퍼티의 경우:
>- 게터: get + 프로퍼티 이름의 첫 글자를 대문자로 변환
>- 예: 프로퍼티 이름이 name이면 게터는 getName()
>- 세터: set + 프로퍼티 이름의 첫 글자를 대문자로 변환
>- 예: 세터는 setName(value)
>Boolean 타입의 프로퍼티가 is로 시작하지 않는 경우:
>- 게터: is + 프로퍼티 이름의 첫 글자를 대문자로 변환
>- 예: 프로퍼티 이름이 active이면 게터는 isActive()
>- 세터: set + 프로퍼티 이름의 첫 글자를 대문자로 변환
>- 예: 세터는 setActive(value)
>Boolean 타입의 프로퍼티가 is로 시작하는 경우:
>- 게터: 프로퍼티 이름 그대로 사용
>- 예: 프로퍼티 이름이 isEnabled이면 게터는 isEnabled()
>- 세터: set + is를 제외한 프로퍼티 이름의 첫 글자를 대문자로 변환
>- 예: 세터는 setEnabled(value)  

### 커스텀 접근자
```kotlin
class Ractangle(val height: Int, val width: Int) {
	val isSquare: Boolean
	get() = height == width
}

// 사용 예시
val rectangle = Rectangle(41, 43)
println(rectangle.isSquare)
```
### 소스 코드
코틀린의 소스코드 구조는 다음과 같은 특징이 있다.
- 같은 패키지에 속해 있다면 다른 파일에서 정의한 선언일지라도 직접 사용
- 클래스 임포트와 함수 임포트에 차이가 없음
- 최상위 함수는 그 이름을 써서 임포트 가능
#### Java와 차이

| 구 분    | 차이점                                                                                |
| :----- | :--------------------------------------------------------------------------------- |
| Java   | - 패키지의 구조와 일치하는 디렉터리 계층 구조<br>- 클래스의 소스코드를 그 클래스가 속한 패키지와 같은 디렉터리에 위치              |
| Kotlin | - 여러 클래스를 한 파일에 넣을 수 있음<br>- 파일 이름도 마음대로 설정<br>- 디스크상의 어느 디렉터리에 소스코드 파일을 위치 시키든 무관 |
위와 같은 차이가 있지만 대부분은 패키지별로 디렉터리를 구성하는 편이 낫다. 자바의 방식을 따르지 않으면 자바 클래스를 코틀린 클래스로 마이그레이션할 때 문제가 발생할 수 있다.
### enum과 when
#### Enum : 생략
#### when
when은 자바의 switch문을 대체할 수 있는 구성 요소다. `break`를 작성하지 않아도 된다는 점이 특징이다.

**Java**
```java
int day = 3;
String dayName;

// 각 분기마다 break 작성
switch (day) {
    case 1:
        dayName = "월요일";
        break;
    case 2:
        dayName = "화요일";
        break;
    case 3:
        dayName = "수요일";
        break;
    default:
        dayName = "알 수 없는 요일";
        break;
}

System.out.println(dayName); // 출력: 수요일
```

**Kotlin**
```kotlin
val day = 3
val dayName = when (day) {
    1 -> "월요일"
    2 -> "화요일"
    3 -> "수요일"
    else -> "알 수 없는 요일"
}

println(dayName) // 출력: 수요일
```

자바는 switch만으로 표현하기 어려운 경우들이 아래와 같이 발생한다. 이때 when의 사용을 보자.
**Java**
```java
int score = 85;
String grade;

if (score >= 90) {
    grade = "A";
} else if (score >= 80) {
    grade = "B";
} else if (score >= 70) {
    grade = "C";
} else {
    grade = "F";
}

System.out.println(grade); // 출력: B
```

**Kotlin**
```kotlin
val score = 85
val grade = when {
    score >= 90 -> "A"
    score >= 80 -> "B"
    score >= 70 -> "C"
    else -> "F"
}

println(grade) // 출력: B
```

when은 다음과 같은 특징을 가진다.
- 표현식으로 값을 반환하기 때문에 변수에 직접 할당 가능
- 인자 없이 조건식으로도 사용할 수 있어, 복잡한 조건 분기에 유용
- 타입 검사, 범위 검사 등 다양한 조건 지원
- 코드의 간결성 향상

**when 임의 객체 사용**
자바의 switch는 상수만을 사용할 수 있다는 규칙이 있는 반면 코틀린의 when은 객체 사용도 가능하다.
```kotlin
fun mix(c1: Color, c2: Color) = 
	when (setOf(c1, c2)) {
		setOf(RED, YELLOW) -> ORANGE
		....
		else -> throw Exception("exception)
	}
```
임의의 객체 사용 시 집합 비교를 사용한다.

**if와 when의 블록 사용**
블록을 사용할 경우 각 분기 블록의 마지막 문장이 블록 전체의 결과가 된다. 이 규칙은 블록이 값을 만들어내야하는 경우 항상 성립하지만 함수에 대해서는 성립하지 않는다.

함수는 식이 본문일 경우 블록이 없으며, 블록이 있는 경우 반드시 `return`을 포함해야하기 때문이다.

### while과 for 루프
#### while
```kotlin
while(조건) {
	...
}

do {
	...
} while(조건) {
	...
}
```
자바와 동일하게 동작한다.

#### for
for 루프는 자바의 for 루프에 해당하는 요소는 없다.
```kotlin
// 1부터 100까지를 포함하는 폐구간
// 두번째 (100) 값을 항상 범위에 포함
for (1 in 100) {
	...
}

// 1부터 99까지 포함하는 반만 닫힌 범위
for (i in 1 until 100) {
	...
}

// 역순
for (i in 100 downTo 1) {
	...
}
// 역순 스텝 조정
for (i in 100 downTo 1 step 2) {
	...
}

// Map 이터레이션
for ((key, value) in maps) {
	...
}
```
---
## 예외처리
자바와 다른 부분은 아래와 같다.
- new 연산자를 사용할 필요가 없다.
- throws 절이 없다.
- throw는 식으로 취급되어 다른 식에 포함 가능하다.
- 체크와 언체크 예외의 구분이 없다.
### try, catch, finally
#### try
식으로 작동한다. if나 when과 마찬가지이므로 변수에 대입이 가능하다. catch를 통해 코드의 진행 여부를 결정 할 수 있다.

catch 블록에서 `return` 키워드 여부에 따라 코드의 진행이 결정된다.

---
## 추가
### 코틀린 메모리 구조
>JVM 위에서 동작하기 때문에 메모리의 구조는 자바와 동일하며, 카비지 컬렉션도 동일한 메커니즘을 사용하여 메모리를 관리

언어적 특성으로 인한 메모리 사용 패턴의 영향
- 코루틴 : 비동기 프로그래밍을 위한 경량 스레드로 힙에 저장된 상태 기계를 통해 관리
- 인라인 함수 : 고차 함수 및 람다 표현식을 사용하는 경우 성능 최적화를 위해 인라인 처리
- 널 안정성 : 컴파일 단계에서 널 관련 오류 방지를 하며, 추가적인 런타임 체크 코드가 삽입 될 가능성이 있음
- 데이터 클래스 : `equals()`, `hashCode()` 등 메서드 자동 생성으로 인한 영향

실무에서는 메모리의 차이가 약간은 있을 수 있지만 보통 미미하기 때문에 고려할만한 상황은 아니다.

### 기본 자료형
>기본 자료형의 종류는 유사하나 코틀린은 기본 자료형도 객체로 취급하여 널 안정성 등의 특징을 적용

#### 코틀린의 기본 자료형

**정수형**
- Byte
- Short
- Int
- Long

**실수형**
- Float
- Double

**문자형**
- Char

**불리언형**
- Boolean

**문자열형**
- String

모두 대문자로 시작하여 클래스라는 것을 알 수 있다. 컴파일러가 최적화를 통해 자바의 기본 자료형으로 변환하여 성능을 유지한다.

다음과 같은 특징이 있다.
- 객체이므로 메서드와 프로퍼티 사용 가능
- 널 안정성 달성
- 타입 추론의 용이성 달성
- 기본 자료형에 대한 특별한 배열 클래스 제공
