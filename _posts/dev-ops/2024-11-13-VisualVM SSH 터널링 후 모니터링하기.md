---
title: VisualVM SSH 터널링 후 모니터링하기i
date: 2024-11-13 10:30:00 +09:00
categories: [dev-ops, infra]
tags:
  [
    VisualVM,
    SSH,
    터널링,
    모니터링,
  ]
---

* * *

VisualVM을 사용하여 SSH 터널링을 설정하고, 원격 서버의 JVM을 모니터링하는 방법을 알아보겠습니다.

이 포스팅은 다음과 같은 경우에 유용할 것으로 예상됩니다.
1. 원격 서버의 포트를 외부로 열지 못할 경우
2. 원격 서버의 SSH에 접속할 수 있는 경우


## ✅ jar 옵션 추가
    -Dcom.sun.management.jmxremote \
    -Dcom.sun.management.jmxremote.port=9010 \
    -Dcom.sun.management.jmxremote.rmi.port=9010 \
    -Dcom.sun.management.jmxremote.authenticate=false \
    -Dcom.sun.management.jmxremote.ssl=false \
    -Dcom.sun.management.jmxremote.local.only=false \
    -Djava.rmi.server.hostname=127.0.0.1 \

* 포트는 자유롭게 설정해주셔도 됩니다.

### 📌 옵션 설명
- `com.sun.management.jmxremote` : JMX 리모트 모니터링을 활성화합니다.
- `com.sun.management.jmxremote.port` : JMX 리모트 모니터링을 위한 포트를 지정합니다.
- `com.sun.management.jmxremote.rmi.port` : JMX 리모트 모니터링을 위한 RMI 포트를 지정합니다.
- `com.sun.management.jmxremote.authenticate` : 인증을 사용할지 여부를 지정합니다. (false: 사용안함)
- `com.sun.management.jmxremote.ssl` : SSL을 사용할지 여부를 지정합니다. (false: 사용안함)
- `com.sun.management.jmxremote.local.only` : 로컬 호스트에서만 접속을 허용할지 여부를 지정합니다. (false: 모든 호스트에서 접속 가능)
- `java.rmi.server.hostname` : RMI 서버의 호스트명을 지정합니다.
- `com.sun.management.jmxremote.password.file` : 비밀번호 파일을 지정합니다. (인증을 사용할 경우 필요)
- `com.sun.management.jmxremote.access.file` : 접근 제어 파일을 지정합니다. (인증을 사용할 경우 필요)
- `com.sun.management.jmxremote.ssl.need.client.auth` : 클라이언트 인증을 요구할지 여부를 지정합니다. (SSL을 사용할 경우 필요)

<br><br>

## ✅ VisualVM SSH 터널링 설정
1. ssh 터널링 설정
```shell
ssh -L 9010:localhost:9010 [ID@HOST]
```
* 위와 같이 설정하면 로컬 9010 포트로 접속하면 원격 서버의 9010 포트로 연결됩니다.
* 윈도우는 PowerShell/ Mac은 Terminal을 사용하여 명령어 실행해주시면 됩니다.

<br>

2. VisualVM 실행  
`File` -> `Add JMX Connection` -> `localhost:9010`


<br><br>

## ✅ 주의사항
- VisualVM을 사용할 때는 JDK가 설치되어 있어야 합니다.
- ssh 연결한 세션을 종료하면 실시간 모니터링이 종료됩니다.







