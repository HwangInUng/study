## 람다로 프로그래밍

### 들어가기 전에
#### 람다란?
>A lambda expression is like a method: it provides a list of formal parameters and a body - an expression or block - expressed in terms of those parameters.
>
>람다표현식은 메서드와 유사하다. 형식 매개변수 목록과 해당 변수로 표현된 본문 표현식 또는 블록을 제공한다.
>Oracle 공식문서 중

#### Java의 람다
>간결한 방식으로 익명 함수를 표현하는 문법적 요소로서 동작한다. 즉, 메서드 몸체를 별도의 이름 없이 바로 식(expression) 형태로 전달할 수 있는 기능을 제공한다.

##### 특징
- 이름 없는 함수 : 함수 자체를 값으로 다룰 수 있어, 변수 할당 및 메서드 인자로 전달 가능
- 간결성 : 익명 클래스를 사용하는 경우에 비해 코드 길이가 줄어들고 가독성 향상
- 함수형 프로그래밍 스타일 지원 : 스트림(Stream) 등과 함께 사용될 때, 함수형 프로그래밍 패러다임에 가까운 스타일로 코드 작성 가능

**코드**
```java
// 익명 클래스
Runnable r = new Runnable() {
  @Override
  public void run () {
    System.out.println("Hello);
  }
};

// 람다 표현식
Runnable rLambda = () -> System.out.println("Hello");
```

##### 동작 방식
>Java의 람다는 단독으로 존재하지 않고, 함수형 인터페이스를 기반으로 동작한다. 여기서 함수형 인터페이스는 단 하나의 추상 메서드를 가진 인터페이스이다.

`Runnable`, `Comparator<T>` 등은 메서드를 각 1개씩만 가지고 있으며, 이런 함수형 인터페이스들을 통해 람다를 동작시킬 수 있다.
추가로 람다에서 중요한 두 가지 항목이 있는데 다음과 같다.

- 타입 추론
  ```java
  stream.filter(x -> x > 10);
  ```
  위 코드에서 x의 타입은 `filter` 메서드의 시그너치에 의해 추론되는데 타입 추론은 이와 같이 람다에서 파라미터의 타입은 컴파일러에 의해 추론되는 것을 의미한다.
- 메서드 참조
  ```java
  // 원래 코드
  (String s) -> s.length();

  // 메서드 참조
  String::length
  ```
  위 코드에서처럼 특정 메서드를 간접적으로 호출하여 축약할 수 있는 메서드 참조가 있다.

##### 장단점
**장점**
- 코드 간결성 : 익명 클래스보다 훨씬 잛은 코드로 기능 구현이 가능
- 가독성 향상 : 코드의 간결성 확보로 보일러플레이트 코드가 줄어들어 가독성 향상
- 함수형 프로그래밍 지원 : 선언적이며, 직관적인 코드 표현 가능
- 생산성 증가 : 코드 양이 절대적으로 줄어들며, 선언적 코드 작성으로 의도가 명확해져 생산성 증가

**단점**
- 디버깅 난이도 증가 : 익명 함수를 대체하므로 디버깅 시 스택 트레이스에 명확한 출처가 남지 않아 추적 제한
- 복잡한 로직에 부적합 : 긴 로직의 경우 오히려 코드 가독성을 망치는 경우가 발생

##### 내부적으로 어떻게 동작할까?
>invokedynamic과 LambdaMetafactory를 사용하여 **바이트 코드 명령어를 활용**해 **런타임**에 함수형 인터페이스의 인스턴스를 생성한다.

- invokedynamic : 자바 바이트코드 상의 명령어, 동적인 메서드 호출 부착점(call site)을 런타임에 결정
- LambdaMetafactory : 부트스트랩 메서드로 이용되는 클래스, 람다 표현식을 적절한 함수형 인터페이스 구현체로 변환하는 공장 역할

Java는 람다 표현식을 컴파일 단계에서 익명 클래스로 직접 변환하지 않고, 필요할 때만 동적으로 구현체를 만들어내어 메모리 사용량 및 클래스 로딩 오버헤드를 감소시키는 방식으로 구현되어 있다.
구체적인 구현 과정은 다음과 같다.
- 컴파일
  - 람다 표현식을 만나면 해당 람다를 호출하는 브리지(bridge) 같은 람다 팩토리 함수를 설정하는 바이트 코드 생성
  - 람다 자체를 별도의 익명 클래스로 컴파일하지 않음
