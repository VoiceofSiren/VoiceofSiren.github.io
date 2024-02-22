---
layout: post
title: "JPA Practice001: #0002 - Defining Domain classes"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. Entity 설계 시 주의점

#### **1) Entity로의 setter() 메서드 사용을 지양하자.**

모든 Entity에 setter() 메서드를 사용할 경우, 변경 포인트가 너무 많기 때문에 유지보수에 어려움이 발생한다.

setter() 메서드를 아예 정의하지 않거나, setter() 메서드의 접근 제한자를 public에서 private으로 바꿔 사용할 것이다.

#### **2) setter() 메서드 대신 연관 관계 매핑 메서드를 사용하자.**

setter() 메서드의 대안으로, 연관 관계에 있는 두 Entity 객체에 대하여 양방향으로 각각의 필드에 상대쪽 Entity 객체를 할당해주는 별도의 연관 관계 매핑 메서드를 사용할 것이다.

#### **3) 모든 연관 관계는 지연 로딩으로 설정하자.**

즉시 로딩을 사용할 경우, 다른 Entity와 연관 관계에 있는 객체를 영속화할 때 연쇄적으로 다른 Entity들까지 한꺼번에 참조될 수 있다.

어떤 SQL이 실행될지에 대한 예측과 추적이 어려우며, JPQL을 실행할 때 'N + 1 문제'가 자주 발생한다.

#### **4) EnumType을 ORDINAL이 아닌 STRING으로 설정하자.**

EnumType을 ORDINAL로 설정한 상태에서 enum 내부 데이터에 추가 및 삭제 등의 변경이 발생하면, 바뀐 순서를 모든 소스 코드에 적용해야 하는 불상사가 발생한다.

#### **5) 컬렉션은 필드에서 초기화하자.**

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
<br/>


## 2. Entity 설계

#### 1) 회원 (Entity)
```java
package jpabook.jpashop.domain;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;

import java.util.ArrayList;
import java.util.List;

@Entity
@Getter
@Setter
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    private String name;

    @Embedded
    private Address address;

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();

}

```

#### 2) 주문 (Entity)
```java
package jpabook.jpashop.domain;

import jakarta.persistence.*;

import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "orders")
public class Order {

    @Id
    @GeneratedValue
    @Column(name = "order_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> orderItems = new ArrayList<>();

    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "delivery_id")
    private Delivery delivery;

    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus orderStatus;

    //==연관 관계 메서드==//
    public void setMember(Member member) {
        this.member = member;
        member.getOrders().add(this);
    }

    public void addOrderItem(OrderItem orderItem) {
        orderItems.add(orderItem);
        orderItem.setOrder(this);
    }

    public void setDelivery(Delivery delivery) {
        this.delivery = delivery;
        delivery.setOrder(this);
    }
}
```

#### 3) 주문 상태 (enum)
```java
package jpabook.jpashop.domain;

public enum OrderStatus {
    ORDER, CANCEL
}

```

#### 4) 주문 상품 (Entity)
```java
package jpabook.jpashop.domain;

import jakarta.persistence.*;
import jpabook.jpashop.domain.item.Item;
import lombok.Getter;
import lombok.Setter;

@Entity
@Getter
@Setter
public class OrderItem {

    @Id
    @GeneratedValue
    @Column(name = "order_item_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    private Order order;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "item_id")
    private Item item;

    private int orderPrice;

    private int count;
}
```

#### 5) 상품 (Entity)
```java
package jpabook.jpashop.domain.item;

import jakarta.persistence.*;
import jpabook.jpashop.domain.Category;
import lombok.Getter;
import lombok.Setter;

import java.util.ArrayList;
import java.util.List;

@Entity
@Getter
@Setter
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "dtype")
public abstract class Item {

    @Id
    @GeneratedValue
    @Column(name = "item_id")
    private Long id;

    private String name;

    private int price;

    private int stockQuantity;

    @ManyToMany(mappedBy = "items")
    private List<Category> categories = new ArrayList<>();
}
```

#### 6) 상품 - 도서 (Entity)
```java
package jpabook.jpashop.domain.item;

import jakarta.persistence.DiscriminatorValue;
import jakarta.persistence.Entity;
import lombok.Getter;
import lombok.Setter;

@Entity
@Getter
@Setter
@DiscriminatorValue("Item - Book")
public class Book extends Item{

    private String author;
    private String isbn;
}
```

#### 7) 상품 - 음반 (Entity)
```java
package jpabook.jpashop.domain.item;

import jakarta.persistence.DiscriminatorValue;
import jakarta.persistence.Entity;
import lombok.Getter;
import lombok.Setter;

@Entity
@Getter
@Setter
@DiscriminatorValue("Item - Album")
public class Album extends Item{

    private String artist;
    private String etc;
}

```

#### 8) 상품 - 영화 (Entity)
```java
package jpabook.jpashop.domain.item;

import jakarta.persistence.DiscriminatorValue;
import jakarta.persistence.Entity;
import lombok.Getter;
import lombok.Setter;

@Entity
@Getter
@Setter
@DiscriminatorValue("Item - Movie")
public class Movie extends Item{

    private String director;
    private String actor;
}

```

#### 9) 배송 (Entity)
```java
package jpabook.jpashop.domain;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;

@Entity
@Getter
@Setter
public class Delivery {

    @Id
    @GeneratedValue
    @Column(name = "delivery_id")
    private Long id;

    @OneToOne(mappedBy = "delivery")
    private Order order;

    @Embedded
    private Address address;

    @Enumerated(EnumType.STRING)
    private DeliveryStatus status;
}
```

#### 10) 배송 상태 (Enum)
```java
package jpabook.jpashop.domain;

public enum DeliveryStatus {
    READY, COMPLETE
}
```

#### 11) 카테고리 (Entity)
```java
package jpabook.jpashop.domain;

import jakarta.persistence.*;
//import jpabook.jpashop.domain.item.CategoryItem;
import jpabook.jpashop.domain.item.Item;
import lombok.Getter;
import lombok.Setter;

import java.util.ArrayList;
import java.util.List;

@Entity
@Getter
@Setter
public class Category {

    @Id
    @GeneratedValue
    @Column(name = "category_id")
    private Long id;

    private String name;

    @ManyToMany
    @JoinTable(name = "category_item",
               joinColumns = {@JoinColumn(name = "category_id")},
               inverseJoinColumns = {@JoinColumn(name = "item_id")})
    private List<Item> items = new ArrayList<>();

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "parent_id")
    private Category parent;

    @OneToMany(mappedBy = "parent")
    private List<Category> children = new ArrayList<>();

    //==연관 관계 메서드==//
    public void addChildCategory(Category child) {
        this.children.add(child);
        child.setParent(this);
    }

}
```

#### 12) 주소 (Embeddable)
```java
package jpabook.jpashop.domain;

import jakarta.persistence.Embeddable;
import lombok.Getter;

@Embeddable
@Getter
public class Address {

    private String city;
    private String street;
    private String zipcode;

    /**
     *  Default Constructor
     *  - necessary: because of the reflection of JAVA
     */
    protected Address() {

    }

    public Address(String city, String street, String zipcode) {
        this.city = city;
        this.street = street;
        this.zipcode = zipcode;
    }
}

```