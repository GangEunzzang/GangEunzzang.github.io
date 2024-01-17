---
title: ' Item 2 :: 생성자에 매개변수가 많다면 빌더를 고려하라'
layout: post
categories: java
tags: Effective-Java
comments: true
---

## Item 2 :: 생성자에 매개변수가 많다면 빌더를 고려하라
정적 팩터리와 생성자의 공통적인 제약은 선택적 매개변수가 많을 때 적절히 대응하기 어렵다. 책에서는 NutritionFacts라는 클래스를 예로 들고 있다.
### 1. 점층적 생성자 패턴(telescoping constructor pattern)
- 매개 변수가 많을 때 프로그래머들이 주로 사용하는 패턴
- telescop는 망원경이라는 뜻. 변수가 늘어나면서 메서드가 추가된 모양이 망원경 같다고 해서 붙여졌다고 한다.

```java
package effectivejava.chapter2.item2.telescopingconstructor;

// 코드 2-1 점층적 생성자 패턴 - 확장하기 어렵다! (14~15쪽)
public class NutritionFacts {
    private final int servingSize;  // (mL, 1회 제공량)     필수
    private final int servings;     // (회, 총 n회 제공량)  필수
    private final int calories;     // (1회 제공량당)       선택
    private final int fat;          // (g/1회 제공량)       선택
    private final int sodium;       // (mg/1회 제공량)      선택
    private final int carbohydrate; // (g/1회 제공량)       선택

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }
    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize  = servingSize;
        this.servings     = servings;
        this.calories     = calories;
        this.fat          = fat;
        this.sodium       = sodium;
        this.carbohydrate = carbohydrate;
    }

    public static void main(String[] args) {
        //이 클래스의 인스턴스를 만들기 위한 클라이언트 코드
        NutritionFacts cocaCola =
                new NutritionFacts(240, 8, 100, 0, 35, 27);
    }
   
}
```
이런 생성자를 쓰다보면 필요없는 매개변수도 넘겨야 하는 경우가 발생한다. 보통 0 같은 기본 값을 넘기는데 위 소스의 경우에도 이 클래스의 인스턴스를 만들기 위해 지방(fat)에 0을 넘겼다. 여기서는 매개변수가 겨우 6개뿐이라 괜찮아보일 수는 있지만, 매개변수의 수가 더 늘어나면 금세 겉잡을 수 없게 된다.  
또 각각의 숫자가 무엇을 뜻하는지 알기가 어렵다.
요약하면, 매개변수 개수가 많아지면 **이런 코드는 작성하기도 어렵고 읽기도 어렵다.**

### 2. 자바빈즈 패턴(JavaBeans pattern)
- 매개변수가 없는 생성자로 객체를 만든 후, 세터 메서드를 호출해 원하는 매개변수의 값을 설정하는 방식

```java
package effectivejava.chapter2.item2.javabeans;

// 코드 2-2 자바빈즈 패턴 - 일관성이 깨지고, 불변으로 만들 수 없다. (16쪽)
public class NutritionFacts {
    // 매개변수들은 (기본값이 있다면) 기본값으로 초기화된다.
    private int servingSize  = -1; // 필수; 기본값 없음
    private int servings     = -1; // 필수; 기본값 없음
    private int calories     = 0;
    private int fat          = 0;
    private int sodium       = 0;
    private int carbohydrate = 0;

    public NutritionFacts() { }
    // Setters
    public void setServingSize(int val)  { servingSize = val; }
    public void setServings(int val)     { servings = val; }
    public void setCalories(int val)     { calories = val; }
    public void setFat(int val)          { fat = val; }
    public void setSodium(int val)       { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }

    public static void main(String[] args) {
        //이 클래스의 인스턴스를 만들기 위한 클라이언트 코드
        NutritionFacts cocaCola = new NutritionFacts();
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
        cocaCola.setCalories(100);
        cocaCola.setSodium(35);
        cocaCola.setCarbohydrate(27);
    }
}
```
이 클래스의 인스턴스를 만들기 위한 클라이언트 코드를 보면 코드가 조금 길어지긴 했지만 인스턴스를 만들기 쉽고, 그 결과 더 읽기 쉬운 코드가 되었다.  
단, 자바빈즈 패턴에서는 객체 하나를 만들려면 메서드를 여러개 호출해야 하고, **객체가 완전히 생성되기 전까지는 일관성이 무너진 상태**에 놓이게 된다. 예를들어 `cocaCola.setCalories(100);`까지만 소스가 수행되었는데 객체가 사용될 여지가 있다.
이처럼 일관성이 무너지는 문제 때문에 **클래스를 불변으로 만들 수 없으며** 스레드 안전성을 얻으려면 프로그래머가 locking과 같은 추가 작업을 해줘야한다.
이런 단점을 완화하고자 생성이 끝난 객체를 수동으로 freezing할 수 있다. 다른 언어에는 freeze()라는 매서드를 제공하지만 Java는 따로 없어 구현을 직접 해야한다.

