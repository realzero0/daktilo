---
layout: post
title:  "Apache Camel"
subtitle: "정의 및 사용법"
date:   2018-01-18 15:40:00
categories: [study]
---

# Apache Camel

- 여러개의 서비스와의 연결이 필요할 때, 사용하기 좋다.

- camel-config.xml에 카멜의 route정보를 저장하고, from - 어디에서, to - 어디로 보낼 것인지 결정한다.

- 여기서 uri로 처리되며, uri 인터페이스를 사용하는 모든 다른 어플리케이션, 서버와 연결할 수 있게 된다.

- 또한 다양한 확장 어플리케이션을 사용해야한다면, 사용자는 camel에게만 모든 job을 전가하고 camel에서만 다양한 확장 프로그램과 연결해 사용할 수 있다는 장점이 있다.


## Route
루트는 어디에서 어디로 갈지를 정의한 시스템간의 인터페이스이다. 1 on 1 or 1 on N의 관계로도 정의할 수 있다.

루트는 크게 Component와 Processor로 정의된다.

## End point
```
myCamelContext.getEndpoint("pop3://john.smith@mailserv.example.com?password=myPassword");
```

## Component
route의 from to에서 각각의 받는 곳과 주는 곳의 uri를 end point라고 정의한다. 각각의 uri를 카멜과 연결할 때, 각각의 프로토콜에 맞는 컴포넌트의 인스턴스를 생성 해야한다. 예를 들어, FTP라면 FTP 컴포넌트, JDBC라면 JDBC 컴포넌트, ActiveMQ라면 JMS프로토콜을 정의해 주어야 한다. 컴포넌트는 연결된 uri를 각각의 프로토콜에 맞춰 데이터를 전송할 수 있게 한다.
Each invocation of JmsComponent.createEndpoint() creates an instance of the JmsEndpoint class

```
Component mailComponent = new org.apache.camel.component.mail.MailComponent();myCamelContext.addComponent("pop3", mailComponent);myCamelContext.addComponent("imap", mailComponent);myCamelContext.addComponent("smtp", mailComponent);
```



## Processor
from에서 받은 데이터를 to로 보낼 때, 거의 대부분 로그를 남기는 등의 로직이 필요하다. 일부 비즈니스 로직을 실행하기 위해서 프로세서를 사용한다.
	1. message format transformation - 메세지의 포맷을 변경(Json to XML, XML to Json)
	2. Routing - 

there are many classes in the Camel library that implement the Processor interface in a way that provides support for a design pattern in the EIP book. For example, ChoiceProcessor implements the message router pattern, that is, it uses a cascading if-then-else statement to route a message from an input queue to one of several output queues. Another example is the FilterProcessor class which discards messages that do not satisfy a stated predicate (that is, condition).