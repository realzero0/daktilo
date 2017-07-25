---
layout: post
title:  "[Boost_Camp] BE FE 코드리뷰"
subtitle: "4주차"
date:   2017-07-25 20:40:00
categories: [study]
---

# BE #

환경에 따른 파일 설정

패스워드 설정

서비스나 DAO에 리퀘스트를 넣는 것은 매우 좋지 않음

properties를 빼는 것을 유동적인 부분에 대해서 해야함 값을 바로 집어 넣는 것은 좋지 않음


# FE #

## 모듈 vs 컴포넌트 ##

모듈? 서로 구분되고, 서로 관련되어 있는 부분들의 집합

몇 개의 상수, 인스턴스, 클래스들의 모음이라는 의미임

npm에서 설치하는 모듈들도 하나의 모듈임

모듈은 일반적으로 배포를 염두에 두고 있는 단위


## 컴포넌트? ##

주로 뷰의 구성요소를 말할 때, 뷰의 일부 요소

뷰 -> MVC에서의 view는 모델을 표현하는 것임 -> 모델을 떠나서는 의미가 없음

뮤직관련 뷰를 만들고 앨범에 관한 뷰를 보여준다고 치자

우리는 거기에 다양한 데이터를 넣고 그 뷰를 보여줄 것이다.

그 뷰에 여러가지 인풋으로 별점을 바꾸거나, 음량을 조절하거나, 다음 앨범으로 옮겨 갈 것이다.

그런면에서 컴포넌트는 기능적으로 UI에 한 부품이 되는 부분을 의미한다.

스피너, 달력 컴포넌트 등을 UI 컴포넌트라고 한다.


## Observer 패턴 ##

의존성을 줄이지만, 의존성 문제를 줄이기 위해 Observer 패턴을 쓰면 안된다.

플레이어 컴포넌트에서, 플레이버튼을 눌러서 플레이가 시작되면,

감춰져 있는 가사를 보여주려고 한다. 그러기 위해서는 플레이되는 시점을 아는 것이 필요하다.

가사는 플레이어를 의존하는 편이 훨씬 낫다. 그 반대의 경우보다.

플레이어의 플레이 버튼이 눌러질 때 가사가 열려야 한다.

```
Player player = new Player();
//아래와 같은 코드는 의존성을 증가시킴
player.setLylicViewer(new LylicViewer());

//의존관계가 현저히 줄어듬
player.on("play");


//아래와 같은 코드는 의존관계가 명백히 드러남, 결국 의존관계인 것임
player.on("showLylic");
```

옵저버는 누가 누군지 몰라야 의미가 있음


```
//var musicService = new MusicService();	// 이런 구조가 더 좋은 구조임
var player = new Player('.player');
var lyricViewer = new LyricViewer('.liricView', player);

function LyricViewer(selector, player) {
	this.$element = $(selector);
	this.player = player;
	this.player.on('playstart', this._onPlayStart.bind(this));
	this.showAlbumCover();
}

LyricViewer.prototype._onPlayStart = function() {
	this.showLyric();
}

function Player(selector) {
	this.$element = $(selector);
	this.$playButton = this.$element.find('.playButton');
	this.$playButton.on('click', this._onClickPlayButton.bind(this));
}

Player.prototype._onClickPlayButton = function() {
	this.togglePlayButton();
	this.trigger('playstart');
}
```


## if else 에서 코드 줄이기 ##