- 동적 생성
  - JVM이 `invokedynamic` 명령을 사용
  - `LambdaMetafactory`를 통해 필요한 시점에 람다 구현체 생성
  - 구현체는 함수형 인터페이스를 구현하는 프록시 클래스 또는 비슷한 형태의 오브젝트로 생성
  - 여기서 비슷한 형태의 오브젝트는 JVM 내부적으로 최적화되고 동적으로 만들어진다는 의미

위와 같은 과정에서 JVM은 런타임 최적화를 통해 람다 호출을 빠르게 만들며, Java는 언어 차원에서 `경량 함수`를 제공할 수 있게되었다.

결론적으로 Java 8 이후에 등장한 람다 표현식과 익명 클래스의 차이는 다음 표와 같다.
|구분|람다 표현식|익명 클래스|
|---|---|---|
|클래스 파일|미생성|생성|
|클래스 형태|클래스 생성 x, 메서드 생성하여 포함|static 중첩 클래스 생성|
|this 사용|람다를 포함하는 클래스의 인스턴스|새로 생성된 클래스 인스턴스|
|메모리 사용량|감소|증가|
|클래스 로딩 오버헤드|감소|증가|

---

### 5.1 람다식과 멤버 참조
#### 5.1.1 람다 소개 : 위 자바의 람다 설명으로 대체한다.

#### 5.1.2 람다와 컬렉션
>람다가 없는 경우 컬렉션을 편리하게 처리할 수 있는 좋은 라이브러리를 제공하기 힘들다.

다음 코드를 보면서 이해해보자.
예시에 사용된 클래스는 다음과 같다.
```kotlin
data class Person(val name: String, val age: Int)
```

**컬렌셕 직접 탐색**
```kotlin
fun findOldest(people: List<Person>) {
    var maxAge = 0
    // null 허용
    var theOldest: Person? = null
    // 반복문을 직접 수행하며 대소 비교
    for (person in people) {
        if (person.age > maxAge) {
            // 조건에 해당하는 결과를 변수에 저장
            maxAge = person.age
            theOldest = person
        }
    }
    // 결과 출력
    println(theOldest)
}
```
직접 탐색의 경우 중복적이고 단순 반복되는 코드들이 작성되기 마련이고, 개발자에 의한 실수가 발생할 수 있다.

**람다 사용**
```kotlin
// 타입 명시
println(people.maxBy{p:Person -> p.age})

// it 키워드 사용
// 람다의 파라미터가 하나이고, 컴파일러가 타입 추론이 가능할 때 사용 가능
println(people.maxBy { it.age })

// 멤버 참조 방식 : ()를 사용
println(people.maxBy(Person::age))
```

자바에는 `maxBy()` 함수처럼 라이브러리 함수가 부족한 상황이다.

#### 5.1.3 람다 식의 문법
>람다는 값처럼 여기저기 전달할 수 있는 동작의 모음이다.

람다에는 몇가지 규칙이 있는데 다음과 같다.
- 복수 개의 인자 중 마지막 인자가 람다인 경우 () 밖으로 빼내서 작성 가능 -> Trailing Lambda 문법
- 인자가 람다만 존재할 경우 ()를 생략하여 작성 가능
- 파라미터 타입은 컴파일러에 의한 추론으로 생략 가능
- 멤버 참조를 사용 가능
- it 키워드 사용 가능

**문법**
```kotlin
// 정의
{x: Int, y: Int -> x + y}

// 호출 및 사용
val sum = {x: Int, y: Int -> x + y}
println(sum(1, 2)}

// Trailing Lambda 문법
people.testFun(x, y, { it.age })
people.testFun(x, y) { it.age }

// 인자가 람다뿐이면 () 생략
people.testFun { it.age }

// 실행부가 여려 줄인 경우
val sum = {x: Int, y: Int ->
  ... 실행부 코드
  x + y  // 반환 위치
}
```
코틀린 람다 문법의 특징은 항상 `{}`로 둘러쌓여 있다. 그렇다면 자바의 문법은 어떤지 살펴보자.

