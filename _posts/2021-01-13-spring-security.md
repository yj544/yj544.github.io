---
title: "Spring Security Architecture"
categories:
  - spring
tags:
  - spring
  - spring_security
  
toc: true
toc_icon: "cog"
toc_sticky: true
---

### 인증과 접근 제어
어플리케이션의 보안은 크게 인증(authentication)과 인가(authorization or access controll) 두 가지가 핵심이다.
    - 인증(authentication) : 사용자가 누구인지 확인
    - 인가(authorization or access controll) : 사용자가 특정 리소스에 대한 접근 권한을 가지고 있는지 확인

#### Authentication
Spring Security에서 인증을 담당하는 핵심 클래스는 AuthenticationManager이다.  
```java
public interface AuthenticationManager {
	Authentication authenticate(Authentication authentication) throws AuthenticationException;
}
```

AuthenticationManager 클래스는 authenticate라는 함수 하나만 포함하고 있는데, 이 함수는 다음과 같이 동작한다.  
  - 전달받은 authentication 객체를 인증하고, 인증에 성공하면 권한 정보를 포함하는 Authentication 객체를 리턴한다.
  - 전달받은 authentication 객체가 유효하지 않으면 AuthenticationException을 던진다.
  - 잘 모르겠으면 null을 리턴한다.

ProviderManager는 AuthenticationManager의 대표적인 구현체이며 AuthenticationProvider 체인을 포함한다.
```java
public class ProviderManager implements AuthenticationManager, MessageSourceAware, InitializingBean {
	private static final Log logger = LogFactory.getLog(ProviderManager.class);
	private AuthenticationEventPublisher eventPublisher = new NullEventPublisher();
	private List<AuthenticationProvider> providers = Collections.emptyList();
  ...
  
	@Override
	public Authentication authenticate(Authentication authentication) throws AuthenticationException {
		...
		for (AuthenticationProvider provider : getProviders()) { //getProviders -> return this.providers;
			if (!provider.supports(toTest)) {
				continue;
			}
			if (logger.isTraceEnabled()) {
				logger.trace(LogMessage.format("Authenticating request with %s (%d/%d)",
						provider.getClass().getSimpleName(), ++currentPosition, size));
			}
			try {
				result = provider.authenticate(authentication);
				if (result != null) {
					copyDetails(authentication, result);
					break;
				}
			}
			catch (AccountStatusException | InternalAuthenticationServiceException ex) {
				prepareException(ex, authentication);
				// SEC-546: Avoid polling additional providers if auth failure is due to
				// invalid account status
				throw ex;
			}
			catch (AuthenticationException ex) {
				lastException = ex;
			}
		}
    // return 값이 null인 경우 부모의 authenticate를 호출
    if (result == null && this.parent != null) {
			// Allow the parent to try.
			try {
				parentResult = this.parent.authenticate(authentication);
				result = parentResult;
			}
			catch (ProviderNotFoundException ex) {
			}
			catch (AuthenticationException ex) {
				parentException = ex;
				lastException = ex;
			}
		}
    ...
    // Parent was null, or didn't authenticate (or throw an exception).
		if (lastException == null) {
			lastException = new ProviderNotFoundException(this.messages.getMessage("ProviderManager.providerNotFound",
					new Object[] { toTest.getName() }, "No AuthenticationProvider found for {0}"));
		}
    ...
}
```
ProviderManager의 authenticate 구현체에서는 AuthenticationProvider 체인을 통해 인증을 순차적으로 진행한다.  
아래에서 설명하겠지만 AuthenticationProvider는 support 함수가 지원된다.   
따라서 AuthenticationProvider 체인을 통해서 하나의 어플리케이션에서 여러개의 다른 인증 메커니즘을 지원할 수 있다.  
또한 AuthenticationProvider가 모두 null을 리턴하는 경우 부모 클래스의 authenticate를 호출한다.  
부모 클래스 조차도 null인 경우 ProviderNotFoundException를 던진다.  

AuthenticationProvider는 AuthenticationManager와 유사한 형태의 인터페이스다.  
한 가지 다른 점이 있다면 매개변수인 Authentication을 지원하는 지 확인할 수 있는 supports 함수가 있다는 것이다.  
```java
public interface AuthenticationProvider {
	Authentication authenticate(Authentication authentication) throws AuthenticationException;
  boolean supports(Class<?> authentication); //Class<? extends Authentication>
}
```

