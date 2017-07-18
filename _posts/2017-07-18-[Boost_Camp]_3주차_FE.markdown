---
layout: post
title:  "[Boost_Camp] 3주차 FE"
subtitle: "3주차"
date:   2017-07-18 15:40:00
categories: [study]
---

# 1 로딩 or 인터랙션 #

느리다?
로딩 or 인터랙션

인터렉션이 느리다? DOM 핸들링과 관련되는 경우가 많음. 최대한 DOM에 적게 접근해야함

```
$(".visual_img").animate({ "left": "+=338px" }, "slow" ); // 각 엘리먼트 이동
function load_category(){
	$list = $(".event_tab_lst.tab_lst_min");
	
	var URL = "http://localhost:8080/api/categories";

	$.ajax({
		url : URL,
		contentType:"application/json",
		type: "get",
		success : function(data){
			for(var i=0;i<data.length;i++){
				$list.append('<li class="item" data-category="'+data[i].id+'"><a class="anchor"> <span>'+data[i].name+'</span> </a></li>');
			}

			

		}
	});

	return false;
}
```


map 메소드를 사용

```
for(var i = 0; i<data.length; i++) {
	$list.append('<li></li>');
}
```

이런 코드는 아래와 같이 개선 가능

```
var arr = [];
for(var i = 0; i<data.length; i++) {
	arr.push('<li></li>');
}
$list.html(arr.join(''));
```

아래와 같이 사용하면 더 깔끔함
```
$list.html(data.map(function(v, i) {
	return "<li></li>";
}).join(''))


// 맵은 아래와 같은 원리임
var arr = ['a', 'b', 'c'];

var transformMap = arr.map(funtion(v, i) {
	return v + i;
});

// arr = ['a0', 'b1', 'c2']
```





# 2 스크립트 위치에 따른 이슈 #

아래와 같은 코드가 많았음
```
//a.js
$('#test').on('click', function() {
	alert(1);
});
```



```
//a.html
//시점의 문제임(js가 불려올 때 엘리먼트가 없음), 또 오타인 경우도 많음, 반드시 위로 써야할 경우에는 $(document).ready(funtionc() {});으로 쓰거나, $(fucntion() {})으로 사용
<html>
	<head>
		<script src="jquery.js"></script>
		<script src="a.js"></script>
	</head>
	<body>
		<div id="test"></div>
	</body>
</html>
```	



# 3 구문 사용 #


```
for(var i in data){
 data[i]
}
```

Array인 경우에는 for in 구문을 쓰지 않는게 좋음

for in 구문을 쓰면 object라고 생각하고 코드를 읽는 경우가 많음. 좋은 컨벤션을 만드는 것보다는, 컨벤션을 지키는게 중요함



# 4 모듈 패턴에서의 엘리먼트 #

모듈 패턴에서 엘리먼트 찾는 것은 모듈 안에서 해야 하는 것인가요? 아니면 엘리먼트 선택을 밖에서 한 다음에 모듈에 삽입해서 사용하도록 해야하나요?

유동적인 태그에 사용할 코드는 아래와 같음

```
var Test = {
	some: function(ele) {
		$(ele).on('click', this.hi);
	},
	hi: function(e) {
		console.log('hi');
	}
}
Test.some($('#id'));
```

비유동적인 태그에 사용할 코드는 아래와 같음 - 이러한 구조에서는 1번에 1개의 객체만 생성되는 느낌이기때문에 아래와 같이 많이 사용함

```
var Test = {
	some: function() {
		$('#id').on('click', this.hi);
	},
	hi: function(e) {
		console.log('hi');
	}
}
Test.some();
```



```
var Test = {
	count:0,
	some: function(ele) {
		$(ele).on('click', this.hi.bind(this));
	},
	hi: function(e) {
		this.count++;
		console.log('hi');
	}
}
Test.some($('#id'));
Test.some($('#id2'));
Test.some($('#id3'));
Test.some($('#id4'));

//각각 다르게 다 증가시키고 싶은데도, 하나만 증가됨. 그리고 어느 하나만 증가함
```

그리고 받을 거면 변하는 엘리먼트를 받아야 한다!!(최상단 1개만 받고 find로 찾아감)
```
<div id="root">
	<span class="prev"></span>
	<span class="next"></span>
</div>
```


# 5 핸들바 이용시 팁 #


