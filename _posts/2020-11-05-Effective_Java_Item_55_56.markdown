---
layout: post
title:  "Effective java item 55, 56"
subtitle: "이펙티브 자바 정리"
date:   2020-11-05 10:47:00
categories: [study]
---

## Item 55: Optional을 신중하게 리턴하라.

Java 8 이전에는 특정 상황에서 값을 리턴하지 못하는 메소드를 작성할 때 선택할 수 있는 것은 2가지이다. null을 리턴하던지, exception을 던지던지. 

Exception은 예외 상황을 위해서만 남겨둬야 한다. (Item 69) Exception을 던지는 것은 매우 비싸다. 왜냐하면 전체 stacktrace가 execption이 생성될 때 잡혀야 하기 때문이다. 

null을 리턴하는 것은 위의 문제가 없다. API 메소드에서 null이 리턴되면 클라이언트 메소드에서는 null을 처리하는 로직이 필요하게 된다. 그게 아니면 프로그래머가 null을 리턴하는 것이 불가능하다는 것을 증명해야한다.

이렇게 하지않으면 NullPointerException이 나게 된다...

Java8이 등장한 뒤에 3번째 방법이 등장했다. Optional<T> 클래스는 하나의 non-null값이나 아무것도 안가질 수 있는 불변(Immutable) 컨테이너이다. Collection<T>을 상속하고 있지는 않지만 1개의 element만 가질 수 있는 불변 컬렉션으로 볼 수도 있다.

Optional<T>로 리턴하는 메소드는 값을 리턴하지 않을 수 있기 때문에 좀 더 유연성을 가지게 된다.

```java
// Returns maximum value in collection - throws exception if empty
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        throw new IllegalArgumentException("Empty collection");

    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);

    return result;
}
```

이 메소드는 콜렉션이 비어있을 때 IllegalArgumentException을 던진다.

```java
// Returns maximum value in collection as an Optional<E>
public static <E extends Comparable<E>>
        Optional<E> max(Collection<E> c) {
    if (c.isEmpty())
        return Optional.empty();
        
    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);

    return Optional.of(result);
}
```

적절한 static factory만 사용하면 끝이다.. 이상황에서 잘못 Optional.of(null)이 호출되게 되면 NullPointerException이 발생하게 된다. Optional.ofNullable(value)를 사용해야 null 값일 때 empty Optional이 리턴된다. 

Optional이 리턴되는 메소드에서는 null 값이 리턴되서는 안된다. 그러지 않으면 Optional 기능의 목적을 잃게 된다.

스트림에 있는 많은 종단 연산(terminal operation)들도 optional을 리턴한다.

```java
// Returns max val in collection as Optional<E> - uses stream
public static <E extends Comparable<E>>
        Optional<E> max(Collection<E> c) {
    return c.stream().max(Comparator.naturalOrder());
}
```

위와 같이 max값을 구하는 메소드를 stream으로 구현하면 optional을 얻을 수 있다.

Optional은 checked exception과 비슷하다. API를 사용하는 유저가 강제로 그 exception을 처리해야만 한다. unchecked exception을 던지던지, null을 리턴하게 되면 유저가 완전히 무시하게 할수도 있다. checked exception을 던지는 것은 client의 boilerplate 코드를 늘린다.

```java
// Using an optional to provide a chosen default value
String lastWordInLexicon = max(words).orElse("No words...");
```

empty optional이 리턴될 수 있다면 기본값을 설정해주던, exception을 던지든 위와 같은 코드가 필요하다.

```java
// Using an optional to throw a chosen exception
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```

Exception을 직접 만들어서 던지게 되면 비용을 조금을 줄일 수 있다.

```java
// Using optional when you know there’s a return value
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```

값이 있다고 가정한다면, 위와 같이 작성가능하고, 만약에 Optional이 empty라면 NoSuchElementException이 발생한다.

기본값을 얻는데 비용이 많이 드는 경우(계산이 필요한 경우)에 orElseGet(Supplier<T>)를 제공해주고 있다. 

filter, map, flatMap, and ifPresent 메소드를 제공하고 있고, 자바9에는 or and ifPresentOrElse도 제공하고 있다.

 Optional은 isPresent() 메소드를 제공하고 있고 매우 유용하다. 위에 소개된 메소드들과 잘 비교하면서 사용할 필요는 있다.

