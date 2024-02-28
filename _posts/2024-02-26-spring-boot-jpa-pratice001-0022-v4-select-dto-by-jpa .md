---
layout: post
title: "JPA Practice001: #0022 - [Performance Optimization V4] Selecting DTO Directly by JPA"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. JPA에서 DTO 객체를 바로 조회

- v3에서 Fetch JOIN 시 SELECT문에서 원치 않는 필드까지 한꺼번에 SELECT되는 문제가 발생하였다.

- 특정 조회에 의존적인 DTO를 새로 만들어 원하는 필드만 SELECT되도록 성능을 개선시키고자 한다.

#### **1) 주문 RestController 코드**

api.OrderSimpleApiController에 아래의 코드를 추가한다.

```java
    @GetMapping("/api/v4/simple-orders")
    public List<OrderSimpleQueryDTO> ordersV4() {
        return orderSimpleQueryRepository.findOrderDtos();
        }
```

#### **2) 주문 조회를 위한 별도의 DTO 생성

repository.order.simplequery 패키지를 생성한다.

해당 패키지에 아래의 OrderSimpleQueryDTO 클래스를 정의한다.

```java
package jpabook.jpashop.repository.order.simplequery;

import jpabook.jpashop.domain.Address;
import jpabook.jpashop.domain.OrderStatus;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;
@Data
@NoArgsConstructor
public class OrderSimpleQueryDTO {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;

    public OrderSimpleQueryDTO(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
    }
}
```

#### **3) DTO 객체 조회를 위한 별도의 Repository 생성**

repository.order.simplequery 패키지에 아래의 OrderSimpleQueryRepository 레퍼지토리를 정의한다.

반드시 new 키워드를 사용해야 하며, 모든 패키지의 이름을 기재해야 한다.

```java
package jpabook.jpashop.repository.order.simplequery;

import jakarta.persistence.EntityManager;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
@RequiredArgsConstructor
public class OrderSimpleQueryRepository {

    private final EntityManager em;

    public List<OrderSimpleQueryDTO> findOrderDtos() {
        return em.createQuery(
                        "select new jpabook.jpashop.repository.order.simplequery.OrderSimpleQueryDTO(o.id, m.name, o.orderDate, o.orderStatus, d.address) " +
                                "from Order o " +
                                "join o.member m " +
                                "join o.delivery d ", OrderSimpleQueryDTO.class)
                .getResultList();
    }
}
```

#### **4) 응답과 요청**

Postman 실행

![IMAGE](/assets/images/spring-boot-jpa-practice001/0022/postman-v4.png)

log 기록

![IMAGE](/assets/images/spring-boot-jpa-practice001/0022/logs-of-selecting-dto.png)

- 조회하고자 하는 필드들만 잘 SELECT 되었음을 확인할 수 있다.