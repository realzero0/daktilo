---
layout: post
title:  "[Boost_Camp] WAS, 세션&쿠키, Maven"
subtitle: "시작 전 준비단계"
date:   2017-07-05 18:40:00
categories: [study]
---



# 프로젝트 시작전 준비단계

## 0. TODO 리스트 과제 리뷰

- 전체적으로 많이 실수했던 점 리뷰하기

## 1. 개발환경 설치

- JDK 설치 및 환경설정

- Eclipse 설치 및 환경설정

- Tomcat 설치 및 Eclipse와 연동하는 간단한 예제 작성

---
# 웹서버 vs WAS

정적페이지를 보여주느냐, 동적페이지를 보여주느냐의 차이

J2EE에 있는 EJB를 Tomcat이 실행시켜주지는 않음.

옛날에는 Tomcat만으로는 EJB를 사용하지 못했음.

Spring이 EJB가 하는 일을 대신해줘서 Tomcat이랑 Spring이랑 함께 사용하게 됨

conf/server.xml에서 포트 8080번을 바꿀 수도 있고, 거기에


<pre><code>&lt;Connector port=&quot;8080&quot; protocol=&quot;HTTP/1.1&quot;
               connectionTimeout=&quot;20000&quot;
               redirectPort=&quot;8443&quot; URIEncoding=&quot;utf-8&quot; /&gt;
</code></pre>


인코딩 추가함

# 이클립스에 적용
이클립스 설정에서 Sever탭에 서버를 추가함

그리고 Dynamic Web Project를 만들어보면
웹앱은 따로 메인메소드를 실행하지 않음.
톰캣의 도움을 항상 받아야함

규칙들이 있고, 약속들을 지키면서 만들어야하고, 서버에 실행하고 싶은게 무엇인지를 알려줘야함

Dynamic Web module을 2.5로 우선 시작

- Project
	- WEB-INF(외부에서 절대 접근이 안됨)
		- lib(라이브러리는 모두 어플리케이션이 포함해야함, maven은 pom.xml에 모든 라이브러리를 dependency로 써놓기만 하면됨)
		- classes(실제 우리가 만드는 클래스들이 들어감)
		- web.xml(모든 정보를 알려줌)
	- html
	- img


Servlet은 클래스이기 때문에 src디렉토리에 만들어져야함

서블릿은 서블릿을 상속받아야함

메소드는 오버라이딩되면 무조건 자식것을 사용

HttpSevlet을 상속하고, Source의 Overring메뉴를 눌러서
service destroy init함수를 오버라이딩함

서블릿을 하나 만들면 web.xml한테 알려줘야함

welcome-file은 url없이 표현될 파일들을 등록하는 것

<pre><code>&lt;servlet&gt;
      &lt;servlet-name&gt;&lt;/servlet-name&gt;
      &lt;servlet-class&gt;&lt;/servlet-class&gt;
  &lt;/servlet&gt;
  &lt;servlet-mapping&gt;
      &lt;servlet-name&gt;&lt;/servlet-name&gt;
      &lt;url-pattern&gt;&lt;/url-pattern&gt;
  &lt;/servlet-mapping&gt;</code></pre>
서블릿은 하나를 만들때 이렇게 한쌍씩 들어가야함

init은 처음 시작할때 수행되는 함수
service는 init이후 불려질때마다 수행되는 함수
destroy는 서버가 새롭게 갱신될 때 사용됨(서버가 접속 세션들을 끝내고 새로 갱신해야함. 종료해야될 때)

서블릿이 request, response 객체를 받아서 controller에 넘겨줌

// get과 post를 구분해서 처리할 경우 doGet, doPost함수를 각각 오버라이딩하면 됨(부모의 Service 메소드가 정의되지 않을 경우, 메소드가 들어오는 방식, post, get에 따라서 각각의 함수로 전달해줌)

서블릿 컨테이너는 서블릿을 올려놓고 동작시킴(서블릿이 독립적으로 동작할 수는 없음)
JSP 컨테이너는 JSP를 올려놓고 동작시킴 - servlet으로 바꿔줌

JSP는 여러가지 객체들이 내장 객체로 만들어져있음. 바로 세션이나 request객체를 불러올 수 있음


forward, redirect

# http 상태, 세션과 쿠키

http프로토콜은 상태유지가 안되는 프로토콜이기 때문에 세션과 쿠키를 사용하는 것


Client - 쿠키

서버는 쿠키를 클라이언트에 보내주고 매번 쿠키를 가지고 들어올때마다 쿠키를 확인하고 해당하는 정보를 보내줌


Server - 세션(내부적으로 쿠키를 이용함)

