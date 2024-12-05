## 4장. 클래스, 객체, 인터페이스
---
### 4.1 클래스 계층 정의
  
#### 4.1.1 코틀린 인터페이스
#### 특징
- 추상 메서드 뿐만 아니라 구현이 있는 메서드 정의 가능
- 자바 8 과 유사한 인터페이스 형태
- 아무런 상태도 가질 수 없음

##### 선언
**Kotlin**
```kotlin
interface Clickable {
    fun click()
}

class Button : Clickable {
    // override fun click() {
    //        println("click")
    // }
    override fun click() = println("click");
}
```
`:`을 이용하여 구현과 확장을 모두 처리한다. 자바와 마찬가지로 클래스는 단 한개만 상속가능하고, 인터페이스는 복수 개로 구현이 가능하다.
또한, 자바에서는 생략 가능한 `override` 키워드를 반드시 사용하여야 한다. 이유는 우연히 시그니처가 같은 메서드를 선언하면 구분을 위해 반드시 붙여야한다.

**Decompile**
```java
public interface Clickable {
   void click();
}

// final로 생성
public final class Button implements Clickable {
   public void click() {
      String var1 = "click";
      System.out.println(var1);
   }
}
```

##### 동일 시그니처 메소드
**Kotlin**
```kotlin
interface Clickable {
    fun click()
    // default 메서드 선언, default 키워드 불필요
    fun showOff() = println("I'm clickable!")
}

interface Focusable {
    fun setFocus(b: Boolean) = println("i ${if (b) "got" else "lost"} focus.")
    fun showOff() = println("I'm focusable!")
}

class Button : Clickable, Focusable {
    override fun click() = println("I was clicked.")

    override fun showOff() {
        // 동일 시그니처 메소드에 대해 하위 클래스에서 구현 강제
        super<Clickable>.showOff();
        super<Focusable>.showOff();
    }
}

fun main(args: Array<String>) {
    // 인스턴스 생성
    val button = Button()
    button.showOff()
    button.setFocus(true)
    button.click()
}
```
두 상위 객체에 동일한 메서드가 존재한다면 그것을 구현하는 하위 객체는 반드시 해당 메서드를 직접 구현하고, `super` 키워드를 이용하여 명시해야한다.

**Decompile**
```java
public interface Clickable {
   void click();

   void showOff();
   // 디폴트 메소드를 구현하는 클래스
   public static final class DefaultImpls {
      public static void showOff(@NotNull Clickable $this) {
         String var1 = "I'm clickable!";
         System.out.println(var1);
      }
   }
}

public final class Button implements Clickable, Focusable {
   public void click() {
      String var1 = "I was clicked.";
      System.out.println(var1);
   }

   // 상위 클래스의 default 메소드들을 구현
   public void showOff() {
      Clickable.DefaultImpls.showOff(this);
      Focusable.DefaultImpls.showOff(this);
   }

   public void setFocus(boolean b) {
      Focusable.DefaultImpls.setFocus(this, b);
   }
}
```

#### 4.1.2 open, final, abstract 변경자
>코틀린에서 기본적으로 선언되는 변경자는 final과 public이다.

코틀린은 취약한 기반 클래스라는 문제를 조금이라도 해결하기 위해 `Effective Java`에서 언급한 것처럼 오버라이드하게 의도된 클래스와 메서드가 아니라면 모두 final로 만들어야 한다는 철학을 이어간다.
이런 이유로 코틀린의 클래스와 메서드는 기본적으로 `final`이다.

##### 상속 허용
**Kotlin**
```kotlin
open class RichButton: Clickable {
    // 기본 open
    open fun animate(){}
    // 오버라이드 불가
    final fun disable() {}

    override fun click() {}
}
```

**Decompile**
```java
public class RichButton implements Clickable {
   public void animate() {
   }
   // final로 인해 오버라이드 불가
   public final void disable() {
   }

   public void click() {
   }

   public void showOff() {
      Clickable.DefaultImpls.showOff(this);
   }
}
```

##### 클래스 내의 변경자
|변경자|조건|설명|
|---|---|---|
|final|오버라이드 x|클래스 멤버의 기본 변경자|
|open|오버라이드 o|반드시 open을 명시해야 오버라이드 가능|
|abstract|반드시 오버라이드 o|추상 클래스의 멤버에만 이 변경자 사용 가능|
|override|오버라이드 적용|오버라이드하는 멤버는 기본적으로 열려있음|

#### 4.1.3 가시성 변경자
>기본적으로 public으로 설정되어 있으며, 여기서 설명하는 모듈은 한꺼번에 컴파일되는 코틀린 파일을 의미한다.

