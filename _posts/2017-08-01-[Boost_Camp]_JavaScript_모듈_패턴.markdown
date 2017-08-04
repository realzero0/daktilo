---
layout: post
title:  "[Boost_Camp] JavaScript 모듈 패턴"
subtitle: "5주차"
date:   2017-08-01 14:40:00
categories: [study]
---

#


```
  function Ticket() {

  }
  Ticket.prototype = new eg.Component();
  Ticket.prototype.constructor = Ticket;


  var TicketController = (function() {
    function plus() {

    }
    function minus() {

    }
    return {
      'init' : function() {
        $('#ticket1').on('click', '.plus', plus);
        $('#ticket1').on('click', '.minus', minus);
      }
    }

  })();

  //트리거는 컴포넌트 내부에서 해야함
```

아래와 같이 바꿀 수 있다.


```
function Ticket() {
  this.countEle = $(id + ' .count');
  $('#ticket1').on('click', '.plus', this.plus.bind(this));
  $('#ticket1').on('click', '.minus', this.minus.bind(this));
}
Ticket.prototype = new eg.Component();
Ticket.prototype.constructor = Ticket;
Ticket.prototype.plus = function(e) {
  var count = parseInt(this.countEle.text(), 10);
  this.countEle.text(++count);
  console.log('plus');
}
Ticket.prototype.minus = function(e) {
  var count = parseInt(his.countEle.text(), 10);
  this.countEle.text(--count)
  console.log('minus');
}
var TicketController = (function() {
  ['a', 'b', 'c'].forEach(function(v, i) {

  });
  $.forEach(function(v, i) {

  });
  // 아래와 같이 반복문으로 쓸 수 있고, each는 i, v의 순서다.
  $('div').each(function(i, v) {
    new Ticket('#' + $(v).attr('id'))
  });
})();
```

컴포넌트를 만들때는 고유한 기능을 가진 하나의 컴포넌트를 만든다.

서비스 컴포넌트, DAO를 어떻게 되는지를 전체적으로 설명할 수 있어야 함.

팀별로 그 부분들의 코드를 보여주면서 설명해야한다.