---
layout: post
title: "Define Domain classes in Spring boot JPA practice001"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. Entity 설계 시 주의점

### 1) Entity로의 setter() 메서드 사용을 지양하자.

모든 Entity에 setter() 메서드를 사용할 경우, 변경 포인트가 너무 많기 때문에 유지보수에 어려움이 발생한다.

setter() 메서드를 아예 정의하지 않거나, setter() 메서드의 접근 제한자를 public에서 private으로 바꿔 사용할 것이다.

### 2) setter() 메서드 대신 연관 관계 매핑 메서드를 사용하자.

setter() 메서드의 대안으로, 연관 관계에 있는 두 Entity 객체에 대하여 양방향으로 각각의 필드에 상대쪽 Entity 객체를 할당해주는 별도의 연관 관계 매핑 메서드를 사용할 것이다.

### 3) 모든 연관 관계는 지연 로딩으로 설정하자.

즉시 로딩을 사용할 경우, 다른 Entity와 연관 관계에 있는 객체를 영속화할 때 연쇄적으로 다른 Entity들까지 한꺼번에 참조될 수 있다.

어떤 SQL이 실행될지에 대한 예측과 추적이 어려우며, JPQL을 실행할 때 'N + 1 문제'가 자주 발생한다.

### 4) 컬렉션은 필드에서 초기화하자.

NullPointerException 문제로부터 안전해질 수 있으며,

Hibernate가 Entity를 영속화할 때 Collection을 감싸서 내장 Collection으로 변환하므로,
해당 Entity와 연관되어 있는 다른 객체를 담는 Collection을 잘못 생성하면 내부 메커니즘에 문제가 발생할 수 있다.


```java
Member member = new Member();
System.out.println(member.getOrders().getClass());
em.persist(member);
System.out.println(member.getOrders().getClass());
```
```plaintext
//출력 결과
class java.util.ArrayList
class org.hibernate.collection.internal.PersistentBag
```



## 2. Entity 설계

