---
title: Spring Security FilterChain 등록시 주의사항
date: 2024-07-31 10:30:00 +09:00
categories: [Trouble Shooting]
tags:
  [
    java,
    Spring,
    스프링,
    Spring Security,
    에러해결,
    Trouble Shooting,
    FilterChain,
  ]
---

* * *

> webIgnorePatterns + FilterChain을 등록할 때 주의할 점을 알아보자.

## ✅ 문제 상황
```java

// 필터 빈 등록
@Component
class JwtAuthenticationFilter extends OncePerRequestFilter { } 

// 필터 등록
.addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class)

  // 필터 거치지 않도록 설정
@Bean
public WebSecurityCustomizer webSecurityCustomizer() {
  return web -> web.ignoring().requestMatchers(
    "/h2-console/**",
    "/v3/api-docs/**"
}

```

위와 같이 설정된 상황에서 webIgnorePatterns에 등록된 URL은 FilterChain을 거치지 않는다.  
라고 알고 있었는데, 실제로는 JwtAuthenticationFilter가 거쳐서 문제가 발생했다.

분명 debug로 확인해보아도  

```text
Security filter chain: [] empty (bypassed by security='none')
```

위와 같이 Security filter chain이 비어있는 것을 확인했는데도 JwtAuthenticationFilter가 거쳐서 문제가 발생했다.

<br><br>


## ✅ 해결 방법

CustomFilter를 Bean으로 등록시 해당 필터는 security Filter Chain에 포함되는게 아니라,  
default Filter Chain에 포함되어서 발생하는 문제였다.

web.ignorePatterns()로 등록된 URL은 security Filter Chain은 거치지 않지만,  
default Filter Chain은 거치게 되어서 문제가 발생했다.

해결 방법은 다음과 같다.

```java
.add FilterBefore(new JwtAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class)
```

위와 같이 Bean으로 등록하지 않고, 직접 생성하여 등록하면 문제가 해결된다.

<br><br>

### 마치며
* FilterChain의 동작과정에 대해 다시 한번 공부할 수 있는 기회였다.






