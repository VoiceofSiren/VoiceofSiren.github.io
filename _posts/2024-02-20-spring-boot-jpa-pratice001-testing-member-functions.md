---
layout: post
title: "JPA Practice001: #005 - Testing Member Functions"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. 회원 기능 테스트 목차

- 구현 기능
  - 회원 등록
  - 회원 목록 조회

- 순서
  - 회원 기능 테스트
<br/>

## 2. 회원 기능 테스트

#### **1) 회원 Service Test 코드 - 과정 1**

```java
package jpabook.jpashop.service;

import jpabook.jpashop.domain.Member;
import jpabook.jpashop.repository.MemberRepository;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.annotation.Transactional;

import static org.junit.Assert.*;


@RunWith(SpringRunner.class)
@SpringBootTest
@Transactional
public class MemberServiceTest {

  @Autowired
  private MemberService memberService;

  @Autowired
  private MemberRepository memberRepository;

  @Test
  public void 회원_가입() throws Exception {
    //given
    Member member = new Member();
    member.setName("kim");

    //when
    Long savedId = memberService.join(member);

    //then
    assertEquals(member, memberRepository.findOne(savedId));

  }

  @Test
  public void 중복_회원_예외() throws Exception {
    //given

    //when

    //then

  }

}
```

위와 같은 코드로 작성할 경우 아래와 같은 쿼리문이 실행된다.

```plaintext
Hibernate: 
    select
        m1_0.member_id,
        m1_0.city,
        m1_0.street,
        m1_0.zipcode,
        m1_0.name 
    from
        member m1_0 
    where
        m1_0.name=?
```

##### **i - 발견된 문제점** 

```plaintext
- insert문이 실행되지 않고 select문만 한 번 실행되었다.
```

##### **ii - 문제점 분석**
```plaintext
-- 트랜잭션 상에서 commit()을 호출하는 시점에 insert문이 DB에서 실행된다.
-- 하지만 MemberRepository에 정의된 메서드 내부에서는 commit()을 호출하지 않았다.
-- @Transactional은 기본적으로 트랜잭션을 commit()하지 않고 rollback()한다.
```


#### **2) 회원 Service Test 코드 - 과정 2**

public void 회원_가입() 위에 @Rollback(value = false)를 추가한다.

```java
@Test
@Rollback(value = false)
public void 회원_가입() throws Exception {
    //given
    Member member = new Member();
    member.setName("kim");

    //when
    Long savedId = memberService.join(member);

    //then
    assertEquals(member, memberRepository.findOne(savedId));

}
```

또는 EntityManager을 주입하여 flush()를 강제 호출하는 아래와 같은 방식으로 코딩해도 된다.
```java
public class MemberServiceTest {

    @Autowired
    private MemberService memberService;

    @Autowired
    private MemberRepository memberRepository;

    @Autowired
    private EntityManager em;

    @Test
    public void 회원_가입() throws Exception {
        //given
        Member member = new Member();
        member.setName("kim");

        //when
        Long savedId = memberService.join(member);

        //then
        em.flush();
        assertEquals(member, memberRepository.findOne(savedId));

    }
}
```

위의 변경 사항을 적용하여 코드를 실행하면 아래처럼 insert문이 실행되는 것을 확인할 수 있다.

```plaintext
Hibernate: 
    insert 
    into
        member
        (city, street, zipcode, name, member_id) 
    values
        (?, ?, ?, ?, ?)
```