![AuthenticationManager와 ProviderMaanger의 구성](https://github.com/spring-guides/top-spring-security-architecture/raw/master/images/authentication.png)


#### AuthenticationManagerBuilder
Spring Security는 어플리케이션에 기본적인 AuthenticationManager를 빠르게 설정할 수 있도록 몇 가지 Configuration Helper를 제공한다.  
그 중에서도 일반적으로 사용되는 것은 AuthentikcationBuilder이다.  
AuthentikcationBuilder에서 커스텀으로 작성한 UserDetailService를 추가하여 in-memory, jdbc, ldap을 통한 인증을 설정할 수 있다.  

```java
@Configuration
public class ApplicationWebSecurity extends WebSecurityConfigurerAdapter{

	@Autowired
	public void initialize(AuthenticationManagerBuilder builder, DataSource dataSource) throws Exception {
		
		//jdbc
		builder.jdbcAuthentication()
				.dataSource(dataSource)
				.withUser("admin").password("secret").roles("ADMIN").and()
				.withUser("user").password("secret").roles("USER");
		
		//in-memory
		builder.inMemoryAuthentication()
				.withUser("admin").password("secret").roles("ADMIN").and()
				.withUser("user").password("secret").roles("USER");
		
		//ldap
		builder.ldapAuthentication()
				.userDnPatterns("uid={0},ou=people")
		        .groupSearchBase("ou=groups")
		        .contextSource()
		          .url("ldap://localhost:8080/dc=springframework,dc=org")
		          .and()
		        .passwordCompare()
		          .passwordEncoder(new BCryptPasswordEncoder())
		          .passwordAttribute("userPassword");
	}
}
```

initialize 함수에 @Autowired가 붙어있는데 이 경우 builder는 Global AuthenticationManager가 된다. 만댝 @Override를 붙여서 사용했다면 builder는 Local AuthenticationManager로 들어올 것이다. 


### SpringSecurityFilterChain
Spring Security는 Servlet Filter 기반으로 동작하며 FilterChainProxy 타입이다.   
또한 SpringBoot에서는 springSecurityFilterChain이라는 이름의 bean으로 Application Context에 등록되며 SecurityProperties.DEFAULT_FILTER_ORDER를 가진다. 
앞단에 FilterChainProxy로 권한을 위임하는 DelegatingFilterProxy가 있기 때문에 실제로는 아래와 같이 구성된다. 
![]()
  
하나의 요청에 하나만 매핑되는 Servlet과 달리 Servlet Filter는 여러 개가 수행되어야 할 수 있고, Filter의 처리 과정에서 요청 또는 응답의 내용이 변경될 수 있기 때문에 순서가 매우 중요하다.
다음은 반드시 지켜져야 하는 FilterChain의 순서이다.

- 1) ChannelProcessingFilter : 경우에 따라 다른 프로토콜로 리다이렉트해야 할 수 있음
- 2) SecurityContextPersistenceFilter : SecurityContext가 요청이 들어왔을 때 SecurityContextHolder를 준비하고 요청이 처리되기 전에 발생한 어떠한 변경사항이든 HttpSession에 복사할 수 있음
- 3) ConcurrentSessionFilter : it uses the SecurityContextHolder functionality and needs to update the SessionRegistry to reflect ongoing requests from the principal
- 4) 인증 처리 메커니즘 : UsernamePasswordAuthenticationFilter, CasAuthenticationFilter, BasicAuthenticationFilter .. 
- 5) SecurityContextHolderAwardRequestFilter
- 6) JaasApiIntegrationFilter
- 7) RememberMeAuthenticationFilter
- 8) AnonymousAuthenticationFilter
- 9) ExceptionTranslationFilter : 보호된 요청을 처리하는 동안 발생할 수 있는 예외의 기본 라우팅과 위임을 처리
     AuthenticationEntryPoint - 인증되지 않은 사용자가 보호된 HTTP 리소스를 요청하는 경우 호출
     AccessDeniedHandler - 사용자가 인증은 되어있으나 해당 리소스에 권한이 없는 경우 호출
- 10) FilterSecurityInterceptor : 권한 부여와 관련된 결정을 AccessDecisionManager에게 위임해 권한 부여 결정 및 접근 제어 결정을 쉽게 만듦