```java
// 타입 생략
BinaryOperation<Integer> sum = (x, y) -> x + y;
// 타입 명시
BinaryOperation<Integer> sum = (Integer x, Integer y) -> x + y;
// 실행부가 여러 줄인 경우
BinaryOperation<Integer> sum = (Integer x, Integer y) -> {
  ... 실행부 코드
  return x + y;
};
```
위와 같이 파라미터는 `()`로 감싸주고, 실행부는 여러 줄인 경우에만 `{}`로 감싸준다.

**주의 사항**
>it을 사용하는 관습은 코드를 간단히 만들지만 람다가 중첩되는 경우 it의 대상을 식별하기가 어렵다.

```kotlin
val people = listOf(Person("Alice", 30), Person("Bob", 25))

// 두 람다 모두 it을 사용
val result = people.filter { it.age > 20 }     // 여기의 it은 Person 객체를 가리킴
    .map { it.name.length }     // 여기의 it은 filter 결과의 각 Person 객체를 가리키지만,

// 개선된 방식
val result = people.filter { person -> person.age > 20 }
                   .map { filteredPerson -> filteredPerson.name.length }
```

#### 5.1.4 현재 영역에 있는 변수에 접근
>람다를 함수 안에서 정의하면 함수의 파라미터뿐 아니라 람다 정의 앞에 선언된 로컬 변수까지 람다에서 모두 사용 가능하다.

이 부분에서의 자바와 코틀린의 차이는 다음과 같다.
- 자바 : `final`로 명시적으로 선언되거나 `effectivly final`로 사실상 바뀌지 않는 상태여야한다.
  - 자바에서 외부 변수에 대한 변경을 시도하면 다음과 같은 컴파일 에러가 발생한다.
  - `"Local variables referenced from a lambda expression must be final or effectively final"`
- 코틀린 : `final` 여부와 관계없이 참조가 가능하며, `var` 변수의 경우 람다 안에서 값을 변경하는게 가능하다.

**변수 접근**
```kotlin
fun printMessagesWithPrefix (messages:Collection<String>, prefix: String) {
    // 인자가 람다뿐이기 때문에 () 생략
    messages.forEach {
        // 람다를 정의한 함수의 파라미터 사용
        println("$prefix $it")
    }
}
```
java에서 작성하면 다음과 같다.
```java
public void printMessagesWithPrefix (List<String> messages, String prefix) {
  messages.forEach(message -> System.out.println(prefix));
}
```

코틀린 코드에서 볼 수 있듯이 자바와 가장 큰 차이점은 바깥의 변수를 변경할 수 있고, `final` 변수가 아닌 변수에도 접근 할 수 있다는 것이다.
다음은 바깥 함수의 로컬 변수 변경하는 예제이다.
```kotlin
fun printProblemCounts (response: Collection<String>) {
    // final이 아닌 함수의 로컬 변수
    var clientErrors = 0
    var serverErrors = 0

    response.forEach {
        // 조건에 맞는 변수 값 변경
        if (it.startsWith("4")) {
            clientErrors++;
        } else if (it.startsWith("5")) {
            serverErrors;
        }
    }

    println("$clientErrors client errors, $serverErrors server errors")
}
```

**포획한 변수(capture)**
>람다 안에서 사용하는 외부 변수를 '람다가 포획한 변수'라고 칭한다. 이렇게 람다가 포획한 변수는 함수 내부에서 참조할 수 있다.

이렇게 람다가 포획한 변수를 함수 내부에서 참조할 수 있는 메커니즘을 클로저(closure)라고 한다.
클로저는 다음과 같은 특징을 가진다.

- 함수가 선언될 당시의 스코프 유지
- 외부 함수의 지역 변수나 상태를 함수 실행 시점에 참조 가능
- 스코프에 대한 참조를 유지하여 외부 함수의 생명 주기가 끝나도 해당 변수를 사용 가능

코틀린 코드를 통해 클로저에 대해 알아보자.
```kotlin
// () -> Int 타입 람다 반환
fun outer(): () -> Int {
    // 지역 변수
    var count = 0
    return {
        count++
        count
    }
}

fun main() {
    // 람다
    val counter = outer()
    println(counter()) // 1
    println(counter()) // 2
}
```
위 코드에서의 클로저 동작은 다음과 같다.

- outer()는 `() -> Int` 타입의 람다를 반환
- `() -> Int`는 outer() 함수 내부에 선언된 `count` 변수를 캡처
- outer() 함수 호출 이후에도 `count` 값에 접근하고 증가가 가능
- `counter`를 통해 람다를 호출할 때마다 `count`가 증가

