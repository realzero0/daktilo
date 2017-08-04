---
layout: post
title:  "[Boost_Camp] WAS 만들기"
subtitle: "5주차"
date:   2017-08-04 10:40:00
categories: [study]
---

# Http 프로토콜 #

네트워크 프로그래밍


Client<-------------->Server
<br>접속, 요청, 응답(네트워크 프로그래밍)

Sever는 여러개의 요청을 받아야 하기 때문에, Server는 Thread로 구성되어야 한다.

접속, 요청, 응답 과정에서 IO가 발생하기 때문에 IO도 해야한다.

Http 프로토콜에서는 대개 클라이언트가 웹브라우저이다.

## 1 Server ##

- 서버 소켓을 하나 생성

- 기다림(Accept 메소드로 요청이 들어올 때까지 기다림

## 2 Client ##

- 소켓을 하나 열고

- 접속을 한다.

- 서버에 접속하면서 소켓에 통로가 하나가 생긴다. 클라이언트의 input 부분 소켓과 서버의 output부분 소켓이 동일하다고 볼 수 있다.


연결이 모두 끝날 때, close로 연결을 종료한다.

IO(input, output)

IO는 주인공 클래스와 장식 클래스가 존재한다. 주인공(목적지, file, console 등)

BufferedReader는 목적지가 없는 장식 클래스이다. 생성자에 반드시 주인공 클래스를 넣어줘야한다. 

장식 클래스는 기능이 가장 중요하다. BufferedReader는 한 줄씩 읽기 위해서 사용(4kb단위 등)

스트림은 데이터가 흘러가는 것을 추상화 시킨 것을 의미한다. 스트림은 단방향이고, IO는 입력과 출력이 동시에 진행되지 않고, input과 output은 각각의 통로를 열어야 한다. 동시에 사용할 수 있는 것은 내부적으로 각각의 통로를 열어준다.

수도꼭지와 샤워꼭지를 나누어서 생각해보면, 수도꼭지가 없으면 샤워꼭지만 가지고 사용할 수가 없는 것과 마찬가지로, 항상 주인공 클래스는 반드시 필요하다. 자기가 원하는 샤워꼭지로 나눠서 쓰면 된다. 샤워꼭지가 수도꼭지랑 안맞으면 중간에 안맞는 부분을 고칠 수 있는 것들이 필요하다. InputStreamReader가 Stream을 Reader로 바꿔 줄 수 있는 것과 같은 이치이다.

## 간단한 Echo Server, Echo Client 만들기

- java.io, java.net, Exception 처리 등을 할 수 있어야 개발 가능합니다.
- 해당 Echo Server는 동시에 여러개의 클라이언트가 접속할 수 없습니다.

![EchoServer를 실행](001.png)

![EchoClient를 실행](002.png)

키보드로 hello [enter] world [enter] ^d 
를 차례로 누릅니다. ^d 는 EoF(End of File)을 의미합니다.

![EchoServer실행 화면](003.png)



```
package kr.or.connect.echo0;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.io.PrintWriter;
import java.net.Socket;
import java.net.ServerSocket;

public class EchoServer {

    public static void main(String[] args) throws Exception {

        int port = 4444;
        ServerSocket serverSocket = new ServerSocket(port);
        System.err.println("Started server on port " + port);

        Socket clientSocket = serverSocket.accept();
        System.err.println("Accepted connection from client");

        BufferedReader br = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
        PrintWriter pw = new PrintWriter(new OutputStreamWriter(clientSocket.getOutputStream()));

        String line = null;

        while((line = br.readLine()) != null){ // 클라이언트로부터 한줄을 읽어들인다.
            System.out.println("echo : " + line);
            pw.println("echo : " + line); // 클라이언트에게 한줄을 출력한다.
            pw.flush();
        }
        clientSocket.close();
        serverSocket.close();
    }
}

```

Server는 클라이언트의 접속을 대기해야 합니다.

- ServerSocket객체를 생성 후 accept()메소드를 호출하게 되면 클라이언트의 접속을 4444번 포트에서 기다리게 됩니다.
- accept()메소드는 클라이언트가 접속할 때까지 멈춰있는다고 해서 블럭킹 메소드라고 말합니다.
- 클라이언트가 접속할 경우 해당 클라이언트와 통신을 할 수 있는 Socket객체가 반환됩니다.
- 클라이언트와 통신하기 위해서 Socket으로부터 InputStream과 OutputStream을 얻었습니다.
- 한줄쓰고, 한줄 읽어들이기 위해서 BufferedReader, PrintWriter를 사용하였습니다.
- 클라이언트에게 정보를 출력할때는 출력후에 flush()메소드르 호출해야합니다.
- socket을 다 사용하고 나면 close()를 호출해야합니다. Exception처리를 잘 할 필요가 있습니다.


```
package kr.or.connect.echo0;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.io.PrintWriter;
import java.net.Socket;

public class EchoClient {
    public static void main(String args[]) throws Exception{
        Socket server = new Socket("127.0.0.1", 4444);
        BufferedReader keyboard = new BufferedReader(new InputStreamReader(System.in));
        BufferedReader br = new BufferedReader(new InputStreamReader(server.getInputStream()));
        PrintWriter pw = new PrintWriter(new OutputStreamWriter(server.getOutputStream()));

        String line = null;
        String readLine = null;
        while((line = keyboard.readLine()) != null){ // 키보드로부터 한줄 읽어들임
            pw.println(line); // server에게 한줄 보냄
            pw.flush();
            readLine = br.readLine(); // server가 보내는 한줄을 읽어들임
            System.out.println(readLine); // server가 보내는 한줄을 출력함
        }
        server.close();
    }
}

```

- 키보드로부터 한줄씩 입력받기 위한 객체를 생성하여 keyboard변수에 할당하였습니다.
- server에 접속하기 위해 Socket객체를 생성합니다. server의 ip와 포트를 생성자에 넣었습니다.
- 해당 socket이 오류가 나지 않고 잘 생성되었다면 server와 접속이 잘 되었다는 것을 의미합니다. server의 경우 이 때 accept()메소드가 클라이언트와 통신할 수 있는 Socket객체를 반환합니다.
- 서버에 정보를 전달하고 읽어오기 위한 객체를 생성하여 변수 br, pw에 할당하였습니다.
- keyboard로부터 읽어들인 정보를 서버에 보내고, 바로 서버가 보내준 값을 읽어들여 화면에 출력합니다.



# Http 요청

- 참고 문서 : https://developer.mozilla.org/ko/docs/Web/HTTP/Overview

## GET방식 요청

![request](/assets/img/http.png)

- 첫번째 줄은 method path 프로토콜 버전이 나온다.
- 두번째 줄부터 빈줄이 나올때까지 Header정보가 나온다.
- 빈줄이 하나 나온다.
- post방식일 경우에는 빈 줄 이후에 body정보가 전달된다.  


```
package kr.or.connect.httprequest0;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;

public class WebServer {
    public static void main(String[] args) throws Exception {

        int port = 8080;
        ServerSocket serverSocket = new ServerSocket(port);
        System.err.println("Started server on port " + port);

        Socket clientSocket = serverSocket.accept();
        System.err.println("Accepted connection from client");

        BufferedReader br = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));

        String line = null;

        while((line = br.readLine()) != null){ // 클라이언트로부터 한줄을 읽어들인다.
            System.out.println("readLine : " + line);
            if("".equals(line))
                break;
        }
        clientSocket.close();
        serverSocket.close();
    }
}

```


- 위의 서버를 실행하고 브라우저에서 http://127.0.0.1:8080 을 실행한다.

```
Started server on port 8080
Accepted connection from client
readLine : GET / HTTP/1.1
readLine : Host: 127.0.0.1:8080
readLine : Connection: keep-alive
readLine : Cache-Control: max-age=0
readLine : Upgrade-Insecure-Requests: 1
readLine : User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.78 Safari/537.36
readLine : Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
readLine : Accept-Encoding: gzip, deflate, br
readLine : Accept-Language: ko,en-US;q=0.8,en;q=0.6
readLine : 
```

결과가 위와 같이 출력되는 것을 알 수 있다. / 에 대한 페이지를 요청하는 것을 알 수 있다.

---

# 멀티 쓰레드 Http 서버 #

