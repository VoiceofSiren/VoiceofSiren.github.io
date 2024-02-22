---
layout: post
title: "JPA Practice001: #0001 - Starting Out with Spring boot JPA practice001"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. 요구사항 분석

#### 기능 목록

- 회원 기능
    - 회원 등록
    - 회원 조회
- 상품 기능
    - 상품 등록
    - 상품 수정
    - 상품 조회
- 주문 기능
    - 상품 주문
    - 주문 내역 조회
    - 주문 취소
- 기타 요구 사항
    - 상품에는 재고 관리가 필요하다.
    - 상품의 종류에는 도서, 음반, 영화가 있다.
    - 상품을 카테고리로 분류할 수 있다.
    - 상품 주문 시 배송 정보를 입력할 수 있다.

## 2. 도메인 모델과 테이블 설계
![IMAGE](/assets/images/spring-boot-jpa-practice001/0001/domain-model.png)
1. 회원, 주문, 상품의 관계: 회원은 여러 상품을 주문할 수 있다. 그리고 한 번 주문할 때 여러 상품을 선택할 수 있으므로
   주문과 상품은 다대다 관계다. 하지만 이런 다대다 관계는 관계형 데이터베이스는 물론이고 엔티티에서도 거의 사용하
   지 않는다. 따라서 그림처럼 주문상품이라는 엔티티를 추가해서 다대다 관계를 일대다, 다대일 관계로 풀어냈다.

2. 상품 분류: 상품은 도서, 음반, 영화로 구분되는데 상품이라는 공통 속성을 사용하므로 상속 구조로 표현했다.

<br/>

![IMAGE](/assets/images/spring-boot-jpa-practice001/0001/member-entity-analysis.png)
1. 회원 (Member): 이름과 Embedded 타입인 주소 (Address), 그리고 주문 (orders) 리스트를 가진다.
2. 주문 (Order): 한 번 주문시 여러 상품을 주문할 수 있으므로 주문과 주문상품 (OrderItem)은 일대다 관계다. 주문은
   상품을 주문한 회원과 배송 정보, 주문 날짜, 주문 상태(status)를 가지고 있다. 주문 상태는 열거형을 사용했는데 주
   문 (ORDER), 취소 (CANCEL)를 표현할 수 있다.
3. 주문상품 (OrderItem): 주문한 상품 정보와 주문 금액 (orderPrice), 주문 수량 (count) 정보를 가지고 있다. (보
   통 OrderLine , LineItem 으로 많이 표현한다.)
4. 상품 (Item): 이름, 가격, 재고 수량 (stockQuantity)을 가지고 있다. 상품을 주문하면 재고 수량이 줄어든다. 상품의
   종류로는 도서, 음반, 영화가 있는데 각각이 사용하는 속성은 조금씩 다르다.
5. 배송 (Delivery): 주문시 하나의 배송 정보를 생성한다. 주문과 배송은 일대일 관계다.
6. 카테고리 (Category): 상품과 N:M 관계를 맺는다. parent, child로 부모, 자식 카테고리를 연결한다.
7. 주소 (Address): 값 타입 (Embedded 타입)이다. 회원과 배송 (Delivery)에서 사용한다.

<br/>

![IMAGE](/assets/images/spring-boot-jpa-practice001/0001/member-table-analysis.png)
1. MEMBER: 회원 엔티티의 Address 임베디드 타입 정보가 회원 테이블에 그대로 들어갔다. 이것은 DELIVERY 테
   이블도 마찬가지다.
2. ITEM: 앨범, 도서, 영화 타입을 통합해서 하나의 테이블로 만들었다. DTYPE 컬럼으로 타입을 구분한다.

<br/>

## 3. 연관관계 매핑 분석

#### **1. 회원과 주문**

1:N, N:1의 양방향 관계이다. 따라서 연관 관계의 주인을 정해야 하는데, 외래 키가 있는 주문을 연관
관계의 주인으로 정하는 것이 좋다. 그러므로 Order.member를 ORDERS.MEMBER_ID 외래 키와 매핑한다.

#### **2. 주문 상품과 주문**

N:1 단방향 관계이다. 외래 키가 주문 상품에 있으므로 주문 상품이 연관관 계의 주인이다. 그러므로
OrderItem.order를 ORDER_ITEM.ORDER_ID 외래 키와 매핑한다.

#### **3. 주문 상품과 상품**

N:1 단방향 관계다. OrderItem.item을 ORDER_ITEM.ITEM_ID 외래 키와 매핑한다.

#### **4. 주문과 배송**

1:1 양방향 관계이다. Order.delivery 를 ORDERS.DELIVERY_ID 외래 키와 매핑한다.

#### **5. 카테고리와 상품**

@ManyToMany를 사용해서 매핑한다. (실무에서 @ManyToMany는 사용하지 말자. 여기서는 다대
다 관계를 예제로 보여주기 위해 추가했을 뿐이다.)