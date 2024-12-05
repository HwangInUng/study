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
