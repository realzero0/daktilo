---
layout: post
title:  "[Boost_Camp] Front End"
subtitle: "시작 전 준비단계"
date:   2017-07-11 14:40:00
categories: [study]
---

# 1 이상한 점 찾기. #

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


## this는 호출할 때 결정된다. ##
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


## ->해결 바인드(바인드의 반환값은 함수임) ##



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


## 어플라이는 인자로 전달하는 데, this를 특정값으로 고정시킴 ##

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

## call ##
call은 바인드랑 거의 비슷하게 사용.






# 2 #

> Q1: 실제로 이벤트를 등록, 처리할 때는 jquery로만 주로 하는지, 아니면 javascript addEventListener 메서드를 직접 사용 하는지 궁금합니다.

서로 다른 코드를 해석하는게 굉장히 비효율적, 그래서 Jquery로 짜는 편이 더 낫기도 함

기능 구현하는 시간만큼 디버깅하는 시간도 오래걸림, Product의 안정성, 속도 측면에서 라이브러리를 사용하는게 훨씬 좋음


# 3 #

> Q3. jsp에서 javascript 로딩은 바로 앞에서 했고 css 로딩은 안에서 했는데 주로 이렇게 많이 하는가요?

화면이 먼저 나오고 나서, javascript가 나중에 로딩함

그런데 요즘에는 자바스크립트의 특정 사이드 이상은 다른 스레드에서 작업하기도 하는데, 최근에는 다른 스레드에서 처리하기 때문에 위에 올리면 더 빠르기도 함.

디자인, 현재 상황, 브라우저의 상황을 고려해서 결정해야함

일반적으로는 CSS가 위에 올려 놓고, 자바스크립트를 아래에 놓음

# 4 #

application/x-www-form-urlencoded와 application/json의 차이는?

header는 브라우저와 서버를 위한 값임.

application/x-www-form-urlencoded(a=1&b=2&c=3 처럼 보내는 포맷임)

application/json(제이슨으로 보냄)


# 5 #

FE 테스트는 어떤식으로 하나요?

모카, 단위테스트 위주

서비스의 UI테스트는 많이 쓰지 않음

블랙박스 테스트처럼 케이스가 어떤 결과값을 내는지 많이 확인함



# 6 API디자인? #



되도록이면 같은 URL에서 RESTFUL을 유지할 수 있도록 하는 것이 좋음

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

	
# 7 class의 활용. #



```
.invisible {
	display: none;
}
$('.editing .category_label, .editing .editBtn').addClass('invisible');
$('.editing .category_edit, .editing .updateBtn').removeClass('invisible');
```
show나 hide하지 않고 addClass나 removeClass하는게 좋음

show나 hide가 굉장히 복잡한 로직에 의해 사용됨

사용하기도하고 큰 문제가 되지 않을 수 있지만 더 효율적임

```
<ul id="test">
	<li>1</li>
	<li>1</li>
	<li class="complete">1</li>
	<li>1</li>
	<li>1</li>
	<li class="complete">1</li>
	<li class="complete">1</li>
	<li>1</li>
	<li>1</li>
	<li>1</li>
	<li>1</li>
	<li class="complete">1</li>
	<li>1</li>
	<li>1</li>
</ul>
```



```
// 하나하나 다루면 안됨. 몽땅 처리할 수 있도록 해야함
<style>
	#test li {
		display : none;
	}
	#test li.complete {
		display : block;
	}

	#test.check li {
		display : block;
	}
	
	#test.check li.complete {
		display : none;
	}
</style>

<script>
	//여기서 선택해서 없앰 하나하나씩 패칭해서 없애지말고 여러개를 한방에 처리하는게 중요하고, 선택자를 이용해서 없애는게 좋음
</script>

<ul id="test">
	<li>1</li>
	<li>1</li>
	<li class="complete">1</li>
	<li>1</li>
	<li>1</li>
	<li class="complete">1</li>
	<li class="complete">1</li>
	<li>1</li>
	<li>1</li>
	<li>1</li>
	<li>1</li>
	<li class="complete">1</li>
	<li>1</li>
	<li>1</li>
</ul>
```


# 9 호이스팅 #

```
test();
some();	//해당 줄에서 에러가 발생. 함수가 어떻게 선언되는지에 따라서 함수의 위치가 달라지게 됨
function test() {

}

var some = fucntion() {	

}
test();
some();
```

위의 코드는 실제 실행될 때 아래와 완전히 같음

```
function test() { // 어디서든 쓰고싶은 함수, 에러를 만날일이 없음

}
var some;	// 나중에 쓰고 싶은 함수
test();
some();	// define되지 않았기 때문에 정상적으로 동작하지 않음
some = fucntion() {

}
test();
some();
```



```
function test() { // 어디서든 쓰고싶은 함수, 에러를 만날일이 없음

}
var some;	// 나중에 쓰고 싶은 함수
test();
some();	// define되지 않았기 때문에 정상적으로 동작하지 않음
some = fucntion a() {
	a();	// 사용 가능
}
a();	// 에러가 남, 함수 안에서만 사용할 수 있는 함수임
test();
some();
```



```
$('.test').on('click',function click() {
	$(this).off(click);//이러기 위해서 사용함
});
```


# 10 이벤트 루프 #

