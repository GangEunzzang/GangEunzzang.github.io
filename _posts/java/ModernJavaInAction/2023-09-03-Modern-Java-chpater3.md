---
title: 'chapter3 :: 람다 표현식'
layout: post
categories: java
tags: Modern-Java-In-Action
comments: true
---

* * *
## 3.1 람다란 무엇인가?
- 메서드로 전달할 수 있는 익명 함수를 단순화 시킨 것
- `파라미터` + `화살표` + `바디`로 이루어진다.

### 람다의 특징
- 익명
  - 보통의 메서드와 달리 이름이 없으므로 익명
- 함수
  - 람다는 메서드처럼 특정 클래스에 종속되지 않으므로 함수라고 부른다.
- 전달
  - 람다 표현시글 메서드 인수로 전달하거나 변수로 저장 가능하다.
- 간결
  - 익명 클래스처럼 많은 자질구레한 코드를 구현할 필요가 없다.

### 자바에서 지원하는 다섯 가지 람다 표현식 예제

```java

(String s) -> s.length() // String 형식의 파라미터를 받으며, int 반환 람다 표현식에는 return 문이 함축되어있어서 명시하지 않아도 된다.
(Apple a) -> a.getWeight() > 150 // Apple 형식의 파라미터를 받으며, boolean 반환
(int x, int y) -> {
  System.out.println("Result : ");
  System.out.println(x+y);
}
// int 형식의 파라미터 두 개를 가지며 리턴 값이 없다.

() -> 42 // 파라미터가 없으며 42를 반환한다.

(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight())

```

* * *
## 3.2 어디에, 어떻게 람다를 사용할까?
함수형 인터페이스 문맥에서 사용할 수 있다.

### 3.2.1 함수형 인터페이스
- 오직 하나의 추상메서드만 지정하는 인터페이스
-  **Predicate**, **Comparator**, **Runnable** 등이 있다.
-  default 메서드는 상관없다.

#### 함수형 인터페이스 예제

```java

public interface Predicate<T> {
  boolean Test (T t);
}

public interface Comparator<T> {
  int compare(T o1, T o2);
}

public interface Runnable {
  void run();
}

public interface ActionListener extends EventListener {
  void actionPerformed(ActionEvent e);
}

public interface Callable<V> {
  V call() throws Exception;
}

public interface PrivilegedAction<T> {
  T run();
}

```

### 3.2.2 함수 디스크립터(function descriptor)
- 함수형 인터페이스의 추상메서드 시그니처는 람다 표현식의 시그니처를 가리킨다.
- 람다 표현식의 시그니처를 서술하는 메서드를 의미한다.
- 예를들어 `Runnable` 인터페이스의 메서드 run은 인수와 반환값이 없으므로 void 반환 `() -> void`로 표기할 수 있다.
- 람다 표현식은 `함수형 인터페이스`를 인수로 받는 메서드에만 람다 표현식을 사용할 수 있다.

#### @FunctionalInterface
- 함수형 인터페이스에 붙는 어노테이션
- 위 어노테이션을 선언했지만 함수형 인터페이스가 아니면 컴파일 에러 발생

* * *

 ## 3.3 람다 활용 - 실행 어라운드 패턴
- 자원처리(DB, 파일처리 등)에 사용하는 `순환 패턴(recurrent pattern)` 은 자원을 열고, 로직을 실행후, 자원을 닫는 순서로 이뤄진다.
- 설정(setup)과 정리(cleanup) 과정은 대부분 비슷하다. 즉, 실제 자원을 처리하는 코드를 설정과 정리 두 과정이 둘러싸는 형태를 갖는다.  

자바의 `try-with-resource` 구문을 사용하면, 자원을 명시적으로 닫을 필요가 없다.

```java
  
public String processFile() throws IOException {
  try ( BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
    return br.readLine(); // 로직 구현
  }
}

```

### 1단계: 동작 파라미터화를 기억하라
현재 코드는 파일에서 한번에 한 줄만 읽을 수 있다.   
만약 한 번에 두 줄을 읽거나 가장 자수 사용되는 단어를 반환하려면 어떻게 해야 할까?
바로 `processFile()`을 동적 파라미터화 시키는 것이다.

```java
String result = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```

### 2단계: 함수형 인터페이스를 이용해서 동작 전달
함수형 인터페이스 자리에 람다를 사용할 수 있다.  
따라서 BufferedReader -> String과 IOException을 throw 할 수 있는 시그니처와 일치하는 함수형 인터페이스 `BufferedReaderProcessor`를 생성한다.

