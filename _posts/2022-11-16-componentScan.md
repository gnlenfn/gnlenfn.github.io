---
title: "컴포넌트 스캔"
excerpt: ""

categories:
  - "Web"
tags:
  - ["Spring"]

date_created: 2022-11-02T23:32:22+09:00
last_modified_at: 2022-11-02T23:32:22+09:00
toc: true
toc_sticky: true
---

# 컴포넌트 스캔

지금까지는 개발자가 직접 설정 파일을 작성하고 `@Bean`을 등록하는 과정을 거쳤다. 하지만 이 역시 너무 귀찮은 과정이니 스프링이 자동으로 해주는 컴포넌트 스캔에 대해 알아보자

```jsx
@Configuration
@ComponentScan(

public class AutoAppConfig {

}
```

설정 파일을 만든 뒤 `@ComponentScan`을 붙이면 끝이다. 기존의 설정 파일과 다르게 `@Bean`으로 등록하는 클래스가 하나도 없다. 그리고 이름 그대로 `@Conponent`가 붙은 클래스르 스캔해서 스프링 빈으로 등록해준다. 그래서 우리가 만든 클래스에 애노테이션만 붙여주면 스프링 빈 등록이 끝난다.

![Untitled](/assets/img/2022-11-16/componentScan.png)

이때 `@Conponent`가 붙은 클래스들의 이름을 첫 글자만 소문자로 하여 빈이 등록된다. 빈 이름을 직접 지정하고 싶다면 애노테이션 안에 설정해주면 된다.

![Untitled](/assets/img/2022-11-16/Autowired.png)

그리고 등록된 빈은 `@Autowired`를 통해 사용한다. 생성자에 애노테이션을 붙이면 스프링 컨테이너가 자동으로 해당 빈을 찾아 주입해준다.

### 컴포넌트 스캔 대상

`@ComponentScan`은 `@Component`뿐 아니라 다른 애노테이션들도 추가로 스프링 빈에 등록한다

- `@Component`
- `@Controller`
- `@Service`
- `@Repository`
- `@Configuration`

해당 클래스의 소스 코드를 보면 모두 `@Component`를 포함하고 있기 때문이다.

또한 원하는 위치부터 컴포넌트를 스캔할 수도 있다. `@ComponentScan` 애노테이션에 `basePackage` 파라미터를 넣어주면 해당 패키지 아래에 있는 컴포넌트를 스캔하여 등록해준다.

### 중복 등록

두 개의 같은 이름 빈이 등록되면 컴포넌트 스캔은 어떻게 처리할까?

- 컴포넌트 스캔에 의해 자동으로 등록되는 빈이 2개 있는데, 이름이 같다
→ `ConflictingBeanDefinitionException` 발생
- 수동으로 등록한 빈과 자동으로 등록한 빈이 충돌한다
→ 수동 빈이 자동 빈을 오버라이딩 해버린다.

이때 두 번째 경우가 해결하기 어려운 버그를 많이 만들어낸다고 한다. 어디서 잘못된 것인지 찾기가 쉽지 않기 때문이다. 그래서 최근 스프링 부트에서는 수동 빈 등록과 자동 빈 등록이 충돌이 나면 오류가 발생하도록 기본값을 바꿨다고 한다.