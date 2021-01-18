---
layout: post
title: Spring Security 인가
category: Spring
tags: [Spring, Spring Security, 인가, 로그인]
permalink: /spring/:year/:month/:day/:title/
comments: true
---

# Index

- Spring Security 인가
  - Spring Security 인가과정
  - AccessDecisionManager와 AccessDecisionVote
  - AccessDecisionManager의 구현체
    - AffirmativeBased
    - ConsensusBased
    - UnanimousBased
- WebSecurity 설정
- 정리
- References

---

# Spring Security 인가

![alt text](/public/img/spring/spring-security-architecture-simple.png "스프링 시큐리티 아키텍처 - 간단")

스프링 시큐리티 아키텍처를 매우 간략하게 그려본 그림이다.

**스프링 시큐리티는 크게 인증과 인가 부분으로 나뉘며 인증 처리는 Authentication Manager 컴포넌트가, 인가 처리는 AccessDecision Manager라는 컴포넌트가 담당하여 처리한다. 그리고 이러한 인증과 인가 작업은 일련의 필터 체인을 통해 이루어진다.**

여기서는 <u>인증된 사용자가 요청한 리소스에 접근 가능한지 판단하는 인가 처리 과정에 대해 알아보자.</u>

# Spring Security 인가과정

Spring Security 필터 체인의 마지막 서블릿 필터는 **FilterSecurityInterceptor** 이다.

이 필터에서 해당 요청의 접근 여부를 결정하게 된다.
![alt text](/public/img/spring/spring-security-filter-chain-details.png "Spring Security 필터 체인 상세")
출처 : https://atin.tistory.com/590

인가 처리란 인증 처리가 선행된 후에 진행되는 과정으로, **인증이 완료된 사용자를 대상으로 한다. 인증이 끝나면 유저가 가진 권한 정보가 Authentication 객체에 담겨져 전달되는데, 이 권한 정보를 참조해서 해당 요청의 승인 여부를 결정하게 된다.**

```
public interface Authentication extends Principal, Serializable {
    // 유저가 가진 권한 목록이 저장되어 있다. ex ROLE_USER, ROLE_ADMIN
    Collection<? extends GrantedAuthority> getAuthorities();

        Object getCredentials();

        Object getDetails();

        Object getPrincipal();

        boolean isAuthenticated();

        void setAuthenticated(boolean var1) throws IllegalArgumentException;
}
```

인증 담당의 Authentication Manager와 마찬가지로 Access Decision Manager 또한 인터페이스이며, 실질적인 처리는 AccessDecisionManager의 구현체들이 수행한다!

# AccessDecisionManager와 AccessDecisionVoter

인가 과정에서 AccessDecision Manager와 AccessDecision Voter라는 두 컴포넌트가 등장한다. **인과 과정은 우리가 의사결정을 위해 투표하는 방식과 매우 유사하다.** 투표 방법은 여러가지가 있는데 그 중 만장일치와 과반수 그리고 한 명이라도 동의하는 경우 3가지가 있다. **AccessDecisionManager은 이러한 결정 방법들을 구현체(알고리즘)로 제공해주고, voter들의 투표를 통해 접근 가능 여부를 판단한다.** Voter는 투표 결과로 GRANTED(승인), ABSTAIN(보류), DENIED(거부) 3가지 중 하나를 선택 할 수 있다.

```
public interface AccessDecisionVoter<S> {
  int ACCESS_GRANTED = 1;
  int ACCESS_ABSTAIN = 0;
  int ACCESS_DENIED = -1;

  boolean supports(ConfigAttribute var1);

  boolean supports(Class<?> var1);

  /*
   * 인자 정보: Authentication: 인증 정보, S (Object): 보호받는 자원(Url로 표현되는 리소스), Collection<ConfigAttribute>: 보호받는 자원과 관련된 설정들
   * voter 구현체마다 구체적인 로직을 넣을 수 있고, 그 voter들을 Manager에 주입해서 사용하면 된다.
   */
  int vote(Authentication var1, S var2, Collection<ConfigAttribute> var3);
}
```

```
public interface AccessDecisionManager {
  void decide(Authentication var1, Object var2, Collection<ConfigAttribute> var3) throws AccessDeniedException, InsufficientAuthenticationException;

  boolean supports(ConfigAttribute var1);

  boolean supports(Class<?> var1);
}
```

