---
layout: post
title:  "Atomic 클래스 그리고 Optimistic Lock, Pessimistic Lock"
subtitle: "Atomic 변수로 알아보는 Optimistic, Pessimistic Lock"
date:   2020-12-09 16:47:00
categories: [study]
---

자바에서 `java.util.concurrent.atomic` 패키지는 Atomic 클래스들을 제공해준다.

`AtomicInteger`, `AtomicLong`, `AtomicBoolean`, `AtomicReference` 등이 제공되고 각각의 타입에 맞게 Atomic한 접근 및 내부적으로 Lock 및 메모리 최신화를 제공한다.

`get`, `set`, `compareAndSet` 등의 메소드가 제공된다.
- `set`메소드의 경우에는 항상 get보다 먼저 발생하고(happens-before relationship)
- `compareAndSet`과 같은 메소드로 메모리 최신화를 가능하도록 했다.


<br>
<br>

```java
class Counter {
    private int c = 0;

    public void increment() {
        c++;
    }

    public void decrement() {
        c--;
    }

    public int value() {
        return c;
    }

}
```
위의 `Counter` 클래스는 변수를 읽을 때는 atomic하나 `increment`, `decrement` 시 atomic하지 않고, 메모리 최신화 이슈까지 있다.

<br>
<br>

```java
class SynchronizedCounter {
    private int c = 0;

    public synchronized void increment() {
        c++;
    }

    public synchronized void decrement() {
        c--;
    }

    public synchronized int value() {
        return c;
    }

}
```
위와 같이 직접 synchronized 키워드를 직접 사용할 수 있다.

그런데 클래스가 더 복잡해지면 불필요한 synchronization으로 인해서 여러 비효율이 생길 수 있다.

`Pessimistic Locking`처럼 직접 락을 획득하고 오퍼레이션이 종료되면 락을 해제하는 방법이다.

단어 그대로, 작업이 이루어지는 내용에 락이 필요한데 기존에 사용하는 락이 없을 것이라는 점을 가정한다.


<br>
<br>

```java
import java.util.concurrent.atomic.AtomicInteger;

class AtomicCounter {
    private AtomicInteger c = new AtomicInteger(0);

    public void increment() {
        c.incrementAndGet();
    }

    public void decrement() {
        c.decrementAndGet();
    }

    public int value() {
        return c.get();
    }

}
```

Atomic 클래스를 사용하면 위와 같이 사용할 수 있다.

synchronized의 기능은 이미 Atomic 클래스에서 다 처리 되기 때문에 메소드에 따로 synchronized 키워드가 필요하지 않다.

또한 Atomic 클래스를 만들어 놓은 뛰어난 개발자들이 최적화해 놓았기 때문에 성능상 이점이 있다.

이 또한 `Optimistic lock`처럼 락이 이미 잘 만들어져있다는 것을 전제하고 따로 락이 없이 오퍼레이션을 진행하고 있다.



출처: https://docs.oracle.com/javase/tutorial/essential/concurrency/atomicvars.html