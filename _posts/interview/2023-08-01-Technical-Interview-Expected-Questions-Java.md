---
title: '[기술 면접] Java 정리'
layout: post
categories: 기술면접
tags: 기술면접
comments: true
---

<details>
<summary> <b> Java 특징 </b> </summary>

<div markdown="1">

- 객체지향 프로그래밍 언어
- 기본 자료형을 제외한 모든 요소가 객체로 표현
- JVM에서 동작하기 때문에 운영체제에 독립적
- GC를 통한 자동 메모리 관리
- 다중 상속이나 타입에 엄격하며, 제약이 많음

</div>
</details>

          
<details>
<summary> <b> 객체지향 프로그래밍(OOP)이란? </b> </summary>
<div markdown="1">

Object를 기준으로 코드를 나누어 구현하는 프로그래밍(Java는 구분 단위가 class)
- 장점: 재사용성이 좋고, 협업하기가 좋다 (캡슐화, 추상화 때문에 쓰기 좋음)

- **캡슐화**: 비슷한 역할을 하는 속성과 메소드들을 하나의 클래스로 모은 것
- **추상화**: 어떤 실체로부터 공통적인 부분이나 관심있는 특성들만 한곳에 모은 것  
   ex) 어떤 하위 클래스들에 존재하는 공통적인 메서드를 인터페이스로 정의하는 것
- **상속**: 클래스를 재사용하는 것, 상속이 있기 때문에 코드를 재활용 할 수 있고 그렇기 때문에 생산성이 높고 유지보수하기 좋음
- **다형성**: 하나의 변수나 메서드가 여러가지 형태를 가질 수 있는 성질. 코드의 유연성을 높이고, 가독성 향상

</div>
</details>

<details>
<summary> <b> SOLID 원칙  </b> </summary>
<div markdown="1">

- 단일 책임 원칙(Single Responsibility Principle)
  - 하나의 클래스는 하나의 책임만 가져야 한다.
  - 클래스가 변경되는 이유는 하나여야만 한다.

- 개방/폐쇄 원칙(Open/Closed Principle)
  - 기존의 코드를 변경하지 않으며 새로운 기능을 추가할 수 있도록 설계해야 한다.
  - 확장에는 열려있어야 하고, 변경에는 닫혀 있어야 한다.

- 리스코프 치환 원칙(Liskov Substitution Principle)
  - 자식 클래스는 부모 클래스의 기능을 무시하지 않으며 확장할 수 있어야 한다.
  - 자식 클래스가 부모클래스로 대체되도 기능이 보장되어야 한다.

- 인터페이스 분리 원칙(Interface Segregation Principle)
  - 클라이언트는 자신이 사용하는 메서드에만 의존해야 한다.
  - 인터페이스를 세분화하여 필요한 메서드만 사용해야 한다.
  - 한 클래스는 자신이 사용하지 않는 인터페이스는 구현하지 않아야 한다.

- 의존 역전 원칙(Dependency Inversion Principle)
  - 고수준 모듈은 저수준 모듈의 구현에 의존해서는 안된다.
  - 추상화된 인터페이스나 클래스에 의존해야 한다.
  - 저수준 모듈이 변경되어도 고수준 모듈은 변경이 필요없는 형태가 이상적이다.
  - 시스템의 결합도를 낮추고 유연성을 높일 수 있다.  

</div>
</details>

<details>
<summary> <b> 오버로딩과 오버라이딩의 차이  </b> </summary>
<div markdown="1">

- 오버라이딩 :
    - 부모 클래스에게 상속받은 메서드를 자식클래스에서 재정의 하는 것
- 오버로딩 :
    - 한 클래스내에 이름이 같은 여러개의 메서드를 재정의 하는 것
    - 매개변수의 타입이나, 갯수가 달라야한다.

</div>
</details>

<details>
<summary> <b> JVM의 구조 </b> </summary>
<div markdown="1">

![jvm.png](/assets\img/jvm.png)

- JVM은 크게 **Class Loader**, **Execution Engine**, **Garbage Collector**, **Runtime Data Area** 로 구성

> **Class Loader** 
  - .class 파일들을 Runtime Data Area에 적재하는 역할을 한다.

> **Execution Engine**
  - Runtime Data Area에 적재된 .class파일들을 기계어로 변경해 **명령어 단위로 실행** 하는 역할
  - 명령어 실행 방법은 Interpreter, JIT(Just-In-Time)컴파일러 두가지 방법이 있다.

