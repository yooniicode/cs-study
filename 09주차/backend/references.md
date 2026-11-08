# 9주차 참고 자료

## IoC 컨테이너

- [The IoC Container (Spring 공식 문서)](https://docs.spring.io/spring-framework/reference/core/beans.html)
  - IoC 컨테이너, 빈 정의, 의존성 주입, 스코프를 다루는 Spring Core의 중심 문서
- [Introduction to the Spring IoC Container and Beans (Spring 공식 문서)](https://docs.spring.io/spring-framework/reference/core/beans/introduction.html)
  - BeanFactory와 ApplicationContext의 관계
- [The BeanFactory API (Spring 공식 문서)](https://docs.spring.io/spring-framework/reference/core/beans/beanfactory.html)
  - BeanFactory와 ApplicationContext의 기능 비교표
- [Inversion of Control Containers and the Dependency Injection pattern (Martin Fowler)](https://martinfowler.com/articles/injection.html)
  - IoC와 DI라는 용어를 정리한 글. 생성자 주입과 setter 주입 비교

## DI / Bean

- [Dependency Injection (Spring 공식 문서)](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)
  - 생성자 주입과 setter 주입, 순환 참조 처리
- [Fine-tuning Annotation-based Autowiring with @Primary or @Fallback (Spring 공식 문서)](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired-primary.html)
  - 같은 타입의 빈이 여러 개일 때 우선순위 지정
- [Bean Scopes (Spring 공식 문서)](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html)
  - singleton, prototype, request 스코프와 스코프 프록시
- [Using the @Configuration annotation (Spring 공식 문서)](https://docs.spring.io/spring-framework/reference/core/beans/java/configuration-annotation.html)
  - `@Bean` 메서드 간 호출과 CGLIB 서브클래싱

## Lifecycle / 확장 지점

- [Customizing the Nature of a Bean (Spring 공식 문서)](https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html)
  - 초기화와 소멸 콜백, Aware 인터페이스, 콜백 호출 순서
- [Container Extension Points (Spring 공식 문서)](https://docs.spring.io/spring-framework/reference/core/beans/factory-extension.html)
  - BeanPostProcessor와 BeanFactoryPostProcessor의 차이와 사용 예
- [Graceful Shutdown (Spring Boot 공식 문서)](https://docs.spring.io/spring-boot/reference/web/graceful-shutdown.html)
  - 종료 시 처리 중인 요청을 마무리하는 설정

## Proxy / AOP

- [Proxying Mechanisms (Spring 공식 문서)](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html)
  - JDK 동적 프록시와 CGLIB 선택 기준, 내부 호출 문제
- [Aspect Oriented Programming with Spring (Spring 공식 문서)](https://docs.spring.io/spring-framework/reference/core/aop.html)
  - Aspect, Advice, Pointcut 등 AOP 개념
- [Dynamic Proxy Classes (Oracle Java 문서)](https://docs.oracle.com/javase/8/docs/technotes/guides/reflection/proxy.html)
  - `java.lang.reflect.Proxy`와 InvocationHandler 동작 방식
- [Using @Transactional (Spring 공식 문서)](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html)
  - 프록시 모드에서 `@Transactional`이 적용되지 않는 경우
