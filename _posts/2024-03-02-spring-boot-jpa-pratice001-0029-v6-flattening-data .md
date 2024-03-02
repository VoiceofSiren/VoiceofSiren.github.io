---
layout: post
title: "JPA Practice001: #0029 - [Collection Fetch Optimization V6] Order Lookup - Optimization of Selecting Collection"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. 컬렉션 조회 최적화

- v5에서 쿼리 실행 횟수가 2회였는데, v6에서는 쿼리 실행 횟수를 1회로 줄여보고자 한다.

#### **목차**

- 주문 조회 v6: JPA에서 DTO를 직접 조회, 플랫 조회 최적화
- API 개발 정리 (다음 글에서)
<br/>

## 2. (v5) JPA에서 DTO 객체를 바로 조회

#### **1) 주문 RestController 코드**

repository.OrderApiController에 아래의 메서드를 추가한다.

```java
    @GetMapping("/api/v6/orders")
public List<OrderFlatDTO> ordersV6() {
        return orderQueryRepository.findAllDtoByFlattening();
        }
```

#### **2) OrderFlatDTO 코드**

repository.order.query 패키지에 아래의 클래스를 추가한다.

```java
package jpabook.jpashop.repository.order.query;

import jpabook.jpashop.domain.Address;
import jpabook.jpashop.domain.OrderStatus;
import lombok.Data;

import java.time.LocalDateTime;

@Data
public class OrderFlatDTO {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;

    private String itemName;
    private int orderPrice;
    private int count;

    public OrderFlatDTO(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address, String itemName, int orderPrice, int count) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
        this.itemName = itemName;
        this.orderPrice = orderPrice;
        this.count = count;
    }
}
```


#### **3) OrderQueryRepository 코드**

OrderQueryRepository에 아래의 메서드를 추가한다.

```java
    public List<OrderFlatDTO> findAllDtoByFlattening() {
        return em.createQuery("" +
        "select new jpabook.jpashop.repository.order.query.OrderFlatDTO(o.id, m.name, o.orderDate, o.orderStatus, d.address, i.name, oi.orderPrice, oi.count) " +
        "from Order o " +
        "join o.member m " +
        "join o.delivery d " +
        "join o.orderItems oi " +
        "join oi.item i", OrderFlatDTO.class)
        .getResultList();
    }
```

#### **3) 요청과 응답**

Postman 실행

![IMAGE](/assets/images/spring-boot-jpa-practice001/0029/postman-v6.png)

JPA log 기록

```plaintext
Hibernate: 
    select
        o1_0.order_id,
        m1_0.name,
        o1_0.order_date,
        o1_0.order_status,
        d1_0.city,
        d1_0.street,
        d1_0.zipcode,
        i1_0.name,
        oi1_0.order_price,
        oi1_0.count 
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
            on i1_0.item_id=oi1_0.item_id
2024-03-02T19:29:53.551+09:00  INFO 67380 --- [nio-8090-exec-1] p6spy                                    : #1709375393551 | took 1ms | statement | connection 7| url jdbc:h2:tcp://localhost/~/jpashop
select o1_0.order_id,m1_0.name,o1_0.order_date,o1_0.order_status,d1_0.city,d1_0.street,d1_0.zipcode,i1_0.name,oi1_0.order_price,oi1_0.count from orders o1_0 join member m1_0 on m1_0.member_id=o1_0.member_id join delivery d1_0 on d1_0.delivery_id=o1_0.delivery_id join order_item oi1_0 on o1_0.order_id=oi1_0.order_id join item i1_0 on i1_0.item_id=oi1_0.item_id
select o1_0.order_id,m1_0.name,o1_0.order_date,o1_0.order_status,d1_0.city,d1_0.street,d1_0.zipcode,i1_0.name,oi1_0.order_price,oi1_0.count from orders o1_0 join member m1_0 on m1_0.member_id=o1_0.member_id join delivery d1_0 on d1_0.delivery_id=o1_0.delivery_id join order_item oi1_0 on o1_0.order_id=oi1_0.order_id join item i1_0 on i1_0.item_id=oi1_0.item_id;
```