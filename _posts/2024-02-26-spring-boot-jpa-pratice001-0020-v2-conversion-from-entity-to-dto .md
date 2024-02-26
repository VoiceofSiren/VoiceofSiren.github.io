---
layout: post
title: "JPA Practice001: #0020 - [Performance Optimization V2] Conversion from Entity to DTO"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. Entity를 DTO로 변환

#### **1) 주문 RestController 코드**

api.OrderSimpleApiController에 아래의 코드를 추가한다.

```java
    @GetMapping("/api/v2/simple-orders")
    public List<SimpleOrderDTO> ordersV2() {
        List<Order> orders = orderRepository.findAllByString(new OrderSearch());
        List<SimpleOrderDTO> result = orders.stream()
                .map(order -> new SimpleOrderDTO(order))
                .collect(Collectors.toList());
        return result;
    }
    
    @Data
    static class SimpleOrderDTO {
        private Long orderId;
        private String name;
        private LocalDateTime orderDate;
        private OrderStatus orderStatus;
        private Address address;

        public SimpleOrderDTO(Order order) {
            orderId = order.getId();
            name = order.getMember().getName();         // LAZY 초기화
            orderDate = order.getOrderDate();
            orderStatus = order.getOrderStatus();
            address = order.getDelivery().getAddress(); // LAZY 초기화
        }
    }
```

#### **2) 응답과 요청**

Postman 실행

![IMAGE](/assets/images/spring-boot-jpa-practice001/0020/postman-v2-orders.png)

log 기록 1 - SELECT ORDERS

![IMAGE](/assets/images/spring-boot-jpa-practice001/0020/jpa-bad-select-1.png)

log 기록 2 - SELECT MEMBER

![IMAGE](/assets/images/spring-boot-jpa-practice001/0020/jpa-bad-select-2.png)

log 기록 3 - SELECT DELIVERY

![IMAGE](/assets/images/spring-boot-jpa-practice001/0020/jpa-bad-select-3.png)

log 기록 4 - SELECT MEMBER

![IMAGE](/assets/images/spring-boot-jpa-practice001/0020/jpa-bad-select-4.png)

log 기록 5 - SELECT DELIVERY

![IMAGE](/assets/images/spring-boot-jpa-practice001/0020/jpa-bad-select-5.png)

#### **3) 문제점 분석 및 해결 방법**

##### **문제점 분석**
```plaintext
- Order 객체들을 조회하는 SELECT문은 처음 한 번만 실행되었다.
- 첫 번째 Order 객체에 대하여 Member를 조회하는 SELECT문과 Delivery를 조회하는 SELECT문이 한 번씩 실행되었다.
- 두 번째 Order 객체에 대하여 Member를 조회하는 SELECT문과 Delivery를 조회하는 SELECT문이 한 번씩 실행되었다.
- 즉, N개의 Order 객체를 조회할 때 각각의 Order 객체에 대하여 연관 관계에 있는 필드 M개를 조회한다고 가정할 때,
  최악의 경우 SELECT문이 총 (1 + NM)번 실행된다.
- 만약 10000개의 Order 객체에 대하여 최악의 경우 각각 프록시 필드 초기화가 10번씩 일어난다고 가정하면
  SELECT문이 총 100001번 실행되는 성능 이슈가 발생한다.
```
##### **해결 방법**
```plaintext
- Fetch JOIN을 사용하면 된다.
```
