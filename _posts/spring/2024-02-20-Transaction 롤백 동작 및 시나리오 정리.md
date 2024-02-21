---
title: Transaction 롤백 동작 및 시나리오 정리
date: 2024-02-19 23:30:00 +09:00
categories: [spring]
tags:
  [
    java,
    transaction,
    트랜잭션,
    Spring,
    스프링
  ]
---

* * *

Unchecked Exception 발생 시에는 롤백 되지만,  
Checked Exception 발생 시에는 롤백되지 않는건 유명한 정보이다.  
하지만 예외를 catch 했을때 어떤식으로 처리가 되는지는 햇갈려서 케이스별로 정리해 보려고 한다. 

![transaction_progation.PNG](..%2F..%2Fassets%2Fimg%2FSpring%2Ftransaction_progation.PNG)

스프링에서 지원해주는 전파속성은 위와 같이 많이 존재하지만  
실제로 자주 사용하는 REQUIRED, REQUIRES_NEW 만 정리해보겠다.

<br><br>


### 트랜잭션 X   + UnCheckedException  

```java
public void test() {  
    boardRepository.save(Board.create());
    throw new RuntimeException();  
}

결과 : 롤백 안됨
```
--- 

<br><br>

### 트랜잭션 O   + UnCheckedException  

```java
@Transactional
public void test() {  
    boardRepository.save(Board.create());
    throw new RuntimeException();  
}

결과 : 정상 롤백
```
--- 

<br><br>

### 트랜잭션 X   + CheckedException  

```java
public void test() throws Exception {  
    boardRepository.save(Board.create());
    throw new IOException();  
}

결과 : 롤백 안됨
```
--- 
<br><br>


### 트랜잭션 O   + CheckedException  

```java
@Transactional
public void test() throws Exception {  
    boardRepository.save(Board.create());
    throw new IOException();  
}

결과 : 롤백 안됨
```

여기까진 알고있는 결과와 일치 했다.      
이제 예외를 잡은경우를 테스트 해보자.
--- 



<br><br>

### 트랜잭션 O   + UnCheckedException  + catch

```java
@Transactional  
public void test() throws Exception {  
    boardRepository.save(Board.create());
  
    try {  
        throw new RuntimeException();  
    } catch (Exception ignored) {  
    }
}

결과 : 롤백 안됨
```

위와 같이 예외를 잡아주면 롤백이 진행되지 않았다.  
사실 이것도 예상가능한 결과이다.  
예외가 전파 되지않고 catch 블록에서 처리됐기 때문이다.   

외부에서 호출할때도 마찬가지이다.

```java
try {
	service.test()
} catch(Exception ignored) {
}

이런식으로 잡아줘도 롤백이 진행되지 않는다.
롤백을 원한다면 catch문 내부에서 명시적으로 예외를 날려줘야 한다.
```
- - - 
<br><br>
### 부모 트랜잭션 X   +  자식 트랜잭션 O   +  UnCheckedException


```java
public void test() throws Exception {  
    test2();  
}  
  
@Transactional  
public void test2() throws Exception {  
    boardRepository.save(Board.create());
  
    throw new RuntimeException();  
}

결과 : 롤백 안됨

위 예제는 유명한 자기호출 이슈이다.

이렇게 적용시 Intellij 에서도 주의 문구를 아래와 같이 띄워준다.

@Transactional self-invocation (in effect, a method within the target object calling another method of the target object) does not lead to an actual transaction at runtime 

```

