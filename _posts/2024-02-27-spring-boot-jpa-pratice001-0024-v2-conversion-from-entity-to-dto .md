---
layout: post
title: "JPA Practice001: #0024 - [Collection Fetch Optimization V2] Conversion from Entity to DTO"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. 컬렉션 조회 최적화

- 주문 내역에서 주문한 여러 상품 정보를을 추가로 조회하는 API를 개발하고자 한다.
- 1:N 연관 관계에 있는 컬렉션을 조회하고 최적화하는 방법을 알아볼 것이다.

#### **목차**

- 주문 조회 v2: Entity를 DTO로 변환
- 주문 조회 v3: Entity를 DTO로 변환 후 Fetch JOIN 최적화 (다음 글에서)
- Paging과 한계 돌파 (다음 글에서)
- 주문 조회 v4: JPA에서 DTO를 직접 조회 (다음 글에서)
- 주문 조회 v5: JPA에서 DTO를 직접 조회, 컬렉션 조회 최적화 (다음 글에서)
- 주문 조회 v6: JPA에서 DTO를 직접 조회, 플랫 조회 최적화 (다음 글에서)
- API 개발 정리 (다음 글에서)
<br/>

## 2. (v2) 주문 조회: Order Entity를 DTO로 변환

#### **1) 주문 RestController 코드**

api.OrderApiController에 아래의 코드를 추가한다.

```java
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
    private List<OrderItem> orderItems;

    public OrderDTO(Order order) {
        orderId = order.getId();
        name = order.getMember().getName();
        orderDate = order.getOrderDate();
        orderStatus = order.getOrderStatus();
        address = order.getDelivery().getAddress();
        order.getOrderItems().stream().forEach(orderItem -> orderItem.getItem().getName()); // Proxy 강제 초기화
        orderItems = order.getOrderItems();
    }
}
```

#### **2) 요청과 응답**

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

![IMAGE](/assets/images/spring-boot-jpa-practice001/0024/postman-v2-dto.png)

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
- 아래와 같은 순서대로 SELECT문이 실행된다.
- ORDERS -> MEMBER -> DELIVERY -> ORDER_ITEM -> ITEM -> ITEM
         -> MEMBER -> DELIVERY -> ORDER_ITEM -> ITEM -> ITEM
```

#### **3) 문제점 분석 및 해결 방법**

##### **문제점 분석**
```plaintext
- OrderItem Entity를 은닉화하는 데까지는 성공하였다.
- 그러나 SELECT문이 무려 11번이나 실행된다.
```
##### **해결 방법**
```plaintext
- Collection을 Fetch JOIN할 것이다.
```