```java

@FunctionalInterface
public interface BufferedReaderProcessor {
  String process(BufferedReader b) throws IOException;
}

위와 같이 정의한 인터페이스를 processFile메서드의 인수로 전달할 수 있다.
```java

public String processFile(BufferedReaderProcessor p) throws IOException {
  // ...
}

```

### 3단계: 동작 실행
이제 `BufferedReaderProcessor`에 정의된 `process` 메서드의 시그니처(BufferedReader -> String)와 일치하는 람다를 전달할 수 있다.  
람다 표현식으로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있다.  
전달된 코드는 함수형 인터페이스의 인스턴스로 전달된 코드와 같은 방식으로 처리된다.  
  
따라서 `processFile` 메서드의 바디 내에서 `BufferedReaderProcessor` 객체의 process를 호출할 수 있다.

```java

public String processFile(BufferedReaderProcessor p) throws IOException {
  try (BufferedReader br = new BufferedReader(new fileReader("data.txt"))) {
    return p.process(br); // BufferedReader 객체 처리
  }
}

```

### 4단계: 람다 전달
이제 람다를 이용해 다양한 동작을 processFile 메서드로 전달할 수 있다.

```java

String oneLine = processFile((BufferedReader br) -> br.readLine()); // 람다
String oneLine = processFile(BufferedReader::readLine); // 메서드 참조

```

* * *

## 3.4 함수형 인터페이스 사용 
- 함수형 인터페이스의 추상메서드 시그니처를 함수 디스크립터(function descriptor)라고 한다.
- 다양한 람다식을 사용하기 위해선 공통의 함수 디스크립터를 기술하는 함수형 인터페이스 집합이 필요하다.
- 자바 8에서는 `java.util.function` 패키지로 여러 가지 새로운 함수형 인터페이스를 제공한다.

### 3.4.1 Predicate
- Predicate 인터페이스는 test라는 추상 메서드를 정의하며 test는 제네릭 형식 T의 객체를 인수로 받아 boolean을 반환한다.
- 우리가 만들었던 인터페이스와 같은 형태인데 따로 정의할 필요 없이 바로 사용할 수 있다는 점이 특징이다.
- 제네릭 T 형식의 객체를 사용하는 boolean 표현식이 필요한 상황에서 Predicate 인터페이스를 사용할 수 있다.

아래 예제처럼 String 객체를 인수로 받는 람다를 정의할 수 있다.

```java
@FunctionalInterface
public interface Predicat<T> {
  boolean test(T t);
}
public <T> List<T> filter(List<T> list, Predicate<T> p) {
  List<T> results = new ArrayList<>();
  for(T t : list) {
    if(p.test(t)) {
      results.add(t);
    }
  }
  return results;
}
Predicat<String> nonEmptyStrinPredicate = (String s) -> !s.isEmpty();
List<String> nonEmpty = filter(listOfStrings, nonEmptyStringPredicate);
```

### 3.4.2 Consumer
- Consumer 인터페이스는 제네릭 형식 T 객체를 받아서 void를 반환하는 accept라는 추상 메서드를 정의한다.
- 제네릭 T 형식의 객체를 인수로 받아서 어떤 동작을 수행하고 싶을 때 Consumer 인터페이스를 사용할 수 있다.
- 예를들어 Integer 리스트를 인수로 받아서 각 항목에 어떤 동작을 수행하는 forEach 메서드를 정의할 때 활용할 수 있다.

아래는 forEach와 람다를 이용해 list의 모든 항목을 출력하는 예제이다.

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}

public <T> void forEach(List<T> list, Consumer<T> c) {
    for (T t : list) {
        c.accept(t);
    }
}
forEach(Arrays.asList(1,2,3,4,5), (Integer i) -> System.out.println(interface));
```
### 3.4.3 Function
- Function<T,R> 인터페이스는 제네릭 형식 T를 인수로 받아 제네릭 형식 R 객체를 반환하는 추상 메서드 apply를 정의한다.
- 입력을 출력으로 매핑하는 람다를 정의할 때 Function 인터페이스를 활용할 수 있다.

아래는 String 리스트를 인수로 받아 각 String의 길이를 포함하는 List<Integer>로 변환하는 map 메서드를 정의한 코드이다.

