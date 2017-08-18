---
layout: post
title:  "[Boost_Camp] BE FE 과제들"
subtitle: "7주차"
date:   2017-08-17 20:40:00
categories: [study]
---

BE 주제
============
1.    MVC & RESTful API
2.    Spring Security 학습 및 적용
3.    View Framework 학습 및 적용
4.    트랜잭션에 대한 이해 및 적용
5.    myBatis , JPA 등 학습 및 적용
6.    단위테스트


# FE 주제 #


Web Security
============
* 보안의 대 전제 - clean client
* 웹 보안이 책임지는 범위/그렇지 않은 범위 - 사용자는 책임지지 않음(사용자의 디바이스, 계정), 기본적인 웹 보안의 범위는 사이트 단위(정확히는 origin), 하나의 사이트가 털려도 다른 사이트가 털리면 안된다. origin(protocol, hostname, port)가 같은 범위 내에서만 스크립트가 돌아가야 한다. 같은 출처에서 온 것만 가능함. ajax는 다른 출처에서 온 통신을 할 수 없고, 이를 위해서는 명시적으로 curls을 열어야한다. 도메인 정책에 따라서 보안정책이 달라지고, 보안정책이 달라지면 서비스의 범위도 달라진다.(ex. 티스토리는 dkdkdk.tistory.com이기 때문에 가능, 네이버는 blog.naver.com/dkdkdkdk이기 때문에 불가능하다.)
* Same Origin Policy
* CORS
* XSS - 스트립트로 접근할 수 있는 영역과 할 수 없는 영역을 나눔
* CSRF
* MITM Attack(Man In the middle, 가로채서 보고, 변조시켜서 보낸다. 퍼블릭 와이파이는 누군가 man in the middle을 심어 놓았을 가능성이 있다.)
* HTTPS/SSL - 클라이언트의 인증서 위조의 문제, 시맨틱 철수, 루트 인증기관에서 브라우저는 인증함, 루트 인증서중에 이상한 인증서가 있으면 문제가 된다. ACTIVEX는 PC를 클린하지 않게 유지하는 점에서도 문제가 있음
* MD5/SHA1/SHA256
* VPN - PPTP, L2TP, OpenVPN
* Password specific hashing algorithm: Argon2, PBKDF2, scrypt or bcrypt
* clickjacking
* Phishing & Pharming
* multi-factor authentication (MFA) - 완전히 분리된 채널에서 검증한다. OTP, 전화 등의 다채널 인증

과제
---
1. 각각의 주제에 대해 
    - 해킹이 발생하는 과정
    - 대응 방법
   을 정리해보세요.
2. 자신이 만든 사이트에서 취약점을 찾고, 이를 공격하는 제3자 사이트(혹은 기타 기법)을 만들어서 자신의 사이트를 공격해 보세요
3. 자신의 사이트에서 이를 방어하세요

참고자료
------
* https://developer.mozilla.org/en-US/docs/Web/Security


ECMAScript 6 (ES2016/ES2017/next)
=================================
* ES5 Items
    * Property Descriptor
* ES6란
* Transpiler - Babel (another option: TypeScript)
* ES6 items
    * let, const and scope
    * template literals
    * arrow function
    * class and inheritance
    * destructuring
    * spread operator
* ES7 items
    * async / await
    * Class properties

과제
---
* 기존에 작성한 프로젝트 코드를 ES6로 전환하세요
* 전환된 코드를 실제 환경에서 돌아가도록 반영하세요
* 결과물(빌드된 파일)의 양을 줄일 수 있는 방법에 대해 검토하고 대안을 적용해 보세요

참고자료
------
* Babel - https://babeljs.io/
* TypeScript - https://www.typescriptlang.org/
* 


Web Performance
===============

<네트워크>

* Performance의 중요성
* Web Page 표시의 병목
* HTTP 와 Connection Pool
* HTTPS/HTTP2
* If-modified-since
* Expire
* DNS
    - DNS Lookup 과정
    - A, CNAME
    - 도메인 분산 vs 도메인 통합 - 요즘은 도메인 통합으로 의견이 기울어짐
* 전송 용량 줄이기
    - Concatenation(파일합치기)
    - Minification(불필요한 공백을 줄임) / Obfuscation(스크립트를 읽기 어렵게 만듬)
    - Gzip
 
    - Optimizing Images


<브라우저 렌더링>

* Chrome 개발자 도구로 성능 측정하기
    - Req, Server Time, Response
    - Performance 패널
* 렌더링 성능
    - DOM Tree vs Render Tree
    - Reflow, Repaint
    - 랜더링 성능 측정하기 

과제
---
* 기존에 작성한 과제를 대상으로 성능 측정을 해보세요
* 성능 개선 포인트를 찾아 반영하세요
* 원하는 사이트를 대상으로 Audit을 진행해보고, 결과 리포트의 각 항목에 대해 조사 및 정리하세요

참고자료
------
* Google PageSpeed: https://developers.google.com/speed/pagespeed/
* Google PageSpeed guide: https://developers.google.com/speed/docs/insights/about
* Steve Sourders blog: https://stevesouders.com/```