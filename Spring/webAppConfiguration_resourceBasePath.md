# @WebAppConfiguration의 resourceBasePath

## 환경

- JUnit5
- Mockito test framework
- Spring framework

## 문제

SpringSecurity와 관련된 기능들을 테스트하기 위한 테스트 코드를 실행하던 중 문제가 발생했습니다. 원래 모든 테스트는 통과했었지만 spring의 web 관련 설정을 작성하는 dispatcher-servlet.xml에 Tiles3 관련 설정을 추가한 이후에 발생한 문제였습니다.

다음은 dispatcher-servlet.xml에 추가한 설정입니다.

```
<beans:bean id="tilesConfigurer" class="org.springframework.web.servlet.view.tiles3.TilesConfigurer">
    <beans:property name="definitions">
        <beans:list>
            <beans:value>/WEB-INF/tiles/tiles-setting.xml</beans:value>
        </beans:list>
    </beans:property>
</beans:bean>
```

logger에 뜬 예외 메시지는 다음과 같습니다.

```
Caused by: java.io.FileNotFoundException: ServletContext resource [/WEB-INF/tiles/tiles-setting.xml] cannot be resolved to URL because it does not exist
```

다음은 test 코드의 일부입니다.

```

@Transactional
@ExtendWith({SpringExtension.class, MockitoExtension.class})
@WebAppConfiguration
@ContextConfiguration(locations = {"file:web/WEB-INF/spring/**/testContext.xml", "file:web/WEB-INF/spring/**/dispatcher-servlet.xml"})
class SecurityTest {

    @Autowired
    private WebApplicationContext ctx;

    @BeforeEach
    void setup() {
        MockitoAnnotations.openMocks(this);

        mockMvc = MockMvcBuilders.webAppContextSetup(ctx)
                .apply(springSecurity())
                .addFilters(new CharacterEncodingFilter("UTF-8", true))
                .alwaysDo(print())
                .build();
    }
}
```

예외 메시지를 살펴보면 dispatcher-servlet.xml에서 설정한 TilesConfigurer의 definitions의 파일경로를 찾지 못하는 문제임을 알 수 있습니다.

## 원인

debugger로 예외가 발생하기 직전, BasicTilesContainerFactory 클래스에서 참조하는 applicationContext 객체에 저장된 값들 중 ServletContext의 변수 **resourceBasePath**의 값이 "**src/main/webapp**"인 것을 확인했습니다. 하지만 dispatcher-servlet에서 사용한 tiles 설정 파일의 실제 경로는 "src/main/webapp/WEB-INF/tiles/tiles-setting.xml"가 아닌 "web/WEB-INF/tiles/tiles-setting.xml"입니다. 애플리케이션의 디렉터리 구조가 기본 설정과 다르기 때문에 발생한 문제였습니다.

### @WebAppConfiguration

이 어노테이션은 통합 테스트를 위해 필요한 applicationContext가 `WebApplicationContext`여야함을 알립니다. 더 자세히는 웹 애플리케이션 루트 경로의 기본값("src/main/webapp")을 사용해서 `webApplicationContext`를 로드해야함을 알리는 어노테이션이죠. 웹 애플리케이션 루트의 경로가 실제와 다르니 실제 경로값을 알려줘야겠죠? 그 방법은 아주 간단합니다.

에너테이션의 루터 경로를 의미하는 value 속성값을 전달해주기만 하면 됩니다. 아래와 같이요.

```
@WebAppConfiguration("web")
```

## MockMvc(Spring MVC Test Framework)

WebApplicationContext를 받아 테스트에서 사용하는 MockMvc에 대해 조금 더 알아보려고 합니다. MockMvc는 spring MVC test framework로 서버를 실제로 실행하지 않고 Spring MVC Controller를 테스하기 위해 사용됩니다.

이 테스트에 사용되는 `MockMvc` 객체를 설정하는 방법은 두 가지가 있습니다.

- standaloneSetup

  Controller를 특정해서 코드를 직접 작성하여 Spring MVC 인프라를 구성하는 방식입니다.

  ```
  class MyWebTests {

    MockMvc mockMvc;

    @BeforeEach
    void setup() {
      this.mockMvc = MockMvcBuilders.standaloneSetup(new AccountController()).build();
    }
  // ...
  }
  ```

  applicationContext 전체를 로드하지 않고 특정 Controller에만 초점을 맞춰서 테스트를 수행할 때 사용합니다. 때문에 통합테스트가 아닌 단위 테스트에 조금 더 적합한 설정 방식입니다.

  이 방식은 spring security같은 필터 체인을 자동으로 적용하지 않기 때문에 보안 과정이 제대로 작동하는지를 확인하는데에는 적합하지 않습니다.

- webAppContextSetup

  `WebApplicationContext` 전체를 로드해서 테스트를 수행할 떄 필요한 객체를 사용합니다. 때문에 단위 테스트가 아닌 통합 테스트에 좀 더 적합한 설정 방식입니다.

  ```
  @SpringJUnitWebConfig(locations = "test-servlet-context.xml")
  class AccountTests {

    @Autowired
    AccountService accountService;

    MockMvc mockMvc;

    @BeforeEach
    void setup(WebApplicationContext wac) {
      this.mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
    }
    // ...
  }
  ```

  `ApplicatinContext`는 `WebApplicationContext`에서 접근할 수 있기 때문에 MockMvc의 WebAppContextSetup에서 다룰 수 있고, 이를 통해 Spring Security를 포함해 애플리케이션에서 사용하는 모든 빈들에 접근할 수 있습니다. 따라서 MockMvc의 이 설정을 사용하여 Spring Security가 예상 대로 작동하고 있는지를 테스트할 수 있습니다.

## 참조

- https://stackoverflow.com/questions/33459740/error-no-url-for-servletcontext-resource-when-running-spring-integrated-test
- https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/context/web/WebAppConfiguration.html
- https://docs.spring.io/spring-framework/reference/testing/spring-mvc-test-framework/server-setup-options.html
