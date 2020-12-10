---
layout: post
title:  "Effective java Chapter 12 Serialization"
subtitle: "이펙티브 자바 정리"
date:   2020-12-07 15:47:00
categories: [study]
---

# Chapter 12. Serialization

## Item 85: Java Serialization말고 같은 기능의 다른 것들을 더 선호하라.

1997년에 처음 Serialization이 나왔을때는, 위험하다고 알려져 있었다. 

실제로 정확성, 성능, 보안, 유지보수 등에 잠재적 문제가 있다. 과거 지지자들은 이점이 더 많다고 했지만, 지금와서 보면 위험이 더 많다.

<br>
<br>

이 책의 2번째 판본에서 나왔던 보안 이슈들은 결국 2016년에 샌프란시스코 도시철도국에 랜섬웨어 공격(2일간 요금 징수 시스템 으로까지 이어졌다.

가장 큰 문제는 "공격 범위"가 너무 넓다는 것이다. 

ObjectInputStream의 readObject 메소드로 deserialize된다.

deserialize 하는 과정에서 이 메소든는 이 해당 타입의 어떤 코드도 실행가능하다.

<br>
<br>

이제 더 이상 자바 Serialization을 사용할 이유는 없다.

이번 챕터에서는 크로스플랫폼 구조데이터(cross-platform structured-data representations)에 대해 다룬다.

<br>
<br>

가장 유명한 것은 JSON과 protobuf로 불리는 Protocol Buffer이다.

언어중립적(language-neutral)으로 불린다.

- Json
    - Json은 브라우저 서버 커뮤니케이션을 위해서 디자인되었다.
    - 자바스크립트를 위해서 개발되었다.
    - test-based이고 사람이 읽기 쉽다.
    - 데이터만 나타낼 수 있다.
    
- Protocol Buffer
    - 구글에서 구조화된 데이터의 저장과 교환을 위해서 만들어졌다.
    - C++를 위해서 개발되었다
    - binary이고 더 효율적이다.
    - 데이터 뿐 아니라 문서화를 위한 스키마(type)도 제공한다.
    - binary도 제공하지만, pbtxt라는 읽기 쉬운 형태도 제공한다.

<br>
<br>

자바 serialization을 완전히 피할 수 없다면,(레거시 어플리케이션 등)
 
절대 믿을 수 없는 데이터를 deserialize해서는 안된다.
 
 믿을 수 없는 source에서 RMI 트래픽을 허용해서는 안된다.
 - RMI(remote method invocation)는 분산되어 있는 시스템에서 객체를 전달 할 수 있게 해준다.
 
<br>
<br>

serialization을 피할 수 없고, 데이터의 안정성을 확신 할 수 없다면, 
 
object deserialization filtering을 사용하면 된다(자바9에 추가)

deserialization하기 전에 데이터 스트림을 필터 할 수 있도롤 해준다.

화이트리스팅하는 편이 블랙리스팅하는 것 보다 낫다.

Serial Whitelist Application Trainer (SWAT)이라는 툴도 화이트 리스팅을 위해 사용할 수 있다고 한다.
 
필터링 하게 되면
- 과도한 메모리 사용
- 너무 깊은 객체 그래프(너무 심하게 깊으면 안된다.)
등에 도움이 된다. 
 
 
<br>
<br>

불행히도 Serialization은 자바 에코시스템에 많이 존재하고 있다.

만약 그런 시스템을 유지보수 한다면, Json이나 Protobuf로 변경하는 것을 심각하게 고민해야 한다.

Serializable 클래스를 정확하고, 안전하고, 효율적으로 관리하는 것은 매우 주의가 필요하다.

이 챕터에서는 언제, 그리고 어떻게 할 수 있는지를 다룬다.

 
<br>
<br>

결론,

Serialization을 피할 수 있으면 피하고, 피할 수 없다면 화이트 리스트 등 최대한 안전하게 사용하도록 해야 한다.

 
<br>
<br>

## Item 86: 아주 주의를 기울여서 Serializable을 구현하라

`implement Serializable`을 추가하는 정도로 특정 클래스의 객체를 Serialize하기 쉽다고 생각하는 사람들 이밚다.

하지만, 당장은 별로 큰 비용이 들지 않더라도, 장기적으로 꽤 많은 비용이 발생할 수 있다.

 
<br>
<br>

Serializable을 구현해서 생기는 가장 큰 비용은
- 배포 이후에 클래스 구현을 변경하기가 어려워진다는 점이다.
    - 한번 Serializable을 구현하게 되면, byte-stream(또는 Serialized form)이 외부에 공개된 API로서 역할을 하게 되버린다.
    - 한번 배포하고 나면 영원히 지원해줘야한다.
    - default serialized form을 사용하면 
        - private이나 package-private instance 필드까지 공개된 API가 되어버린다.
        - 정보은닉(information hiding)을 위한 효율성이 매우 떨어지게 된다.

 
<br>
<br>

default serialized form을 사용하고, 나중에 클래스의 내부를 바꾸려 한다면?
- serialized form의 호환 불가능한 변경이 필요하다.
- `ObjectOutputStream.putFields` `ObjectInputStream.readFields`를 사용해서 유지 보수 할 수 있지만, 어렵고, 이런 코드를 포함해야한다는 문제가 있다.
- high-quality serialized form을 잘 만들어서 길게 사용할 수 잇게 해야한다.

 
<br>
<br>

Serializable을 상속하면 생길 수 있는 문제점
- stream unique identifiers (serial version UID)가 필요하다.
    - 지정하지 않으면 자동으로 SHA-1 해시 function을 통해 런타임에 만들어진다.
    - 위 값은 클래스 이름, 구현하고 있는 인터페이스, 멤버 변수, 컴파일러에 의해 만들어지는 멤버 변수 등에 영향을 받는다.
    - 위 중에 하나만 바껴서 default UID는 변경되서 호환성을 지키지 못하게 된다.
- 버그와 보안 문제가 일어나기 쉽다.
    - deserialization은 숨은 생성자다. 이 생성자가 불필요한 권한을 획득하지 못하도록 주의를 기울이기 힘들다.
    - default deserialization에 의존하게 되면 객체에 invariant corruption과 잘못된 접근을 가능하게 한다.
- 새 버전의 클래스가 나오면 테스트에 드는 비용이 증가한다.
    - 새버전, 옛날 버전 모두 serialization, deserialization 해봐야 한다.
- 가볍게 여길 수 있는 결정이 아니다.
    - Value Class(BigInteger, Instnat 같은)나 Collection 클래스에 주로 사용된다.
    - 쓰레드 풀처럼 active entity에는 사용되지 않는다.
- 상속을 위한 클래스나 인터페이스는 Serializable을 상속하지 않는 편이 낫다.
    - 해당 클래스를 상속해서 사용하는 사람에게 큰 부담으로 작용할 수 있다.
    - 특정 프레임워크에서 Serializable을 모두 구현한 하위 클래스가 필요한 경우 등에는 사용할 수 있다.
    - Throwable, Component 클래스 등이 상위 클래스에서 구현되어 있는 경우이다.
    - 불변이어야 하는 변수가 있으면 `finalize` 메소드를 overriding 못하게 해야한다.
    - 만약 불변값이 기본값이면 안되는 경우에는 `readObjectNoData` 메소드를 추가해야 한다.
    
        ```java
        // readObjectNoData for stateful extendable serializable classes
        private void readObjectNoData() throws InvalidObjectException {
            throw new InvalidObjectException('Stream data required');
        }
        ```
 
<br>
<br>

Serializable을 상속하지 않기로 결정했다면,
- 상속을 위해 설계되었다면 하위 클래스에 더 많은 노력이 필요하다.
- 상위 클래스에는 파라미터가 없는 접근가능한 생성자가 필요하다. 그렇지 않으면 serialization proxy pattern을 사용해야 한다.(Item 90)

 
<br>
<br>

내부 클래스는 default serialized form을 사용하면 Serializable을 상속해서는 안된다.
- 컴파일러가 만드는 필드가 생기기 때문이다.

 
<br>
<br>

결론,

Serializable을 상속하기에는 단점이 많으니, 잘 고려해서 상속해야 한다.

 
<br>
<br>


## Item 87: custom serialized form의 사용을 고려하라.

시간이 없는데 클래스를 작성해야 한다면, 최대한 API 설계에 집중하는 것이 적절하다.

만들어 놓고 다음 릴리즈에서 새로 구현해서 배포하면 된다고 생각되지만,

default serialization form을 사용하면 다음 릴리즈때 영원히 종속받게 된다..

 
<br>
<br>

default serialized form이 적절하다는 확신이 없으면 사용하면 안된다.
- custom serialized form을 만들었을 때 default하고 거의 같다면 사용할 수 있다.

 
<br>
<br>

default serialized form
- 상대적으로 요율적이다.
- physical representation이 logical content하고 같으면 사용할 수 있다.

```java
// Good candidate for default serialized form
public class Name implements Serializable {
    /**
     * Last name. Must be non-null.
     * @serial
     */
    private final String lastName;

    /**
     * First name. Must be non-null.
     * @serial
     */
    private final String firstName;
    /**
     * Middle name, or null if there is none.
     * @serial
     */
    private final String middleName;

    ... // Remainder omitted
}
```

- 사람의 이름을 대표 하는 위와 같은 예제에서는 적절하다.


 
<br>
<br>

default serialized form이 적절하다고 판단하더라도, 종종 `readObject` 메소드를 반드시 제공해야한다.
- 앞선 `Name` 클래스 예제에서 `readObject` 메소드는 `lastName`, `firstName`이 non-null만 가능하도록 할 수 있다.


<br>
<br>

앞선 예제는 private 필드라도 공개 API가 되기 때문에 주석이 달려있다. 

`@serial` 태그를 사용해서 java-doc이 serialized form에 관한 페이지를 만들 수 있게 한다.


<br>
<br>

```java
// Awful candidate for default serialized form
public final class StringList implements Serializable {
    private int size = 0;
    private Entry head = null;

    private static class Entry implements Serializable {
        String data;
        Entry  next;
        Entry  previous;
    }

    ... // Remainder omitted
}
```

위 예제는
- Logical : 스트링의 리스트
- Physical : 더블 링크드 리스트

default serialized form을 사용하면 모든 리스트를 복사해야한다.

physical, logical에 대한 차이가 많음에도 default serialzed form을 사용하면 4가지 단점이 있다.
- 내부 구현이 외부 API로 공개 된다.
    - StringList.Entry가 Public API가 되고, StringList는 반드시 링크드리스트를 사용해야만 한다.
- 과도한 공간을 차지할 수 있다.
    - serialized form이 과도하게 커지면, 링크드 리스트로서 커져야하기 때문에 문제가 될 수 있다.
- 과도한 시간이 소요될 수 있다.
    - 위 예제처럼 다음 참조 객체를 향한 여정을 시작할 수 있다.
- 스택 오버플로우가 발생할 수 있다.
    - 재귀적(recursive) 호출이 객체에 대해 발생해서 스택 오버플로우가 발생할 수 있다.
    - StringList 기준 1000-1800개의 객체로 발생가능하다.

<br>
<br>

```java
// StringList with a reasonable custom serialized form
public final class StringList implements Serializable {
    private transient int size   = 0;
    private transient Entry head = null;

    // No longer Serializable!
    private static class Entry {
        String data;
        Entry  next;
        Entry  previous;
    }

    // Appends the specified string to the list
    public final void add(String s) { ... }

    /**
     * Serialize this {@code StringList} instance.
     *
     * @serialData The size of the list (the number of strings
     * it contains) is emitted ({@code int}), followed by all of
     * its elements (each a {@code String}), in the proper
     * sequence.
     */
    private void writeObject(ObjectOutputStream s)
            throws IOException {
        s.defaultWriteObject();
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (Entry e = head; e != null; e = e.next)
            s.writeObject(e.data);
    }

    private void readObject(ObjectInputStream s)
            throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        int numElements = s.readInt();

        // Read in all elements and insert them in list
        for (int i = 0; i < numElements; i++)
            add((String) s.readObject());
    }

    ... // Remainder omitted
}
```


<br>
<br>

어떤 serialized form을 사용하던, serial version UID는 명시적으로 정의하라.

- 약간의 성능 향상이 있을 수 있고,
```java
private static final long serialVersionUID = randomLongValue;
```
위와 같이 정의할 수 있다.

- UID는 unique할 필요가 없다.
- 새 버전의 클래스를 만들더라도 UID는 같게 할 수 있다.
- 호환성을 깨고 싶지 않다면, 한번 만든 UID는 변경해서는 안된다.


<br>
<br>

결론,

클래스가 serializable해야한다면, serializable form을 어떻게 정의할지 깊게 고민해야한다.


<br>
<br>

## Item 88: `readObject` 메소드는 방어적으로 작성하라.

```java
// Immutable class that uses defensive copying
public final class Period {
    private final Date start;
    private final Date end;
    /**
     * @param  start the beginning of the period
     * @param  end the end of the period; must not precede start
     * @throws IllegalArgumentException if start is after end
     * @throws NullPointerException if start or end is null
     */
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end   = new Date(end.getTime());
        if (this.start.compareTo(this.end) > 0)
            throw new IllegalArgumentException(
                          start + ' after ' + end);
    }

    public Date start () { return new Date(start.getTime()); }

    public Date end () { return new Date(end.getTime()); }

    public String toString() { return start + ' - ' + end; }

    ... // Remainder omitted
}
```

Item 50에서 mutable 클래스인 Date을 이용한 Immutable 클래스를 만들었다.

위 클래스는 Logical가 Physical 데이터가 같기 때문에 `implement Serializable`만 추가하면 Serializable하다.

그런데 이렇게 하면 클래스의 불변성을 장담할 수 없다.



<br>
<br>

원인은 `readObject` 메소드가 public 생성자로서 역할을 하기 때문이다.
- Validity check이 필요
- 방어적 복사를 할 필요가 있다.

위 두가지가 안되면, 공격자는 클래스의 불변성을 깨뜨릴 수 있다.


<br>
<br>

`readObject` 메소드
- 바이트 스트림을 하나의 파라미터로 하는 생성자다.
- 보통은 바이트 스트림은 일반적으로 생성된 객체를 serializing해서 만들어진다.
- 강제로 바이트 스트림을 만들어서 클래스의 불변성을 깨뜨릴 수 있다.
- 그런 byte stream은 일반적인 생성자에서 만들어질 수 없는 객체를 만드는데 사용될 수도 있다.


<br>
<br>

Period 예제에서는 시작 값이 끝 값보다 뒤에 있는 객체를 만들 수도 있따.

```java
public class BogusPeriod {
  // Byte stream couldn't have come from a real Period instance!
  private static final byte[] serializedForm = {
    (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06,
    0x50, 0x65, 0x72, 0x69, 0x6f, 0x64, 0x40, 0x7e, (byte)0xf8,
    0x2b, 0x4f, 0x46, (byte)0xc0, (byte)0xf4, 0x02, 0x00, 0x02,
    0x4c, 0x00, 0x03, 0x65, 0x6e, 0x64, 0x74, 0x00, 0x10, 0x4c,
    0x6a, 0x61, 0x76, 0x61, 0x2f, 0x75, 0x74, 0x69, 0x6c, 0x2f,
    0x44, 0x61, 0x74, 0x65, 0x3b, 0x4c, 0x00, 0x05, 0x73, 0x74,
    0x61, 0x72, 0x74, 0x71, 0x00, 0x7e, 0x00, 0x01, 0x78, 0x70,
    0x73, 0x72, 0x00, 0x0e, 0x6a, 0x61, 0x76, 0x61, 0x2e, 0x75,
    0x74, 0x69, 0x6c, 0x2e, 0x44, 0x61, 0x74, 0x65, 0x68, 0x6a,
    (byte)0x81, 0x01, 0x4b, 0x59, 0x74, 0x19, 0x03, 0x00, 0x00,
    0x78, 0x70, 0x77, 0x08, 0x00, 0x00, 0x00, 0x66, (byte)0xdf,
    0x6e, 0x1e, 0x00, 0x78, 0x73, 0x71, 0x00, 0x7e, 0x00, 0x03,
    0x77, 0x08, 0x00, 0x00, 0x00, (byte)0xd5, 0x17, 0x69, 0x22,
    0x00, 0x78
  };

  public static void main(String[] args) {
    Period p = (Period) deserialize(serializedForm);
    System.out.println(p);
  }

  // Returns the object with the specified serialized form
  static Object deserialize(byte[] sf) {
    try {
      return new ObjectInputStream(
          new ByteArrayInputStream(sf)).readObject();
    } catch (IOException | ClassNotFoundException e) {
      throw new IllegalArgumentException(e);
    }
  }
}
```

`serializedForm`은 바이트 배열을 만들어서 일부를 수정한 것이다.

실행하게 되면 결과는

```
Fri Jan 01 12:00:00 PST 1999 - Sun Jan 01 12:00:00 PST 1984.
```

위와 같이 나오게 된다.


<br>
<br>

이런 문제를 해결하기 위해서는

```java
// readObject method with validity checking - insufficient!
private void readObject(ObjectInputStream s)
        throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // Check that our invariants are satisfied
    if (start.compareTo(end) > 0)
        throw new InvalidObjectException(start +' after '+ end);
}
```

이렇게 하면 앞서 나온 공격은 막을 수 있다.

또 다른 문제가 있다.

```java
public class MutablePeriod {
    // A period instance
    public final Period period;

    // period's start field, to which we shouldn't have access
    public final Date start;

    // period's end field, to which we shouldn't have access
    public final Date end;
    public MutablePeriod() {
        try {
            ByteArrayOutputStream bos =
                new ByteArrayOutputStream();
            ObjectOutputStream out =
                new ObjectOutputStream(bos);

            // Serialize a valid Period instance
            out.writeObject(new Period(new Date(), new Date()));

            /*
             * Append rogue 'previous object refs' for internal
             * Date fields in Period. For details, see 'Java
             * Object Serialization Specification,' Section 6.4.
             */
            byte[] ref = { 0x71, 0, 0x7e, 0, 5 };  // Ref #5
            bos.write(ref); // The start field
            ref[4] = 4;     // Ref # 4
            bos.write(ref); // The end field

            // Deserialize Period and 'stolen' Date references
            ObjectInputStream in = new ObjectInputStream(
                new ByteArrayInputStream(bos.toByteArray()));
            period = (Period) in.readObject();
            start  = (Date)   in.readObject();
            end    = (Date)   in.readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new AssertionError(e);
        }
    }
}
```

```java
public static void main(String[] args) {
    MutablePeriod mp = new MutablePeriod();
    Period p = mp.period;
    Date pEnd = mp.end;

    // Let's turn back the clock
    pEnd.setYear(78);
    System.out.println(p);

    // Bring back the 60s!
    pEnd.setYear(69);
    System.out.println(p);
}
```

```
Wed Nov 22 00:21:29 PST 2017 - Wed Nov 22 00:21:29 PST 1978
Wed Nov 22 00:21:29 PST 2017 - Sat Nov 22 00:21:29 PST 1969
```

결과는 위와 같이 나오게 된다.

기존에 주어진 레퍼런스에 대한 변경으로 객체에 영향을 미치게 된다.

이 이유는 `Period`의 `readObject` 메소드가 방어적 복사를 하고 있지 않기 때문이다.

객체가 deserializae 될 때, 클라이언트가 가질 수 있는 레퍼런스 필드를 방어적으로 카피해야 한다.


<br>
<br>

```java
// readObject method with defensive copying and validity checking
private void readObject(ObjectInputStream s)
        throws IOException, ClassNotFoundException {
    s.defaultReadObject();

    // Defensively copy our mutable components
    start = new Date(start.getTime());
    end   = new Date(end.getTime());

    // Check that our invariants are satisfied
    if (start.compareTo(end) > 0)
        throw new InvalidObjectException(start +' after '+ end);
}
```

유효성 체크 이전에 방어적 카피 하는 것에 주목해야 한다.

방어적 복사는 또한, final 필드에 대해서는 불가능하다.

`readObject` 메소드를 위해서는 `start`, `end` 필드를 모두 nonfinal로 해야한다.

serialization proxy pattern (Item 90)을 사용하면 `readObject`없이도 가능하긴하다.


<br>
<br>

nonfinal serializable 클래스에 적용되는 `readObject` 메소드와 생성자의 또 다른 공통점이 있다.

- 생성자와 같이 `readObject` 메소드는 절대로 override 가능한 메소드를 호출해서는 안된다. (직접적으로든 간접적으로든)
    - 하위클래스에 overriding된 메소드가 하위클래스가 deserialization하기 전에 실행될 수 있다.


<br>
<br>

결론,

`readObject` 메소드를 작성할 때는 public 생성자와 같게 생각해야한다.

- private 필드를 가진 클래스는 방어적 복사를 해야한다. (immutable 클래스의 mutable 컴포넌트)
- 불변성을 확인하고 불변이 아닌 경우에는 `InvalidObjectException`을 던져야 한다.
- 전체 객체 그래프는 deserialize 이후에 유효한지 확인되어야 하면, ObjectInputValidation 인터페이스를 사용하라(이 챕터에서 나오지 않음)
- Overridable method를 `readObject` 메소드에서 호출하지 마라.


<br>
<br>

## Item 89: 객체 제어를 위해서 readResolve보다 enum을 선호하라.

Item 3에서는 싱글톤 패턴에 대해 이야기했다.

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {  ... }

    public void leaveTheBuilding() { ... }
}
```

위 예제는 Item 3에서 이야기 했듯이, `implements Serializable`가 추가되면 더 이상 싱글톤의 역할을 하지 못한다.

이 경우, default serialized form을 사용하던, 

custom serialized form을 사용하던, 

클래스가 `readObject` 메소드를 직접 제공하던, 

결과는 같다.


<br>
<br>

`readObject` 메소드에서는 새롭게 생성된 객체를 리턴한다. 이는 클래스가 생성될 때의 객체와 다르다.

`readResolve` 메소드는 `readObject` 메소드에서 생성된 객체를 다른 객체로 바꿀 수 있다.

`Elvis` 싱글톤 예제에서

```java
// readResolve for instance control - you can do better!
private Object readResolve() {
    // Return the one true Elvis and let the garbage collector
    // take care of the Elvis impersonator.
    return INSTANCE;
}
```
위와 같이 사용하면 싱글톤 프로퍼티를 보장할 수 있다.

이 메소드는 deserialize된 객체를 무시하고, 클래스가 생성될 때 만들어진 `Elvis` 객체를 리턴한다.

`readResolve ` 메소드에 의존하면, 객체 참조 타입의 모든 인스턴스 필드는 `transient`로 정의되어야 한다.

- transient: 멤버 변수가 serialized 되지 않음을 나타낸다. JVM은 본래 값을 무시하고 기본값을 입력한다.

`transient`로 정의되지 않으면 공격자가 `MutablePeriod`의 공격처럼 공격할 수 있다.
- `readResolve` 메소드 호출 이전에 deserialize 되면서 생기는 공격이다. 레퍼런스를 훔칠 수 있게 해준다.


<br>
<br>

```java
// Broken singleton - has nontransient object reference field!
public class Elvis implements Serializable {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { }

