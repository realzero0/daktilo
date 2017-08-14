---
layout: post
title:  "[Boost_Camp] BE FE 코드리뷰"
subtitle: "7주차"
date:   2017-08-14 20:40:00
categories: [study]
---

# FE #

parents / closest


element를 기준으로 find해야 "여러 인스턴스를 생성하도록 허용할 수 있다."

비동기 호출 = setTimeout을 이용. setTimeout(func, 0)일 경우, 대략적으로 16ms 정도의 시간이 들고 비동기 호출이 가능하다.




thenable object만 Enable하다??

thenable -> then이 가능한 객체 -> 객체 안에 then이라는 객체가 있는 객체 -> 대표적으로 promise이다.

promise는 A타입 A+타입, B타입이 있다.

Deffered object(AJAX가 리턴하는)도 thenable 객체이다.

blue bird 또한 thenable이다.


```
requestAnimationFrame(function() {
	callback(data);
});
```

평균적으로 브라우저는 60프레임을 유지한다.	

16.8ms마다 1개의 프레임을 보내준다. 이 시간도 메인 스레드가 실행되는 상황에 따라 달라진다.

16ms이전에 변경된 내용은 거의 반영되지 않기 때문에, requestAnimationFrame은 변경되야할 시점에 시행될 수 있게 해준다.


# BE #

MVC에 대한 이해, RESTFUL API에 대한 심화 이해

Spring security login interceptor(authentification, authorization)

단위 테스트에 대한 심화 작성

view에 대한 framework(tiles, thymeleaf 2.xx)

transaction(@transactional, 좀 더 추가적인 트랜젝션에 대한 공부가 필요함)

mybatis(ORM, sql mapper, jpa)

기본적인 것들을 잘 이해하는 것이 중요

유효성 체크를 잘해야한다. spring validation을 활용하면 편리하게 사용할 수 있음

property 주입, 생성자 주입, setter 주입

spring이기 때문에 property에 주입이 가능하다. 그런데 spring이 아니면 종속성을 주입할 수 있는 방법이 없다.

문제 해결 후 책으로 지식을 쌓아가는 것이 좋다.

XSS(cross site scripting) 게시물 내부에 스크립트를 심어서 사용자의 정보를 빼내거나 불필요한 파일을 다운로드 받게 하는 것



restful api의 위치에 대해 생각해볼 것

uri(Uniform Resource Identifier) > url


api/products/1/categories/1 ->

큰 부분에서부터 작은 부분으로

get/categories/1/products


page는 쿼리 파라미터로 붙이는게 보통임

에러 exception advice

적절한 값으로 바꿀 것, null, 익셉션 발생시켜야 함. 익셉션 정보를 유지해야한다.

aws cloud watch라는 서비스를 이용하면 log error를 알림을 줄 수 있다.

sql에서 제일 좋은 쿼리는 조건을 최소한의 데이터만 가져오게끔하는 것.

원본 테이블에서 줄여가는 것이 좋음

explain이라는 예약어를 수행해보면 좋음 derived 테이블을 작성하지 않는 것이 좋음



HttpServletResponse가 서비스단으로 내려올 때를 어떻게 결정할 것인가

Service가 service를 호출하는 경우도 있음


@Transactional의 여러 옵션들에 대해 알아보기

message converter

add interceptor, add messageconverter

add argumentresolver -> 처리할 수 있는 애들을 불러서 있는 애가 있으면 처리해줌 viewResolver도 여러개 등록할 수 있음


restful api를 사용하면 http status 코드를 이용하기도 많이 함.

return


@PathVariable에 정규식 쓸 수 있음
{productId:[0-9]+}


logger가 static선언을 하는 이유는 옛날에는 service가 싱글톤이 아니었음

싱글톤이 아닐 경우에는 static이 필요없음

빈 앞에다가 @Scope 등을 사용하면 싱글톤이 아님, 세션별, 리퀘스트별, 매번 생성하는 경우가 있음

in action, 강컴닷컴 -> 공부하기 좋은 책들

java8, spring 기본 원리 책들이 필요함.



DB 확장 Scale out Scale in

db는 scale out하는게 너무 힘들다.

그래서 db에는 가급적 일을 안시킨다.