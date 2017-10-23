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