    private String[] favoriteSongs =
        { 'Hound Dog', 'Heartbreak Hotel' };
    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }

    private Object readResolve() {
        return INSTANCE;
    }
}
```

```java
public class ElvisStealer implements Serializable {
    static Elvis impersonator;
    private Elvis payload;

    private Object readResolve() {
        // Save a reference to the 'unresolved' Elvis instance
        impersonator = payload;

        // Return object of correct type for favoriteSongs field
        return new String[] { 'A Fool Such as I' };
    }
    private static final long serialVersionUID = 0;
}
```

```java
public class ElvisImpersonator {
  // Byte stream couldn't have come from a real Elvis instance!
  private static final byte[] serializedForm = {
    (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x05,
    0x45, 0x6c, 0x76, 0x69, 0x73, (byte)0x84, (byte)0xe6,
    (byte)0x93, 0x33, (byte)0xc3, (byte)0xf4, (byte)0x8b,
    0x32, 0x02, 0x00, 0x01, 0x4c, 0x00, 0x0d, 0x66, 0x61, 0x76,
    0x6f, 0x72, 0x69, 0x74, 0x65, 0x53, 0x6f, 0x6e, 0x67, 0x73,
    0x74, 0x00, 0x12, 0x4c, 0x6a, 0x61, 0x76, 0x61, 0x2f, 0x6c,
    0x61, 0x6e, 0x67, 0x2f, 0x4f, 0x62, 0x6a, 0x65, 0x63, 0x74,
    0x3b, 0x78, 0x70, 0x73, 0x72, 0x00, 0x0c, 0x45, 0x6c, 0x76,
    0x69, 0x73, 0x53, 0x74, 0x65, 0x61, 0x6c, 0x65, 0x72, 0x00,
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x02, 0x00, 0x01,
    0x4c, 0x00, 0x07, 0x70, 0x61, 0x79, 0x6c, 0x6f, 0x61, 0x64,
    0x74, 0x00, 0x07, 0x4c, 0x45, 0x6c, 0x76, 0x69, 0x73, 0x3b,
    0x78, 0x70, 0x71, 0x00, 0x7e, 0x00, 0x02
  };

  public static void main(String[] args) {
    // Initializes ElvisStealer.impersonator and returns
    // the real Elvis (which is Elvis.INSTANCE)
    Elvis elvis = (Elvis) deserialize(serializedForm);
    Elvis impersonator = ElvisStealer.impersonator;

    elvis.printFavorites();
    impersonator.printFavorites();
  }
}
```

이렇게 공격이 가능하다.


<br>
<br>

`favoriteSongs` 필드를 transient로 하면 해결할 수 있다.

그렇지만 Elvis를 single-element enum type으로 바꾸는 편이 더 낫다.

- 자바가 정의된 상수외에는 다른 인스턴스를 생성하지 못하게 해준다.
- 공격자가 `AccessibleObject.setAccessible`와 같은 메소드를 사용한다면, 

```java
// Enum singleton - the preferred approach
public enum Elvis {
    INSTANCE;
    private String[] favoriteSongs =
        { 'Hound Dog', 'Heartbreak Hotel' };
    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
}
```

인스턴스의 갯수를 컴파일 타임에 모른다면 Enum 타입으로 하기 힘들다.


<br>
<br>

`readResolve` 메소드의 접근자(accessibility)는 매우 중요하다.

- `readResolve` 메소드를 final 클래스에 둔다면, private이어야 한다.
- nonfinal 클래스라면, 접근자를 더 깊게 고려해야한다.
    - private이면, 어떤 하위 클래스에도 적용할 수 없다.
    - package-private이면, 같은 패키지에 있는 하위 클래스에만 적용할 수 있다.
    - protected or public이면, `readResolve` 메소드를 override하지 않은 모든 하위클래스에 적용된다.
    - protected or public이고 하위 클래스가 ovveride하고 있지 않으면, 하위클래스 객체는 상위 클래스 객체를 생성하면서 `ClassCastException`를 발생 시킬 수 있다.
    
결론,

enum type을 불변 객체 제어를 위해서 사용하자.

<br>
<br>

## Item 90: serialized instances 대신 serialization proxy를 고려하라.

`Serializable`를 구현하는 것은 버그와 보안 문제가 생길 가능성을 높인다는 점을 이 챕터 내내 이야기 했다.

보이지 않는 생성자가 생기면서 발생하는 문제가 대부분이다.

serialization proxy pattern을 사용하면 가능하다.

<br>
<br>

serialization proxy pattern 만드는 법
- 감싸고 있는 클래스의 logical state을 저장하는 객체를 private static nested class에 설계
- 이 nested 클래스가 serialization proxy로 역할한다.
    - 파라미터가 감싸고 있는 클래스인 생성자가 필요하다.
    - 일관성 체크나 방어적 복사는 따로 하지 않아도 된다.
    - proxy의 default serialization from은 완벽히 감싸고 있는 클래스에 적용할 수 있다.
- Serialize proxy하고 감싸고 있는 클래스는 각각 `Serializable`을 구현해야한다.


<br>
<br>

```java
// Serialization proxy for Period class
private static class SerializationProxy implements Serializable {
    private final Date start;
    private final Date end;