```
var template = Handlebars.templates.product(product);
$($ul_lst_event_boxes[0]).append(template);
```


```
var template = Handlerbars.compile(html);
template({
	a: 1,
	b: 2
});

function addTodo(data) {
	var template = Handlerbars.compile(todo); // 1번만 호출하면 되기 때문에 밖에다가 빼는게 나음
	$('#li').append(template(data));
}
```



6


```
var categorylist = (function(){
	var curruntCategoryId = 0;
	
	return {
		viewProductByCategory: function(event){
			
		},
		
		viewMoreProductList: function(event){
			
		}, 
		
		updateCount: function(event){

		}, 
		
		scrollViewMoreProductList: function(){

		}
	}
})();

$('ul.event_tab_lst').on('click', 'li.item', categorylist.viewProductByCategory);
$('ul.event_tab_lst').on('click', 'li.item', categorylist.updateCount);
$('ul.lst_event_box').on('click', 'li.item', function(event){
	event.preventDefault();
	event.stopPropagation();
	
	var id = $(this).data('id');
	var baseUrl = 'http://localhost:8080';
	var url = baseUrl + '/product/detail/' + id;
	window.location.href = url;
});

// 처음에 전체 리스트 출력하도록
$('li.item:first-child').trigger('click');

// 더보기 이벤트 추가
$('div.more > button.btn').on('click', categorylist.viewMoreProductList);

$(window).on("scroll", categorylist.scrollViewMoreProductList);
```



7

mouseenter, mouseleave,moverover,mouseout의 차이는

8

// 대문자.
var category = (function(){
	return {
		 deleteData:function(){
			$parent = $(this).parent();
			var id = $parent.attr('id');
			var URL = "http://localhost:8080/api/categories/"+id;

		},
		 updateData:function(){
			$parent = $(this).parent();
                }
	}
})();
9

<script id="project-item-template" type="text/x-handlebars-template"></script>
10

if(categoryId == 0 || categoryId == undefined)
   alert(1);
else
   alert(2)
11

Q2. [FE] Javascript의 모듈 패턴을 이용할 때 분리한 모듈 js 파일을 html에서 로드 하는 것과 js 파일에서 바로 로드 하는 방법 중 어떤 것이 더 좋을까요?

Q3. [FE] npm으로 받은 라이브러리 브라우저에서 사용하는 방법(import)

Q4. [FE] 간단한 데이터 입력일 경우 핸들바를 안쓰고 jquery css 선택자로 주입하는 것도 괜찮을까요?
그리고 두개의 ul 태그에 번갈아서 넣을 경우 그냥 ul 태그 밖에 li 태그 템플릿을 두고 주입 해서 구현했는데 이렇게 사용해도 되는 걸까요?

12

깃도 풀리퀘스트와 머지 과정이 어떻게 돌아가는지 이론적으론 알겠으나 구체적인 부분, 실제 사용하는 부분에서 조금 막막한게 사실입니다. 풀리퀘스트를 보낸 이후에도 푸쉬를 하면 변경내용이 반영되는 것은 같은데 그게 머지될 때는 어느 부분까지 되는 건지 잘 모르겠습니다.

13

이벤트 딜리게이션

14

지난 주 정리하기

15

event emitter

		function Rectangle(){
		}
		Rectangle.prototype = new eg.Component();
		Rectangle.prototype.constructor = Rectangle;
		Rectangle.prototype.some = function(){
		 console.log("a");
		}	

		var a = new Rectangle();
		a.some();
		a.on("click",function(){
			console.log("aa");
		})
		a.trigger("click");
function extend(superClass, def) {
	var extendClass = function extendClass() {
		// Call a parent constructor
		superClass.apply(this, arguments);

		// Call a child constructor
		if (typeof def.init === "function") {
			def.init.apply(this, arguments);
		}
	};

	var ExtProto = function() {};
	ExtProto.prototype = superClass.prototype;

	var extProto = new ExtProto();
	for (var i in def) {
		extProto[i] = def[i];
	}
	extProto.constructor = extendClass;
	extendClass.prototype = extProto;

	return extendClass;
};

var Test = extend(eg.Component,{
	init : function(){
	},
	some : function(){
		this.trigger("hi",{"a":1})
	}
});

var test = new Test();
function callback(e){
	console.log("hi", e);
	test.off(callback);
}
test.on("hi",callback);