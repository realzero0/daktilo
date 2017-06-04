---
layout: post
title:  "[Boost_Camp] Framework, IOC, DI"
subtitle: "시작 전 준비단계"
date:   2017-07-05 19:40:00
categories: [study]
---



## 2. Servlet/JSP & Spring & Spring-MVC & Srping JDBC학습

- Servlet/JSP에 대하여 이해하고 이를 이용해 개발 할 수 있다.


- Spring에 대하여 이해하고 이를 이용해 개발 할 수 있다.


# Framework vs Library

Framework는 반제품과 같다고 생각하면 됨

뼈대가 있는 상태에서 집을 짓는다고 생각하면 됨

Framework는 맞춤구두 같은 애는 아니지만,

서블릿 또한 하나의 프레임워크. 확장할 수 있는 포인트가 필요함.

- 확장할 수 있는 포인트를 HookPoint라고 함

- 프레임웍의 동작, 어떤 환경, 여러 제반 정보를 프레임워크에 알려줘야함

- EJB는 분산처리를 위한 것(여러 대의 컴퓨터를 하나인 것처럼 동작시킴), EJB가 종속성이 매우 높은 반면, 스프링은 모듈화가 잘 되어 있어서 필요한 것만 가져다 쓰면 됨

- 프레임워크는 확장포인트가 있음, Servlet의 경우에는 HttpServlet, Spring의 경우에는 XML파일 등이 있음

<br>


# IoC

제어의 역전

IoC는 개발자가 빈 객체를 생성하지 않고 프레임워크가 관리하는 방식이다.

팩토리 메소드 패턴에서 우리가 팩토리를 생성해야 했다면 프레임워크는 그 팩토리를 생성하지 않고 선언만으로도 빈 객체를 사용할 수 있게 해준다.

여기에는 DL과 DI방식이 있으며,

DL은 Dependency Look-up이라는 것이며 빈 객체를 검색해서 의존성을 주입하는 방식인데, 현재는 잘 사용되지는 않는다.

DI는 생성자, Setter, 어노테이션을 이용해서 의존성 주입을 하며 빈 객체를 XML을 통해 알려주거나, 자바 객체를 이용해 알려주기도 한다.


## Bean factory & ApplicationContext

- Bean factory는 공장, 객체가 쓰여질 때 객체를 만듬

- Apllication Context는 조금 더 큰 공장, 객체가 생성되는 시점의 차이가 남, 쓰여질 객체들을 만들 것임, 기본적인 디자인 패턴이 싱글톤패턴이기 때문에 크게 메모리에 영향을 미치지 않고, 기능이 더 많기 때문에 Application Context를 주로 사용

## Bean Factory

- 단순 컨테이너, 객체 생성 및 DI 처리

## [DL vs DI 참고자료](http://dev.anyframejava.org/docs/anyframe/4.0.0/reference/html/ch06.html)

DI의 Injection의 의미는 setter나 생성자에 ref로 다른 Bean의 의존성을 주입하고, 그런 의존관계를 Spring Framework이 자동으로 설정해준다는 의미이고 이로인해서 의존성을 굉장히 낮추게 됨

## ApplicationContext

- Bean Factory 기능 + α
- Annotation
- 스프링 설정
- 트렌젝션 관리
- 메시지 처리등등의 다양한 부가 가능


## ApplicationContext를 이용하여 String문자열 값 읽어오기

아래 방식은 Look-up방식(getBean을 이용, DL방식)<br>
1 xml 설정파일에 직접 등록

- /src/main/resources/applicationContext.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="name" class="java.lang.String">
        <constructor-arg value="carami"/> <!-- 생성자를 통해 주입받겠다는 것 -->
    </bean>
</beans>
```

- /src/main/java
에 carami.spring.core.examples 패키지 아래의 SpringApplication.java

```
package carami.spring.core.examples;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class SpringApplication {

    private static final String CONTEXT_PATH = "applicationContext.xml";

    public static void main(String args[]) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(CONTEXT_PATH);
// ApplicationContext로만 선언해도 됨
        String name = (String)context.getBean("name");
	// Look-up해오는 방식
        System.out.println("name : " + name);

        context.close();
    }
}

```

2 자바코드로 빈 등록하기
carami.spring.core.config 패키지 아래의 ApplicationContextConfig

java config를 이용하여 설정.

@Configuration 어노테이션이 붙어있으면 java config로 인식한다.
@ComponentScan 어노테이션을 이용하여 탐색할 package를 찾는다.

```
package carami.spring.core.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration // 설정이야, Xml을 대신한다는 의미
@ComponentScan(basePackages = "carami.spring.core") // 저 패키지 내에 있는 것들을 읽어와
public class ApplicationContextConfig {
    @Bean // 빈을 등록해라
    public String name() {
        return "carami";
    }

}

```

/src/main/java
에 carami.spring.core.examples 패키지 아래의 SpringApplication.java

```
package carami.spring.core.examples;

import carami.spring.core.config.ApplicationContextConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class SpringApplication {

    private static final String CONTEXT_PATH = "applicationContext.xml";

    public static void main(String args[]) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ApplicationContextConfig.class);
        String name = (String)context.getBean("name"); // look-up하는 방식임(DL) - API에 의해 만들어진 빈을 사용하기 때문에 Lookup

        System.out.println("name : " + name);

        context.close();
    }
}

