# Spring Security 계층권한(HierarchyImpl) 설정하기

## RoleHierarchy

Spring Security에서는 인증된 사용자는 권한(Authentication)을 가질 수 있습니다. 또 권한은 여러개일 수 있습니다. 일반 사용자 외에도 관리자, 슈퍼 관리자가 있을 수 있죠. Spring Security에서는 이 권한들 간의 계층을 설정할 수 있도록 RoleHierarchy 인터페이스를 제공합니다.

```
package org.springframework.security.access.hierarchicalroles;

public interface RoleHierarchy {

	Collection<? extends GrantedAuthority> getReachableGrantedAuthorities(
			Collection<? extends GrantedAuthority> authorities);

}
```

`RoleHierarchy` 의 구현체 `RoleHierarchyImpl`도 제공합니다.

```
public class RoleHierarchyImpl implements RoleHierarchy {

	private String roleHierarchyStringRepresentation = null;

	public void setHierarchy(String roleHierarchyStringRepresentation) {
		this.roleHierarchyStringRepresentation = roleHierarchyStringRepresentation;
		logger.debug(LogMessage.format("setHierarchy() - The following role hierarchy was set: %s",
				roleHierarchyStringRepresentation));
		buildRolesReachableInOneStepMap();
		buildRolesReachableInOneOrMoreStepsMap();
	}
}
```

`RoleHierarchyImpl#setHierarchy` 메서드는 문자열 타입의 권한 계층 표현을 객체에 저장하는 역할을 합니다.

```
private void buildRolesReachableInOneStepMap() {
  private Map<String, Set<GrantedAuthority>> rolesReachableInOneStepMap = null;

  this.rolesReachableInOneStepMap = new HashMap<>();
  for (String line : this.roleHierarchyStringRepresentation.split("\n")) {
    // Split on > and trim excessive whitespace
    String[] roles = line.trim().split("\\s+>\\s+");
    for (int i = 1; i < roles.length; i++) {
      String higherRole = roles[i - 1];
      GrantedAuthority lowerRole = new SimpleGrantedAuthority(roles[i]);
      Set<GrantedAuthority> rolesReachableInOneStepSet;
      if (!this.rolesReachableInOneStepMap.containsKey(higherRole)) {
        rolesReachableInOneStepSet = new HashSet<>();
        this.rolesReachableInOneStepMap.put(higherRole, rolesReachableInOneStepSet);
      }
      else {
        rolesReachableInOneStepSet = this.rolesReachableInOneStepMap.get(higherRole);
      }
      rolesReachableInOneStepSet.add(lowerRole);
      logger.debug(LogMessage.format(
          "buildRolesReachableInOneStepMap() - From role %s one can reach role %s in one step.",
          higherRole, lowerRole));
    }
  }
}
```

`RoleHierarchyImpl#buildRolesReachableInOneStepMap` 메서드는 setHierarchy 메서드로 저장된 문자열 타입의 권한 계층 표현(roleHierarchyStringRepresentation)을 쪼개서 Map에 나눠 담습니다. 위의 코드는 '\n'으로 나뉘어진 한 줄의 문자열에서 '>'을 기준으로 문자열을 쪼개어 순서에 따라(왼쪽이 더 높은 권한) Map에 나누어 저장합니다.

예를 들어

```
"ROLE_SUPER_ADMIN > ROLE_HR_ADMIN > ROLE_USER"
```

위의 문자열은 `ROLE_SUPER_ADMIN`이 `ROLE_HR_ADMIN`과 `ROLE_USER`의 권한을 포함하고, `ROLE_HR_ADMIN`이 `ROLE_USER`의 권한을 포함한다는 의미입니다.

위의 내용을 Spring Security Config 파일에 적용을 해보겠습니다.

## Security config 적용

```
@Configuration
@EnableWebSecurity
public class SecurityConfig {

  @Bean
  public RoleHierarchy roleHierarchy() {
      RoleHierarchyImpl roleHierarchy = new RoleHierarchyImpl();
      String hierarchy = """
              ROLE_SUPER_ADMIN > ROLE_RSC_ADMIN > ROLE_USER
              ROLE_SUPER_ADMIN > ROLE_EMPL_ADMIN > ROLE_USER
              """;
      roleHierarchy.setHierarchy(hierarchy);
      return roleHierarchy;
  }
}
```

저는 `ROLE_SUPER_ADMIN`이 `ROLE_RSC_ADMIN`, `ROLE_EMPL_ADMIN`, `ROLE_USER`의 권한도 가지고 `ROLE_RSC_ADMIN`, `ROLE_EMPL_ADMIN`이 각각 `ROLE_USER`의 권한을 포함하는 구조로 작성했습니다.

