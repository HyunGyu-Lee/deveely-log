---
layout: post
title: "Spring Security + JWT를 활용한 토큰 기반 인증 구현 (with Spring Boot)"
date:   2020-06-16 13:04:19
category: backend-spring-framework
tags: [Spring Framework,Spring Security]
comments: true
draft: false
---
![Image1](./images/common/spring-security.png)
Spring Security는 Spring Framework 기반 웹 애플리케이션의 보안을 담당하는 프레임워크입니다. 
> Spring Security is a framework that provides authentication, authorization, and protection against common attacks. With first class support for both imperative and reactive applications, it is the de-facto standard for securing Spring-based applications.

웹 보안은 기본적으로 요청하는 사용자를 식별하는 인증(Authenticate)과 인증된 사용자가 보호된 리소스에 접근할 권한이 있는지 확인하는 인가(Authorize)이 기본 바탕입니다.
이 글에서는 여러 인증방식 중 REST 서비스 등에서 주로 이용되는 토큰 기반 인증을 소개하고 구현하는 과정을 보여드리고자 합니다.

(개인적으로 공부하면서 해본 것을 기록한 것이라 구현 과정이나 글 내용에 오류가 있을 수 있습니다. 잘못된 부분이 있다면 댓글로 알려주시면 감사하겠습니다!)

# 토큰 기반 인증? JWT?
![Image2](./images/2020-06-16-spring-security-with-jwt/1.png)

먼저 JWT는 JSON Web Token의 약자로 Json형태로 표현된 정보를 전달하는 하나의 방식으로, 토큰 자체에 모든 정보(토큰 기본정보, 전달할 정보, 검증됐다는 시그니쳐 정보 등)를 
스스로 지니고 있다는 것이 큰 특징입니다. (자가 수용적, Self-Contained)
웹 서버의 경우 헤더나 파라미터를 통해 손쉽게 넘길 수 있어 인증이 필요한 REST 서비스에서 주로 활용됩니다.

토큰 기반 인증은 주로 클라이언트와 연결고리가 없는 모던 웹 서비스에서 사용하는 인증방식입니다.
백엔드는 상태가 없는(stateless) REST 서비스로 구성하기 때문에 로그인 정보나 기타 정보를 세션에 저장하지 않습니다. 
상태가 없이 서버로 들어오는 클라이언트의 요청으로 판단하기 위해 요청에 포함된 토큰을 해석하고, 권한이 있는 유저인지 판단하여 서버의 리소스에 접근하는 것을 허용합니다.

인증 과정을 간략하게 얘기하면 
1. 클라이언트가 인증 API를 통해 인증 요청
2. 서버는 인증 진행 후 유효한 경우 토큰 발급 (JWT)
3. 다음 요청 시 인증 토큰을 요청에 포함시켜 요청
4. 서버는 요청에 포함된 인증 토큰을 해석해 권한 검사

# Spring Security 핵심 개념과 토큰 기반 인증
토큰 기반 인증을 Spring Security로 구현하기 위해 핵심 개념을 아래와 같이 간략하게 정리해봤습니다.
- 접근 주체 (Principal)
  - 보호된 리소스에 접근하려는 주체
- 인증 (Authenticate, Authentication)
  - 접근 주체가 누구인지 식별하는 과정 
  - 식별이 완료된 정보는 인증 객체로 표현되어 이후 절차에서 참조될 수 있도록 전파됨
- 인가 (Authorize)
  - 현재 사용자가 보호된 리소스에 대해 권한이 있는지 검사
  - 인증을 통해 확인한 사용자가 보유한 권한(Roles)을 확인해 요청한 서비스(API)를 수행할 권한이 있는지 확인하는 과정
- 권한
  - 인증된 접근 주체가 요청한 서비스를 수행할 자격이 있음을 증명하는 것

토큰 기반 인증은 `인증`을 토큰 기반으로 수행한다는 것입니다.
즉 요청에 포함된 토큰을 해석하고, 해당 사용자가 누구인지 확인하고 인증 객체를 생성해 프레임워크에 넘겨주는 과정을 구현해보도록 하겠습니다.

# 예제 진행
- Spring Boot 2.3.1
- Lombok

### 1. Spring Security 의존성 추가
먼저 Spring Boot 프로젝트에 security 의존성을 추가해줍니다.

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### 2. Security 설정
기본적인 REST API 서버로 사용하기 위한 설정만 해둔 상태입니다.
이번에 다루는 주요 주제가 Security 설정이 아니기때문에 간략한 설명만 아래 코드에 주석으로 달아두겠습니다.

