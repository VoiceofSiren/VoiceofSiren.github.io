---
layout: post
title: "JPA Practice001: #0010 - Testing Order Methods"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. 주문 도메인 개발 목차

- 구현 기능
  - 상품 주문
  - 주문 내역 조회
  - 주문 취소

- 순서
  - 주문 기능 테스트
  - 주문 검색 기능 개발 (다음 글에서)
<br/>

## 2. 주문 기능 테스트

- 테스트 요구 사항
  - 상품 주문이 성공해야 한다.
  - 상품을 주문할 때 재고 수량을 초과하면 안 된다.
  - 주문 취소가 성공해야 한다.

#### **1) 주문 Service Test 코드 - 과정 1**

상품 주문 생성에 대한 테스트 코드를 먼저 작성하였다.

```java
package jpabook.jpashop.service;

import jakarta.persistence.EntityManager;
import jpabook.jpashop.domain.Address;
import jpabook.jpashop.domain.Member;
import jpabook.jpashop.domain.Order;
import jpabook.jpashop.domain.OrderStatus;
import jpabook.jpashop.domain.item.Book;
import jpabook.jpashop.domain.item.Item;
import jpabook.jpashop.repository.OrderRepository;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.annotation.Transactional;

import static org.junit.Assert.*;

@RunWith(SpringRunner.class)
@SpringBootTest
@Transactional
public class OrderServiceTest {

  @Autowired private EntityManager em;
  @Autowired private OrderRepository orderRepository;
  @Autowired private OrderService orderService;

  @Test
  public void 상품_주문_생성() throws Exception {
    //given
    Member member = new Member();
    member.setName("회원1");
    member.setAddress(new Address("서울", "강가", "123123"));
    em.persist(member);

    Book book = new Book();
    book.setName("시골 JPA");
    book.setPrice(10000);
    book.setStockQuantity(10);
    em.persist(book);

    int orderCount = 2;

    //when
    Long orderId = orderService.makeOrder(member.getId(), book.getId(), orderCount);

    //then
    Order foundOrder = orderRepository.findOne(orderId);

    assertEquals("상품 주문 시 상태는 ORDER", OrderStatus.ORDER, foundOrder.getOrderStatus());
    assertEquals("주문한 상품 종류의 수가 정확해야 한다.", 1, foundOrder.getOrderItems().size());
    assertEquals("주문 가격은 가격 * 수량이다.", 10000 * orderCount, foundOrder.getTotalPrice());
    assertEquals("주문 수량만큼 재고가 줄어야 한다.", 10 - orderCount, book.getStockQuantity());
  }

  @Test
  public void 상품_주문_취소() throws Exception {
    //given

    //when

    //then

  }

  @Test
  public void 상품_주문_재고_수량_초과() throws Exception {
    //given

    //when

    //then

  }
}
```

상품_주문_생성()만을 실행시킬 경우 테스트를 정상적으로 통과하였음을 확인할 수 있다.

#### **2) 주문 Service Test 코드 - 과정 2**

테스트를 위한 용도로 Member Entity와 Book Entity를 생성하는 메서드를 추가하였다.
주문하려는 상품의 재고보다 주문 수량이 많을 경우 NotEnoughStockException 예외를 발생시키는 상품_주문_재고_수량_초과() 테스트 코드를 구현하였다.

```java
package jpabook.jpashop.service;

import jakarta.persistence.EntityManager;
import jpabook.jpashop.domain.Address;
import jpabook.jpashop.domain.Member;
import jpabook.jpashop.domain.Order;
import jpabook.jpashop.domain.OrderStatus;
import jpabook.jpashop.domain.item.Book;
import jpabook.jpashop.exception.NotEnoughStockException;
import jpabook.jpashop.repository.OrderRepository;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.annotation.Transactional;

import static org.junit.Assert.*;

@RunWith(SpringRunner.class)
@SpringBootTest
@Transactional
public class OrderServiceTest {

  @Autowired private EntityManager em;
  @Autowired private OrderRepository orderRepository;
  @Autowired private OrderService orderService;

  @Test
  public void 상품_주문_생성() throws Exception {
    //given
    Member member = createMemberForTest();
    Book book = createBookForTest("시골 JPA", 10000, 10);
    int orderCount = 2;

    //when
    Long orderId = orderService.makeOrder(member.getId(), book.getId(), orderCount);

    //then
    Order foundOrder = orderRepository.findOne(orderId);

    assertEquals("상품 주문 시 상태는 ORDER", OrderStatus.ORDER, foundOrder.getOrderStatus());
    assertEquals("주문한 상품 종류의 수가 정확해야 한다.", 1, foundOrder.getOrderItems().size());
    assertEquals("주문 가격은 가격 * 수량이다.", 10000 * orderCount, foundOrder.getTotalPrice());
    assertEquals("주문 수량만큼 재고가 줄어야 한다.", 10 - orderCount, book.getStockQuantity());
  }

  @Test(expected = NotEnoughStockException.class)
  public void 상품_주문_재고_수량_초과() throws Exception {
    //given
    Member member = createMemberForTest();
    Book book = createBookForTest("시골 JPA", 10000, 10);
    int orderCount = 11;

    //when
    orderService.makeOrder(member.getId(), book.getId(), orderCount);

    //then
    fail("재고 수량 부족 예외가 발생해야 한다.");

  }

  @Test
  public void 상품_주문_취소() throws Exception {
    //given

    //when

    //then

  }

  public Member createMemberForTest() {
    Member member = new Member();
    member.setName("회원1");
    member.setAddress(new Address("서울", "강가", "123123"));
    em.persist(member);
    return member;
  }

  public Book createBookForTest(String name, int price, int stockQuantity) {
    Book book = new Book();
    book.setName(name);
    book.setPrice(price);
    book.setStockQuantity(stockQuantity);
    em.persist(book);
    return book;
  }


}
```
<br/>