```java

@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}

public <T, R> List<R> map(List<T> list, Function<T, R> f) {
    List<R> results = new ArrayList<>();
    for (T t : list) {
      results.add(f.apply(t));
    }
    return results;
}

//[7,2,6]
List<Integer> l = map(
        Arrays.asList("lambdas","in","action"),
        (String s) -> s.length() // Function의 apply 메서드를 구현하는 람다
);

```

#### 구현방식
1. 함수형 인터페이스를 선언한다.  
2. 함수형 인터페이스를 동작 파라미터화 시켜서 자신이 원하는 로직을 처리하는 메서드를 만든다.  
3. 해당 메서드를 호출할때 파라미터로 람다를 넘길 수 있다.  

#### 기본형 특화
- 자바의 모든 형식은 참조형 or 기본형에 해당한다.
- 하지만 제네릭 내부 구현 때문에 제네릭 파라미터에는 **참조형**만 사용할 수 있다.
- 자바에서는 오토박싱을 지원하는데, 이러한 변환과정에서 비용이 들기 때문에 오토박싱을 피할 수 있는 함수형 인터페이스도 제공한다.

아래 코드의 `IntPredicate`는 1000이라는 값을 박싱하지 않지만, `Predicate<Integer>는 값을 Integer 객체로 반환하는 코드이다.

```java
public interface IntPredicate {
    boolean test(T t);
}
IntPredicate evenNumbers = (int i) -> i % 2 == 0;
evenNumbers.test(1000) // 참 (박싱 없음)

Predicate<Integer> oddNumbers = (Integer i) -> i % 2 != 0;
oddNumbers.test(1000); // 거짓 (박싱)
```

* * *

## 3.5 형식 검사, 형식 추론, 제약
람다 표현식을 처음 설명할 때 람다로 함수형 인터페이스의 인스턴스를 만들 수 있다고 언급했다.   
람다 표현식 자체에는 람다가 어떤 함수형 인터페이스를 구현하는지의 정보가 포함되어 있지 않다.   
따라서 람다 표현식을 더 제대로 이해하려면 람다의 실제 형식을 파악해야 한다.

### 3.5.1 형식 검사
- 람다가 사용되는 콘텍스트(context)를 이용해서 람다의 형식(type)을 추론할 수 있다.
- 어떤 콘텍스트에서 기대되는 람다 표현식의 형식을 `대상 형식(target type)`이라고 한다.  

### 3.5.2 같은 람다, 다른 함수형 인터페이스
- 대상 형식이라는 특징 때문에 같은 람다 표현식이라도 호환되는 추상 메서드를 가진 다른 함수형 인터페이스로 사용될 수 있다.
- 예를 들어 이전에 살펴본 `Callable`과 `PrivilegedAction` 인터페이스는 인수를 받지 않고 제네릭 형식 T를 반환하는 함수를 정의한다.  

### 3.5.3 형식 추론
- 자바 컴파일러는 람다 표현식이 사용된 콘텍스트(대상 형식)를 이용해서 람다 표현식과 관련된 함수형 인터페이스를 추론한다.
- 대상 형식을 이용해서 함수 디스크립터를 알 수 있으므로 컴파일러는 람다의 시그니처도 추론할 수 있다.

### 3.5.4 지역 변수 사용
- 지금까지 살펴본 모든 람다 표현식은 인수를 자신의 바디 안에서만 사용했다.
- 람다 표현식에서는 익명 함수가 하는 것처럼 `자유 변수`(파라미터로 넘겨진 변수가 아닌 외부에서 정의된 변수)를 활용할 수 있다.
- 이와 같은 동작을 `람다 캡처링`이라고 부른다.   

다음은 portNumber 변수를 캡처하는 람다 예제이다.

``` java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
```

하지만 자유 변수에도 제약이 있다.   
람다는 인스턴스 변수와 정적 변수를 자유롭게 캡처(자신의 바디에서 참조) 할 수 있다.  
하지만 그러러면 `지역 변수는 명시적으로 final로 선언되어 있거나, 실질적으로 final로 선언된 변수와 똑같이 사용 되어야 한다.`     
아래 코드는 porNumber에 값을 두 번 할당하므로 컴파일에러가 발생한다.

```java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
portNumber = 31337;
```

* * *
## 3.6 메서드 참조(Method Reference)
- Java 8의 새로운 기능
- 특정 메서드만을 호출하는 람다 표현식의 축양형
- 기존의 메서드 정의를 재활용하여 람다처럼 전달 가능
- 때로는 람다 표현보다 더 가독성이 좋으며 자연스러울 수 있다.   

```java
 inventory.sort((Apple a1, Apple a2) ->a1,getWeight().compareTo(a2.getWeight()));  // 람다식

 inventory.sort(comparing(Apple::getWeight));  // 메서드 참조 방식
