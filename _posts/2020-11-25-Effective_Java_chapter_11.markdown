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

성는 향상을 위해서 원자적 데이터를 읽고 쓸때 동기화를 생략할 수 있다고 생각할 수 있다.