---
layout: post
title:  "[Boost_Camp] Spring MVC2"
subtitle: "시작 전 준비단계"
date:   2017-07-06 12:40:00
categories: [study]
---



ServletContextConfig.java 파일에 추가  - 리소스에 대한 설정 추가
```
@Override
   public void addResourceHandlers(ResourceHandlerRegistry registry) {
       registry.addResourceHandler("/resources/**").addResourceLocations("/resources/");  //   webapp/resources 경로를 의미
   }
```

MemberService  

```
package carami.todo.service;

import carami.todo.domain.Member;

public interface MemberService {
    public Member get(Long id);
    public Member addMember(Member member);
    public int delete(Long id);
    public int update(Member member);
    public Member getByEmail(String email);
}

```

MemberServieceImpl

```
package carami.todo.service.impl;

import carami.todo.dao.MemberDao;
import carami.todo.domain.Member;
import carami.todo.service.MemberService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;


@Service
public class MemberServiceImpl implements MemberService {
    @Autowired
    MemberDao memberDao;

    @Override
    @Transactional(readOnly = true)
    public Member get(Long id) {
        return memberDao.selectById(id);
    }

    @Override
    @Transactional(readOnly = false)
    public Member addMember(Member member){
        Long insert = memberDao.insert(member);
        member.setId(insert);
        return member;
    }

    @Override
    public int delete(Long id) {
        return memberDao.delete(id);
    }

    @Override
    public int update(Member member) {
        return memberDao.update(member);
    }

    @Override
    public Member getByEmail(String email) {
        return memberDao.selectByEmail(email);
    }
```
MainController
```
package carami.todo.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/")
public class MainController {
    @GetMapping
    public String index(){
        return "index";
    }
}

```

MemberFormParam
```
package carami.todo.dto;

public class MemberFormParam {
    private String name;
    private String email;
    private String passwd1;
    private String passwd2;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getPasswd1() {
        return passwd1;
    }

    public void setPasswd1(String passwd1) {
        this.passwd1 = passwd1;
    }

    public String getPasswd2() {
        return passwd2;
    }

    public void setPasswd2(String passwd2) {
        this.passwd2 = passwd2;
    }
}

```
MemberParam
```
package carami.todo.dto;

public class MemberParam {
    private String name;
    private String email;
    private String passwd;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getPasswd() {
        return passwd;
    }

    public void setPasswd(String passwd) {
        this.passwd = passwd;
    }
}

```
```
package carami.todo.controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import carami.todo.domain.Member;
import carami.todo.dto.MemberFormParam;
import carami.todo.service.MemberService;

@Controller
@RequestMapping("/members")
public class MemberController {

    @Autowired
    private MemberService memberService;


    // 회원 가입시 form에서 알맞은 값을 입력하였는지 프론트 javascript를 이용하여 검증을 해야겠지만,
    // 프론트를 무시하고 요청을 보낼 수도 있기 때문에 서버에서도 해당 값이 올바른지 검증하는 코드가 반드시 존재해야 한다.
    @PostMapping
    public String create(@ModelAttribute MemberFormParam memberFormParam) {
        if (memberFormParam.getName() == null || memberFormParam.getName().length() == 0 ||
                memberFormParam.getEmail() == null || memberFormParam.getEmail().length() == 0 ||
                memberFormParam.getPasswd1() == null || memberFormParam.getPasswd1().length() == 0 ||
                memberFormParam.getPasswd2() == null || memberFormParam.getPasswd2().length() == 0) {
            return "redirect:/"; // 이름, email, passwd1, passwd2 중에서 하나라도 입력하지 않은 것이 있을 경우 이동, 새로운 요청이 생긴다
        }else {
            if (memberFormParam.getPasswd1().equals(memberFormParam.getPasswd2())) { //사용자가 입력한 암호가 같을 경우
                Member member = new Member();
                member.setName(memberFormParam.getName());
                member.setEmail(memberFormParam.getEmail());
                member.setPasswd(memberFormParam.getPasswd1());
                Member resultMember = memberService.addMember(member);

                return "redirect:/"; // 회원 가입 후 봐야할 화면으로 redirect
            } else {
                return "redirect:/"; // 암호, 암호 가 일치하지 않음
            }
        }
    } // 회원 가입 끝

    // email을 로그인 id로 사용하려면 table 생성에서도 해당 값은 unique하게 설정되어 있어야 한다.
    // 그리고 null을 허용하면 안된다. email 칼럼에 대하여 unique, not null제약 조건을 추가한다.
    @PostMapping("/login")
    public String login(@RequestParam(name="email") String email, @RequestParam(name="passwd") String passwd, HttpServletRequest request){
        Member member = memberService.getByEmail(email);
        if(member != null && passwd.equals(member.getPasswd())){
            HttpSession session = request.getSession();
            session.setAttribute("loginInfo", member);
            return "redirect:/";
        }else{ // member가 없거나 passwd가 일치하지 않는다.
            return "redirect:/";
        }
    }

    @GetMapping("/logout")
    public String login(HttpServletRequest request){
        HttpSession session = request.getSession();
        session.removeAttribute("loginInfo");
        return "redirect:/";
    }

}

```

index.jsp
```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>

<html>
<head>
    <title>main page</title>
</head>
<body>
<h1>main page</h1>
<hr>

<img src="/resources/image.png"/>
<br><br>

로그인 폼<br>
<c:if test="${sessionScope.loginInfo == null}">



<form method="post" action="/members/login">
    email : <input type="text" name="email"><br>
    암호 : <input type="password" name="passwd"><br>
    <input type="submit" value="확인">

</form>
</c:if>

<c:if test="${sessionScope.loginInfo != null}">
    "${sessionScope.loginInfo.name}" 님 환영합니다. <a href="/members/logout">로그아웃</a>
</c:if>
<br><br>


회원 가입 폼<br>
<form method="post" action="/members">
    이름 : <input type="text" name="name"><br>
    email : <input type="text" name="email"><br>
    암호 : <input type="password" name="passwd1"><br>
    암호 확인 : <input type="password" name="passwd2"><br>
    <input type="submit" value="확인">

</form>
</body>
</html>

```
