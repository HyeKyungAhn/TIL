# Spring Security, JWT로 Login 기능 구현하기(1) - SecurityConfig 파일 구성

SecurityConfig의 파일에는 어떤 빈들이 있는지 알아보고 그 역할을 파악하자.

## 환경

- Spring Legacy - Spring Framework 5.x.x
- Maven
- Sring Security 5.x.x

## 로그인 구현

### 의존성 추가

```
<dependency>
  <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-web</artifactId>
    <version>5.8.9</version>
</dependency>

<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>5.8.9</version>
</dependency>

<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <version>5.8.9</version>
    <scope>test</scope>
</dependency>
```

### Security Config 파일 구성

Security Config 파일에서는 spring security에서 사용자 인증을 하는데 필요한 필터들을 다룬다.

개발자의 필요에 따라 어떤 필터는 사용하지 않기도하고 또 특정한 용도로 쓰이는 커스텀 필터들을 추가하기도 한다. 나는 JWT 인증 방식을 사용하기 때문에 sessioin을 사용하여 사용자 정보를 서버에 저장하는 필터들은 사용하지 않았다.

또한 spring security에서 커스텀 필터들을 관리, 제공할 수 있도록 SecurityConfig.java 에서 등록한다.

#### SecurityFilterChain

```
@Configuration
@EnableWebSecurity
public class SecurityConfig {

        @Autowired
    private UserDetailServiceImpl userService;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.csrf(AbstractHttpConfigurer::disable)
                .httpBasic(AbstractHttpConfigurer::disable)
                .formLogin(AbstractHttpConfigurer::disable)
                .addFilterAfter(jsonUsernamePasswordSignInFilter(), LogoutFilter.class)
                .authorizeHttpRequests(authorize ->
                        authorize
                                .requestMatchers("/", "/signup", "/signin").permitAll()
                                .anyRequest().authenticated())
                .logout(logout ->
                        logout.logoutSuccessUrl("/").invalidateHttpSession(true).permitAll())
                .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS));

        return http.build();
    }
    ...
}
```

- @EnableWebSecurity

  Spring Security 관련된 기능(필터)를 사용할 수 있도록 Servlet Container에 Filter를 자동 등록한다. (AbstractSecurityWebApplicationInitializer, servlet3.0+)

- `WebSecurityConfigurerAdapter`

  spring security 설정을 하는데 사용되는 API. Spring 5.7.0-M2 에서 더이상 사용되지 않는 클래스이다. 따라서 이 클래스를 상속받아 설정파일을 작성하지 않고 Java 클래스와 에너테이션을 사용해서 자동으로 설정을 구성하는 방식(component-base configure)으로 작성했다.

- `SecurityConfig#securityFilterChain`

  Spring Security가 사용하는 필터들을 묶어 관리한다. `HttpSecurity` 객체를 사용해서 특정한 Http 요청을 처리할 때 사용할 수 있는 보안 필터들을 구성할 수 있다.

  나는 JWT 인증방식과 CSRF 방어를 위해 Cookie의 HttpOnly + SameSite 속성을 사용하기 때문에 위와 같은 설정을 했다. 프로젝트를 진행하면서 위의 설정이 바뀔 수도 있다. 이 메서드 내에서 사용한 HttpSecurity 클래스의 메서드에 대한 정보는 [공식문서](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/web/builders/HttpSecurity.html)에서 확인할 수 있다.

#### UserDetailService

```
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;
import site.roombook.dao.EmplDao;
import site.roombook.domain.EmplDto;

@Service
public class UserDetailServiceImpl implements UserDetailsService {

    @Autowired
    private EmplDao emplDao;

    @Override
    public UserDetails loadUserByUsername(String emplId) throws UsernameNotFoundException {
        EmplDto emplDto = emplDao.selectEmplById(emplId).orElseThrow(() -> new IllegalArgumentException(emplId));
        return EmplDto.EmplDtoBuilder()
                .emplId(emplDto.getEmplId())
                .pwd(emplDto.getPwd())
                .build();
    }
}

```

- SecurityConfig 구현에서 잠시 빠져나와 `UserDetailService`를 구현한 `UserDetailServiceImpl`을 보자. 이 서비스 빈에서는 DB에서 사용자의 실제 데이터를 조회해서 `UserDetail` 객체로 생성해 반환한다. 이후 인증 시 입력된 정보와 비교하는데 사용된다.

#### DaoAuthenticationProvider, BCryptPasswordEncoder

