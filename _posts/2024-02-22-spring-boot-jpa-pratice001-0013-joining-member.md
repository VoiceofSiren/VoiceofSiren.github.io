---
layout: post
title: "JPA Practice001: #0013 - Joining Member"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. 웹 계층 개발 목차

- 회원 등록
- 회원 목록 조회 (다음 글에서)
- 상품 등록 (다음 글에서)
- 상품 목록 (다음 글에서)
- 상품 수정 (다음 글에서)
- 변경 감지와 병합 (merge) (다음 글에서)
- 상품 주문 (다음 글에서)
- 주문 목록 검색, 취소 (다음 글에서)

## 2. 회원 등록

Form 객체를 사용하여 화면 계층과 서비스 계층을 명확하게 분리하고자 한다.

##### **1) 회원 Form 객체**

```java
package jpabook.jpashop.controller;

import jakarta.validation.constraints.NotEmpty;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class MemberForm {

    @NotEmpty(message = "회원 이름은 필수입니다.")
    private String name;
    
    private String city;
    private String street;
    private String zipcode;
}
```

##### **2) 회원 Controller**

@Valid를 통해 MemberForm의 name 필드에 걸려있는 유효성을 검증하게 하였다.

BindingResult를 사용하여 유효성에 맞지 않을 경우에 대한 예외 처리를 하였다.

```java
package jpabook.jpashop.controller;

import jakarta.validation.Valid;
import jpabook.jpashop.domain.Address;
import jpabook.jpashop.domain.Member;
import jpabook.jpashop.service.MemberService;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;

@Controller
@RequiredArgsConstructor
public class MemberController {

    private final MemberService memberService;

    @GetMapping("/members/new")
    public String createForm(Model model) {
        model.addAttribute("memberForm", new MemberForm());
        return "members/createMemberForm";
    }

    @PostMapping("/members/new")
    public String create(@Valid MemberForm form, BindingResult result) {

        if (result.hasErrors()) {
            return "members/createMemberForm";
        }

        Address address = new Address(form.getCity(), form.getStreet(), form.getZipcode());

        Member member = new Member();
        member.setName(form.getName());
        member.setAddress(address);

        memberService.join(member);

        return "redirect:/";
    }
}
```

##### **3) 회원 등록 폼 html**

templates/members/createMemberForm.html

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header" />
<style>
    .fieldError {
        border-color: #bd2130;
    }
</style>
<body>
<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader"/>
    <form role="form" action="/members/new" th:object="${memberForm}" method="post">
        <div class="form-group">
            <label th:for="name">이름</label>
            <input type="text" th:field="*{name}" class="form-control" placeholder="이름을 입력하세요"
                   th:class="${#fields.hasErrors('name')}? 'form-control fieldError' : 'form-control'">
            <p th:if="${#fields.hasErrors('name')}"
               th:errors="*{name}">Incorrect date</p>
        </div>
        <div class="form-group">
            <label th:for="city">도시</label>
            <input type="text" th:field="*{city}" class="form-control"
                   placeholder="도시를 입력하세요">
        </div>
        <div class="form-group">
            <label th:for="street">거리</label>
            <input type="text" th:field="*{street}" class="form-control"
                   placeholder="거리를 입력하세요">
        </div>
        <div class="form-group">
            <label th:for="zipcode">우편번호</label>
            <input type="text" th:field="*{zipcode}" class="form-control"
                   placeholder="우편번호를 입력하세요">
        </div>
        <button type="submit" class="btn btn-primary">Submit</button>
    </form>
    <br/>
    <div th:replace="fragments/footer :: footer" />
</div> <!-- /container -->
</body>
</html>
```

##### **4) 실행 시 화면**

- 회원 등록 시 실행 화면

![IMAGE](/assets/images/spring-boot-jpa-practice001/0013/members-createMemberForm-html.png)

- 회원 등록 시 log 기록

![IMAGE](/assets/images/spring-boot-jpa-practice001/0013/logs-of-joining-member.png)

- h2 DBMS

![IMAGE](/assets/images/spring-boot-jpa-practice001/0013/h2-select-member.png)

- 유효성 검증 전

![IMAGE](/assets/images/spring-boot-jpa-practice001/0013/before-validation.png)

- 유효성 검증 후

![IMAGE](/assets/images/spring-boot-jpa-practice001/0013/after-validation.png)