자세한 설명은 아래 공식문서 참고  
[spring docs 참고](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html#aop-understanding-aop-proxies)
- - - 

<br><br>

### 부모 트랜잭션 O  +  자식 트랜잭션 X   +  UnCheckedException


```java
@Transactional  
public void test() throws Exception {  
    test2();  
}  
  
public void test2() throws Exception {  
    boardRepository.save(Board.create());
  
    throw new RuntimeException();  
}

결과 : 정상 롤백
```
- - - 
<br><br>




## 부모: REQUIRED  +  자식: REQUIRES_NEW

위와같이 전파속성을 설정하고 여러 케이스를 테스트 해보겠다.


### 자식: UncheckedException

```java

// 1번째 케이스
@Transactional  
public void test() throws Exception {  
    test2();  
}  
  
@Transactional(propagation = Propagation.REQUIRES_NEW)  
public void test2() throws Exception {  
    boardRepository.save(Board.create());
    throw new RuntimeException();  
}

결과 : 정상 롤백


-------------------------------------------------------------------
// 2번째 케이스
@Transactional  
public void test() throws Exception {   
    boardRepository.save(Board.create());  
    test2();  
}  
  
@Transactional(propagation = Propagation.REQUIRES_NEW)  
public void test2() throws Exception {    
    boardRepository.save(Board.create());
    throw new RuntimeException();  
}


첫번째 케이스와 다르게 부모 트랜잭션에서도 데이터를 저장해주고 있다.
잠시 생각해보면, parent와 child가 서로 다른 트랜잭션이기 때문에
parent의 데이터는 저장되고 child는 롤백 될 거라고 예상할 수 있다.

틀렸다..

코드상으로는 자식의 예외가 부모까지 전파되기 때문에 둘 다 롤백이 된다.

결과 : 둘 다 롤백


-------------------------------------------------------------------
// 3번째 케이스

@Transactional  
public void test()  {  
    boardRepository.save(Board.builder().title("te").content("te").build()); 
    System.out.println("em.getDelegate() = " + em.getDelegate());  
  
    try {  
        test2();  
    } catch (Exception ignored) {  
    }  
}  
  
@Transactional(propagation = Propagation.REQUIRES_NEW)  
public void test2() {  
    System.out.println("em.getDelegate() = " + em.getDelegate());  
    boardRepository.save(Board.create());  
    throw new RuntimeException();  
}


그럼 이번엔 자식에서 발생한 예외를 catch해보자

여태까지 배웠던 지식대로면 해당 코드는
child는 롤백되지만 parent는 정상 commit 되어야 한다.

근데 위 코드를 실행해보면 둘 다 커밋이 되는걸 알수가있다.

이유가 왜일까?..

비밀은 프록시에 있다.

eneityManager로 getDelegate 값을 찍어보면 두 메서드의 세션값이 똑같은걸 확인할 수 있다.

세션값이 같다는건 트랜잭션이 같다는걸 의미한다.
같은 Bean 내에서는 전파속성이 제대로 동작하지 않는다. (내부적으로 같은 트랜잭션범위를 같기때문에)

원하는 결과를 얻기 위해선 빈을 분리해 줘야한다.



-------------------------------------------------------------------
// 4번째 케이스

@Transactional  
public void test()  {  
    boardRepository.save(Board.builder().title("te").content("te").build()); 
    System.out.println("em.getDelegate() = " + em.getDelegate());  
  
    try {  
		testService2.test2();
    } catch (Exception ignored) {  
    }  
}  

 
public class TestService2 {  
    @Transactional(propagation = Propagation.REQUIRES_NEW)  
    public void test2() {  
        boardRepository.save(Board.create());  
        System.out.println("em.getDelegate() = " + em.getDelegate());  
        throw new RuntimeException();
    }  
}

이런식으로 해주면 원하는대로 parent은 commit되고
child는 rollback되는걸 확인할 수 있다.
```
- - - 
<br><br>
### 부모: UnCheckedException

```java

자식 코드는 아래와 같이 고정이다.
@Transactional(propagation = Propagation.REQUIRES_NEW)  
public void test2() {  
    boardRepository.save(Board.create());  
}

@Transactional  
public void test()  {  
    boardRepository.save(Board.builder().title("te").content("te").build());  
     testService2.test2();  
    throw new RuntimeException();
}

parent: rollback
child: commit

독립적인 트랜잭션으로 동작하는걸 확인할 수 있다.

```


CheckedException은 따로 테스트를 진행하지 않겠습니다.     
CheckedException의 기본 rollback은 옵션은 false 이며,    
예외 발생시 롤백을 원한다면 @Transactional 어노테이션의 rollbackFor 옵션을 주면 됩니다.


### 참고
- [[https://mangkyu.tistory.com/269](https://mangkyu.tistory.com/269)

