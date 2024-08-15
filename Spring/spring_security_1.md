# spring security - architecture

spring security는 servlet과 reative 기반 애플리케이션을 지원한다. 나는 spring MVC 웹 애플리케이션을 구현하고 있으니 servlet 기반 spring security의 구성을 파악하려 한다.

## Servlet Filter

servlet 기반 spring security는 Servlet Filter를 기반으로 한다. 그럼 어떻게 Servlet Filter로 spring security의 기능이 적용되는걸까?

<p align="center">
<img src="../img/filterchain.png" width="30%" height="50%" align="center">
</p>

클라이언트로부터 HttpServletRequest가 오면 Servlet Container는 그 요청을 받아서 처리할 수 있는 FilterChain에 요청을 넘긴다. 이 FilterChain은 요청을 처리하기 위해 필요한 여러개의 filter와 하나의 servlet(spring MVC의 경우 DispatcherServlet의 인스턴스)을 가질 수 있다.

이 FilterChain은 강력한 기능을 수행할 수 있는데,

- 첫째로는 다음 Filter(downstream filter)나 servlet이 호출되는 것을 막을 수 있다. 이후의 처리를 막고 Filter가 HttpServletResponse를 작성할 수 있다.
- 둘째로는 다음 Filter 또는 servlet이 사용하는 HttpServletRequest, HttpServletResponse를 수정할 수 있다.

Filter가 요청과 응답을 중간에서 처리하거나 수정할 수 있기 때문에 인증, 인가 등의 보안이 필요한 작업에서 Servlet Filter가 사용된다.

그럼 이제부터 FilterChain 내부를 자세히 들여다보자

## DelegatingFilterProxy

<p align="center">
<img src="../img/delegatingfilterproxy.png" width="30%" height="50%" align="center">
</p>

DelegatingFilterProxy는 javax.servlet.Filter의 구현체로 Servlet Container와 Spring ApplicationContext의 가교역할을 하는 필터이다. 그럼 왜 이 둘을 잇는 역할을 하는걸까?

Servlet Container는 사용자의 요청과 응답을 처리하고 Filter와 Servlet을 생성 및 관리 등을 한다. Servlet Filter들은 web.xml에 등록해서 servlet container가 직접 관리를 한다. 하지만 spring에서 관리하는 Filter Bean은 servlet container가 알 수 없기 때문에 web.xml에 직접 등록해야한다. Spring Bean의 자동 의존 주입(web.xml에 일일히 다 등록하지 않아도 됨!), AOP, Transaction, 보안 기능을 쉽고 편하게 사용할 수 있는 장점들을 누리기 위해 Spring이 관리하는 Filter를 호출할 때에는 DelegatingFilterProxy를 활용한다.

DelegatingFilterProxy는 안에 구현된 로직은 없다. 단순히 Spring Bean에 작업을 위임하는 기능만 할 뿐이다. Servlet Container는 여전히 요청을 받아 처리할 수 있는 FilterChain에 넘겨주는 동일한 작업을 한다. 사용하는 Filter중 spring이 관리하는 Filter가 포함되어있는 경우 DelegatingFilterProxy를 호출해서 Spring Bean이 관리하는 Bean Filter0에 작업을 위임한다(넘겨준다).

DelegatingFIlterProxy을 사용할 때 이점 중 하나인 Delaying look up은 서블릿 컨테이너가 필터를 등록할 때 필터를 조회하지 않고 실제로 필터가 사용될 때 조회를 하는 것을 말한다.

Servlet Container가 필터를 등록할 때는 Conainer를 실행하기 전이다. Spring의 Bean(필터)은 ContextLoaderListener가 ApplicationContext를 초기화 할 때 로드된다. spring이 Bean을 로드하기도 전에 Servlet Container에서 필터를 등록하는 상황이 벌어질 수도 있다. 각각 필터를 로드하는 시기가 다른 문제를 DelegatingFilterProxy를 통해서 지연할 수 있다.

## FilterChainProxy

<p align="center">
<img src="../img/filterchainproxy.png" width="60%" height="100%" align="center">
</p>

관련된 필터들을 묶은 chain을 관리하고 요청에 따라 chain을 선택하고 순서대로 Filter를 적용한다. DelegatingFilterProxy으로부터 호출되며 Spring Bean이 관리하는 Bean이다.

## SecurityFilterChain

