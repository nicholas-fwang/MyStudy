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
<pre>
자바에서는 실행 시간에 객체가 어느 클래스에 속하는지 알아낼 수 있다.
equals나 toString 메서드 구현 시 유용하고
클래스를 어떻게 로드했는지 알아내어 클래스와 관련된 데이터, 즉 리소스를 로드할 수도 있다.
</pre>

<h2>4.4.1 Class 클래스</h2>

<pre>
객체의 참조를 저장하는 Object 타입의 변수가 있다.
이 변수에서 객체의 더 많은 정보(객체가 속한 클래스 등..)를 얻고 싶다고 해보자.
</pre>

```java
Object obj = ...;
Class<?> cl = obj.getClass();
```

<pre>
generic은 추후에 하자.
getName()을 통해 클래스의 이름을 알아낼 수 있다.
</pre>

```java
System.out.println("This is object is an instance of " + cl.getName());

// 정적 메서드 Class.forName을 사용해서 얻을 수도 있다.
String className = "java.util.Scanner";
Class<?> cl = Class.forName(className);
```

<pre>
Class.forName의 용도는 컴파일 시간에는 알려지지 않은 클래스의 Class 객체를 생성하는 것이다.
원하는 클래스를 미리 알고 싶으면 Class.forName 대신 클래스 리터럴을 사용하자.
</pre>

```java
Class<?> cl = java.util.Scanner.class;
Class<?> cl2 = String[].class // String[] 배열 타입을 작성한다.
Class<?> cl3 = Runnable.class // Runnable 인터페이스를 작성한다.

// equals 메서드
if(other.getClass() == Employee.class)
```

<h2>4.4.2 리소스 로드하기</h2>
<pre>
Class 클래스를 이용해 config 파일이나 image 등 리소스를 찾아올 수 있다.
클래스 파일과 같은 디렉터리에 리소스를 넣었을 때 다음과 같이 해당 파일에 대응하는 입력스트림을 열 수 있다.
</pre>

```java
InputStream stream = MyClass.class.getResourceAsStream("config.txt");
Scanner in = new Scanner(stream);
```

<pre>
리소스에는 서브 디렉터리가 포함될 수 있다.
상대경로나 절대경로로 지정할 수도 있다.
MyClass.class.getResourceAsStream("/config/menus.txt")
는 MyClass가 속한 패키지의 루트를 담고 있는 디렉터리에서 config/menus.txt를 찾는다.
</pre>

<h2>4.3.3 클래스 로더</h2>
<pre>
가상 머신에서는 명령어를 클래스 파일에 저장한다.
각 클래스 파일에는 단일 클래스나 인터페이스에 해당하는 명령어를 담는다.
클래스 파일은 file system, JAR, Remote, 메모리 등에서 동적으로 생성할 수도 있다.
클래스 로더는 바이트를 로드해서 가상 머신의 클래스나 인터페이스로 변환하는 역할을 한다.
</pre>

<pre>
가상머신은 main 메서드가 호출될 메인 class부터 시작해 필요할 때 클래스 파일을 로드한다.
메인 class에서 java.lang.System이나 java.util.Scanner 같은 클래스를 사용한다면 이 클래스를 로드하고
이 클래스들이 각 의존 클래스를 로드한다.
</pre>

<pre>
자바 프로그램을 실행할 때 최소 3가지 클래스 로더가 연관된다.
1. 부트스트랩 로더 : 자바 라이브러리 클래스를 로드한다. 보통은 jre/lib/rt.jar 파일에서 로드
부트스트랩 로더는 가상 머신의 일부다.
2. 확장 클래스 로더 : jre/lib/ext 디렉터리에서 '표준 확장'을 로드한다.
3. 시스템 클래스 로더 : 애플리케이션 클래스를 로드한다. 또한 클래스 패스에 있는 디렉터리와 JAR 파일에서 클래스를 찾는다.
</pre>

<pre>
부트스트랩 클래스 로더에 대응하는 ClassLoader 객체는 없다.
예를들면 String.class.getClassLoader()는 null을 반환한다.
자바 구현에서는 확장 클래스 로더와 시스템 클래스 로더로 자바를 구현한다.
두 클래스 모두 URLClassLoader 클래스의 인스턴스다.

자신만의 URLClassLoader 인스턴스를 생성하면 클래스 패스에 없는 디렉터리나 JAR 파일에 클래스를 로드할 수 있다.
플러그인을 로드할 때 이 방법을 이용한다.
</pre>

```java
URL[] = urls {
    new URL("file:///path/to/directory/"),
    new URL("file://path/to/jarfile.jar")
};
String className = "com.mycompany.plugins.Entry";
try (URLClassLoader loader = new URLClassLoader(urls)) {
    Class<?> cl = Class.forName(className, true, loader);
    // 이제 cl 인스턴스를 생성한다 (4.5.4)
}
```