```java
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

	private static final String[] PUBLIC_URI = {
			"/some-public-apis"
	};

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
      // 개발 편의성을 위해 CSRF 프로텍션을 비활성화
			.csrf()
				.disable()
      // HTTP 기본 인증 비활성화
			.httpBasic()
				.disable()
      // 폼 기반 인증 비활성화  
			.formLogin()
				.disable()
      // stateless한 세션 정책 설정  
			.sessionManagement()
				.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
				.and()
      // 리소스 별 허용 범위 설정  
			.authorizeRequests()      
				.antMatchers(PUBLIC_URI)
					.permitAll()
				.anyRequest()
					.authenticated()
				.and()
      // 인증 오류 발생 시 처리를 위한 핸들러 추가  
			.exceptionHandling()
				.authenticationEntryPoint(new RestAuthenticationEntryPoint())
		;
	}
}
```

### 3. JWT 라이브러리 추가 및 토큰 인증 클래스 정의
기본적인 설정은 마치고, 본격적으로 토큰 기반 인증을 구현합니다.
먼저 JWT를 사용하기 위해 의존성을 추가해줍니다.
```xml
<dependency>
  <groupId>io.jsonwebtoken</groupId>
  <artifactId>jjwt</artifactId>
  <version>0.5.1</version>
</dependency>
```

다음 인증 토큰을 표현한 인터페이스와 구현 클래스를 간단히 작성합니다.

```java
public interface AuthenticationToken {
    String getToken();
}

@Builder
@Getter
public class JwtAuthenticationToken implements AuthenticationToken {
    private String token;
}
```

### 4. 인증 토큰 처리 인터페이스 및 구현 클래스 작성
인증에 사용되는 토큰 유형이 달라지거나, 인증 토큰을 획득하는 방식 등 변경 가능성이 있는 부분이 많다고 생각이 들어서 인터페이스를 먼저 정의했습니다.
각 메소드에 대한 설명은 주석으로 대체하겠습니다.

```java
public interface AuthenticationTokenProvider {

	/***
	 * HTTP 요청에서 토큰 취득
	 * @param request HTTP 요청
	 * @return 토큰
	 */
	String parseTokenString(HttpServletRequest request);

	/***
	 * 토큰 발급
	 * @param userNo 유저 No
	 * @return 토큰
	 */
	AuthenticationToken issue(Long userNo);

	/***
	 * 토큰에서 userNo 취득
	 * @param token 토큰
	 * @return userNo
	 */
	Long getTokenOwnerNo(String token);

	/***
	 * 토큰 유효성 검사
	 * @param token 토큰
	 * @return 유효여부
	 */
	boolean validateToken(String token);

}
```

다음은 구현 클래스입니다. 위에서 정의한 `AuthenticationTokenProvider`를 impletements하여 구체적인 동작을 정의해줍니다.
```java
@Component
public class JwtAuthenticationTokenProvider implements AuthenticationTokenProvider {
    private static final Logger logger = LoggerFactory.getLogger(JwtAuthenticationTokenProvider.class);

    // 보통 설정파일에 관리하고 `@Value` 등으로 주입받아 사용하는 것을 추천
    private static final String SECRET_KEY = "SOME_SECRET_KEY";
    private static final long EXPIRATION_MS = 1000 * 60 * 60 * 24; 

    @Override
    public String parseTokenString(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        if (bearerToken != null && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        return null;
    }

    @Override
    public AuthenticationToken issue(Long userNo) {
        return AuthenticationToken.builder().token(buildToken(userNo)).build();
    }

    // JWT 토큰 생성
    private String buildToken(Long userNo) {
        LocalDateTime now = LocalDateTime.now();
        LocalDateTime expiredAt = now.plus(EXPIRATION_MS, ChronoUnit.MILLIS);
        return Jwts.builder()
                .setSubject(String.valueOf(userNo))
                .setIssuedAt(Date.from(now.atZone(ZoneId.systemDefault()).toInstant()))
                .setExpiration(Date.from(expiredAt.atZone(ZoneId.systemDefault()).toInstant()))
                .signWith(SignatureAlgorithm.HS512, SECRET_KEY)
                .compact();
    }

    @Override
    public Long getTokenOwnerNo(String token) {
        Claims claims = Jwts.parser()
                .setSigningKey(SECRET_KEY)
                .parseClaimsJws(token)
                .getBody();
        return Long.parseLong(claims.getSubject());
    }

    @Override
    public boolean validateToken(String token) {
        if (StringUtils.isNotEmpty(token)) {
            try {
                Jwts.parser().setSigningKey(SECRET_KEY).parseClaimsJws(token);
                return true;
            } catch (SignatureException e) {
                logger.error("Invalid JWT signature", e);
            } catch (MalformedJwtException e) {
                logger.error("Invalid JWT token", e);
            } catch (ExpiredJwtException e) {
                logger.error("Expired JWT token", e);
            } catch (UnsupportedJwtException e) {
                logger.error("Unsupported JWT token", e);
            } catch (IllegalArgumentException e) {
                logger.error("JWT claims string is empty.", e);
            }
        }
        return false;
    }
}
```

