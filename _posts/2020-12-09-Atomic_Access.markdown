---
layout: post
title:  "Java Atomic Access"
subtitle: "Java에서 Atomic하다는게 무엇인가.."
date:   2020-12-09 15:47:00
categories: [study]
---

일반적으로 우리는 여러 용어를 혼용해 사용한다.

자바에서 Synchronized 키워드가 사용된 블록은 
- 메모리 참조에 있어서 가장 최신의 메모리를 참조하고(CPU 캐시 등에 저장된 내용이 아닌) : 동기화 문제
- 오퍼레이션에 있어서 1개 블록이 atomic한 1개의 오퍼레이션으로 인식된다. : Atomic (CPU의 오퍼레이션 상으로는 Atomic은 아니다)
- 쓰레드간의 경합에 있어서 해당 블록이 먼저 사용된(락을 먼저 획득한) 쓰레드가 우선적으로 독점적(exclusive)으로 자원을 사용할 수 있게 해준다. : 쓰레드 경합

synchronized 블록을 사용할 경우 앞서 언급한 3가지 점이 모두 해결된 상태를 얻을 수 있다.

<br>
<br>

Atomic?
- Atomic하다는 것은 1번만 일어난다는 것을 의미한다.
- 도중에 멈출 수 없다. -> 완전히 일어나던지, 아예 일어나지 않던지 두가지 중 하나밖에 안된다.
- Atomic 액션의 사이드 이펙트는 Atomic 액션이 끝날때까지 나타나지 않는다.

a++ 등과 같은 오퍼레이션은 atomic action이 아니다. 복잡한 오퍼레이션이다.

그렇다면 자바에서 아무런 키워드 없이 atomic action인 경우는 무엇인가?
- reference variable과 primitive variable(참조형과 기본형, long과 double은 아니다.)에 대한 읽기, 쓰기는 atomic하다.
    - long, double은 64bit이기 때문에 1개 오퍼레이션으로 저장 불가능하다.
- volatile 키워드로 정의된 변수는 long, double이라고 하더라도 읽기, 쓰기시 atomic하다.
    - volatile이면 객체에 대한 참조를 캐시에 의존하지 않고 메모리에 직접 의존하게 된다.


<br>
<br>

atomic action의 특징
- 쓰레드간 경합이 불가능하다. 한번에 1개 쓰레드만 action할 수 있다.
- 그렇다고 해도, synchronized atomic action이 불필요 한 것은 아니다.
    - 메모리 일관성(consistency) 에러는 발생할 수 있기 때문이다.
        - volatile 키워드를 사용하면 일관성 에러를 줄일 수 있다.
            - volatile을 사용한 변수는 모든 쓰레드에 항상 보이기 때문이다.
            - 특정 쓰레드가 volatile 변수를 읽는다면, 해당 변수의 최신 상태를 읽을 수 있을 뿐 아니라, 사이드 이펙트까지 함께 보게 된다는 것을 의미한다.
            
        
간단한 atomic 변수 접근을 사용하면 synchronized 키워드를 사용하는 것보다 훨씬 효율적이다.
- lock이 없어지면서 굉장히 효율적일 수 있다.

다만, CPU 캐시 등에 저장되어서 최신 변수 상태를 가져오지 못하는 문제에 대해서는 주의가 필요하다.


출처: https://docs.oracle.com/javase/tutorial/essential/concurrency/aatomic.html