---
title: 'chapter2 :: 동적 파라미터화 코드 전달하기'
layout: post
categories: java
tags: Modern-Java-In-Action
comments: true
future: true
---

유저가 어떤 상황에서 일을 하든 소비자 요구사항은 항상 바뀐다.  
변화하는 요구사항에 맞춰 엔지니어링적인 비용을 최소화 하며,  
그뿐 아니라 새로 추가한 기능은 쉽게 구현할 수 있어야 하며 장기적인 관점에서 유저보수가 쉬워야 한다.  

**동적 파라미터**를 이용하면 이문제에 효과적으로 대응할 수 있다.

#### 동적 파라미터란?
- 어떻게 실행할 것인지 결정하지 않은 코드 블록을 의미
- 코드 블록은 나중에 프로그램에서 호출하며, 실행이 나중으로 미뤄짐

* * *

## 2.1 변화하는 요구사항에 대응하기

사과 색을 정의하는 Color enum이 존재한다고 가정하자.
```java
enum Color { RED, GREEN }
```

### 첫 번째 시도: 녹색 사과만 필터링하기
```java
public static List filterGreenApples(List<Apple> inventory) {
	List<Apple> result = new ArrayList<>(); // 사과 누적 리스트
	for (Apple apple : inventory) {
		if (GREEN.equals(apple.getColor()) { 
			result.add(apple);
		}
	} 
	return result; 
}
```

평생 녹색 사과만을 필터링하길 원한다면 좋겠지만,  
갑자기 농부가 변심하여 녹색 말고 빨간사과도 필터링 하고 싶어졌다.  
그럼 해당 코드를 고치면 되겠지만,  
농부가 좀 더 다양한 색으로 필터링을 원한다면 변화에 적절한 대응을 할 수 없다.   

> - 거의 비슷한 코드가 반복 존재한다면 그 코드를 추상화한다.

### 두 번째 시도: 색을 파라미터화
```java
 public static List filterApplesByColor(List<Apple> inventory, Color color) {
 	List<Apple> result = new ArrayList<>();
 	for (Apple apple: inventory) { 
 		if (apple.getColor().equals(color)) {
 			result.add(apple);
 		}
 	}
 	return result; 
 }
```

이제 농부가 어떤 색을 원하든 해당 메서드만 호출하면, 색으로 필터링 할 수 있다.  
하지만 농부가 색 이외에도 무게로 사과를 필터링하고 싶다고하면 어떻게 해야 할까?  

`색을 파라미터화` 한 것처럼 `무게를 파라미터화` 하면 된다.

```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
  List<Apple> result = new ArrayList<>();
  for(Apple apple : inventory) {
    if(apple.getWeight() > weight) {
      result.add(apple);
    }
  }
  return result;
}
```
위 코드도 좋은 해결책이지만, 각 사과에 필터링 조건을 적용하는 부분의 코드가 색 필터링 코드와 대부분 중복된다.  