### 5. Spring Security 인증 절차에 추가하기
1 ~ 4번 과정을 통해 토큰 기반 인증을 수행하는데 필요한 작업을 진행했습니다.
이제 실제 인증을 수행하는 검사기를 만들고, 인증 절차에 추가해주어야 하는데요.
여기서는 Filter를 통해 검사하고 필터체인에 등록하도록 하겠습니다.

```java
public class TokenAuthenticationFilter extends OncePerRequestFilter {
	@Autowired
	private AuthenticationTokenProvider authenticationTokenProvider;

	@Autowired
	private UserService userService;

	@Override
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {
		String token = authenticationTokenProvider.parseTokenString(request);
		if (authenticationTokenProvider.validateToken(token)) {
			Long userNo = authenticationTokenProvider.getTokenOwnerNo(token);
			try {
				User member = (User) userService.loadUserByUsername(userNo.toString());
				UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(member,
						member.getPassword(), member.getAuthorities());
				authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
				SecurityContextHolder.getContext().setAuthentication(authentication);
			} catch (UsernameNotFoundException e) {
				throw new ForbiddenException("유효하지않은 인증토큰 입니다. 인증토큰 회원 정보 오류");
			}
		}
		filterChain.doFilter(request, response);
	}
}
```
인증에 필요한 기능을 제공해주는 `AuthenticationTokenProvider`를 통해 인증을 수행하는 로직을 구현합니다.
토큰이 유효한지 확인 후 회원정보를 조회 및 인증 객체를 생성해 프레임워크에 전달하는 간단한 로직입니다.

다음으로는 이 필터를 아래와 같이 추가해 검사가 수행될 수 있도록 해줍니다.
```java
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

	private static final String[] PUBLIC_URI = {
			"/some-public-apis"
	};

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
      // 개발 편의성을 위해 CSRF 프로텍션을 비활성화
			.csrf()
				.disable()
      // HTTP 기본 인증 비활성화
			.httpBasic()
				.disable()
      // 폼 기반 인증 비활성화  
			.formLogin()
				.disable()
      // stateless한 세션 정책 설정  
			.sessionManagement()
				.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
				.and()
      // 리소스 별 허용 범위 설정  
			.authorizeRequests()      
				.antMatchers(PUBLIC_URI)
					.permitAll()
				.anyRequest()
					.authenticated()
				.and()
      // 인증 오류 발생 시 처리를 위한 핸들러 추가  
			.exceptionHandling()
				.authenticationEntryPoint(new RestAuthenticationEntryPoint())
		;

    // 토큰 인증 필터 추가
    http.addFilterBefore(tokenAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
	}

  @Bean
	public TokenAuthenticationFilter tokenAuthenticationFilter() {
		return new TokenAuthenticationFilter();
	}
}
```

# 마치며
토큰 인증 검사와 Spring Security를 사용해 구현하는 방법을 아주 간단하게 알아봤습니다.
Spring Security는 설계가 굉장히 탄탄하게 되어있고, 인증 절차 전 영역에 걸쳐 개발자가 원하는 지점을 커스터마이징 하기 쉽게 되어있습니다.
다만 이번 공부를 하면서.. `Spring Security를 사용해 구현`한다는게 Filter를 사용한 부분밖에 없고, Security의 핵심개념을 잘 살리지 못한 것 같아 아쉬웠습니다.
이 부분은 좀 더 공부해서, 더 나은 방법으로 구현하는 방식을 포스팅해야겠다는 생각이 듭니다.