```java
Optional<ProcessHandle> parentProcess = ph.parent();
System.out.println("Parent PID: " + (parentProcess.isPresent() ?
    String.valueOf(parentProcess.get().pid()) : "N/A"));
```

```java
System.out.println("Parent PID: " +
  ph.parent().map(h -> String.valueOf(h.pid())).orElse("N/A"));
```

위를 잘 비교해봐야 한다.

Stream<Optional<T>> 스트림에서 Stream<T>로 모든 element가 nonempty optional일 필요가 있는 경우도 있다.

```java
streamOfOptionals
    .filter(Optional::isPresent)
    .map(Optional::get)
```

그 경우에는 위와 같이 사용할수 있다.

Java9에서는 Optional이 stream() 메소드를 제공한다. element가 존재하는 경우만 stream으로 흘려보내준다.

```java
streamOfOptionals.
    .flatMap(Optional::stream)
```

Container 타입인 경우에는 Optional을 사용해서는 안된다

- collections, maps, streams, arrays, and optionals
- Optional<List<T>>이렇게 사용하지 말고 빈 List<T>를 리턴하면 된다.(item 54)
    - 이렇게 리턴하면 optional을 처리하기 위해서 생기는 boilerplate 코드가 없어진다.

언제 T말고 Optional<T>를 사용해야 하는가?

- 리턴값이 없을 수 있는 메소드를 작성할 때
- 리턴값이 없는 경우 클라이언트에서 특별한 프로세싱이 필요할 때

Optional의 문제점들

- Optional도 객체이므로 생성 비용이 발생한다.
- Optional 객체의 값에 접근할 때도 간접 접근이다.
- 아주 성능이 중요한 순간에는 부적절할수도 있다.
- Boxed primitive type인 경우 성능 차이가 매우 심하게 난다. 2단계 boxing이 필요하기 때문..
    - primitive type에 사용해야하면 OptionalInt, OptionalLong, and OptionalDouble를 사용하자
    - 거의 대다수의 Optional에 있던 메소드는 구현되어 있다.
    - boxed primitive인 경우에는 절대 사용하지 말자.

사용하지 말아야 할 경우

- Map의 key사용하면 key 2개로 사용하는 것과 같다.
- Optional을 key, value, element로 사용하면 안된다.
    - sort, 등 여러 면에서 성능 저하 및 복잡성 증대만 일으키기 때문이다.

- Instance 필드에 Optional을 두는 것은 고민해봐야 한다.
    - bad smell이 나는 코드가 될 가능성이 높다.
    - 필요할 수도 있다.
        - 영양 성분 예시 NutritionFacts(item 2)에서
            - 모든 영양성분을 가지고 있게 할 수 없다.
            - 모든 필드는 primitive type이고 없다는 것을 직접 표현하기가 힘들다.
            - getter메소드에서 Optional 팔드가 리턴되게 할 수 있다.
            - 이런 경우에는 가능할 수도 있다.

결론

값을 항상 리턴하지 않는 메소드가 필요한 경우 Optional을 사용할 수 있다. 컬렉션에 사용될지, Container 객체인지, 성능에 영향을 없는지 잘 살펴볼 필요는 있다.

## Item 56: 모든 공개된 API에 doc 코멘트를 달아라

API가 사용가능하려면, 문서화되어야 한다. 전통적으로 수동으로 관리되어왔다. Javadoc 유틸리티가 등장하면서 코멘트를 이용해서 이를 관리할 수 있게 되었다.

Java 9, {@index}

Java 8, {@implSpec}

Java 5, {@literal} and {@code}

위의 태그들이 추가되어 왔다.

공개된 class, interface, constructor, method, and field declaration은 doc 코멘트를 달아야 한다.

클래스가 serializable하다면 serialized form을 문서화 해야한다(item 87)

문서화 없이는 API를 사용하기가 힘들고, 매우 에러가 생기기 쉽다. Public API는 javadoc을 제공할 수 없기 때문에 default 생성자를 사용하면 안된다.

또한 코드가 지속적으로 유지보수 되기 위해서는 공개되지 않은 클래스, 인터페이스, 생성자, 메소드, 필드에 대해서도 doc 코멘트를 작성해야 한다.

메소드의 doc comment

- 간결하게 method와 클라이언트를 표현해야 한다.
- 메소드가 뭘하는지 표현해야한다. (어떻게 X)
- 메소드가 실행되기 전의 전제조건(precondition)을 미리 나열해야한다.
    - unchecked exceptions인 경우 간접적으로 @throws 태그를 통해 일반적으로 표현된다.
    - @param 태그로도 표현된다.