<p align="center">
<img src="../img/securityfilterchain.png" width="60%" height="100%" align="center">
</p>
Security Filter들을 묶어놓은 Chain이다. SecurityFilterChain에 속한 Security Filter들은 보통 Bean인데 DelegatingFilterProxy가 아닌 FilterChainProxy에 등록이 되어있다. 그 이유로는

- 첫째 FilterChainProxy가 모든 spring security servlet 기반 기능의 시작점이기 때문이다.
- 둘째 그렇기 때문에 security servlet의 핵심 객체인 FilterChainProxy에 debug point를 걸어두면 요청을 어떻게 처리하고 있는지 파악하는데 도움이 된다.

또 Servlet Container에서는 Filter를 URL만으로 호출하는데 FilterChainProxy에서는 RequestMatcher 인터페이스로 HttpServletRequest의 모든 요소를 기반으로 호출할 수 있다.

FilterChainProxy는 SecurityFilterChain을 선택할 때 가장 처음 일치하는 SecurityFilterChain을 호출한다. 자세한 정보는 [이곳](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-print-filters)을 참조하자

## Security Filters

SecurityFilterChain을 이루는 Security Filter들은 다양한 목적으로 사용될 수 있다. Filter들이 적용되는 순서가 있다. 굳이 알필요는 없지만 분명 알아야할 때는 있을 것이다.

```
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(Customizer.withDefaults())
            .authorizeHttpRequests(authorize -> authorize
                .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults())
            .formLogin(Customizer.withDefaults());
        return http.build();
    }

}
```

위의 코드 스니펫을 보면 여러가지 Filter들(csrf, httpBasic...)에 관해 설정한 것을 확인할 수 있다. 하지만 Filter는 작성한 순서와는 상관없이 적용된다. 적용되는 순서를 알고 싶다면 [이 클래스 구현](https://github.com/spring-projects/spring-security/blob/6.3.1/config/src/main/java/org/springframework/security/config/annotation/web/builders/FilterOrderRegistration.java)을 참조하거나(default 순서) Security Filter에 대한 정보를 확인할 수 있는 log를 확인할 수 있다.

## Filter Chain에 Custom Filter를 적용하기

## Handling Security Exception

<p align="center">
<img src="../img/exceptiontranslationfilter.png" width="60%" height="100%" align="center">
</p>

1. 사용자가 리소스에 접근할 때 AccessDeniedException 또는 AuthenticationException이 발생한다. 예외가 발생할 때 ExceptionTranslationFilter가 예외를 처리한다.

- AccessDeniedException : 접근 거부
- AuthenticationException : 인증이 필요한 경우 또는 인증 실패

2. AuthenticationException 처리
   ExceptionTranslationFilter가 이 예외를 처리한다.

   - SecurityContextHolder가 지워지고(누가 인증하고 있는지를 저장하는 곳)
   - HttpServletRequest가 저장된다(인증 성공 시 재요청을 보내기 위함)
   - AuthenticationEntryPoint로 자격 증명을 요청하는 HTTP응답을 보내는데 사용된다. (ex. 로그인 페이지로 리다이렉트)

3. AccessDeniedException
   사용자가 특정 리소스에 접근 권한이 없음을 알리고 그에 따른 처리를 한다.AccessDeniedHandler가 이 예외를 처리하기위해 호출된다.

만약 2,3번의 예외가 일어나지 않는다면 ExceptionTranslationFilter는 아무것도 하지 않음.

## Saving Requests Between Authentication

인증되지 않은 사용자가 '/user/history'라는 리소스에 접근했다고 가정하자. 이 리소스는 회원들만 접근할 수 있기 때문에 사용자는 로그인을 해야한다. 로그인 성공 후, 사용자가 접근하려고 했던 '/user/history' 리소스에 바로 리다이렉트할 수 있도록 HttpServletRequest를 어딘가 저장해야한다. 이런 경우에 Spring Security는 RequestCache를 사용한다.

RequestCacheAwareFilter는 RequestCache를 사용해서 HttpServletReqeust를 저장한다. HttpSessionRequestCache가 기본값으로 사용된다. 이 방법은 주로 사용자의 상태정보를 session에 저장하는 방식에서 사용된다. JWT 기반 인증으로 서버의 무상태성을 유지할 경우에는 보통 클라이언트 측에서 상태를 유지하는 방법을 사용한다고 한다. 써먹어봐야지.

## 참조

https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-print-filters

- custom filter 적용, logging, RequestCache 적용하지 않기..
- 내용 및 사진 출처