### 3. 빌더 패턴(JavaBeans pattern)
 - 점층적 생성자 패턴의 안전성과 자바빈즈 패턴의 가독성을 겸비한 패턴.
 1. 빌더 패턴에서 클라이언트는 필요한 객체를 직접 만들지 않고 **필수 매개변수**만으로 생성자(혹은 static 팩토리)를 호출해 빌더 `Builder`객체를 얻는다.
 2. 그런 다음 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 객체 **선택 매개변수**들을 설정한다.
 3. 매개변수가 없는 `build` 메소드를 호출해 만들려는 객체를 얻는다.

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
        .calories(100).sodium(35).carbohydrate(27).build();
```
위 클라이언트 코드는 쓰기도 쉽고, 읽기도 쉽다. 빌더패턴은 파이썬과 스칼라에 있는 명명된 선택적 매개변수(`Named Optional Parameter`)를 흉내낸 것이다.  

잘못된 매개변수를 최대한 일찍 발견하려면 빌더의 생성자(아래소스코드 표시 참고 - 1)와 메서드(2)에서 입력 매개변수의 유효성을 검사하고, build 메서드가 호출하는 생성자(3의 `return new NutritionFacts(this);`)에서 여러 매개변수에 걸친 불변식을 검사하면 된다. 빌더로부터 매개변수를 복사한 후 해당 객체의 필드들을 검사하고, 검사를 실패하면 `IllegalArgumentException` 에러메시지를 통해 어떤 매개변수가 잘못되었는지를 알려줄 수 있다.

```java
package effectivejava.chapter2.item2.builder;

// 코드 2-3 빌더 패턴 - 점층적 생성자 패턴과 자바빈즈 패턴의 장점만 취했다. (17~18쪽)
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 - 기본값으로 초기화한다.
        private int calories      = 0;
        private int fat           = 0;
        private int sodium        = 0;
        private int carbohydrate  = 0;

        //수행순서(1)
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }

        //수행순서(2)
        public Builder calories(int val)
        { calories = val;      return this; }
        public Builder fat(int val)
        { fat = val;           return this; }
        public Builder sodium(int val)
        { sodium = val;        return this; }
        public Builder carbohydrate(int val)
        { carbohydrate = val;  return this; }

        //수행순서(3)
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    //수행순서(4)
    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                .calories(100).sodium(35).carbohydrate(27).build();
    }
```

NutiriFacts 클래스는 불변이며, 모든 매개변수의 기본값들을 한 곳에 모아뒀다.  
- **플루언트 API(Fluent API) / 메서드 연쇄(method chaining)**  
빌더의 세터 메서드들(2)은 빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다.  
이런 방식을 메서드 호출이 흐르듯 연결된다는 뜻으로 플루언트 API혹은 메서드 연쇄라한다.
- #### 불변과 불변식  
    - **불변(immutable or immutability)**: 어떠한 변경도 허용하지 않음  
        ex) String 객체는 한번 만들어지면 절대 값을 바꿀 수 없는 불변 객체
    - **불변식(invariant)**: 프로그램이 실행되는 동안, 혹은 정해진 기간 동안 반드시 만족해야하는 조건. 즉, 변경을 허용할 수는 있으나 주어진 조건 내에서만 허용  
        ex)
        리스트의 크기는 0 이상이어야 함 - 음수인경우 불변식이 깨짐.  
        기간을 표현하는 Period 클래스에서 start 필드는 End 필드의 값보다 앞서야 함 - 두 값이 역전되는 경우 불변식이 깨짐  
    - 따라서 불변 객체에도 불변식은 존재할 수 있으며, 넓게 보면 불변은 불변식의 극단적인 예임.
   
빌더패턴은 클래스 계층구조와 함께 쓰기에 좋다. 추상 빌더를 가지고 있는 추상클래스를 만들고 하위 클래스에서는 추상 클래스를 상속받으며 각 하위 클래스용 빌더도 추상 빌더를 상속받아 만들 수 있다.   

``` java
package item02;