<pre><code>&lt;%    %&gt;</code></pre>

스크립틀릿

<pre><code>&lt;%=    %&gt;</code></pre>
응답결과를 바로 나타냄

쿠키를 저장하고, 그 쿠키를 리퀘스트객체를 이용해서 확인(setPath는 쿠키를 지우기에도 용이하게 함, maxAge를 -1로 하면 브라우저가 실행되는 동안 종료됨)
쿠키는 같은 이름이 두번 있을 수 없기 때문에 유지시간이 0인 같은 이름의 쿠키를 추가해서 지울 수 있다.

리다이렉트를 할 경우, 리퀘스트는 각각임. 각각 리스폰스에 담아서 가져와야함

// 요청이 들어올 경우 즉각 답이 필요할 경우 리퀘스트 객체에 잠깐 넣어놓기만 함, 짧은 경우엔 쿠키에 저장하는 편이 낫다. 긴 경우에는 세션에 넣는 편이 더 나음(로그인 등)

URL은 요청하나당 1개, 포워딩의 경우는 1개의 요청을 처리하는 것이고 리다이렉트의 경우에는 매번 새로운 요청을 생성해냄

Present Layer - Service Layer - Data Layer(DAO)
데이터 레이어에서는 CRUD 정도의 역할만 하고, 서비스에서 비지니스 로직을 처리함

DTO(data transfer object), VO(value object)는 가방에 물건을 넣듯이, 필요한 데이터들을 객체화해서 처리함


PreparedStatement 객체는 완벽하게 쿼리가 완성되지는 않음, Statement 객체는
			// 쿼리를 완전히 작성하고 사용, Prepared가 더 성능이 좋음, 쿼리문이 들어오면 컴파일 처럼 DBMS가 해석함,
			// PreparedStatement는 미리 해석해 놓은 것을 갖다 쓰기 때문에 값이 들어가는 자리만 바뀌게 되서 굉장히 성능향상에 좋음

세션은 서버에 저장을 하는 기술임, 세션 객체 자체가 key와 value로 이루어져 있음 key는 String이지만 value는 객체가 저장될 수 있음
세션id는 유일한 값이고 클라이언트마다 1개씩만 가지게 됨. request.getSession을 수행하면 세션아이디를 찾아서 해당하는 세션을 불러오던지, 세션이 없으면 세션을 생성하고 쿠키로 구운뒤 전달함. boolean값을 받는 경우가 있는데, false가 들어간 getSession은 세션이 있으면 return해주고 없으면 만들지는 않음. JSP는 자동으로 발급해주는데, 서블릿에서는 직접 가져다 써야함. 스프링에서는 세션 객체를 선언하면 스프링이 알아서 가지고 옴

session.removeAtribute로 하면 세션의 속성을 제거할 수 있음
session.invalidate을 하면 다 지워짐. 잘 사용하지는 않음

정말 계속 유지되야할 정보만을 세션에 넣어둘 것

모델2에서는 모든 요청을 서블릿 1개가 처리함. 스프링에서는 디스패처가 이를 처리함

# 포워딩

포워딩은 Servlet이 요청을 받아서 JSP에서 처리하라고 넘겨버리는 것을 의미함. 로직은 Servlet이 수행하고, 실제 뿌리는 것을 JSP가 수행하는 것

Request에다가 맡기는데, 이를 setAttribute로 입력하고, JSP가 받아오는 형태임

포원드로 보냈는지, 리다이렉트로 보냈는지 생각해야함


# Scope

Page Scope - 하나의 페이지에서만 존재

Request Scope - 요청이 되서 나갈때까지 있음(요청이 1개 - 하나의 요청을 계속 이어가면서 사용하는 것임)
Redirect는 요청이 바뀜(새로운 요청을 만들어냄)

Session Scope - 세션이 끝날 때까지 존재함

Application Scope - 서버가 시작할 때부터 끝날 때까지 있음(모두에게 공유하기 때문에 함부로 맡기면 안됨 - 전체 영역에서 공유해야할 때)



필터, 리스너

# ServletConfig

서블릿마다 하나씩 추상화되서 만들어짐. 이와 연결된 context는 Application Scope임




# 프론트엔드

템플릿 엔진을 통해서 markup을 통해서 data를 바인딩함 - 이게 더 효과적임

prop/attr/data의 차이 - 직접 찾아볼 것
attr은 HTML의 속성(Attribute)에 접근하는 함수이고
prop은 자바스크립트의 Property에 접근하는 함수이다. 

data는 해당하는 Element에 Javascript Type의 value를 저장하고 저장되어 있는 값을 읽음.