즉, `() -> Int` 타입의 람다는 `count`를 캡처하여 계속해서 참조하는 것이다.
캡처된 정보는 어디에 어떻게 보관하는지 알아보자.

- 코틀린 컴파일러가 람다를 구현하기 위해 익명 클래스 혹은 특별한 객체를 생성
- 해당 객체에 필드로 캡처된 변수에 대한 참조를 저장
- 람다 객체는 이 필드를 통해 외부 변수의 상태를 유지 및 접근
- 스택에 쌓였던 count는 람다가 캡처 시 람다 객체의 필드로 옮겨져 힙 영역에 위치

주의 해야할 부분은 람다를 이벤트 핸들러 또는 비동기적으로 실행되는 코드로 사용할 경우 함수 호출이 종료된 후 로컬 변수가 변경될 수 있다는 점이다.

#### 5.1.5 멤버 참조
>프로퍼니타 메서드를 단 하나만 호출하는 함수 값을 만들어주며 이중콜론(::)을 사용한다.

기본적인 사용법은 다음과 같다.
```kotlin
// 람다식
val getAge = {person: Person -> person.age}
// 멤버 참조
val getAge = Person::age
```

최상위에 선언된 함수나 프로퍼티 참조도 가능하다.
```kotiln
fun salute() = println("Salute")

run(::salute)
```

인자가 여럿인 다른 함수한테 작업을 위임하는 경우 람다를 정의하지 않고, 직접 위임 함수에 대한 참조를 제공하면 편리하다.
```kotiln
val action = {person: Person, message: String ->
  sendEmail(person, message) // 람다가 작업을 위임한 함수
}

val nextAction = ::sendEmail // 람다 대신 멤버 참조로 사용 가능
```

**생성자 참조**
```kotlin
// 인스턴스를 생성하는 동작을 값으로 저장
val createPerson = ::Person
val p = createPerson("Alice", 29)
```
확장 함수도 멤버 함수와 똑같은 방식으로 참조가 가능하다는 점을 기억하자.

**바운드 멤버 참조**
코틀린 1.1부터 지원하는 참조 방식으로 멤버 참조를 생성할 때 클래스 인스턴스를 함께 저장하여 나중에 호출하여 사용 가능하다.

```kotlin
val createPerson = ::Person
val p = createPerson("Alice", 29) // 인스턴스 저장
println(p::age) // 바운드 멤버 참조
```

---

### 5.2 컬렉션 함수형 API
#### 5.2.1 필수적인 함수 : filter와 map
filter와 map은 컬렉션을 활용할 때 기반이 되는 함수로 다음과 같은 특징이 있다.

- filter
  - 컬렉션을 이터레이션할 때 주어진 술어를 만족하는 요소만 반환
  - 기존 컬렉션과 반환되는 컬렉션의 요소 수가 다를 수 있음
  - 원소 자체를 변환할 수 없음
- map
  - 컬렉션의 모든 요소를 순회하며 작업을 수행
  - 기존 컬렉션과 반환되는 컬렉션의 요소 수가 동일
  - 원소에 대한 변경이 가능
 
추가적으로 자바에는 컬렉션 자체에 filter와 map이 존재하지 않기 때문에 `list.stream().map()` 처럼 `stream()`을 사용하여야한다.

**코드**
```kotlin
val list = listOf(1, 2, 3, 4)

// filter
val filterdList = list.filter {it % 2 == 0}
println(filterdList.size)
>> 2

// map
val zeroList = list.map {it * 0}
println(zeroList.size)
>> 4
```
자바 코드는 다음과 같다.

```java
// filter
List<Integer> list = Arrays.asList(1, 2, 3, 4);

List<Integer> filteredList = list.stream()
                                 .filter(it -> it % 2 == 0)
                                 .collect(Collectors.toList());
System.out.println(filteredList.size());
>>2

// map
List<Integer> zeroList = list.stream()
                             .map(it -> it * 0)
                             .collect(Collectors.toList());
System.out.println(zeroList.size());
>>4
```

멤버 참조를 이용하면 아래와 같이 작성도 가능하다.
```kotlin
data class Person (val name: String, val age: Int)

val people = listOf(Person("hwang", 33), Person("park", 32))
println(people.map{Person::name})
>> ["hwang", "park"]
```

