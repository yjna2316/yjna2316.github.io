---
layout: post
title: Spring Security 인증
category: Spring
tags: [Spring, Spring Security, 인증, 로그인]
permalink: /spring/:year/:month/:day/:title/
comments: true
---

# Index

- Spring Security 인증
  - Spring Security 아키텍처
  - Spring Security 인증 과정
- 정리
- References

---

# Spring Security 인증

## Spring Security 아키텍처

![alt text](/public/img/spring/spring-security-architecture-simple.png "스프링 시큐리티 아키텍처 - 간단")

이전 포스트에서 스프링 시큐리티 아키텍처를 매우 간략하게 그려본 그림이다.

**스프링 시큐리티는 크게 인증과 인가 부분으로 나뉘며 인증 처리는 Authentication Manager 컴포넌트, 인가 처리는 AccesionDecision Manager라는 컴포넌트가 담당하여 처리한다. 그리고 이러한 인증과 인가 작업은 일련의 필터 체인을 통해 이루어진다.**

여기서는 스프링 시큐리티가 어떻게 사용자의 신원을 확인하고 있는지 알아보자.

## Spring Security 인증 과정

스프링 시큐리티가 사용자 인증을 위해 하는 과정은 다음과 같다.
![alt text](/public/img/spring/spring-security-authentication-process.png "스프링 시큐리티 아키텍처1 - 인증 과정")

출처 : http://www.springbootdev.com

1. 유저가 로그인을 시도한다.
2. ID/PASSWORD로 로그인한 경우 AuthenticationFilter(UsernamePasswordAuthenticationFilter)에서 인증을 처리한다.
   - UsernamePassAuthenticationFilter는 /login에 응답하는 필터이다.
   - 인증 필터는 인증 메커니즘 모델에 따라 여러 종류가 있다.
     - 여기서는 ID/Password 기반 인증을 예로 들었지만, HTTP Basic authentication 요청은 BasicAuthenticationFilter에서, HTTP Digest authentication은 DigestAuthenticatonFilter에서 등 각 요청의 인증 메커니즘과 관련된 필터에서 인증 처리가 수행된다.
     - HTTP 인증 참고: https://developer.mozilla.org/ko/docs/Web/HTTP/Authentication
3. 요청에서 유저ID와 패스워드를 추출해 Authentication Token을 생성한 후 Authentication Manager에게 넘겨준다.

- Authentication 토큰은 사용자 인증정보를 나타내는 객체이다. 인터페이스로 구성되며 우리는 구현체인 UsernamePasswordAuthenticationToken을 사용한다.

  ```
  public interface Authentication extends Principal, Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();

    Object getCredentials();

    Object getDetails();

    Object getPrincipal();

    boolean isAuthenticated();

    void setAuthenticated(boolean var1) throws IllegalArgumentException;
  }
  ```

- 토큰은 AuthenticationManager의 authenticate 메소드로 전달한다.

  ```
  Authentication authentication = authenticationManager.authenticate(token);
  ```

  - 그런데 사실 **AuthenticationManager는 인터페이스라서 실제 인증은 구현체인 ProviderManager가 수행한다**

  ```
  public interface AuthenticationManager {
    Authentication authenticate(Authentication authentication) throws AuthenticationException;
  }
  ```

  ```
  public class ProviderManager implements AuthenticationManager, MessageSourceAware, InitializingBean {
    ...
    private List<AuthenticationProvider> providers; // ProviderManager는 실제 인증을 수행하는 provider들을 리스트로 들고 있다.
    ...
  }
  ```

<br>
4.**구현체 ProviderManager는 인증을 수행할 일꾼(Provider) 정보들을 리스트로 들고 있는데**, 이 리스트를 순회하면서 해당 요청을 인증해 줄 수 있는 provider를 찾아 인증을 처리한다.

- provider의 supports 메소드를 통해 true를 반환하는 provider를 찾아 그 친구에게 인증 처리를 시킨다.
- 역시나 provider 또한 AuthenticationProvider 인터페이스의 구현체이고 수 개의 구현체들이 있다. ID/Password의 경우 구현체 DaoAuthenticationProvider가 인증 처리를 해준다.

  ```
  public interface AuthenticationProvider {

      Authentication authenticate(Authentication authentication) throws AuthenticationException;

      boolean supports(Class<?> authentication);
  }
  ```

