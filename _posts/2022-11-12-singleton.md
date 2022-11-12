# 스프링 컨테이너와 스프링 빈

앞서서 `ApplicationContext`를 생성하여 DI를 사용했다. 이때 `ApplicationContext`가 바로 스프링 컨테이너이다. 스프링 컨테이너는 전에 만들어본 `AppConfig`처럼 애노테이션 기반의 자바 클래스로 만들 수도 있고, XML 기반으로 만들 수도 있다. 최근에는 대부분 자바 클래스로 만든다고 한다.

## 스프링 컨테이너 생성 과정

스프링 컨테이너 생성은 크게 생성 - 등록 - 의존관계 주입 두 단계로 나누어진다고 이해할 수 있다.
(실제로는 한 번에 처리됨)

### 1. 스프링 컨테이너 생성

`new AnnotationConfigApplicationContext(AppConfig.class)` 를 만들면 컨테이너가 만들어진다. 이때 구성 정보를 지정해 주어야한다. 

![Untitled](/assets/img/2022-11-12/SpringContainer.png)

### 2. 스프링 빈 등록

![Untitled](/assets/img/2022-11-12/AddBean.png)

컨테이너가 생성되면 지정한 스프링 빈을 모두 등록한다. `@Bean` 애노테이션이 붙어있으면 스프링이 실행되면서 해당 부분을 모두 빈으로 등록한다. 이때 빈의 이름은 기본적으로 메소드의 이름과 일치하고 직접 이름을 지정해줄 수도 있다. 이때 빈의 이름은 유니크해야 충돌이 일어나지 않는다.

### 3. 의존관계 설정

![Untitled](/assets/img/2022-11-12/Dependency.png)

스프링 빈이 모두 등록되고 나면 설정정보를 참고해서 의존 관계를 자동으로 주입(DI)한다. 

## 등록된 모든 빈 조회

```java
public class ApplicationContextInfoTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("모든 빈 출력")
    void findAllBeans() {
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            Object bean = ac.getBean(beanDefinitionName);

            System.out.println("name = " + beanDefinitionName + " / object = " + bean);
        }
    }

    @Test
    @DisplayName("애플리케이션 빈 출력")
    void findApplicationBeans() {
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);

            if (beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
                Object bean = ac.getBean(beanDefinitionName);
                System.out.println("name = " + beanDefinitionName + " / object = " + bean);
            }
        }
    }

}
```

- 모든 빈 출력
    - `getBeanDefinitionNames`를 통해 등록된 모든 빈 이름을 조회하고
    - `ac.getBean()`으로 빈 객체 (인스턴스)를 조회한다
- 애플리케이션 빈 출력
    - 빈의 role을 확인할 수도 있는데 `ROLE_APPLICATION`은 사용자가 정의한 빈을 가리킨다
    - `ROLE_INFRASTRUCTURE`는 스프링 내부에서 사용하는 빈
- 빈을 조회할 때는 우선 타입으로 조회하고, 일치하는 이름을 그다음 찾는다
    - `ac.getBean('name', type)`
    - 조회 대상이 없으면 `NoSuchBeanDefinitionException` 예외 발생

```java
public class ApplicationContextSameBeanFindTest {
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SameBeanConfig.class);

    @Test
    @DisplayName("타입으로 조회 시 같은 이름 있으면 중복 오류")
    void findBeanByTypeDuplicate() {
        assertThrows(NoUniqueBeanDefinitionException.class,
                () -> ac.getBean(MemberRepository.class));
    }

    @Test
    @DisplayName("타입으로 조회 시 같은 이름 있으면 빈 이름을 지정하면 된다")
    void findBeanByName() {
        MemberRepository memberRepository = ac.getBean("memberRepository1", MemberRepository.class);
        assertThat(memberRepository).isInstanceOf(MemberRepository.class);
    }

    @Test
    @DisplayName("특정 타입으로 모두 조회하기")
    void findALlBeanByType() {
        Map<String, MemberRepository> beansOfType = ac.getBeansOfType(MemberRepository.class);
        for (String key : beansOfType.keySet()) {
            System.out.println("key = " + key + " value = " + beansOfType.get(key));
        }
        System.out.println("beansOfType = " + beansOfType);
        assertThat(beansOfType.size()).isEqualTo(2);
    }

    @Configuration
    static class SameBeanConfig {

        @Bean
        public MemberRepository memberRepository1() {
            return new MemoryMemberRepository();
        }

        @Bean
        public MemberRepository memberRepository2() {
            return new MemoryMemberRepository();
        }
    }
}
```

만약 동일한 타입의 스프링 빈이 둘 이상 있으면? → 오류가 발생하기 때문에 빈 이름을 지정해야 함

`getBeansOfType` 을 통해 해당 타입의 빈과 이름을 맵 형태로 얻을 수 있다.

![Untitled](/assets/img/2022-11-12/beanResult.png)

## 빈 조회 - 상속 관계

![Untitled](/assets/img/2022-11-12/inherit.png)

- 부모 타입의 빈을 조회하면 자식 타입도 함께 조회된다

## BeanFactory와 ApplicationContext

**BeanFactory**

- 스프링의 최상위 컨테이너
- 스프링 빈을 관리하고 조회하는 역할
- 지금까지의 대부분 기능이 BeanFactory가 제공하는 기능

**ApplicationContext**

- BeanFactory의 기능을 모두 상속받아 사용
- App을 개발할 때는 빈 관리 외에도 수많은 기능을 필요로하는데 이 모든 것을 포함하는 ApplicationContext를 사용하는 것이다 —> 그래서 위 코드에서 항상 ac로 메소드 호출

![Untitled](/assets/img/2022-11-12/AppContext.png)

## XML 등의 다양한 설정 형식 지원

스프링은 자바 코드 외에도 XML, Groovy 등 다양한 설정 정보 형식을 지원한다.

![Untitled](/assets/img/2022-11-12/variousContext.png)

대표적으로 XML형식이 있는데, 설정 방법은 해당 커밋(61121dd6a8bba3fab4e6809aca2bcc83dd11e7df)을 참조하면 된다.

### BeanDefinition

스프링은 BeanDefinition 추상화를 통해 이렇게 다양한 형식을 지원하는 것이다. 역할과 구현을 나누었기 때문에 가능한 것이다. 어떤 형식의 설정 정보가 들어오든 읽어서 BeanDefinition을 만들면 빈 설정 정보가 생성되는 것이다. 따라서 스프링 컨테이너는 해당 메타 정보를 사용하기 때문에 형식이 어떤 것이든 관계없이 스프링 컨테이너를 만들 수 있다.

실무에서 직접 BeanDefinition을 생성해서 등록할 일은 거의 없다고한다. 그냥 스프링이 다양한 설정 정보를 추상화해서 사용할 수 있다는 정도만 알고 넘어가자.