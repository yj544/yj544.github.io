---
title: "Spring AOP"
categories:
  - spring
tags:
  - springAOP
  - AOP-proxy
toc: true
toc_icon: "cog"
toc_sticky: true
---

### [Spring AOP](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop)
### AOP(Aspect Oriented Programming)
- AOP는 관점 지향 프로그래밍이라고 하며 OOP와는 다른 관점(Aspect)로 접근한다. 즉, OOP에서 모듈화의 핵심은 클래스라고 하면 AOP에서는 관점(Aspect)이다.
- 따라서 OOP에서 모듈화를 통해 발생하는 문제를 AOP의 Aspect를 통해 해결할 수 있다고 한다.
- OOP에서 모듈화를 통해 발생하는 문제를 AOP의 Aspect를 통해 해결할 수 있다고 한다.
- ![AOP Crosscutting](/image/aop.png)
  - 세 클래스가 있고, 각 클래스는 자신이 수행해야 할 핵심 기능(Process())을 가지고 있다.
  - 단, 핵심 기능을 수행하기 전/후로 부가 기능(beforeProcess(), afterProcess())이 있으며, 여기에서는 트랜잭션 처리와 같은 공통적인 작업을 진행한다.
  - 핵심 기능을 기준으로는 모듈화된 구성에 문제가 없지만, 부가 기능에서는 각각의 클래스가 중복된 로직을 수행하게 된다.
  - AOP는 이 부분에 중점을 두며 메인 기능 관점이 아닌 부가 기능 관점에서 여러 클래스가 공통적으로 필요한 작업들을 분리하여 재사용하는 것을 목표로 한다.
  - 여기서 여러 클래스에 걸쳐 공통적으로 필요한 작업(ex. 로깅, 트랜잭션, 보안)을 Crosscutting Concerns 라고 부르기도 한다. 

### AOP 주요 용어 
- ``Aspect``
  - 여러 곳에서 공통으로 쓰이는 부가 기능을 의미한다. Spring AOP에서 Aspect는 @Aspect Annotation으로 지원된다.

- ``JoinPoint``
  - 함수가 호출되거나 예외를 처리하는 등 프로그램이 실행되는 동안의 어떤 시점을 의미한다. 
  - Spring AOP에서 JoinPoint는 항상 함수 호출을 의미한다.

- ``Advice``
  - 특정 JoinPoint에서 Aspect에 의해 수행되는 Action을 의미하며, 실질적으로 수행되는 부가 기능 구현체이다. 
  - Spring을 포함한 대부분의 AOP Framework에서 advice는 Interceptor로 구현된다.
  - Advice는 다음과 같은 타입이 있다. 
    - before : JoinPoint가 실행되기 전의 advice. 예외가 발생하지 않는 한 JoinPoint의 실행을 방지할 수는 없다.
    - after returning : JoinPoint가 예외 없이 정상적으로 완료된 이후에 수행되는 advice.
    - after throwing : JoinPoint 수행 중 예외가 발생한 경우 수행되는 advice.
    - after (finally) : JoinPoint의 결과에 상관 없이 종료만 되면 무조건 수행되는 advice.
    - around : JoinPoint를 둘러싼 around로 가장 강력한 advice이다. 함수 호출 전/후에 특정 동작을 수행할 수 있고, JoinPoint를 수행하거나 자체 적인 리턴값을 반환하거나 예외를 던지거나 선택할 수 있다. 단, arond의 경우 반드시 proceed() 함수를 호출해야 실제 함수가 호출된다. 

- ``Pointcut``
    - JoinPoint에 대한 상세 정의이다. Advice는 Pointcut 표현식과 연관되어 Pointcut과 일치하는 모든 JoinPoint에서 실행된다. 
    - JoinPoint를 Pointcut 표현식으로 매칭시키는 것은 AOP의 핵심이며, Spring에서는 기본적으로 AspectJ Pointcut을 사용한다. 
    
- ``Introduction``
    - 대상 클래스에 코드 변경 없이 새로운 함수나 필드를 추가하는 기능을 말한다. 예를 들어 Bean에 IsModified 인터페이스를 구현하여 캐싱을 쉽게 처리할 수 있다.
- ``Target object``
    - Aspect가 적용될 대상 객체로 Advised object라고 불리기도 한다. Spring AOP는 런타임 프록시로 구현되었기 때문에 Target object 또한 프록시된 객체이다.
- ``AOP proxy``
    - Aspect를 지원하기 위해 AOP 프레임워크에 의해 생성된 객체. Spring 프레임워크에서 AOP proxy는 JDK dynamic proxy 또는 CGLib proxy이다. 
- ``Weaving``
    - Aspect를 다른 애플리케이션 타입이나 객체에 연결시켜서 Advised object를 만드는 것.
    - Weaving은 컴파일 시점, 로드 시점, 런타임에 수행할 수 있는데 Spring AOP는 런타임 시점에 Weaving을 수행한다. 