```
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
      .authorizeHttpRequests(authorize -> authorize
        .dispatcherTypeMatchers(DispatcherType.FORWARD, DispatcherType.ERROR).permitAll()
        .requestMatchers("/js/**", "/css/**", "/img/**").permitAll()
        .requestMatchers("/", "/signin/**", "/signup/**").permitAll()
        .requestMatchers( "/spaces/**").hasRole("USER") //hasRole 사용 시 prefix ROLE_ 생략가능
        .requestMatchers("/admin-spaces/**").hasRole("RSC_ADMIN")
        .requestMatchers("/dept/**").hasRole("EMPL_ADMIN")
        .anyRequest().authenticated())

    return http.build();
}
```

위의 코드에서는 요청의 URL을 `RequestMatcher`로 특정 리소스를 요청할 때 어떤 권한이 필요한지 매핑합니다.

여기서 하나 문제가 발생했습니다. `ROLE_SUPER_ADMIN` 권한을 가진 사용자가 /dept/\*\* URL로 요청을 했는데 권한 불일치로 AccessDeniedException이 발생했습니다. 왜 이 문제가 발생했는지 알아보기 위해서 먼저 이 문제와 관련된 Spring Security 구조를 살펴보겠습니다.

## Spring Security Architecture 일부 훑어보기

### 인증(Authentication)

인증의 모든 과정은 사용자의 권한 확인과 권한의 저장으로 요약할 수 있습니다.

필터들을 거치면서 사용자의 권한을 확인하고, 그 권한 정보(`GrantedAuthority`)는 `AuthenticationManager`에 의해 `Authentication` 객체에 저장됩니다.

### 인가(Authorization)

인가 과정에서는 `AuthorizationManager`가 (요청한 리소스에 대한)최종 접근 통제 결정을 내릴 때 인증 과정에서 넘어온 `Authentication` 객체를 읽습니다.

#### AuthorizationManager

`AuthorizationManager` 인터페이스는 사용자가 보호되고 있는 리소스에 대한 접근 권한이 있는지 여부를 판단하는데 사용됩니다. 이 인터페이스의 구현체는 여러가지가 있는데, 그 중 하나가 `AuthorityAuthorizationManager` 입니다.

#### AuthorityAuthorizationManager

이 객체는 실질적으로 사용자가 요청한 리소스에 대한 접근 권한이 있는지를 판단하는 중요한 객체입니다. 이 객체는 권한(Authority) 집합을 포함하는데, 이 말을 조금 더 자세히 풀어보겠습니다.

```
public final class AuthorityAuthorizationManager<T> implements AuthorizationManager<T> {

	private static final String ROLE_PREFIX = "ROLE_";

	private final List<GrantedAuthority> authorities;

	private RoleHierarchy roleHierarchy = new NullRoleHierarchy();

```

`AuthorityAuthorizationManager` 객체의 private 멤버 중 `authorities`는 권한 정보인 `GrantedAuthority` List 입니다.

```
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
      .authorizeHttpRequests(authorize -> authorize
        .requestMatchers( "/spaces/**", "/options/**").hasRole("USER")
        .requestMatchers("/admin-spaces/**").hasRole("RSC_ADMIN")
        .requestMatchers("/depts/**").hasRole("EMPL_ADMIN")
        .anyRequest().authenticated())

    return http.build();
}
```

변수 `authorities`는 위의 코드의 hasRole이 실행될 때마다 초기화가 됩니다. 다르게 말하면 RequestMatcher마다 하나의 `AuthorityAuthorizationManager`가 생성된다는 뜻입니다. 위의 예에서는 총 3개가 생성되겠네요. `AuthorityAuthorizationManager`에는 requestMatcher의 URL 패턴에 매핑되는 권한들이 변수 `authorities`에 저장됩니다. `/admin-spaces/**` 패턴에 해당하는 `AuthorityAuthorizationManager` 객체의 변수 `authorities`에는 ROLE_RSC_ADMIN 권한이 저장되고 ROLE_RSC_ADMIN 권한을 가진 사용자만 해당 패턴의 요청에 접근할 수 있게 됩니다. 그래서 `AuthorityAuthorizationManager`객체가 권한 집합을 포함한다고 표현하는 것입니다.

이 객체에는 다른 private 필드, `roleHierarchy`가 있습니다.