이는 소프트웨어 공학의 **DRY(Don't Repeat yourself)** 같은 것을 반복하지 말 것 원칙을 어기는 것이다.  

### 세 번째 시도 : 가능한 모든 속성으로 필터링
```java
public static List<Apple> filterApples(List<Apple> inventory, Color color, int weight, boolean flag) {
  List<Apple> result = new ArrayList<>();
  for(Apple apple : inventory) {
    if((flag && apple.getColor().equals(color)) || (!flag && apple.getWeight() > weight)) {
      result.add(apple);
    }
  }
  return result;
}
```
위 메서드를 아래처럼 사용할 수 있다.
```java
List<Apple> greenApples = filterApples(inventory, GREEN, 0, true);
List<Apple> heavyApples = filterApples(inventory, null, 150, false);
```

위 코드는 형편없는 코드이다. true와 false가 뭘 의미하는 걸까?  
게다가 앞으로 요구사항이 바뀌었을때 유연하게 대응할 수도 없다.  
예를 들어 사과의 크기,모양,출하지 등으로 필터링 하고 싶어진 경우는 어떻게 할 것인가?..

**실전에선 절대로 사용해선 안되는 코드** 이다.

* * *
## 2.2 동적 파라미터화
사과의 어떤 속성에 기초해서 boolean값을 반환(예를 들어 사과가 녹색인가? 150g 이상인가?) 하는 방법이 있다.  
참 또는 거짓을 반환하는 함수를 `predicate`라고한다. `선택 조건을 결정하는 인터페이스`를 정의하자

```java
public interface ApplePredicate {
  boolean test (Apple apple);
}
```

다음 예제처럼 다양한 선택 조건을 대표하는 여러 버전의 ApplePredicate를 정의할 수 있다.

```java
public class AppleHeavyWeightPredicate implements ApplePredicate {
  public boolean test(Apple apple) {
    return apple.getWeight() > 150;
  }
}

public class AppleGreenColorPredicate implements ApplePredicate {
  public boolean test(Apple apple) {
    return GREEN.equals(apple.getColor());
  }
}
```
위 조건에 따라 filter 메서드가 다르게 동작할 것이라고 예상할 수 있다.  
이를 **전략 디자인 패턴(strategy design pattern)** 이라고 한다.

### 네 번째 시도 : 추상적 조건으로 필터링

```java
public static List<Apple> filterApples(List<Apple> inventory, ApplePredicate p) {
  List<Apple> result = new ArrayList<>();
  for(Apple apple : inventory) {
    if(p.test(apple)) {
      result.add(apple);
    }
  }
  return result;
}
```

첫 번째 코드에 비해 더 유연한 코드를 얻었으며 동시에 가독성도 좋아졌을 뿐 아니라 사용하기도 쉬워 졌다.  
예를 들어 농부가 150그램이 넘는 빨간 사과를 검색해달라고 부탁하면 우리는 ApplePredicate를 적절하게 구현하는 클래스만 만들면 된다.

```java
public class AppleRedAndHeavyPredicate implements ApplePredicate {
  public boolean test(Apple apple) {
    return RED.equals(apple.getColor()) && apple.getWeight() > 150;
  }
}
```

우리가 전달한 ApplePredicate 객체에 의해 filterApples 메서드의 동작이 결정된다. 즉, 우리는 filterApples 메서드의 동작을 파라미터화 한 것이다.  
> 즉, 우리는 전략 디자인 패턴(Strategy Design Pattern)과 동작 파라미터화를 통해서 필터 메서드에 전략(Strategy)을 전달 함으로써 더 유연한 코드를 만들었다.

* * *
## 2.3 복잡한 과정 간소화
위에 예시처럼 메서드로 새로운 동작을 전달하려면 Predicate 인터페이스를 선언하고, 이를 구현하는 여러 클래스를 정의하고 인스턴스화 해야한다.  
이는 상당히 번거로운 작업이며 시간 낭비이다.  

그럼 이 과정을 간소화 시켜보자.

### 2.3.1 익명 클래스
- 자바의 지역 클래스와 비슷한 개념이다.
- 말 그대로 이름이 없는 클래스이다.
- 익명 클래스를 이용하면 클래스 선언과 인스턴스화를 동시에 할 수 있다.
- 즉석에서 필요한 구현을 만들어서 사용할 수 있다.

### 2.3.2 다섯 번째 시도 : 익명 클래스 사용
익명 클래스를 이용하여 ApplePredicate를 구현하는 객체를 만드는 방법으로 필터링 예제를 다시 구현하는 코드이다.

```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() { 
  public boolean test(Apple apple) {
    return RED.equals(apple.getColor());
  }
});
```

위 예제처럼 인스턴스를 생성하지 않아도 인수값을 전달할 수 있다.  
그러나, 여전히 많은 공간을 차지하고, 많은 개발자들에게 익명 클래스는 익숙하지 않다는 단점이 있다.

### 2.3.3 여섯 번째 시도 : 람다 표현식 사용
자바 8의 람다 표현식을 이용해서 위 예제 코드를 다음처럼 간단하게 재구현할 수 있다.

```java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

이전 코드보다 훨씬 간단하고 읽기 쉽지 않은가?  
코드가 더욱 간결해지면서 문제를 더 잘 설명하는 코드가 되었다. 이렇게 복잡성 문제를 해결할 수 있다.   
![img.png](/assets\img/modern-java-chpater2-lamda.png)

### 2.3.4 일곱 번째 시도 : 리스트 형식으로 추상화

```java
public interface Predicate<T> {
  boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p) { # 형식 파라미터 T 등장
  List<T> result = new ArrayList<>();
  for(T e : list) {
    if(p.test(e)) {
      result.add(e);
    }
  }
  return result;
}
```

이제 바나나, 오렌지, 정수, 문자열 등의 리스트에 필터 메서드를 사용할 수 있다.

```java
List<Apple> redApples = filter(inventory, (Apple apple) -> RED.equals(apple.getColor()));
List<Integer> evenNumbers = filter(numbers, (Integer i) -> i % 2 == 0);
```

이렇게 해서 유연성과 간결함 두 마리의 토끼를 모두 잡을 수 있다.

* * *
## 2.4 실전 예제
지금까지 동작 파라미터화가 변화하는 요구사항에 쉽게 적응하는 유용한 패턴임을 확인했다.
그럼 실전 예제를 통하여 적용해보자

### 2.4.1 Comparator로 정렬하기
자바 8의 List에는 sort 메서드가 포함되어 있다. 다음과 같은 인터페이스를 갖는 java.util.Comparator 객체를 이용해서 sort의 동작을 파라미터화 할 수 있다.

```java
//java.util.Comparator
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```

Comparator를 구현해서 sort 메서드의 동작을 다양화 할 수 있다.  
예를들어 무게가 적은 순서로 목록에서 사과를 정렬하는 것도 가능하다.

```java
inventory.sort(new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) { 
        return a1.getWeight().compareTo(a2.getWeight());
    }
});
```

요구사항이 바뀌면 새로운 요구사항에 맞는 Comparator를 만들어 sort메서드에 전달할 수 있다.  
위 코드를 람다 표현식으로 이용하면 아래와 같은 코드가 된다.

```java
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight)));
```

### 2.4.2 Runnable로 코드 블록 생성하기
자바 스레드를 이용하면 병렬로 코드 블록을 실행할 수 있다.  
자바 8까지는 Thread 생성자에 객체만을 전달할 수 있었으므로 보통 결과를 반환하지 않는 void run 메서드를 포함하는 익명 클래스가 Runnable 인터페이스를 구현하도록 하는 것이 일반적이었다.  
자바에서는 Runnable 인터페이스를 이용해서 실행할 코드 블록을 지정할 수 있다.

```java
// java.lang.Runnable
public interface Runnable {
    void run();
}
```

Runnable을 이용해서 다양한 동작을 스레드로 실행할 수 있다.

```java
Thread t = new Thread(new Runnable() {
    public void run() {
        System.out.println("Hello");
    }
});   // 익명 클래스 방식