```
@Bean
public DaoAuthenticationProvider daoAuthenticationProvider() {
    DaoAuthenticationProvider daoAuthenticationProvider = new DaoAuthenticationProvider();

    daoAuthenticationProvider.setUserDetailsService(userService);
    daoAuthenticationProvider.setPasswordEncoder(passwordEncoder());

    return daoAuthenticationProvider;
}

@Bean
public BCryptPasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

- `DaoAuthenticationProvider`

  인증을 처리하는 필터인 `AuthenticationProvider`의 구현체이다. 각 구현체마다 다른 인증방식으로 인증을 처리하는데, `DaoAuthenticationProvider`는 `UserDetailService`와 `PasswordEncoder`를 사용해서 username/password를 인증한다.

- `BCryptPasswordEncoder`

  `PasswordEncode`의 구현체다. 여기서는 `DaoAuthenticationProvider`에서 `UserDetail`에 저장되어있는 비밀번호를 로그인 시 입력한 비밀번호와 비교하기 위해 빈으로 등록한다.

#### AuthenticationManager

```
@Bean
public AuthenticationManager authenticationManager() {
    DaoAuthenticationProvider provider = daoAuthenticationProvider();
    return new ProviderManager(provider);
}
```

- `AuthenticationManager`

  인증 처리 방법을 정의하는 API이다. 인증이 필요할 때 이 빈이 인증을 처리할 수 있는 필터를 호출하기 때문에 `DaoAuthenticationProvider` 객체를 생성자에 넣어 반환한다.

#### JsonUsernamePasswordAuthenticationFilter

```
@Bean
public JsonUsernamePasswordAuthenticationFilter jsonUsernamePasswordSignInFilter() {
    JsonUsernamePasswordAuthenticationFilter jsonUsernamePasswordLoginFilter = new JsonUsernamePasswordAuthenticationFilter(new ObjectMapper());
    jsonUsernamePasswordLoginFilter.setAuthenticationManager(authenticationManager());
    jsonUsernamePasswordLoginFilter.setAuthenticationSuccessHandler(signInSuccessJWTProvideHandler());
    jsonUsernamePasswordLoginFilter.setAuthenticationFailureHandler(signInFailureHandler());
    return jsonUsernamePasswordLoginFilter;
}
```

- `AbstractAuthenticationProcessingFilter#attemptAuthentication`에서는 로그인 시 사용자가 입력한 정보를 저장한 `Authentication` 객체를 생성해서 반환한다.

#### SignInSuccessJWTProvideHandler, SignInFailureHandler

```
  @Bean
  public SignInSuccessJWTProvideHandler signInSuccessJWTProvideHandler() {
      return new SignInSuccessJWTProvideHandler();
  }

  @Bean
  public SignInFailureHandler signInFailureHandler() {
      return new SignInFailureHandler();
  }
```

- `SignInSuccessJWTProvideHandler`, `SignInFailureHandler` 두 빈은 각각 인증 성공 시, 실패 시의 동작을 구현한다.

다음은 두 빈의 전체 코드이다.

```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.authentication.SimpleUrlAuthenticationFailureHandler;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class SignInFailureHandler extends SimpleUrlAuthenticationFailureHandler {
    private static final Logger logger = LoggerFactory.getLogger(SignInFailureHandler.class);

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
        response.setStatus(HttpServletResponse.SC_OK);

        try {
            response.getWriter().write("fail");
        } catch (IOException e) {
            e.printStackTrace();
        }
        logger.info("로그인에 실패했습니다");
    }
}
```

```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.web.authentication.SimpleUrlAuthenticationSuccessHandler;
import site.roombook.service.JwtService;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class SignInSuccessJWTProvideHandler extends SimpleUrlAuthenticationSuccessHandler {

    @Autowired
    JwtService jwtService;

    private static final Logger log = LoggerFactory.getLogger(SignInSuccessJWTProvideHandler.class);

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        UserDetails userDetails = (UserDetails) authentication.getPrincipal();
        log.info( "로그인에 성공합니다 JWT를 발급합니다. username: {}" ,userDetails.getUsername());

        String accessToken = jwtService.createAccessToken(userDetails.getUsername());
        String refreshToken = jwtService.createRefreshToken();

        jwtService.updateRefreshToken(userDetails.getUsername(), refreshToken);
        jwtService.sendToken(response, accessToken, refreshToken);
    }
}

```

