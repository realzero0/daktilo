---
layout: post
title:  "Effective java item 68"
subtitle: "이펙티브 자바 정리"
date:   2020-11-12 16:47:00
categories: [study]
---



Java Language Specification (JLS, 6.1)에는 잘 만들어진 네이밍 컨벤션이 있다. 네이밍 컨벤션은 2가지 카테고리로 나눌 수 있다. 

- 활자그대로, (typographical)
- 문법적으로 (grammatical)

활자그대로의 네이밍 컨벤션은 꽤 많다. packages, classes, interfaces, methods, fields, and type variables에 걸쳐서 있다. 절대 위반해서는 안되고, 어기게 되면 API가 사용하기 어려워지게 된다. 사용하는 클라이언트가 잘못된 추측으로 에러가 생길 수도 있다.

- 패키지와 모듈 이름은 계층구조이어야 한다. 각각은 점(.)으로 구분되어야 한다.
    - 컴포넌트들은 소문자여야 하고, 숫자도 허용된다.
    - 외부에서 사용해야하는 패키지는 조직의 인터넷 도메인의 반대 순서로 시작해야 한다(com.kakaobank)
        - java, javax같은 표준 라이브러리는 예외다.
    - JLS, 6.1을 보면 자세히 나와있다.
    - 각각의 컴포넌트는 짧고(8글자이내), 의미있는 줄임말(util)을 사용하면 좋다.
    - 의미있는 계층구조를 가지면 좋고, 그 구조에 따라 파생될 수 있다(예, java.util.concurrent.atomic)
- 클래스와 인터페이스(enum, annotation type포함)
    - 1개 이상의 단어 그리고 각 단어의 1번째 글자는 대문자여야 한다. (List or FutureTask)
    - 줄임말은 사용하지 않는편이 낫다. (mix, max 처럼 정말 자주 사용되는 것이 아닌 이상)
        - 줄임말이 전부 대문자여야하는지에 대해서는 본래대로 사용하는게 구분에 더 용이하지 않을까? (HTTPURL or HttpUrl)
- 메소드와 필드이름
    - 클래스, 인터페이스와 같지만 첫글자는 소문자 (예 remove or ensureCapacity)
    - 줄임말이 첫 단어로 나오면 소문자로 시작해야한다(자주나오는 min, max 같이)
    - constant 필드의 경우에는 전부 대문자에 underscore _로 단어를 구분한다. (예 VALUES or NEGATIVE_INFINITY)
        - static final 필드여야 한다
- 지역변수
    - 완전 줄인 이름을 사용해도 된다. (예 i, denom, houseNum)
    - 메소드 파라미터의 경우에는 더 신경써서 이름을 만들어야 한다. method documentation에서 뗄 수 없는 부분이기 때문이다.
- Type 파라미터
    - T는 아무 타입
    - E는 element 타입
    - K, V는 key value 타입
    - X는 exception
    - R은 리턴 타입
    - 아무타입을 나열할 경우 T, U, V또는 T1, T2, T3로 할 수 있다.

문법적 네이밍 컨벤션

- 유연하고 논란이 더 많다.

- 패키지
    - 없다
- 객체화 가능한 클래스(enum 포함)
    - 명사나 명사구로 만든다 (Thread, PriorityQueue, or ChessPiece)
- 객체화가 안되는 유틸리티 클래스
    - 보통 복수로 표현된다 (예 Collectors or Collections)
- 인터페이스
    - 클래스처럼 네이밍 된다(Collection or Comparator)
    - 형용사로 끝낼 수 있다 (able or ible를 뒤에 붙여서, 예 Runnable, Iterable, or Accessible)
- 어노테이션 타입
    - 여러 부분에 다 사용되기 때문에 명사 동사 전치사 형용사 다 가능하다
    - 예 BindingAnnotation, Inject, ImplementedBy, or Singleton
- 메소드
    - 동사나 동사구로 사용된다. (예 append or drawImage)
    - boolean을 리턴하는 메소드는 is로 시작한다. has도 사용되기도 한다 (isDigit, isProbablePrime, isEmpty, isEnabled, or hasSiblings)
    - non-boolean이나 객체의 속성을 리턴하면 명사, 명사구, get으로 시작하는 동사구로 사용될 수 있다. (size, hashCode, or getTime)
        - get으로 시작하는 형태는 주로 자바빈즈(JavaBeans) 명세에 뿌리를 두고 있다.
        - 보통 getter/setter의 한 묶음 형태로 만드는 경우가 많다.
    - 일부 메소드는 특별한것을 위해서 사용되기도 한다
        - 타입을 또다른 타입으로 반환하는 경우에는 **toType** 의 형태로 짓는다.
            - toString, toArray 등..
        - 객체의 내용을 다른 뷰로 보여주는 메서드는 **asType** 의 형태로 짓는다.
            - asList, asMap 등..
        - 객체의 값을 기본 타입(primitive type)으로 반환하는 경우에는 **typeValue**의 형태로 짓는다.
            - intValue, longValue 등...
    - 스태틱 팩토리 메소드는 다양하다(item 1)
        - from, of, valueOf, instance, getInstance, newInstance, getType, and newType
- 필드
    - 필드에 대한 문법적 컨벤션은 확실히 정해진 것도 아니고 class, interface, and method names보다  덜 중요하다. 잘 디자인된 API는 필드가 공개되어 있지 않기 때문이다.
    - boolean 타입일 경우 initialized, composite처럼 is가 생략되기도 한다.
    - 다른 타입인 경우, 명사 또는 명사구로 만들어진다 (height, digits, or bodyStyle)
    - 지역변수의 경우에는 필드와 비슷하지만 반드시 지킬 필요는 없다.

결론,

표준 네이밍을 잘 사용하도록 하자. Java Language Specification에는 계속해서 사용해야 컨벤션의 의미가 생긴다고 한다.