import java.util.EnumSet;
import java.util.Objects;

public abstract class Pizza {
	public enum Topping{
		HAM, MUSHROOM, ONION
	}
	
	final EnumSet<Topping> toppings;
	
	//Builder<T extends Builder<T>>: 자기자신의 하위타입을 받는 빌더. 재귀적인 타입 매개변수라고도 함.
	abstract static class Builder<T extends Builder<T>>{
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class); //기본 인스턴스는 비어있는 EnumSet으로 셋팅
        
        public T addTopping(Topping topping) {//addToping 메서드를 사용하여 토핑을 추가할 수 있음
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }
 
        abstract Pizza build();//여기서 실제 타입은 Pizza가 아님. Pizza는 abstract라서 new로 생성 불가
        					   //Pizza의 하위 타입을 여기서 만들게 됨
        					   //Convariant 리턴 타입을 위한 준비 작업(Convariant 리턴 타입: 메서드가 오버라이딩될 때 더 좁은 타입(서브클래스)으로 교체할 수 있다는 것)

        // 하위 클래스는 이 메서드를 재정의(overriding)하여
        // "this"를 반환하도록 해야 한다.
        protected abstract T self();
	} 
	
	Pizza(Builder<?> builder){
		 toppings = builder.toppings;//Pizaa가 가지고 있는 토핑을 빌더가 가지고 있는 토핑으로 바꿔줌
	}
}
```
```java
package item02;

import java.util.Objects;

// 코드 2-5 뉴욕 피자 - 계층적 빌더를 활용한 하위 클래스 (20쪽)
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {//피자의 빌더를 상속하여 만든 하위 클래스
        private final Size size;

        public Builder(Size size) {//필수값으로 사이즈를 받아오게 함
            this.size = Objects.requireNonNull(size);//requireNonNull: null 인지 아닌지 체크를 한 후 셋팅해줌
        }

        @Override public NyPizza build() {//리턴타입이 뉴욕 피자임. build 호출하는 클라이언트가 타입 캐스팅을 할 필요 없이 뉴욕피자로 바로 받을 수 있음
            return new NyPizza(this);//this라는 Builder 자체를 넘겨줌
        }

        @Override protected Builder self() { return this; }
    }

    private NyPizza(Builder builder){//매개변수 Builder는 10~22라인의 Builder임.
        super(builder);//Pizza 클래스의 'Pizza(Builder<?> builder)'가 호출되어 토핑도 셋팅이됨
        size = builder.size;//Builder에서 받은 size를 NyPizza에 셋팅할 수 있음
    }

    @Override public String toString() {
        return toppings + "로 토핑한 뉴욕 피자";
    }
}
```
```java
package item02;

//코드 2-6 칼초네 피자 - 계층적 빌더를 활용한 하위 클래스 (20~21쪽)
public class Calzone extends Pizza {
	private final boolean sauceInside;

	public static class Builder extends Pizza.Builder<Builder> {
		private boolean sauceInside = false; // 기본값

		public Builder sauceInside() {
			sauceInside = true;
			return this;//Builder 객체를 넘김
		}

		@Override
		public Calzone build() {//빌드에서는 칼조네를 리턴
			return new Calzone(this);//Builder 자기 자신을 넘김
		}

		@Override
		protected Builder self() {
			return this;
		}
	}

	private Calzone(Builder builder) {//build에서 넘긴 Builder를 받음
		super(builder);
		sauceInside = builder.sauceInside;//Builder에서 만들어진 sauceInside 플래그를 Calzone 클래스의 소스인사이드에 셋팅할 수 있음 
	}

	@Override
	public String toString() {
		return String.format("%s로 토핑한 칼초네 피자 (소스는 %s에)", toppings, sauceInside ? "안" : "바깥");
	}
}
```

### 참고
- [[이팩티브 자바] #1 생성자 대신 static 팩토리 메소드를 고려해 볼 것](https://www.youtube.com/watch?v=X7RXP6EI-5E&t=1173s)
- [[이팩티브 자바] #2 생성자 매개변수가 너무 많아? 빌더 패턴을 써 봐](https://www.youtube.com/watch?v=OwkXMxCqWHM&t=4s)
