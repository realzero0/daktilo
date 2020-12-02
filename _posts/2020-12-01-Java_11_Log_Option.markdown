---
layout: post
title:  "Java 11 Log Option 정리"
subtitle: "Java8에서 달라지는 Java11 Log Option"
date:   2020-12-01 10:47:00
categories: [study]
---

OpenJDK11은 OpenJDK8 이후의 첫번째 LTS(긴 기간 지원해주는)버전이다.
이 버전은 JVM 로그 설정이 많이 바뀌었다. 표준화(standardization) 및 단일화(unification)되는 방향으로 바뀌었다.

## JVM Log label

보통 자바 코드로 작성된 코드는 아래와 같다.

```java
Logger logger = LogFactory.getLooger("core-logger");
logger.info("this is core logger log");
```

```java
2020-02-05 10:50:52.670  INFO [core-logger] [22] [pool-13-thread-1]: this is core logger log
```

실제 결과도 위와 같이 나온다.ㅏ