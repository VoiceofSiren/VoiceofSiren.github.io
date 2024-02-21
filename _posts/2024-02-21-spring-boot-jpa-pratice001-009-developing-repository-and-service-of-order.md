---
layout: post
title: "JPA Practice001: #009 - Developing Repository and Service of Order"
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
  - 주문 Repository 개발
  - 주문 Service 개발
  - 주문 검색 기능 개발 (다음 글에서)
  - 주문 기능 테스트 (다음 글에서)
<br/>

## 2. 주문 Repository 개발

#### **1) 주문 Repository 코드**

검색 기능은 동적 쿼리로 추후에 구현할 예정이다.

```java
package jpabook.jpashop.repository;

import jakarta.persistence.EntityManager;
import jpabook.jpashop.domain.Order;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Repository;

@Repository
@RequiredArgsConstructor
public class OrderRepository {

  private final EntityManager em;

  public Long save(Order order) {
    em.persist(order);
    return order.getId();
  }

  public Order findOne(Long orderId) {
    return em.find(Order.class, orderId);
  }

  // 검색 기능 - 구현 예정
  // public List<Order> findAll(OrderSearch orderSearch) { ... }
  
}

```
<br/>

## 3. 주문 Service 개발

#### **1) 주문 Service 코드**

검색 기능은 동적 쿼리로 추후에 구현할 예정이다.

```java
package jpabook.jpashop.service;

import jpabook.jpashop.domain.Delivery;
import jpabook.jpashop.domain.Member;
import jpabook.jpashop.domain.Order;
import jpabook.jpashop.domain.OrderItem;
import jpabook.jpashop.domain.item.Item;
import jpabook.jpashop.repository.ItemRepository;
import jpabook.jpashop.repository.MemberRepository;
import jpabook.jpashop.repository.OrderRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class OrderService {

  private final MemberRepository memberRepository;
  private final ItemRepository itemRepository;
  private final OrderRepository orderRepository;

  /**
   * 주문 생성
   */
  @Transactional
  public Long makeOrder(Long memberId, Long itemId, int count) {

    // 1. Entity 조회
    Member member = memberRepository.findOne(memberId);
    Item item = itemRepository.findOne(itemId);

    // 2. 배송 정보 생성
    Delivery delivery = new Delivery();
    delivery.setAddress(member.getAddress());

    // 3. 주문 상품 생성
    OrderItem orderItem = OrderItem.createOrderItem(item, item.getPrice(), count);

    // 4. 주문 생성
    Order order = Order.createOrder(member, delivery, orderItem);

    // 5. 주문 저장
    return orderRepository.save(order);     // cascade = CascadeType.ALL 옵션에 따라 OrderItem과 Delivery도 연쇄적으로 persist됨.
  }


  /**
   * 주문 취소
   */
  @Transactional
  public void cancelOrder(Long orderId) {

    // 1. 주문 Entity 조회
    Order order = orderRepository.findOne(orderId);

    // 2. 주문 취소
    order.cancel();
  }


  /**
   * 주문 검색
   */
/*
    public List<Order> findOrders(OrderSearch orderSearch) {
        return orderRepository.findAll(orderSearch);
    }
*/
}

```

#### **2) 기능 설명**

- 주문 생성 - makeOrder()
```plaintext
-- 회원 식별자, 상품 식별자, 주문 수량 정보를 받아서 실제 주문 (Order) Entity를 생성한 후 저장한다.
```

- 주문 취소 - cancelOrder()
```plaintext
-- 주문 식별자를 받아서 주문 Entity를 조회한 후 주문 Entity에 주문 취소를 요청한다.
```

- 주문 검색 - findOrders()
```plaintext
-- OrderSearch라는 검색 조건을 가진 객체로 주문 Entity를 검색한다.
-- 추후 구현할 예정이다.
```
<br/>