또한, 코틀린에는 패키지 전용 가시성이 존재하지 않는다. 이유는 패키지를 네임스페이스 관리 용도로만 사용하기 때문이다.
대신 `internal`이라는 새로운 가시성을 도입했다.
- internal은 모듈 내부에서만 볼 수 있음
- 모듈은 한번에 한꺼번에 컴파일되는 코틀린 파일을 의미
- 인텔리J, 이클립스, 메이븐, 그레이르 등이 모듈
- 모듈의 구현에 대해 진정한 캡슐화를 제공한다는 장점 보유

추가로 코틀린은 `private` 가시성이 최상위 선언에 대해 허용한다. 최상위 선언은 그 선언이 들어있는 파일 내부에서만 사용할 수 있다.

|변경자|클래스 멤버|최상위 선언|
|---|---|---|
|public|모든 곳에서 접근|모든 곳에서 접근 가능|
|internal|같은 모듈 안에서만 접근 가능|같은 모듈 안에서만 접근 가능|
|protected|하위 클래스 안에서만 접근 가능|최상위 선언에 적용할 수 없다|
|private|같은 클래스 안에서만 접근 가능|같은 파일 안에서만 접근 가능|

코틀린의 `protected`는 오직 어떤 클래스나 그 클래스를 상속한 클래스 안에서만 보인다는 점을 유의해야한다.

#### 4.1.4 내부 클래스와 중첩된 클래스
>기본적으로 중첩된 클래스로 동작하며, inner class를 명시적으로 요청해야 외부 클래스에 접근이 가능해진다.

**Kotlin**
```kotlin
class Outer {
    class Nested {
        // 이 상태에서는 Outer 접근 불가
    }
}

// inner 키워드와 this@Outer를 사용하여 접근
class Outer {
    inner class Nested {
        fun getOuterReference() : Outer = this@Outer
    }
}
```
코틀린의 중첩 클래스에 아무런 변경자가 붙지 않으면 자바 `static` 중첩 클래스와 동일하다.
이를 내부 클래스로 변경하고 싶다면 `inner` 키워드를 붙여주어야한다.

**Decompile**
```java
// 중첩 클래스
public final class Outer {
   // static 키워드가 사용됨
   public static final class Nested {
   }
}

// inner 키워드를 사용한 내부 클래스
public final class Outer {
   public final class Nested {
      @NotNull
      public final Outer getOuterReference() {
         return Outer.this;
      }
   }
}
```

#### 4.1.5 봉인된 클래스
>sealed 키워드를 사용하며 클래스 계층 정의 시 계층 확장을 제한하고, 클래스 계층 구조에서 제한된 개수의 클래스를 나타낼 때 사용한다.

**Kotlin**
```kotlin
sealed class Expr {
    class Num(val value:Int) : Expr()
    class Sum(val left: Expr, val right: Expr) : Expr()
}

// else 생략 가능
fun eval(e: Expr) : Int =
    when(e) {
        is Expr.Num -> e.value
        is Expr.Sum -> eval(e.right) + eval(e.left)
    }

fun main(args: Array<String>) {
    // (1 + 2) + 4 = 7
    println(eval(Expr.Sum(Expr.Sum(Expr.Num(1), Expr.Num(2)), Expr.Num(4))))
}
```
`when`은 `sealed` 키워드가 붙은 봉인된 클래스에 대한 하위 클래스를 모두 검사하게되어 else를 지워도 정상적으로 동작한다.
또한, `sealed`로 표시된 클래스는 자동으로 `open`이 된다.

**Decompile**
```java
// sealed class 자체는 항상 추상 클래스
public abstract class Expr {
   private Expr() {
   }

   // $FF: synthetic method
   public Expr(DefaultConstructorMarker $constructor_marker) {
      this();
   }
  
   public static final class Num extends Expr {
      private final int value;

      public final int getValue() {
         return this.value;
      }

      public Num(int value) {
         super((DefaultConstructorMarker)null);
         this.value = value;
      }
   }

   public static final class Sum extends Expr {
      @NotNull
      private final Expr left;
      @NotNull
      private final Expr right;

      @NotNull
      public final Expr getLeft() {
         return this.left;
      }

      @NotNull
      public final Expr getRight() {
         return this.right;
      }

      public Sum(@NotNull Expr left, @NotNull Expr right) {
         Intrinsics.checkNotNullParameter(left, "left");
         Intrinsics.checkNotNullParameter(right, "right");
         super((DefaultConstructorMarker)null);
         this.left = left;
         this.right = right;
      }
   }
}
```
<a href='https://kotlinlang.org/docs/sealed-classes.html#declare-a-sealed-class-or-interface'>공식 문서</a>에 따르면 현재 코틀린의 문법에서 봉인된 클래스나 인터페이스는 반드시 중첩 클래스만 사용하는 것이 아니라 외부에 선언이 가능하다.

정확히는 봉인된 클래스가 **정의된 모듈과 패키지 외부**에 다른 하위 클래스가 나타날 수 없다.

