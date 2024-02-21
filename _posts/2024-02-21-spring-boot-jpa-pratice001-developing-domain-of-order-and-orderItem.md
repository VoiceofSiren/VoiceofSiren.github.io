---
layout: post
title: "JPA Practice001: #008 - Developing Domain of Order and OrderItem"
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
  - 주문 Entity
  - 주문 상품 Entity 개발
  - 주문 Repository 개발 (다음 글에서)
  - 주문 Service 개발 (다음 글에서)
  - 주문 검색 기능 개발 (다음 글에서)
  - 주문 기능 테스트 (다음 글에서)
<br/>

## 2. 주문 Entity 개발

#### **1) 주문 Entity 코드**

domain.Order 클래스에 생성 메서드, 비즈니스 로직, 조회 로직을 추가하였다.

```java
package jpabook.jpashop.domain;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;

import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

@Entity
@Getter
@Setter
@Table(name = "orders")
public class Order {

  @Id
  @GeneratedValue
  @Column(name = "order_id")
  private Long id;

  @ManyToOne(fetch = FetchType.LAZY)
  @JoinColumn(name = "member_id")
  private Member member;

  @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
  private List<OrderItem> orderItems = new ArrayList<>();

  @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
  @JoinColumn(name = "delivery_id")
  private Delivery delivery;

  private LocalDateTime orderDate;

  @Enumerated(EnumType.STRING)
  private OrderStatus orderStatus;

  //==연관 관계 메서드==//
  public void setMember(Member member) {
    this.member = member;
    member.getOrders().add(this);
  }

  public void addOrderItem(OrderItem orderItem) {
    orderItems.add(orderItem);
    orderItem.setOrder(this);
  }

  public void setDelivery(Delivery delivery) {
    this.delivery = delivery;
    delivery.setOrder(this);
  }

  //==생성 메서드==//
  public static Order createOrder(Member member, Delivery delivery, OrderItem... orderItems) {
    Order order = new Order();
    order.setMember(member);
    order.setDelivery(delivery);
    for (OrderItem orderItem: orderItems) {
      order.addOrderItem(orderItem);  // 연관 관계 메서드
    }
    order.setOrderStatus(OrderStatus.ORDER);
    order.setOrderDate(LocalDateTime.now());
    return order;
  }

  //==비즈니스 로직==//
  /**
   * 주문 취소
   */
  public void cancel() {
    if (delivery.getStatus() == DeliveryStatus.COMPLETE) {
      throw new IllegalStateException("이미 배송이 완료된 상품의 주문은 취소할 수 없습니다.");
    }

    this.setOrderStatus(OrderStatus.CANCEL);
    for (OrderItem orderItem: this.orderItems) {
      orderItem.cancel();         // OrderItem Entity의 비즈니스 로직 호출
    }
  }

  //==조회 로직==//
  /**
   * 전체 주문 가격 조회
   */
  public int getTotalPrice() {
        /*
        int totalPrice = 0;
        for (OrderItem orderItem: this.orderItems) {
            totalPrice += orderItem.getTotalPrice();
        }
        return totalPrice;
        */
    return orderItems.stream()
            .mapToInt(OrderItem::getTotalPrice)
            .sum();
  }
}
```

#### **2) 기능 설명**

- 생성 메서드 - createOrder()
```plaintext
-- Order Entity를 생성할 때 사용된다.
-- 주문 회원 (Member), 배송 정보 (Delivery), 주문 상품 (OrderItem...)의 정보를 인자로 받아서 Order Entity를 생성한다.
```

- 비즈니스 로직: 주문 취소 - cancel()
```plaintext
-- 주문 취소 시 사용된다.
-- 주문 상태 (OrderStatus)를 취소 (OrderStatus.CANCEL)로 변경하고 각각의 주문 상품 (OrderItem)에 주문 취소를 알린다.
-- 만약 이미 배송이 완료된 상품이라면 취소하지 못 하도록 예외를 발생시킨다.
```

- 조회 로직: 전체 주문 가격 조회 - getTotalPrice()
```plaintext
-- 주문 시 해당 주문의 전체 주문 가격을 조회할 때 사용된다.
-- 전체 주문 가격을 알려면 각각의 주문 상품 가격을 알아야 한다.
-- 연관된 주문 상품들의 가격을 조회하여 더한 값을 반환한다.
```
<br/>

## 3. 주문 상품 Entity 개발

#### **1) 주문 상품 Entity 코드**

domain.OrderItem 클래스에 생성 메서드, 비즈니스 로직, 조회 로직을 추가하였다.

```java
package jpabook.jpashop.domain;

import jakarta.persistence.*;
import jpabook.jpashop.domain.item.Item;
import lombok.Getter;
import lombok.Setter;

@Entity
@Getter
@Setter
public class OrderItem {

  @Id
  @GeneratedValue
  @Column(name = "order_item_id")
  private Long id;

  @ManyToOne(fetch = FetchType.LAZY)
  @JoinColumn(name = "order_id")
  private Order order;

  @ManyToOne(fetch = FetchType.LAZY)
  @JoinColumn(name = "item_id")
  private Item item;

  private int orderPrice;

  private int count;

  //==생성 메서드==//
  public static OrderItem createOrderItem(Item item, int orderPrice, int count) {
    OrderItem orderItem = new OrderItem();
    orderItem.setItem(item);
    orderItem.setOrderPrice(orderPrice);
    orderItem.setCount(count);

    item.removeStock(count);    // Item Entity의 비즈니스 로직 호출
    return orderItem;
  }

  //==비즈니스 로직==//
  public void cancel() {
    this.item.addStock(this.count);
  }

  //==조회 로직==//

  /**
   * 주문 상품 전체 가격 조회
   */
  public int getTotalPrice() {
    return this.orderPrice * this.count;
  }
}

```

#### **2) 기능 설명**

- 생성 메서드 - createOrderItem()
```plaintext
-- 상품 (Item), 상품 가격, 수량 정보를 인자로 받아 OrderItem Entity를 생성할 때 사용된다.
-- item.removeStock(count)를 호출하여 주문한 수량만큼 상품의 재고를 줄인다.
```

- 비즈니스 로직: 주문 취소 - cancel()
```plaintext
-- item.addStock(count)를 호출해서 취소한 주문 수량만큼 상품의 재고를 증가시킨다.
```

- 조회 로직: 전체 주문 가격 조회 - getTotalPrice()
```plaintext
-- 주문 가격에 수량을 곱한 값을 반환한다.
```
<br/>
```