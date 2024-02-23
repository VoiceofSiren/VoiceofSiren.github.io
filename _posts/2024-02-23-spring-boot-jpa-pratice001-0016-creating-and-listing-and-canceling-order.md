---
layout: post
title: "JPA Practice001: #0016 - Creating, Listing, and removing Order"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. 웹 계층 개발 목차

- 상품 주문
- 주문 목록 검색, 취소

## 2. 상품 주문

##### **1) 주문 Controller**

controller.OrderController

```java
package jpabook.jpashop.controller;

import jpabook.jpashop.domain.Member;
import jpabook.jpashop.domain.item.Item;
import jpabook.jpashop.service.ItemService;
import jpabook.jpashop.service.MemberService;
import jpabook.jpashop.service.OrderService;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;

import java.util.List;

@Controller
@RequiredArgsConstructor
public class OrderController {

    private final OrderService orderService;
    private final MemberService memberService;
    private final ItemService itemService;

    @GetMapping("/order")
    public String createOrderForm(Model model) {

        List<Member> members = memberService.findAll();
        List<Item> items = itemService.findAll();

        model.addAttribute("members", members);
        model.addAttribute("items", items);

        return "orders/createOrderForm";
    }

    @PostMapping("/order")
    public String createOrder(@RequestParam(name = "memberId") Long memberId,
                              @RequestParam(name = "itemId") Long itemId,
                              @RequestParam(name = "count") int count) {

        orderService.makeOrder(memberId, itemId, count);
        return "redirect:/orders";
    }
}

```

##### **2) 주문 생성 폼 html**

templates/orders/createOrderForm.html

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header" />
<body>
<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader"/>
    <form role="form" action="/order" method="post">
        <div class="form-group">
            <label for="member">주문회원</label>
            <select name="memberId" id="member" class="form-control">
                <option value="">회원선택</option>
                <option th:each="member : ${members}"
                        th:value="${member.id}"
                        th:text="${member.name}" />
            </select>
        </div>
        <div class="form-group">
            <label for="item">상품명</label>
            <select name="itemId" id="item" class="form-control">
                <option value="">상품선택</option>
                <option th:each="item : ${items}"
                        th:value="${item.id}"
                        th:text="${item.name}" />
            </select>
        </div>
        <div class="form-group">
            <label for="count">주문수량</label>
            <input type="number" name="count" class="form-control" id="count"
                   placeholder="주문 수량을 입력하세요">
        </div>
        <button type="submit" class="btn btn-primary">Submit</button>
    </form>
    <br/>
    <div th:replace="fragments/footer :: footer" />
</div> <!-- /container -->
</body>
</html>

```

주문 생성 화면

![IMAGE](/assets/images/spring-boot-jpa-practice001/0016/create-order-form.png)

주문 생성 시 log 기록

![IMAGE](/assets/images/spring-boot-jpa-practice001/0016/logs-of-creating-order.png)

<br/>

## 3. 주문 목록 검색, 취소

### **3-1. 주문 목록 검색**

##### **1) 상품 Controller**

controller.itemController에 아래의 메서드를 추가한다.

```java
    @GetMapping("/items")
    public String list(Model model) {
        List<Item> items = itemService.findAll();
        model.addAttribute("items", items);
        return "items/itemList";
    }
```

##### **2) 주문 목록 html**

templates/orders/orderList.html

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header"/>
<body>
<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader"/>
    <div>
        <div>
            <form th:object="${orderSearch}" class="form-inline">
                <div class="form-group mb-2">
                    <input type="text" th:field="*{memberName}" class="form-control" placeholder="회원명"/>
                </div>
                <div class="form-group mx-sm-1 mb-2">
                    <select th:field="*{orderStatus}" class="form-control">
                        <option value="">주문상태</option>
                        <option th:each="status : ${T(jpabook.jpashop.domain.OrderStatus).values()}"
                                th:value="${status}"
                                th:text="${status}">option
                        </option>
                    </select>
                </div>
                <button type="submit" class="btn btn-primary mb-2">검색</button>
            </form>
        </div>
        <table class="table table-striped">
            <thead>
            <tr>
                <th>#</th>
                <th>회원명</th>
                <th>대표상품 이름</th>
                <th>대표상품 주문가격</th>
                <th>대표상품 주문수량</th>
                <th>상태</th>
                <th>일시</th>
                <th></th>
            </tr>
            </thead>
            <tbody>
            <tr th:each="item : ${orders}">
                <td th:text="${item.id}"></td>
                <td th:text="${item.member.name}"></td>
                <td th:text="${item.orderItems[0].item.name}"></td>
                <td th:text="${item.orderItems[0].orderPrice}"></td>
                <td th:text="${item.orderItems[0].count}"></td>
                <td th:text="${item.orderStatus}"></td>
                <td th:text="${item.orderDate}"></td>
                <td>
                    <a th:if="${item.orderStatus.name() == 'ORDER'}" href="#"
                       th:href="'javascript:cancel('+${item.id}+')'"
                       class="btn btn-danger">CANCEL</a>
                </td>
            </tr>
            </tbody>
        </table>
    </div>
    <div th:replace="fragments/footer :: footer"/>
</div> <!-- /container -->
</body>
<script>
    function cancel(id) {
        var form = document.createElement("form");
        form.setAttribute("method", "post");
        form.setAttribute("action", "/orders/" + id + "/cancel");
        document.body.appendChild(form);
        form.submit();
    }
</script>
</html>
```

##### **3) 실행 시 화면**

주문 조회 화면

![IMAGE](/assets/images/spring-boot-jpa-practice001/0016/listing-order.png)

주문 조회 시 log 기록

![IMAGE](/assets/images/spring-boot-jpa-practice001/0016/logs-of-listing-order-1.png)

![IMAGE](/assets/images/spring-boot-jpa-practice001/0016/logs-of-listing-order-2.png)


### **3-2. 주문 취소**

##### **1) 주문 Controller**

controller.OrderController에 아래의 메서드를 추가한다.

```java
    @PostMapping("/orders/{orderId}/cancel")
    public String cancelOrder(@PathVariable(name = "orderId") Long orderId) {
        orderService.cancelOrder(orderId);
        return "redirect:/orders";
    }
```

##### **2) 실행 시 화면**

주문 취소 후 화면

![IMAGE](/assets/images/spring-boot-jpa-practice001/0016/after-canceling-order.png)

주문 취소 시 log 기록

![IMAGE](/assets/images/spring-boot-jpa-practice001/0016/logs-of-canceling-order.png)