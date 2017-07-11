---
layout: post
title:  "[Boost_Camp] JSTL, EL, Paging"
subtitle: "2주차"
date:   2017-07-11 19:40:00
categories: [study]
---

# jstl (JSP Standard Tag Library)
- JSP 페이지에서 논리적인 판단,반복문의 처리, 데이터베이스 등의 처리를 하는 코드를 깔끔하게 작성하기
위해서 작성한 표준화된 커스텀 태그이다.

```
<%
    if (list.size() > 0) {
        for (int i = 0 ; i < list.size() ; i++) {
            Categoery category = (Categoery) list.get(i);
%>
    <%= category.getName() %>
    ...
<%
        }
    } else {
%>
    데이터가 없습니다.
<%
    }
%>

```
위와 같은 코드를 아래처럼 쓸 수 있다.

```
<c:if test="!empty ${list}">
   <c:foreach varName="category" list="${list}">
      ${category.name}
   </c:foreach>
</c:if>
<c:if test="empty ${list}">
      데이터가 없습니다.
</c>
```

## jstl이 제공하는 태그의 종류
|라이브러리|하위 기능|접두어|관련URI|
|---|---|---|---|
|코어|변수지원,흐름 제어,URL 처리|c|http://java.sun.com/jsp/jstl/core |
|XML|XML 코어,흐름 제어,XML 변환|x|http://java.sun.com/jsp/jstl/xml |
|국제화|지역,메시지 형식,숫자 및 날짜 형식 |fmt|http://java.sun.com/jsp/jstl/fmt |
|데이터베이스|SQL|sql|http://java.sun.com/jsp/jstl/sql |
|함수|콜렉션 처리,String 처리|fn|http://java.sun.com/jsp/jstl/functions |

## 코어태그 : 변수 지원 태그 - set, remove
*변수 설정 : 지정한 영역에 변수를 생성한다. *
```
<c:set var="varName" scope="session" value="someValue" />
<c:set var="varName" scope="request">
some Value
</c:set>
```
var - EL에서 사용될 변수명
scope - 변수값이 저장될 영역(page, request, session, application)
value - 변수값

*변수 제거*
```
<c:remove var="varName" scope="request" />
```

*프로퍼티, 맵처리 *
```
<c:set target="${member}" property="propertyName" value="anyValue" />

//member 객체가 자바빈일 경우: member.setPropertyName(anyvalue)
//member 객체가 맵(map)일 경우: member.put(propertyName, anyValue);

/*
target -<c:set>으로 지정한 변수 객체
property - 프로퍼티 이름
value - 새로 지정할 프로퍼티 값
*/
```

## if

**문법**
```
<c:if test="조건">
...
...
</c:if>
```

**예제**
```
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<c:if test="true">
무조건 수행
</c:if>

<c:if test="${param.name == 'kang'}">
param.name 의 값이 kang 일때 수행
</c:if>

<c:if test="${3000 < param.price}">
price가 3000보다 클때 수행
</c:if>

```

## choose

**문법**
```
<c:choose>
  <c:when test="조건1" >
    조건1일 true일때 수행
  </c:when>
  <c:when test="조건2" >
    조건 2가 true일때 수행
  </c:when>
  <c:otherwise>
    위에 나온 조건들이 모두 만족하지 않을때 실행
  </c:otherwise>
</c:choose>
```
## forEach  -  배열 및 Collection에 저장된 요소들을 차례로 수행

**문법**
```
<c:forEach var="변수" items="아이템" [begin="시작번호"] [end="끝번호"]>
${변수}
</c:forEach>

/*
var - EL에서 사용될 변수명
items - 배열, List, Iterator, Enumeration, Map 등의 Collection
begin - items에 지정한 목록에서 값을 읽어올 인덱스의 시작값
end - item에 지정한 목록에서 값을 읽어올 인덱스의 끝값

item이 Map인 경우 변수에 저장되는 객체는 Map.Entry이다.
따라서, 변수값을 사용할 때는 ${변수.key}와 ${변수.value}를 사용해서 맵에 저장된 항목의 <키, 값> 매핑에 접근할 수 있다.

*/
```

## redirect - 지정한 페이지로 리다이렉트. response.sendRedirect()와 비슷하게 사용

