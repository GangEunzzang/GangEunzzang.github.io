---
title: Dispatcher Servlet 톺아보기
date: 2025-07-18 10:30:00 +09:00
categories: [DeepDive]
tags:
  [
    Java,
    Spring,
    SpringBoot,
    SpringMVC,
    Servlet,
    DispatcherServlet,
    DeepDive,
    딥다이브,
    톺아보기
  ]
---

`Dispatcher Servlet`은 Spring MVC의 핵심 컴포넌트로, 
`Front Controller` 이라고도 불립니다.  

위 글에서는 `Dispatcher Servlet`의 역할과 동작방식 구현이유 등을 톺아보겠습니다.


* * *

###  🍳 기술스택 
- Java 21
- Spring Boot 3.2.5
- Spring Boot Starter Web


## ✅ DisPatcher Servlet의 진화 과정

### 1. HttpServlet 직접 사용

`HttpServlet`을 직접 사용하여 요청을 처리하는 방식은 다음과 같습니다.

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.getWriter().write("Hello, World!");
    }
}
```

이렇게 매번 `HttpServlet`을 상속받아 구현해줘야 했고 보일러 플레이트 코드 발생과,  
공통 요청 처리 로직을 중복으로 작성해야 하는 문제가 있었습니다.


## ✅ 바이너리 로그

### 📌 Mysql에 쿼리 실행

```shell
# -f: force 실행(여러가지 충돌이 날 수 있어 강제 처리)
mysql -u root -p -f testDB < binlog.sql
```

데이터를 조회하면 정상적으로 복구된 것을 확인할 수 있다.

* * *

<br><br>

## 마치며
역시 백업은 중요한 것 같다. 위 예제는 간단한 예제이지만 실무에서는 더욱 복잡하게 얽혀있을 것이다.  
유실된 데이터를 복구하는 것도 중요하지만, 데이터를 실수로 날리는 것을 방지하는 것도 중요한 것 같다.


