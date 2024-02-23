---
layout: post
title: "JPA Practice001: #0015 - Updating Item"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. 웹 계층 개발 목차

- 상품 수정
- 상품 주문 (다음 글에서)
- 주문 목록 검색, 취소 (다음 글에서)

## 2. 상품 수정

controller.BookForm

##### **1) Book Form 객체**

```java
package jpabook.jpashop.controller;

import jakarta.validation.constraints.NotEmpty;
import jpabook.jpashop.domain.item.Book;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class BookForm {

    private Long id;

    @NotEmpty(message = "상품명 입력은 필수입니다.")
    private String name;
    private int price;
    private int stockQuantity;

    private String author;
    private String isbn;

    public static Book BookForm_to_Book(BookForm bookForm) {
        Book book = new Book();
        book.setName(bookForm.getName());
        book.setPrice(bookForm.getPrice());
        book.setStockQuantity(bookForm.getStockQuantity());
        book.setAuthor(bookForm.getAuthor());
        book.setIsbn(bookForm.getIsbn());
        return book;
    }
}
```

##### **2) 상품 Update DTO**

dto.ItemUpdateDTO 객체를 추가한다.

```java
package jpabook.jpashop.dto;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class ItemUpdateDTO {

    private String name;
    private int price;
    private int stockQuantity;
    private String author;
    private String isbn;
}
```

#### **3) 상품 Service**

service.ItemService에 아래의 메서드를 추가한다.

```java
    @Transactional
    public void updateBook(Long itemId, ItemUpdateDTO itemUpdateDTO) {

        // 영속 상태
        Book foundBook = (Book) itemRepository.findOne(itemId);
        foundBook.setName(itemUpdateDTO.getName());
        foundBook.setPrice(itemUpdateDTO.getPrice());
        foundBook.setStockQuantity(itemUpdateDTO.getStockQuantity());
        foundBook.setAuthor(itemUpdateDTO.getAuthor());
        foundBook.setIsbn(itemUpdateDTO.getIsbn());

        // Dirty check -> em.persist() 및 tx.commit() 필요 없이 자동으로 update됨.
    }
```

##### **4) 상품 Controller**

기존의 controller.ItemController에 아래의 메서드를 추가한다.

```java
    @GetMapping("/items/{itemId}/edit")
    public String updateItemForm(@PathVariable(name = "itemId") Long itemId, Model model) {
        Book item = (Book) itemService.findOne(itemId);

        BookForm form = new BookForm();
        form.setId(item.getId());
        form.setName(item.getName());
        form.setPrice(item.getPrice());
        form.setStockQuantity(item.getStockQuantity());
        form.setAuthor(item.getAuthor());
        form.setIsbn(item.getIsbn());

        model.addAttribute("form", form);
        return "items/updateItemForm";
    }

    @PostMapping("/items/{itemId}/edit")
    public String updateItem(@PathVariable(name = "itemId") Long itemId,
                             @ModelAttribute(name = "form") ItemUpdateDTO itemUpdateDTO,
                             Model model) {
        itemService.updateBook(itemId, itemUpdateDTO);
        return "redirect:/items";
        
    }
```



##### **5) 상품 수정 폼 html**

templates/items/updateItemForm.html

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header" />
<body>
<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader"/>
    <form th:object="${form}" method="post">
        <!-- id -->
        <input type="hidden" th:field="*{id}" />
        <div class="form-group">
            <label th:for="name">상품명</label>
            <input type="text" th:field="*{name}" class="form-control"
                   placeholder="이름을 입력하세요" />
        </div>
        <div class="form-group">
            <label th:for="price">가격</label>
            <input type="number" th:field="*{price}" class="form-control"
                   placeholder="가격을 입력하세요" />
        </div>
        <div class="form-group">
            <label th:for="stockQuantity">수량</label>
            <input type="number" th:field="*{stockQuantity}" class="form-control" placeholder="수량을 입력하세요" />
        </div>
        <div class="form-group">
            <label th:for="author">저자</label>
            <input type="text" th:field="*{author}" class="form-control"
                   placeholder="저자를 입력하세요" />
        </div>
        <div class="form-group">
            <label th:for="isbn">ISBN</label>
            <input type="text" th:field="*{isbn}" class="form-control"
                   placeholder="ISBN을 입력하세요" />
        </div>
        <button type="submit" class="btn btn-primary">Submit</button>
    </form>
    <div th:replace="fragments/footer :: footer" />
</div> <!-- /container -->
</body>
</html>
```

##### **4) 실행 시 화면**

상품 수정 전 조회 화면

![IMAGE](/assets/images/spring-boot-jpa-practice001/0015/before-updating.png)

상품 수정 form 화면

![IMAGE](/assets/images/spring-boot-jpa-practice001/0015/update-item-form.png)

상품 수정 후 조회 화면

![IMAGE](/assets/images/spring-boot-jpa-practice001/0015/after-updating.png)

상품 수정 시 log 기록

![IMAGE](/assets/images/spring-boot-jpa-practice001/0015/logs-of-updating-item.png)