좀더 자세히 알아보자.

- AccessDecisionManager 인터페이스는 컴포넌트는 2가지 메소드를 제공한다.
- **supports**
  - AccessDecisionManager 구현체가 현재 요청을 지원하는지 여부를 판단하는 두개의 메소드를 제공한다.
  - 하나는 java.lang.Class 타입을 파라미터로 받고 다른 하나는 ConfigAttribute 타입을 파라미터로 받는다.
  - 인증 과정에서 ProviderManager의 supports 메소드 기능과 유사해 보인다.
- **decide**
  - request context와 security configuration을 참조하여 접근 승인 여부를 결정한다. 리턴값은 없지만, 접근 거부를 의미하는 예외를 던져 요청이 거부되었음을 알려준다.
- 인증된 사용자의 리소스 접근 여부를 판단하는 **3개의 기본 구현체를 제공한다**
  - **AffirmativeBased**: voter가 1개 이상 승인하면 접근 승인
  - **ConsensusBased**: 과반수가 승인하면 요청 승인
  - **UnanimousBased**: 모든 voter가 승인해야 요청 승인
- **AccessDecisionManager는 다수의 AccessDecisionVoter로 구성된다.**
  - RoleVoter는 보호 리소스에 접근하기 위한 권한을 사용자가 지니고 있는지 확인한다.

# AccessDecisionManager의 구현체

## AffirmativeBased

```

public class AffirmativeBased extends AbstractAccessDecisionManager {
  public AffirmativeBased(List<AccessDecisionVoter<?>> decisionVoters) {
      super(decisionVoters);
    }

  public void decide(Authentication authentication, Object object, Collection<ConfigAttribute> configAttributes) throws AccessDeniedException {
    int deny = 0;
    Iterator var5 = this.getDecisionVoters().iterator();

    while(var5.hasNext()) {
      AccessDecisionVoter voter = (AccessDecisionVoter)var5.next();
      int result = voter.vote(authentication, object, configAttributes); // voter가 투표한 결과
      if (this.logger.isDebugEnabled()) {
        this.logger.debug("Voter: " + voter + ", returned: " + result);
      }

      switch(result) {
      case -1:
        ++deny;
        break;
      case 1:
        return;
      }
    }

    if (deny > 0) {
      throw new AccessDeniedException(this.messages.getMessage("AbstractAccessDecisionManager.accessDenied", "Access is denied"));
    } else {
      this.checkAllowIfAllAbstainDecisions();
    }
  }
}
```

- 승인이 하나라도 있으면 종료하는 DecisionManager 구현체이다.

## ConsensusBased

```
public class ConsensusBased extends AbstractAccessDecisionManager {
  public ConsensusBased(List<AccessDecisionVoter<?>> decisionVoters) {
    super(decisionVoters);
  }

  public void decide(Authentication authentication, Object object, Collection<ConfigAttribute> configAttributes) throws AccessDeniedException {
    int grant = 0;
    int deny = 0;
    Iterator var6 = this.getDecisionVoters().iterator();

    while(var6.hasNext()) {
      AccessDecisionVoter voter = (AccessDecisionVoter)var6.next();
      int result = voter.vote(authentication, object, configAttributes);
      if (this.logger.isDebugEnabled()) {
        this.logger.debug("Voter: " + voter + ", returned: " + result);
      }

      switch(result) {
      case -1:
        ++deny;
        break;
      case 1:
        ++grant;
      }
    }

    if (grant <= deny) {
      if (deny > grant) {
        throw new AccessDeniedException(this.messages.getMessage("AbstractAccessDecisionManager.accessDenied", "Access is denied"));
      } else if (grant == deny && grant != 0) {
        if (!this.allowIfEqualGrantedDeniedDecisions) {
          throw new AccessDeniedException(this.messages.getMessage("AbstractAccessDecisionManager.accessDenied", "Access is denied"));
        }
      } else {
        this.checkAllowIfAllAbstainDecisions();
      }
    }
  }
   public void setAllowIfEqualGrantedDeniedDecisions(boolean allowIfEqualGrantedDeniedDecisions) {
    this.allowIfEqualGrantedDeniedDecisions = allowIfEqualGrantedDeniedDecisions;
  }
}
```

