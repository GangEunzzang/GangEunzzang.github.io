---
title: Graceful Shutdown 동작 과정
date: 2024-02-28 23:30:00 +09:00
categories: [spring]
tags:
  [
    java,
    graceful,
    graceful-shutdown,
    Spring,
    스프링
  ]
---

* * *

Spring Boot Application에서 Controller가 요청을 처리하고 응답이 되지 않았는데 종료요청이 오면 어떻게 될까요?  
Client는 응답을 받지 못하고 timeout이 발생합니다.

해당 상황을 예방하기 위하여 `Graceful Shutdown`에 대해 알아보겠습니다.

<br><br>

##  ✅ Graceful Shutdown이란?

* Graceful Shutdown이 진행되면 Client의 요청을 거부한다.
* 처리 중인 요청이 있다면 완료 후 종료 시킨다.

### 📌 테스트

아래와 같이, Controller를 하나 만들었습니다.

```java
@GetMapping("/graceful")
    public ResponseEntity<Void> gracefulTest() {
        log.info("----- client 요청 ----- ");
        threadSleep();
        log.info("----- 요청 처리 완료 ----");

        return ResponseEntity.noContent().build();
    }

    public void threadSleep() {
        try {
            Thread.sleep(20000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
```
테스트 진행은 다음과 같습니다.

1. Server로 요청 전송
2. 응답 전 서버를 종료시킨다.

#### 결과
```text
[2024-03-12 22:07:17:8832] [http-nio-8081-exec-1] INFO  [com.devblog.controller.BoardController.gracefulTest:86] - ----- client 요청 ----- 
[2024-03-12 22:07:21:12410] [SpringApplicationShutdownHook] INFO  [org.springframework.boot.web.embedded.tomcat.GracefulShutdown.shutDownGracefully:53] - Commencing graceful shutdown. Waiting for active requests to complete
[2024-03-12 22:07:37:28838] [http-nio-8081-exec-1] INFO  [com.devblog.controller.BoardController.gracefulTest:88] - ----- 요청 처리 완료 ----
[2024-03-12 22:07:37:28871] [http-nio-8081-exec-1] DEBUG [org.springframework.orm.jpa.support.OpenEntityManagerInViewInterceptor.afterCompletion:111] - Closing JPA EntityManager in OpenEntityManagerInViewInterceptor
[2024-03-12 22:07:37:28912] [tomcat-shutdown] INFO  [org.springframework.boot.web.embedded.tomcat.GracefulShutdown.doShutdown:78] - Graceful shutdown complete
[2024-03-12 22:07:37:28942] [SpringApplicationShutdownHook] INFO  [org.springframework.orm.jpa.AbstractEntityManagerFactoryBean.destroy:651] - Closing JPA EntityManagerFactory for persistence unit 'default'
[2024-03-12 22:07:37:28949] [SpringApplicationShutdownHook] INFO  [com.zaxxer.hikari.HikariDataSource.close:350] - HikariPool-1 - Shutdown initiated...
[2024-03-12 22:07:38:29069] [SpringApplicationShutdownHook] INFO  [com.zaxxer.hikari.HikariDataSource.close:352] - HikariPool-1 - Shutdown completed.
```

위와 같이 서버 종료 요청후 graceful shutdown이 진행되며 응답 후 종료된것을 확인할 수 있습니다.

<br><br>

## ✅ 적용 방법

### 📌 application.yml
```yaml
server:
  shutdown: graceful 
  # default value: immediate(즉시 종료)

# 타임아웃 지정
spring:
  lifecycle:
    timeout-per-shutdown-phase: 60s
    #default value : 30s
```

### 📌 서버 종료 방식 지정(linux kill)
위 설정을 해도 linux에서 `kill -9` 명령을 날린다면 graceful shutdown이 동작하지 않습니다.

`kill -15` 명령어로 변경해야 합니다.
* -`9(SIGKILL)` : 프로세스 즉시 종료
* `-15(SIGTERM` : 프로세스 정상 종료 요청 (Application Process에게 종료 시그널 )

kill -15 명령어가 실행된다면 Spring의 `SpringApplicationShutdownHook` 이라는 객체를 통해 Spring을 종료시키기 시작합니다.

#### kill 명령어 리스트
```text
$ kill -l
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
```

<br><br>

## ✅ 동작 원리

### 📌 Graceful이 설정된 Tomcat

```java
public class TomcatWebServer implements WebServer {

    private final GracefulShutdown gracefulShutdown;

    public TomcatWebServer(Tomcat tomcat, boolean autoStart, Shutdown shutdown) {
        Assert.notNull(tomcat, "Tomcat Server must not be null");
        this.tomcat = tomcat;
        this.autoStart = autoStart;
        this.gracefulShutdown = (shutdown == Shutdown.GRACEFUL) ? new GracefulShutdown(tomcat) : null;
        initialize();
    }
}
```
서버 실행시 shutdown 설정값이 graceful이라면 `GracefulShutdown` 객체를 넣어줍니다.

### 📌 종료 과정

종료 요청시 아래의 `shutDownGracefully` 메서드가 호출됩니다. 
```java
public void shutDownGracefully(GracefulShutdownCallback callback) {
        if (this.gracefulShutdown == null) {
            callback.shutdownComplete(GracefulShutdownResult.IMMEDIATE);
        } else {
            this.gracefulShutdown.shutDownGracefully(callback);
        }
    }
```

그리고 `GracefulShutdown`의 `shutDownGracefully` 메소드를 호출합니다.
```java
void shutDownGracefully(GracefulShutdownCallback callback) {
        logger.info("Commencing graceful shutdown. Waiting for active requests to complete");
        (new Thread(() -> {
            this.doShutdown(callback);
        }, "tomcat-shutdown")).start();
    }

    private void doShutdown(GracefulShutdownCallback callback) {
        List<Connector> connectors = this.getConnectors();
        connectors.forEach(this::close);

        try {
            Container[] var3 = this.tomcat.getEngine().findChildren();
            int var4 = var3.length;

            for(int var5 = 0; var5 < var4; ++var5) {
                Container host = var3[var5];
                Container[] var7 = host.findChildren();
                int var8 = var7.length;

                for(int var9 = 0; var9 < var8; ++var9) {
                    Container context = var7[var9];

                    while(this.isActive(context)) {
                        if (this.aborted) {
                            logger.info("Graceful shutdown aborted with one or more requests still active");
                            callback.shutdownComplete(GracefulShutdownResult.REQUESTS_ACTIVE);
                            return;
                        }

                        Thread.sleep(50L);
                    }
                }
            }
        } catch (InterruptedException var11) {
            Thread.currentThread().interrupt();
        }

        logger.info("Graceful shutdown complete");
        callback.shutdownComplete(GracefulShutdownResult.IDLE);
    }
```

먼저, `doShutdown 메서드에서 connector들을 닫음으로써 새로운 요청들을 받지 않도록` 합니다.

`doShutdown 메서드 내부 while문에서 isActive라는 메서드를 통해 현재 처리중인 요청이 있는지 확인`하고   
존재하면 루프에서 50ms씩 기다리면서 지속적으로 완료되지 않은 요청에 대한 확인을 합니다.



