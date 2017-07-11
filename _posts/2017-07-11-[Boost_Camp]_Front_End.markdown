---
layout: post
title:  "[Boost_Camp] Front End"
subtitle: "시작 전 준비단계"
date:   2017-07-11 14:40:00
categories: [study]
---

#1 이상한 점 찾기.

```
function deleteCategoryRequest(categoryId){
	$.ajax({
		url:'/api/delete/'+categoryId,
		method:"delete",
		processData:true,
		contentType : "application/json; charset=UTF-8",
		success: deleteList(categoryId)
	})
}
function deleteList(categoryId){
	$('.category#'+categoryId).remove();	
}
```

동작은 할 수 있음

그러나 이 코드는 deleteCategoryRequest가 호출되서 success이하의 함수가 호출되고자 하는 코드로 보임

그러나 이 코드를 실행할경우 동시에 실행됨

bind, apply, call, this를 배운다.


###this는 호출할 때 결정된다.
한번 호출 되면 함수가 끝날때까지 변경할 수 없음
setTimeout, setInterval는 window가 기본 this임
element event등은 모두 element가 기본 this임

```
function test() {
	return 'a';
}

var Test = {
	'a' : 1,
	'init' : fucntion() {
		$('#test').on('click', this.click);
	},
	'click':function() {
		//this -> element(this가 Test가 아닌 것은 알고 있음) 이런애들은 기본적으로 this가 element임
//settimeout, setinterval은 기본적으로 this가 window임
		console.log(this.a);
	}
};

Test.init();
```


###->해결 바인드(바인드의 반환값은 함수임)



```
function test() {
	return 'a';
}

var Test = {
	'a' : 1,
	'init' : fucntion() {
		// this -> Test
		// 바인드로 하면 this라는 애로 무조건 호출하게 되는 함수가 반환됨
		$('#test').on('click', this.click.bind(this));
	},
	'click':function() {
		// this -> Test로 바뀌게 됨
		console.log(this.a);
	}
};

Test.init();
```

-> 바인딩한 것에 추가로 인자를 넣을 수 있음 

```
function test() {
	return 'a';
}

var Test = {
	'a' : 1,
	'init' : fucntion() {
		// this -> Test
		// 바인드로 하면 this라는 애로 무조건 호출하게 되는 함수가 반환됨
		$('#test').on('click', this.click.bind(this, a, a, a));
	},
	'click':function(event, a, b, c) {
		// a = 1
		// b = 1
		// c = 1
		// this -> Test로 바뀌게 됨
		console.log(this.a);
	}
};

Test.init();
```







```
<div id="test"></div>
```




```
var Test = {
	'a' : 1,
	'init' : fucntion() {
		console.log(this.a);
		// 중간에 자기가 임의로 this를 변경할 수는 없음(유동적이기는 함)
	}
};

Test.init();
// 1임
var init = Test.init;
conlsole.log(init === Test.init); // -> true

init(); // ??
```


this는 함수를 실행하는 .앞에가 누구냐임(init은 .앞에 없으므로 window가 this)

.앞이 없으면 window가 됨


```
var a = 10;
var Test = {
	'a' : 1,
	'init' : fucntion(fp) {
		fp();	// 호출될 때 결정됨, window를 확인 그렇기 때문에 a는 10임
	},
	'some' : function() {
		console.log(this.a);
	}
};

Test.init(Test.some);
```


###어플라이는 인자로 전달하는 데, this를 특정값으로 고정시킴

```
var a = 10;
var Test = {
	a : 1
}

function test(a, b, c) {
	console.log(this.a);
}

test();	//10이 출력
test.apply(Test, {1, 2});
test.call(Test, 1, 2);

test.apply(Test2, {1, 2});
test.call(Test2, 1, 2);


test.bind(Test);	// 반환값이 function임 폴리필라이브러리(표준화해주는 라이프러리) Jquery의 proxy와 비슷함

Function.prototype.bind = function(thisObj, param) {
	return fuction() {
		this.apply(thisObj);
	}
}

```

###call
call은 바인드랑 거의 비슷하게 사용.






2

Q1: 실제로 이벤트를 등록, 처리할 때는 jquery로만 주로 하는지, 아니면 javascript addEventListener 메서드를 직접 사용 하는지 궁금합니다.




3

Q3. jsp에서 javscript 로딩은 바로 앞에서 했고 css 로딩은 안에서 했는데 주로 이렇게 많이 하는가요?



4

application/x-www-form-urlencoded와 application/json의 차이는?



5

FE 테스트는 어떤식으로 하나요?



6

API디자인?



```
$('#categoryTable').on('click','.update',function() {
	var button=$(this);
	var id=button.parent().siblings('.categoryId').text();
	var name=button.parent().siblings('.categoryName').children().val();
	var data = {'id':id,'name':name}
	$.ajax({
		url:'/api/task1',
		headers: {
			'Content-Type': 'application/json'
		},
		data : JSON.stringify(data),
		type:'PUT',
 	 success:function() {
	 }
	});
});
$('#categoryTable').on('click','.delete',function() {
	var button=$(this);
	$.ajax({
		url:'/api/task1/'+$(this).val(),
		type:'DELETE',
 	 success:function() {
 		button.parents('tr').remove();
	 }
	});
});
```

	
7

class의 활용.



```
.invisible {
	display: none;
}
$('.editing .category_label, .editing .editBtn').addClass('invisible');
$('.editing .category_edit, .editing .updateBtn').removeClass('invisible');
```


9

호이스팅