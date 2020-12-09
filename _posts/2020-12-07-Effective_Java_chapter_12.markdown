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



