---
layout: post
title:  "[Boost_Camp] Spring Dispatcher 정리"
subtitle: "시작 전 준비단계"
date:   2017-07-05 20:40:00
categories: [study]
---



- Spring MVC에 대하여 이해하고 이를 이용해 개발 할 수 있다.
	- 하다보니까 더 좋은것들을 써먹으려고 함 -> 디자인 패턴은 원래 건축학에서 유래됨
	- 웹에서 들어오는 입구를 하나로 만들려고 함


[dipatcher servlet](https://www.javatpoint.com/spring-3-mvc-tutorial)

[추가 참조](http://wiki.gurubee.net/display/DEVSTUDY/01.DispatcherServlet)

![](http://wiki.gurubee.net/download/attachments/26740984/7_dispatcherservlet.png)

모든 요청을 1개가 다 받자는 것. Dispatcher servlet은 모든 요청을 받음. 또한 특정 URL패턴의 요청만 받도록 제한할 수 있음. Dispatcher servlet은 요청이 들어오면 핸들러에게 어떤 컨트롤러를 불러올지를 물어봄. 이렇게 분리되어있는 로직은 Dispatcher servlet의 과부하를 줄여주고 보안성을 높임

1 리퀘스트가 Dispatcher Servlet에 들어오면

2 Dispatcher Servlet은 해당 요청을 처리할 컨트롤러를 찾기 위해 Handler mapping의 도움을 받는다.(xml, 객체 모두를 이용할 수 있느나 현재는 객체를 이용하는 게 더 많음)

2.5 핸들러 매핑에서 적당한 컨트롤러를 찾아서 매핑함(delegate request, 포워딩, 위임함)

3 요청을 수행할 컨트롤러에게 해당 요청에 대한 비즈니스 로직 기능을 수행하도록 지시(handleRequest())

4 컨트롤러는 모델앤뷰 객체를(스트링 객체를 반환하거나) 반환한다
 - Model : 전달해줄 정보, 스프링이 자동으로 Request 객체에 넣어주고 View에 보여줄 수 있게 만든다.
 - View : jsp 등 view에 사용되는 웹 언어/형식(쉽게 말하면 jsp의 위치가 어디냐 ex) board/list.jsp 인데 여기서 list인 값만 리턴해줌

5 컨트롤러는 이름(list)만 반환하기 때문에 뷰리졸버를 이용해 데이터와 결합한다
 - 대표적으로 prefix, 서피스 등임(list가 들어오면 -> board/list.jsp로 변환해줌)
 - 뷰의 위치를 정해서 최종적으로 보내는 것임
 - 뷰의 역할을 하는 객체에 포워딩하는 것, 이름만 넘기도록 약속한 것
 - 뷰 리졸버는 실제 뷰들 포워딩함

6 뷰에서 보여준다


Spring mvc 를 사용하기 위한 pom.xml

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>carami</groupId>
	<artifactId>todo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>

	<properties>
		<jdk-version>1.8</jdk-version>
		<source-encoding>UTF-8</source-encoding>
		<resource-encoding>UTF-8</resource-encoding>
		<deploy-path>deploy</deploy-path>

		<!-- spring framework -->
		<spring-framework.version>4.3.5.RELEASE</spring-framework.version>

		<logback.version>1.1.3</logback.version>
		<jcl.slf4j.version>1.7.12</jcl.slf4j.version>

	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${spring-framework.version}</version>
		</dependency>

		<!--Servlet -->
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>jstl</artifactId>
			<version>1.2</version>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>3.1.0</version>
			<scope>provided</scope>
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

			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-clean-plugin</artifactId>
				<version>2.6.1</version>
			</plugin>

			<plugin>
				<groupId>org.apache.tomcat.maven</groupId>
				<artifactId>tomcat7-maven-plugin</artifactId>
				<version>2.2</version>
				<configuration>
					<port>8080</port>
					<path>/</path>
				</configuration>
			</plugin>

		</plugins>
	</build>

</project>
```

src/main/java 아래에 carami.todo.config 패키지를 생성합니다.

```
package carami.todo.config;

import org.springframework.context.annotation.Configuration;

@Configuration
public class RootApplicationContextConfig {
}
```
```
public class WebInitializer implements WebApplicationInitializer {
    private static final String CONFIG_LOCATION = "carami.todo.config";
    private static final String MAPPING_URL = "/";

    public WebInitializer(){

    }

    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        WebApplicationContext context = getContext();


        // encoding filter 설정
        EnumSet<DispatcherType> dispatcherTypes = EnumSet.of(DispatcherType.REQUEST, DispatcherType.FORWARD);

        CharacterEncodingFilter characterEncodingFilter = new CharacterEncodingFilter();
        characterEncodingFilter.setEncoding("UTF-8");
        characterEncodingFilter.setForceEncoding(true);

        FilterRegistration.Dynamic characterEncoding = servletContext.addFilter("characterEncoding", characterEncodingFilter);
        characterEncoding.addMappingForUrlPatterns(dispatcherTypes, true, "/*");

        // dispatchder servlet 설정
        servletContext.addListener(new ContextLoaderListener(context));
        ServletRegistration.Dynamic dispatcher = servletContext.addServlet("DispatcherServlet", new DispatcherServlet(context));
        dispatcher.setLoadOnStartup(1);
        dispatcher.addMapping(MAPPING_URL);
    }
    private AnnotationConfigWebApplicationContext getContext() {
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
        context.setConfigLocation(CONFIG_LOCATION);
        return context;
    }
}
```
```
package carami.todo.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.view.InternalResourceViewResolver;
import org.springframework.web.servlet.view.JstlView;

@Configuration
@EnableWebMvc
@ComponentScan(basePackages = {"carami.todo.controller"})
public class ServletContextConfig extends WebMvcConfigurerAdapter {
    @Bean
    public ViewResolver viewResolver() {
         InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setViewClass(JstlView.class);
        viewResolver.setPrefix("/WEB-INF/views/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }
}
```

carami.todo.controller 패키지를 생성합니다.

```
package carami.todo.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

/**
 * Created by 강경미 on 2017. 6. 3..
 */
@Controller
public class HelloController {
    @GetMapping(path = "/")
    public String hello(Model model){
        return "hello";
    }

}
```

src/main/webapp/WEB-INF/views 폴더를 생성한다. 해당 폴더 안에 hello.jsp 를 작성한다.

```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>hello</title>
</head>
<body>
<h1>Hello World</h1>
</body>
</html>
```

터미널을 실행한 후 해당 프로젝트 폴더로 이동한다. maven path설정을 하기 전에 연 터미널이라면 mvn이 실행이 안될 수 있다.
이때는 터미널을 닫았다가 다시 연다.

mvn clean install<br>
mvn tomcat7:run

위와 같이 실행한 후 http://localhost:8080/ 을 브라우저로 연다. Hello가 출력되면 성공

아래는 mvn tomcat7:run을 했을때 나오는 화면. tomcat을 설치하지 않고 maven에 내장된 방식으로 실행할 수 있다.
어떤 값이 로그로 출력되었는지 살펴보도록 하자.
```
477 urstory:todo $ mvn tomcat7:run
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building todo 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] >>> tomcat7-maven-plugin:2.2:run (default-cli) > process-classes @ todo >>>
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ todo ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] Copying 0 resource
[INFO]
[INFO] --- maven-compiler-plugin:3.3:compile (default-compile) @ todo ---
[INFO] Nothing to compile - all classes are up to date
[INFO]
[INFO] <<< tomcat7-maven-plugin:2.2:run (default-cli) < process-classes @ todo <<<
[INFO]
[INFO] --- tomcat7-maven-plugin:2.2:run (default-cli) @ todo ---
[INFO] Running war on http://localhost:8080/
[INFO] Creating Tomcat server configuration at /DEVEL/eclipse_workspace/todo/target/tomcat
[INFO] create webapp with contextPath:
Jun 24, 2017 3:04:02 PM org.apache.coyote.AbstractProtocol init
정보: Initializing ProtocolHandler ["http-bio-8080"]
Jun 24, 2017 3:04:02 PM org.apache.catalina.core.StandardService startInternal
정보: Starting service Tomcat
Jun 24, 2017 3:04:02 PM org.apache.catalina.core.StandardEngine startInternal
정보: Starting Servlet Engine: Apache Tomcat/7.0.47
Jun 24, 2017 3:04:04 PM org.apache.catalina.core.ApplicationContext log
정보: No Spring WebApplicationInitializer types detected on classpath
Jun 24, 2017 3:04:04 PM org.apache.catalina.core.ApplicationContext log
정보: Initializing Spring root WebApplicationContext
Jun 24, 2017 3:04:04 PM org.springframework.web.context.ContextLoader initWebApplicationContext
정보: Root WebApplicationContext: initialization started
Jun 24, 2017 3:04:04 PM org.springframework.web.context.support.AnnotationConfigWebApplicationContext prepareRefresh
정보: Refreshing Root WebApplicationContext: startup date [Sat Jun 24 15:04:04 KST 2017]; root of context hierarchy
Jun 24, 2017 3:04:04 PM org.springframework.web.context.support.AnnotationConfigWebApplicationContext loadBeanDefinitions
정보: Successfully resolved class for [carami.todo.config.RootApplicationContextConfig]
Jun 24, 2017 3:04:04 PM org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor <init>
정보: JSR-330 'javax.inject.Inject' annotation found and supported for autowiring
Jun 24, 2017 3:04:04 PM org.springframework.web.context.ContextLoader initWebApplicationContext
정보: Root WebApplicationContext: initialization completed in 376 ms
Jun 24, 2017 3:04:04 PM org.apache.catalina.core.ApplicationContext log
정보: Initializing Spring FrameworkServlet 'dispatcher'
Jun 24, 2017 3:04:04 PM org.springframework.web.servlet.DispatcherServlet initServletBean
정보: FrameworkServlet 'dispatcher': initialization started
Jun 24, 2017 3:04:04 PM org.springframework.web.context.support.AnnotationConfigWebApplicationContext prepareRefresh
정보: Refreshing WebApplicationContext for namespace 'dispatcher-servlet': startup date [Sat Jun 24 15:04:04 KST 2017]; parent: Root WebApplicationContext
Jun 24, 2017 3:04:04 PM org.springframework.web.context.support.AnnotationConfigWebApplicationContext loadBeanDefinitions
정보: Successfully resolved class for [carami.todo.config.ServletContextConfig]
Jun 24, 2017 3:04:04 PM org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor <init>
정보: JSR-330 'javax.inject.Inject' annotation found and supported for autowiring
Jun 24, 2017 3:04:05 PM org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping register
정보: Mapped "{[/],methods=[GET]}" onto public java.lang.String carami.todo.controller.HelloController.hello(org.springframework.ui.Model)
Jun 24, 2017 3:04:05 PM org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter initControllerAdviceCache
정보: Looking for @ControllerAdvice: WebApplicationContext for namespace 'dispatcher-servlet': startup date [Sat Jun 24 15:04:04 KST 2017]; parent: Root WebApplicationContext
Jun 24, 2017 3:04:05 PM org.springframework.web.servlet.DispatcherServlet initServletBean
정보: FrameworkServlet 'dispatcher': initialization completed in 673 ms
Jun 24, 2017 3:04:05 PM org.apache.coyote.AbstractProtocol start
정보: Starting ProtocolHandler ["http-bio-8080"]

```


## XML 파일로 작성할 경우 예


web.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" id="WebApp_ID" version="2.5">
  <display-name>springMVCExam</display-name>
 <!-- WAS가 열릴 때, 여러가지 설정들을 저장함 context param에 -->
 <context-param>
 	<param-name>contextConfigLocation</param-name>
 	<param-value>classpath:/applicationContext.xml</param-value>
 </context-param>
  <!-- 어떤 특정한 이벤트가 일어날 때 실행되게 하는 것을 의미함, 세션에 어트리뷰트가 추가될 때, 리퀘스트에 어떤게 지워지게 하는 등의 방법임, ContextListener는 WAS가 시작되거나 종료될 때 수행됨, LoaderListener는 WAS가 시작될 때, 리스너가 등록되어 있을 경우 context-param을 불러오도록 약속함 -->
  <listener>
  	<listener-class>
  		org.springframework.web.context.ContextLoaderListener
  	</listener-class>
  </listener>
  <!-- 필터는 요청이 URL-pattern, 응답이 나가기 전에 수행하는 것임 -->
  <filter>
		<filter-name>encodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>encodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>

  <servlet>
  	<servlet-name>action</servlet-name>
  	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  	<init-param>
  		<param-name>contextConfigLocation</param-name>
  		<param-value>WEB-INF/config/spring-mvc-config.xml</param-value>
  	</init-param>
  </servlet>
  <!-- 이런 Url이 들어올 때만 이 서블릿이 실행한다는 의미, 정적인 페이지를 빼기 위한 설정을 따로 함 -->
  <servlet-mapping>
  	<servlet-name>action</servlet-name>
  	<url-pattern>*.sku</url-pattern>
  </servlet-mapping>



  <welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.htm</welcome-file>
    <welcome-file>index.jsp</welcome-file>
    <welcome-file>default.html</welcome-file>
    <welcome-file>default.htm</welcome-file>
    <welcome-file>default.jsp</welcome-file>
  </welcome-file-list>
</web-app>
```

spring-mvc-config.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
	http://www.springframework.org/schema/context
	http://www.springframework.org/schema/context/spring-context-3.0.xsd">
    <!-- 컨트롤러가 무엇인지 알아야함, 핸들러 맵핑이 알아야함, 컨트롤러를 각각을 다 안찾고 베이스 패키지를 등록하게끔 함, 컨트롤러에 어노테이션이 붙어있음 -->
	<context:component-scan base-package="kr.ac.skuniv.controller" />

	<bean id="viewResolver"
      class="org.springframework.web.servlet.view.UrlBasedViewResolver">
   	 <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
   	 <property name="prefix" value="/jsp/"/>
   	 <property name="suffix" value=".jsp"/>

</bean>

</beans>
```

[필터 참고자료](http://wiki.gurubee.net/pages/viewpage.action?pageId=26740229)