```



```
    @Bean
    public String name() {
        return "carami";
    }
```
ApplicationContenxt는 필요한 모든 객체를 미리 생성해 놓는다.
@Bean이 붙어있는 메소드 이름이 name()이면 id값이 name으로 객체를 생성한다는 것을 의미한다. 해당 객체는 리턴하는 String객체이다.

---

아래 방식은 Bean을 등록하는 방법임
이를 DI(Dependency Injection이라고 함)
Injection 방식에는 Setter, 생성자 방식이 있고,
이는 각각 XML과 자바 객체를 이용하는 방식이 있음

## Setter Injection


1 xml 설정파일에 직접 등록

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="name" class="java.lang.String">
        <constructor-arg value="carami"/>
    </bean>

    <bean id="user1" class="carami.spring.core.examples.User">
        <property name="name" ref="name"/>
    </bean>

    <bean id="user2" class="carami.spring.core.examples.User">
        <property name="name" value="홍길동"/>
    </bean>
</beans>
```


user1의 경우 id="name"으로 설정된 정보의 레퍼런스로 값을 주입받고
user2의 경우 "홍길동" 값(value)을 직접 주입받았다.

```
// Bean클래스
public class User {

    // private하게 필드를 선언
    private String name;

    // 기본 생성자가 있어야 한다
    public User(){

    }

    // private한 필드를 접근하기 위한 seter, getter메소드가 있어야 한다.
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
```
package carami.spring.core.examples;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class SpringApplication {

    private static final String CONTEXT_PATH = "applicationContext.xml";

    public static void main(String args[]) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(CONTEXT_PATH);
        User user1 = (User)context.getBean("user1");
        System.out.println(user1.getName());

        User user2 = (User)context.getBean("user2");
        System.out.println(user2.getName());

        context.close();
    }
}

```

2 자바코드로 빈 등록하기

```
package carami.spring.core.config;

import carami.spring.core.examples.User;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = "carami.spring.core")
public class ApplicationContextConfig {
  @Bean
  public String name() {
      return "carami";
  }

  @Bean
  public User user1(String name) {
      User user = new User();
      user.setName(name);
      return user;
  }

  @Bean
  public User user2() {
      User user = new User();
      user.setName("홍길동");
      return user;
  }
}

```


id가 user1(메소드 이름) 으로 Bean을 등록한다. 이때 user1메소드의 파라미터로 name이 있는데, 해당 값은 위의 name()메소드가 반환한 값이 자동으로 들어간다.
즉 id가 name인 값을 자동으로 받는다는 것을 의미한다. 해당 빈의 값은 리턴하는 User객체를 의미한다.

id가 user2(메소드 이름) 으로 Bean을 등록한다. 해당 빈의 값은 리턴하는 User객체를 의미한다.



```
package carami.spring.core.examples;

import carami.spring.core.config.ApplicationContextConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class SpringApplication {

  private static final String CONTEXT_PATH = "applicationContext.xml";

  public static void main(String args[]) {
      AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ApplicationContextConfig.class);
      User user1 = (User)context.getBean("user1");
      System.out.println(user1.getName());

      User user2 = (User)context.getBean("user2");
      System.out.println(user2.getName());
      context.close();
  }
}


id가 user1과 user2인 객체를 요청한다. 모두 User객체이며 해당 객체의 getName()값을 호출하여 값을 출력하면 아래와 같이 출력되는 것을 확인할 수 있다.

carami
홍길동


## Constructor Injection
```
    <bean id="user3" class="carami.spring.core.examples.User">
        <constructor-arg index="0" ref="name"/>
    </bean>

    <bean id="user4" class="carami.spring.core.examples.User">
        <constructor-arg index="0" value="홍길동"/>
    </bean>
```

constructor-arg 는 생성자를 이용하여 주입하라는 의미다. index="0"은 첫번째 파라미터에 name레퍼런스의 값을 설정하라는 것을 의미한다.


```


    @Bean
    public User user3(String name) {
        User user = new User(name);
        return user;
    }

    @Bean
    public User user4() {
        User user = new User("홍길동");
        return user;
    }
```

id가 user3으로 등록된 Bean은 User객체로 설정된다. 해당 User객체는 id가 name인 값을 받아들여 생성자에 값을 설정하여 반환하는 것을 알 수 있다.

---


DI는 생성자, Setter를 이용해서 의존성 주입을 할 수 있다.

이를 위한 방식에는 XML, 어노테이션의 두가지 방식이 있을 수 있다.

최근에는 자바 객체를 이용해 어노테이션으로 빈을 등록하는 경우가 많다.

Component scan은 약속된 어노테이션을 찾는 역할을 함. @Configuration은 컨피그 객체라는 것을 알려줌


---

## JUnit을 이용한 Test

JUnit을 사용하기 위해서 pom.xml에 추가.
```
          <!-- spring test -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring-framework.version}</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>RELEASE</version>
        </dependency>
    </dependencies>
```

test코드 추가

```
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import carami.spring.core.config.ApplicationContextConfig;
import carami.spring.core.examples.User;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = ApplicationContextConfig.class)
public class SpringTest {
	@Autowired
	User user1;

	@Test
	public void testUser(){
		System.out.println(user1.getName());
	}
}

```