```
public final class AuthorityAuthorizationManager<T> implements AuthorizationManager<T> {

	private static final String ROLE_PREFIX = "ROLE_";

	private final List<GrantedAuthority> authorities;

	private RoleHierarchy roleHierarchy = new NullRoleHierarchy();

	public void setRoleHierarchy(RoleHierarchy roleHierarchy) {
		Assert.notNull(roleHierarchy, "roleHierarchy cannot be null");
		this.roleHierarchy = roleHierarchy;
	}
```

이 변수는 `AuthorityAuthorizationManager#setRoleHierarchy` 메서드로 초기화하지 않는 이상 `NullRoleHierarchy` 객체로 초기화됩니다. 여기가 바로 문제의 원인입니다. 이제껏 `RoleHierarchyImpl`객체에 권한 계층 정보를 저장해 bean으로 등록했지만 이 bean이 사용되지 않았기 때문에 문제가 발생한 것입니다.

저는 지금 spring legacy 버전(5.3.31)과 spring security 5.8.9 버전을 사용하고 있습니다. spring boot(6+)에서는 `RoleHierarchy`, `ExpressionHandler` 객체를 bean으로 등록하면 자동으로 권한 계층을 `AuthorizationManager`들에 전반적으로 적용시킬 수 있지만, 제가 사용하고 있는 버전에서는 `expressionHandler`를 사용할 수 있는 방법을 찾지 못했습니다.

그래서 우선 문제 해결을 위해 4가지 종류의 권한에 해당하는 `AuthorityAuthorizationManager` bean들을 각각 등록했습니다. 꼭 bean으로 등록하지 않아도 `SecurityConfig#securityFilterChain`의 로컬 변수로 선언해서 사용할 수도 있습니다. 권한에 따라 다른 `AuthorityAuthorizationManager`가 생성되기때문에 총 4개를 bean으로 등록했습니다.

```
    @Bean
    public AuthorityAuthorizationManager<RequestAuthorizationContext> userAuthorityAuthorizationManager() {
        AuthorityAuthorizationManager<RequestAuthorizationContext> userAuthManager = AuthorityAuthorizationManager.hasRole("USER");
        userAuthManager.setRoleHierarchy(roleHierarchy());
        return userAuthManager;
    }

    @Bean
    public AuthorityAuthorizationManager<RequestAuthorizationContext> rscAdminAuthorityAuthorizationManager() {
        AuthorityAuthorizationManager<RequestAuthorizationContext> rscAdminAuthManager = AuthorityAuthorizationManager.hasRole("RSC_ADMIN");
        rscAdminAuthManager.setRoleHierarchy(roleHierarchy());
        return rscAdminAuthManager;
    }

    @Bean
    public AuthorityAuthorizationManager<RequestAuthorizationContext> emplAdminAuthorityAuthorizationManager() {
        AuthorityAuthorizationManager<RequestAuthorizationContext> emplAdminAuthManager = AuthorityAuthorizationManager.hasRole("EMPL_ADMIN");
        emplAdminAuthManager.setRoleHierarchy(roleHierarchy());
        return emplAdminAuthManager;
    }

    @Bean
    public AuthorityAuthorizationManager<RequestAuthorizationContext> superAdminAuthorityAuthorizationManager() {
        AuthorityAuthorizationManager<RequestAuthorizationContext> superAdminAuthManager = AuthorityAuthorizationManager.hasRole("SUPER_ADMIN");
        superAdminAuthManager.setRoleHierarchy(roleHierarchy());
        return superAdminAuthManager;
    }
```

그리고 이 빈들이 적용될 수 있도록 `access` 메서드를 사용해 문제를 해결했습니다.

```
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
      .authorizeHttpRequests(authorize -> authorize
        .dispatcherTypeMatchers(DispatcherType.FORWARD, DispatcherType.ERROR).permitAll()
        .requestMatchers("/js/**", "/css/**", "/img/**").permitAll()
        .requestMatchers("/", "/signin/**", "/signup/**").permitAll()
        .requestMatchers( "/spaces/**").access(userAuthorityAuthorizationManager())
        .requestMatchers("/admin-spaces/**").access(rscAdminAuthorityAuthorizationManager())
        .requestMatchers("/dept/**").access(emplAdminAuthorityAuthorizationManager())
        .anyRequest().authenticated())

    return http.build();
}
```

분명 AuthorizationManager의 hierarchy 변수를 전역으로 초기화하는 객체가 존재할 것 같기도 하고, 이 문제를 spring boot에서 해결한 것일 수도 있다는 생각도 듭니다. 긴 글 읽어주셔서 감사합니다. 도움이 되었으면 좋겠습니다!
