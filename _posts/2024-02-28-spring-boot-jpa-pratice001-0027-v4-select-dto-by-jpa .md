---
layout: post
title: "JPA Practice001: #0027 - [Collection Fetch Optimization V4] Selecting DTOs Directly by JPA"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. 컬렉션 조회 최적화

- 1:N 연관 관계에서 컬렉션 Fetch JOIN을 사용할 경우 Paging이 불가하다는 v3에서의 문제점을 해결해보고자 한다. 

#### **목차**

- 주문 조회 v4: JPA에서 DTO를 직접 조회 
- 주문 조회 v5: JPA에서 DTO를 직접 조회, 컬렉션 조회 최적화 (다음 글에서)
- 주문 조회 v6: JPA에서 DTO를 직접 조회, 플랫 조회 최적화 (다음 글에서)
- API 개발 정리 (다음 글에서)
<br/>

## 2. (v4) JPA에서 DTO 객체를 바로 조회

#### **1) 주문 RestController 코드**

repository.OrderApiController에 아래의 메서드를 추가한다.

```java
    @GetMapping("/api/v4/orders")
    public List<OrderQueryDTO> ordersV4() {
        return orderQueryRepository.findOrderQueryDTOs();
    }
```

repository.order.query 패키지를 생성하고 해당 패키지에 아래의 세 클래스를 추가한다.

#### **2) OrderItemQueryDTO 코드**

```java
package jpabook.jpashop.repository.order.query;

import lombok.Data;

@Data
public class OrderItemQueryDTO {

    private Long orderId;
    private String itemName;
    private int orderPrice;
    private int count;

    public OrderItemQueryDTO(Long orderId, String itemName, int orderPrice, int count) {
        this.orderId = orderId;
        this.itemName = itemName;
        this.orderPrice = orderPrice;
        this.count = count;
    }
}
```

#### **3) OrderQueryDTO 코드**

```java
package jpabook.jpashop.repository.order.query;

import jpabook.jpashop.domain.Address;
import jpabook.jpashop.domain.OrderStatus;
import lombok.Data;

import java.time.LocalDateTime;
import java.util.List;

@Data
public class OrderQueryDTO {
    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;
    private List<OrderItemQueryDTO> orderItems;

    public OrderQueryDTO(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
    }
}
```

#### **4) OrderQueryRepository 코드**

```java
package jpabook.jpashop.repository.order.query;

import jakarta.persistence.EntityManager;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
@RequiredArgsConstructor
public class OrderQueryRepository {

    private final EntityManager em;

    public List<OrderQueryDTO> findOrderQueryDTOs() {
        List<OrderQueryDTO> result = findOrders();

        result.forEach(orderQueryDTO -> {
            List<OrderItemQueryDTO> orderItems = findOrderItems(orderQueryDTO.getOrderId());
            orderQueryDTO.setOrderItems(orderItems);
        });

        return result;
    }

    private List<OrderItemQueryDTO> findOrderItems(Long orderId) {
        return em.createQuery("" +
                        "select new jpabook.jpashop.repository.order.query.OrderItemQueryDTO(oi.order.id, i.name, oi.orderPrice, oi.count) " +
                        "from OrderItem oi " +
                        "join oi.item i " +
                        "where oi.order.id = :orderId", OrderItemQueryDTO.class)
                .setParameter("orderId", orderId)
                .getResultList();
    }

    private List<OrderQueryDTO> findOrders() {
        return em.createQuery("" +
                                "select new jpabook.jpashop.repository.order.query.OrderQueryDTO(o.id, m.name, o.orderDate, o.orderStatus, d.address) " +
                                "from Order o " +
                                "join o.member m " +
                                "join o.delivery d"
                        , OrderQueryDTO.class)
                .getResultList();
    }
}
```

#### **5) 요청과 응답**

Postman 실행

![IMAGE](/assets/images/spring-boot-jpa-practice001/0027/postman-v4.png)

#### **6) 문제점 분석 및 해결 방법**

##### **문제점 분석**
```plaintext
- 'N + 1' 문제가 발생했다.
```
##### **해결 방법**
```plaintext
- 다음 글에서 Collection 조회 최적화를 진행해보고자 한다.
```
<br/>