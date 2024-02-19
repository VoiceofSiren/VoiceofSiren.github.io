---
layout: post
title: "Implementation Requirements and Application Architecture in Spring boot JPA practice001"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. 구현 요구사항

![IMAGE](assets/images/0003/implementation-requirements.png)

#### **1) 구현할 기능 **

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

#### **2) 예제 단순화를 위해 구현하지 않을 기능**

- 로그인과 권한 관리
- 파라미터 검증과 예외 처리
- 도서 이외의 상품
- 카테고리
- 배송 정보

## 2. 애플리케이션 아키텍처

![IMAGE](assets/images/0003/application-architecture.png)

#### **1) 계층형 구조 사용**

- Controller, WEB: 웹 계층
- Service: 비즈니스 로직, 트랜잭션 처리
- Repository: JPA를 직접 사용하는 계층
- Domain: Entity가 모여있는 계층

#### **2) 패키지 구조**

- jpabook.jpashop
  - domain
  - exception
  - repository
  - service
  - web

#### **3) 개발 순서**

- Service, Repository 계층 개발
- 테스크 케이스를 작성하여 검증
- Web 계층에 적용