**주의 사항**
컬렉션 API를 사용하다보면 체이닝 순서 및 술어 조건으로 중첩되게 사용하는 경우가 발생한다.
조건에 대한 구분을 명확히하여, 사전에 계산이 가능한 부분은 계산 후 컬렉션 API를 활용하도록 연습하자.

```kotlin
// maxBy()가 people의 요소 수만큼 중첩으로 반복되어 실행
people.filter { it.age == people.maxBy(Person::age)!!.age}

// 개선
// 참고로 !!. 은 null이 아니라고 단언하는 연산자
val maxAge = people.maxBy{Person::age}!!.age
people.filter {it.age == maxAge}
```
위 코드처럼 최대 값을 구하고, 그 값을 기준으로 filter를 수행하면된다.

마지막으로 filter와 map을 Map에 적용해보자.
```kotlin
val numbers = mapOf(0 to "zero", 1 to "one")

// mapValues를 통해 Map의 value수만큼 이터레이션을 진행하며 값 변경
println(numbers.mapValues {it.value.toUpperCase()}
>> {0=ZERO, 1=ONE}
```

코틀린에서 두 줄짜리 코드가 자바로 구현하면 어떻게 되는지 알아보자.
```java
Map<Integer, String> numbers = new HashMap<>();
numbers.put(0, "zero");
numbers.put(1, "one");

// Stream API를 사용해 값(value)을 변경한 새로운 Map 생성
Map<Integer, String> upperCaseMap = numbers.entrySet().stream()
        .collect(Collectors.toMap( // Collectors 유틸 클래스를 통해 맵으로 변환
                Map.Entry::getKey,  // key를 호출
                e -> e.getValue().toUpperCase() // 코틀린에서 실제 사용된 부분
        ));

System.out.println(upperCaseMap); 
>>{0=ZERO, 1=ONE}
```

코틀린은 이 외에도 Map을 다루는 함수가 다음과 같이 존재한다.

- filterKeys
- mapKeys
- filterValues
- mapValues

#### 5.2.2 all, any, count, find : 컬렉션에 술어 적용
각 함수는 다음과 같은 역할을 수행한다.

- all : 모든 원소가 술어를 만족하는지 여부
- any : 하나의 원소라도 술어를 만족하는지 여부
- count : 술어를 만족하는 원소들의 수
- find : 술어를 만족하는 가장 첫 번째 원소 반환

사용법은 다음과 같다.
```kotlin
fun allAndOthers() {
    val people = listOf(Person("Alice", 27), Person("Jay", 32), Person("H", 33))

    // 모든 원소가 25살 이상인지
    println(people.all { it.age > 25 })
    // 원소중 20대가 있는지
    println(people.any { it.age < 30 })
    // 30대인 사람은 몇명인지
    println(people.count { it.age > 29 })
    // 30대 중 가장 첫 번째 원소
    println(people.find { it.age > 29 }!!.name)
}

allAndOthers()
>> true
>> true
>> 2
>> Jay
```

all, any, count, find가 자바에서는 어떻게 표현되는지 알아보자.
```java
public static void main(String[] args) {
    List<Person> people = Arrays.asList(
        new Person("Alice", 27),
        new Person("Jay", 32),
        new Person("H", 33)
    );

    // 모든 원소가 25살 이상인지
    boolean allOver25 = people.stream().allMatch(p -> p.getAge() > 25);
    System.out.println(allOver25);

    // 원소 중 20대가 있는지(any)
    boolean any20s = people.stream().anyMatch(p -> p.getAge() < 30);
    System.out.println(any20s);

    // 30대인 사람은 몇명인지(count)
    long countOver29 = people.stream().filter(p -> p.getAge() > 29).count();
    System.out.println(countOver29);

    // 30대 중 가장 첫 번째 원소(findFirst)
    Person firstOver29 = people.stream().filter(p -> p.getAge() > 29).findFirst().orElse(null);
    if (firstOver29 != null) {
        System.out.println(firstOver29.getName());
    } else {
        System.out.println("No one found");
    }
}
```
코드에서 볼 수 있듯이 모든 함수는 `stream()`을 이용하여 처리하고, count와 find의 경우는 filter()를 통해 사전 분리를 수행한 후 함수를 한번 더 호출한다.

