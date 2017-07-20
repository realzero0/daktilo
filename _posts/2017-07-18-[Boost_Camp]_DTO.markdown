---
layout: post
title:  "[Boost_Camp] DTO"
subtitle: "3주차"
date:   2017-07-18 20:40:00
categories: [study]
---

# 1 dto  조인 #

사람마다 다 다름. Map을 이용하기도 하고, DTO를 이용하기도 함. 어떤식으로 사용할지는 팀마다 적절히 선택해야 한다.

DTO를 많이 만드는게 크게 문제가 되지는 않음

<br>
# 2 int vs Integer #

Integer는 null 값이 허용된다는 장점이 있다.

int는 0이 넘어올 경우, 정해줘야 한다.

무엇이 더 좋은가는 프로그래머가 결정할 부분이다.

<br>
# 3 REST API 주소 #

REST API의 주소는 집합의 단위에 맞게 생각해보는게 중요하다.

여러가지 컨벤션이 있을 수 있다.

어떤 RESTFUL API의 주소를 어떻게 결정할지는 Case by Case이다.

더 가독성 좋은 주소를 결정하는 게 가장 좋은 것이다.

가장 중요한 것은 팀이 결정한 URL 컨벤션이고, 나머지는 현재 사이트들의 경향, 또는 유행을 잘 따라가서 해보는 것이 중요하다.

다만 팀이 정한 규칙은 일관성이 있어야 한다.

<br>
# 4 코드 컨벤션 #

DtoDao 등은 사용하지 않음

DAO는 테이블별로만 만들어 놓는게 좋음

최소한의 코드 컨벤션은 지키면서 코딩하는 게 좋음

너무 줄여서 코드를 작성할 필요는 없음


<br>
# 5 컬렉션 #

List, Set, Map으로 리턴 타입을 사용하는 게 더 좋다.

자료구조를 적절히 사용해서 사용하는 게 더 좋다.

<br>
# 6 에러페이지 #

에러페이지를 등록하는 방법 찾아보기

<br>
# 7 html이냐 자바스크립트냐 #

네이게이션 부분에서는 html을 쓰는게

검색엔진이 페이지를 검사하는데도 더 좋음

조금씩 해보고 해결해 나가는 게 더 좋음

<br>
# 8 bind #

```
function foo(name , age) {
	
}

foo('john', 30);
```


메모리가 2개 생김 1개는 'john'을 가리키게 됨

```
var Person = (function() {
	
	return function(name, age) {
		return {
			intro: function() {
				console.log(name, age);
			}
		}
	}
})();

var john = Person('John', 30);
var jane = Person('Jane', 30);

john.intro();
jane.intro();
```

프로그램에서 1개만 존재해야하는 것이 아닌 이상 객체로 만드는 것이 좋음

1개여야한다 -> spec

spec적으로 1개여야만 하는게 아닌 이상, 여러개를 만들 수 있는 객체인 것이 좋음

john이라는 객체가 intro()를 호출할 때, 유효범위의 대상으로 넘어감.

여기서 유효범위는 객체가 아니고 함수를 단위로 함(로컬에 없는 변수는 자유변수, 자유번수는 부모 scope로 올라감, 이런식으로 찾아서 올라가는 것을 scope 체인이라고 부름)

property를 타는 방법과 scopechain을 타는 방법 두가지 방법으로 자바스크립트에서는 접근하기 때문에

객체 내부에 있는 property에 접근하기 위해서는 반드시 this를 명시해줘야 한다.

```
var Person = (function() {
	
	return function(sex, name, age) {
		
	}
})();

//this는 null로 하고, 첫번째 인자를 'M'으로 묶겠다는 것, Person은 그대로 Person임
var Man = Persion.bind(null, 'M');
//아래와 같이 할 수도 있음
var man2 = Persion.bind(null, 'M')('John', 30);



var man1 = Man('John', 30);
```



jquery object는 $prev 처럼 $를 붙여씀

html()은 제대로 값이 나오지 않는 경우가 많음, 값으로 사용해야하는 것들은 data attribute를 사용하는게 좋음

```
//function declaration
function foo() {

}

//function literal
var foo = function() {}

var x = 1234;
var y = function() {}
```

declaration이 더 좋은 경우가 많음

호이스팅 되기 때문에 자유롭게 사용할 수 있음

호이스팅을 의도적으로 피하고 싶지 않은 이상 안해도 되는데, 의도적으로 피하고 싶은 상황이라면 문제가 생길 수 있음

모듈이 다루는 단위도 블럭 단위

모듈은 Cyclic Dependencies 순환의존 관계이면 안됨

A -> B, B -> C, C -> A 인 경우를 제거해야함



브라우저는 스크립트를 만나면 DOME 파싱을 중단함

스크립트를 파싱하고 실행한 뒤에

다음 DOME을 DOME TREE에 추가하기 시작함

준비가 다 끝나면 $(function() {}) 또는 ready로 가능함.

defer는 끝에서 실행하라는 속성, async는 다 받고 나서 실행함.

body onload는 모든 컨텐츠가 다 로딩되었을 때 뜸,

ready는 onload보다는 빠르게 실행할 수 있음 Dome만 로딩되면 되기 때문임