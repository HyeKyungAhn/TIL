# Spring Security, JWT로 Login 기능 구현하기(2) - JwtService

## Jwt 서비스 구현

JWT 토큰을 생성하고 관리하는 책임을 가지고 있는 `JwtService`를 구현하려고 합니다.

```
public interface JwtService {
    String createAccessToken(String username);

    String createRefreshToken();

    void saveRefreshToken(String username, String refreshToken);

    void expireRefreshToken(String username);

    void sendToken(HttpServletResponse response, String accessToken, String refreshToken) throws IOException;

    String extractAccessToken(HttpServletRequest request) throws IOException, ServletException;

    String extractRefreshToken(HttpServletRequest request) throws IOException, ServletException;

    String extractUsername(String accessToken);

    boolean isValidAccessToken(String accessToken);

    Authentication getAuthentication(String accessToken);
}
```

다음은 `JwtService`를 구현한 `JwtServiceImpl`입니다.

```
@Service
public class JwtServiceImpl implements JwtService{

    @Autowired
    private EmplDao emplDao;

    @Autowired
    private UserDetailsService userDetailsService;

    @Value("${jwt.secret}")
    private String secret;
    @Value("${jwt.access.expiration}")
    private long accessTokenValidityInSeconds;
    @Value("${jwt.refresh.expiration}")
    private long refreshTokenValidityInSeconds;

    private static final String ACCESS_TOKEN_COOKIE_NAME = "JAT"; //AccessToken
    private static final String REFRESH_TOKEN_COOKIE_NAME = "JRT"; //RefreshToken
    ...
}
```

- JWT access token과 refresh token은 둘 다 cookie에 담아서 클라이언트에 보내려고 합니다. 그 이유에 대해서는 조금 있다 설명하려고 합니다.
- `ACCESS_TOKEN_COOKIE_NAME`, `REFRESH_TOKEN_COOKIE_NAME` 상수는 JWT를 담는 쿠키의 이름 값을 의미합니다.
- JWT 토큰을 생성하는데 필요한 secret key나 만료 시간 값은 @Value를 사용하여 외부 파일에서 가져오려고 합니다.

<br>
다음은 구현한 메서드들 입니다.

```
private static final String USERNAME_CLAIM = "id";

@Override
public String createAccessToken(String username) {
    return JWT.create()
            .withSubject(ACCESS_TOKEN_COOKIE_NAME)
            .withExpiresAt(new Date(System.currentTimeMillis() + accessTokenValidityInSeconds * 1000))
            .withClaim(USERNAME_CLAIM, username)
            .sign(Algorithm.HMAC512(secret));
}

@Override
public String createRefreshToken() {
    return JWT.create()
            .withSubject(REFRESH_TOKEN_COOKIE_NAME)
            .withExpiresAt(new Date(System.currentTimeMillis() + refreshTokenValidityInSeconds * 1000))
            .sign(Algorithm.HMAC512(secret));
}
```

- JWT는 Header, Payload, Signature 세 부분으로 이루어져있습니다. 그 중 payload에 저장되는 정보의 집합을 말하는 것이 claim입니다. 여기서 `.withClaim()`메서드로 JWT에 저장하려는 것은 사용자의 id 정보입니다. 이 정보는 private claim으로 송신자와 수신자 사이에 약속만 되어있다면 주고 받을 수 있는 데이터입니다.

- 이 두 메서드는 인증이 성공했을 때 JWT를 생성하는 메서드입니다.
  - 제가 구현하려는 애플리케이션은 사용자의 상태 정보를 유지하지 않는(stateless) JWT 인증 방식을 사용합니다.
  - 또 저는 인증을 위해 JWT(Json Web Token)는 총 두 가지 access token, refresh token을 생성하려고 합니다.
  - access token
    - 요청을 보내면 spring security에서 요청을 가로채어 포함되어있는 access token이 유효한지 확인한 후 위변조 되지 않은 token이라면 계속 요청을 수행합니다.
    - 사용자의 상태 정보를 저장하지 않는 서버에서 인증된 사용자의 모든 요청 마다 신원 확인을 위해 DB를 조회하지 않으려고 access token을 사용합니다.
    - access token은 서버에서 필요한 사용자 정보가 모두 있고, 서버에 저장되어있는 secret key와 token의 signature로 위변조 여부를 확인할 수 있기 때문에 사용자의 신원을 확인할 수 있는 신뢰할만한 데이터입니다.
    - access token이 탈취될 경우 공격자가 토큰 정보만 가지고 있으면 마치 인증된 사용자처럼 행동할 수 있습니다. 이를 방지하기 위한 여러가지 조치 중 access token의 만료시간을 비교적 짧게 설정하는 방법이 있습니다.
  - refresh token
    - access token이 만료될 경우, 이 token을 재발급 받는 용도로 사용하는 token입니다.
    - access token의 만료 시간을 짧게 설정할 경우에 사용자가 계속 새로 로그인을 해야하는 불편함을 방지하기 위해 사용합니다.
    - access token을 재발급 받을 때 DB 에 저장되어있는 refresh token과 요청에 딸려온 refresh token을 비교하고, 일치할 경우에 access token을 재발급합니다.
    - 이 토큰 또한 만료 시간이 있지만 상대적으로 더 긴 편입니다.
