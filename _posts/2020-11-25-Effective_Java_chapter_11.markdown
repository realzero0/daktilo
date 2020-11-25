---
layout: post
title:  "Effective java Chapter 11 병렬처리"
subtitle: "이펙티브 자바 정리"
date:   2020-11-25 10:47:00
categories: [study]
---

# Chapter 11. 병렬 처리

## Item 78: 공유된 mutable 데이터의 접근을 동기화(Synchronize)하라

`synchronized` 키워드는 1개의 쓰레드에서 1개의 메소드나 block이 실행됨을 보장한다. 완벽한 동기화는 어떤 메소드도 특정 객체를 불완전한 상태에서 보지 않음을 의미한다.

동기화 없이는 하나의 쓰레드에서의 변경이 다른 쓰레드에서 보이지 않는다. 또한 이전 변경의 결과를 각각의 쓰레드가 볼 수 있게 된다.

long이나 double이 아닌 변수는 원자적(atomic)이다(JLS, 17.4, 17.7). 다른 말로 하면 long이나 double이 아니면 여러개의 쓰레드로 값을 병렬적으로 변경하더라도 결과는 나온다는 의미다.

성는 향상을 위해서 원자적 데이터를 읽고 쓸때 동기화를 생략할 수 있다고 생각할 수 있다. 동기화는 쓰레드 간의 상호 배체(mutual exclusion)뿐 아니라 믿을 수 있는 커뮤니케이션이 가능하도록 한다.

이는 자바 언어의 사양(specification)의 일부가 메모리 모델로 알려져 있기 때문이다. 
메모리 모델은 언제 그리고 어떻게 특정 쓰레드에서 변경이 다른 쓰레드에 보이는지를 결정한다.

데이터가 원자적(atomic)으로 접근된다고 하더라도, 공유된 mutable 데이터에 동기화 실패의 결과는 심각할 수 있다.

다른 쓰레드에서 특정 쓰레드를 멈추는 것을 예로 들면,

- 라이브러리에서는 Thread.stop 제공한다.
    - 이는 옛날에 deprecated 되었다. 본질적으로 unsafe하기 때문이다.
    - 이는 data 변질(corruption)을 일으킨다. Thread.stop을 사용해서는 안된다.
    - 첫번째 쓰레드가 자기 스스로를 멈출 수 있게 하는 boolean 값을 가지고 다른 쓰레드가 그 값을 변경할 수 있게 하면 된다.
    
 ```java
// Broken! - How long would you expect this program to run?
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

위 예제가 1초 정도 동작하고 종료할 것이라고 생각할 것이다.

그렇지만 위 예제는 저자의 컴퓬터에서 아예 멈추지 않았다.

문제는 동기화가 없이는 언제 background 쓰레드가 메인쓰레드에 있는 값의 변경을 알지 장담할 수 없다는 것이다.

```java
while (!stopRequested)
    i++;
```
동기화 없이는 위 코드가 JVM에서 아래 코드로 변환될 수도 있다.
```java
if (!stopRequested)
    while (true)
        i++;
```

이런 최적화를 hoisting이라고 알려져 있다. 
- 이 결과는 liveness failure이다. 
    - 프로그램은 진행을 실패하게 된다.
    - 이 문제를 해결할 수 있는 한가지 방법은 stopRequested 필드의 접근을 동기화 하는 것이다.


```java
// Properly synchronized cooperative thread termination
public class StopThread {
    private static boolean stopRequested;