- 메소드가 실행되고 난 후행조건(postcondition)을 나열해야한다.
- side-effect를 문서화 해야한다. (postcndition을 획득하기 위해서는 필요하지는 않은 시스템에서 발생할 수 있는 변화)
    - 예) background 쓰레드를 실행시킨다
- @param은 모든 파라미터, @return은 void가 아닌 경우 리턴타입, @throws는 모든 exception(checked or unchecked)
    - @return 태그의 내용이 메소드 설명과 같다면 생략가능
    - @param이나 @return은 보통 명사구로 표현됨

javadoc은 html로 코멘트들을 변환한다.

```java
/**
 * Returns true if this collection is empty.
 *
 * @implSpec
 * This implementation returns {@code this.size() == 0}.
 *
 * @return true if this collection is empty
 */
public boolean isEmpty() { ... }
```

위와 같이 implSpec을 이용해서 상속해서 사용했을 때에 관한 코멘트를 남길 수 있다.

```java
/**
* A geometric series converges if {@literal |r| < 1}.
**/
```

{@literal} 태그로 < > &를 표현할수 있다.

doc 코멘트는 코멘트로도 html로도 읽기 편해야 한다.

첫줄은 전체적인 요약을 나타내고, 같은 description을 가진 것들은 만들면 안된다.

기간 같은게 포함되면 주의해야한다.

```java
/**
* This method complies with the {@index IEEE 754} standard.
**/
```

Java9부터는 index를 나타낼 수 있는 태그가 생겼다. 클라이언트가 엄청 큰 API를 찾아갈 때 유용하게 사용될 수 있다.

제네릭의 경우 모든 type parameter를 표시해야한다.

```java
/**
 * An object that maps keys to values.  A map cannot contain
 * duplicate keys; each key can map to at most one value.
 *
 * (Remainder omitted)
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 */
public interface Map<K, V> { ... }
```

enum타입의 경우에는

```java
/**
 * An instrument section of a symphony orchestra.
 */
public enum OrchestraSection {
    /** Woodwinds, such as flute, clarinet, and oboe. */
    WOODWIND,

    /** Brass instruments, such as french horn and trumpet. */
    BRASS,

    /** Percussion instruments, such as timpani and cymbals. */
    PERCUSSION,

    /** Stringed instruments, such as violin and cello. */
    STRING;
}
```

위와 같이 한줄씩 달수도 있다.

annotation 타입의 경우에는

```java
/**
 * Indicates that the annotated method is a test method that
 * must throw the designated exception to pass.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
     /**
      * The exception that the annotated test method must throw
      * in order to pass. (The test is permitted to throw any
      * subtype of the type described by this class object.)
      */
    Class<? extends Throwable> value();
}
```

타입과 member에 다 문서화가 필요하다.

Package-level의 코멘트는 package-info.java에 작성할 수 있다.

module-level의 코멘트(item 15, java9)는 module-info.java에 작성할 수 잇다.

종종 무시되는 문서화는 thread-safety와 serializablility에 관한 것이다.

메소드가 thread-safety한지에 관계없이 thread-safety에 관해 문서화 해야한다. (item 82)

클래스가 serializable하다면, serialized form을 문서화 해야한다(item 87)

Javadoc은 method 코멘트를 상속할 수 있다. {@inheritDoc} tag를 사용하면, 상위 클래스 코멘트의 일부를 가져올 수 있다. 복사하기보다는 상속해서 사용하는 것이 가능하다는 의미이다. 나중에 유지보수의 비용을 줄여줄수 있다.

복잡한 API일 경우에는 코멘트 뿐 아니라 다른 형태의 문서화도 필요하다(아키텍처 그림 등)

Javadoc은 자동으로 이 챕터 내용을 체크할 수 있는 기능이 있다. Java7에서는 -Xdoclint 옵션으로 가능했고, Java 8 and 9은 자동으로 이 기능이 켜져있다.

checkstyle같은 plugin도 도움이 될 수 있다. Html의 결과를 W3C-validator 확인해 볼 수도 있다.

이미 만들어진 다른 코멘트를 많이 보는 것이 도움이 된다.

결론

문서화는 공개된 API에는 필수적으로 해야하고, 가장 쉽게 클라이언트에게 API기능을 알려줄 수 있는 방법이다.