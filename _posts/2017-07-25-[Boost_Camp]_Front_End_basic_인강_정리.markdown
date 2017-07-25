---
layout: post
title:  "[Boost_Camp] Front End basic 인강 정리"
subtitle: "4주차"
date:   2017-07-25 14:40:00
categories: [study]
---

# 사전지식 Jquery > $ #

기본 컨셉(Jquery)

문법

![](/assets/img/jquery1.png)

돔을 찾고 행위를 써내려감

메서드 사용 패턴 : 엘리먼트를 찾고 원하는 메소드를 찾아 사용


setter/getter의 구분

- 파라미터, 인자의 갯수에 따라 setter로 동작(Object를 이용해서 여러개를 동시에 Key, value를 넣는 경우도 있음 css의 경우)
- 파라미터, 인자의 갯수에 따라 getter로 동작(가장 첫번째 element의 값만 우선 반환, array로 반환하지 않음)

getter가 아닌 메소드의 경우에는 chaining해서 붙여 쓸 수 있다.

$('li').height(500).weight(500);인 경우 첫번째 선택한 엘리먼트를 가리킴


파라메터 갯수(css, attr, prop, offset, data...)
파라메터 여부(val, height, width...)



많이 사용하는 메서드들 살펴보기
돔을 제어(append, prepend, html)
애니메이션(animate, fadeOut, slideUp)
이벤트(on, event object)
class 제어(addClass, removeClass, toggleClass)
Ajax($.ajax)
 

cheatsheet 소개
https://oscarotero.com/jquery/
 

기타 팁
dom이 들어가는 인자는 기본적으로 native element, html 문자, jQuery객체 모두 가능하다.

```
$('li')
$(ele)
$('<div></div>')

$('li').append('<div></div>'); // html이 통째로 들어가기도 함
```