#### 5.2.3 groupBy + 5.2.4 flatMap과 flatten
각 함수의 역할은 다음과 같다.

- groupBy : 컬렉션의 원소 그룹에 따라 여러 컬렉션으로 분할
- flatMap : 중첩된 컬렉션에서 한 리스트로 모으는 역할로 변환이 필요한 경우 사용
- flatten : flatMap과 유사하게 동작하며 변환이 필요하지 않은 경우 사용

코드를 살펴보자.
```kotlin
fun groupByAndOthers () {
    val people = listOf(Person("Alice", 27), Person("Jay", 32), Person("H", 33))
    val strings = listOf("test", "cola")

    println(people.groupBy { it.age })
    println(strings.flatMap { it.toList() })
}

>> {27=[Person(name=Alice, age=27)], 32=[Person(name=Jay, age=32)], 33=[Person(name=H, age=33)]}
>> [t, e, s, t, c, o, l, a]
```

자바로 변환된 코드를 확인해보자.
```java
public static void main(String[] args) {
    List<Person> people = Arrays.asList(
        new Person("Alice", 27),
        new Person("Jay", 32),
        new Person("H", 33)
    );
    List<String> strings = Arrays.asList("test", "cola");

    // groupBy { it.age }
    Map<Integer, List<Person>> groupedByAge = people.stream()
        .collect(Collectors.groupingBy(Person::getAge));
    System.out.println(groupedByAge);

    // flatMap { it.toList() } - 문자열을 문자 리스트로 평탄화
    // toList()와 유사한 동작: 문자열을 char 스트림으로 변환 후 Collect
    List<Character> flattenedChars = strings.stream()
        .flatMap(s -> s.chars().mapToObj(c -> (char) c))
        .collect(Collectors.toList());
    System.out.println(flattenedChars);
}
```

위에서부터 코드를 보면 항상 반복되는 코드가 있는데 바로 최종 연산을 수행하는 `Colletors.xxx()`이다.
스트림을 사용하면 최종 연산이 수행될 때 모든 계산이 이루어지기 때문에 최종 연산을 사용하지 않으면 에러가 발생한다.
이 부분에서 코틀린과 자바의 코드 가독성과 길이, 중복 등의 차이가 발생하는 것 같다.

컬렉션을 다룰 때에는 위 함수뿐만 아니라 그 때 필요한 부분을 찾아보고 적용할 필요가 있다.
구현을 하는 시간보다 함수를 찾아서 적용하는 시간이 더 적게 들기 때문이다.

---

### 5.3 지연 계산(Lazy) 컬렉션 연산
>시퀀스를 사용하여 중간 임시 컬렉션을 사용하지 않고 컬렉션 연산을 연쇄하는 방법에 대해 알아보자.

컬렉션을 사용하여 연쇄적으로 함수를 호출하면 함수마다 새로운 컬렉션이 임시 결과를 저장한다.
수행하는 횟수가 많아진다면 임시 결과를 저장할 새로운 컬렉션이 계속 필요할 것이다.

이 부분을 해결하기 위해 사용할 수 있는 것이 `Sequence`이다. 자바에서는 `Stream`과 동일하다.
다만, 자바의 `Stream`은 병렬처리가 가능하다는 장점이 있다.

먼저 아래 코드로 문법을 보자.

```kotlin
people.asSequence() // 원본 컬렉션 시퀀스로 변환
  .map(Person::name)
  .filter{it.startWith("A")}
  .toLsit() // 최종 연산
```

위 코드를 자바로 바꾸면 아래와 같다.
```java
people.stream()
  .map(Person::getName)
  .filter(person -> person.name.startWith("A"))
  .collect(Collectors.toList());
```

코틀린에서는 큰 컬렉션에 대한 연산은 `Sequence`를 사용하는 것을 규칙으로 삼으라고 권하고 있다.
중간 연산을 최소화하고 원소가 실제로 필요할 때 비로소 계산되기 때문이다.

#### 5.3.1 시퀀스 연산 실행 : 중간 연산과 최종 연산
시퀀스에는 중간 연산과 최종 연산이 있다.

중간 연산은 다른 시퀀스를 반환하며 최초 시퀀스의 원소를 변환하는 방법을 알고 있다.
최종 연산은 결과를 반환하며 결과는 중간 연산을 통해 얻은 정보를 통해 반환한다.

