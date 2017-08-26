---
layout: post
title:  "[Boost_Camp] VPN - PPTP, L2TP, OpenVPN"
subtitle: "8주차"
date:   2017-08-22 20:40:00
categories: [study]
---

VPN이란? 
---
![](https://raw.githubusercontent.com/tjdcks12/reservation-system/master/resources/img/L2TP.png)

회사나 군대에서 사용하는 인트라넷은 외부에서 접속할 수 없고, 내부에서만 접속할 수 있다.

그런데 회사 외부에서도 회사 내부의 인트라넷에 접속해서 회사 업무를 해야할 때가 분명히 있다.

그를 위해서는 어떻게 할까?

인터넷 회선을 암호화된 규격을 이용해서 마치 본사에 접속된 하나의 subnet 내의 회선처럼 쓰게 만드는 것이 VPN의 핵심적인 내용이다.



## VPN vs Proxy

VPN은 추가로 접속하는 컴퓨터를 추가하는 형태를 취한다. 예를 들어 공유기에 접속한다고 하면, 우리가 새롭게 공유기에 접속한 유저는 개인 IP를 받고 하나의 클라이언트로 접속할 수 있게 된다. 모든 접속을 암호화 하는 것이 VPN의 특징이다.

반면 Proxy의 경우에는 이미 IP가 할당되어 있는 컴퓨터를 라우터로 이용하는 것이다. 사용자의 아이피를 Proxy 서버의 아이피인 것처럼 보이게하고, 사용자의 데이터는 프록시 서버를 거치게 된다. 이 과정에서의 모든 데이터는 암호화되지 않는다는 단점이 있다.


ssl

이런 문제 -> 어떤 기술을 도입 -> 필요한 기술들, 경험들 중에서 문제, 그것이 어떤 결과로 나타나내는 것 -> 어떻게 해결이 된건지!! -> 그래서 우리의 느낌

https /// jsonphijacking jsonpsecurity -> jsonp를 왜 써야하는지 -> jsonp의 문제점이 문제점 -> 보안 해결 -> 사례들을 설명함


xss -> 일반적인 것 말고 재밌는 것. 별점주는 것->> xss를 잘 지키지 않으면 생기는 문제점 ->> 우리팀에는 잘 동작한다.

xss 좀 더 디테일한 정의

오늘, 내일에는 만들어봐야한다..

jsonphijacking - CORS랑 연관해서 이야기



ssl은 정리만

xss랑 jsonp로 정의함 cors랑 -> 문제가 왜 발생했는지부터 설명해야한다.

xss 방지 필터 spring

xss + (jsonp + cors) -> 문제 -> 왜 문제 -> 결과 -> 외부사례들, spring하고 엮어서