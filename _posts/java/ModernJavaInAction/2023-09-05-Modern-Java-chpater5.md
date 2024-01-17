---
title: 'chapter5 :: 스트림 활용'
layout: post
categories: java
tags: Modern-Java-In-Action
comments: true
---

* * *
## 5.1 필터링
* * *

### 5.1.1 프레디케이트로 필터링
- filter 메서드는 `프레디케이트`를 인수로 받아서 프레디케이트와 일치하는 모든 요소를 포함하는 스트림을 반환한다. 

```java
List<Dish> vegetarianMenu = menu.stream()
        .filter(Dish::isVegetarian)
        .collect(toList());
```

### 5.1.2 고유 요소 필터링
- 스트림은 고유 요소로 이루어진 스트림을 반환하는 `distinct` 메서드도 지원한다. (고유 여부는 스트림에서 만든 객체의 hashCode, equals로 결정된다.)

리스트의 모든 짝수를 선택하고 중복을 필터링하는 예제

```java
List<Integer> numbers = Arrays.asList<1, 2, 1, 3, 3, 2, 4);
numbers.stream()
  .filter(i -> i % 2 == 0)
  .distinct()
  .forEach(System.out::println);
```

* * *
## 5.2 스트림 슬라이싱
* * *

### 5.2.1 프레디케이트를 이용한 슬라이싱
- 자바 9의 스트림의 요소를 효과적으로 선택할 수 있도록 `takeWhile`, `dropWhile` 두 가지 새로운 메서드를 지원한다.

```java
List<Dish> specialMenu = Arrray.asList(
  new Dish("seasonal fruit", true, 12, Dish.Type.OTHER),
  new Dish("prawns", false, 300, Dish.Type.FISH),
  new Dish("rice", true, 350, Dish.Type.OTHER),
  new Dish("chicken", false, 400, Dish.Type.MEAT),
  new Dish("french fries", true, 530, Dish.Type.OTHER),
);
```

위에 데이터를 이용하여 실습해보자.  
현재 데이터는 이미 정렬되어 있는 상태이다.

만약 320 칼로리 이하의 요리를 선택하려면 filter를 적용하려고 할 것이다.   
filter는 모든 요소를 순회해야 하지만 `TakeWhile` 을 이용하면 반복작업을 중단 할 수 있다.   

320 칼로리보다 크거나 같은 요리가 나왔을때 반복작업을 중단할 수 있는 예제이다.

```java
List<Dish> slicedMenu1
  = specialMenu.stream()
    .takeWhile(dish -> dish.getCalories() < 320)
    .collect(toList());
```

만약 나머지 요소 즉 320칼리보다 큰 요소를 탐색하려면 `dropWhile`을 이용하면 된다.  
dropWhile은 프레디케이트가 처음으로 거짓이 되는 지점까지 발견된 요소를 버리고, 남은 요소를 반환한다.

```java
List<Dish> slicedMenu2
  = specialMenu.stream()
    .dropWhile(dish -> dish.getCalories() < 320)
    .collect(toList());
```

### 5.2.2 스트림 축소
- 스트림은 주어진 값 이하의 크기를 갖는 새로운 스트림을 반환하는 limit(n) 메서드를 지원한다.

### 5.2.3 요소 건너뛰기
- 스트림은 처음 n개 요소를 제외한 스트림을 반환하는 skip(n) 메서드를 지원한다.   
- n개 이하의 요소를 포함하는 스트림에 skip(n)을 호출하면 빈 스트림이 반환된다.

* * *

## 5.3 매핑
- 스트림 API의 map과 flatMap 메소드는 특정 데이터를 선택하는 기능을 제공한다.

### 5.3.1 스트림의 각 요소에 함수 적용하기
- 스트림은 함수를 인자로 받은 map 메소드를 지원한다.
- 인수로 제공된 함수는 각 요소에 적용되며 함수를 적용한 결과가 새로운 요소로 매핑된다.

```java
ist<String> dishNames = menu.stream()
        .map(Dish::getName)
        .collect(Collectors.toList());
```

### 5.3.2 스트림 평면화

영단어가 담긴 리스트에서 각 단어의 알파벳을 포함하는 리스트를 반환한다고 가정하자.  
예를 들어, `["Hello", "World"]`에서 `["H", "e", "l", "o", "W", "r", "d"]`를 포함하는 리스트가 반환되어야 한다.

