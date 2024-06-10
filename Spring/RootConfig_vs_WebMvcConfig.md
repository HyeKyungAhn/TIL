# RootContext vs Web Application Context

Spring MVC를 사용해 구현하고 있는 웹 애플리케이션에서 Transaction과 관련한 문제가 있었다.

Controller가 호출하는 여러 Service들의 트랜잭션을 묶어 관리하기 위해 Service들을 호출하는 Service를 만들고 @Transactional을 적용했다.문제는 JUnit5로 테스트를 할 때는 서비스들 중 하나에서 예외가 발생하면 rollback이 되는데 실제 웹 애플리케이션을 실행 중일 때는 rollback이 되지 않는 것이었다.

여러 블로그에서 문제점을 Root Context와 Web Application Context의 설정파일에서 component scan 태그(설정 파일을 xml 형식으로 작성했다.)의 package 범위를 제한하거나 include-filter, exclude-filter로 특정 애너테이션이나 패키지, 클래스를 제외하거나 포함하는 방식으로 해결하는 것을 보고 문제의 원인을 정확하게 파악하기 위해 WebApplicationContext와 설정파일에 대해 좀 더 살펴봤다.

## ApplicationContext

Spring application에 환경 설정을 제공하기 위한 interface.
ServletContext와 함께 동작하며 Container와 communicate 할 수 있음.

### ServletContext

servlet은 작은 프로그램을 말한다.

## WebApplicationContext

ApplicationContext를 상속해서 **_Web_** application에 필요한 환경설정을 제공한다.

일반적으로 web.xml에 등록된 모든 applicationContext는 WebApplicationContext이고 root webapp context와 servlet app context로 나뉜다.

### root context

---

- 전체 spring application(webapp)에 관한 전역 context이다.
- service, repository, 트랜잭션 관리 빈, 일반적인 컴포넌트나 application에서 사용되는 빈들을 관리
- application 내 모든 servlet context가 root context를 참조할 수 있다(역은 불가하다)
- 따라서 spring application 전체에서 공유되는 빈들(Spring Security, DB 접근 서비스 등)을 관리한다.
- ContextLoaderLinstener에 의해 로드된다.

#### root (application) context 설정 방법

web.xml에 context-param 태그로 root-context.xml 설정 파일을 등록한다.

```
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>
        /WEB-INF/root-context.xml
            /WEB-INF/applicationContext-security.xml
    </param-value>
</context-param>
```

root context는 web.xml에 선언되어있는 ContextLoaderListener에 의해 생성된다.

```
<listener>
  <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

### servlet context

---

- 특정 서블릿에 필요한 빈을 관리한다. 개발자가 임의로 지정할 수 있다.
  - 서블릿이란 특정한 기능을 처리하는 자바 객체를 말한다(=작은 프로그램)
- 해당 서블릿에서만 빈을 사용하기 때문에 다른 서블릿에서 사용되는 같은 이름의 빈과의 충돌을 피할 수 있다.
- servlet context는 여러 개일 수 있다.
- Spring MVC에서 사용되는 servlet context에서는 **_web layer_**와 관련된 빈을 관리한다(Controller, view Resolver, handler mapping 등)
- Dispatcher Servlet에 의해 로드된다.

#### servlet context 설정 방법

init-param 태그를 사용해서 servlet context 설정파일을 등록한다. DispatcherServlet이 app-servlet.xml 설정파일로 servlet context를 생성한다.

```
<servlet>
   <servlet-name>myservlet</servlet-name>
   <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
   <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>app-servlet.xml</param-value>
   </init-param>
</servlet>
```

## 문제 해결 과정

트랜잭션 관련 설정을 RootContext의 설정 파일에 작성했다.

```
// root context 설정파일

<context:component-scan base-package="site.domain">

<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <property name="dataSource" ref="dataSource"/>
</bean>
```

```
//servlet context 설정파일(spring-servlet.xml)
<context:component-scan base-package="site.domain"/>
```

문제의 원인부터 말하자면, root context, servlet context 설정 파일 에서 component-scan 태그로 동일한 package 범위를 설정한 것이 원인이었다.

두 context에서 동일한 패키지를 스캔하면 동일한 빈 두개를 생성해서 각각 관리하게 된다. 이 상태에서 Web application에 사용자가 요청을 하면 먼저 Dispatcher servlet이 요청을 받아 요청을 처리할 수 있는 빈에 요청을 위임하는데 우선 DispatcherServlet의 자식 context에서 빈을 찾고 없으면 Root Application Context에서 빈을 찾아 요청을 처리한다.

root context와 servlet context가 같은 빈을 가지고 있다면 사용자로부터 요청을 받을 때 DispatcherServlet 늘 ServletContext에 저장된 빈에서 요청을 처리하는데, ServletContext를 생성할 때 사용한 설정 파일에는 트랜잭션과 관련된 설정이 없기 때문에 @Transactional이 선언된 객체임에도 트랜잭션과 관련된 advice가 적용(weaving)되지 않는다는 점이 문제의 원인이다.

이를 해결하기 위해서 아래와 같은 방법들을 사용할 수 있다.

1. servlet context의 설정 파일에 transaction 관련 설정을 추가해준다.
2. transaction과 관련된 행위들을 @Service가 선언된 객체에 옮기고 servlet context와 root context가 스캔하는 패키지의 범위를 분리한다.

1번은 여전히 중복 생성의 문제가 남아있으니 사용하지 않기로 한다.
나는 2번의 방법으로 문제를 해결했다.

### 해결 방법

```
// root context 설정파일

<context:component-scan base-package="site.domain">
  <context:exclude-filter type="regex" expression="site\.domain\.controller\.*"/>
</context:component-scan>
```

```
// servlet context 설정파일

<context:component-scan base-package="site.domain.controller"/>
```

web layer 관련된 빈을 관리하는 Servlet Context에서는 사용자의 요청을 처리할 수 있는 controller 객체들이 구현되어있는 site.domain.controller 패키지만 스캔할 수 있도록 base-package의 범위를 수정했다. 마찬가지로 Root Application Context에서는 exclude-filter 태그를 활용하여 controller외의 다른 빈들만 스캔해서 중복 스캔의 문제를 피할 수 있도록 스캔의 범위를 수정했다.

## 참조

- https://stackoverflow.com/questions/24472299/component-scan-issue-transactional-doesnt-work-with-controller
- https://stackoverflow.com/questions/24367503/method-not-being-intercepted-by-transaction-advisor-even-though-adding-transact/24389655#24389655
- https://stackoverflow.com/questions/11708967/what-is-the-difference-between-applicationcontext-and-webapplicationcontext-in-s
- https://stackoverflow.com/questions/3652090/difference-between-applicationcontext-xml-and-spring-servlet-xml-in-spring-frame
