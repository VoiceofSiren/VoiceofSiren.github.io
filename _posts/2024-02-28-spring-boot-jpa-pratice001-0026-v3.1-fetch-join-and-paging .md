---
layout: post
title: "JPA Practice001: #0026 - [Collection Fetch Optimization V3.1] Collection Fetch JOIN and Paging"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. 컬렉션 조회 최적화

- 1:N 연관 관계에서 컬렉션 Fetch JOIN을 사용할 경우 Paging이 불가하다는 v3에서의 문제점을 해결해보고자 한다. 

#### **목차**

- Paging과 한계 돌파
- 주문 조회 v4: JPA에서 DTO를 직접 조회 (다음 글에서)
- 주문 조회 v5: JPA에서 DTO를 직접 조회, 컬렉션 조회 최적화 (다음 글에서)
- 주문 조회 v5: JPA에서 DTO를 직접 조회, 플랫 조회 최적화 (다음 글에서)
- API 개발 정리 (다음 글에서)
<br/>

## 2. (v3.1) Paging 한계 돌파

#### **1) 주문 RestController 코드**

repository.OrderApiController에 아래의 메서드를 추가한다.

```java
    @GetMapping("/api/v3.1/orders")
    public List<OrderDTO> ordersV3_paging() {
        List<Order> orders = orderRepository.findAllWithMemberDelivery();
        return orders.stream()
        .map(order -> new OrderDTO(order))
        .collect(Collectors.toList());
    }
```

#### **2) 결과 분석**

- 원하는 정보가 잘 출력되지만 OrderItem과 Item에 대하여 Fetch JOIN을 사용하지 않아 역시나 'N + 1' 문제가 발생하였다.

#### **3) 주문 RestController 코드**

api.OrderApiController의 ordersV3_paging메서드를 아래와 같이 수정한다.

```java
    @GetMapping("/api/v3.1/orders")
    public List<OrderDTO> ordersV3_paging(@RequestParam(value = "offset", defaultValue = "0") int offset,
                                          @RequestParam(value = "limit", defaultValue = "100") int limit) {
        List<Order> orders = orderRepository.findAllWithMemberDelivery(offset, limit);
        return orders.stream()
        .map(order -> new OrderDTO(order))
        .collect(Collectors.toList());
    }
```

#### **4) 주문 Repository 코드**

repository.OrderRepository에 있는 findAllWithMemberDelivery() 메서드를 Overloading한다.

```java
    public List<Order> findAllWithMemberDelivery(int offset, int limit) {
        return em.createQuery(
                        "select o from Order o " +
                                "join fetch o.member m " +
                                "join fetch o.delivery d ", Order.class)
                .setFirstResult(offset)
                .setMaxResults(limit)
                .getResultList();
    }
```

#### **5) 요청과 응답**

Postman 실행

![IMAGE](/assets/images/spring-boot-jpa-practice001/0026/postman-v3_1-overloading.png)

#### **6) 문제점 분석 및 해결 방법**

##### **문제점 분석**
```plaintext
- offset이 1이고 JPA의 Paging 인덱스는 0부터 시작하므로 2개의 Order 중 index가 0인 것을 제외한 나머지 한 개만 성공적으로 조회되었다.
- 다만, 아직 'N + 1' 문제가 해결되지 않았다.
```
##### **해결 방법**
```plaintext
- application.yml에서 spring.jpa.properties.hibernate.default_batch_fetch_size의 값을 100으로 지정한다.
```
<br/>

## 3. Batch Fetch Size - Global Setting

#### **1) application.yml**

```yaml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/jpashop
    username: sa
    password:
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
        show_sql: true
        format_sql: true
        default_batch_fetch_size: 100

  thymeleaf:
    prefix: classpath:/templates/
    suffix: .html

server:
  port: 8090

logging:
  level:
    org:
      hibernate:
        sql: debug
        type: trace

```

#### **2) 요청과 응답**

Postman 실행

![IMAGE](/assets/images/spring-boot-jpa-practice001/0026/postman-v3_1-batch-size.png)

