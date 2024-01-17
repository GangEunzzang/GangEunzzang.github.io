---
title: 'chapter7 :: 병럴 데이터 처리와 성능 '
layout: post
categories: java
tags: Modern-Java-In-Action
comments: true
---

* * *
### Java7 이전
 - 데이터 컬렉션을 병럴로 처리하기가 어려웠음  
   1) 데이터를 서브파트 분할  
   2) 분할된 서브파트를 각각의 스레드로 할당  
   3) 의도치 않은 race condition이 발생하지 않도록 적절한 동기화 작업  
   4) 부분 결과를 합침 

### Java7 이후
- fork/join 프레임워크 제공
  - 더 쉽게 병렬화를 수행하며 에러 최소화 가능

### 7.1 병렬 스트림
* * * 
- 컬렉션에 paralleStream 호출시 병렬 스트림이 생성
- 병렬 스트림이란 각각의 스레드에서 처리할 수 있도록 스트림 요소를 여러 청크로 분할한 스트림

#### 병렬 스트림 활용 예시
- ex) n을 인수로 받아 1 ~ n까지의 모든 숫자의 합계를 반환하는 메서드 구현  

```java
public long sequentialSum(long n) {
  return Stream.iterate(1L, i->i+1)
    .limit(n)
    .reduce(0L, Long::sum);
}


public long iterativeSum(long n) {
  long result = 0;
  for(long i = 1L; i <= n; i++) {
    result += i;
  }
  return result;
}
```

위의 경우 n이 커진다면 병렬로 처리하는 것이 좋지만, 병렬작업은 고려할 부분이 많다.  
  >* 결과 변수는 어떻게 동기화 할것인지?     
  >* 몇 개의 스레드를 사용해야 되는지?    
  >*  숫자는 어떻게 생성해야 되는지?    
  >* 생성된 숫자는 누가 더할 것인지?

해당 문제는 병렬 스트림을 이용하면 모두 쉽게 해결할 수 있다.

* * *
### 7.2 순차 스트림을 병렬 스트림으로 변환하기
순차 스트림에 `parallel()` 메서드를 호출하면 기존의 함수형 리듀싱 연산(숫자 합계 계산)이 병렬로 처리된다.

```java
public long parallelSum(long n) {
  return Stream.iterate(1L, i->i+1) 
    .limit(n)
    .parallel()
    .reduce(0L, Long::sum);
}
```
병렬 스트림을 이용하면 스트림이 여러 청크로 나눠져 연산을 수행한다.


