---
layout: post
title:  "[Boost_Camp] Clickjacking"
subtitle: "8주차"
date:   2017-08-21 20:40:00
categories: [study]
---

Clickjacking이란? 
---
클릭재킹이란 두개의 iframe을 이용해서 사용자의 클릭시, fishing 사이트나 여러 정보들을 갈취해가는 공격 방법을 의미한다.

이를 막기 위해서는 iframe을 방지하도록 해야하는데, 

HttpServletResponse.setHeader( "X-Frame-Options", "SAMEORIGIN" );

위와 같은 설정으로 cross site iframe을 방지할 수 있다.