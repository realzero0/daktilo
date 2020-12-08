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

