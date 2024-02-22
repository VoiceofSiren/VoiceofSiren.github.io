---
layout: post
title: "JPA Practice001: #0011 - Developing OrderSearch"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. 주문 도메인 개발 목차

- 구현 기능
  - 상품 주문
  - 주문 내역 조회
  - 주문 취소

- 순서
  - 주문 검색 기능 개발
<br/>

## 2. 주문 검색 기능 개발

#### **1) repository.OrderSearch 클래스**

```java
package jpabook.jpashop.repository;

import jpabook.jpashop.domain.OrderStatus;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class OrderSearch {

  private String memberName;          // 회원 이름
  private OrderStatus orderStatus;    // 주문 상태 [ORDER, CANCEL]
}

```

#### **2) repository.OrderRepository 클래스**

기존의 OrderRepository 클래스 내부에 findAllByString() 메서드를 추가하였다.

```java
package jpabook.jpashop.repository;

import jakarta.persistence.EntityManager;
import jakarta.persistence.TypedQuery;
import jakarta.persistence.criteria.CriteriaBuilder;
import jpabook.jpashop.domain.Order;
import jpabook.jpashop.domain.OrderStatus;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Repository;
import org.springframework.util.StringUtils;

import java.util.List;

@Repository
@RequiredArgsConstructor
public class OrderRepository {

  private final EntityManager em;

  public Long save(Order order) {
    em.persist(order);
    return order.getId();
  }

  public Order findOne(Long orderId) {
    return em.find(Order.class, orderId);
  }

  /**
   * JPQL
   */
  public List<Order> findAllByString(OrderSearch orderSearch) {
    //language=JPAQL
    String jpql = "select o From Order o join o.member m";
    boolean isFirstCondition = true;
    //주문 상태 검색
    if (orderSearch.getOrderStatus() != null) {
      if (isFirstCondition) {
        jpql += " where";
        isFirstCondition = false;
      } else {
        jpql += " and";
      }
      jpql += " o.status = :status";
    }
    //회원 이름 검색
    if (StringUtils.hasText(orderSearch.getMemberName())) {
      if (isFirstCondition) {
        jpql += " where";
        isFirstCondition = false;
      } else {
        jpql += " and";
      }
      jpql += " m.name like :name";
    }
    TypedQuery<Order> query = em.createQuery(jpql, Order.class)
            .setMaxResults(1000); //최대 1000건
    if (orderSearch.getOrderStatus() != null) {
      query = query.setParameter("status", orderSearch.getOrderStatus());
    }
    if (StringUtils.hasText(orderSearch.getMemberName())) {
      query = query.setParameter("name", orderSearch.getMemberName());
    }
    return query.getResultList();
  }

}

```