### AOP Proxy
- CGLib의 경우 리패키징하여 spring-core 라이브러리 안에 포함된 형태임
- Spring AOP는 프록시 클래스가 하나 이상의 인터페이스를 구현한 경우 JDK dynamic proxy를 사용하고 그 외의 경우 CGLib 프록시를 사용함
- 프록시 될 대상 객체가 어떤 인터페이스도 구현하지 않은 경우 CGLib 프록시가 사용됨 
- 인터페이스를 구현한 경우에도 CGLib를 사용할 수 있지만 다음을 고려해야함
  - CGLib를 사용하면 런타임에 생성되는 하위 클래스에서 재정의할 수 없기 때문에 final 함수를 권장하지 않는다.
  - CGLib 프록시 인스턴스는 Objenesis를 통해 생성되므로 Spring 4.0을 기준으로 프록시 객체 생성자는 더이상 호출되지 않는다. 
  - JVM이 생성자 bypassing을 허용하지 않는 경우에만 두 번의 생성자가 호출될 수 있음

### Spring AOP
- IoC Container와 함께 사용되어 Aspect는 일반적인 Bean이 정의되는 방식으로 설정된다.  
- bean으로 등록된 대상의 함수 호출에 대한 JoinPoint만 지원한다. 
- 다른 AOP Framework의 목적이 완벽한 AOP를 제공하는 것이라면, Spring AOP의 목표는 AOP를 Spring IoC와 연동하여 엔터프라이즈 애플리케이션에서 가장 흔하게 발생하는 문제(중복 코드, 프록시 클래스 작성 번거로움, 객체간 결합도 증가)를 해결하는 것이다. 
- 기본적으로 standard JDK dynamic proxies를 사용한다. 
- AspectJ에서 쓰는 Annotation @AspectJ 을 동일하게 사용하며 Spring의 AspectJ는 PointCut 파싱 및 매칭에 대해서 AspectJ5 일부 라이브러리를 사용한다. 

### 적용해보기 
##### 1) dependency 추가 
  ``` xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
  ```
##### 2) Annotation 추가
  - Spring AOP를 활성화하기 위해서는 ``@EnableAspectJAutoProxy`` 애노테이션을 추가해야 한다.
  ```java
  @SpringBootApplication
  @EnableAspectJAutoProxy
  public class AopSampleApplication {
    public static void main(String[] args) {
      SpringApplication.run(AopSampleApplication.class, args);
    }
  }  
  ```
  - @EnableAspectJAutoProxy 애노테이션은 내부적으로 @Import 애노테이션을 쓰기 때문에 @Configuration 애노테이션과 함께 선언되어야 한다. 
  ```java
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Import(AspectJAutoProxyRegistrar.class)
  public @interface EnableAspectJAutoProxy {
    //...
  }  
  ```
  - @SpringBootApplication 도 내부적으로는 @Configuration 이기 때문에 가능하다.
  ```java
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Inherited
  @SpringBootConfiguration
  @EnableAutoConfiguration
  @ComponentScan(excludeFilters = { 
    @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
    @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
  public @interface SpringBootApplication {
    //...
  }

  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Configuration
  @Indexed
  public @interface SpringBootConfiguration {
    //...
  }  
  ```

