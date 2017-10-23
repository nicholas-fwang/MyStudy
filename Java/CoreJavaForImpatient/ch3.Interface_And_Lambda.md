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
```java
class test {
    public static void main(String[] args) {
        Employee e1 = new Employee(2.0);
        Employee e2 = new Employee(1.0);
        System.out.println(e1.compareTo(e2));
    }
}

interface Comparable<T> {
    int compareTo(T other);
}

class Employee implements Comparable<Employee> {
    private double salary;

    public Employee(double salary) {
        this.salary = salary;
    }

    @Override
    public int compareTo(Employee other) {
        return Double.compare(salary, other.salary);
    }
}
```

<pre>
어떤 객체가 올지 모르기 때문에 T(타입 파라미터)를 제네릭 파라미터로 사용한다.
구현 class(Employee)에서 compareTo 메서드를 구현한다.
private 변수인 salary가 접근 가능하다.
제네릭 부분에서 다시 봐바야 할듯
</pre>

<h2>3.3.2 Comparator 인터페이스</h2>
<pre>
위 Comparable을 사용해서 sort를 할 수 있다.
Employee[] employees
Arrays.sort(employees)
가 가능하다.
하지만 Employee가 아닌 String 같이 수정할 수 없는 class라면 다른 방법을 사용해야 한다.
String의 길이에 따라 정렬을 한다고 생각해보자.
</pre>

```java
import java.util.Arrays;
import java.util.Comparator;

public class Main {
    public static void main(String[] args) {
        String[] friends = { "Peter", "Paul", "Mary" };
        Arrays.sort(friends, new LengthComparator());
        for(String friend : friends) {
            System.out.println(friend);
        }
    }
}

class LengthComparator implements Comparator<String> {
    @Override
    public int compare(String o1, String o2) {
        return o1.length() - o2.length();
    }
}
```
<h2>3.3.3 Runnable 인터페이즈</h2>
<pre>
특정 테스크를 별도의 스레드에서 실행하거나 스레드 풀에 넣기 위해 구현해야하는 인터페이스.
메서드는 한 개만 갖는다.
</pre>

```java
import java.util.Arrays;
import java.util.Comparator;

public class Main {
    public static void main(String[] args) {
        Runnable task = new HelloTask();
        Thread thread = new Thread(task);
        thread.start();
    }
}

class HelloTask implements Runnable {
    @Override
    public void run() {
        for(int i=0; i<1000; i++) {
            System.out.println("Hello, World");
        }
    }
}
```
<h2>3.3.4 사용자 인터페이스 콜백</h2>
<pre>
이벤트가 발생했을 때 수행되어야 할 액션을 지정한다.
Handler라는 이름으로 많이 사용된다.
T는 이벤트가 발생했을 때 전달될 이벤트 타입이다.
</pre>

```java

public class Main {
    public static void main(String[] args) {
        Button btn = new Button("Cancel");
        btn.setOnAction(new CancelAction());
    }
}

interface EventHandler<T> {
    void handle(T event);
}

class CancelAction implements EventHandler<ActionEvent> {
    public void handle(ActionEvent event) {
        System.out.println(event.getID());
    }
}
```

<h2>3.4 람다 표현식</h2>
<h2>3.4.1 람다 표현식 문법</h2>
<pre>
자바는 객체 지향 언어다.
자바에는 함수 타입이 없다.
그래서 함수를 객체로 표현한다.
</pre>

```java
//        Arrays.sort(friends, new LengthComparator());
        Comparator<String> comp
                = (first, second) -> first.length() - second.length();
        Arrays.sort(friends, comp);
```

<pre>
Comparator interface에서 구현돼야할 메서드는 compare 하나 뿐이었다.
compare 메서드는 파라미터가 2개고 타입은 String으로 제네릭에서 지정이 되어있다.
따라서 compare 함수가 객체로 표현됐다.
</pre>

```java
//        Runnable task = new HelloTask();
        Runnable task = () -> {
            for(int i=0; i<1000; i++) {
                System.out.println("Hello World");
            }
        };
```

<pre>
Runnable interface 역시 run 메서드 하나 뿐이다.
run 메서드는 파라미터가 없기 때문에 ()가 비어있다.
</pre>

<h2>함수형 인터페이스</h2>
<pre>
Comparator, Runnable 처럼 추상 메서드가 한 개인 인터페이스를 함수형 인터페이스라고 부른다.
람다는 함수형 인터페이스일 경우에만 사용 가능하다.(함수가 객체로 표현돼야 하기 때문에)
</pre>

```java
Arrays.sort(friends, (first, second) -> first.length() - second.length());
```

<pre>
따라서 위와 같이 직접 함수 람다식을 넣어도 된다. (원래는 Comparator 객체가 파라미터)
</pre>

```java
public interface Predicate<T> {
  boolean test(T t);
}
```

<pre>
표준 라이브러리에서는 Predicate 함수형 인터페이스가 있다.
list.removeIf(e -> e == null);
ArrayList 클래스의 removeIf 메서드는 파라미터로 Predicate를 받는다.
위 문장은 리스트에서 모든 null 값을 제거한다.
</pre>

```java
public class Main {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<>();
        list.add("TEST1");
        list.add("TEST1");
        list.add("TEST2");
        list.add("TEST3");
        list.add("TEST4");
        Consumer<String> style = (String s) -> System.out.println(s);

        list.forEach(style);

        Predicate<String> predicate = s -> s.equals("TEST1");

        list.removeIf(predicate);

        System.out.println("After removeIf");
        list.forEach(style);
    }

}
```
<pre>
Consumer 역시 함수형 인터페이스를 위한 객체다.
</pre>

