---
layout: post
title:  "[Boost_Camp] JavaScript 객체 지향 정리"
subtitle: "4주차"
date:   2017-08-01 10:40:00
categories: [study]
---

# 프로토 타입 체인을 이용한 상속 #

## 속성 상속 ##

- 자바스크립트의 모든 변수는 객체이며, Object.prototype을 확장해서 사용한다.

- 프로토타입은 각각의 새로운 프로토타입과 연결되어 사용될 수 있으며, 여러개의 체인으로 구성할 수 있다.


```
// o라는 객체가 있고, 속성 'a' 와 'b'를 갖고 있다고 하자.
// {a: 1, b: 2}
// o.[[Prototype]]은 속성 'b'와 'c'를 가지고 있다. 
// {b: 3, c: 4}
// 마지막으로 o.[[Prototype]].[[Prototype]]은 null이다. 
// null은 프로토타입의 종단을 말하며 정의에 의해서 추가 [[Prototype]]은 없다. 
// {a: 1, b: 2} ---> {b: 3, c: 4} ---> null

console.log(o.a); // 1
// o는 'a'라는 속성을 가지는가? 그렇다. 속성의 값은 1이다.

console.log(o.b); // 2
// o는 'b'라는 속성을 가지는가? 그렇다. 속성의 값은 2이다.
// 프로토타입 역시 'b'라는 속성을 가지지만 이 값은 쓰이지 않는다. 이것을 "속성의 가려짐(property shadowing)" 이라고 부른다.

console.log(o.c); // 4
// o는 'c'라는 속성을 가지는가? 아니다. 프로토타입을 확인해보자.
// o.[[Prototype]]은 'c'라는 속성을 가지는가? 가지고 값은 4이다.

console.log(o.d); // undefined
// o는 'd'라는 속성을 가지는가? 아니다. 프로토타입을 확인해보자.
// o.[[Prototype]]은 'd'라는 속성을 가지는가? 아니다. 다시 프로토타입을 확인해보자.
// o.[[Prototype]].[[Prototype]]은 null이다. 찾는 것을 그만두자.
// 속성이 발견되지 않았기 때문에 undefined를 반환한다.
```

## 메소드 상속 ##

- 자바스크립트에서 메소드라는 것은 없다. 자바스크립트는 객체의 속성으로 함수를 지정할 수 있고, 속성값을 사용하듯이 함수를 사용할 수 있다

- 속성 값으로 지정한 함수의 상속 역시 위에서 본 속성의 상속과 동일하다. (단 위에서 언급한 "속성의 가려짐" 대신 "메소드 오버라이딩, method overriding" 라는 용어를 사용한다)

- 상속된 함수가 실행 될 때,  this 라는 변수는 상속된 오브젝트를 가르킨다. 그 함수가 프로토타입의 속성으로 지정되었다고 해도 말이다.

```
var o = {
  a: 2,
  m: function(b){
    return this.a + 1;
  }
};

console.log(o.m()); // 3
// o.m을 호출하면 'this' 는 o를 가리킨다.

var p = Object.create(o);
// p 는 프로토타입을 o로 가지는 오브젝트이다.

p.a = 12; // p 에 'a'라는 새로운 속성을 만들었다.
console.log(p.m()); // 13
// p.m이 호출 될 때 'this' 는 'p'를 가리킨다.
// 따라서 o의 함수 m을 상속 받으며,
// 'this.a'는 p.a를 나타내며 p의 개인 속성 'a'가 된다.
```
---

# 객체를 생성하는 방법 #

## 문법 생성자로 객체 생성 ##

```
var o = {a: 1};

// o 객체는 프로토타입으로 Object.prototype 을 가진다.
// 이로 인해 o.hasOwnProperty('a') 같은 코드를 사용할 수 있다.
// hasOwnProperty 라는 속성은 Object.prototype 의 속성이다.
// Object.prototype 의 프로토타입은 null 이다.
// o ---> Object.prototype ---> null

var a = ["yo", "whadup", "?"];

// Array.prototype을 상속받은 배열도 마찬가지다.
// (이번에는 indexOf, forEach 등의 메소드를 가진다)
// 프로토타입 체인은 다음과 같다.
// a ---> Array.prototype ---> Object.prototype ---> null

function f(){
  return 2;
}

// 함수는 Function.prototype 을 상속받는다.
// (이 프로토타입은 call, bind 같은 메소드를 가진다)
// f ---> Function.prototype ---> Object.prototype ---> null
```

## 생성자를 이용 ##

```
function Graph() {
  this.vertexes = [];
  this.edges = [];
}

Graph.prototype = {
  addVertex: function(v){
    this.vertexes.push(v);
  }
};

var g = new Graph();
// g 'vertexes' 와 'edges'를 속성으로 가지는 객체이다.
// 생성시 g.[[Prototype]]은 Graph.prototype의 값과 같은 값을 가진다.
```

## Object.create 이용 ##

ECMAScript 5 이후에 가능

```
var a = {a: 1}; 
// a ---> Object.prototype ---> null

var b = Object.create(a);
// b ---> a ---> Object.prototype ---> null
console.log(b.a); // 1 (상속됨)

var c = Object.create(b);
// c ---> b ---> a ---> Object.prototype ---> null

var d = Object.create(null);
// d ---> null
console.log(d.hasOwnProperty); // undefined이다. 왜냐하면 d는 Object.prototype을 상속받지 않기 때문이다.
```

## class 키워드 이용 ##

ECMAScript2015 이후에 가능

```
'use strict';

class Polygon {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
}

class Square extends Polygon {
  constructor(sideLength) {
    super(sideLength, sideLength);
  }
  get area() {
    return this.height * this.width;
  }
  set sideLength(newLength) {
    this.height = newLength;
    this.width = newLength;
  }
}

var square = new Square(2);
```

## 성능 이슈 ##

프로토타입 체인의 속성을 모두 검색해서 그 속성을 찾아내기 때문에 성능에 영향을 미칠 수 있다.

객체만이 가지고 있는 속성인지, 그게 아니라 체인 내에 있는 속성인지를 알아내기 위해서는 Object.prototype에서 모든 오브젝트로 상속된 hasOwnProperty 메소드를 이용할 필요가 있다. 

```
console.log(g.hasOwnProperty('vertices'));
// true

console.log(g.hasOwnProperty('nope'));
// false

console.log(g.hasOwnProperty('addVertex'));
// false

console.log(g.__proto__.hasOwnProperty('addVertex'));
//__proto__은 부모의 프로토타입을 리턴한다
// true
```

- 참고: undefined인지 여부만 확인하는 것으로는 충분하지 않다. 여전히 속성이 존재할 수도 있는데 단지 그 값에 undefined가 할당되어 있을 수도 있기 때문이다.








Object.prototype.hasOwnProperty()