- `JsonUsernamePasswordAuthenticationFilter`

  이 필터는 `AbstractAuthenticationProcessingFilter`의 서브 클래스로 사용자 인증에 사용하는 base 필터다. 로그인을 form 요청이 아니라 RESTful 한 API로 구현할 것이기 때문에 JSON 형식의 데이터를 받아 처리할 수 있도록 기능을 확장, 변경했다. 인증의 전반적인 로직을 관리하는 필터이다. 인증 정보를 받아 인증을 수행하고 결과의 성공 실패 여부에 따라 처리하는 역할을 한다.

  다음은 `JsonUsernamePasswordAuthenticationFilter`을 구현한 전체 코드이다.

  ```
  import com.fasterxml.jackson.databind.ObjectMapper;
  import org.springframework.security.authentication.AuthenticationServiceException;
  import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
  import org.springframework.security.core.Authentication;
  import org.springframework.security.core.AuthenticationException;
  import org.springframework.security.web.authentication.AbstractAuthenticationProcessingFilter;
  import org.springframework.security.web.util.matcher.AntPathRequestMatcher;
  import org.springframework.util.StreamUtils;

  import javax.servlet.ServletException;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.io.IOException;
  import java.nio.charset.StandardCharsets;
  import java.util.Map;

  public class JsonUsernamePasswordAuthenticationFilter extends AbstractAuthenticationProcessingFilter {
      private final ObjectMapper objectMapper;

      private static final String DEFAULT_LOGIN_REQUEST_URL = "/signin";
      private static final String HTTP_METHOD = "POST";
      private static final String CONTENT_TYPE = "application/json";
      private static final String USERNAME_KEY="username";
      private static final String PASSWORD_KEY="password";

      private static final AntPathRequestMatcher DEFAULT_LOGIN_PATH_REQUEST_MATCHER =
              new AntPathRequestMatcher(DEFAULT_LOGIN_REQUEST_URL, HTTP_METHOD);

      public JsonUsernamePasswordAuthenticationFilter(ObjectMapper objectMapper) {
          super(DEFAULT_LOGIN_PATH_REQUEST_MATCHER);
          this.objectMapper = objectMapper;
      }

      @Override
      public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException, IOException, ServletException {
          if (request.getContentType() == null || !request.getContentType().equals(CONTENT_TYPE)) {
              throw new AuthenticationServiceException("Authentication Content-Type not supported: " + request.getContentType());
          }

          String messageBody = StreamUtils.copyToString(request.getInputStream(), StandardCharsets.UTF_8);

          Map<String, String> usernamePasswordMap = objectMapper.readValue(messageBody, Map.class);

          String username = usernamePasswordMap.get(USERNAME_KEY);
          String password = usernamePasswordMap.get(PASSWORD_KEY);

          UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);

          return this.getAuthenticationManager().authenticate(authRequest);
      }
  }
  ```

#### 전체 SecurityConfig.java 코드

```

import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.ProviderManager;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.logout.LogoutFilter;
import site.roombook.global.signin.filter.JsonUsernamePasswordAuthenticationFilter;
import site.roombook.global.signin.handler.SignInFailureHandler;
import site.roombook.global.signin.handler.SignInSuccessJWTProvideHandler;
import site.roombook.service.UserDetailServiceImpl;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Autowired
    private UserDetailServiceImpl userService;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.csrf(AbstractHttpConfigurer::disable)
                .httpBasic(AbstractHttpConfigurer::disable)
                .formLogin(AbstractHttpConfigurer::disable)
                .addFilterAfter(jsonUsernamePasswordSignInFilter(), LogoutFilter.class)
                .authorizeHttpRequests(authorize ->
                        authorize
                                .requestMatchers("/", "/signup", "/signin").permitAll()
                                .anyRequest().authenticated())
                .logout(logout ->
                        logout.logoutSuccessUrl("/").invalidateHttpSession(true).permitAll())
                .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS));

        return http.build();
    }

    @Bean
    public DaoAuthenticationProvider daoAuthenticationProvider() {
        DaoAuthenticationProvider daoAuthenticationProvider = new DaoAuthenticationProvider();

        daoAuthenticationProvider.setUserDetailsService(userService);
        daoAuthenticationProvider.setPasswordEncoder(passwordEncoder());

        return daoAuthenticationProvider;
    }

    @Bean
    public BCryptPasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public AuthenticationManager authenticationManager() {
        DaoAuthenticationProvider provider = daoAuthenticationProvider();
        return new ProviderManager(provider);
    }

    @Bean
    public JsonUsernamePasswordAuthenticationFilter jsonUsernamePasswordSignInFilter() {
        JsonUsernamePasswordAuthenticationFilter jsonUsernamePasswordLoginFilter = new JsonUsernamePasswordAuthenticationFilter(new ObjectMapper());
        jsonUsernamePasswordLoginFilter.setAuthenticationManager(authenticationManager());
        jsonUsernamePasswordLoginFilter.setAuthenticationSuccessHandler(signInSuccessJWTProvideHandler());
        jsonUsernamePasswordLoginFilter.setAuthenticationFailureHandler(signInFailureHandler());
        return jsonUsernamePasswordLoginFilter;
    }

    @Bean
    public SignInSuccessJWTProvideHandler signInSuccessJWTProvideHandler() {
        return new SignInSuccessJWTProvideHandler();
    }

    @Bean
    public SignInFailureHandler signInFailureHandler() {
        return new SignInFailureHandler();
    }
}

```

## 참조

- https://ttl-blog.tistory.com/273
- https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter
  - WebSecurityConfigurerAdapter 관련
- https://docs.spring.io/spring-security/reference/features/exploits/csrf.html#csrf-explained
  - csrf