- 암호화 시 사용하는 알고리즘은 HMAC512입니다.

```
@Override
public void saveRefreshToken(String username, String refreshToken) {
    jwtDao.expireTokenByEmplId(username);

            try {
                jwtDao.insertToken(JwtDto.JwtDtoBuilder()
                        .tknId(UUID.randomUUID().toString())
                        .tkn(refreshToken)
                        .creEmplId(username)
                        .creDtm(LocalDateTime.now())
                        .expiDtm(LocalDateTime.now().plusMinutes(1L)).build());
            } catch (DataIntegrityViolationException e) {
                throw new IllegalArgumentException("해당 사용자가 없습니다.");
            }
}

@Override
public void expireRefreshToken(String username) {
    if (jwtDao.expireTokenByEmplId(username) == 0) {
        throw new IllegalArgumentException("해당 사용자가 없거나 만료할 토큰이 없습니다.");
    }
}
```

위의 두 메서드는 refresh token을 DB에 저장하고 삭제하는 메서드입니다. saveRefreshToken 메서드는 기존의 사용자 정보에 refresh token에 관련된 정보를 추가하는 것으로 구현하다 Refresh token을 사용자 정보에서 분리해서 따로 관리하게 되어 updateRefreshToken에서 saveRefreshToken로 메서드명을 변경한 이력이 있습니다.

```
@Override
public void sendToken(HttpServletResponse response, String accessToken, String refreshToken) throws IOException {
    response.setContentType("application/json;charset=UTF-8");
    response.setStatus(HttpServletResponse.SC_OK);

    setTokenInHttpOnlyCookie(response, ACCESS_TOKEN_COOKIE_NAME, accessToken);
    setTokenInHttpOnlyCookie(response, REFRESH_TOKEN_COOKIE_NAME, refreshToken);

    setCookieSameSite(response);
}

private void setTokenInHttpOnlyCookie(HttpServletResponse response, String cookeName, String token) {
    Cookie cookie = new Cookie(cookeName, token);
    cookie.setHttpOnly(true);
    cookie.setPath("/");
    cookie.setMaxAge(60); //1분
    cookie.setHttpOnly(true);

    response.addCookie(cookie);
}

private void setCookieSameSite(HttpServletResponse response) {
    Collection<String> headers = response.getHeaders(HttpHeaders.SET_COOKIE);
    boolean firstHeader = true;
    for (String header : headers) {
        if (firstHeader) {
            response.setHeader(HttpHeaders.SET_COOKIE, String.format("%s; %s", header, "SameSite=Strict"));
            firstHeader = false;
            continue;
        }
        response.addHeader(HttpHeaders.SET_COOKIE, String.format("%s; %s", header, "SameSite=Strict"));
    }
}
```

`JwtServiceImpl#sendToken` 메서드는 생성한 token들을 cookie에 저장하고, 이 쿠키들을 response에 다른 정보들과 함께 저장하는 메서드입니다. 여기서는 Cookie의 HttpOnly와 Samesite라는 속성을 강조하고 싶은데요, 이 속성들은 XSS, CSRF 취약점을 보완하기 위한 방법입니다.

- XSS(Cross-Site Scripting)

  공격자가 사용자가 이용하는 웹페이지에 **악의적인 스크립트를 삽입**하여 사용자가 의도하지 않은 프로그램을 실행하는 것을 말합니다. 일반적으로 Javascript를 통해 쿠키에 저장된 민감한 정보를 탈취하려고 합니다.

- CSRF(Cross-Site Request Forgery)

  이 취약점은 사용자가 A 사이트에 **로그인을 한 뒤** B 사이트를 이용할 때 공격자가 사용자의 A 사이트에 대한 **특정 요청(송금 요청, 개인정보 전달 등)을 유도**하는 공격 방식입니다.

  이 공격에는 쿠키가 이용되는데, 쿠키의 특정 도메인이나 URL에 요청을 보낼 때 자동으로 포함되는 특성 때문입니다. 서버에서는 사용자와 공격자의 요청을 구분할 수 없습니다. CSRF 방식으로 공격자가 서버에 요청을 보낼 때 쿠키에 사용자의 인증 정보가 포함되어 있다면 서버는 그 인증 정보로 사용자로 인식하고 응답을 제공합니다. 쿠키를 악용하면 cross-origin 간 read는 불가능해도 write는 된다는 것을 알 수 있습니다.

