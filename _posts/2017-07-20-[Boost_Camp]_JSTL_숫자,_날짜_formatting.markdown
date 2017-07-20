---
layout: post
title:  "[Boost_Camp] JSTL_숫자,_날짜_Formatting"
subtitle: "3주차"
date:   2017-07-20 20:40:00
categories: [study]
---


```
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
```


formatNumber
formatDate
parseDate :  문자열로 표시된 날짜를 Date 객체로 변환
parseNumber : 문자열로 표시된 날짜를 숫자로 변환
setTimezone
timeZone

-----------------------------------------------------------------------------
# 1 formatNumber #

value : 숫자
type : number (숫자), percent (%), currency (통화)
groupingUsed : true / false -- 구분자(,)
currencyCode : 통화코드(KRW)
currentSymbol : 통화표시기호
var : 저장할 변수 이름
scope : 변수 적용 범위
pattern : 직접 출력 양식 지정. java.text.DecimalFormat 클래스에서 정의된 패턴 사용


```
<c:set var="price" value="10000" />
<fmt:formatNumber value="${price}" type="number" var="numberType" />
```


통화

```
<fmt:formatNumber value="${price}" type="currency" currencySymbol="원" /> <br>
```

퍼센트
```
<fmt:formatNumber value="${price}" type="percent" groupingUsed="false" /> <br>
```

숫자
```
${numberType}
```

-----------------------------------------------------------------------------
# 2 formatDate #

value : 데이터
type : date (날짜만), time (시간만), both (날짜시간)
dateStyle : full, long, default, medium, short (java.text.DateFormat에서 지정한 스타일)
timeStyle : java.text.DateFormat에서 지정한 스타일 사용
pattern : 직접 출력 양식 지정. java.text.SimpleDateFormat에 정의된 패턴 사용
var : 변수
scope : 변수 적용 범위
timeZone : 시간대 변경. setTimeZone 태그에서 명시



```
<c:set var="now" value="<%= new java.util.Date() %>" />
<fmt:formatDate value="${now}" type="time" /> <br>
<fmt:formatDate value="${now}" type="date" dateStyle="full" /> <br>
<fmt:formatDate value="${now}" type="date" dateStyle="short" /> <br>
<fmt:formatDate value="${now}" type="both" dateStyle="full" timeStyle="full" /> <br>
<fmt:formatDate value="${now}" pattern="z a h:mm" /> <br>
```

-----------------------------------------------------------------------------
# 3 setTimezone, timeZone #


```
<c:set var="now" value="<%= new java.util.Date() %>" />

<fmt:formatDate value="${now}" type="both" dateStyle="full" timeStyle="full" /> (기본시간대) <br>

<fmt:timeZone value="Hongkong">
      <fmt:formatDate value="${now}" type="both" dateStyle="full" timeStyle="full" /> ( Hongkong )
</fmt:timeZone>
```



시간대 목록

```
<%
    String[] ids = java.util.TimeZone.getAvailableIDs();
    for (int i = 0 ; i < ids.length ; i++) {
         out.println(ids[i]+"<br>");
    }
%>
```

-----------------------------------------------------------------------------


# 세자리마다 콤마찍어주는 함수 #

```
function number_format(num){ 
	return String(num).replace(/(\d)(?=(?:\d{3})+(?!\d))/g, '$1,'); 
};
```