<h2>3.5 메서드 참조와 생성자 참조</h2>
<h2>3.5.1 메서드 참조</h2>
<pre>
함수형 인터페이스가 이미 구현되어 있을 수 있다.
</pre>

```java
public class Main {
    public static void main(String[] args) {

        String[] strings = new String[] {"TEST1", "TEST2", "test1"};

        consumer(strings);

//        Arrays.sort(strings, (x, y) -> x.compareToIgnoreCase(y));
        Arrays.sort(strings, String::compareToIgnoreCase);

        System.out.println("After sort");
        consumer(strings);
    }

    public static void consumer(String[] strings) {
        for(String s : strings) {
            System.out.println(s);
        }
    }

}
```

<pre>
String::compareToIgnoreCase 는 대소문자를 상관안하고 정렬하는 함수형 인터페이스다.
</pre>

```java
public class Main {
    public static void main(String[] args) {

        ArrayList<String> strings = new ArrayList<>();
        strings.add("TEST");
        strings.add("TEST");
        strings.add("TEST");
        strings.add(null);
        strings.add(null);

        consumer(strings);

//        strings.removeIf(s -> s == null);
        strings.removeIf(Objects::isNull);
        System.out.println("After");
        consumer(strings);
    }

    public static void consumer(String[] strings) {
        for(String s : strings) {
            System.out.println(s);
        }
    }

    public static void consumer(List<String> strings) {
        for(String s : strings) {
            System.out.println(s);
        }
    }
}
```

<pre>
Objects:isNull 은 값이 null인지 확인하는 메서드다.
list.forEach(System.out::println);

:: 연산자는 메서드 이름과 클래스를 분리하거나, 메서드 이름과 객체 이름을 분리한다.

1. 클래스::인스턴스메서드
(x,y) -> x.compareToIgnoreCase(y)
==> String::compareToIgnoreCase
첫 번째 파라미터가 메서드의 수신자가 되고 나머지는 해당 메서드로 전달

2. 클래스::정적메서드 (Objects:isNull)
x -> Objects.isNull(x)
==> Objects:isNull
모든 파라미터가 정적 메서드로 전달

3. 객체::인스턴스메서드
System.out.println(x)
==> System.out::println
주어진 객체의 메서드가 호출되고, 파라미터는 인스턴스 메서드로 전달
</pre>

<h2>3.5.2 생성자 참조</h2>
<pre>
Stream<Employee> stream = names.stream().map(Employee::new);
new라는 메서드 이름을 사용한다.
</pre>

<h2>3.6 람다 표현식 처리하기</h2>
<h2>3.6.1 지연 실행 구현하기</h2>
<pre>
람다는 사용하는 목적은 지연 실행이다.
- 별도의 스레드에서 코드 실행
- 코드를 여러번 실행
- 알고리즘의 올바른 지점에서 코드 실행
- 이벤트(클릭, 데이터 수신..)가 일어날 때 코드 실행
- 필요할 때만 코드 실행
</pre>

```java
public class Main {
    public static void main(String[] args) {
        repeat(10, () -> System.out.println("Hello, World"));
    }
    public static void repeat(int n, Runnable action) {
        for(int i=0; i<n; i++) action.run();
    }
}
```

<pre>
액션을 n번 반복하기 위해 카운트와 액션을 전달한다.
액션은 함수다. 하지만 자바는 함수를 전달할 수 없다.
그래서 단일 메서드를 가진 Runnable 인터페이스를 파라미터로 한다.
파라미터에서 구현한 람다 함수는 Runnable의 run 메서드에서 실행된다.
</pre>

```java
public class Main {
    public static void main(String[] args) {
        repeat(10, i -> System.out.println("Countdown: " + (9-i)));
    }

    public static void repeat(int n, IntCounsmer action) {
        for(int i=0; i<n; i++) action.accept(i);
    }
}

interface IntCounsmer {
    void accept(int value);
}
```

<pre>
몇번째 반복 수행인지를 알고 싶을 때 위처럼 개선하면 된다.
직접 구현보다는 제공되는 준 함수형 인터페이스를 사용하는 것이 좋다.
</pre>

```java
import java.awt.*;
import java.awt.image.BufferedImage;

public class Main {
    public static void main(String[] args) {
        BufferedImage frenchFlag = createImage(150, 100, (x, y) -> x < 50 ? Color.BLUE : x < 100 ? Color.WHITE : Color.RED);
    }

    public static BufferedImage createImage(int width, int height, PixelFunction f) {
        BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_BGR);

       for(int x=0; x<width; x++) {
          for(int y=0; y<height; y++) {
               Color color = f.apply(x, y);
               image.setRGB(x, y, color.getRGB());
          }
       }
       return image;
   }
}

@FunctionalInterface
interface PixelFunction {
    Color apply(int x, int y);
}
```

<pre>
위처럼 함수형 인터페이스를 anootation을 이용해서 직접 만들 수도 있다.
파라미터로 int, int를 받고 Color를 반환하고 싶을 때 적당한 내부 함수형 인터페이스가 없기 때문에 만들었다.
하는 역할은 x,y 좌표에 Color를 반환해주고 createImage를 호출할 때 좌표에 맞는 Color를 지정하는 함수를 구현한다.
</pre>

<h2>3.7 람다 표현식과 변수 유효 범위</h2>

<pre>
람다 표현식에서 변수명을 조심하자
int first = 0;
Comparator<String> comp = (first, second) -> first.length() - second.length();
// first 변수명이 이미 정의됐다.

public class Application {
  public void doWork() {
      Runnable runner = () -> { ...; System.out.println(this.toString()); ... };
      ...
  }
}

여기서 this는 Application을 가리킨다. Runnable 인스턴스가 아니다.
함수형 인터페이스로 run 메서드 안 this가 아니다.
</pre>