```java
words.stream()                        
.map(word -> word.split(""))    
.distinct()                    
.collect(Collectors.toList());  
```

위 코드에서 map으로 전달한 람다는 각 단어의 `String[]`을 반환해서 원하는 반환값을 얻을 수 없다.  
문자열 배열을 받아 문자열 스트림을 만드는 `Arrays.stream()` 메소드를 사용해보자.

```java
words.stream()                          
        .map(word -> word.split(""))  
        .map(Arrays::stream)            
        .distinct()                     
        .collect(Collectors.toList());  // List<Stream<String>>
```

위 방법은 원하던 결과인 List이 아닌 List<Stream>을 반환해준다.

#### flatMap 사용
- `flatMap`을 사용하면 다음처럼 문제를 해결할 수 있다.
- 스트림의 각 값을 다른 스트림으로 만든 다음 모든 스트림을 하나의 스트림으로 연결하는 기능을 수행한다.

```java
words.stream()                         
        .map(word -> word.split(""))    
        .flatMap(Arrays::stream)       
        .distinct()                   
        .collect(Collectors.toList());  
```

* * *

## 5.4 검색과 매칭
- 특정 속성이 데이터 집합에 있는지 여부를 검사한다.
- allMatch, anyMatch, noneMatch, findFirst, findAny 등 제공

### 프레디케이트가 적어도 한 요소와 일치하는지 확인
- `anyMatch` 메서드 사용

```java
if(menu.stream().anyMatch(Dish::isVegetarian)) {
  System.out.println("The menu is somewhat vegetarian friendly!!");
}
```

### 프레디케이트가 모든 요소와 일치하는지 확인
- `allMatch` 메서드 사용

```java
boolean isHealthy = menu.stream().allMatch(dish -> dish.getCalories() < 1000);
```

### 프레디케이트가 모든 요소와 일치하지 않는지 확인
- `noneMatch` 메서드 사용

```java
boolean isHealthy = menu.stream().noneMatch(dish -> dish.getCalories() >= 1000);
```

### 쇼드서킷(Short Circuit)이란?
예를 들어 여러 and 연산으로 연결된 커다란 불리언 표현식을 평가한다고 가정하자.  
표현식에서 하나라도 거짓이라는 결과가 나오면 나머지 표현식의 결과와 상관없이 전체 결과도 거짓이 되는데, 이러한 상황을 쇼트서킷이라고 부른다.   
스트림의 `allMatch`, `noneMatch`, `findFirst`, `findAny`, `limit` 등의 연산은 모든 스트림의 요소를 처리하지 않고도 반환할 수 있다.   
즉, 원하는 요소를찾았으면 즉시 결과를 반환할 수 있다.

### 요소 검색
- `findAny` 메서드 사용 
  - 스트림에서 임의의 요소를 반환한다.

```java
Optional<Dish> dish = menu.stream()
  .filter(Dish::isVegetarian)
  .findAny();
```

### 첫 번째 요소 찾기
- `findFirst` 메서드 사용

```java
List<Integer> someNumbers = Arrays.asList(1,2,3,4,5);
Optional<Integer> firstSquareDivisibleByThree = someNumbers.stream()
  .map(n -> n * n)
  .filter(n -> n % 3 == 0)
  .findFirst(); //9
```

### `findFirst`와 `findAny`는 언제 사용할까?
findFirst와 findAny의 반환값 차이는 다르지 않다.   
그러나 두 개의 메서드가 모두 필요한 이유는 `병렬성` 때문이다.  
병렬 실행에서는 첫 번째 요소를 찾기 어렵다.  
따라서 요소의 반환 순서가 상관없다면 병렬스트림에서는 제약이 적은 `findAny`를 사용한다.

* * * 

## 5.5 리듀싱
- 모든 스트림 요소를 처리해서 값으로 도출하는 연산 의미
- 함수형 프로그래밍 언어 용어로는 `폴드`라고 부른다.

### 5.5.1 요소의 합

먼저 리스트의 숫자 요소를 더하는 코드를 확인해보자.

```java
int sum = 0;
for (int x : numbers) {
	sum += x;
}
```

위 코드는 reduce를 사용하여 다음과 같이 변경할 수 있다.
```java
int sum = numbers.stream().reduce(0, (a,b) -> a + b);

// 메서드 참조 버전
int sum = numbers.stream().reduce(0, Integer::sum);
```

