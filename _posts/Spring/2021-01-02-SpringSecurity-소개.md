---
layout: post
title: Spring Security 소개
category: Spring
tags: [Spring, Spring Security, 인증, 권한, 로그인, 필터 체인]
permalink: /spring/:year/:month/:day/:title/
comments: true
---

# Index

- Spring Security
  - 주요 개념과 용어
  - Spring Security 아키텍처
  - Spring Security 필터
- 정리
- References

---

# Spring Security

<br>

**로그인/로그아웃 기능은 어떻게 구현해야할까? 내가 올린 게시물을 친구들만 볼 수 있게 제한하려면 어떻게 해야할까?** 전자는 인증, 후자는 권한과 관련있는 개념인데 이러한 인증과 권한 처리를 편리하게 도와주는 도구가 바로 스프링 시큐리티(Spring Security)이다.

스프링 시큐리티란 **스프링 기반 어플리케이션의 보안(인증과 권한)을 담당하는 프레임워크**이다. 스프링 시큐리티를 사용하지 않는다면, 직접 세션(HttpSession)을 생성하고, 세션을 확인하고, 리다이렉트 등 필요한 모든 작업을 해줘야 한다. 따라서 보안 문제는 스프링 시큐리티에게 맡기고 우린 핵심 로직만 개발하면 된다.(물론 스프링 시큐리티의 번거로운 설정을 간소화시켜주는 래핑 프레임워크 스프링 부트 시큐리티도 있다. 이부분은 나중에 정리하겠다)

<!-- (여기서 프레임워크라는 말을 통해, 고정된 기본적인 틀이 짜여져있고 우리는 필요한 코드들을 주어진 틀에 맞게 끼워맞춰개발해야함을 의미한다.) -->

여기서는 Spring Security의 주요 용어들과 개념, 그리고 동작 원리에 대해 알아보자.

## 주요 개념과 용어

- **접근 주체(Principal)**: 보호된 리소스에 접근하는 사용자
- **인증(Authentication)**:
  - '증명하다'라는 의미로, **유저가 누구인지 신원을 확인하는 과정**
  - 유저 아이디와 비밀번호를 이용하여 로그인 하는 과정
  - 관련 컴포넌트: AuthenticationManager, AuthenticationProvider
- **인가(Authorization)**:
  - 인증된 사용자가 **리소스에 접근할 수 있는 권한이 있는지 확인** 또는 권한 부여\*\*(Access)
  - 관련 컴포넌트: AccessDecisionManager, AccessDecisionVoter
- **GrantedAuthority**
  - 인증된 사용자가 가지고 있는 권한 (ROLE)
- **SecurityContext**
  - 접근 주체(Aithentication)과 인증정보(GrantedAuthority)를 담고 있는 Context
  - **ThreadLocal 영역에 저장되고**, SecurityContextHolder(인메모리 세션 저장소) 통해 context 정보를 가져온다 -> 추가 필요

## Spring Security 아키텍처

![alt text](/public/img/spring/spring-security-architecture-simple.png "스프링 시큐리티 아키텍처 - 간단")

스프링 시큐리티 아키텍처를 매우 간략하게 그림으로 그려보았다. 큰 덩어리로 아키텍처를 먼저 훑어보고 나서 하나씩 설명해 나가겠다.

스프링 시큐리티 구조는 크게 인증과 인가 부분으로 나뉜다. 요청이 들어오면 먼저 인증 처리를 거치는데, 이는 Authentication Manager라는 컴포넌트가 담당한다. 인증 처리가 끝나야만 인가 처리가 수행되며 이는 AccesionDecision Manager라는 컴포넌트가 담당한다. **여기서 특이한 점은 인증과 인가가 여러개의 Filter들을 기반으로 수행된다는 것이다.**

## Spring Security 필터

기본 설정시 Spring Security는 일련의 서블릿 필터 체인으로 자동 구성된다. **필터란, 요청이 수행되기 전이나 수행된 후에 추가 작업을 수행할 수 있는 별도의 객체이다.(아래 그림 참조)** 스프링 MVC에서 요청을 가장 먼저 받게 되는 친구는 DispatcherServlet이다. 필터를 이용하게 되면 DispatcherServlet 보다 먼저 요청을 가로 채거나 요청 점검이 가능하다. 예를 들어 필터에서 클라이언트가 원래 요청한 자원이 아닌 다른 자원으로 redirect 시킨다거나 다음 필터에게 요청과 응답을 전달하지 않고 바로 클라이언트에게 응답하고 끝낼 수 있다.

![alt text](/public/img/spring/spring-security-filter-chain.png "필터 기본 구조")
출처 : https://youmekko.github.io/2018/04/26/2018-04-26-Filter/