> **Runtime Data Areas**
  - 자바 프로그램을 실행할 때 사용되는 데이터들을 저장하는 영역
  - Stack Area: 지역 변수, 파라미터 등이 생성되는 영역. 실제 객체는 Heap에 할당되고 해당 레퍼런스만 Stack에 저장
  - Heap Area: 동적으로 생성된 오브젝트와 배열이 저장되는 곳으로 GC의 대상 영역
  - Method Area: 클래스 멤버 변수, 메소드 정보, Type 정보, Constant Pool, static, final 변수 등이 생성됩니다. 상수 풀(Constant Pool)은 모든 Symbolic Reference를 포함


</div>
</details>

<details>
<summary> <b> Java 실행방식  </b> </summary>
<div markdown="1">

- 자바 컴파일러(javac)가 자바 소스코드(.java)를 읽어 자바 바이트코드(.class)로 변환
- Class Loader를 통해 class 파일들을 JVM으로 로딩
- 로딩된 class파일들은 Execution engine을 통해 해석
- 해석된 바이트코드는 Runtime Data Area에 배치되어 실질적인 수행이 이뤄짐

</div>
</details>

<details>
<summary> <b> GC가 무엇인지, 필요한 이유는 무엇인지, 동작 방식 설명 </b> </summary>
<div markdown="1">

- GC는 힙 영역에서 사용하지 않는 객체들을 제거하는 작업을 총칭
- 객체룰 제거하는 작업이 필요한 이유는 Java는 개발자가 메모리를 직접 해제해줄 수 없기때문
- GC를 수행할 때는 GC를 수행하는 스레드 이외의 스레드는 모두 정지(Stop-the-world)
- GC는 Minor GC, Major GC로 구분할 수 있다

> **Minor GC**
  - young 영역에서 일어난다.
  - Eden 영역이 가득 참에서 부터 시작된다. 
  - Eden 영역에서 참조가 남아있는 객체를 mark하고 survivor 영역으로 복사한다. 
  - Eden 영역을 비운다. 
  - Survivor 영역도 가득차면 같은 방식으로 다른 Survivor 영역에 복사하고 비운다.
  - 이를 반복하다 보면 계속 해서 살아남는 객체는 old 영역으로 이동하게 된다.

> **Major GC**
  - old 영역에서 일어난다.
  - minor와 반대로 삭제되어야 할 객체를 mark하고 지운다(sweep).
  - 메모리는 단편화 된 상태이므로 이를 한 군데에 모아준다.

- 이것이 중요한 이유는 GC 수행시 시스템이 멈추기 때문에 의도치 않은 장애의 원인이 될 수 있기 때문이다.
- 힙 영역을 조정하는 것을 GC 튜닝이라 하고, JVM 메모리는 절대 마음대로 조정해선 안된다.

</div>
</details>

<details>
<summary> <b> 컬렉션이란?  </b> </summary>
<div markdown="1">

![collection-structor.png](/assets\img/collection-structor.png)

- List,Set,Map등 여러 자료구조를 묶어 하나로 그룹화한 객체를 말한다.
- 컬렉션 클래스들이 데이터를 다룰 때 기본형은 사용할 수 없다.

</div>
</details>

<details>
<summary> <b> 제네릭이란?  </b> </summary>
<div markdown="1">

- 클래스 내부에서 사용할 데이터 타입을 인스턴스를 생성할 때 확정하는 것
- 컴파일 과정에서 타입체크를 해주기 때문에 타입 안정성을 높이고 형변환의 번거로움을 줄여준다.

</div>
</details>


<details>
<summary> <b> 애노테이션이란?  </b> </summary>
<div markdown="1">

- 인터페이스를 기반으로 한 문법으로 주석처럼 코드에 달아 클래스에 특별한 의미를 부여하거나 기능을 주입할 수 있습니다.
- built-in annotation은 상속받아서 메소드를 오버라이드 할 때 나타나는 @Override 애노테이션이 그 대표적인 예
- 메타 애너테이션은 애노테이션을 선언할 때 사용하는 애노테이션입니다.
  •	@Retention: 애노테이션 유지 범위를 지정합니다. (소스, 클래스, 런타임)
  •	@Inherit: 애노테이션을 하위 클래스까지 전달여부를 지정합니다. 이 애노테이션이 있으면 하위 클래스까지 상속이 가능합니다.
  •	@Target: 해당 애노테이션을 어디에 사용할 지 결정합니다. (타입, 필드, 메서드, 파라미터, 생성자, 로컬변수, 애노테이션 타입)

</div>
</details>

<details>
<summary> <b> 인터페이스 추상클래스 차이점  </b> </summary>
<div markdown="1">

- 추상클래스
  - 객체의 추상적인 상위 개념으로 공통된 개념을 표현할 때 사용
  - 단일 상속만 가능
  - 추상클래스를 상속하는 집합간에는 연관관계가 있음
- 인터페이스
  - 구현 객체가 같은 동작을 한다는 것을 보장하기 위해 사용
  - 다중 상속이 가능
  - 인터페이스를 구현하는 집합간에는 관계가 없을 수 있음

