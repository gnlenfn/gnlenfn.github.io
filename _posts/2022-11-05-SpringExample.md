---
title: "스프링 기본예제 만들기"
excerpt: ""

categories:
  - "Web"
tags:
  - ["Spring", "OOP"]

date_created: 2022-11-05T10:47:09+09:00
last_modified_at: 2022-11-05T10:47:09+09:00
toc: true
toc_sticky: true
---

# 기본 예제 만들기

  

스프링을 사용하지 않고 순수 자바 코드만 사용하여 예제를 만들어본다. 이때 설정을 쉽게 하기 위해 스프링 부트로 프로젝트를 생성했지만, 그 방법은 포스팅에서 생략.

  

이 후 스프링 기능을 추가하며 스프링에서 제공하는 기능들이 어떤 역할을 하고 어떻게 작동하는지 알아본다.

  

## 회원 도메인 설계 시나리오

  

![Untitled](/assets/img/2022-11-05/회원도메인.png)

  

- 클라이언트를 통해 회원 가입과 회원 목록을 볼 수 있다.

- 회원 등급에는 일반, VIP가 있다

- 회원 데이터를 위한 DB를 구축하고 외부 시스템과 연동할 수 있다

    - 아직 어떤 DB를 사용할 지 확정되지 않았음을 상정하고 추후 객체 지향 개발을 통해 쉽게 갈아 끼우는 것을 해보자

  

![Untitled](/assets/img/2022-11-05/회원세부클래스.png)

  

- 회원 서비스 역할은 MemberService 인터페이스에 정의하고 실제 구현은 MemberServiceImpl에서 한다

- 회원 저장소 역할은 MemberRepository 인터페이스에 정의하고 현재 임시로 사용할 MemoryMemberRepository를 구현하고 추후에는 DB를 구현할 DbMemberRepository 것

→ 구현인 만큼 `Impl` 이름을 통일하는 건 어땠을까

  

```java

package hello.core.member;

public class MemberServiceImpl implements MemberService {

  

    private final MemberRepository memberRepository = new MemoryMemberRepository();

  

    public void join(Member member) {

        memberRepository.save(member);

    }

  

    public Member findMember(Long memberId) {

        return memberRepository.findById(memberId);

    }

}

```

  

그런데, 위의 `MemberServiceImpl` 구현체를 보면 `memberRepository`를 구현체 안에서 선언하고 있다. 그렇다면 만약 추후에 메모리 저장이 아닌 DB를 사용하게 된다면 해당 코드를 수정해야 할 것이다.

  

하지만 지금 객체 지향의 원칙 SOLID 중 OCP와 DIP를 동시에 만족시키는 것을 확인하기 위해 공부하고있는데, 둘 다 지켜지지 않았다는 뜻이다.

  

- 기능 확장 시 코드 수정이 필요하므로 OCP원칙 위배

- 의존 관계가 인터페이스 뿐 아니라 구현체에도 의존하고 있어 DIP 위배

  

계속해서 진행하면서 해당 부분을 어떻게 해결할 지 알아 볼 것이다.

  

## 주문과 할인 도메인 설계

  

- 회원은 상품을 주문할 수 있다

- 현재 할인 정책은 VIP회원에게 1000원을 할인해준다. (추후 변경 가능성 농후)

- 아직 할인 정책이 확정되지 않았고 언제 결정될 지 알 수 없다

  

![Untitled](/assets/img/2022-11-05/주문할인도메인.png)

  

역할과 구현을 분리했기 때문에 유연하게 구현 객체를 재사용할 수 있다. **2. 회원 조회**의 경우 앞선 회원 도메인 파트를 그대로 사용하면 되고, 할인 정책 역시 구현체를 갈아끼워 새로운 정책을 적용할 수 있도록 하면 된다.

  

```java

public class OrderServiceImpl implements OrderService {

  

private final MemberRepository memberRepository = new MemoryMemberRepository();

private final DiscountPolicy discountPolicy = new FixDiscountPolicy();

  

@Override

public Order createOrder(Long memberId, String itemName, int itemPrice) {

  

    Member member = memberRepository.findById(memberId);

    int discountPrice = discountPolicy.discount(member, itemPrice);

    return new Order(memberId, itemName, itemPrice, discountPrice);

    }

}

```

  

하지만 바로 앞에서 언급한 OCP, DIP 위배는 여전히 해결되지 않았다. 현재 1000원 고정 할인에서 물건 가격의 10%할인으로 바뀐다면 해당 정책의 구현 뿐 아니라 할인을 적용하든 주문 구현체의 코드 역시 수정이 필요해진다.

  

---

  

따라서 구현과 역할을 분리하는 것 외에도 다른 것이 추가되어야 SOLID 원칙에 걸맞는 객체 지향 애플리케이션을 만들 수 있는 것이다.