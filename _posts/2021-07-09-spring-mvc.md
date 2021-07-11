---
title: "Spring MVC"
categories:
  - spring
tags:
  - springMVC
toc: true
toc_icon: "cog"
toc_sticky: true
---

### Servlet & Servlet Container
#### Servlet 
- (주로)웹 서버를 지원하는 Java Class(javax.servlet)이며, 요청-응답 모델 기반으로 동작
- Servlet Lifecycle (Servlet Container에 의해 제어됨)
  - 객체 생성 : 요청 발생 및 Servlet 매핑 후 Servlet 객체가 없는 경우 클래스 로딩 및 객체 생성
  - init() : 객체 초기화를 위한 init 함수 호출, init 함수는 Servlet 생명 주기 중 단 한 번만 호출됨
  - service() : Servlet Container가 쓰레드를 생성하고 Servlet의 service() 함수를 호출, service() 함수 내에서 요청에 따라 적절한 함수(doGet, doPost, doPut, doDelete)를 호출
  - destroy() : Servlet이 제거될 때 한 번만 호출되며 DB 연결 해제, 백그라운드 쓰레드 중지, 기타 작업 정리를 수행. destory 함수 호출 이후에 Servlet 객체는 GC 수집 대상이 됨
  - GC에 의한 객체 삭제
  
#### Servlet Container 
- Servlet 구동 환경을 의미하며 Servlet의 Lifecycle을 제어함.
- JSP Container 및 Servlet Container를 합쳐 Web Container라고 함

### [Spring MVC](https://docs.spring.io/spring-framework/docs/3.2.x/spring-framework-reference/html/mvc.html)
![spring mvc](/image/springmvc.png)
- Spring MVC는 DispatcherServlet을 중심으로 설계되었으며 다른 웹 MVC Framework와 마찬가지로 request-driven 방식
- DispatcherServlet은 handler mapping, view resolution, locale/theme resolution 및 요청 전달 등 핵심 기능을 수행함
- WebApplicationInitializer는 Servler Container를 초기화할 때 필요한 코드 기반 설정을 감지 및 설정하는 Spring MVC의 인터페이스임

#### Bean Types in the WebApplicationContext
- Spring의 DispatcherServlet은 요청을 처리하고 적절한 뷰로 렌더링할 때 몇 가지 빈을 사용함

- HandlerMapping : 요청을 핸들러, 전처리기, 후처리기와 매핑함
- HandlerAdapter : DispatcherServlet으로부터 요청 처리를 위임받아 각 핸들러에 알맞는 함수를 호출하여 요청을 처리함
- HandlerExceptionResolver : 요청이 핸들러에서 처리되는 도중 예외가 발생했을 때 처리를 담당함
- ViewResolver : ModelAndView 객체에 담긴 뷰 이름을 통해 실제 View 객체를 찾거나 생성 후 반환함
- LocaleResolver : 클라이언트가 사용하는 Locale을 추출함
- MultipartResolver : 파일 처리를 위해 Multipart 요청을 파싱하며 MultipartHttpServletRequest로 래핑하여 전달함
- ThemeResolver
- FlashMapManager 

#### DispatcherServlet
- HttpServlet을 상속받은 Servlet이며 web.xml에 정의되어 있음
- 각 DispatcherServlet은 자신만의 WebApplicationContext를 가짐

#### DispatcherServlet 처리 순서 
- 1) 다른 핸들러 및 요소가 사용할 수 있도록 WebApplicationContext를 요청의 attribute 형태로 바인딩 시킴 
	- 기본 값은 DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE
		```java
		public static final String WEB_APPLICATION_CONTEXT_ATTRIBUTE = DispatcherServlet.class.getName() + ".CONTEXT";
		```
- 2) Locale 처리가 필요한 경우 Locale Resolver를 바인딩
- 3) 사용될 테마를 결정해야 할 경우 Theme Resolver를 바인딩
- 4) 요청이 Multipart인 경우 MultipartResolver를 바인딩
- 5) 적절한 핸들러를 찾아 핸들러와 연결된 체인(전처리, 핸들러, 후처리)을 실행 
- 6) 모델이 반환되면 뷰를 랜더링함
- 7) 도중에 예외가 발생하는 경우 HandlerExceptionResolver를 통해 커스텀 동작을 수행

#### @Controller
- Spring MVC의 기본 핸들러는 @Controller 및 @RequestMapping 애노테이션을 기반으로 함
- 요청을 처리하는 각 함수는 기본적으로 Model을 전달받고 View 이름을 리턴
- 리턴받은 View 이름을 통해 ViewResolver가 View를 찾아 랜더링
	```java
	@Controller
	public class HelloWorldController {

		@RequestMapping("/helloWorld")
		public String helloWorld(Model model) {
			model.addAttribute("message", "Hello World!");
			return "helloWorld";
		}
	}
	```
- 리턴 값이 View가 아닌 데이터인 경우 @ResponseBody 애노테이션을 사용할 수 있으며, 이 때는 리턴 값이 Http Response Body로 전달됨

#### @RestController
- Rest API 서비스에서 사용하는 @RestController는 @Controller와 @ResponseBody가 함께 존재하는 형태임
	```java
	@Target(ElementType.TYPE)
	@Retention(RetentionPolicy.RUNTIME)
	@Documented
	@Controller
	@ResponseBody
	public @interface RestController {
		@AliasFor(annotation = Controller.class)
		String value() default "";

	}
	```