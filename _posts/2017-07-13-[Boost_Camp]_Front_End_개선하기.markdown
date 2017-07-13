---
layout: post
title:  "[Boost_Camp] Front End 개선하기"
subtitle: "2주차"
date:   2017-07-13 10:40:00
categories: [study]
---

# npm #

1 nodejs 홈페이지에서 설치한다.

2 npm init을 통해서 패키지를 설정한다.

js파일들이 있는 폴더 내에 패키지를 설정한다

3 npm install 등을 이용해서 설치한다(각 모듈별로 설치방법이 다르니 홈페이지를 확인해서 설치한다)

설치 후 extranous는 해당 모듈을 프로젝트에 포함시켰지만 온전하게 포함시킨 것은 아닌 상태임

설치 시 --save flag(옵션)는 package.json폴더 내에 dependency 설정을 해준다. 온전히 포함되게 해줌


원하는 패키지에 포함시키기 위해서는 require를 이용. require는 모듈 객체를 리턴함

