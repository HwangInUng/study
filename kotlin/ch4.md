## 4장. 클래스, 객체, 인터페이스
---
### 4.1 클래스 계층 정의
  
#### 4.1.1 코틀린 인터페이스

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

|변경자|클래스 멤버|최상위 선언|
|---|---|---|
|public|모든 곳에서 접근|모든 곳에서 접근 가능|
|internal|같은 모듈 안에서만 접근 가능|같은 모듈 안에서만 접근 가능|
|protected|하위 클래스 안에서만 접근 가능|최상위 선언에 적용할 수 없다|
|private|같은 클래스 안에서만 접근 가능|같은 파일 안에서만 접근 가능|

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
>생성자가 없다면 기본 생성자를 자동으로 추가해준다.

**Kotlin**
```kotlin
open class Box()
open class NewBox // 괄호 생략 가능

// 생성자
class User(val name: String)
class Person(val name: String, val id: String = "기본") // 기본 값 추가
```

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

#### 부 생성자
>default parameter가 있기 때문에 잘 쓰이는 경우는 없지만 자바와 유사하게 `this` 또는 `super` 등을 활용 가능하다.

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
