---
layout: post
title:  "[Boost_Camp] FE 터치 이벤트"
subtitle: "4주차"
date:   2017-07-25 17:40:00
categories: [study]
---

## 1 bind를 안써도 될 때는 쓰지 말자 ##

이상한 점 찾기.

this를 고정시키고 싶을 때, bind를 사용한다. bind를 굳이 사용하지 않아도 될 때, bind를 사용하지 않는다.


```
function bindEvents() {
	$('.thumb').on('click', popupViewer.bind(this));
	$('#layer')
		.on('click', '.com_img_btn.close', hideViewer.bind(this))
		.on('click', '.com_img_btn.nxt', clickNext.bind(this))
		.on('click', '.com_img_btn.prev', clickPrev.bind(this));
	$('#photoviewer')
		.on('click', toggleBtnVisible.bind(this))
		.on('touchstart', '.wrapper img', viewerTouchStart.bind(this))
		.on('touchmove', '.wrapper img', viewerTouchMove.bind(this))
		.on('touchend', '.wrapper img', viewerTouchEnd.bind(this));
}
```



## 2 left와 translateX의 차이는? ##

translate -> x, y, z로 이동 시킬 수 있음

rotate -> x, y, z로 회전

scale -> x, y, z로 크기 변형

matrix -> 행렬을 적음(매트릭스를 이용)


아래 네 개는 모두 positioned element임

left -> position이 된 상태여야 함. position의 default값은 static임 static이 아닌 모든 경우는 positioned임

right

top

bottom


translate는 축이 3개임 translate3d(x, y, z)

성능이 좋아지려면 translate을 사용하는게 좋다고 함

성능에는 interaction성능과 loading성능이 있는데, loading은 요청횟수, interaction성능은 DOM을 얼만큼 touch하느냐가 중요

left는 이동할때마다 매번 DOM을 새로 그림 -> 굉장히 비효율적임

translate는 옵션을 준 Obj만 분리됨, 움직이고 합성함

레이어를 너무 많이 주면, 레이어마다 모두 메모리가 됨.

레이어를 그리는 것은 CPU로 그리는데, 레이어를 가지고 있는 것은 GPU인데

모바일 환경에서는 GPU 메모리가 없는 경우도 있기 때문에 문제가 발생하기 쉬움(브라우저 강제 종료 등)

will-change라는 옵션이 최근에 생김 - 레이어를 띄울지 안띄울지 브라우저가 메모리 상황에 따라 결정할 수 있게 함

css에 will-change transform으로 설정함

stacking order - 레이어의 우선순위를 결정하는 것



```
$(this).css('transform', 'translatex(' + (-t) + '%)');
```


## 3 기본적으로 브라우저는 타이머가 돌아간다 ##

브라우저는 상태머신이다.

상태머신은 우리가 해야하는 상태가 복잡할 때,

메일 서비스에서 2단보기, 쓰기, 테마, 여러가지 조건에 의해

현재 상태를 바꿔야 할 때 기능 구현이 힘들어짐

어떤 상태를 하나로 고정하고 다른 상태를 결정

이런 면에서 브라우저는 기본적으로 상태머신이다.

DOM

```
<html>
	<head>
	</head>
<body>
	<div>
		<ul>
			<li>1</li>
			<li>2</li>
		</ul>
	</div>
</body>
</html>
```

브라우저는 위와 같은 코드를 DOM tree로 만든다.


```
#id {
	description
}

#id.div {
	description
}
```

스타일시트, css를 tree로 만들고

body부분만 다시 css와 합쳐서 render obj를 만든다.

render obj를 이용해서 화면을 그리게 된다.

브라우저는 DOM의 정보는 Timer가 돌아갈 때마다 업데이트한다.

그래서 left의 정보를 바꿔도 업데이트가 되지 않는다.

그런데 특정 상황에서는 강제로 업데이트 되는데, 이를 Forced Layout(또는 reflow)라고 한다.



