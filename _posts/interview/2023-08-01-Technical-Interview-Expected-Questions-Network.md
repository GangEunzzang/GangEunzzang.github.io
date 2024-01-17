---
title: '[기술 면접] 네트워크 정리'
layout: post
categories: 기술면접
tags: 기술면접
comments: true
---

<details>
<summary> <b> https://www.google.com 을 접속했을 때 일어나는 일  </b> </summary>
<div markdown="1">

1) 브라우저가 URL에 적힌 값을 파싱해서 HTTP Request Message를 생성하고, OS에 전송 요청을 한다. 이때, Domain으로 요청을 보낼 수 없기 때문에 DNS LookUp을 수행한다.
2)

</div>
</details>

<details>
<summary> <b> GET과 POST 차이  </b> </summary>
<div markdown="1">

- GET
    - 요청 데이터가 `HTTP Request Message`의 Header 부분에 url 이 담겨서 전송된다.
    - 전송할 수 있는 데이터의 크기가 제한적이다.
    - 보안이 필요한 데이터에 대해서는 적절하지 않다.

- POST
    - 요청 데이터가 `HTTP Request Message`의 Body 부분에 데이터가 담겨서 전송된다.
    - 서버의 상태를 변경시키기 때문에 멱등성이 유지되지 않는다.

부수적인 차이점을 좀 더 살펴보면 GET 방식의 요청은 브라우저에서 `Caching` 할 수 있다.  
때문에 POST 방식으로 요쳥해야 할 것을 보내는 데이터가 작고 보안적인 문제가 없다는 이유로 GET 방식으로 요청한다면,  
기존에 캐싱 되었던 데이터가 응답될 가능성이 존재한다.

</div>
</details>

<details>
<summary> <b> TCP와 UDP의 차이  </b> </summary>
<div markdown="1">

- TCP(Transmission Control Protocol)
    - 신뢰성 있는 데이터 전송을 위한 연결 지향성 프로토콜
    - UDP에 비해 속도가 느리다
    - 파일 전송, 이메일 전송과 같은 신뢰성이 중요한 서비스에 사용된다.


- UDP(User Datagram Protocol)
    - 연걸 설정 및 확인 단계 없이 데이터 전송하는 비연결 지향성 프로토콜
    - 데이터 손실이나 순서 변경 가능성이 존재하며, 수신 확인 또는 재전송을 처리하지 않는다.
    - 실시간 스트리밍, 온라인 게임, DNS 등과 같은 서비스에 사용.

</div>
</details>


<details>
<summary> <b> HTTP 프로토콜에 대해 설명 </b> </summary>
<div markdown="1">

- Hypertext Transfer Protocol을 의미
- 컴퓨터 간의 데이터 전달을 위한 약속을 나타내며, 데이터를 요청하는 쪽은 클라이언트, 받는 쪽은 서버라고 지칭
- 기존에는 Hypertext를 의미하는 HTML 문서를 교환했지만, 이제는 다양한 미디어 리소스를 주고받는 형태로 발전

</div>
</details>


<details>
<summary> <b> HTTP 메서드 종류 </b> </summary>
<div markdown="1">

- GET(리소스 조회)
    - 보통 리소스를 조회할 때 사용하며, 서버에 전달하고 싶은 데이터는 query를 통해서 전달한다. 
    - 메시지 바디를 사용해서 데이터를 전달할 수는 있지만, 지원하지 않는 곳이 많아서 권장하지 않는다.
  

- POST(요청 데이터 처리, 데이터 등록에 사용)
    - 데이터 요청을 처리하고, 메시지 바디를 통해 서버로 데이터를 전달한다. 
    - 주로 신규 리소스를 등록하거나 프로세스 처리에 사용된다.


- PUT(리소스를 대체, 해당 리소스가 없으면 생성)
    -  소스가 있으면 대체하고 리소스가 없으면 생성한다. 쉽게 말해 데이터를 덮어쓴다.


- PATCH(리소스를 일부만 변경)
    - PUT과 마찬가지로 리소스를 수정하지만, 리소스를 일부분만 변경할 수 있다.


- DELETE(리소스 삭제)
    - 리소스 삭제할때 사용

</div>
</details>


<details>
<summary> <b> HTTP 메서드 특징 </b> </summary>
<div markdown="1">

- 안전(Safe Method)
  - 계속해서 메서드를 호출해도 리소스를 변경하지 않는다.
  - 주요 메서드 중에는 GET 메서드가 안전하다고 볼 수 있다.


- 멱등(Idempotent  Method)
  - 메서드를 계속 호출해도 결과가 동일함
  - GET,PUT,DELETE는 멱등하지만, POST, PATCH는 멱등하지 않다.


- 캐싱(Caching)
  - GET,HEAD,POST,PATCH 모두 캐시가 가능하다.
  - 실제로는 GET과 HEAD만 주로 캐싱이 사용된다.

</div>
</details>




<details>
<summary> <b> HTTP Status Code 설명  </b> </summary>
<div markdown="1">

- 1XX (information - 조건부 응답)
    - 클라이언트가 서버에 정보를 요청했지만 아직 처리중임을 의미


- 2XX (Successful - 성공)
    - 서버가 브라우저의 요청을 수신하고 성공적으로 처리했음을 의미    

         
- 3XX (Redirection - 리디렉션)
    - 요청된 페이지가 일시적으로 또는 영구적으로 이동되었음을 클라이언트에 알림
    - 원래 요청한 리소스를 더 이상 사용할 수 없다.


- 4XX (Client Error - 요청 오류)
    - 잘못된 요청으로 서버가 이해를 못해 요청을 수행할 수 없음을 의미


- 5XX (Server Error - 서버 오류)
    - 서버 오류로 인해 서버가 요청을 정상 처리 하지 못함을 의미

</div>
</details>

<details>
<summary> <b> 중요한 HTTP Status Code  </b> </summary>
<div markdown="1">

- 200 (OK)
    - 모든 것이 정상적으로 수행 되었음을 의미


- 301 (Moved Permanently)
    - URL이 영구적으로 다른 위치로 이동했음을 의미
    - 해당 요청 및 이후의 모든 요청은 다른 URL로 리디렉션 되어야 한다.


- 302 (Found / Moved Temporarily)
    - URL이 일시적으로 다른 위치로 이동했음을 의미


- 401 (Unauthorized)
    - 클라이언트가 인증되지 않았거나, 유효한 인증 정보가 부족하여 요청이 거부됨
    - ex) 사용자가 로그인되지 않은 경우


- 403 (Forbidden)
    - 서버가 해당 요청을 이해했지만, 권한이 없어 요청이 거부됨
    - ex) 사용자가 권한이 없는 요청을 하는 경우


- 500 (Server Error)
    - 서버가 사용자의 리소스 요청을 처리할 수 없을 때 나타난다. 
    - 서버 구성 에러로 인해 발생하는 일반적인 에러입니다


- 503 (Service Unavailable)
    - 버를 현재 사용할 수 없으며 그 결과 클라이언트의 요청을 처리할 수 없음을 나타낸다.

</div>
</details>

## 참고
- [백엔드 개발자로 입사를 준비하며 받았던 질문, 예상했던 질문, 인터넷 참고한 질문(CC BY-NC)](https://github.dev/ksundong/backend-interview-question)
- [기술면접 정리](https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/Network)
