<h1>4. 상속과 리플렉션</h1>

<h2>4.1 클래스 확장하기</h2>
<h2>4.1.3 메서드 오버라이딩</h2>

<pre>
서브클래스는 슈퍼클래스의 비공개 인스턴스 변수에 접근할 수 없다.
</pre>

<h2>4.1.7 최종 메서드와 최종 클래스</h2>

<pre>
슈퍼클래스의 메서드를 final로 선언하면 서브클래스는 해당 메서드를 오버라이드 할 수 없다.
getClass는 반드시 final로 선언해야하는 메서드다.(서브클래스가 class를 변경할 수 있으면 안된다.)
</pre>

<h2>4.1.10 익명 서브클래스</h2>

```java
ArrayList<String> names = new ArrayList<String>(100) {
  public void add(int index, String element) {
    super.add(index, element);
    System.out.println("Adding %s at %d\n", element, index);
  }
};
```

<h2>4.1.12 super를 이용한 메서드 표현식</h2>

<pre>
람다표현식에서 봤듯이 객체::인스턴스메서드 형태를 super에서도 할 수 있다.
super::인스턴스메서드
</pre>

```java
public class Main {
    public static void main(String[] args) {
        ConcurrentWorker w = new ConcurrentWorker();
        w.work();
    }
}

class Worker {
    public void work() {
        for(int i=0; i<100; i++) System.out.println("Working");
    }
}

class ConcurrentWorker extends Worker {
    public void work() {
        Thread t = new Thread(super::work);
        t.start();
    }
}
```

<pre>
super::work는 Runnable의 run 메서드와 동일한 역할을 한다.
</pre>

<h2>4.2 Object: 보편적 슈퍼클래스</h2>

```java
String toString() // 객체 문자열 표현
boolean equals(Object other) // 비교
int hashCode() // 객체에 해당하는 해시 코드
Class<?> getClass() // 객체가 속한 클래스를 기술한 Class 객체
protected Object clone() // 객체의 사본(얕은 복사)
protected void finalize() // 가비지 컬렉터가 객체를 회수할 때 호출하는 메서드, 오버라이드 x
wait, notify, notifyAll // 병행 프로그래밍 참조
```

<h2>4.2.1 toString 메서드</h2>
<pre>
배열의 원소를 출력하고 싶다면
</pre>

```java
String[] s = new String[]{"bob", "json"};
System.out.println(Arrays.deepToString(s));
```

<h2>4.2.2 equals 메서드</h2>
<pre>
equals 메서드를 오버라이드 할 때는 이와 호환되는 hashCode 메서드도 반드시 같이 제공해야 한다.
</pre>

```java
boolean euqals(Object other) {
    if(this == other) return true; // 객체 동일 여부 검사, 빠른 검사

    if(other == null) return false; // other이 null 이면

    if(getClass() != other.getClass()) return false; // 클래스 비교

    Item o = (Item) other
    return Objects.equals(desc, o.desc) && price == o.price; // 인스턴스 변수 값 비교
}
```

<pre>
instanceof로 검사하면 안된다.
Item x
OtherItem extends Item y
상속관계에서 instanceof가 통과되고 변수값 비교에서 서로 다른 비교를 하게 된다.
x.equals(y)와 y.equals(x)는 다른 비교다.
</pre>

<h2>4.2.3 hashCode 메서드</h2>
<pre>
해시 코드는 객체에서 파생되는 정숫값. 중복될 수 있다. 가능한 중복을 피해서 만들어야한다.
hashCode와 equals 메서드는 반드시 호환
</pre>

<h2>4.2.4 객체 복제하기</h2>
<pre>
clone은 많이 사용하지 않는다.
기본적으로 얕은 복사
protected로 선언되어있기 때문에 사용하려면 반드시 clone을 오버라이딩 해야한다.
얕은 복사는 객체에 있는 모든 인스턴스 변수를 복제된 객체로 단순 복사
</pre>

<pre>
clone 구현 판단
1. clone 메서드를 구현하지 않아도 되는가?
2. 구현해야 한다면 상속받은 clone 메서드를 사용해도 괜찮은가?
--> Cloneable 인터페이스를 구현해야 한다. 안하면 CloneNotSupportedException 발생
유효범위를 projected -> public, 반환 타입 변경
3. 그렇지 않으면 clone 메서드에서 깊은 복사를 수행해야 하는가?
</pre>