</div>
</details>

<details>
<summary> <b> 클래스는 무엇이고 객체는 무엇인가?  </b> </summary>
<div markdown="1">

- 클래스: 객체를 정의하는 틀 또는 설계도와 같은 의미
- 객체
  - 식별 가능한 개체 또는 사물.
  - 구별 가능한 식별자, 특징적인 행동, 변경 가능한 상태를 가짐
  - 인스턴스들을 통칭하는 용도로 사용

</div>
</details>

<details>
<summary> <b> 정적(static)이란 무엇인가?  </b> </summary>
<div markdown="1">

- static은 클래스 멤버라고 하며, 클래스 로더가 클래스를 로딩해서 메소드 메모리 영역에 적재할 때 클래스별로 관리됨
- static 키워드를 통해 생성된 정적멤버들은 Permanent(1.7까지) 또는 heap영역(1.8 이후)에 저장
- 저장된 메모리는 모든 객체가 공유하며 하나의 멤버를 어디서든지 참조할 수 있음
- 그러나, 1.7 버전까지는 GC의 관리 영역 밖에 존재하기 때문에 프로그램 종료시까지 메모리가 할당된 채로 존재
- 너무 남발하게 되면 시스템 성능에 악영향을 줄 수 있음
- 단 1.8 이후부터는 static Object가 heap영역으로 바뀌면서 참조를 잃은 경우 GC의 대상이 될 수 있음

</div>
</details>

<details>
<summary> <b> String, StringBuilder, StringBuffer 차이 설명    </b> </summary>
<div markdown="1">

- String은 불변 객체이며, 스레드 안전을 보장하고, 문자열 수정 작업이 자주 발생시 성능 저하 발생   
- StringBuilder, StringBuffer는 가변 타입이다. 문자열을 수정하는 작업에 효율적이다.
- StringBuilder는 Thread-safe 하지 않다.
- StringBuffer는 내부적으로 synchronized 키워드를 사용하여 Thread-safe 하다.

</div>
</details>

<details>
<summary> <b> Error와 Exception 차이  </b> </summary>
<div markdown="1">

- **Error:** 
  - 시스템 수준에서 발생하는 예외 상황
  - 개발자가 예측하기 어렵고, 처리방법을 코드에 명시 불가능
  - 시스템이 종료됨

- **Exception:**
  - 개발자가 예상할 수 있는 예외 상황
  - 처리 방법을 코드에 명시 가능

</div>
</details>

<details>
<summary> <b> Checked Exception과 Unchecked Exception 차이 </b> </summary>
<div markdown="1">

- **Checked Exception**
  - 컴파일시점에 확인가능한 예외로 예측이 가능하다
  - 반드시 예외처리를 해야한다
  - 트랜잭션 롤백이 일어나지 않는다.

- **Unchecked Exception**
  - 런타임시점에 확인가능한 예외로 예측할 수 없다.
  - 트랜잭션이 롤백된다.

</div>
</details>

<details>
<summary> <b> 강한 결합 vs 느슨한 결합  </b> </summary>
<div markdown="1">

- **강한 결합**:
  - 두 객체 간에 서로 깊게 의존하는 것
  - 즉, 한 객체가 다른 객체의 내부 상태나 구현 방법에 직접적으로 의존하는 경우
  - 코드 변경이 어렵고, 유지보수성이 떨어지고, 테스트 수행도 어렵다.

- **느슨한 결합**:
  - 두 객체 간에 의존성이 낮은 것
  - 한 객체가 다른 객체에 직접적인 정보를 가지지 않고, 인터페이스나 추상클래스 등을 통해 간접 의존하는 경우

</div>
</details>

<details>
<summary> <b> Call By Value & Call By Reference  </b> </summary>
<div markdown="1">

- **Call By Value**:
  - 메서드 호출시 인자로 전달되는 값의 복사본이 전달
  - 메서드 내부에서 인자값이 변경되어도 원본에는 영향 없음
  - 기본 데이터 타입인 경우 적용

- **Call By Reference**:
  - 메서드 호출시 인자로 전달되는 객체의 주소값이 전달
  - 메서드 내부에서 인자 객체의 값을 변경하면 원본도 변경(동일한 주소를 참조하기 때문에)

자바는 Call By Reference 개념이 없다.
객체를 넘길때 인자값을 변경하면 실제 인자값도 변경되기 때문에 햇갈릴 수 있지만
이는 직접적인 참조를 넘긴 게 아닌, 주소 값을 복사하여 넘기기 때문이다.

</div>
</details>



## 참고
- [백엔드 개발자로 입사를 준비하며 받았던 질문, 예상했던 질문, 인터넷 참고한 질문(CC BY-NC)](https://github.dev/ksundong/backend-interview-question)