##### 3) Aspect 선언
  - @Aspect 애노테이션을 추가한다.
  - @Aspect 애노테이션은 Bean으로 자동 등록되는 애노테이션이 아니기 때문에 @Configuration, @Component 등의 애노테이션을 함께 추가해야 한다.
  ```java
  @Component
  @Aspect
  @Slf4j
  public class ServiceAspect {

  }
  ```

  ##### 4) Pointcut 정의
    - Pointcut을 통해 Advice가 실행되어야 할 JoinPoint를 지정한다.
    ```java
    @Component
    @Aspect
    @Slf4j
    public class ServiceAspect {
      
      //com.example.aop.target.UserService 클래스에서 호출되는 모든 함수에 대해 Pointcut을 지정
      @Pointcut("execution(* com.example.aop.target.UserService.*(..))")
      private void pointcut1() {}

      //com.example.aop.target.service 패키지 및 하위 패키지에 대해 Pointcut을 지정
      @Pointcut("within(com.example.aop.target.service..*)")
      private void pointcut2() {}   
    }
    ```
    - Pointcut 정의 
      - execution : 함수 호출에 매칭시킴
        - 형식 : execution({접근제어자(생략 가능)} {반환값} {함수명}({파라미터}))
        ```java
        @Pointcut("execution(public com.example.aop.target.User com.example.aop.target.UserService.findById(String))")

        //wildcard matches : 첫 번째 *는 모든 리턴 값을 의미, 두 번째 *는 모든 함수를 의미, 세 번째 (..)는 모든 파라미터를 의미 
        @Pointcut("execution(* com.example.aop.target.UserService.*(..))")
        ```

      - within : 특정 타입에 매칭시킴
        ```java
        //UserService 클래스에서 호출되는 모든 함수
        @Pointcut("within(com.example.aop.target.UserService)")

        //target 패키지에 속하는 모든 클래스에서 호출되는 모든 함수 
        @Pointcut("within(com.example.aop.target.*)")

        //target 패키지 및 하위 패키지에 속하는 모든 클래스에서 호출되는 모든 함수
        @Pointcut("within(com.example.aop.target..*)")
        ```

      - this / target
        - this : 객체가 주어진 타입에 대한 객체인 경우 매칭시킴
        - target : 객체가 주어진 타입에 대한 타겟 객체인 경우의 JoinPoint에 매칭시킴 
        - this는 CGLib 기반 프록시 생성 시 사용하고, target은 JDK 기반 프록시 생성 시 사용한다. 
        ```java
        public class GroupMemberService implements MemberService {
          //...
        }

        public class UserService {
          //...
        }
        ```
        - GroupMemberService가 하나 이상의 인터페이스를 구현하고 있기 때문에 JDK 기반 프록시를 사용한다.
        - 프록시 된 객체가 Proxy 클래스의 객체가 되고, MemberService 인터페이스를 구현하기 때문에 다음과 같이 Pointcut을 정의할 수 있다.
          ```java
          @Pointcut("this(com.example.aop.target.MemberService)")
          ```
        - UserService는 어떤 인터페이스도 구현하고 있지 않기 때문에 CGLib 기반 프록시를 사용한다.
        - 프록시 된 객체는 UserService 또는 하위 클래스가 될 것이기 때문에 다음과 같이 Pointcut을 정의할 수 있다.
          ```java
          @Pointcut("this(com.example.aop.target.UserService)")
          ```

      - args : 주어진 arguments의 형식을 포함하는 함수 호출에 매칭시킴
        ```java
        @Pointcut("execution(* *..find*(Long))")
        @Pointcut("execution(* *..find*(Long,..))")
        ```

      - @target : 주어진 타입에 대한 애노테이션을 포함하고 있는 클래스에 매칭시킴
        ```java
        //@Repository 애노테이션을 가지는 클래스를 포함
        @Pointcut("@target(org.springframework.stereotype.Repository)")
        ```
      - @args : 함수 arguments로 특정 애노테이션이 포함된 타입을 전달받는 경우 매칭시킴     
      - @within : 주어진 애노테이션 타입 이내의 JoinPoint를 매칭시킴
        ```java
        //아래 두 Pointcut은 동일한 의미를 가짐
        @Pointcut("@within(org.springframework.stereotype.Repository)")
        @Pointcut("within(@org.springframework.stereotype.Repository *)")
        ```

      - @annotation : 특정 애노테이션을 포함한 대상에 매칭시킴
        - @target과 유사해보이는데 @target은 대상 클래스라면 @annotation은 대상(함수가 될 수도 있음)인 게 다른 것 같음
    - Pointcut 조합
      - &&, ||, ! 연산자를 통한 Pointcut 조합 가능
      ```java
      @Pointcut("@target(org.springframework.stereotype.Repository)")
      public void repositoryMethods() {}

      @Pointcut("execution(* *..create*(Long,..))")
      public void firstLongParamMethods() {}

      @Pointcut("repositoryMethods() && firstLongParamMethods()")
      public void entityCreationMethods() {}
      ```

##### 5) Advice 정의
  ```java
  @Component
  @Aspect
  @Slf4j
  public class ServiceAspect {
    
    //com.example.aop.target.UserService 클래스에서 호출되는 모든 함수에 대해 Pointcut을 지정
    @Pointcut("execution(* com.example.aop.target.UserService.*(..))")
    private void pointcut1() {}

    //com.example.aop.target.service 패키지 및 하위 패키지에 대해 Pointcut을 지정
    @Pointcut("within(com.example.aop.target.service..*)")
    private void pointcut2() {}

    //Before Advice에 대해 pointcut1 Pointcut을 지정
    @Before("pointcut1()")
    public void beforeLog(JoinPoint joinPoint) {
      log.info("start");
    }

    //After Advice에 대해 pointcut2 Pointcut을 지정
    //pointcut이 다른 패키지에 위치하는 경우 패키지를 포함한 전체 이름으로 작성
    @After("com.example.aop.ServiceAspect.pointcut2()")
    public void afterLog(JoinPoint joinPoint) {
      log.info("end");
    }      
  }
  ```
  - Pointcut을 Advice와 함께 정의할 수 있음
  ```java
    @After("within(com.example.aop.target.service..*)")
    public void afterLog(JoinPoint joinPoint) {
      log.info("end");
    }  
  ```