- 승인이 과반수 이상일 때 종료되는 DecisionManager 구현체이다.
- grant(승인의 수)와 deny(거절의 수)를 기반으로 검증한다.
- 동점일 때의 처리는 allowIfEqualGrantedDeniedDecisions를 통해 정할 수가 있다.

## UnanimousBased

```
public class UnanimousBased extends AbstractAccessDecisionManager {
  public UnanimousBased(List<AccessDecisionVoter<?>> decisionVoters) {
    super(decisionVoters);
  }

  public void decide(Authentication authentication, Object object, Collection<ConfigAttribute> attributes) throws AccessDeniedException {
    int grant = 0;
    List<ConfigAttribute> singleAttributeList = new ArrayList(1);
    singleAttributeList.add((Object)null);
    Iterator var6 = attributes.iterator();

    while(var6.hasNext()) {
      ConfigAttribute attribute = (ConfigAttribute)var6.next();
      singleAttributeList.set(0, attribute);
      Iterator var8 = this.getDecisionVoters().iterator();

      while(var8.hasNext()) {
        AccessDecisionVoter voter = (AccessDecisionVoter)var8.next();
        int result = voter.vote(authentication, object, singleAttributeList);
        if (this.logger.isDebugEnabled()) {
          this.logger.debug("Voter: " + voter + ", returned: " + result);
        }

        switch(result) {
        case -1:
          throw new AccessDeniedException(this.messages.getMessage("AbstractAccessDecisionManager.accessDenied", "Access is denied"));
        case 1:
          ++grant;
        }
      }
    }

    if (grant <= 0) {
      this.checkAllowIfAllAbstainDecisions();
    }
  }
}
```

- 권한이 한 개도 없으면 실패하는 DecisionManager 구현체이다.

# WebSecurity 설정

```
@Configuration
@EnableWebSecurity
public class WebSecurityConfigure extends WebSecurityConfigurerAdapter {

  @Override
    protected void configure(HttpSecurity http) throws Exception {
      http
        .csrf()
          .disable()
        .headers()
          .disable()
        .exceptionHandling() // 예외 핸들러 등록
          .accessDeniedHandler(accessDeniedHandler) // accessDeniedHandler()는 권한 체크에서 실패할 때 수행되는 핸들러
          .authenticationEntryPoint(unauthorizedHandler)  // authenticationEntryPoint()는 인증되지 않은 사용자가 보호된 리소스에 접근했을 때 수행되는 핸들러
          .and()
        .sessionManagement()
          .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
          .and()
        .authorizeRequests() // URL별 권한 관리 설정
          .antMatchers("/api/auth").permitAll()
          .antMatchers("/api/user/join").permitAll()
          .antMatchers("/api/**").hasRole(Role.USER.name())
          .accessDecisionManager(accessDecisionManager()) // 사용할 accessDecisionManager 주입 (커스터마이징)
          .anyRequest().permitAll()
          .and()
        .formLogin()
          .disable();
      http
        .addFilterBefore(jwtAuthenticationTokenFilter(), UsernamePasswordAuthenticationFilter.class); // 사용자 정의 필터
	}
}
```

- **@EnableWebSecurity** 어노테이션은 웹에서 시큐리티 기능을 사용하겠다는 어노테이션으로 부트에서는 관련 설정이 자동으로 적용된다.
- 자동 설정 그대로 사용할 수도 있지만, 요청, 권한, 기타 설정에 대해서는 필수적으로 최적화 설정이 필요하다. 이를 위해 WebSecurityConfigurerAdapter를 상속받고 configure 메소드를 오버라이드하여 원하는 형식의 시큐리티를 설정한다.
- **configure() 메소드 설정 프로퍼티**
  - authorizeRequests()을 통해 URL별로 권한을 다르게 설정할 수가 있다.
  - URL 패턴은 antMatchers()를 이용한다.
    ```
     .authorizeRequests() // URL별 권한 관리 설정
          .antMatchers("/api/auth").permitAll()
          .antMatchers("/api/user/join").permitAll()
          .antMatchers("/api/**").hasRole(Role.USER.name()) // WebExpressionVoter 구현체 통해 투표가 이뤄진다.
          .accessDecisionManager(accessDecisionManager()) // 사용할 accessDecisionManager 주입 (커스터마이징)
          .anyRequest().permitAll()
    ```
  - 위 코드에서 "/api"로 시작하는 요청 중 로그인/가입일 때는 제외하고, 모두 Role.User 권한을 갖고 있어야만 접근이 가능하다
  - `.anyRequest().permitAll()` - 위에서 설정한 요청("/api") 이외의 리퀘스트 요청은 모두 허가한다.

