---
layout: post
title: "JPA Practice001: #011 - Developing OrderSearch"
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

## 2. 주문 Repository 개발

#### **1) 상품 Service Test 코드**

```java
package jpabook.jpashop.service;

import jpabook.jpashop.domain.item.Book;
import jpabook.jpashop.domain.item.Item;
import jpabook.jpashop.exception.NotEnoughStockException;
import jpabook.jpashop.repository.ItemRepository;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.annotation.Rollback;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.annotation.Transactional;

import static org.junit.Assert.*;

@RunWith(SpringRunner.class)
@SpringBootTest
@Transactional
public class ItemServiceTest {

  @Autowired
  private ItemService itemService;

  @Autowired
  private ItemRepository itemRepository;

  @Test
  @Rollback(value = false)
  public void 상품_등록() throws Exception {
    //given
    Item item = new Book();
    item.setName("book1");
    item.setStockQuantity(5);

    //when
    Long savedId = itemService.save(item);

    //then
    assertEquals(item, itemRepository.findOne(savedId));

  }

}
```

위와 같은 코드로 작성할 경우 아래와 같은 쿼리문이 실행된다.

```sql
insert into item (name,price,stock_quantity,author,isbn,dtype,item_id) values ('book1',0,5,NULL,NULL,'Item - Book',1);
```

상품 등록이 잘 되었다.
<br/>

## 3. 상품 기능 테스트 - 상품 재고 감소

#### **1) 상품 Service Test 코드**

```java
package jpabook.jpashop.service;

import jpabook.jpashop.domain.item.Book;
import jpabook.jpashop.domain.item.Item;
import jpabook.jpashop.exception.NotEnoughStockException;
import jpabook.jpashop.repository.ItemRepository;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.annotation.Rollback;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.annotation.Transactional;

import static org.junit.Assert.*;

@RunWith(SpringRunner.class)
@SpringBootTest
@Transactional
public class ItemServiceTest {

  @Autowired
  private ItemService itemService;

  @Autowired
  private ItemRepository itemRepository;

  @Test(expected = NotEnoughStockException.class)
  public void 재고_감소() throws Exception {
    //given
    Item item = new Book();
    item.setStockQuantity(5);
    Long savedId = itemService.save(item);
    Item findItem = itemService.findOne(savedId);
    int stockToRemove = 3;

    //when
    findItem.removeStock(stockToRemove);
    findItem.removeStock(stockToRemove);

    //then
    fail("예외가 발생해야 한다.");

  }

}
```

위와 같은 코드로 작성할 경우 아래와 같은 쿼리문이 실행된다.

```sql
insert into item (name,price,stock_quantity,author,isbn,dtype,item_id) values (NULL,0,5,NULL,NULL,'Item - Book',1);
update item set name=NULL,price=0,stock_quantity=2,author=NULL,isbn=NULL where item_id=1;
```

update문이 한 번만 실행되었음을 확인할 수 있다.