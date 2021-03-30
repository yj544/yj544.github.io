---
title: "Spring IoC Container 그리고 Dependency Injection"
categories:
  - spring
tags:
  - spring
  
toc: true
toc_icon: "cog"
toc_sticky: true
---

### POJO
- POJO(란 )Plain Old Java Object)는 말 그대로 간단한 자바 오브젝트로, 마틴 파울러가 2000년에 컨퍼런스를 준비하며 만들어낸 용어이다. 
- EJB와 같은 복잡하고 제한이 많은 기술을 사용하는 것보다 자바의 단순한 오브젝트를 이용하는 것이 이점이 있다고 생각했으며, 이 단순한 오브젝트에 이름을 붙인 것이 POJO이다.
	```
	In the talk we were pointing out the many benefits of encoding business logic into regular java objects rather than using Entity Beans. 
	We wondered why people were so against using regular objects in their systems and concluded that it was because simple objects lacked a fancy name. 
	So we gave them one, and it's caught on very nicely.
	```
- POJO의 세 가지 규칙
    - 미리 지정된 클래스를 extends 하는 것
      - ex) public class Foo extends javax.servlet.http.HttpServlet { //... }
    - 미리 지정된 인터페이스를 implements 하는 것
      - ex) public class Bar implements javax.ejb.EntityBean { //... }
    - 미리 정의된 Annotation을 포함하는 것
      - ex) @javax.persistence.Entity public class Baz { //... }
      - POJO 기반의 프레임워크들은 기술적인 이슈로 Annotation을 사용하고 있다.

- Spring은 POJO 프로그래밍을 지원하는 프레임워크이다. 
- Spring은 POJO를 지원하기 위해 Ioc/DI, AOP, PSA 기술을 사용하며, 여기에 설계 정보를 더해 POJO가 어떻게 관계를 맺고 동작하는지를 판단한다. 

#### EJB
- EJB는 Enterprise Java Bean의 줄임말로 엔터프라이즈 시스템을 구현하기 위한 서버 사이드 컴포넌트 모델이다.
- 특징
	- 인스턴스 풀링 : 객체를 미리 생성하고 메모리에 올려두어 서비스의 안정성을 지원
	- 트랜잭션 처리 : 컨테이너에서 자동으로 모든 함수에 대한 트랜젝션 처리를 하며 이로 인한 안정적인 데이터 처리가 가능
	- 퍼시스턴스 관리 : JavaBeans의 상태를 메모리에서 자동으로 관리
- 종류
	- Session Bean : 클라이언트에 의해 호출될 수 있는 비즈니스 로직을 포함하며, Stateful Session Bean 및 Stateless Session Bean 두 가지 유형이 있다.
	- Message Driven Bean : 특정 메시지에 대한 리스너 역할을 한다. 
	- Entity Bean : JPA(Java Persistence API)로 대체되어 현재는 사용하지 않음
- JavaBeans : Java로 작성된 컴포넌트를 말하며 아래의 조건을 만족해야 한다.
	- 클래스는 직렬화 되어야 함
	- 기본 생성자를 가지고 있어야 함
	- 멤버변수에 get,set 혹은 표준 명명법을 따르는 함수를 통해 접근할 수 있어야 함
	- 필요한 이벤트를 처리하는 메서드를 포함해야 함
- 다만 복잡한 아키텍쳐, 낮은 생산성과 느린 성능, 불필요한 기술 복잡도 등의 문제가 있었으며 이는 POJO가 나오게 된 배경이 된다.

### Inversion of Control (제어의 역전)
Inversion of Control은 Dependency Injection이라고도 하며, 의존 관계를 직접 만드는 게 아니라 프레임워크로부터 주입을 받아 사용하는 방법을 말한다. 
프레임워크를 사용할 때 프레임워크가 제공하는 함수를 사용하는 것이 일반적이지만, 반대의 상황이 되면서 "제어가 역전되었다" 라고 표현하는 것이다.    

### Spring IoC Container
Spring IoC Container는 말 그대로 Spring에서 IoC를 지원하는 컨테이너를 말하는데 이중에서도 [BeanFactory](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/BeanFactory.html)는 Spring IoC Container의 최상위 계층에 있는 인터페이스이다.  

또한 이러한 BeanFactory를 상속받아서 BeanFactory가 제공하는 기능을 모두 제공하면서도 그 외 여러 기능을 추가적으로 제공하는 [ApplicationContext](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationContext.html)가 있다. 실제로 Spring IoC container라고 표현하는 대상은 ApplicationContext라고 보면 된다.  

```java
public interface ApplicationContext extends 
	EnvironmentCapable, //Environment 사용 가능
	ListableBeanFactory, //BeanFactory
	HierarchicalBeanFactory, //BeanFactory
	MessageSource, //메시지 다국화 지원
	ApplicationEventPublisher, //어플리케이션 이벤트를 리스너에게 보내주는 기능
	ResourcePatternResolver {
 
	@Nullable
	String getId();

	String getApplicationName();

	String getDisplayName();

	long getStartupDate();

	@Nullable
	ApplicationContext getParent();
	AutowireCapableBeanFactory getAutowireCapableBeanFactory() throws IllegalStateException;
}
```
ApplicationContext에는 몇 가지 구현체가 있는데, 독립형 어플리케이션을 위한 ClassPathXmlApplicationContext, FileSystemXmlApplicationContext 와 웹 어플리케이션을 위한 WebApplicationContext 가 포함된다.  

### Bean
Spring IoC Container에서 관리하는 객체를 말한다. 
Container는 적절한 형식으로 작성된 메타 데이터 설정을 읽어서 자신이 관리할 Bean에 대한 정보를 얻는다.  
이런 Bean에 대한 정보는 Spring 초기에 가장 전통적인 방식인 XML으로 작성했으며, Spring 2.5 이후부터는 Annotation 기반으로 작성할 수가 있다.   
따라서 @Service @Controller 등 클래스에 Annotation을 추가함으로써 보다 손쉽게 Bean 등록을 할 수 있어졌다.   

예를 들어 MemberController라는 클래스를 빈으로 등록하려는 경우, 아래와 같이 @Configuration이 달린 클래스에서 MemberController를 리턴하는 함수에 @Bean을 추가함으로써 등록할 수 있다.  
```java
@Configuration
public class ConfigBean {
	@Bean
	public MemberController memberController() {
		return new MemberController();
	}
}
```

그리고 SpringBoot를 쓰는 경우 빈으로 등록하고자 하는 클래스에 어노테이션을 추가함으로써 손쉽게 등록할 수도 있다.  
```java
@Controller
public class MemberController {
	//...
}
```

이는 SpringBoot로 프로젝트를 만드는 경우 기본적으로 추가되는 @SpringBootApplication 어노테이션 덕분인데, 해당 어노테이션을 따라가보면 @ComponentScan 이라는 어노테이션이 추가되어있는 것을 볼 수 있다.   
```java
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
}
```

즉, Main 함수가 존재하는 클래스가 포함된 패키지부터 시작하여 하위 클래스들 중 @Controller 와 같은 어노테이션이 추가된 클래스를 찾아 빈으로 등록한다.  

#### Scope
IoC Container에서는 Scope라는 Bean의 범위를 정해놓는다. Default Scope는 singleton이다.   
즉, 별도의 설정을 하지 않은 경우 Bean은 컨테이너 내에서 단일 객체로 존재하게 되며 이는 런타임 시 성능 최적화에 유리하다.  
- singleton : 하나의 Bean에 대해서 단 하나의 객체만 존재
- prototype : 하나의 Bean에 대해서 객체 수 제한이 없음 (계속 생성 가능)
- request : 하나의 Bean에 대해서 Http Request LifeCycle 내에 단 하나의 객체만 존재
- session : 하나의 Bean에 대해서 Http Session LifeCycle 내에 단 하나의 객체만 존재
- global session :  하나의 Bean에 대해서 Global Http Session LifeCycle 내에 단 하나의 객체만 존재

#### Lifecycle Interface
또한 Container는 Bean과 상호작용 할 수 있는 Lifecycle interface를 제공한다.  

```java
public class Member implements InitializingBean, DisposableBean{

	@Override
	public void afterPropertiesSet() throws Exception {
		//컨테이너에 빈이 정상적으로 설정된 이후 호출. 초기화 작업을 진행
	}

	@Override
	public void destroy() throws Exception {
		//빈이 포함된 컨테이너가 삭제될 때 호출.
	}
}
```

다만 공식 문서에서는 위와 같은 방법은 불필요하게 코드가 Spring에 결합되기 때문에 @PostConstruct 와 @PreDestroy 어노테이션을 사용할 것을 권하고 있다. 
```java

public class Member{

	@PostConstruct
	public void init() throws Exception {
		//컨테이너에 빈이 정상적으로 설정된 이후 호출. 초기화 작업을 진행
	}
	
	@PreDestroy
	public void destroy() throws Exception {
		//빈이 포함된 컨테이너가 삭제될 때 호출.
	}
}
```

### 의존성 주입
@Autowired 어노테이션을 추가하여 주입받을 수 있다.  
@Autowired 어노테이션은 생성자, Setter, 또는 변수에 추가할 수 있는데 생성자에 추가하는 경우는 다음과 같다.  
```java
@Controller
public class MemberController {
	private final MemberRepository repository;
	
	@Autowired //스프링 4.3부터 생략 가능
	public MemberController(MemberRepository repositry) {
		this.repository = repositry;
	}
}
```

MemberController가 @Controller 어노테이션을 통해 Bean으로 등록되었고, 해당 클래스의 생성자에 @Autowired가 붙어있으면서 매개변수로 repositry 객체를 받기 때문에 스프링에서 해당 매개변수에도 의존성을 주입해준다.  
사실 스프링 4.3부터는 위 예시의 생성자에 @Autowired를 붙이지 않아도 자동으로 의존성을 주입한다. 
클래스에 생성자가 하나만 존재하는데 파라미터로 전달받는 클래스가 빈으로 등록되어 있다면 의존성을 주입해주기 때문이다.  

마찬가지로 멤버변수와 Setter에 추가하는 방법도 위와 동일하게 @Autowired를 붙여주면 된다.  
```java
@Controller
public class MemberController {
	
	@Autowired
	private MemberRepository repository;

	private UserRepository userRepository;

	//Spring IoC container가 MemberController를 생성한 이후 
	//Setter 함수 호출하면서 파라미터에 userRepository 의존성을 주입해준다.
	@Autowired
	public void setUserRepository(UserRepository userRepository) {
		this.userRepository = userRepository;
	}
}
```

만약에 의존성을 주입받겠다고 @Autowired를 추가했으나 찾지 못하는 경우 아래와 같은 오류가 발생한다. 
```
Field managerRepository in com.demo.test.MemberController required a bean of type 'com.demo.test.ManagerRepository' that could not be found.
```

이런 여러가지 의존성 주입 방법 중 스프링에서는 생성자를 통한 의존성 주입 방법을 권장한다.
필수로 필요한 객체의 의존성을 주입받지 못한 경우 해당 클래스의 객체 또한 생성할 수 없도록 강제할 수 있기 때문에 더 좋다고 한다.

다만 생성자를 통한 의존성 주입 시 순환 참조가 발생할 수 있음을 알아야 한다.
Bean으로 등록되는 두 클래스 A,B가 있는데, A는 B를 B는 A를 사용한다면 순환 참조가 발생함에 따라 어플리케이션이 구동되지 않는다.
이런 상황이 발생하지 않도록 수정하거나, 반드시 그래야 한다면 Setter 또는 변수를 통한 의존성 주입을 받도록 해야한다.
