---
layout: post
title:  "[Boost_Camp] Cross Site Scripting(XSS)"
subtitle: "7주차"
date:   2017-08-18 20:40:00
categories: [study]
---

XSS란? 
---
XSS(Cross-site Scripting)란 공격자가 공격하려는 사이트에 공격 코드를 주입하는 것을 의미한다. XSS는 가장 흔히 나타나는 공격 중 하나이며, 특히 Text-only나 BBCode처럼 자바스트립트를 사용할 수 없는 곳에서는 XSS가 발생하지 않지만, 자바스크립트 주입이 가능하다면 여러 공격이 가능할 수 있다.

XSS가 전체 사이트에 심어지게 될 경우, 모든 유저의 정보가 위험해질 수도 있다.

자바스크립트로 대부분의 공격이 이루어지며, 이를 통해 얻은 정보는 쿠키, 세션 정보를 얻을 수 있다.

이를 통해 사용자는 유저인 척을 하면서 사용자의 정보를 얻을 수 있다.


1. Non Persistent Script - 공격자에게만 동작하는 스크립트를 의미한다.
2. Persistent Script - 웹사이트 전체에서 동작할 수 있는 스크립트를 의미한다.




XSS의 원리


XSS의 예제