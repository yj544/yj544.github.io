---
title: "Spring AOP"
categories:
  - spring
tags:
  
toc: true
toc_icon: "cog"
toc_sticky: true
---

### [Spring AOP](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop)
#### AOP(Aspect Oriented Programming)
- AOP는 관점 지향 프로그래밍이라고 하며 OOP와는 다른 관점(Aspect)로 접근한다. 즉, OOP에서 모듈화의 핵심은 클래스라고 하면 AOP에서는 관점(Aspect)이다.
- 따라서 OOP에서 모듈화를 통해 발생하는 문제를 AOP의 Aspect를 통해 해결할 수 있다고 한다.
- OOP에서 모듈화를 통해 발생하는 문제를 AOP의 Aspect를 통해 해결할 수 있다고 한다.
- ![AOP Crosscutting](/image/aop.png)
  - 세 클래스가 있고, 각 클래스는 자신이 수행해야 할 핵심 기능(Process())을 가지고 있다.
  - 단, 핵심 기능을 수행하기 전/후로 부가 기능(beforeProcess(), afterProcess())이 있으며, 여기에서는 트랜잭션 처리와 같은 공통적인 작업을 진행한다.
  - 핵심 기능을 기준으로는 모듈화된 구성에 문제가 없지만, 부가 기능에서는 각각의 클래스가 중복된 로직을 수행하게 된다.
  - AOP는 이 부분에 중점을 두며 메인 기능 관점이 아닌 부가 기능 관점에서 여러 클래스가 공통적으로 필요한 작업들을 분리하여 재사용하는 것을 목표로 한다.
  - 여기서 여러 클래스에 걸쳐 공통적으로 필요한 작업(ex. 로깅, 트랜잭션, 보안)을 Crosscutting Concerns 라고 부르기도 한다. 

#### AOP 주요 용어 
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
    Aspect가 적용될 대상 객체로 Advised object라고 불리기도 한다. Spring AOP는 런타임 프록시로 구현되었기 때문에 Target object 또한 프록시된 객체이다.
- ``AOP proxy``
    Aspect를 지원하기 위해 AOP 프레임워크에 의해 생성된 객체. Spring 프레임워크에서 AOP proxy는 JDK dynamic proxy 또는 CGLIB proxy이다. 
- ``Weaving``
    Aspect를 다른 애플리케이션 타입이나 객체에 연결시켜서 Advised object를 만드는 것.
    Weaving은 컴파일 시점, 로드 시점, 런타임에 수행할 수 있는데 Spring AOP는 다른 순수 Java AOP 프레임워크와 같이 런타임 시점에 Weaving을 수행한다. 

#### Spring AOP
- IoC Container와 함께 사용되어 Aspect는 일반적인 Bean이 정의되는 방식으로 설정된다.  
- bean으로 등록된 대상의 함수 호출에 대한 JoinPoint만 지원한다. 
- 다른 AOP Framework의 목적이 완벽한 AOP를 제공하는 것이라면, Spring AOP의 목표는 AOP를 Spring IoC와 연동하여 엔터프라이즈 애플리케이션에서 가장 흔하게 발생하는 문제(중복 코드, 프록시 클래스 작성 번거로움, 객체간 결합도 증가)를 해결하는 것이다. 
- 기본적으로 standard JDK dynamic proxies를 사용한다. 
- AspectJ에서 쓰는 Annotation @AspectJ 을 동일하게 사용하며 Spring의 AspectJ는 PointCut 파싱 및 매칭에 대해서 AspectJ5 일부 라이브러리를 사용한다. 

#### 적용해보기 
- 1) dependency 추가 
  		<dependency>
	        <groupId>org.springframework.boot</groupId>
	        <artifactId>spring-boot-starter-aop</artifactId>
	    </dependency>
  
  