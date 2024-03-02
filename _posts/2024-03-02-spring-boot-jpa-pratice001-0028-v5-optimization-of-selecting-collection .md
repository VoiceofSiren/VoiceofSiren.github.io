---
layout: post
title: "JPA Practice001: #0028 - [Collection Fetch Optimization V5] Order Lookup - Optimization of Selecting Collection"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. 컬렉션 조회 최적화

- v4에서 발생했던 'N + 1 문제'를 해결해보고자 한다. 

#### **목차**

- 주문 조회 v5: JPA에서 DTO를 직접 조회, 컬렉션 조회 최적화
- 주문 조회 v6: JPA에서 DTO를 직접 조회, 플랫 조회 최적화 (다음 글에서)
- API 개발 정리 (다음 글에서)
<br/>

## 2. (v5) JPA에서 DTO 객체를 바로 조회

#### **1) 주문 RestController 코드**

repository.OrderApiController에 아래의 메서드를 추가한다.

```java
    @GetMapping("/api/v5/orders")
    public List<OrderQueryDTO> ordersV5() {
        return orderQueryRepository.findAllByDtoOptimization();
    }
```

#### **2) OrderQueryRepository 코드**

OrderQueryRepository에 아래의 메서드를 추가한다.

```java
    public List<OrderQueryDTO> findAllByDtoOptimization() {
        List<OrderQueryDTO> orders = findOrders();

        List<Long> orderIds = orders.stream()
        .map(orderQueryDTO -> orderQueryDTO.getOrderId())
        .collect(Collectors.toList());

        List<OrderItemQueryDTO> orderItems = em.createQuery("" +
        "select new jpabook.jpashop.repository.order.query.OrderItemQueryDTO(oi.order.id, i.name, oi.orderPrice, oi.count) " +
        "from OrderItem oi " +
        "join oi.item i " +
        "where oi.order.id in :orderIds", OrderItemQueryDTO.class)
        .setParameter("orderIds", orderIds)
        .getResultList();

        Map<Long, List<OrderItemQueryDTO>> orderItemMap = orderItems.stream()
        .collect(Collectors.groupingBy(orderItemQueryDTO -> orderItemQueryDTO.getOrderId()));

        orders.forEach(orderQueryDTO -> orderQueryDTO.setOrderItems(orderItemMap.get(orderQueryDTO.getOrderId())));

        return orders;
    }
```

- ToOne 관계에 있는 것들은 처음 한 번만 조회하고, 그 이후 ToMany 관계에 있는 것들은 JPQL의 IN 연산자를 통해 한 번에 조회한다.

#### **3) 요청과 응답**

Postman 실행

![IMAGE](/assets/images/spring-boot-jpa-practice001/0027/postman-v4.png)

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
        d1_0.zipcode 
    from
        orders o1_0 
    join
        member m1_0 
            on m1_0.member_id=o1_0.member_id 
    join
        delivery d1_0 
            on d1_0.delivery_id=o1_0.delivery_id
2024-03-02T18:54:17.112+09:00  INFO 7736 --- [nio-8090-exec-1] p6spy                                    : #1709373257112 | took 0ms | statement | connection 7| url jdbc:h2:tcp://localhost/~/jpashop
select o1_0.order_id,m1_0.name,o1_0.order_date,o1_0.order_status,d1_0.city,d1_0.street,d1_0.zipcode from orders o1_0 join member m1_0 on m1_0.member_id=o1_0.member_id join delivery d1_0 on d1_0.delivery_id=o1_0.delivery_id
select o1_0.order_id,m1_0.name,o1_0.order_date,o1_0.order_status,d1_0.city,d1_0.street,d1_0.zipcode from orders o1_0 join member m1_0 on m1_0.member_id=o1_0.member_id join delivery d1_0 on d1_0.delivery_id=o1_0.delivery_id;
Hibernate: 
    select
        oi1_0.order_id,
        i1_0.name,
        oi1_0.order_price,
        oi1_0.count 
    from
        order_item oi1_0 
    join
        item i1_0 
            on i1_0.item_id=oi1_0.item_id 
    where
        oi1_0.order_id in (?, ?)
2024-03-02T18:54:17.153+09:00  INFO 7736 --- [nio-8090-exec-1] p6spy                                    : #1709373257153 | took 1ms | statement | connection 7| url jdbc:h2:tcp://localhost/~/jpashop
select oi1_0.order_id,i1_0.name,oi1_0.order_price,oi1_0.count from order_item oi1_0 join item i1_0 on i1_0.item_id=oi1_0.item_id where oi1_0.order_id in (?,?)
select oi1_0.order_id,i1_0.name,oi1_0.order_price,oi1_0.count from order_item oi1_0 join item i1_0 on i1_0.item_id=oi1_0.item_id where oi1_0.order_id in (1,2);
```