<pre>
Class.forName의 두번째 파라미터(true)는 대상 클래스르 로드한 후 정적 초기화가 일어남을 보장한다.

URLClassLoader는 파일 시스템에서 클래스를 로드한다.
다른 곳에서 클래스를 로드하려면 자신만의 클래스 로더를 구현해야 한다. 클래스 로더는 findClass 메서드만 구현하면 된다.
</pre>

```java
public class MyClassLoader extends ClassLoader {
    ...
    @Override
    public Class<?> findClass(String name)
      throws ClassNotFoundException {
      byte[] bytes = "the byte of the class file";
      return defineClass(name, bytes, 0, bytes.length);
    }
}
```

<h2>4.4.4 컨텍스트 클래스 로더</h2>
<pre>
대부분은 클래스 로딩 과정을 신경 쓰지 않아도 된다.
메서드가 클래스를 동적으로 로드하는데 또 다른 클래스 로더가 로드한 클래스에서 이 메서드를 호출하면
문제가 생길 수 있다.
그 예를 보자.
</pre>

<pre>
1. 시스템 클래스 로더가 로드하는 유틸리티 클래스를 만들었다.
</pre>

```java
public class Util {
    Object createInstance(String className) {
        Class<?> cl = Class.forName(className);
    }
}
```

<pre>
2. 또 다른 클래스 로더(사용자가 만든 클래스 로더)가 플러그인 JAR에서 클래스를 읽어오는 플로그인을 로드한다.

3. 이 플러그인은 Util.createInstance("com.mycompany.plugins.MyClass")를 호출해서
플러그인 JAR에 들어 있는 클래스의 인스턴스를 생성한다.

즉, 플러그인 클래스 로더가 내부 클래스 로더의 클래스에 접근해야 된다.
플러그인 클래스 로더는 자신의 로더를 이용해 해당 클래스를 로드하려 하기 때문에 에러가 발생한다.
이런 현상을 클래스 로더 역전이라고 한다.

이를 해결 하기 위해서 클래스 로더를 같이 전달해야 한다.
</pre>

```java
public class Util {
    public Object createInstance(String className, ClassLoader loader) {
        Class<?> cl = Class.forname(className, true, loader);
        ...
    }
    ...
}
```

<pre>
또 다른 방법은 현재 스레드의 컨텍스트 클래스 로더를 사용하는 것이다.
메인 스레드 클래스 로더 -> 시스템 클래스 로더
새 스레드에 별도 처리를 하지마 않으면 모두 새로운 스레드 역시 시스템 클래스 로더를 바라본다.
</pre>

```java
Thread t = Thread.currentThread();
t.setContextClassLoader(loader);
```

```java
public class Util {
    public Object createInstance(String className) {
        Thread t = Thread.currentThread();
        ClassLoader loader = t.getContextClassLoader();
        Class<?> cl = Class.forname(className, true, loader);
        ...
    }
    ...
}
```

<h2>4.4.5 서비스 로더</h2>
<pre>
프로그램에서 플러그인을 구현할 때 ServiceLoader 클래스를 이용해 공통 인터페이스를 준수하는 플러그인을 손쉽게 로드할 수 있다.
서비스의 각 인스턴스에서 제공해야 하는 메서드를 포함하는 인터페이스(또는 슈퍼클래스)를 정의한다.

예를 들어 암호화 제공 서비스
</pre>

```java
package com.corejava.crypt;

public interface Cipher {
    byte[] encrypt(byte[] source, byte[] key);
    byte[] decrypt(byte[] source, byte[] key);
    int strength();
}
```

<pre>
서비스 제공자는 이 서비스를 구현하는 클래스를 하나 이상 제공
</pre>

```java
package com.corejava.crypt.impl;

public class CaeserCihper implements Cipher {
    byte[] encrypt(byte[] source, byte[] key) {
        ...
    }
    byte[] decrypt(byte[] source, byte[] key) {
        ...
    }
    int strength() {
        ...
    }
}
```

<pre>
구현 클래스의 패키지는 인터페이스에 제한되지 않는다.
인자없는 생성자는 필수다

UTF-8 방식으로 인코드된 텍스트 파일에 클래스 이름을 추가한다.
(META-INF/services 디렉토리)
META-INF/services/com.corejava.crypt.impl.CaeserCihper

프로그램에서 다음과 같이 서비스 로더를 초기화 한다.
</pre>

```java
public static ServiceLoader<Cipher> cipherLoader = ServiceLoader.load(Cipher.class);

public static Cihper getCipher(int minStrength) {
    for(Cipher cipher : cihperLoader)
        if(cihper.strength() >= minStrength) return cipher;
    return null;
}
```

<pre>
TODO : ServiceLoader로 배포한 플러그인 따라해보기
</pre>

<h2>4.5 리플렉션</h2>
<pre>
SKIP
</pre>