Thread t = new Thread(() -> System.out.println("Hello"));  // 람다 방식

```

### 2.4.3 Callable을 결과로 반환하기
자바 5부터 지원하는 ExecutorService 인터페이스는 task 제출과 실행 과정의 연관성을 끊어준다.  
ExecutorService를 이용하면 task를 Thread pool로 보내고 결과를 Future로 저장할 수 있다는 점이 스레드와 Runnable을 이용하는 방식과는 다르다.

Callable 인터페이스를 이용해 결과를 반환하는 task를 만든다는 것만 알아두자.

```java
public interface Callable<V> {
    V call();
}
```

```java
ExecutorService executorService = Executors.newCachedThreadPool();
Future<String> threadName = executorService.submit(new Callable<String>() {
    @Override
    public String call() throw Exception {
            return Thread.currentThread().getName();
    }
});  // 익명 클래스 방식

Future<String> threadName = executorService.submit(() -> Thread.currentThread().getName()); // 람다 방식
```


## 마치며
- 동작 파라미터화에서는 메서드 내부적으로 다양한 동작을 수행할 수 있도록 코드를 메서드 인수로 전달한다.
- 동적 파라미터화를 이용하면 변화하는 요구사항에 더 잘 대응할 수 있는 코드를 구현할 수 있다.
- 코드 전달 기법을 이용하면 동작을 메서드의 인수로 전달할 수 있다.