JWT를 클라이언트 측에서 안전하게 저장하기 위해서는 위의 대표적인 취약점에 대응해야 합니다. 다음은 JWT를 저장하는 2가지 방법입니다. 물론 이외에도 다른 방법들이 있습니다.

1. Response의 Authorization Header로 넣기

   - 장점 : CSRF 공격에 안전하다. 'Authrization' Header의 정보는 브라우저가 새로운 요청 시 자동으로 추가하기 않기 때문에다.

   - 단점 : 'Authrization' Header로 넘어온 JWT을 안전하게 저장해야한다. 브라우저의 local storage에 저장하면 공격자가 주입한 javascript가 local storage의 정보를 읽어 탈취할 수 있다(XSS 공격에 취약)

2. HttpOnly 속성의 Cookie에 저장하기

   - 장점 : 클라이언트 측의 javascript는 SOP(동일 출처 정책)으로 HttpOnly 속성의 쿠키에 접근할 수 없기 때문에 XSS 으로부터 정보탈취를 방지할 수 있다.

   - 단점 : CSRF 공격에 취약하다. 결국 Cookie에 저장하기 때문에 공격자의 요청 유도 시 사용자의 인증 정보가 포함된 Cokkie가 자동으로 포함되기 때문이다. 이 단점을 해소하기 위해서 CSRF Token 또는 SameSite 쿠키 정책 등을 사용한다. 하지만 각각 단점이 존재한다.
     - sameSite 설정
       - sameSite, 즉 같은 사이트 내에서의 요청에 쿠키를 함께 보내는 설정이다.
       - 같은 사이트인지(same site) 어떻게 판별할까? public suffix(com, net)에서 한 단계 내려 간 도메인(사용자들이 만들수 있어야)까지 (즉 www.test.com에서 test.com) 같으면 same site이다.
       - SameSite 상태 값
         - None: 모든 문맥에서 전부 전송 가능
         - Lax: 기본적으로 same site일 때 전송하지만 achor tag, href (top level navigation) 이동의 경우 쿠키 붙어서 감
           - Get 요청으로 데이터가 수정되는 일이 없어야.
         - strict: same site일 경에만 쿠키를 전송

어떤 방법을 사용해야 JWT를 클라이언트 측에서 안전하게 저장할 수 있을까요?

모두 상황에 따라 다를 것이라고 생각합니다. 제가 구현하는 웹 애플리케이션의 경우 외부 사이트(third party)으로 요청을 보낼 일이 없기 때문에 두 개의 토큰을 **Cookie**로 각각 클라이언트에 보내고 두 쿠키의 설정값을 **HttpOnly = true, SameSite = Strict**로 설정하는 것으로 결정했습니다.

```
@Override
public String extractAccessToken(HttpServletRequest request) {
    return Optional.ofNullable(request.getCookies())
            .flatMap(cookies -> Arrays.stream(cookies)
                    .filter(cookie -> ACCESS_TOKEN_COOKIE_NAME.equals(cookie.getName()))
                    .map(Cookie::getValue)
                    .findFirst())
                    .orElse(null);
}

@Override
public String extractRefreshToken(HttpServletRequest request) throws IOException, ServletException {
    return Optional.ofNullable(request.getCookies())
            .flatMap(cookies -> Arrays.stream(cookies)
                    .filter(cookie -> REFRESH_TOKEN_COOKIE_NAME.equals(cookie.getName()))
                    .map(Cookie::getValue)
                    .findFirst())
            .orElse(null);
}
```

위의 두 메서드는 사용자가 요청을 했을 때 인증 여부를 확인하기 위해 Cookie에 저장된 token을 꺼내는 메서드입니다.

```
@Override
public String extractUsername(String accessToken) {
    return JWT.require(Algorithm.HMAC512(secret))
        .build()
        .verify(accessToken)
        .getClaim(USERNAME_CLAIM).asString();
}

@Override
public boolean isValidAccessToken(String accessToken) {
    try {
        JWT.require(Algorithm.HMAC512(secret))
                .build()
                .verify(accessToken);
        return true;
    } catch (JWTVerificationException e) {
        return false;
    }
}

@Override
public Authentication getAuthentication(String accessToken) {
    UserDetails userDetails = userDetailsService.loadUserByUsername(extractUsername(accessToken));
    return new UsernamePasswordAuthenticationToken(userDetails, "", userDetails.getAuthorities());
}
```

`JwtServiceImpl#extractUsername`은 access token에 저장되어있는 유저의 정보(id)를 추출하는 메서드입니다.

`JwtServiceImpl#isValidAccessToken`는 access token이 위변조가 되었는지 확인합니다.

`isValidAccessToken#getAuthentication`는 access token으로 UserDetails 객체에 저장되어있는 사용자의 정보를 바탕으로 `Authentication`객체를 생성하여 반환합니다.

## 참조

- https://ttl-blog.tistory.com/272
