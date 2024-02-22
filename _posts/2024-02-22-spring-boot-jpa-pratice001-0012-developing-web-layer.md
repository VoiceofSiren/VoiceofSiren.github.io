---
layout: post
title: "JPA Practice001: #0012 - Developing Web Layer"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. 웹 계층 개발 목차

- 홈 화면과 레이아웃
- 회원 등록 (다음 글에서)
- 회원 목록 조회 (다음 글에서)
- 상품 등록 (다음 글에서)
- 상품 목록 (다음 글에서)
- 상품 수정 (다음 글에서)
- 변경 감지와 병합 (merge) (다음 글에서)
- 상품 주문 (다음 글에서)
- 주문 목록 검색, 취소 (다음 글에서)

## 2. 홈 화면과 레이아웃

HomeController를 등록한다.

##### **1) controller.HomeController**
```java
package jpabook.jpashop.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@Slf4j
public class HomeController {

    @RequestMapping("/")
    public String home() {
        log.info("home controller");
        return "home";
    }
}
```

##### **2) application.yml**

Thymeleaf 템플릿 엔진을 사용하기 위해 기존의 application.yml 파일에 해당 설정을 추가한다.

```yaml
spring:
  thymeleaf:
      prefix: classpath:/templates/
      suffix: .html
```

##### **3) Thymeleaf templates 등록**

home.html

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header">
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader" />
    <div class="jumbotron">
        <h1>HELLO SHOP</h1>
        <p class="lead">회원 기능</p>
        <p>
            <a class="btn btn-lg btn-secondary" href="/members/new">회원 가입</a>
            <a class="btn btn-lg btn-secondary" href="/members">회원 목록</a>
        </p>
        <p class="lead">상품 기능</p>
        <p>
            <a class="btn btn-lg btn-dark" href="/items/new">상품 등록</a>
            <a class="btn btn-lg btn-dark" href="/items">상품 목록</a>
        </p>
        <p class="lead">주문 기능</p>
        <p>
            <a class="btn btn-lg btn-info" href="/order">상품 주문</a>
            <a class="btn btn-lg btn-info" href="/orders">주문 내역</a>
        </p>
    </div>
    <div th:replace="fragments/footer :: footer" />
</div> <!-- /container -->
</body>
</html>
```

fragments/header.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head th:fragment="header">
    <!-- Required meta tags -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrinkto-fit=no">
    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="/css/bootstrap.min.css" integrity="sha384-
ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T"
          crossorigin="anonymous">
    <!-- Custom styles for this template -->
    <link href="/css/jumbotron-narrow.css" rel="stylesheet">
    <title>Hello, world!</title>
</head>
```

fragments/bodyHeader.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<div class="header" th:fragment="bodyHeader">
    <ul class="nav nav-pills pull-right">
        <li><a href="/">Home</a></li>
    </ul>
    <a href="/"><h3 class="text-muted">HELLO SHOP</h3></a>
</div>
```

fragments/footer.html

```html
```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<div class="footer" th:fragment="footer">
    <p>&copy; Hello Shop V2</p>
</div>
```

##### **4) view 리소스 등록**

Bootstrap v5.3.3의 css와 js를 다운로드 받은 후 resources/static 하위에 추가하였다.

추가적으로 아래의 jumbotron-narrow.css 파일도 추가하였다.
```css
/* Space out content a bit */
body {
 padding-top: 20px;
 padding-bottom: 20px;
}
/* Everything but the jumbotron gets side spacing for mobile first views */
.header,
.marketing,
.footer {
 padding-left: 15px;
 padding-right: 15px;
}
/* Custom page header */
.header {
 border-bottom: 1px solid #e5e5e5;
}
/* Make the masthead heading the same height as the navigation */
.header h3 {
 margin-top: 0;
 margin-bottom: 0;
 line-height: 40px;
 padding-bottom: 19px;
}
/* Custom page footer */
.footer {
 padding-top: 19px;
 color: #777;
 border-top: 1px solid #e5e5e5;
}
/* Customize container */
@media (min-width: 768px) {
 .container {
 max-width: 730px;
 }
}
.container-narrow > hr {
 margin: 30px 0;
}
/* Main marketing message and sign up button */
.jumbotron {
 text-align: center;
 border-bottom: 1px solid #e5e5e5;
}
.jumbotron .btn {
 font-size: 21px;
 padding: 14px 24px;
}
/* Supporting marketing content */
.marketing {
 margin: 40px 0;
}
.marketing p + h4 {
 margin-top: 28px;
}
/* Responsive: Portrait tablets and up */
@media screen and (min-width: 768px) {
 /* Remove the padding we set earlier */
 .header,
 .marketing,
 .footer {
 padding-left: 0;
 padding-right: 0;
 }
 /* Space out the masthead */
 .header {
 margin-bottom: 30px;
 }
 /* Remove the bottom border on the jumbotron for visual effect */
 .jumbotron {
 border-bottom: 0;
 }
}
```

##### **5) 실행 시 화면**

- 홈 페이지

![IMAGE](/assets/images/spring-boot-jpa-practice001/0012/home-page.png)