#### 초기값 없음
초기값을 받지 않도록 오버로드된 reduce도 있다. 하지만 Optional 객체를 반환한다.
```java
Optional<Integer> sum = numbers.stream().reduce(Integer::sum);
```

### 5.5.2 최댓값과 최솟값

```java
Optional<Integer> max = numbers.stream().reduce(Integer::max); // 최댓값
Optional<Integer> min = numbers.stream().reduce(Integer::min); // 최솟값
```

### reduce 메서드의 장점과 병렬화
기존의 단계적 반복으로 합계를 구하는 것 vs reduce를 이용해서 합계를 구하는 것  
- 기존방식 -> sum 변수를 공유해야 하므로 쉽게 병렬화 하기가 어렵다.
- reduce -> 내부 반복이 추상화되면서 내부 구현에서 병렬로 reduce를 실행한다.

* * *
## 5.7 숫자형 스트림
* * *
### 5.7.1 기본형 특화 스트림
- 스트림 API는 박싱 비용을 피할 수 있도록 세 가지 기본형 특화 스트림을 제공한다.
  - IntStream
  - DoubleStream
  - LongStream
- sum, max 등 숫자 관련 메서드를 제공한다.
- 오직 박싱 과정에서 일어나는 효율성과 관련 있으며 스트림에 추가 기능을 제공하지는 않는다.

### 숫자 스트림으로 매핑
- `mapToInt`, `mapToDouble`, `mapToLong` 세 가지 메서드를 가장 많이 사용한다.   
- 위 메서드들은 map 과 같은 기능을 수행하지만, Stream<T> 대신 특화된 스트림을 반환한다.

```java
int calories = menu.stream()            // <- Stream<Dish> 반환
        .mapToInt(Dish::getCalories)    // <- IntStream 반환
        .sum();
```

### 객체 스트림으로 복원하기
- IntStream의 map 연산은 IntUnaryOperator(int -> int)를 인수로 받는다.
- 숫자 스트림에서 스트림으로 복원하기 위해서는 `boxed()` 를 사용할 수 있다.

```java
List<Integer> calories = menu.stream()  // Stream<Dish>
        .mapToInt(Dish::getCalories)    // IntStream
        .boxed()                        // Stream<Integer>
        .collect(Collectors.toList());
```

### 기본값: OptionalInt
- 합계 예제에서는 0이라는 기본값이 있었으므로 별 문제가 없었다.
- IntStream에서 최대값을 찾을 때는 0이라는 기본값 때문에 잘못된 결과가 도출될 수 있다.
- 스트림에서 요소가 없는 상황과 실제 최댓값이 0인 상황을 구별하기 위해서 값의 존재 여부를 가리킬 수 있는 컨테이너 클래스 Optional을 Integer, String 등의 참조 형식으로 파라미터화할 수 있다.

```java
int maxCalory = menu.stream()
        .mapToInt(Dish::getCalories)
        .max()          // OptionalInt
        .orElse(1);     // 값이 없을 때 기본 최댓값을 명시적으로 설정
```

### 5.7.2 숫자 범위
프로그램에서 특정 범위의 숫자를 이용해야 할 때 IntStream과 LongStream에서는 `range`와 `rangeClosed` 두 정적 메소드를 사용할 수 있다.
- range : 종료값이 결과에 포함되지 않음
- rangeClosed : 종료값이 결과에 포함됨


## 마치며
- 스트림 API를 이용하면 복잡한 데이터 처리 질의를 표현할 수 있다.
- `filter`, `distinct`, `takeWhile`, `dropWhile`, `skip`, `limit` 메서드로 스트림을 필터링하거나 자를 수 있다.
- 소스가 정렬되어 있다는 사실을 알고 있을 때 `takeWhile`과 `dropWhile` 메서드를 효과적으로 사용할 수 있다.
- `map`, `flatMap` 메서드로 스트림의 요소를 추출하거나 반환할 수 있다.
- `findFirst`, `findAny` 메서드로 스트림의 요소를 검색할 수 있다.
- `allMatch`, `noneMatch`, `anyMatch` 메서드를 이용해서 주어진 프레디 케이트와 일치하는 요소를 스트림에서 검색할 수 있다.