```
for(var i = 0; i < data.length ; i++){
                if(data[i].priceType === 1){
                    price += "성인(만 19 ~ 64세)"+addCommaPrice(data[i].price)+"원";
                    addPrcieInfo("성인", addCommaPrice(data[i].price), addCommaPrice(data[i].price*(1-data[i].discountRate)), data[i].discountRate*100);
                }else if(data[i].priceType === 2){
                    price += "/ 청소년(만 13 ~ 18세)"+addCommaPrice(data[i].price)+"원";
                    addPrcieInfo("청소년", addCommaPrice(data[i].price), addCommaPrice(data[i].price*(1-data[i].discountRate)), data[i].discountRate*100);
                }else if(data[i].priceType === 3){
                    price += "/ 어린인(만 4 ~ 12세)"+addCommaPrice(data[i].price)+"원";
                    addPrcieInfo("어린이", addCommaPrice(data[i].price), addCommaPrice(data[i].price*(1-data[i].discountRate)), data[i].discountRate*100);
                }else if(data[i].priceType === 4){
                    price += "/ 국가유공자, 장애인, 65세 이상"+addCommaPrice(data[i].price)+"원";
                    addPrcieInfo("국가유공자, 장애인, 65세 이상",addCommaPrice(data[i].price), addCommaPrice(data[i].price*(1-data[i].discountRate)), data[i].discountRate*100);
                }
            }
}

```


```
var TICKET_PRICE_STR = {
	'0': {desc:
	'1': 
}
var priceDesc = [];

data.forEach(function(item) {
	priceDesc.push()
});
```

## 적절한 인자의 갯수에 대해서 ##

인자가 3개에서 4개가 넘어가면 parameter obj를 생각해보는 것이 좋다.

```
function ajax(method, url, contentType, data, productId) {
	
	
}

new ajax('GET', '/products', null, null, 'a1');	// 이런식으로하면 가독성도 떨어지고 간결하지 못함



function ajax(params) {
	var obj = {};
	Object.assign(obj,
	this.options = Object,assign({}, {	// 첫번째 파라미터는 빈 객체임
		method: 'GET',
		url: null,
		contentType: 'application/json',
		data: null,
		productId: null
	}, params);

}	

//이런 경우에는 아래와 같이 하는 게 좋음, 아래와 같은 경우가 가독성이 더 좋고 간결함
new ajax({
	url: '/prouducts',
	productId: 'a1'
});
```

들여쓰기는 잘해야 한다.

코드에서 버그가 나는 것은 괜찮아도, 들여쓰기는 잘해줘야함


## 이름 짓기 ##

이름을 짓는데는 모두 이유가 있어야 한다.

DetailModule이라는 이름에 대해,,,,
*Data, *Info, *Obj, *Module, *Manager...

데이터의 본질적인 속성, 클래스의 경우에는 이 클래스가 어디에 속하는지,

레이어 상에서의 역할, 다루는 데이터 등에 대한 조합이어야한다.

suffix로서 필요하다고 생각한다면 집어넣고, 필요없다면 없애는 편이 났다.

Manager는 일반적으로 프로그램의 수명주기를 관리 할 수 있을 때 사용한다.


스크립트 태그에 async, defer 속성을 넣어주면 즉시실행, 돔이 다 로드된 뒤에 실행으로 설정할 수 있다.

자바스크립트가 멀티스레드일 경우에는 무엇이 문제일까?

```
window.a = 10;
console.log(window.a);
```
10이 나온다.

멀티쓰레드일 경우에는 어떻게 바뀔지 모른다.

싱글쓰레드가 UI 제어에 훨씬 효과적이기 때문에 달라진다.



읽기과 쓰기를 동시에 사용하는 함수는 만들지 않는게 좋다.

isDone()이런 상태 판단 함수에는 변경 기능을 넣어서는 안된다.

set하는 함수에는 판단하는 코드를 넣지 않는게 좋다. set의 결과에 대한 값만 넣는 게 좋음


## console.log() vs alert() vs throw new Error() ##

js에서의 Error처리는 절대 그냥 넘기지 말 것

에러처리를 넣은 이유는 에러가 나면 코드는 실행을 멈추어야 한다.

프로그램의 흐름을 끊고 넘어가야 함.

파라미터가 없어서 에러가 나면 더 이상 진행하기가 불가능함

에러가 나는 쪽과 에러를 처리하는 쪽을 분리하는데,

logging하고 버리는 쪽은 UI바로 직전에서 처리함

에러를 throw하면 스택을 다 버려버릴 수 있음


## 여러가지 jquery를 동시에 사용할 때 ##
```
// 변경된 사항이 올 때만 써도 아무 문제가 없음
(function(window) {
	window.open(url);
})(window);
```