- 문법 : $(selector).data(key, value)
- key : string type의 변수로 data가 저장될 key값입니다.
- value : object type으로 JavaScript 에서 지원하는 모든 type의 데이터를 저장할수 있습니다.



<pre><code>$('.filters').children().children().removeClass();</code></pre>

이런 코드는 새롭게 무엇인가 DOME구조가 변경될 경우에 문제가 됨.

[CSS 선택자](http://www.clearboth.org/css3_1_by_isdn386/)는 반드시 알아야함


익명함수를 많이 쓰는 것은 굉장히 유지보수를 힘들게 함 -> 모듈 패턴을 학습할 것

[Module Pattern](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#modulepatternjavascript)


이벤트 델리게이션

이벤트를 하나씩 어태치하고 디태치하지 않음. 전체적으로 디태치하고 어태치하는게 일반적인 방법임. 상단에 핸들러를 달고, 하단에 추가하면 되는 식?

다음주는 이벤트 루프, 자주 실수하는 코드리뷰

---



- MySQL 설치 및 TEST를 위한 간단한 JDBC 프로그래밍 - 짝꿍의 MySQL에 접속하여 데이터 가지고 오기

---
#  Mysql 설치
- http://dev.mysql.com/downloads/mysql/ 에서 다운로드   
- http://withcoding.com/26 설치시 참고
- path C:\Program Files\MySQL\MySQL Server 5.7\bin 추가
  - cmd에서 실행 가능
  - mysql -u 사용자명  -p   

  - 사용자 생성
     - create user userid
     - create user userid@localhost identified by 'password'
     - create user 'userid'@'%' identified by 'password'
  - 사용자 제거
    - drop user 'userid';
    - delete from user where user ='userid'
  - 권한
    - GRANT ALL on DB명.* TO userid@'localhost'
    - GRANT ALL on DB명.* TO userid;
    - GRANT ALL on DB명.* TO userid@'xxx.xxx.xxx.%'
  - 데이터베이스 생성
    - create database DB명

  - 명령들
    - show tables;
    - show databases;
    - use DB명

  - 테이블 생성
    - create table test(
id varchar(10) primary key,
name varchar(32) not null,
password varchar(10) not null);

  - 데이터입력
    - insert into test values ('carami','강경미','carami');
  - 데이터조회
    - select * from test;
  - 데이터 수정
    - update test set password = '1234';
  - 데이터삭제
    - delete from test where id = 'test';

    ## mysql 에 데이터베이스와 사용자를 추가한다.

    mysql -uroot -proot mysql

    // tododb 데이터베이스 생성
    create database tododb;

    // id : carami , password : carami
    create user 'carami'@'%' identified by 'carami';

    // carami에게 tododb에 대한 모든 권한을 줌
    GRANT ALL on tododb.* TO carami@'%'

    // 위에서 설정한 내용이 바로 적용되도록 한다.
    FLUSH PRIVILEGES;

- 테이블 생성

	CREATE  TABLE `MEMBER` (
    `id` INT NOT NULL AUTO_INCREMENT,
    `name` VARCHAR(50) NULL ,
    `email` VARCHAR(100) NULL ,
    `passwd` VARCHAR(50) NULL ,
    PRIMARY KEY (`id`)
    );

    외부에서 접근 가능하도록 방화벽에서 3306 포트를 열어놓는다.

# JDBC 프로그래밍
- DB접속 테스트
<pre><code>
  public class ConnectionTest {
	   public static void main(String[] args) throws SQLException {
		     Connection connection = null;		
		     connection = DriverManager.getConnection(&quot;jdbc:mysql://localhost:3306/caramidb&quot;, &quot;carami&quot;, &quot;carami&quot;);
		     if(connection != null){
			        System.out.println(&quot;success&quot;);
			        connection.close();
		     }else
			      System.out.println(&quot;fail&quot;);						
	   }
   }</code></pre>

  - mysql-connector-java-5.1.42-bin.jar 라이브러리 추가




# Connection
Connection이라는 객체를 리턴, 껍데기만 정의해서 줌.
어떤 DB를 사용하고 있던지, 인터페이스에 정의된 함수만
Overriding되어 있으면 됨. 그렇기 때문에 어떤 DB던지 사용가능
각각의 getConnection으로 나온 객체는 Connection 인터페이스를 구현한 객체를 리턴함. 인터페이스를 상속한 객체는 인터페이스를 가리킬 수 있기 때문에 가능. 자바6부터는 Class.ForName이 없어도 가능.




- Junit을 이용한 Conection 테스트

    JDBC Connection Test

    src/test/java

    에

    carami.jdbc 패키지를 생성한다. 해당 패키지에 다음과 같은 클래스를 작성한다.

<pre><code>package carami.jdbc;


    import org.junit.Assert;
    import org.junit.Test;

    import java.sql.Connection;
    import java.sql.DriverManager;

    public class JdbcTest {

    	@Test
    	public void connectionTest() throws Exception{
    		Connection connection = null;
    		connection = DriverManager.getConnection(&quot;jdbc:mysql://localhost:3306/tododb&quot;, &quot;carami&quot;, &quot;carami&quot;);
    		Assert.assertNotNull(connection);
    	}
    }</code></pre>
// 
Junit 3은 testcase라는 객체를 반드시 extends해야한다는 약점이 있음
Junit 4는 POJO(어떤 객체에도 종속되지 않은 객체)를 충실히 따름. 상속관계가 아니더라도 상속하는 것처럼 내부에서는 사용될 수 있음(어노테이션을 대신 사용, 결합도를 낮출 수 있음)

import static은 해당 객체의 static메소드를 사용한다는 의미임

    - DB입력 테스트
<pre><code>public class InsertTest {
  public static void main(String[] args) throws Exception {
    Connection connection = null;
    PreparedStatement ps = null;
    connection = DriverManager.getConnection(&quot;jdbc:mysql://localhost:3306/caramidb&quot;, &quot;carami&quot;, &quot;carami&quot;);
    String sql = &quot;insert into member values (?,?,?)&quot;;
    ps = connection.prepareStatement(sql);
    ps.setString(1, &quot;testID&quot;);
    ps.setString(2, &quot;테스트&quot;);
    ps.setString(3, &quot;test&quot;);

    int resultCount = ps.executeUpdate();		
    System.out.println(resultCount);
  }
  }</code></pre>

---

1 maven 설치

- IDE( eclipse , intelliJ 등)에 기본 내장되어 있을 수 있으나 터미널에서 독립적으로 실행할 수 있도록 설치를 한다.
- https://maven.apache.org/download.cgi 에서 다운로드
  - 이 글을 쓰는 시점에서는 가장 최신 버전인인 apache-maven-3.5.0-bin.tar.gz 를 다운로드 한다.
  - 특정 폴더에 압축을 해제한다.  /boost/apache-maven-3.3.9 로 압축을 해제한다.
  - path에 /boost/apache-maven-3.3.9/bin 을 추가한다.
  - 터미널을 새롭게 열고 mvn -v 명령으로 명령이 실행되는지 확인한다.

2 maven 프로젝트 생성

1) Eclipse를 실행한다.
  
2) new - project -> maven - maven project -> next 버튼 클릭
     -> Create a simple project 선택, use default workspace location 선택 -> next 버튼 클릭
     -> goal id : carami (보통 도메인 이름을 거꾸로 적음)
        Artifact id : todo (프로젝트 이름을 보통 소문자로 씀)
        packaging : war (웹 어플리케이션 프로젝트니깐) -> finish 버튼 클릭
  
3) 처음 프로젝트를 생성하면 프로젝트에 오류표시가 납니다. 웹 어플리케이션인데 필요한 파일이 존재하지 않기 때문입니다. 참고로 maven은 기본설정이 jdk 1.5 로 되어 있습니다. problems 부분에서 maven - update project를 하라는 메시지가 있다면, 이클립스에서 프로젝트를 선택 후에 우측버튼을 클릭하고 maven - udpate project를 선택합니다. maven 프로젝트의 설정파일은 pom.xml 입니다. 해당 파일을 다음과 같이 수정합니다.

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>carami/groupId>
    <artifactId>todo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>	<!-- 상수라고 생각하면 됨, 계속 바뀔 것들을 넣는게 좋음 -->
		<jdk-version>1.8</jdk-version>
		<source-encoding>UTF-8</source-encoding>
		<resource-encoding>UTF-8</resource-encoding>

		<!-- spring framework -->
		<spring-framework.version>4.3.5.RELEASE</spring-framework.version>
	</properties>

	<dependencies> <!-- 실제 라이브러리를 추가하는 부분 -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>${spring-framework.version}</version> <!-- 상수값을 사용할 때는 이렇게 사용하면 됨 -->
		</dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring-framework.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring-framework.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>${spring-framework.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-expression</artifactId>
            <version>${spring-framework.version}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.3</version>
                <configuration>
                    <source>${jdk-version}</source>
                    <target>${jdk-version}</target>
                    <encoding>${source-encoding}</encoding>
                    <useIncrementalCompilation>false</useIncrementalCompilation>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```