<br>
5.이제 DB에 있는 사용자 정보와 비교를 해보자. 
  * UserDetailsService를 통해 DB에서 사용자 정보를 읽어온다.
  * UserDetailsService 또한 인터페이스이다. 따라서 해당 인터페이스를 구현한 빈을 생성하면 스프링 시큐리티는 해당 빈을 사용하게 된다. 
  즉, 어떤 데이터베이스(인메모리, jdbc, 등)로 부터 읽어들일지 개발자가 결정할 수 있다. 
    ~~~
    public interface UserDetailsService {
      UserDetails loadUserByUsername(String username) throws UsernameNotFoundException; 
    }
    ~~~

<br>
6.UserDetailsService는 로그인한 ID에 해당하는 정보를 DB에서 읽어들여 응답을 위한 Authentication 토큰(유저 인증 정보를 담은 객체)을 새로 생성해 반환해준다.

- 인증이 성공하면 토큰에 유저의 권한 정보(ex. ROLE\_\*)와 함께 인증되었음을 표시해준다. Authentication.isAuthenticated = true
- 인증된 UserDetails 정보(Authentication 토큰)을 인메모리 세션 저장소인 SecurityContextHolder에 저장한다.

```
  SecurityContextHolder.getContext().setAuthentication(authentication);
```

- JWT 토큰이 아닌 세션을 사용하는 경우 유저에게 sesson ID(JSESSION ID)와 함께 응답을 해준다. 이후 요청에서는 요청 쿠키에서 JSESSION ID 정보를 통해 이미 로그인 정보가 저장되어 있는지 확인해서 있다면 인증을 통과 시킨다.

다음은 위 과정을 나타낸 또 다른 그림들이다. 잘 정리되어 있어서 가져왔다.

![alt text](/public/img/spring/spring-security-authentication-process1.png "스프링 시큐리티 아키텍처2 - 인증 과정")
출처: https://sjh836.tistory.com/165

![alt text](/public/img/spring/spring-security-authentication-process2.png "스프링 시큐리티 아키텍처3 - 인증 과정")
출처: https://springsource.tistory.com/80
<br>

드디어 인증 과정이 끝났다. 이제 다음 포스트에서 인가 과정에 대해 알아보자.

# 정리

지금까지 우리는 스프링 시큐리티의 동작원리 그리고 인증 처리 프로세스에 대해 알아 보았다.

- 인증이란 유저의 신원을 확인하는 과정이다.

- 스프링 시큐리티는 필터 기반으로 동작하며, 인증 과정은 AuthenticationFilter에서 AuthenticationManager, AuthenticationProvider(s), UserDetailsService를 통해 이루어진다. 여기서 특이한 점은 인증 전과 인증 후에 Authentication Token이 만들어지는데 principal 내용이 달라진다는 것이다. (인증전 principal 타입은 String, 인증 후 principal 타입은 Object)

- 만약 JWT 토큰을 사용하기 위해 우리가 직접 구현을 해야한다면(커스텀) 기존 시큐리티의 주어진 구조를 따르면서 필요한 코드를 집어 넣어 개발해야 한다. 대신 스프링은 우리가 새로 구현한 내용들은 - ex) AuthenticationManager가 우리가 만든 provider를 사용할 수 있도록 - 설정 파일에 빈으로 등록해주면 된다.

다음 포스트에서는 인가 처리 과정에 대해 알아보자.

## References

**스프링티 시큐리티 전반**<br>
프로그래머스 웹백엔드 구현 스터디<br>
https://www.boostcourse.org/web326/lecture/58997<br>
https://sjh836.tistory.com/165<br>
https://springsource.tistory.com/80<br>
**Authentication 아키텍처와 프로세스** <br>
https://springbootdev.com/2017/08/23/spring-security-authentication-architecture/#more-54<br>
