# BeanDefinition

## BeanDefinition이란

`BeanDefinition`은 **빈을 어떻게 만들지 적어둔 설계도(메타데이터)** 다.

Spring 컨테이너는 설정 정보를 바로 객체로 만들지 않는다. 먼저 `BeanDefinition`으로 바꿔서 등록하고, 이걸 보고 빈을 만든다.

```text
@Component 클래스  ─┐
@Configuration/@Bean ─┼─▶ BeanDefinition ─▶ BeanFactory ─▶ 빈 객체
XML <bean>         ─┘
```

설정 방식이 무엇이든 컨테이너 입장에서는 `BeanDefinition`만 보면 된다. 설정 형식과 컨테이너가 분리되어 있어서 새로운 설정 방식을 추가하기 쉽다.

## 담고 있는 정보

| 속성 | 의미 |
| --- | --- |
| `beanClassName` | 생성할 빈의 클래스 이름 |
| `factoryBeanName` | `@Bean` 방식일 때 팩토리 역할 빈 이름 (예: `appConfig`) |
| `factoryMethodName` | 빈을 만들 팩토리 메서드 이름 (예: `orderService`) |
| `scope` | singleton, prototype 등 |
| `lazyInit` | 지연 초기화 여부 |
| `primary` | `@Primary` 여부 |
| `dependsOn` | 먼저 생성되어야 하는 빈 |
| `initMethodName` / `destroyMethodName` | 초기화, 소멸 메서드 이름 |
| `constructorArgumentValues` / `propertyValues` | 생성자 인자, 프로퍼티 값 |

등록 방식에 따라 채워지는 값이 다르다.

- `@Component` 스캔: `beanClassName`에 실제 클래스가 들어간다.
- `@Bean`: `beanClassName`은 비어 있고 `factoryBeanName`과 `factoryMethodName`이 들어간다. 설정 클래스의 메서드를 호출해서 빈을 만든다는 뜻이다.

### 직접 확인해보기

```java
AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

for (String name : ac.getBeanDefinitionNames()) {
    BeanDefinition bd = ac.getBeanDefinition(name);
    if (bd.getRole() == BeanDefinition.ROLE_APPLICATION) {   // 내가 등록한 빈만
        System.out.println(name + " = " + bd);
    }
}
```

`getRole()`로 직접 등록한 빈(`ROLE_APPLICATION`)과 Spring 내부 빈(`ROLE_INFRASTRUCTURE`)을 구분할 수 있다.

## BeanDefinition 등록 흐름

1. `AnnotatedBeanDefinitionReader`, `ClassPathBeanDefinitionScanner` 같은 리더가 설정을 읽는다.
2. `BeanDefinition`을 만들어 `BeanDefinitionRegistry`에 이름과 함께 등록한다. (`DefaultListableBeanFactory`가 이 인터페이스를 구현한다.)
3. `BeanFactoryPostProcessor`가 등록된 정의를 수정하거나 추가한다.
4. 컨테이너가 정의를 보고 빈을 만든다.

같은 이름의 `BeanDefinition`이 두 번 등록되면 Spring Boot는 기본적으로 시작을 실패시킨다(`spring.main.allow-bean-definition-overriding=false`).

## BeanFactoryPostProcessor

`BeanFactoryPostProcessor`는 **빈 객체가 만들어지기 전에 BeanDefinition을 다루는 확장 지점**이다.

```java
@Component
public class LazyAllPostProcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
        BeanDefinition bd = beanFactory.getBeanDefinition("reportService");
        bd.setLazyInit(true);   // 객체 생성 전이라 설계도를 바꿀 수 있다
    }
}
```

대표적인 구현체는 다음과 같다.

- `ConfigurationClassPostProcessor`: `@Configuration`, `@ComponentScan`, `@Import`, `@Bean`을 분석해 BeanDefinition을 등록한다. Spring Boot 자동 구성도 이 과정에서 들어온다.
- `PropertySourcesPlaceholderConfigurer`: BeanDefinition에 있는 `${...}` 플레이스홀더를 실제 프로퍼티 값으로 바꾼다.

`BeanFactoryPostProcessor`를 `@Bean`으로 등록할 때는 `static` 메서드로 선언해야 한다. 다른 빈보다 먼저 만들어져야 하는데, 일반 메서드면 설정 클래스 인스턴스부터 만들어야 하기 때문이다.

### BeanPostProcessor와 비교

| | BeanFactoryPostProcessor | BeanPostProcessor |
| --- | --- | --- |
| 대상 | BeanDefinition (설계도) | 빈 객체 (인스턴스) |
| 시점 | 빈 생성 전 | 빈 생성 후 초기화 전후 |
| 예시 | 컴포넌트 스캔, 플레이스홀더 치환 | `@Autowired` 처리, AOP 프록시 생성 |

`BeanPostProcessor`는 [Bean Lifecycle](./bean-lifecycle.md)에서 자세히 정리했다.