```json
[
  {
    "orderId": 1,
    "name": "userA",
    "orderDate": "2024-02-28T12:35:22.554611",
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
    "orderDate": "2024-02-28T12:35:22.605709",
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
Hibernate:
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
    o1_0.order_status
from
    orders o1_0
        join
    member m1_0
    on m1_0.member_id=o1_0.member_id
        join
    delivery d1_0
    on d1_0.delivery_id=o1_0.delivery_id
offset
    ? rows
    fetch
    first ? rows only
    2024-02-28T12:35:57.289+09:00  INFO 28284 --- [nio-8090-exec-1] p6spy                                    : #1709091357289 | took 0ms | statement | connection 7| url jdbc:h2:tcp://localhost/~/jpashop
select o1_0.order_id,d1_0.delivery_id,d1_0.city,d1_0.street,d1_0.zipcode,d1_0.status,m1_0.member_id,m1_0.city,m1_0.street,m1_0.zipcode,m1_0.name,o1_0.order_date,o1_0.order_status from orders o1_0 join member m1_0 on m1_0.member_id=o1_0.member_id join delivery d1_0 on d1_0.delivery_id=o1_0.delivery_id offset ? rows fetch first ? rows only
select o1_0.order_id,d1_0.delivery_id,d1_0.city,d1_0.street,d1_0.zipcode,d1_0.status,m1_0.member_id,m1_0.city,m1_0.street,m1_0.zipcode,m1_0.name,o1_0.order_date,o1_0.order_status from orders o1_0 join member m1_0 on m1_0.member_id=o1_0.member_id join delivery d1_0 on d1_0.delivery_id=o1_0.delivery_id offset 0 rows fetch first 100 rows only;
Hibernate:
select
    oi1_0.order_id,
    oi1_0.order_item_id,
    oi1_0.count,
    oi1_0.item_id,
    oi1_0.order_price
from
    order_item oi1_0
where
        oi1_0.order_id in (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
    2024-02-28T12:35:57.307+09:00  INFO 28284 --- [nio-8090-exec-1] p6spy                                    : #1709091357307 | took 0ms | statement | connection 7| url jdbc:h2:tcp://localhost/~/jpashop
select oi1_0.order_id,oi1_0.order_item_id,oi1_0.count,oi1_0.item_id,oi1_0.order_price from order_item oi1_0 where oi1_0.order_id in (?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?)
select oi1_0.order_id,oi1_0.order_item_id,oi1_0.count,oi1_0.item_id,oi1_0.order_price from order_item oi1_0 where oi1_0.order_id in (1,2,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL);
2024-02-28T12:35:57.314+09:00 DEBUG 28284 --- [nio-8090-exec-1] o.h.sql.results.internal.ResultsHelper   : Collection fully initialized: [jpabook.jpashop.domain.Order.orderItems#1]
2024-02-28T12:35:57.314+09:00 DEBUG 28284 --- [nio-8090-exec-1] o.h.sql.results.internal.ResultsHelper   : Collection fully initialized: [jpabook.jpashop.domain.Order.orderItems#2]
Hibernate:
select
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
    i1_0.director
from
    item i1_0
where
        i1_0.item_id in (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
    2024-02-28T12:35:57.319+09:00  INFO 28284 --- [nio-8090-exec-1] p6spy                                    : #1709091357319 | took 0ms | statement | connection 7| url jdbc:h2:tcp://localhost/~/jpashop
select i1_0.item_id,i1_0.dtype,i1_0.name,i1_0.price,i1_0.stock_quantity,i1_0.artist,i1_0.etc,i1_0.author,i1_0.isbn,i1_0.actor,i1_0.director from item i1_0 where i1_0.item_id in (?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?,?)
select i1_0.item_id,i1_0.dtype,i1_0.name,i1_0.price,i1_0.stock_quantity,i1_0.artist,i1_0.etc,i1_0.author,i1_0.isbn,i1_0.actor,i1_0.director from item i1_0 where i1_0.item_id in (1,2,3,4,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL);

```

- SELECT문이 총 (1 + 1 + 1) 번 실행되었음을 확인할 수 있다.
- 각 Entity의 Primary key를 기준으로 SQL의 IN 연산자를 사용하여 batch size (==100) 개수만큼 끊어서 한꺼번에 조회한다.
