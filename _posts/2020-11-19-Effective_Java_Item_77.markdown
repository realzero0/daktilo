---
layout: post
title:  "Effective java item 77"
subtitle: "이펙티브 자바 정리"
date:   2020-11-19 10:47:00
categories: [study]
---

API 설계자가 exception을 던지게 만들었다는 것은 뭔가 프로그래머에게 전달할 무엇인가가 있는 것이다. 무시해서는 안된다.

```java
// Empty catch block ignores exception - Highly suspect!
try {
    ...
} catch (SomeException e) {
}
```

위와 같이 무시하는 것은 매우 쉽다.

아무것도 없는 catch block은 exception의 목적을 잃게 한다. 비유하자면 집에 불이 났다는 알람을 무시하는 것과 비슷하다. 아무일도 없을 수 있지만, 진짜 불일 경우에는, 문제가 된다.

exception을 무시해도 되는 상황도 있다.

- FileInputStream을 닫을 때는 적절할 수 있다.
    - 파일의 상태를 바꾸지 않으면, recovery 액션을 할 이유가 없다.
    - 그리고 이미 파일에서 필요한 정보를 얻었다면 close exception 때문에 실제 메소드를 중단할 필요가 없다.
    - 그렇지만 로깅해서 예외 상황이 실재로 생길 때 조사하기 편하게 하는 편이 낫다.

```java
Future<Integer> f = exec.submit(planarMap::chromaticNumber);
int numColors = 4; // Default; guaranteed sufficient for any map
try {
    numColors = f.get(1L, TimeUnit.SECONDS);
} catch (TimeoutException | ExecutionException ignored) {
    // Use default: minimal coloring is desirable, not required
}
```

위와 같이 exception을 무시한다고 하면, 왜 무시해도 되는지에 대한 코멘트를 달아주어야 한다.

결론,

checked이든 unchecked이든 어떤 예외도 무시하게 되면 실제 문제를 숨길 수 있다. 꼭 필요한 경우에만 무시하고, 무시하는 경우에도 로그는 남겨둬서 나중에 조사하기 편하게 하자.