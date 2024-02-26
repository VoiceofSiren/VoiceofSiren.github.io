---
layout: post
title: "JPA Practice001: #0021 - [Performance Optimization V3] Fetch JOIN"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. Fetch JOIN

- JPA의 Fetch JOIN 기능을 사용하여 성능 최적화 v2에 있었던 'N + 1 문제'를 해결하고자 한다.

#### **1) 주문 RestController 코드**

api.OrderSimpleApiController에 아래의 코드를 추가한다.

```java
    @GetMapping("/api/v3/simple-orders")
public List<SimpleOrderDTO> ordersV3() {
        List<Order> orders = orderRepository.findAllWithMemberDelivery();
        return orders.stream()
        .map(order -> new SimpleOrderDTO(order))
        .collect(Collectors.toList());
        }
```

#### **2) 주문 Repository 코드**

repository.OrderRepository에 아래의 코드를 추가한다.

```java
    /**
     * Fetch JOIN
     *      - Proxy 객체가 아닌 진짜 Entity를 채워넣어서 조회한다.
     */
    public List<Order> findAllWithMemberDelivery() {
        return em.createQuery(
                "select o from Order o " +
                            "join fetch o.member m " +
                            "join fetch o.delivery d ", Order.class)
            .getResultList();
    }
```

#### **3) 응답과 요청**

Postman 실행

![IMAGE](/assets/images/spring-boot-jpa-practice001/0021/postman-v3.png)

log 기록

![IMAGE](/assets/images/spring-boot-jpa-practice001/0021/logs-of-fetch-join.png)

- SELECT문이 한 번만 실행되었음을 확인할 수 있다.