**문법**
```
<c:redirect url="리다이렉트할URL">
    <c:param name="파라미터이름" value="파라미터값" />
</c:redirect>

```
- url - 리다이렉트 URL
- <c:param>은 리다이렉트할 페이지에 전달할 파라미터 지정
**예제**
```
<c:redirect url="/ifTag.jsp">
    <c:param name="name" value="bk" />
</c:redirect>
```
## out - JspWriter에 데이터를 출력한다
문법
```
<c:out value="value" escapeXml="{true|false}" default="defaultValue" />

```

- value - JspWriter에 출력할 값을 나타낸다. 일반적으로 value 속성의 값은 String과 같은 문자열이다. 만약 value의 값이 - - - java.io.Reader의 한 종류라면 out 태그는 Reader로부터 데이터를 읽어와 JspWriter에 값을 출력한다.
- escapeXml - 이 속성의 값이 true일 경우 아래 표와 같이 문자를 변경한다. 생략할 수 있으며, 생략할 경우 기본값은 true이다.
- default - value 속성에서 지정한 값이 존재하지 않을 때 사용될 값을 지정한다


## catch - 태그 몸체에서 발생한 예외를 변수에 저장
**문법**
```
<c:catch var="exName">

   예외가 발생할 수 있는 코드 (에러가 발생하면 에러가 exName 변수에 저장)

   </c:catch>

   ${exName} 사용
```
-  var - 예외 객체를 저장할 변수명


# EL (Expression Language)
## 사용목적
- 값을 표현하는 데 사용되는 새로운스크립트 언어로서 JSP의 기본 문법을 보완하는 역할을 한다.  
- <%= %> , out.println()과 같은 자바코드를 더 이상 사용하지 않고 좀더 간편하게 출력을 지원하기 위한 도구.
  배열이나 컬렉션에서도 사용되고, JavaBean의 프로퍼티에서도 사용된다.


## 기능
- JSP의 네 가지 기본 객체가 제공하는 영역의 속성 사용
- 집합 객체에 대한 접근 방법 제공
- 수치 연산, 관계 연산, 논리 연산자 제공
- 자바 클래스 메소드 호출 기능 제공
- 표현언어만의 기본 객체 제공

## 문법
- ${expr}
- 예) ${sessionScope.member.id}  

## 기본객체
|기본객체 | 설명 |
| -------|-------------|
|pageContext|JSP의 page 기본 객체와 동일|
|pageScope |pageContext 기본 객체에 저장된 속성의 <속성, 값> 매핑을 저장한 Map 객체|
|requestScope|request 기본 객체에 저장된 속성의 <속성, 값> 매핑을 저장한 Map 객체|
|sessionScope|session 기본 객체에 저장된 속성의 <속성, 값> 매핑을 저장한 Map 객체|
|param|요청 파라미터의 <파라미터이름, 값> 매핑을 저장한 Map 객체. 파라미터 값의 타입은 String 으로서, request.getParameter(이름)의 결과와 동일 |
|paramValues|요청 파라미터의 <파라미터이름, 값배열> 매핑을 저장한 Map 객체. 값의 타입은 String[] 으로서, request.getParameterValues(이름)의 결과와 동일|
|header|요청 정보의 <헤더이름, 값> 매핑을 저장한 Map 객체. request.getHeader(이름)의 결과와 동일 |
|headerValues|요청 정보의 <헤더이름, 값 배열> 매핑을 저장한 Map 객체. request.getHeaders(이름)의 결과와 동일|
|cookie|<쿠키 이름, Cookie> 매핑을 저장한 Map 객체. request.getCookies()로 구한 Cookie 배열로부터 매핑을 생성|
|initParam|초기화 파라미터의 <이름, 값> 매핑을 저장한 Map 객체. application.getInitParameter(이름)의 결과와 동일|

### 데이터 타입
- 불리언 타입 - true 와 false

- 정수타입 - 0~9로 이루어진 정수 값 음수의 경우 '-'가 붙음.

- 실수타입 - 0~9로 이루어져 있으며, 소수점('.')을 사용할 수 있고,
    3.24e3과 같이 지수형으로 표현 가능하다.

- 문자열 타입 - 따옴표( ' 또는 " )로 둘러싼 문자열. 만약 작은 따옴표(')를 사용해서 표현할 경우 값에 포함된 작은 따옴표는 \' 와 같이 \ 기호와 함께 사용해야 한다. \ 기호 자체는 \\ 로 표시한다.

