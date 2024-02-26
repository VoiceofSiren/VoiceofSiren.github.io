---
layout: post
title: "JPA Practice001: #0019 - (v1) Lazy Loading Strategy and Query Optimization"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. 지연 로딩과 조회 성능 최적화

- 주문, 배송 정보 및 회원을 조회하는 API를 개발하고자 한다.
- 지연 로딩으로 인해 발생하는 성능 문제를 단계적으로 해결하고자 한다.
<br/>

## 2. (v1) 간단한 주문 조회: Entity를 직접 노출

#### **1) 주문 RestController 코드**

api.OrderSimpleApiController

```java
package jpabook.jpashop.api;

import jpabook.jpashop.domain.Order;
import jpabook.jpashop.repository.OrderRepository;
import jpabook.jpashop.repository.OrderSearch;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;


/**
 * xToOne
 * Order
 * Order -> Member (ManyToOne)
 * Order -> Delivery (OneToOne)
 */
@RestController
@RequiredArgsConstructor
public class OrderSimpleApiController {

    private final OrderRepository orderRepository;

    @GetMapping("/api/v1/simple-orders")
    public List<Order> ordersV1() {
        List<Order> allOrders = orderRepository.findAllByString(new OrderSearch());
        return allOrders;
    }
}

```

#### **2) 응답과 요청**

Postman 실행

![IMAGE](/assets/images/spring-boot-jpa-practice001/0019/failed.png)

log 기록

![IMAGE](/assets/images/spring-boot-jpa-practice001/0019/logs-of-failure.png)

#### **3) 에러 분석 및 해결 방법**

##### **에러 분석**
```plaintext
- Order 객체를 조회하려니 Order와 연관 관계에 있는 다른 Entity를 참조하고, 그 Entity에서 다시 Order로 참조하는 무한 루프가 발생한다.
```
##### **해결 방법**
```plaintext
- Order에 대한 모든 참조 필드에 @JsonIgnore annotation을 추가한다.
```
#### **4) 응답과 요청**

Postman 실행

![IMAGE](/assets/images/spring-boot-jpa-practice001/0019/proxy-related-errors.png)

#### **5) 에러 분석 및 해결 방법**

##### **에러 분석**
```plaintext
- Order와 연관 관계에 있는 Member를 참조하고 있는 member 필드가 초기화되어 있지 않다.
- 해당 member 필드의 fetch 전략이 Lazy로 되어 있기 때문에 Order를 조회할 때 Member를 DB에서 조회하지 않고
  ByteBuddyInterceptor() 라이브러리를 이용하여 Order의 member 필드를 Proxy 객체로 초기화한다.
```

##### **해결 방법**
```plaintext
- Hibernate5 모듈을 추가해야 한다.
```

build.gradle

- dependencies에 아래의 코드를 추가한다.
```plaintext
implementation 'com.fasterxml.jackson.datatype:jackson-datatype-hibernate5-jakarta'
```

JpashopApplication

- JpashopApplication 클래스 내부에 아래의 코드를 추가한다.
```java
    @Bean
	Hibernate5JakartaModule hibernate5Module() {
        return new Hibernate5JakartaModule();
    }
```

#### **6) 응답과 요청**

![IMAGE](/assets/images/spring-boot-jpa-practice001/0019/order-entities.png)