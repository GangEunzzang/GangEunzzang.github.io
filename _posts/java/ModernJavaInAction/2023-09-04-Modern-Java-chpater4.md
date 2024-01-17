---
title: 'chapter4 :: 스트림 소개'
layout: post
categories: java
tags: Modern-Java-In-Action
comments: true
---

* * *

## 스트림이란 무엇인가?

* 스트림은 Java 8 API에 새로 추가된 기능이다.
* 스트림을 이용하면 선언형(데이터를 처리하는 임시 구현 코드 대신 질의로 표현하는 방법)으로 컬렉션 데이터 처리가 가능하다.
* 멀티스레드 코드를 구현하지 않아도 데이터를 투명하게 병렬로 처리할 수 있다.

#### 기존코드 -> 스트림 사용 예제

* 기존 코드

```java

List<Dish< lowCaloricDishes = new ArrayList<>();
for(Dish dish : menu) {
  if(dish.getCalories() < 400) {
    lowCaloricishes.add(dish);
  }
}

Collections.sort(lowCaloricDishes, new Comparator<Dish>() {
  public int compare(Dish dish1, Dish dish2) {
    reutnr Integer.compare(dish1.getCalories(), dish2.getCalories());
});

List<Strin lowCaloricDishesName = new ArrayList<>();
for(Dish dish : lowCaloricDishes) {
  lowCaloricDishesName.add(dish.getName());
}

```

* 스트림 사용

```java
import static java.util.Comparator.comparing;
import static java.util.stream.Collectors.toList;

List<String> lowCaloricDishesName = menu.stream()
  .filter(d -> d.getCalories() < 400) 
  .sorted(comparing(Dish::getCalories))
  .map(Dish::getName) 
  .collect(toList());
```

다음과 같이 변환했을때의 장점이 느껴지는가?  
* 선언형으로 코드를 구현할 수 있다.
* 가비지 변수를 만들지 않는다.
* 여러 빌딩 블록 연산을 연결하여 데이터 처리 파이프라인을 만들 수 있으며, 가독성과 명확성이 유지된다.
* filter, map, sorted, collect 같은 연산은 `고수준 빌딩 블록(high-level building block)`으로 이루어져 있어 특정 스레딩 모델에 제한되지 않고 자유롭게 어떤 상황에서든 사용할 수 있다.

* * *
## 스트림 시작하기

스트림이란 `데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소`로 정의할 수 있다.
* **연속된 요소** : 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스를 제공한다.
* **소스** : 컬렉션, 배열, I/O 자원 등의 데이터 제공 소스로부터 데이터를 소비한다.
* **데이터 처리 연산** : 함수형 프로그래밍 언어에서 일반적으로 지원하는 연산과 DB와 비슷한 연살을 지원하며, 순차적으로 또는 병렬로 실행할 수 있다.

### 스트림 중요한 특징 2가지

* `파이프라이닝(Pipelining)`  
  * 스트림 연산은 스트림 연산끼리 연결해서 커다란 파이프라인을 구성할 수 있도록 스트림 자신을 반환한다.    
    그 덕분에 `게으름(lazyness)`, `쇼트서킷(short-circuiting)` 같은 최적화도 얻을 수 있다

* `내부 반복`
  * 컬렉션은 반복자를 이용해서 명시적으로 반복하지만, 스트림은 내부 반복을 지원한다.
 
* * *

## 스트림과 컬렉션

* 컬렉션과 스트림 모두 연속된 요소 형식의 값을 저장하는 자료구조의 인터페이스를 제공한다.
* 컬렉션과 스트림의 가장 큰 차이는 데이터를 언제 계산하느냐 이다.
* 컬렉션은 현재 자료구조가 포함하는 모든 값을 메모리에 저장하는 자료구조이다. 즉, 컬렉션의 모든 요소는 컬렉션에 추가하기 전에 계산되어야 한다.
* 스트림은 이론적으로 요청할 때만 요소를 계산 하는 고정된 자료구조다. 이러한 스트림의 특성은 게으른 생성 을 가능하게 한다.
* 스트림은 반복자와 마찬가지로 한 번만 탐색할 수 있다. 탐색된 스트림의 요소는 소비된다.

### 외부 반복과 내부 반복
컬렉션 인터페이스는 사용자가 직접 요소를 반복해야 한다. (외부 반복, external iteration)  
스트림 라이브러리는 반복을 알아서 처리하고 결과 스트림값을 어딘가 저장해준다. (내부 반복, internal iteration)

#### 외부 반복 예제
```java
List<String> names = new ArrayList<>();
for (Dish dish : menu) {
    names.add(dish.getName());
}

List<String> names = new ArrayList<>();
Iterator<String> iterator = menu.iterator();
while (iterator.hasNext()) {    // 명시적 반복
    Dish dish = iterator.next();
    names.add(dish.getName());
}

```

#### 내부 반복 예제
```java
List<String> names = menu.stream()
        .map(Dish::getName) // 요리명 추출
        .collect(Collectors.toList());  // 파이프라인 실행(반복자 필요 없음)
```

내부 반복을 이용하면 장점은 다음과 같다.  
  1. 작업을투명하게 병렬로 처리할 수 있다.
  2. 더 최적화된 다양한 순서로 처리할 수 있다.

외부 반복에서는 병렬성을 스스로 관리해야 한다.

* * * 
## 스트림 연산

### 중간 연산
* 단말 연산을 스트림 파이프라인에 실행하기 전까진 아무 연산도 수행하지 않는다.
  즉, `게으르다(lazy)`는 것이므로 지연 연산을 수행한다.
* `filter`나 `sorted` 같은 중간 연산은 다른 스트림을 반환한다. 따라서 여러 중간 연산을 연결해 질의로 만들 수 있다.

### 최종 연산
 * 최종 연산은 스트림 파이프라인에서 결과를 도출한다 이를 통해 List, Integer, void 등 스트림 이외의 결과가 반환된다.

## 마치며

* 스트림은 소스에서 추출된 연속 요소로, 데이터 처리 연산을 지원한다.
* 스트림은 내부 반복을 지원한다. 내부 반복은 filter, map, sorted 등의 연산으로 반복을 추상화한다.
* 스트림에는 중간 연산과 최종 연산이 있다.
* 중간 연산은 filter와 map처럼 스트림을 반환하면서 다른 연산과 연결되는 연산이다. 중간 연산을 이용해서 파이프라인을 구성할 수 있지만 중간 연산으로는 어떤 결과도 생성할 수 없다.
* forEach나 count처럼 스트림 파이프라인을 처리해서 스트림이 아닌 결과를 반환하는 연산을 최종 연산이라고 한다.
* 스트림의 요소는 요청할 때 게으르게 계산된다.


