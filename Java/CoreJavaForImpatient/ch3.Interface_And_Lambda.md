<h1>3. 인터파에스와 람다표현식</h1>

<h2>3.1.3 타입 변환과 instance</h2>

<pre>
interface로 선언하고 impl class로 생성했을 때,
객체의 타입을 interface로 변환하고 싶을 때, instanceof를 사용하자.
</pre>

```java
IntSequence sequence = new DigitSequence(1729);
if(sequence instanceof DigitSequence) {
  DigitSequence digitSequence = (DigitSequence) sequence;
}
```

<h2>3.1.5 인터페이스 확장하기</h2>

<pre>
interface는 또 다른 interface를 확장해서 추가 메서드를 제공할 수 있다.
Closeable 이란 interface의 close는 예외가 일어날 때 리소스를 닫는 메서드이다.
</pre>

```java
public interface Closeable {
  void close();
}
public interface Channel extends Closeable {
  boolean isOpen();
}
```

<h2>3.1.7 상수</h2>
<pre>
interface에서 변수는 자동으로 public static final 이다.
</pre>

```java
public interface SwingConstants {
  int NORTH = 1 // public static final int
  int EAST = 3;
}
```

<h2>3.2.1 정적 메서드</h2>
<pre>
자바8에서는 interface에 정적 메서드가 포함될 수 있다.
특히 팩토리 메서드는 인터페이스와 아주 잘맞는다.
</pre>

```java
public interface IntSequence {
  public static DigitSequence digitsOf(int n) {
    return new DigitSequence(n);
  }
}
```

<h2>3.2.2 기본 메서드</h2>
<pre>
물론 기본 메서드를 구현할 수 도 있다.
이 때는 반드시 default 제어자를 붙여야 한다.
</pre>

```java
public interface IntSequence {
  // 기본 메서드
  default boolean hasNext() { return true; }
  // 추상 메서드
  int next();
}
...
//DigitSequence digit = new DigitSequence();
IntSequence digit = new DigitSequence();
System.out.println(digit.hasNext());
```

<h2>3.2.2 기본 메서드의 충돌 해결하기</h2>
<pre>
2개의 interface에서 같은 이름의 default 메서드가 구현돼있다면
이 2개를 상속받는 class는 어떤 메서드를 사용할까?
아래처럼 명시해주어야 한다.
</pre>

```java
public interface Person {
  default int getId() {return 0;}
}
public interface Identified {
  default int getId() {return Math.abs(hashCode());}
}
public class Employee implements Person, Identified {
  public int getId() {
    return Identified.super.getId();
  }
}
```

<h2>3.3 인터페이스의 예</h2>

<h2>3.3.1 Comparable 인터페이스</h2>