<pre>
얕은 복사
</pre>

```java
public class Employee implements Cloneable {
    public Employee clone() throws CloneNotSupportedException {
        return (Employee) super.clone();
    }
}
```

<pre>
간단한 깊은 복사
인스턴스 변수도 clone을 사용할 경우,
ArrayList.clone은 얕은복사로 값을 참조하기만 해서 원소의 값이 바뀌면 복사본도 같이 바뀐다.
또한 반환타입이 Object이기 때문에 타입변환을 할 경우 경고가 발생한다.
</pre>

```java
public Message clone() {
    Message cloned = new Message(sender, text);
    cloned.recipients = new ArrayList<>(recipients);
    return cloned;
}
```

```java
public Message clone() {
    try {
        Message cloned = (Message) super.clone();
        // 인스턴스 변수 복제
        @SuppressWarning("unchecked") ArrayList<String> clonedRecipients
            = (ArrayList<String>) recipients.clone();
        cloned.recipients = clonedRecipients;
        return cloned;
    } catch(CloneNotSupportedException ex) {
        return null;
    }
}
```

<pre>
여기서 Message 클래스가 Cloneable이고 final 이다(super.clone()).
</pre>

<h2>4.3 열거</h2>
<h2>열거의 메서드</h2>

```java
public enum Size { SMALL, MEDIUM, LARGE, EXTRA_LARGE };
```

<pre>
enum은 equals를 사용하지 않는다. 그냥 == 으로 비교해라.
toString을 구현하지 않아도 된다. 자동으로 주어진다. ("SMALL", "MEDIUM"..)
toString의 역은 valueOf 이다.
</pre>

```java
Size notMySize = Size.valueOf("SMALL");
```

<pre>
주어진 이름에 해당하는 인스턴스가 없으면 예외를 던진다.
</pre>

```java
for(Size s : Size.values()) {
  System.out.println(s);
}
```

<pre>
values로 모든 인스턴스를 순회할 수 있다.
</pre>

<h2>4.3.2 생성자, 메서드, 필드</h2>

```java
public enum Size {
  SMALL("S"), MEDIUM("M"), LARGE("L"), EXTRA_LARGE("XL");

  private String abbreviation;

  Size(String abbreviation) {
    this.abbreviation = abbreviation;
  }

  public String getAbbreviation() {
    return abbreviation;
  }
}
```

<pre>
enum의 생성자는 반드시 private (비우면 private)
</pre>

<h2>4.3.3 인스턴스의 구현부</h2>
<pre>
enum 인스턴스의 각각에 메서드를 추가할 수 있다.
enum에 정의된 메서드를 오버라이드 해야 한다.
</pre>

```java
public enum Operation {
  ADD {
    public int eval(int arg1, int arg2) { return arg1 + arg2; }
  },
  SUBTRACT {
    public int eval(int arg1, int arg2) { return arg1 - arg2; }
  },
  MULTIPLY {
    public int eval(int arg1, int arg2) { return arg1 * arg2; }
  },
  DIVIDE {
    public int eval(int arg1, int arg2) { return arg1 / arg2; }
  };

  public abstract int eval(int arg1, int arg2);
}

Operation op = Operation.valueOf("ADD");
int result = op.eval(first, second);
```

<h2>4.3.3 정적 멤버</h2>
<pre>
enum은 정적 멤버(static)를 가질 수 있다.
하지만 생성 순서가 enum 상수 -> 정적 멤버가 순서다.
따라서 enum 생성자에서는 정적 멤버를 참조할 수 없다.
</pre>

```java
enum Modifier {
  PUBLIC, PRIVATE, PROTECTED, STATIC, FINAL, ABSTRACT;
  private static int maskBit = 1;
  private int mask;
  public Modifier() {
    mask = maskBit; // 에러!!
    maskBit *= 2;
  }
}
```

<pre>
해결 방안은 정적 초기화 블록에서 초기화하는 것이다.
</pre>

```java
enum Modifier {
  PUBLIC, PRIVATE, PROTECTED, STATIC, FINAL, ABSTRACT;
  private int mask;
  static {
    int maskBit = 1;
    for(Modifier m : Modifier.values()) {
      m.mask = maskBit;
      maskBit *= 2;
    }
  }
  public Modifier() {
    mask = maskBit; // 에러!!
    maskBit *= 2;
  }
}
```

<h2>4.4 실행 시간 타입 정보와 리소스</h2>
