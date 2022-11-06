---
title: "객체 지향 원리 적용하기"
excerpt: ""

categories:
  - "Web"
tags:
  - ["Spring", "OOP"]

date_created: 2022-11-06T22:08:36+09:00
last_modified_at: 2022-11-06T22:08:36+09:00
toc: true
toc_sticky: true
---
 

## Previously

  

기본적인 예제를 구현하여 회원 도메인과 주문 및 할인 도메인으로 구성된 쇼핑몰을 간단하게 구성해봤다. 하지만 객체 지향 프로그래밍의 원칙 중 OCP(개방 폐쇄 원칙)과 DIP(의존 관계 역전 원칙)이 제대로 지켜지지 않고 있는 것을 확인했다.

  

이런 문제가 있음을 인지하고 어떻게 객체 지향 원리를 적용하는지 알아보자.

  

[코드 레포](https://github.com/gnlenfn/SpringBasic){: target="_blank"}

  

## 새로운 할인 정책 개발

  

이전 상황에서 1000원의 정액 할인이 적용되어 있었는데, 구매 가격의 10% 할인이 최종 정책으로 결정되었다면 이미 역할과 구현을 분리해둔 만큼 10% 할인을 해주는 정책을 구현하여 구현부로 갈아끼우면 손쉽게 적용이 가능하다.

  

```java

public class OrderServiceImpl implements OrderService{

  

    private final MemberRepository memberRepository = new MemoryMemberRepository();

  

    //private final DiscountPolicy discountPolicy = new FixDiscountPolicy();

    private final DiscountPolicy discountPolicy = new RateDiscountPolicy();

    ...

}

```

  

그런데, 새로 구현한 정책을 적용하기 위해서는 주문 구현부의 코드를 변경해야하기 때문에 앞서 언급한대로 OCP, DIP 원칙이 지켜지지 못하고 있다. 할인 정책을 구현체에 의존하다보니 원칙을 지키지 못하는 것이다. 따라서 인터페이스만 의존하도록 바꿔주어야 하는데…

  

```java

public class OrderServiceImpl implements OrderService{

  

    private final MemberRepository memberRepository = new MemoryMemberRepository();

    private final DiscountPolicy discountPolicy;

    ...

}

```

  

인터페이스만 의존하도록하면 새로운 할인 정책의 구현체는 가리키는 곳이 없어진다. (NullPointerException)

  

![Untitled](/assets/img/2022-11-06/expected.png)

  

우리는 `OrderSeriviceImpl`이 `DiscountPolicy`라는 인터페이스에 의존하여 해당 인터페이스의 구현체가 바뀌면 새로운 내용이 적용될 수 있기를 기대했다. 하지만 예제는 인터페이스 뿐 아니라 구현체에도 함께 의존하고 있기 때문에 객체 지향 원칙을 위반하게 되는 것이다.

  

결국 `OrderServiceImpl`이 아니라 다른 객체에서 할인 정책에 대한 **구현체를 대신 생성하여 구현체에 주입해주어야 한다**.

  

## AppConfig

  

애플리케이션의 전체 동작 방식을 구성하기 위해 구현 객체를 생성하고, 연결하는 책임을 가진 설정 클래스를 만들어서 의존 관계를 연결해주도록 한다.

 

- AppConfig는 애플리케이션의 실제 동작에 필요한 구현 객체를 생성한다

- AppConfig가 생성한 인스턴스의 참조를 생성자를 통해 주입(연결) 해준다.

- 각 구현부에는 생성자를 만들어 주입을 받는다

  

```java

public class MemberServiceImpl implements MemberService {

    private final MemberRepository memberRepository;

    public MemberServiceImpl(MemberRepository memberRepository) {

        this.memberRepository = memberRepository;

    }

  

...

}

```

  

이렇게 구현부에서는 생성자를 통해 만들어진 인스턴스를 주입받고 AppConfig는 인스턴스를 생성하여 주입해주는 것

  

![Untitled](/assets/img/2022-11-06/split.png)

  

그 결과 구성 영역과 사용 영역시 분리되었다. 현재 고정 할인으로 정해진 정책이 정률 할인으로 바뀐다면, AppConfig를 통해 `RateDiscountPolicy`를 생성하고 각 사용 영역의 객체들에게 주입해준다.

  

지금까지는 스프링 없이 순수 자바로만 객체 지향 원리를 적용한 애플리케이션을 만들었다

**(commit cc6d0e5a8b1ff746ea34f31351c1bf3c59440de2)**

  

## 스프링으로 전환

  

스프링으로 위의 코드들을 전환하면 **(commit 1e6879be699d6d926e7462229e9056f1273642d7)** 애노테이션을 활용하게 된다.

  

스프링이 실행되면 `@Configuration`이 붙은 것을 설정 정보로 사용하여 `ApplicationContext` 컨테이너에 `@Bean`으로 등록한 것들을 모두 생성한다. 이것들을 스프링 빈이라고 하며 메서드 명을 빈의 이름으로 사용한다.

  

기존에는 개발자가 직접 구성 영역을 만들어 (AppConfig) 구현체 생성 및 의존관계 주입을 해주었다면, 이제 스프링 컨테이너에 객체를 스프링 빈으로 등록하여 스프링 컨테이너에서 해당 빈을 찾아 사용하도록 변경되었다. 아직은 적용 안됐지만 Autowired 같은 것들이 이렇게 사용된다고 할 수 있다.

  

## ****IoC, DI, 그리고 컨테이너****

  

`AppConfig` 클래스를 만들어서 빈을 등록하고 사용하도록 코드를 바꿔보았다. 이렇게 객체를 생성하고 관리하면서 의존 관계를 연결해주는 것을 IoC컨테이너 혹은 DI 컨테이너라고 한다. 의존 관계 주입에 초점을 맞추어 DI (Dependency Injection) 이라고 주로 부른다.

[이전 포스팅](/_posts/2022-11-02-OOP.md){: target="_blank"}에서 스프링이 객체 지향을 추구하고 그로 인해 대규모 개발에서 이점이 있다고 언급을 했는데, 객체 지향에 맞는 애플리케이션을 구축하고 나면 이 부분을 느낄 수 있는 것 같다. 코드가 복잡해지더라도 코드의 수정이 필요할 때 개발자가 일일이 모든 부분을 찾아 수정할 필요 없이 정해진 부분만 수정하면 문제없이 적용 가능하기 때문이다. 
OCP, DIP 원칙과 객체 지향의 다형성을 활용해 유지보수와 확장이 용이한 애플리케이션을 만들 수 있는 것이다. 하지만 추상화 된 코드는 처음에 보기엔 쉽게 이해하지 못할 수 있기 때문에 열심히 공부할 필요가 있겠다.

  

다음에는 스프링에서 어떻게 빈을 관리하는지 조금 더 자세히 작성해보겠다.