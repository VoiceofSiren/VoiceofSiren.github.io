---
layout: post
title: "JPA Practice001: #0025 - [Collection Fetch Optimization V3] Collection Fetch JOIN"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. 컬렉션 조회 최적화

- JPA의 Fetch JOIN 기능을 사용하여 성능 최적화 v2에 있었던 'N + 1 문제'를 해결하고자 한다.

#### **목차**

- 주문 조회 v3: Entity를 DTO로 변환 후 Fetch JOIN 최적화
- Paging과 한계 돌파 (다음 글에서)
- 주문 조회 v4: JPA에서 DTO를 직접 조회 (다음 글에서)
- 주문 조회 v5: JPA에서 DTO를 직접 조회, 컬렉션 조회 최적화 (다음 글에서)
- 주문 조회 v6: JPA에서 DTO를 직접 조회, 플랫 조회 최적화 (다음 글에서)
- API 개발 정리 (다음 글에서)
<br/>

## 2. (v3) Fetch JOIN 사용

#### **1) 주문 Repository 코드**

repository.OrderRepository에 아래의 메서드를 추가한다.

```java
    public List<Order> findAllWithItem() {
        return em.createQuery(
        "select o from Order o " +
        "join fetch o.member m " +
        "join fetch o.delivery d " +
        "join fetch o.orderItems oi " +
        "join fetch oi.item", Order.class
                ).getResultList();
    }
```

#### **2) 주문 RestController 코드**

api.OrderApiController에 아래의 메서드를 추가한다.

```java
    @GetMapping("/api/v3/orders")
    public List<OrderDTO> ordersV3() {
        List<Order> orders = orderRepository.findAllWithItem();
        return orders.stream()
                .map(order -> new OrderDTO(order))
                .collect(Collectors.toList());
    }
```

#### **3) 요청과 응답**

Postman 실행

![IMAGE](/assets/images/spring-boot-jpa-practice001/0024/postman-v2-entity.png)

```json
[
  {
    "orderId": 1,
    "name": "userA",
    "orderDate": "2024-02-27T14:20:47.176633",
    "orderStatus": "ORDER",
    "address": {
      "city": "서울",
      "street": "1",
      "zipcode": "1111"
    },
    "orderItems": [
      {
        "id": 1,
        "item": {
          "id": 1,
          "name": "JPA1 BOOK",
          "price": 10000,
          "stockQuantity": 99,
          "categories": null,
          "author": null,
          "isbn": null
        },
        "orderPrice": 10000,
        "count": 1,
        "totalPrice": 10000
      },
      {
        "id": 2,
        "item": {
          "id": 2,
          "name": "JPA2 BOOK",
          "price": 20000,
          "stockQuantity": 98,
          "categories": null,
          "author": null,
          "isbn": null
        },
        "orderPrice": 20000,
        "count": 2,
        "totalPrice": 40000
      }
    ]
  },
  {
    "orderId": 2,
    "name": "userB",
    "orderDate": "2024-02-27T14:20:47.217274",
    "orderStatus": "ORDER",
    "address": {
      "city": "진주",
      "street": "2",
      "zipcode": "2222"
    },
    "orderItems": [
      {
        "id": 3,
        "item": {
          "id": 3,
          "name": "SPRING1 BOOK",
          "price": 20000,
          "stockQuantity": 197,
          "categories": null,
          "author": null,
          "isbn": null
        },
        "orderPrice": 20000,
        "count": 3,
        "totalPrice": 60000
      },
      {
        "id": 4,
        "item": {
          "id": 4,
          "name": "SPRING2 BOOK",
          "price": 40000,
          "stockQuantity": 296,
          "categories": null,
          "author": null,
          "isbn": null
        },
        "orderPrice": 40000,
        "count": 4,
        "totalPrice": 160000
      }
    ]
  }
]
```

JPA log 기록

```plaintext
- 아래와 같은 순서대로 SELECT문이 실행된다.
- ORDERS -> MEMBER -> DELIVERY -> ORDER_ITEM -> ITEM -> ITEM
         -> MEMBER -> DELIVERY -> ORDER_ITEM -> ITEM -> ITEM
```

#### **3) 문제점 분석 및 해결 방법**

##### **에러 분석**
```plaintext
- OrderItem Entity의 모든 정보가 노출되었다.
```
##### **해결 방법**
```plaintext
- Order Entity를 DTO로 변환하는 것뿐만 아니라 OrderItem Entity도 DTO로 변환해줘야 한다.
```

-------------------------------------------------------------


## 2. (v2) 주문 조회: OrderItem Entity를 DTO로 변환

#### **1) 주문 RestController 코드**

api.OrderApiController