- 널 타입 - null

## 객체 접근 규칙
- 문법   ${<표현1>.<표현2>}
1. <표현1>을 <값1>로 변환한다.
2. <값1>이 null이면 null을 리턴한다.
3.  <값1>이 null이 아닐 경우 <표현2>를 <값2>로 변환한다.
  -  <값2>가 null이면 null을 리턴한다.
4. <값1>이 Map, List, 배열인 경우
  - <값1>이 Map이면
    *  <값1>.containsKey(<값2>)가 false이면 null을 리턴한다.
    *  그렇지 않으면 <값1>.get(<값2>)를 리턴한다.
  - <값1>이 List나 배열이면
    * <값2>가 정수값인지 검사한다. (정수값이 아닐 경우 에러 발생)
    * <값1>.get(<값2>) 또는 Array.get(<값1>, <값2>)를 리턴한다.
    * 위 코드가 예외를 발생하면 에러를 발생한다.
5. <값1>이 다른 객체이면
  - <값2>를 문자열로 변환한다.
  - <값1>이 이름이 <값2>이고 읽기 가능한 프로퍼티를 포함하고 있다면 프로퍼티의 값을 리턴한다.
  - 그렇지 않을 경우 에러를 발생한다.

## 수치 연산자
- + : 덧셈
- - : 뺄셈
- * : 곱셈
- / 또는 div : 나눗셈
- % 또는 mod : 나머지
```
숫자가 아닌 객체와 수치 연산자를 사용할 경우 객체를 숫자 값으로 변환후 연산자를 수행:  ${"10"+1} → ${10+1}
숫자로 변환할 수 없는 객체와 수치 연산자를 함께 사용하면 에러를 발생:  ${"열"+1} → 에러
수치 연산자에서 사용되는 객체가 null이면 0으로 처리:  ${null + 1} → ${0+1}
```
## 비교 연산자
-  == 또는 eq
- != 또는 ne
- < 또는 lt
- > 또는 gt
- <= 또는 le
- >= 또는 ge
- 문자열 비교: ${str == '값'} str.compareTo("값") == 0 과 동일

## 논리연산자
- && 또는 and
- || 또는 or
- ! 또는 not

## empty 연산자
- 문법    
```
empty<값>
```
- <값>이 null이면 true를 리턴한다.
- <값>이 빈 문자열("")이면 true를 리턴한다.
- <값>이 길이가 0인 배열이면 true를 리턴한다.
- <값>이 빈 Map이면 true를 리턴한다.
- <값>이 빈 Collection이면 true를 리턴한다.
- 이 외의 경우에는 false를 리턴한다.

##  비교선택 연산자
- 문법  
```
<수식> ? <값1> : <값2>
```
- <수식>의 결과값이 true이면 <값1>을 리턴하고, false이면 <값2>를 리턴



# PAGING #

# LIMIT

- SELECT * FROM CATEGORY LIMIT 10;
> CATEGORY 테이블의 레코드를 처음부터 10건 읽어옵니다.  

- SELECT * FROM CATEGORY LIMIT 10 OFFSET 15;
> CATEGORY 테이블의 레코드를 15번째부터 10건 읽어옵니다.  LIMIT는 가져올 row 의 수를 지정하고, OFFSET은 몇 번째 row부터 가져올건지 결정합니다.  

- SELECT * FROM Orders LIMIT 15, 10;   
> 위의 쿼리와 결과가 같습니다.  15가 몇 번째부터 가져올것인지를 결정하고, 10이 몇 건을 가져올 것인지를 결정합니다.  



# 참고

- 오라클 DB는 LIMIT를 제공하지 않습니다.  아래 쿼리처럼 rownum을 이용해서 결과를 얻어와야 합니다.  
select r, * from (

select rownum r, * from (

select * from CATEGORY order by name desc )

where rownum <=25)

where r>=15

> 결과가 15번 row부터  25번 row 까지 읽어옵니다.   
rownum 이 먼저 결정되고 정렬이 됨으로 쿼리문 안에 정렬하는 구문이 있을때는 반드시 보기처럼
수행해야 원하는 결과를 얻어올 수 있습니다.  
