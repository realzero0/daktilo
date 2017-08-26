---
layout: post
title:  "[Boost_Camp] Cross Site Request Forgery(CSRF)"
subtitle: "8주차"
date:   2017-08-21 20:40:00
categories: [study]
---

CSRF란? 
---
XSS(Cross-site Scripting)의 공격이 가능한 사이트에서 특정 이상의 권한을 가진 사용자가 특정 행위를 하도록 하는 방법이다. 관리자의 구매 행위 등이 여기에 해당한다.

공격자는 URL 패턴을 분석해 특정 URL의 역할을 구분하고, 그런 행위를 하도록 스크립트 코드를 만든다.