    private static synchronized void requestStop() {
        stopRequested = true;
    }

    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested())
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```

위와 같이 동기화하게 되면 문제가 없어진다.

- 읽는 메소드, 쓰는 메소드 모두 동기화되어있음에 주목해야 한다.
- 동기화는 읽고 쓰는 오퍼레이션 모두를 동기화 하지 않으면 동작을 보장할 수 없다.
- 가끔 쓰는 곳이나 읽는 곳에만 동기화되서 잘 동작하는 경우가 있다. 하지만 이 예제에는 아니다.


StopThread 내의 메소드는 동기화 없이도 원자적(atomic)이다. 
이 예제의 결과는 쓰레드간 결과의 커뮤니케이션(communication effect)을 위한 것이고, 
쓰레드의 상호 배제(mutual exclusion)를 위한 것은 아니다.

적은 라인의 코드를 사용할 수록 동기화로 인한 성능은 더 향상된다.


```java
// Cooperative thread termination with a volatile field
public class StopThread {
    private static volatile boolean stopRequested;

    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

위 예제에서는 volatile 키워드가 사용되었는데, 
- 이는 thread간의 상호배제(mutual exclusion)에는 아무 역할을 하지 못한다.
- 어떤 쓰레드에서도 가장 최근에 작성된 값을 읽을 수 있게 한다(CPU Cache가 아니라 메인 메모리에서 가장 최신값을 가져오기 때문(?))

그렇지만 volatile 키워드를 사용할 때는 주의해야 한다.
```java
// Broken - requires synchronization!
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```

위 예제에서 문제는 연산자(++)은 원자적이지 않다.
nextSerialNumber++를 여러 쓰레드에서 부르게 되면.. 
국 같은 값을 보고 값을 증가시키는 경우가 발생하게 된다.

이를 safety failure라고 하고, 프로그램은 잘못된 값을 계산해낸다.

이를 고치기 위한 하나의 방법은 generateSerialNumber에 synchronized 키워드를 넣는 것이다.
그렇게 하면 volatile 키워드를 지워도 된다.


```java
// Lock-free synchronization with java.util.concurrent.atomic
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
}
```
위와 같이 AtomicLong과 같이 java.util.concurrent.atomic에 있는 클래스를 사용하면 된다.

위 패키지는 primitive type에 대해서 lock이 필요없는 thread-safe한 변수를 사용할 수 있게 해준다.

```java
// Lock-free synchronization with java.util.concurrent.atomic
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
}
``` 

이 주제에 나오는 문제를 피하는 가장 좋은 방법은 mutable 데이터를 공유하지 않는 것이다. immutable 데이터를 공유하던지 아예 공유하지 않으면 된다.

다른 말로, 하나의 쓰레드 내에서만 mutable 데이터를 사용하라는 의미이기도 하다.

사용하지 않기로 결정했다면 그런 결정을 잘 관리해서 나중에도 잘 유지되도록 할 필요가 있다.

사용하고 있는 프레임워크나 라이브러리에 대한 깊은 이해를 가져야 알 수 없는 쓰레드에 의해서 생기는 영향을 최소화할 수 있다.

하나의 쓰레드가 데이터 객체를 수정하고 다른 쓰레드에 동기화해서 공유하는 것은 가능할 수 있다. 

다른 쓰레드들은 다시 수정되지 않는 한 따로 동기화 없이 객체를 읽을 수 있다. 

이런 객체를 effectively immutable한 객체라고 한다.

하나의 쓰레드에서 다른 쓰레드로 객체의 참조(reference)를 전달하는 것을 safe publication이라고 한다.

safe publication에는 많은 방법이 있다.
- static field에 저장
- volatile field에 저장
- final field로 저장
- 일반적인 locking으로 접근가능한 field에 저장
- concurrent collection에 저장

결론,

여러 쓰레드에서 접근 가능한 mutable 데이터를 공유할 때는 읽고 쓸 때 모두 동기화 해야한다.


## Item 79: 과도한 동기화(synchronization)을 피하라

과도한 동기화는 성능 저하, 데드락, 예측하기 힘든 동작으로 이어질 수 있다.

liveness and safety failure를 피하기 위해서는 클라이언트에게 동기화된 메소드나 block의 제어를 넘겨주면 안된다.
- 동기화된 범위는 override하도록 디자인하면 안된다.
- 클라이언트에게는 alien이어야 한다. 메소드가 뭘하는지도 몰라야하고, 메소드의 권한도 없어야 한다.
- alien메소드의 동작에 의존하면서 그 메소드를 동기화된 다른 곳에서 호출하게 되면 exception, 데드락, 데이터 변질(corruption)이 발생할 수 있다.

```java
// Broken - invokes alien method from synchronized block!
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) { super(set); }

    private final List<SetObserver<E>> observers
            = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) {
        synchronized(observers) {
            for (SetObserver<E> observer : observers)
                observer.added(this, element);
        }
    }

    @Override public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element);  // Calls notifyElementAdded
        return result;
    }
}
```
위 예제는 새 element가 추가될 때 클라이언트가 notification을 구독(subscribe)할 수 있게 해준다.

Item 18의 ForwardingSet을 재사용하고 있다.

```java
@FunctionalInterface public interface SetObserver<E> {
    // Invoked when an element is added to the observable set
    void added(ObservableSet<E> set, E element);
}
```
callback을 위한 인터페이스는 위와 같을 것이다.

위의 인터페이스는 BiConsumer<ObservableSet<E>,E>과 구조적으로는 동일하다.
- 인터페이스와 메소드 명이 가독성을 높여주고
- 인터페이스가 여러개의 callback으로 결합되기 때문에 새로 인터페이스를 정의했다.

```java
public static void main(String[] args) {
    ObservableSet<Integer> set =
            new ObservableSet<>(new HashSet<>());

    set.addObserver((s, e) -> System.out.println(e));

    for (int i = 0; i < 100; i++)
        set.add(i);
}
```

위와 같은 예제에서는 문제없이 동작한다.

```java
set.addObserver(new SetObserver<>() {
    public void added(ObservableSet<Integer> s, Integer e) {
        System.out.println(e);
        if (e == 23)
            s.removeObserver(this);
    }
});
```

만약 값이 23일 경우에는 자기 스스로를 지우는 예제이다..

람다에서 익명클래스를 사용한 이유는 자기 자신을 전달하기 위해서이다.

위 예제가 0부터 23까지 print한 뒤에 종료될 것으로 판단할 수 있다.

그렇지만 ConcurrentModificationException이 발생한다.

왜?

- added는 observers 리스트에 대해서 notifyElementAdded 메소드가 호출될 때 호출된다.
- added내에서는 remove 메소드로 observers 객체의 element를 지우고 있는데, 순회를 하면서 지우고 있끼 때문에 문제가 된다.
- notifyElementAdded 메소드의 순회는 동기화된 block에서 concurrent modification을 막는다.
- 하지만 쓰레드 내에서 observable set이나 observers 리스트 수정을 막지는 못한다.


```java
// Observer that uses a background thread needlessly
set.addObserver(new SetObserver<>() {
   public void added(ObservableSet<Integer> s, Integer e) {
      System.out.println(e);
      if (e == 23) {
         ExecutorService exec =
               Executors.newSingleThreadExecutor();
         try {
            exec.submit(() -> s.removeObserver(this)).get();
         } catch (ExecutionException | InterruptedException ex) {
            throw new AssertionError(ex);
         } finally {
            exec.shutdown();
         }
      }
   }
});
```

이 예제를 실행하면 예외는 나오지 않고, 데드락이 걸리게 된다.

메인 쓰레드는 이미 락을 걸고 있고, background 쓰레드가 s.removeObserver로 락을 걸려고 하면 실패한다.

메인 쓰레드는 다시 background 쓰레드가 끝나기를 기다리지만 끝나지 않는다.

이 예제 자체는 사실 그렇게 말이 되는 상황은 아니지만, 동기화된 곳에서 alien 메소드를 실행하는 것은 데드락을 일으키는 것은 맞다.

```java
// Alien method moved outside of synchronized block - open calls
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized(observers) {
        snapshot = new ArrayList<>(observers);
    }
    for (SetObserver<E> observer : snapshot)
        observer.added(this, element);
}
```

위와 같이 alien 메소드의 호출을 동기화된 block 밖으로 보내면 가능하다. 위 예제는 snapshot 객체를 만들어서 동기화된 block 밖으로 보냈다.

자바 라이브러리에서 제공하는 CopyOnWriteArrayList 같은 컬렉션을 이용하면 더 간단하게 가능하다.

위 라이브러리를 사용하면 모든 변경하는 오퍼레이션에서 새로운 copy를 생성하고 locking이 필요없다. 변경이 잦으면 성능이 매우 느리지만, 위 예제와 같이 변경이거의 없는 경우에는 매우 유용하다.

```java
// Thread-safe observable set with CopyOnWriteArrayList
private final List<SetObserver<E>> observers =
        new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

private void notifyElementAdded(E element) {
    for (SetObserver<E> observer : observers)
        observer.added(this, element);
}
```

위와 같이 CopyOnWriteArrayList를 사용하면 외부의 직접적인 동기화가 없어지게 된다. alien 메소드가 동기화 된 곳 밖에서 호출되는 것을 open call이라고 한다.

실패를 방지하는 것 외에도 open all은 병렬성을 높이게 된다. alien 메소드는 언제 끝날지 알수 없을 수가 있다. 만약에 alien 메소드가 동기화된 곳에서 호출되면 다른 쓰레드들은 보호된 리소스에 대한 접근이 불필요하게 거부될 수 있다.

동기화된 곳에서는 최소한의 일만을 해야한다
- 락을 획득
- 공유된 데이터를 조사
- 데이터를 변환
- 락을 해제

시간이 많이 드는 작업을 해야한다면, Item 78을 어기지 않는 범위에서 최대한 동기화 블록의 범위를 줄여야 한다.

---
앞서 본 이야기는 맞는 결과(correctness)에 관한 문제다.

성능 이야기를 해보자.

자바의 초기 릴리즈 이후에 동기화 성능은 많이 좋아졌지만, 과도한 동기화는 절대 해서는 안된다.

멀티 코어에서는 과도한 동기화는 CPU 사용 시간을 점유하는 것 뿐 아니라
- 병렬성을 잃게 한다
- 모든 코어가 일관성있는 메모리 뷰를 봐야하기 때문에 지연이 발생한다.
- VM의 코드 실행 최적화를 방해한다.

mutable 클래스를 작성해야하면 2가지 옵션이 있다.
- 모든 동기화를 생략하고 병렬 접근이 필요할 때, 클라이언트가 외부적으로 동기화 하는 것을 허용하는 것
- 클래스를 thread-safe하게 만들어서 내부적으로 동기화 할 수도 있다.

두번째 옵션을 사용해야 더 높은 병렬성을 얻을 수 있다.


만약 내부적으로 동기화하는 클래스를 만들게 되면 높은 병렬성을 획득하기 위한 lock splitting, lock striping, and nonblocking concurrency control 등이 가능하다.
- lock splitting : 데이터를 여러 데이터로 나눠서 락을 분할
- lock striping : 컬렉션 등의 범위를 나누어서 락을 분할

하나의 스태틱 필드가 여러 쓰레드에서 접근가능하다면 동기화해야한다.

이 경우에는 외부 쓰레드가 동기화 하는 게 불가능하다. 관계 없는 클라이언트가 동기화 없이 해당 메소드를 실핼 할 수 있기 때문이다.

결론,

데드락과 데이터 변질(corruption)을 막기 위해서는 alien 메소드를 동기화 블럭에서 부르면 안된다. 최소한만 동기화 블럭에서 실행해야 한다.
