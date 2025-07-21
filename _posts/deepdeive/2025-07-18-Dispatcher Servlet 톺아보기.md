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
    DeepDive
  ]
---

`Dispatcher Servlet`은 Spring MVC의 핵심 컴포넌트로, 
`Front Controller` 이라고도 불립니다.  

위 글에서는 `Dispatcher Servlet`의 역할과 동작방식 구현이유 등을 톺아보겠습니다.

##  🍳 분석 환경
- Java 21
- Spring Boot 3.2.5
- Spring Boot Starter Web

* * *


## ✅ DisPatcher Servlet의 진화 과정
  
`Servlet`이전 `CGI(Common Gatewat Interface)` 라는 기술이 존재하지만, 너무 과거 기술이므로 생략하겠습니다.

### 1. Servlet의 등장 (Java에 HTML을 담다)

`HttpServlet`을 사용하여 요청을 처리하는 방식은 다음과 같습니다.

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.getWriter().write("Hello, World!");
        response.setContentType("text/plain");
        response.setCharacterEncoding("UTF-8");
    }
}
```

위와같이 매번 `HttpServlet`을 상속받아 구현해줘야 했고 보일러 플레이트 코드 발생과,  
공통 요청 처리 로직을 중복으로 작성해야 하는 문제가 있었습니다.

그리고 자바 코드에서 `HTML`영역을 직접 작성해야 됐기 때문에, 유지보수성이 매우 떨어졌습니다.

<br> 

### 2. JSP의 등장 (HTML에 Java를 담다)

`Servlet`의 문제점을 해결하기 위해 `JSP(Java Server Pages)`가 등장했습니다.

`JSP`는 HTML과 Java 코드를 혼합하여 작성할 수 있는 기술로 HTML을 더 쉽게 작성할 수 있게 해주었습니다.

```html
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Hello JSP</title>
</head>
<body>
    <h1>Hello, JSP!</h1>
    <p>Current time: <%= new java.util.Date() %></p>

</body>
</html>
```

위와 같이 기본 `HTML` 영역을 작성하고, `<%= %>` 구문을 사용하여 `Java` 코드를 삽입할 수 있었습니다.

jsp의 등장은 자바 단에서의 뷰 영역 코드를 작성하지 않아, 유지보수성이 높아졌습니다.  
하지만 여전히 `JSP`는 비즈니스 로직과 뷰가 혼합되어 있다는 문제가 존재합니다.

<br>

### 3. MVC의 등장 

초기 `JSP`에는 여러가지 코드들이 다 모여있었습니다.  
db 접근 코드, 비즈니스 로직, 뷰 코드 등등...

이렇게 여러 객체들이 한 jsp 안에 모여있으니 재사용성과 유지보수성이 떨어졌습니다.

이때 `MVC(Model-View-Controller)` 패턴이 등장했습니다.

`MVC`는 비즈니스 로직과 뷰를 분리하여, 유지보수성과 재사용성을 높이는 패턴입니다.

서버쪽 코드는 자바쪽에만 작성하고, JSP 는 오로지 뷰 영역만 작성하는 방식으로 변경되었습니다.

하지만 아직도 한개의 `Servlet`을 매번 작성해야 했고,  
보일러 플레이트 코드가 발생하는 문제는 여전히 존재했습니다.

<br>

### 4. Spring의 등장
위의 모든 문제를 해결하고자 `Spring Framework`가 등장했습니다.

`Spring Framework`는 `Dispatcher Servlet`을 통해 모든 요청을 중앙 집중식으로 처리할 수 있게 해주었습니다.

매번 `Servlet`을 작성할 필요 없이, `@Controller` 어노테이션만 붙여주면 자동으로 빈으로 등록하고 

`Dispatcher Servlet`이 요청을 받아 적절한 핸들러로 매핑까지 해줍니다.

이제 `Dispatcher Servlet`이 어떻게 구현됐고, 어떤식으로 작업을 처리하는지 자세히 살펴보겠습니다.

<br><br>

## ✅ Dispatcher Servlet의 동작 과정


* * *

<br><br>

## 마치며