```java
package jpabook.jpashop.api;

import jpabook.jpashop.domain.Address;
import jpabook.jpashop.domain.Order;
import jpabook.jpashop.domain.OrderItem;
import jpabook.jpashop.domain.OrderStatus;
import jpabook.jpashop.repository.OrderRepository;
import jpabook.jpashop.repository.OrderSearch;
import lombok.Data;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.time.LocalDateTime;
import java.util.List;
import java.util.stream.Collectors;

@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;

    @GetMapping("/api/v2/orders")
    public List<OrderDTO> ordersV2() {
        List<Order> orders = orderRepository.findAllByString(new OrderSearch());
        return orders.stream()
                .map(order -> new OrderDTO(order))
                .collect(Collectors.toList());
    }

    @Data
    static class OrderDTO {
        private Long orderId;
        private String name;
        private LocalDateTime orderDate;
        private OrderStatus orderStatus;
        private Address address;
        private List<OrderItemDTO> orderItems;

        public OrderDTO(Order order) {
            orderId = order.getId();
            name = order.getMember().getName();
            orderDate = order.getOrderDate();
            orderStatus = order.getOrderStatus();
            address = order.getDelivery().getAddress();
            orderItems = order.getOrderItems().stream()
                    .map(orderItem -> new OrderItemDTO(orderItem))
                    .collect(Collectors.toList());
        }
    }

    @Data
    static class OrderItemDTO {

        private String itemName;
        private int orderPrice;
        private int count;

        public OrderItemDTO(OrderItem orderItem) {
            itemName = orderItem.getItem().getName();
            orderPrice = orderItem.getOrderPrice();
            count = orderItem.getCount();
        }
    }
}
```

#### **2) 요청과 응답**

Postman 실행

![IMAGE](/assets/images/spring-boot-jpa-practice001/0025/postman-v3-duplicated.png)

```json
[
  {
    "orderId": 1,
    "name": "userA",
    "orderDate": "2024-02-27T14:32:38.93015",
    "orderStatus": "ORDER",
    "address": {
      "city": "서울",
      "street": "1",
      "zipcode": "1111"
    },
    "orderItems": [
      {
        "itemName": "JPA1 BOOK",
        "orderPrice": 10000,
        "count": 1
      },
      {
        "itemName": "JPA2 BOOK",
        "orderPrice": 20000,
        "count": 2
      }
    ]
  },
  {
    "orderId": 2,
    "name": "userB",
    "orderDate": "2024-02-27T14:32:39.00471",
    "orderStatus": "ORDER",
    "address": {
      "city": "진주",
      "street": "2",
      "zipcode": "2222"
    },
    "orderItems": [
      {
        "itemName": "SPRING1 BOOK",
        "orderPrice": 20000,
        "count": 3
      },
      {
        "itemName": "SPRING2 BOOK",
        "orderPrice": 40000,
        "count": 4
      }
    ]
  }
]
```

JPA log 기록

```plaintext
- JPA 구현체인 Hibernate는 아래의 H2 DBMS SQL을 실행시킨다.
```

```sql
select
        o1_0.order_id,
        d1_0.delivery_id,
        d1_0.city,
        d1_0.street,
        d1_0.zipcode,
        d1_0.status,
        m1_0.member_id,
        m1_0.city,
        m1_0.street,
        m1_0.zipcode,
        m1_0.name,
        o1_0.order_date,
        oi1_0.order_id,
        oi1_0.order_item_id,
        oi1_0.count,
        i1_0.item_id,
        i1_0.dtype,
        i1_0.name,
        i1_0.price,
        i1_0.stock_quantity,
        i1_0.artist,
        i1_0.etc,
        i1_0.author,
        i1_0.isbn,
        i1_0.actor,
        i1_0.director,
        oi1_0.order_price,
        o1_0.order_status
    from
        orders o1_0
    join
        member m1_0
            on m1_0.member_id=o1_0.member_id
    join
        delivery d1_0
            on d1_0.delivery_id=o1_0.delivery_id
    join
        order_item oi1_0
            on o1_0.order_id=oi1_0.order_id
    join
        item i1_0
            on i1_0.item_id=oi1_0.item_id;
```

H2 DBMS

![IMAGE](/assets/images/spring-boot-jpa-practice001/0025/h2-duplicated-order-id.png)

#### **3) 문제점 분석 및 해결 방법**

##### **문제점 분석**
```plaintext
- order_id는 Primary Key로서 Order Entity를 유일하게 구분해줘야 하는데 동일한 order_id가 두 번씩 중복으로 조회되었다.
```
##### **해결 방법**
```plaintext
- JPQL의 SELECT문에 distinct 키워드를 추가한다.
```
```jpaql
select distinct o from Order o 
    join fetch o.member m
    join fetch o.delivery d
    join fetch o.orderItems oi
    join fetch oi.item i
```

- 컬렉션 Fetch JOIN 사용 시 Paging이 불가하며, 2개 이상의 컬렉션에 대하여 Fetch JOIN을 사용하지 말아야 한다.