중요한 점은 중간 연산은 항상 **지연 계산**된다. 즉 최종 연산이 호출되어 원소가 쓰임이 생겼을 때 계산이 발생한다.
최종 연산을 수행하면 연기됐던 모든 계산이 수행된다.

또한, 컬렉션과 시퀀스의 연산은 다음 그림과 같은 차이가 있다.
- 컬렉션 : map() 완료 -> filter () 수행
- 시퀀스 : map()과 filter()를 원소 마다 적용하여 수행

<img width="400" alt="스크린샷 2024-12-12 오후 6 25 37" src="https://github.com/user-attachments/assets/fdb9f7da-5e63-40fd-b7aa-a06759003e65" />

이렇게 수행하면 조건에 해당하는 원소가 식별되었을 때 연산을 종료하여 추가적인 연산을 수행하지 않아도 된다.

#### 5.3.2 시퀀스 만들기
asSquence()를 사용하지 않고 시퀀스를 만드는 방법은 다음과 같다.

```kotlin
val naturalNumbers = generateSequence(0) { it + 1 }
val numbersTo100 = naturalNumbers.takeWhile { it <= 100 }

// 여기서 최종 연산은 sum()
println(numbersTo100.sum())
>> 5050
```

---

### 5.4 자바 함수형 인터페이스 활용 : 상단 자바 설명으로 대체

---

### 5.5 수신 객체 지정 람다: with와 apply
수신 객체를 명시하지 않고 람다의 본문 안에서 다른 객체의 메서드를 호출할 수 있게 하는 기능이다.

#### 5.5.1 with 함수
어떤 객체의 이름을 반복하지 않고 다양한 연산을 수행하는 기능을 제공하는 라이브러리 함수이다.

```kotlin
// with 미사용
fun alphabet(): String {
  val result = StringBuilder()

  for (letter in 'A'..'Z') {
    result.append(letter)
  }

  result.append("\nNow I know the alphabet!)
  return result.toString()
}

// with 사용
fun alphabet(): String {
    val stringBuilder = StringBuilder()

    return with(stringBuilder) { // 객체 지정
        // this가 모두 생략되어도 동작함
        for (letter in 'A'..'Z') {
            append(letter)
        }
        append("test")
        toString() // 반환
    }
}
```
with은 다음과 같은 규칙이 있다.

- 첫 번째 인자로 받은 객체를 두 번째 인자로 받은 람다의 수신 객체로 만듦
- 람다 본문에서 this를 이용해 수신 객체에 접근 가능
- 프로퍼티나 메서드 이름만으로도 접근 가능

|일반 함수|일반 람다|
|---|---|
|확장함수|수신 객체 지정 람다|

만약, with의 수신 객체로 지정한 객체와 with을 사용하는 클래스 안에 이름이 동일한 메서드가 있는 경우 다음과 같이 레이블을 붙이고 호출한다.
```kotlin
this@OuterClass.toString()
```

#### 5.5.2 apply 함수
with과 거의 유사하지만 수신 객체를 반환한다는 차이가 있으며 확장 함수로 정의되어 있다.

```kotlin
fun alphabet () = StringBuilder().apply {
  for(letter in 'A'..'Z') {
    append(letter)
  }

  append("\n Good")
}.toString() // 결과 반환
```

apply는 인스턴스를 생성하면서 즉시 프로퍼티 중 일부를 초기화 할 때 유용하게 사용이 가능하다.
```kotlin
fun createView(context: Context) =
  TextView(context).apply { // 함수의 파라미터를 생성자 인자로 전달
    text = "sample text"
    textSize = 20.0
    setPadding(10, 0, 0, 0)
  }
```

참고로 자바는 람다에서 this를 사용하는 경우 람다를 호출한 외부 클래스의 인스턴스를 가리킨다.
즉, 람다를 포함하는 객체에 대한 인스턴스를 가리키지 않는다는 것이 with과 apply와의 차이다.

---

### 정리
람다에 대한 기본적인 개념부터 코틀린과 람다의 차이 관점에서 코틀린에서의 람다 프로그래밍을 알아보았다.
아직 미숙한 개념들이 많지만 확실한 건 자바에서 사용할 때 발생하던 보일러 플레이트 코드들은 굉장히 많이 줄어든 것 같다는 느낌을 받았다.