**예시**
```kotlin
// Create a sealed interface
sealed interface Error

// Create a sealed class that implements sealed interface Error
sealed class IOError(): Error

// Define subclasses that extend sealed class 'IOError'
class FileReadError(val file: File): IOError()
class DatabaseError(val source: DataSource): IOError()

// Create a singleton object implementing the 'Error' sealed interface
object RuntimeError : Error
```
예시의 계층구조는 다음과 같다.

![image](https://github.com/user-attachments/assets/1cef9b74-0212-468c-b61a-34e03a60d31c)

---
### 4.2 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언

#### 주 생성자
>클래스를 초기화할 때 주로 사용하는 간략한 생성자로, 클래스 본문 밖에서 정의

주 생성자 앞에 별다른 어노테잇ㄴ이나 가시성 변경자가 없다면 `constructor`를 생략할 수 있다.

**Kotlin**
```kotlin
// 주 생성자
open class Box()
open class NewBox // 괄호 생략 가능

// 주 생성자 간략히 표현
class User(val name: String)
class Person(val name: String, val id: String = "기본") // 기본 값 추가
```

주 생성자에 전달되는 파라미터 앞에 `val` 키워드를 사용하여 정의와 초기화를 한번에 처리할 수 있다.

**Decompile**
```java
// 생성자 생략(자동 생성)
public class Box {
}

// 생성자 추가 및 기본 값
public final class Person {
   @NotNull
   private final String name;
   @NotNull
   private final String id;

   @NotNull
   public final String getName() {
      return this.name;
   }

   @NotNull
   public final String getId() {
      return this.id;
   }

   // 생성자 추가
   public Person(@NotNull String name, @NotNull String id) {
      Intrinsics.checkNotNullParameter(name, "name");
      Intrinsics.checkNotNullParameter(id, "id");
      super();
      this.name = name;
      this.id = id;
   }

   // 기본 값을 포함한 생성자 추가
   // $FF: synthetic method
   public Person(String var1, String var2, int var3, DefaultConstructorMarker var4) {
      if ((var3 & 2) != 0) {
         var2 = "기본";
      }

      this(var1, var2);
   }
}
```
추가로 `constructor` 앞에 `private` 키워드를 붙이면 생성자에 접근할 수 없는데 이 부분을 적용할 수 있는 경우는 다음과 같다.
- 유틸리티 함수를 담아두는 역할만 하는 클래스는 인스턴스를 생성할 필요가 없다.
- 싱글턴인 클래스의 경우 미리 정한 팩토리 메서드 등을 활용해 인스턴스를 전달받으므로 생성자가 필요없다.

#### 부 생성자
>default parameter가 있기 때문에 잘 쓰이는 경우는 없지만 자바와 유사하게 `this` 또는 `super` 등을 활용 가능하며 클래스 본문 안에서 정의

부 생성자가 필요한 주된 이유는 자바와의 상호운용성 때문이기도 하지만 인스턴스를 생성할 때 파라미터 목록이 다른 생성 방법이 여럿 존재하는 경우에는 부 생성자를 여러개 두는 것이다.
하지만 이 부분도 주 생성자에 초기 값을 부여하는 방식으로 대체할 수 있다.

**Kotlin**
```kotlin
class MyButton : View {
    // 이렇게 상속하듯이 콜론(:) 을 이용한다.
    constructor(context: Context) : super(ctx) {
        // ...
    }

    constructor(context: Context, attr: AttributeSet) : super(ctx, attr) {
        // ...
    }

    constructor(context: Context, _name: String) : this(ctx) {// 다른 생성자를 재사용
        name = _name
    }
}
```

#### 인터페이스에 선언된 프로퍼티 구현
>인터페이스에 추상 프로퍼티 선언을 넣을 수 있다.

이 말은 해당 인터페이스를 구현하는 클래스가 프로퍼티 값을 얻을 수 있는 방법을 구현해야한다는 의미이다.

```kotlin
// 추상 프로퍼티 선언
interface User {
  val nickname: String
}
```

위 인터페이스의 추상 프로퍼티를 구현하는 방법은 아래와 같다.
```kotlin
// 주 생성자에 있는 프로퍼티
class PrivateUser (override val nickname: String) : User

// 커스텀 게터 사용
class SubscribingUser(val email: String) : User {
    override val nickname: String
        get() = email.substringBefore('@')
}

// 프로퍼티 초기화 식
class FacebookUser (val accountId: Int) : User {
    override val nickname =  getFacebookName(accountId)
}
```

#### 게터와 세터에서 뒷받침하는 필드에 접근
>어떤 값을 저장할 때 그 값을 변경하거나 읽을 때마다 정해진 로직을 실행하는 유형의 프로퍼티를 만드는 방법을 알아본다.

```kotlin
// 세터에서 값이 들어오면 뒷받침하는 필드 접근하기
class TestUser (val name: String) {
    var address: String = "test"
        set (value: String) {
            println("""
                Address was changed for $name
            """.trimIndent())
            field = value
        }
}
```