    SerializationProxy(Period p) {
        this.start = p.start;
        this.end = p.end;
    }

    private static final long serialVersionUID =
        234098243823485285L; // Any number will do (Item  87)
}
```

앞선 예제의 `Period` 클래스에 대해서 proxy를 적용하면 위와 같다.

```java
// writeReplace method for the serialization proxy pattern
private Object writeReplace() {
    return new SerializationProxy(this);
}
```

`writeReplace` 메소드를 위와 같이 추가하면 된다.

감싸고 있는 클래스말고 private 클래스를 리턴하게 된다. `writeReplace` 메소드는 감싸고 있는 객체를 private nested 클래스로 변환시켜준다.

```java
// readObject method for the serialization proxy pattern
private void readObject(ObjectInputStream stream)
        throws InvalidObjectException {
    throw new InvalidObjectException("Proxy required");
}
```

readObject 메소드는 불필요하기 때문에 공격을 방지하기 위해서 위와 같이 Exception을 던지도록 하면 된다.

마지막으로 `SerializationProxy` 클래스에 `readResolve` 메소드를 제공해서 감싸고 있는 클래스와 논리적으로 동등한 클래스를 리턴하도록 하면 된다.
- 이 메소드가 있어서 proxy객체를 다시 감싸고 있는 클래스(enclosing class)로 바꿀 수 있다.
- 생성자나 static factory method를 그대로 사용할 수 있어서, 클래스의 불변을 깨지 않는다.

```java
// readResolve method for Period.SerializationProxy
private Object readResolve() {
    return new Period(start, end);    // Uses public constructor
}
```


<br>
<br>

다른 방법보다 나은 점
- 필드가 final로 정의될 수 있어서, 레퍼런스 탈취등에 대한 공격을 막기 쉽다.
- 본래 serialized 객체말고 다른 클래스를 가질 수 있다.
    - `EnumSet`은 생성자 없이 static factory만 있다. 
        - 클라이언트에서는 `EnumSet`을 리턴하지만, OpenJDK 구현체는 갯수에 따라서 64개를 기준으로 작으면 `RegularEnumSet` 많으면 `JumboEnumSet`을 리턴한다.
        - 60개의 원소를 가진 `EnumSet`을 serialize한다고 하면, 5개 원소가 추가되고 안되냐에 따라서 구현체가 달라지게 된다.
        
            ```java
            // EnumSet's serialization proxy
            private static class SerializationProxy <E extends Enum<E>>
                    implements Serializable {
                // The element type of this enum set.
                private final Class<E> elementType;
            
                // The elements contained in this enum set.
                private final Enum<?>[] elements;
            
                SerializationProxy(EnumSet<E> set) {
                    elementType = set.elementType;
                    elements = set.toArray(new Enum<?>[0]);
                }
            
                private Object readResolve() {
                    EnumSet<E> result = EnumSet.noneOf(elementType);
                    for (Enum<?> e : elements)
                        result.add((E)e);
                    return result;
                }
            
                private static final long serialVersionUID =
                    362491234563181265L;
            }
            ```
          
        - 위와 같이 Proxy를 사용하면 간단하게 만들 수 있다.
    
    

<br>
<br>

proxy 패턴의 2가지 한계점
- 유저가 상속해서 사용할 수 있는 클래스에는 호환가능하지 않다.
- Circular 객체 의존 관계인 경우에는 안된다.
    - `ClassCastException`이 발생한다. 왜냐하면 아직 필요한 객체가 만들어지지 않았기 때문이다.


<br>
<br>

방어적 복사로 serialize하는 것에 비해서 성능 상 더 뛰어나지는 못하다. 저자의 컴퓨터에서 14퍼센트 정도 더 느렸다.


<br>
<br>


결론,

적절한 조건에서 serialization proxy pattern을 사용하면 매우 유용하게 사용될 수 있다.
