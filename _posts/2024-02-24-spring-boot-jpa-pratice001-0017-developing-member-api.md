---
layout: post
title: "JPA Practice001: #0017 - Developing Member API"
categories: junk
author:
  - Youngmoo Park
meta: "Springfield"
---

이 예제는 Inflearn에서 김영한 강사님이 강의하시는 '실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발' 강의를 수강하면서 클론 코딩해보는 예제를 다루고 있습니다.

## 1. 회원 API 개발 목차

- API 개발 준비
- 회원 등록 API
- 회원 수정 API
- 회원 조회 API
<br/>

## 2. API 개발 준비

https://www.postman.com/downloads/

postman 설치 후 실행
<br/>

## 3. 회원 등록 API 개발

- v1 대신 v2를 사용할 것이다.

#### **1) 회원 등록 API 코드**

api.MemberApiController

```java
package jpabook.jpashop.api;

import jakarta.validation.Valid;
import jpabook.jpashop.domain.Member;
import jpabook.jpashop.service.MemberService;
import lombok.Data;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;



@RestController // @Controller + @ResponseBody
@RequiredArgsConstructor
public class MemberApiController {

    private final MemberService memberService;

    @PostMapping("/api/v1/members")
    public CreateMemberResponse saveMemberV1(@RequestBody @Valid Member member) {
        Long id = memberService.join(member);
        return new CreateMemberResponse(id);
    }

    @PostMapping("/api/v2/members")
    public CreateMemberResponse saveMemberV2(@RequestBody @Valid CreateMemberRequest request) {

        Member member = new Member();
        member.setName(request.getName());
        Long id = memberService.join(member);
        return new CreateMemberResponse(id);
    }

    @Data
    static class CreateMemberRequest {
        private String name;
    }

    @Data
    static class CreateMemberResponse {
        private Long id;

        public CreateMemberResponse(Long id) {
            this.id = id;
        }
    }
}
```

#### **2) 기술 설명**

- /api/v1/members
```plaintext
-- Member Entity를 그대로 파라미터에 넣는 방식이다.
-- API별로 어떤 값만 받을 수 있는지를 알 수 없다.
-- 하나의 Entity만으로는 원하는 정보를 모두 가져오기에 역부족인 경우가 빈번히 발생한다.
-- 또한, Entity를 외부에 노출시키는 것은 좋지 않다.  
```

- /api/v2/members
```plaintext
-- v1과 달리 유효성 검증을 위해 만든 별도의 DTO 클래스이다.
-- API에 맞는 클래스를 별도로 만들어주기 때문에, 해당 클래스만 봐도 어떤 값만 전달하면 되는지 쉽게 파악할 수 있다.
```

#### **3) 요청과 응답**

postman을 이용한 요청과 응답

![IMAGE](/assets/images/spring-boot-jpa-practice001/0017/save-member-v2-postman.png)

JPA Insert문

![IMAGE](/assets/images/spring-boot-jpa-practice001/0017/save-member-v2-jpa.png)
<br/>

## 4. 회원 수정 API 개발

- HTTP의 PUT 메서드를 사용할 것이다.

#### **1) 회원 수정 API 코드**

api.MemberApiController에 아래의 메서드를 추가한다.

```java
    @PutMapping("/api/v2/members/{id}")
    public UpdateMemberResponse updateMemberV2(@PathVariable(name = "id") Long id,
                                               @RequestBody @Valid UpdateMemberRequest request) {
        memberService.update(id, request.getName());
        Member updatedMember = memberService.findOne(id);
        return new UpdateMemberResponse(updatedMember.getId(), updatedMember.getName());
    }
```

#### **2) 회원 수정 요청 및 응답 클래스**

api.MemberApiController 내부에서만 쓰이기 때문에 MemberApiController 내부에 static 클래스로 정의한다.

```java
    @Data
    static class UpdateMemberRequest {
        private String name;
    }

    @Data
    @AllArgsConstructor
    static class UpdateMemberResponse {
        private Long id;
        private String name;
    }
```

#### **3) 회원 수정 Service 코드**

service.MemberService 내부에 아래의 메서드를 추가한다.

```java
    /**
     * 단일 회원 수정
     */
    @Transactional
    public void update(Long id, String name) {
        Member member = memberRepository.findOne(id);
        member.setName(name);
    }
```

#### **4) 요청과 응답**

회원 수정 전 

![IMAGE](/assets/images/spring-boot-jpa-practice001/0017/before-updating-member.png)

postman을 이용한 요청과 응답

![IMAGE](/assets/images/spring-boot-jpa-practice001/0017/update-member-v2-postman.png)

JPA Update문

![IMAGE](/assets/images/spring-boot-jpa-practice001/0017/logs-of-updating-member.png)

회원 수정 후

![IMAGE](/assets/images/spring-boot-jpa-practice001/0017/after-updating-member.png)
<br/>

## 5. 회원 조회 API 개발

- HTTP의 GET 메서드를 사용할 것이다.

#### **1) 회원 조회 API 코드**

api.MemberApiController에 아래의 메서드를 추가한다.

```java

```java
    @GetMapping("/api/v2/members")
    public Result membersV2() {
        List<Member> members = memberService.findAll();
        List<MemberDTO> collect = members.stream()
                .map(member -> new MemberDTO(member.getName()))
                .collect(Collectors.toList());
        return new Result(collect.size(),collect);
    }
```

#### **2) 회원 조회 DTO 클래스**

api.MemberApiController 내부에서만 쓰이기 때문에 MemberApiController 내부에 static 클래스로 정의한다.

```java
    @Data
    @AllArgsConstructor
    static class Result<T> {
        private int count;
        private T data;
    }

    @Data
    @AllArgsConstructor
    static class MemberDTO {
        private String name;
    }
```

#### **3) 요청과 응답**

postman을 이용한 요청과 응답

![IMAGE](/assets/images/spring-boot-jpa-practice001/0017/update-member-v2-postman.png)

JPA Update문

![IMAGE](/assets/images/spring-boot-jpa-practice001/0017/logs-of-listing-member.png)
