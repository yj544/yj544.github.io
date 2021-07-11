---
title: "Unit Test"
categories:
  - spring
tags:
  - junit
  - spring-test
  
toc: true
toc_icon: "cog"
toc_sticky: true
---

### Dependency 
- junit과 spring-test
```java
<dependencies>
	<dependency>
		<groupId>junit</groupId>
		<artifactId>junit</artifactId>
		<version>4.13.1</version>
		<scope>test</scope>
	</dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>5.2.3.RELEASE</version>
      <scope>test</scope>
    </dependency>
</dependencies>
```

- spring-boot 프로젝트를 생성하면 기본적으로 추가되는 spring-boot-starter-test dependency에는 모두 포함되어 있음
```java
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
	<exclusions>
		<exclusion>
			<groupId>org.junit.vintage</groupId>
			<artifactId>junit-vintage-engine</artifactId>
		</exclusion>
	</exclusions>
</dependency>
```

- 따라서 spring-boot 프로젝트 생성 시 다음과 같이 test 디렉토리를 포함한 프로젝트가 생성됨
```java
unit-test
ㄴsrc/main/java
ㄴsrc/main/resources
ㄴsrc/test/java
  ㄴcom.demo.test
    ㄴUnitTestApplicationTests.java
```

### [SpringBootTest](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing)
- spring-boot-starter-test에는 다음 라이브러리가 포함됨
	- JUnit5
	- Spring Test & Spring Boot Test
	- AssertJ
	- Hamcrest
	- Mockito
	- JSONassert
	- JsonPath

#### @SpringBootTest 
- spring-boot-test에서 제공하는 애노테이션으로 테스트에 사용되는 ApplicationContext를 쉽게 생성 및 조작할 수 있음
- JUnit4를 사용하는 경우 @SprintBootTest 애노테이션은 반드시 @RunWith(SpringRunner.class) 와 함께 사용해야함

##### classes
- 테스트에 사용할 빈을 생성할 수 있음
```java
@SpringBootTest(classes = {UserService.class})
public class ClassTest {
    @Autowired
    private UserService userService;
}
```

##### @TestConfiguration
- 기존의 설정을 커스터마이징하려는 경우 @TestConfiguration을 사용할 수 있음
- TestConfiguration은 ComponentScan 과정에서 생성되며 TestConfiguration이 속한 테스트가 실행될 때 빈을 생성하여 등록됨
	```java
	@SpringBootTest
	public class ClassTest {
		//...
		@TestConfiguration
		public static class TestConfig {
			@Bean
			public RestTemplate restTemplate() {
				return new RestTemplate() {
					@Override
					public <T> T getForObject(String url, Class<T> responseType, Object... uriVariables) throws RestClientException {
						System.out.println("Good");
						if (responseType == String.class) {
							return (T) "Good";
						} else {
							throw new IllegalArgumentException();
						}
					}
				};
			}
		}
	}
	```
- 만약 TestConfiguration이 외부 파일에 있거나, classes로 클래스를 제한하여 ComponentScan에 TestConfiguration이 포함되지 않은 경우 @Import를 통해 추가할 수 있음
	```java
	@SpringBootTest
	@Import(TestConfig.class)
	public class ClassTest {
		//...
	}	

	@TestConfiguration
	public static class TestConfig {
		//...
	}

	```

##### webEnvironment	
- MOCK : WebApplicationContext를 로드하여 내장 Servlet Container가 아닌 Mock 서블릿 환경을 구성함, @AutoConfigureMockMvc 애노테이션을 함께 사용하면 별다른 설정 없이 MockMvc 테스트 가능
- RANDOM_PORT : EmbeddedWebApplicationContext를 로드하여 서블릿 환경을 구성함, 랜덤 포트로 구동
- DEFINED_PORT : EmbeddedWebApplicationContext를 로드하여 서블릿 환경을 구성함, 지정된 포트로 구동
- NONE : SpringApplication을 통해 ApplicationContext를 로드하며 서블릿 환경은 구성하지 않음
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Import(TestConfig.class)
public class ClassTest {
	//...
}
```

##### TestRestTemplate
- RestTemplate에 비해 실패에 관대하다? 400 ~ 500 에러에 있어서 exception을 던지지 않고 대신 response entity에서 getstatuscode로 확인할 수 있게 한다
- RestTemplate을 상속받은 게 아니고 포함하고 있는 형태로 RestTemplate의 기능 외 추가적인 기능을 제공
	```java
	public class TestRestTemplate {

		private final RestTemplateBuilder builder;
		private final HttpClientOption[] httpClientOptions;
		private final RestTemplate restTemplate;
		//...
	}
	```
- TestRestTemplate는 Servlet Container를 사용하여 테스트를 수행하는 반면 비슷한 역할을 하는 MockMvc는 Servlet Container 없이 SpringMVC 동작을 재현할 수 있도록 지원하는 클래스로 요청을 DispatcherServlet에 직접 보냄

