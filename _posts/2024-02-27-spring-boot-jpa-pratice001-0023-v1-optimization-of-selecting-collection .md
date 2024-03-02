---
layout: post
title: "JPA Practice001: #0023 - [Collection Fetch Optimization V1] Order Lookup - Direct Entity Exposure"
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

- 주문 조회 v1: Entity 직접 노출
- 주문 조회 v2: Entity를 DTO로 변환 (다음 글에서)
- 주문 조회 v3: Entity를 DTO로 변환 후 Fetch JOIN 최적화 (다음 글에서)
- Paging과 한계 돌파 (다음 글에서)
- 주문 조회 v4: JPA에서 DTO를 직접 조회 (다음 글에서)
- 주문 조회 v5: JPA에서 DTO를 직접 조회, 컬렉션 조회 최적화 (다음 글에서)
- 주문 조회 v6: JPA에서 DTO를 직접 조회, 플랫 조회 최적화 (다음 글에서)
- API 개발 정리 (다음 글에서)
<br/>

## 2. (v1) 간단한 주문 조회: Entity를 직접 노출

- Order Entity와 연관되어 있는 Member와 Delivery까지만 조회하고자 한다.

#### **1) 주문 RestController 코드**

api.OrderApiController

```java
package jpabook.jpashop.api;

import jpabook.jpashop.domain.Order;
import jpabook.jpashop.domain.OrderItem;
import jpabook.jpashop.repository.OrderRepository;
import jpabook.jpashop.repository.OrderSearch;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;

    @GetMapping("/api/v1/orders")
    public List<Order> ordersV1() {
        List<Order> orders = orderRepository.findAllByString(new OrderSearch());
        for (Order order : orders) {
            order.getMember().getName();                // LAZY 초기화
            order.getDelivery().getAddress();           // LAZY 초기화
            List<OrderItem> orderItems = order.getOrderItems();
            orderItems.stream().forEach(orderItem -> orderItem.getItem().getName());    // LAZY 초기화
        }
        return orders;
    }
}
```

#### **2) 응답과 요청**

Postman 실행

![IMAGE](/assets/images/spring-boot-jpa-practice001/0023/postman-v1.png)

```json
[
    {
        "id": 1,
        "member": {
            "id": 1,
            "name": "userA",
            "address": {
                "city": "서울",
                "street": "1",
                "zipcode": "1111"
            }
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
        ],
        "delivery": {
            "id": 1,
            "address": {
                "city": "서울",
                "street": "1",
                "zipcode": "1111"
            },
            "status": null
        },
        "orderDate": "2024-02-27T11:46:13.202287",
        "orderStatus": "ORDER",
        "totalPrice": 50000
    },
    {
        "id": 2,
        "member": {
            "id": 2,
            "name": "userB",
            "address": {
                "city": "진주",
                "street": "2",
                "zipcode": "2222"
            }
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
        ],
        "delivery": {
            "id": 2,
            "address": {
                "city": "진주",
                "street": "2",
                "zipcode": "2222"
            },
            "status": null
        },
        "orderDate": "2024-02-27T11:46:13.281544",
        "orderStatus": "ORDER",
        "totalPrice": 220000
    }
]
```


log 기록

```plaintext
- 아래와 같은 순서대로 SELECT문이 실행된다.
- ORDERS -> MEMBER -> DELIVERY -> ORDER_ITEM -> ITEM -> ITEM
         -> MEMBER -> DELIVERY -> ORDER_ITEM -> ITEM -> ITEM
```

#### **3) 문제점 분석 및 해결 방법**

##### **에러 분석**
```plaintext
- SELECT문이 무려 11번이나 실행되었다.
```
##### **해결 방법**
```plaintext
- ENTITY를 직접 노출하지 않고 DTO로 변환하여 사용할 것이다.
```