```
$(this).css({
	'max-height': $(window).height(),
	'max-width': $(window).width()
});
$(this).parent().css({
	'height': $(window).height(),
	'width': $(window).width(),
	'margin-top': -this.height / 2,
	'margin-left': -this.width / 2
});	
```


강제로 reflow가 되는 상황이 어떤 상황이나면 아래와 같은 메소드들이고,

아래와 같은 메소드들이 호출될 것 같은 라이브러리 함수는 사용하지 않는 것이 좋다.

웬만하면 DOM에서 가져오는 정보는 caching해서 가져오는 것이 좋다.

[출처](https://gist.github.com/paulirish/5d52fb081b3570c81e3a)
### Element

##### Box metrics
* `elem.offsetLeft`, `elem.offsetTop`, `elem.offsetWidth`, `elem.offsetHeight`, `elem.offsetParent`
* `elem.clientLeft`, `elem.clientTop`, `elem.clientWidth`, `elem.clientHeight`
* `elem.getClientRects()`, `elem.getBoundingClientRect()`

##### Scroll stuff
* `elem.scrollBy()`, `elem.scrollTo()`
* `elem.scrollIntoView()`, `elem.scrollIntoViewIfNeeded()`  
* `elem.scrollWidth`, `elem.scrollHeight`
* `elem.scrollLeft`, `elem.scrollTop` also, setting them


##### Focus
* `elem.focus()`  can trigger a *double* forced layout ([source](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/dom/Element.cpp&q=updateLayoutIgnorePendingStylesheets%20-f:out%20-f:test&sq=package:chromium&l=2369&ct=rc&cd=4&dr=C))

##### Also…
* `elem.computedRole`, `elem.computedName`  
* `elem.innerText` ([source](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/dom/Element.cpp&q=updateLayoutIgnorePendingStylesheets%20-f:out%20-f:test&sq=package:chromium&l=2626&ct=rc&cd=4&dr=C))

### getComputedStyle 

`window.getComputedStyle()` will typically force style recalc ([source](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/dom/Document.cpp&sq=package:chromium&type=cs&l=1860&q=updateLayoutTreeForNodeIfNeeded))

`window.getComputedStyle()` will force layout, as well, if any of the following is true: 

1. The element is in a shadow tree
1. There are media queries (viewport-related ones). Specifically, one of the following: ([source](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/css/MediaQueryExp.cpp&sq=package:chromium&type=cs&l=163&q=MediaQueryExp::isViewportDependent))
  * `min-width`, `min-height`, `max-width`, `max-height`, `width`, `height`
  * `aspect-ratio`, `min-aspect-ratio`, `max-aspect-ratio`
  * `device-pixel-ratio`, `resolution`, `orientation` 
1. The property requested is one of the following:  ([source](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/css/CSSComputedStyleDeclaration.cpp&sq=package:chromium&l=457&dr=C&q=isLayoutDependent))
  * `height`, `width`
  * `top`, `right`, `bottom`, `left`
  * `margin` [`-top`, `-right`, `-bottom`, `-left`, or *shorthand*] only if the margin is fixed.
  * `padding` [`-top`, `-right`, `-bottom`, `-left`, or *shorthand*] only if the padding is fixed.
  * `transform`, `transform-origin`, `perspective-origin`
  * `translate`, `rotate`, `scale`
  * `webkit-filter`, `backdrop-filter`
  * `motion-path`, `motion-offset`, `motion-rotation`
  * `x`, `y`, `rx`, `ry`

### window

* `window.scrollX`, `window.scrollY`
* `window.innerHeight`, `window.innerWidth`
* `window.getMatchedCSSRules()` only forces style


### Forms

* `inputElem.focus()`
* `inputElem.select()`, `textareaElem.select()` ([source](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/html/HTMLTextFormControlElement.cpp&q=updateLayoutIgnorePendingStylesheets%20-f:out%20-f:test&sq=package:chromium&l=192&dr=C))

### Mouse events

* `mouseEvt.layerX`, `mouseEvt.layerY`, `mouseEvt.offsetX`, `mouseEvt.offsetY` ([source](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/events/MouseRelatedEvent.cpp&q=f:mouserelatedevent%20computeRelativePosition&sq=package:chromium&type=cs&l=132))

### document

* `doc.scrollingElement` only forces style

### Range

* `range.getClientRects()`, `range.getBoundingClientRect()`

### SVG

* Quite a lot; haven't made an exhaustive list , but [Tony Gentilcore's 2011 Layout Triggering List](http://gent.ilcore.com/2011/03/how-not-to-trigger-layout-in-webkit.html) pointed to a few.


## 4 컴포넌트는 element 하나만 보낼 수 있도록 만든다. ##

```
function Test(ele) {	//이런식의 코드는 의존관계를 줄여줌
	$(ele).find('.selected');
}

var test = new Test($('#rootId'));	// 루트 주변에 있다면 루트 기준으로 작성하는 편이 좋음


function Test(ele, min, max, something) {	// 너무 많은 매개변수는 지양하는 편이 좋음 아래 함수와 같이 객체를 만들어서 리펙토리한다.

}
function Test(ele, info) {	// 이와 같이 변경하는 것이 일반적이다.

}

var test = new Test($('#rootId'), {
	min: 10,
	max: 20
});
```

컴포넌트를 쓰기 쉽게 만드는게 가장 중요하고, 스마트 디폴트(아무 설정도 없이 디폴트를 만들 수 있는 것)이 잘되면 좋음


## 5 모듈명은 대문자 ##


```
var clickEventListener = (function(){
})();
```


6


```
var li_template =
	"<li class='category'>"+
		"<input class='categoryName' type='text' name='name'>"+
		"<button class='update'>수정</button>"+
		"<button class='delete'>삭제</button>"+
	"</li>";
var clickEventListener = (function(){
	return {
		deleteCategoryRequest : function(event){
		},
		updateCategoryRequest : function(event){
		},
		addCategoryRequest : function(event){
		}
	};
})();

$(document).on("click", ".delete", clickEventListener.deleteCategoryRequest);
$(document).on("click", ".update", clickEventListener.updateCategoryRequest);
$(document).on("click", ".add", clickEventListener.addCategoryRequest);

```
document에 바인딩하면 모든 DOM을 검색함.

또 순서가 중요해짐. init의 위치가 어디에 있는지가 중요


7


```
function Test(){
	function some(){
		console.log("some1");
	}

	return {
		some : some
	}
}

function Test(){
	this.some = function(){
		console.log("some2");
	}
}

function Test(){

}
Test.prototype.some = function(){
	console.log("some3");
}

var test = new Test(); // 이것처럼 호출할 때, 위의 세가지는 중 첫번째와 두번째는 비슷하지만 세번째는 다르다.

//첫번째와 두번째는 new를 하면 각각의 인스턴스를 생성하게 된다.

//세번째는 하나의 객체를 바라보고있다. 그 뒤에 수정하게 되면 또 다른 함수를 바라보게 된다. 프로토타입의 함수를 바꾸면 둘다 모두 바뀌게 된다.

//첫번째는, new 할 경우, 반환할 경우, 그 리턴한 객체를 반환한다. 첫번째는 obj로 인식되고, 두번째는 Test로 인식됨, 두번째는 some이라는 함수를 매번 만들게 되고, 세번째는 가리키게 된다. 일반적으로는 3번째 방법으로 많이 사용한다.
```



9

```
component.count = 1;

component.setCount(1);	//이와 같이 바꾸는게 좋다.
```


10

```
function Rolling(){
}
Rolling.prototype.constructor = Rolling;
Rolling.prototype = {
   some : function(){}
}
// 한번에 객체로 정의하고자 했지만, constructor를 새로 정의해줘야 prototype으로 설정할 수 있다. obj.constructor === Rolling으로 하면 비교할 수 있는데, prototype에 obj가 입력되면 비교가 안된다. 그래서 constructor를 다시 덮어씌워줄 필요가 있다.

function Rolling(){
}
Rolling.prototype = {
   some : function(){}
}
Rolling.prototype.constructor = Rolling;
// 위와 같은 코드가 정답이다.
```