```
@Bean
public AccessDecisionManager accessDecisionManager() {
  List<AccessDecisionVoter<?>> decisionVoters = new ArrayList<>();
  decisionVoters.add(new WebExpressionVoter());
  decisionVoters.add(connectionBasedVoter()); // 직접 구현한 voter
  return new UnanimousBased(decisionVoters);
}
```

- **인가처리 커스터마이징**
  - 위 코드에서는 만장일치(UnanimousBased) 기반 AccessDecisionManager 구현체를 리턴하는 메서드이고, AccessDecisionManager는 Voter들의 리스트를 참조하고 있다.
  - 우리가 직접 구현한 voter 객체를 사용하고 싶다면, 사용할 accessDecisionManager 구현체의 Voter 리스트에 넣어줘야 한다.
  - 그 후 `.accessDecisionManager(accessDecisionManager()`처럼 사용할 manager 빈 객체를 accessDecisionManager()에 주입시켜준다.
- DecisionManager가 configure에 설정되어 있다면 FilterSecurityInterceptor 필터가 해당 DecisionManager를 호출해준다.
  - **DecisionManager를 따로 설정하지 않았다면, 디폴트로 AffirmativeBased(접근 승인 voter가 1개 이상) 기반의 Manager와 WebExpressionVoter의 voter 1개로 인가처리가 진행된다.**
  - WebExpressionVoter가 아까 configure()에 등록한 권한을 비교하여 투표한다
- DecisionManager는 N개의 Voter를 들고 있으며, 요청에 따른 Voter들의 투표결과에 따라 승인이 이뤄진다. 투표결과는 DecisionManager의 구현체에 따라 달라진다.

# 정리

- 인가 과정은 FilterSecurityInterceptor 필터에서 AccessDecision Manager와 Voter를 통해 이뤄진다. AccessDecisionManager는 AccessDecisionVoter 구현체들을 리스트로 들고 있으며, voter들의 투표 결과와 DecisionManager의 투표 정책에 따라 리소스 접근 여부가 결정된다.

- 설정 클래스에서 configure() 메소드를 오버라이딩해서 URL별 권한을 설정할 수 있다.

- 상황에 따라 Voter 구현체를 커스텀해서 사용할 수도 있다. 예를 들어 내가 올린 게시물을 본인과 친구만 볼 수 있게 하고 싶다면, 내 게시물에 대한 요청이 들어왔을 때, 요청한 주체가 게시물에 접근 권한이 있는지 확인해야 한다. 이때 특정 API 패턴(ex.게시물 조회 api)에 대해서만 확인을 하는 voter를 만든 후 Manager에 주입해주면 된다.

---

이전 포스트들을 통해서 지금까지 우리는 스프링 시큐리티의 동작원리 그리고 핵심 개념인 인증과 인가 처리 프로세스에 대해 알아 보았다.

인증이란 유저의 신원을 확인하는 과정이며 인가란 요청한 리소스에 접근할 수 있는 권한이 있는 유저인지 확인하는 과정이다.

스프링 시큐리티는 이러한 인증과 권한을 구현할 수 있도록 도와주는 프레임워크이기 때문에 주어진 틀내에서 우리 상황에 맞게 필요한 코드를 그 틀을 해치지 않으면서 잘 끼워맞춰 개발하면 된다.

## References

**스프링티 시큐리티 전반**<br>
프로그래머스 웹백엔드 구현 스터디<br>
https://www.boostcourse.org/web326/lecture/58997<br>
https://sjh836.tistory.com/165<br>

**스프링시큐리티 인가** <br>
https://zgundam.tistory.com/57<br>
https://velog.io/@allen/스프링-시큐리티에서의-인가<br>