#### **3) 주문 Service Test 코드 - 과정 3**

```java
package jpabook.jpashop.service;

import jakarta.persistence.EntityManager;
import jpabook.jpashop.domain.Address;
import jpabook.jpashop.domain.Member;
import jpabook.jpashop.domain.Order;
import jpabook.jpashop.domain.OrderStatus;
import jpabook.jpashop.domain.item.Book;
import jpabook.jpashop.exception.NotEnoughStockException;
import jpabook.jpashop.repository.OrderRepository;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.annotation.Transactional;

import static org.junit.Assert.*;

@RunWith(SpringRunner.class)
@SpringBootTest
@Transactional
public class OrderServiceTest {

    @Autowired private EntityManager em;
    @Autowired private OrderRepository orderRepository;
    @Autowired private OrderService orderService;

    @Test
    public void 상품_주문_생성() throws Exception {
        //given
        Member member = createMemberForTest();
        Book book = createBookForTest("시골 JPA", 10000, 10);
        int orderCount = 2;

        //when
        Long orderId = orderService.makeOrder(member.getId(), book.getId(), orderCount);

        //then
        Order foundOrder = orderRepository.findOne(orderId);

        assertEquals("상품 주문 시 상태는 ORDER", OrderStatus.ORDER, foundOrder.getOrderStatus());
        assertEquals("주문한 상품 종류의 수가 정확해야 한다.", 1, foundOrder.getOrderItems().size());
        assertEquals("주문 가격은 가격 * 수량이다.", 10000 * orderCount, foundOrder.getTotalPrice());
        assertEquals("주문 수량만큼 재고가 줄어야 한다.", 10 - orderCount, book.getStockQuantity());
    }

    @Test(expected = NotEnoughStockException.class)
    public void 상품_주문_재고_수량_초과() throws Exception {
        //given
        Member member = createMemberForTest();
        Book book = createBookForTest("시골 JPA", 10000, 10);
        int orderCount = 11;

        //when
        orderService.makeOrder(member.getId(), book.getId(), orderCount);

        //then
        fail("재고 수량 부족 예외가 발생해야 한다.");

    }

    @Test
    public void 상품_주문_취소() throws Exception {
        //given
        Member member = createMemberForTest();
        Book book = createBookForTest("시골 JPA", 10000, 10);
        int orderCount = 2;
        Long orderId = orderService.makeOrder(member.getId(), book.getId(), orderCount);

        //when
        orderService.cancelOrder(orderId);

        //then
        Order foundOrder = orderRepository.findOne(orderId);

        assertEquals("주문 취소 시 상태는 CANCEL이다.", OrderStatus.CANCEL, foundOrder.getOrderStatus());
        assertEquals("주문이 취소된 상품의 재고가 원복되어야 한다.", 10, book.getStockQuantity());

    }

    public Member createMemberForTest() {
        Member member = new Member();
        member.setName("회원1");
        member.setAddress(new Address("서울", "강가", "123123"));
        em.persist(member);
        return member;
    }

    public Book createBookForTest(String name, int price, int stockQuantity) {
        Book book = new Book();
        book.setName(name);
        book.setPrice(price);
        book.setStockQuantity(stockQuantity);
        em.persist(book);
        return book;
    }
    
}
```
지금까지 구현한 상품_주문_생성()과 상품_주문_재고_수량_초과(), 상품_주문_취소()를 실행시킬 경우 모두 테스트를 정상적으로 통과하였음을 확인할 수 있다.

![IMAGE](/assets/images/spring-boot-jpa-practice001/0010/all-tests-passed.png)