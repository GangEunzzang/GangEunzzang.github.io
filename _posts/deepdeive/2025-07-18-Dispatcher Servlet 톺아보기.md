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

`Dispatcher Servlet`은 다음과 같은 과정으로 요청을 처리합니다.

![dispatcher-servlet-flow](https://docs.spring.io/spring-framework/docs/3.2.x/spring-framework-reference/html/images/mvc.png)

### 1. 요청 수신 및 HandlerMapping 조회

클라이언트로부터 HTTP 요청이 들어오면, `Dispatcher Servlet`이 가장 먼저 요청을 받습니다.

```java
@Override
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = null;
    HandlerExecutionChain mappedHandler = null;
    
    try {
        processedRequest = checkMultipart(request);
        // HandlerMapping을 통해 요청에 맞는 핸들러 조회
        mappedHandler = getHandler(processedRequest);
        if (mappedHandler == null) {
            noHandlerFound(processedRequest, response);
            return;
        }
        // ...
    }
}
```

`getHandler()` 메서드에서는 등록된 `HandlerMapping` 구현체들을 순회하며 요청을 처리할 수 있는 핸들러를 찾습니다.

### 2. HandlerAdapter 조회

찾은 핸들러를 실행할 수 있는 `HandlerAdapter`를 조회합니다.

```java
HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
```

Spring MVC에서는 다양한 형태의 핸들러를 지원하기 위해 어댑터 패턴을 사용합니다:
- `RequestMappingHandlerAdapter`: `@Controller` + `@RequestMapping`
- `HttpRequestHandlerAdapter`: `HttpRequestHandler` 인터페이스
- `SimpleControllerHandlerAdapter`: `Controller` 인터페이스

### 3. HandlerInterceptor 전처리 실행

핸들러 실행 전에 `HandlerInterceptor`의 `preHandle()` 메서드들이 실행됩니다.

```java
if (!mappedHandler.applyPreHandle(processedRequest, response)) {
    return;
}
```

인터셉터에서 `false`를 반환하면 요청 처리가 중단됩니다.

### 4. 핸들러(Controller) 실행

`HandlerAdapter`를 통해 실제 컨트롤러 메서드를 실행합니다.

```java
ModelAndView mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
```

여기서 `@Controller` 클래스의 메서드가 실행되고, 비즈니스 로직이 처리됩니다.

### 5. HandlerInterceptor 후처리 실행

핸들러 실행 후 `HandlerInterceptor`의 `postHandle()` 메서드들이 실행됩니다.

### 6. ViewResolver를 통한 View 조회

컨트롤러에서 반환한 뷰 이름을 바탕으로 `ViewResolver`가 실제 `View` 객체를 찾습니다.

```java
if (mv != null && !mv.wasCleared()) {
    render(mv, request, response);
}

private void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) {
    View view = resolveViewName(mv.getViewName(), mv.getModelMap(), locale, request);
    view.render(mv.getModelMap(), request, response);
}
```

### 7. View 렌더링

찾은 `View` 객체를 통해 모델 데이터를 바탕으로 최종 응답을 생성합니다.

<br>

## ✅ Dispatcher Servlet의 핵심 컴포넌트

### HandlerMapping

요청 URL을 분석하여 어떤 핸들러가 처리할지 결정하는 인터페이스입니다.

주요 구현체:
- `RequestMappingHandlerMapping`: `@RequestMapping` 기반
- `BeanNameUrlHandlerMapping`: 빈 이름과 URL 매핑
- `SimpleUrlHandlerMapping`: 설정 기반 URL 매핑

### HandlerAdapter

다양한 종류의 핸들러를 실행할 수 있게 해주는 어댑터입니다.

```java
public interface HandlerAdapter {
    boolean supports(Object handler);
    ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler);
}
```

### ViewResolver

논리적 뷰 이름을 실제 `View` 객체로 변환합니다.

```java
public interface ViewResolver {
    View resolveViewName(String viewName, Locale locale) throws Exception;
}
```

주요 구현체:
- `InternalResourceViewResolver`: JSP 뷰 리졸버
- `ThymeleafViewResolver`: Thymeleaf 템플릿 엔진
- `ContentNegotiatingViewResolver`: 요청에 따라 적절한 뷰 선택

<br>

## ✅ Dispatcher Servlet의 초기화 과정

`Dispatcher Servlet`은 서블릿 컨테이너에 의해 초기화될 때 다음과 같은 과정을 거칩니다:

### 1. WebApplicationContext 생성

```java
@Override
protected WebApplicationContext initWebApplicationContext() {
    WebApplicationContext rootContext = 
        WebApplicationContextUtils.getWebApplicationContext(getServletContext());
    
    WebApplicationContext wac = null;
    
    if (this.webApplicationContext != null) {
        // 이미 생성된 컨텍스트가 있다면 사용
        wac = this.webApplicationContext;
    } else {
        // 새로운 컨텍스트 생성
        wac = createWebApplicationContext(rootContext);
    }
    
    return wac;
}
```

### 2. 전략 인터페이스 초기화

```java
@Override
protected void onRefresh(ApplicationContext context) {
    initStrategies(context);
}

protected void initStrategies(ApplicationContext context) {
    initMultipartResolver(context);      // 파일 업로드 처리
    initLocaleResolver(context);         // 로케일 처리
    initThemeResolver(context);          // 테마 처리
    initHandlerMappings(context);        // 핸들러 매핑
    initHandlerAdapters(context);        // 핸들러 어댑터
    initHandlerExceptionResolvers(context); // 예외 처리
    initRequestToViewNameTranslator(context); // 뷰 이름 변환
    initViewResolvers(context);          // 뷰 리졸버
    initFlashMapManager(context);        // 플래시 맵 관리
}
```

<br>

## ✅ Spring Boot에서의 Dispatcher Servlet

Spring Boot에서는 `DispatcherServletAutoConfiguration`을 통해 자동 설정됩니다:

```java
@AutoConfiguration
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass(DispatcherServlet.class)
public class DispatcherServletAutoConfiguration {

    @Configuration(proxyBeanMethods = false)
    @EnableConfigurationProperties(WebMvcProperties.class)
    protected static class DispatcherServletConfiguration {

        @Bean(name = DEFAULT_DISPATCHER_SERVLET_BEAN_NAME)
        public DispatcherServlet dispatcherServlet(WebMvcProperties webMvcProperties) {
            DispatcherServlet dispatcherServlet = new DispatcherServlet();
            // 설정 적용
            return dispatcherServlet;
        }
    }
}
```

주요 설정:
- 기본 URL 패턴: `/*`
- 기본 이름: `dispatcherServlet`
- 로드 온 스타트업: `1`

<br>

## ✅ 실무에서의 활용 팁

### 1. HandlerInterceptor 활용

공통 관심사를 처리할 때 유용

```java
@Component
public class LoggingInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        log.info("Request URL: {}", request.getRequestURL());
        return true;
    }
}
```

### 2. 커스텀 HandlerMapping

특별한 요구사항이 있을 때 직접 구현 가능합니다:

```java
@Component
public class CustomHandlerMapping extends AbstractHandlerMapping {
    @Override
    protected Object getHandlerInternal(HttpServletRequest request) {
        // 커스텀 매핑 로직
        return null;
    }
}
```

### 3. 성능 모니터링

`HandlerInterceptor`를 통해 요청 처리 시간을 측정할 수 있습니다:

```java
@Component
public class PerformanceInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        request.setAttribute("startTime", System.currentTimeMillis());
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        long startTime = (Long) request.getAttribute("startTime");
        long processingTime = System.currentTimeMillis() - startTime;
        log.info("Processing time: {}ms", processingTime);
    }
}
```

* * *

<br><br>

## 마치며

`Dispatcher Servlet`은 Spring MVC의 핵심으로, Front Controller 패턴을 통해 모든 요청을 중앙 집중식으로 처리합니다.

초기 서블릿의 문제점들을 해결하고, 관심사의 분리를 통해 유지보수성과 확장성을 크게 향상시켰습니다.

실무에서는 `HandlerInterceptor`를 활용한 공통 처리, 커스텀 컴포넌트 구현 등을 통해 더욱 유연한 웹 애플리케이션을 개발할 수 있습니다.

Spring Boot의 자동 설정 덕분에 복잡한 설정 없이도 강력한 웹 애플리케이션을 빠르게 구축할 수 있게 되었습니다.