스프링 시큐리티는 이렇게 여러 Filter들로 구성되어 동작된다. 각 필터들은 각자만의 역할이 있고 그 역할이 끝나면 다음 필터를 호출하게 된다. 이를 **필터 체인**이라 하는데, 수행 **순서는 서버의 설정 파일(web.xml)에 등록된 순서대로** 수행된다 -- 필터가 동작하려면 설정 파일에 그 필터가 어떤 페이지를 필터링하는지 작성해야한다. 그래야 요청이 들어왔을 때 해당 url과 맵핑되어 있는 필터들이 동작된다. 설정 파일 대신 @WebFilter 어노테이션으로도 필터 등록이 가능하다 -- **FilterChain은** 필터가 실행될 때 doFilter() 메소드의 인자로 전달되는 객체로, **설정 파일에 설정된 모든 필터 정보를 가지고 있어 필터들의 실행 순서를 알고 있다**. 따라서 우리는 FilterChain 객체의 doFilter() 메소드만 호출하면 알아서 다음 필터가 호출된다.

### 필터 객체 기본 구조

```
 public class SampleFilter implements Filter {
    ..
    public void doFilter(ServletRequset request, ServletResponse response, FilterChain chain) throws IOException, ServletException{
        //1. 서블릿 수행전에 수행할 코드: request 파라미터 이용해 요청의 필터 작업을 수행
        //2. FilterChain 객체를 이용하여 다음 필터 호출
        chain.doFilter(req, res);
        //3. 서블릿 수행 후에 수행할 코드: response 파라미터 이용해 응답의 필터 작업 수행
    }
    ..
 }
```

### 필터 체인 상세

Spring Security를 각자의 상황에 맞게 커스텀하기 위해서는 아래 필터 체인을 이해하는 것이 좋다. 하지만, 처음 부터 모두 알고 이해하기에는 수가 많고 난해하므로 지금은 이런게 있구나 정도로만 넘어가자. 자세한 건 로그인 관련 개발을 직접 해보면서 여러번 반복해서 보는게 더 효과적일 것이다. 아래 그림은 Spring Security 필터 체인과 각 필터에서 사용하는 객체들(Repository, Handler, Manager 등)에 대해 잘 표현해주는 그림이 있어 가져왔다.

![alt text](/public/img/spring/spring-security-filter-chain-details.png "Spring Security 필터 체인 상세")
출처 : https://atin.tistory.com/590

### 스프링 시큐리티가 제공하는 필터들의 역할

- **SecurityContextPersistenceFilter**: SecurityContextRepository에서 SecurityContext를 가져오거나 저장하는 역할
- **LogoutFilter**: 설정된 로그아웃 URL(기본값:/logout)로 오는 요청을 감시하여, 해당 유저를 로그아웃 시킴
- **(UsernamePassword)AuthenticationFilter**: (아이디와 비밀번호를 사용하는 form 기반 인증) 설정된 로그인 URL(기본값: /login) 요청을 감시해 유저를 인증함
  - AuthenticationManager 통한 인증 실행
  - 인증 성공 시, 얻은 Authentication 객체를 SecurityContext에 저장한 후 AuthenticationSuccessHandler 실행
  - 인증 실패 시, AuthenticationFailureHandler 실행
- DefaultLoginPageGeneratingFilter: 인증 위한 로그인폼 URL 감시
- BasicAuthenticationFilter: HTTP 기본 인증 헤더를 감시하여 처리
- RequestCacheAwareFilter: 로그인 성공 후, 원래 요청 정보를 재구성하기 위해 사용
- SecurityContextHolderAwareRequestFilter: HttpServletRequestWrapper를 상속한 SecurityContextHolderAware RequestWapper 클래스로 HttpServletRequest 정보를 감싼다. SecurityContextHolderAwareRequestWrapper 클래스는 필터 체인상의 다음 필터들에게 부가정보를 제공
- AnonymousAuthenticationFilter : 호출되는 시점까지 사용자 정보가 인증되지 않았다면 인증토큰에 사용자가 익명 사용자로 나타남
- SessionManagementFilter : 인증된 사용자와 관련된 모든 세션을 추적
- **ExceptionTranslationFilter** : 보호된 요청을 처리하는 중에 발생할 수 있는 예외를 위임하거나 전달하는 역할
- **FilterSecurityInterceptor** : 접근 권한 확인을 위해 요청을 AccessDecisionManager로 위임. 이 필터가 실행되는 시점에는 사용자가 인증됐다고 판단

글이 길어지므로 일단 여기서 글을 마치겠다.

# 정리

지금까지 우리는 스프링 시큐리티의 역할, 주요 개념과 용어들, 간략한 스프링 시큐리티 아키텍처와 함께 필터에 대해 알아보았다.

다음 포스트에서는 실제 스프링 시큐리티가 인증과 인가 처리를 위해 하는 과정들을 자세히 살펴보겠다.

## References

**스프링티 시큐리티 전반**<br>
프로그래머스 웹백엔드 구현 스터디<br>
https://www.boostcourse.org/web326/lecture/58997<br>
https://sjh836.tistory.com/165<br>

**Filter와 Filter Chain**<br>
처음 해보는 Servlet & JSP 웹 프로그래밍<br>