```

### 메서드 참조 방법
1. 정적 메서드 참조
   - Integer::parseInt
   - ```java
     Function<String, Integer> stringToInteger = Integer::parselnt;
     ```
     
2. 인스턴스 메서드 참조
   - String::length
   - 메서드 참조를 이용하여 람다 표현식의 파라미터로 전달할 때 사용
   - ```java
     BiPredicate<List<String>, String> contains = List::contains;
     ```
     
3. 기존 객체의 인스턴스 메서드 참조
   - 현존하는 외부 객체의 메서드를 호출할 때 사용
   - 비공개 헬퍼 메서드를 정의한 상황에서 유용하게 활용 가능
   - ```java
     Predicate<String> startsWithNumber = this::startsWithNumber
     ```

### 생성자 참조
`ClassName::new` 처럼 class명과 new 키워드를 이용하여 기존 생성자의 참조를 만들 수 있다.

```java
// Supplier<Apple> c1 = () -> new Apple();
Supplier<Apple> c1 = Apple::new;
Apple a1 = c1.get();
```

* * *
## 람다, 메서드 참조 활용하기
처음에 다룬 사과 리스트를 다양한 정렬 기법으로 정렬하는 문제로 돌아가보자  



### 1단계: 코드 전달
어떻게 sort 메서드에 정렬 전략을 전달할 수 있을까?
List API에서 sort 메서드를 제공하므로 직접 정렬 메서드를 구현할 필요는 없다.  
sort메서드는 다음과 같은 시그니처를 갖는다.

```java
void sort(Comparator<? super E> c)
```

sort 메서드는 `Comparator` 객체를 인수로 받아 두 사과를 비교한다.   

```java
public class AppleComparator implements Comparator<Apple> {
	public int compare(Apple a1, Apple a2) { 
    	return a1.getWeight().compareTo(a2.getWeight()); 
    } 
}

inventory.sort(new AppleComparator());
``` 
   
### 2단계: 익명 클래스 사용
한번만 사용할 Comparator를 구현체를 만들어 구현하는 것보다는 익명 클래스를 이용하는 것이 더욱 좋다.   

```java
inventory.sort(new Comparator<Apple>() {
	public int compare(Apple a1, Apple a2) { 
    	return a1.getWeight().compareTo(a2.getWeight()); 
 	} 
}
```
   
### 3단계: 람다 표현식 사용 
아직도 코드가 장황한 느낌이 있다.   
위에서 배운 람다 표현식을 적용해보자.   

```java

// 1단계
inventory.sort((Apple a1, Apple a2) -> 
	a1.getWeight().compareTo(a2.getWeight()));

// 2단계 (컴파일러가 타입을 추론하기 때문에 타입 생략 가능)
inventory.sort((a1, a2) -> 
	a1.getWeight().compareTo(a2.getWeight()));

// 3단계 (import에 Comparator를 포함하고, Comparator의 comparing 정적 메서드 사용 가능
import static java.util.Compartor.comparing;
inventory.sort(comparing(apple -> apple.getWeight());

```
  
### 4단계: 메서드 참조 사용
메서드 참조를 적용하여 더욱 깔끔하게 변경할 수 있다.  

```java
inventory.sort(comparing(Apple::getWeight));
```

코드만 짧아진 게 아니라 의미도 명확해졌다.
코드 자체로 `Apple을 weight 별로 비교하여 정렬하라는 의미를 바로 파악할 수 있다.`

## 마치며
- 람다는 `파라미터 -> 바디` 로 이루어진다.
- 많은 default 메서드가 존재하더라도 `추상 메서드`가 하나이면 함수형 인터페이스이다.
- 람다 표현식의 시그니처를 서술하는 메서드를 `함수 디스크립터` 라고 한다.
- 메서드 참조를 이용하면 기존의 메서드 구현을 재사용하고 직접 전달할 수 있다.
- 자바 8의 혁신인 람다와 메서드참조를 적극 이용하여 코드의 가독성을 높이자 ~! 




