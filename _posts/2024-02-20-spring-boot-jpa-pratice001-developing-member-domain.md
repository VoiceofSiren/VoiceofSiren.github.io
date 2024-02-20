---
layout: post
title: "JPA Practice001: #004 - Developing Member Domain and Repository"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. 회원 도메인 개발 목차

- 구현 기능
  - 회원 등록
  - 회원 목록 조회

- 순서
  - 회원 Repository 개발
  - 회원 Service 개발
  - 회원 기능 테스트 (다음 글에서)
<br/>

## 2. 회원 Repository 개발

#### **1) 회원 Repository 코드**

```java
package jpabook.jpashop.repository;

import jakarta.persistence.EntityManager;
import jakarta.persistence.PersistenceContext;
import jpabook.jpashop.domain.Member;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
@RequiredArgsConstructor
public class MemberRepository {

    @PersistenceContext
    private final EntityManager em;

    public Long save(Member member) {
        em.persist(member);
        return member.getId();
    }

    public Member findOne(Long id) {
        return em.find(Member.class, id);
    }

    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    public List<Member> findByName(String name) {
        return em.createQuery("select m from Member m where m.name = :name", Member.class)
                .setParameter("name", name)
                .getResultList();
    }
}

```

#### **2) 기술 설명**

- @Repository
```plaintext
-- JPA 저장소를 표시하는 annotation.
-- Spring bean으로 자동 등록된다.
-- JPA 예외를 String 기반 예외로 변환한다.
```

- @RequiredArgsConstructor
```plaintext
-- 생성자에 필수 의존성을 명시하는 annotation.
-- @Autowired와 함께 사용하면 생성자 주입 방식을 사용할 수 있다.
```

- @PersistenceContext
```plaintext
-- JPA EntityManager를 주입하는 annotation.
-- @PersistenceUnit private EntityManagerFactory emf;를 거의 사용하지 않게 된다.
```
<br/>

## 3. 회원 Service 개발

#### **1) 회원 Service 코드**

```java
package jpabook.jpashop.service;

import jpabook.jpashop.domain.Member;
import jpabook.jpashop.repository.MemberRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service
@RequiredArgsConstructor
@Transactional
public class MemberService {

    private final MemberRepository memberRepository;

    /**
     * 회원 가입
     */
    @Transactional
    public Long join(Member member) {
        validateMemberDuplication(member); // 중복 회원 검증
        return memberRepository.save(member);
    }

    /**
     * 중복 회원 검증
     */
    private void validateMemberDuplication(Member member) {
        List<Member> findMembers = memberRepository.findByName(member.getName());
        if (!findMembers.isEmpty()) {
            throw new IllegalStateException("이미 존재하는 회원입니다.");
        }
    }

    /**
     * 회원 전체 조회
     */
    @Transactional(readOnly = true)
    public List<Member> findAll() {
        return memberRepository.findAll();
    }

    /**
     * 단일 회원 조회
     */
    @Transactional(readOnly = true)
    public Member findOne(Long memberId) {
        return memberRepository.findOne(memberId);
    }

}

```

#### **2) 기술 설명**

- @Service
```plaintext
-- 비즈니스 로직을 처리하는 객체를 표시하는 annotation.
-- Spring bean으로 자동 등록된다.
```

- @Transactional
```plaintext
-- 트랜잭션을 관리하는 annotation.
-- 메서드에 적용하면 해당 메서드 내에서 일어나는 모든 작업은 하나의 트랜잭션으로 처리된다.
-- JPA를 통해 데이터를 조회하거나 조작하는 모든 기능은 트랜잭션 내부에서 실행되어야 하고, 이 annotation이 있어야 지연 로딩을 사용할 수 있다.
-- DB Driver가 지원하면 DB에서 성능이 향상된다.
-- readOnly=true: 조회 전용 메서드에서 사용하면 영속성 컨텍스트를 flush하지 않으므로 약간의 성능이 향상된다. (default: readOnly=false)
```

- @Autowired
```plaintext
-- 생성자, setter 메서드 또는 필드에 사용하여 의존성을 자동으로 주입하는 annotation.
-- 필드에 사용하는 경우 @RequiredArgsConstructor와 함계 사용하면 더욱 안전하게 객체를 생성할 수 있다.
```
<br/>

## 4. 회원 기능 테스트

- 테스트 요구사항
  - 회원 가입이 성공적으로 이루어저야 한다.
  - 회원 가입 시 동일한 이름의 회원이 있다면 예외가 발생해야 한다.

#### **1) 회원 가입 테스트 코드**
```java
package jpabook.jpashop;

import jpabook.jpashop.domain.Member;
import jpabook.jpashop.repository.MemberRepository;
import org.assertj.core.api.Assertions;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.annotation.Rollback;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.annotation.Transactional;


@RunWith(SpringRunner.class)
@SpringBootTest
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    @Transactional
    @Rollback(value = false)
    public void testMember() throws Exception {
        //given
        Member member = new Member();
        member.setName("memberA");

        //when
        Long savedId = memberRepository.save(member);
        Member findMember = memberRepository.findOne(savedId);

        //then
        Assertions.assertThat(findMember.getId()).isEqualTo(member.getId());
        Assertions.assertThat(findMember.getName()).isEqualTo(member.getName());
        Assertions.assertThat(findMember).isEqualTo(member);
    }


}
```