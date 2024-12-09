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
