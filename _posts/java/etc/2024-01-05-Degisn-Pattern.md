---
title: '디자인 패턴 정리'
layout: post
categories: java
tags: 기타
comments: true
---

디자인 패턴은 무수히 많다 그 중 대표적인 GoF 디자인 패턴에 대해 알아보자

## 싱글톤 패턴 (SingleTon Pattern)
* 객체를 하나만 만들어 사용하는 패턴
* Socket Connection, DB JDBC Connection, Spring Bean 등 사용

```java
public class Singleton {
    private Singleton(){}

    private static final Singleton singleton = new Singleton();
    
    public static Singleton getInstance() {
        return singleton;
    }
}
```

위와 같이 구현해도 리플렉션으로 private 생성자에 접근 하여 객체 생성시 여러 개의 인스턴스를 생성할 수 있다.   
enum으로 구현하면 완벽한 싱글톤을 보장 받을 수 있으니 구현 방식은 따로 찾아보길 바란다.

## 플라이 웨이트 패턴 (FlyWeight Pattern)
* 인스턴스를 가능한 한 공유해서 사용함으로써 메모리를 절약하는 패턴
* 중복 생성될 가능성이 높거나, 자원 생성 비용은 큰데 사용 빈도가 낮은 경우 적용하면 좋음

```java
public class Flyweight {
    private static Map<String, Subject> map = new HashMap<>();

    public Subject getSubject(Subject subject) {
        String key = subject.name + subject.age;
        if (map.containsKey(key)) {
            return map.get(key);
        }
        
        map.put(key, subject);
        return subject;
    }
}

public class Subject {
    public String name;
    public int age;
    
    public Subject(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```
중복되는 공통된 객체를 생성 할 일이 있다면 자원을 눈에 띄게 아낄수 있을 것이다.

## 빌더 패턴 (Builder Pattern)

기본적으로 멤버 변수가 많아질수록 생성자를 이용하여 생성하기가 힘들어진다.  
순서를 일치시키기도 힘들뿐더러 (물론 ide의 도움이 있긴 하지만 ~ ... 코드를 읽을때 불편함이 있다.)   
기본 생성자로 생성시켜 setter로 일일이 넣어주는 것도 가독성 측면에서 어찌보면 떨어질 수도 있을 것이다.

### 장점
* 가독성/유지보수 향상
* 불변성 확보

```java
class Solution {
    private int a;
    private int b;
    private int c;

    public Solution(int a, int b, int c) {
        this.a = a;
        this.b = b;
        this.c = c;
    }

    public static SolutionBuilder builder() {
        return new SolutionBuilder();
    }
    public static class SolutionBuilder {
        private int a;
        private int b;
        private int c;

        public SolutionBuilder a(int a) {
            this.a = a;
            return this;
        }

        public SolutionBuilder b(int b) {
            this.b = b;
            return this;
        }

        public SolutionBuilder c(int c) {
            this.c = c;
            return this;
        }

        public Solution build() {
            return new Solution(a, b, c);
        }
    }
}
```

사실 대부분 lombok이 지원해주는 @Builder를 사용하여 빌더를 직접 구현할일이 없을것이다.  
하지만 내부 코드가 어떻게 구현이 되는지 이번기회에 한번 접해봤으면 좋겠다 !


## 어댑터 패턴(Adapter Pattern)

* 호출당하는 쪽의 메서드를 호출하는 쪽의 코드에 대응하도록 중간에 변환기(어댑터)를 통해 호출하는 패턴
* 호환성이 없는 기존 클래스의 인터페이스를 변환해서 재사용하기 위해 필요
* 서로 다른 두 인터페이스(클래스) 사이에 통신이 가능하게 하는 것

보통 아래와 같은 구조로 이해하면 된다. (간단한 예시)
```java
public class AdaptorSocket implements Electornic110V {

    private Electornic220V electornic220V;

    public AdaptorSocket(Electornic220V electornic220V) {
        this.electornic220V = electornic220V;
    }

    @Override
    public void powerOn() {
        electornic220V.connect();
    }
}
```   

```java
// APlayer interface
public interface APlayer {
    void play(String fileName);

    void stop();
}

// APlayer 구현체
public class APlayerImpl implements APlayer{

    @Override
    public void play(String fileName) {
        System.out.println("A " + fileName);
    }

    @Override
    public void stop() {

    }
}


// BPlayer interface
public interface BPlayer {
    void playFile(String fileName);

    void stopFile();
}


// BPlayer 구현체
public class BPlayerImpl implements BPlayer{

    @Override
    public void playFile(String fileName) {
        System.out.println("B " + fileName);
    }

    @Override
    public void stopFile() {

    }
}


// 어댑터 BPlayer를 APlayer로
public class BToAAdapter implements APlayer{
    private BPlayer media;

    public BToAAdapter(BPlayer media) {
        this.media = media;
    }

    @Override
    public void play(String fileName) {
        System.out.print("Using Adapter : ");
        media.playFile(fileName);
    }

    @Override
    public void stop() {
    }

    // 변환 가능 ~
    public static void main(String[] args) {
        APlayer player1 = new APlayerImpl();
        player1.play("aaa.mp3");

        BPlayer player2 = new BPlayerImpl();
        player2.playFile("bbb.mp3");

        player1 = new BToAAdapter(new BPlayerImpl());
        player1.play("ccc.mp3");
    }
}

```
위와 같이 adapter를 이용하여 APlayer의 기존 코드 변화없이 다른 객체와 연결시킬 수 있다.


## 파사드 패턴(Facade Pattern)
* 여러 객체와 실제 사용하는 객체 사이에 복잡한 의존관계가 있을때 사용
* 중간에 facade 객체를 두고 여기서 제공하는 기능을 사용하는 방식(facede가 여러 객체를 매핑해줌)

```java
public class Computer {
    private boolean turnedOn;

    public void turnOn() {
        turnedOn = true;
        System.out.println("computer on");
    }
    public void turnOff() {
        turnedOn = false;
        System.out.println("computer off");
    }
}


public class Light {
    private boolean turnedOn;

    public void turnOn() {
        turnedOn = true;
        System.out.println("light on");
    }
    public void turnOff() {
        turnedOn = false;
        System.out.println("light off");
    }
}
```
위와 같은 2개의 객체가 있다고 하면 매번 외출을 할 때마다
```java
light.turnOff();  
computer.turnOff();
```

이런식으로 두 개의 객체를 다뤄 줘야 한다.   
여기에 facade 패턴을 적용해보자 !

```java
public class Home {
    private Computer computer;
    private Light light;
    
    public Home(Computer computer, Light light) {
        this.computer = computer;
        this.light = light;
    }
    
    public void out() {
        computer.turnOff();
        light.turnOff();
    }
    
    public void in() {
        computer.turnOn();
        light.turnOn();
    }
}
```

이런식으로 객체를 묶어 파사드 패턴을 적용할 수 있다.


## 데코레이터 패턴(Decorator Pattern)
* 주어진 상황 및 용도에 따라 어떤 객체에 책임(기능)을 동적으로 추가하는 패턴

```java
// 커피를 제조할때는 여러가지 재료를 추가해야 한다. Component 인터페이스에서는 재료들을 추가해주는 함수를 구현
public interface Component {
    String add(); //재료 추가
}


// BaseComponent에서는 Component를 상속받아 커피의 기본 재료가 되는 에스프레소를 넣는것으로 정의
public class BaseComponent implements Component {
    @Override
    public String add() {
        return "에스프레소";
    }
}

// Decorator는 커피의 재료들의 근간이 되는 추상클래스. 재료들은 이 Decorator를 상속받아 재료를 추가한다
public abstract class Decorator implements Component {
    private Component coffeeComponent;

    public Decorator(Component coffeeComponent) {
        this.coffeeComponent = coffeeComponent;
    }

    public String add() {
        return coffeeComponent.add();
    }
}

//물을 추가해주는 클래스
public class WaterDecorator extends Decorator {
    public WaterDecorator(Component coffeeComponent) {
        super(coffeeComponent);
    }

    @Override
    public String add() {
        return super.add() + " + 물";
    }
}


// 우유를 추가해주는 클래스
public class MilkDecorator extends Decorator {
    public MilkDecorator(Component coffeeComponent) {
        super(coffeeComponent);
    }

    @Override
    public String add() {
        return super.add() + " + 우유";
    }
}


// 호출
public class Main {

    public static void main(String[] args) {
        Component espresso = new BaseComponent();
        System.out.println("에스프레소 : " + espresso.add());

        Component americano = new WaterDecorator(new BaseComponent());
        System.out.println("아메리카노 : " + americano.add());

        Component latte = new MilkDecorator(new WaterDecorator(new BaseComponent()));
        System.out.println("라떼 : " + latte.add());
    }
}


/**
 * ----------- 결과 ------------
 * 에스프레소 : 에스프레소
 * 아메리카노 : 에스프레소 + 물
 * 라떼 : 에스프레소 + 물 + 우유
 */

```
위 예제와 같이 동적으로 기능을 추가할 수 있다.

## 브릿지 패턴(Bridge Pattern)
* 기능의 계층과 구현의 계층을 연결시키는 패턴
* 추상화(abstraction)를 구현으로부터 분리하여 각각 독립적으로 변화할 수 있도록 하는 패턴


```java
public interface IRobot {

    void powerOn();
    void powerOff();

}

public class RobotModel1 implements IRobot {
    @Override
    public void powerOn() {
        System.out.println("type1 : power On");
    }

    @Override
    public void powerOff() {
        System.out.println("type1 : power Off");
    }
}

public class RobotModel2 implements IRobot {
    @Override
    public void powerOn() {
        System.out.println("type2 : power On");
    }

    @Override
    public void powerOff() {
        System.out.println("type2 : power Off");
    }
}


// 해당 객체로 로봇들 손쉽게 제어 가능 기능을 추가하여도 여기에만 추가하면됨
public class IAction {
    private IRobot robot;

    public IAction(IRobot robot) {
        this.robot = robot;
    }

    void powerOn() {
        robot.powerOn();
    }

    void powerOff() {
        robot.powerOff();
    }

    void cook() {
        System.out.println("요리 시작 !!");
    }

    void laundry() {
        System.out.println("빨래 시작 !!");
    }
}

    public static void main(String[] args) {
        IAction action = new IAction(new RobotModel1());
        action.powerOn();
        action.cook();
        action.laundry();
        action.powerOff();

        action = new IAction(new RobotModel2());
        action.powerOn();
        action.cook();
        action.laundry();
        action.powerOff();

    }
```

## 전략 패턴 (Strategy Pattern)
* 런타임 시점에 알고리즘 전략을 선택하여 객체 동작을 실시간으로 바꾸는 행위 패턴
* 동작들을 미리 전략으로 정의하고 손쉽게 전략을 교체할 수 있는 경우 적합하다.


```java
// 전략 - 추상화된 알고리즘
interface Weapon {
    void offensive();
}

class Sword implements Weapon {
    @Override
    public void offensive() {
        System.out.println("칼을 휘두르다");
    }
}

class Shield implements Weapon {
    @Override
    public void offensive() {
        System.out.println("방패로 밀친다");
    }
}

class CrossBow implements Weapon {
    @Override
    public void offensive() {
        System.out.println("석궁을 발사하다");
    }
}


// 컨텍스트 - 전략을 등록하고 실행
class TakeWeaponStrategy {
    Weapon wp;

    void setWeapon(Weapon wp) {
        this.wp = wp;
    }

    void attack() {
        wp.offensive();
    }
}


// 클라이언트 - 전략 제공/설정
class User {
    public static void main(String[] args) {
        // 플레이어 손에 무기 착용 전략을 설정
        TakeWeaponStrategy hand = new TakeWeaponStrategy();

        // 플레이어가 검을 들도록 전략 설정
        hand.setWeapon(new Sword());
        hand.attack(); // "칼을 휘두르다"

        // 플레이어가 방패를 들도록 전략 변경
        hand.setWeapon(new Shield());
        hand.attack(); // "방패로 밀친다"

        // 플레이어가 석궁을 들도록 전략 변경
        hand.setWeapon(new Crossbow());
        hand.attack(); // "석궁을 발사하다"
    }
}
```

어떤 예제보다 위 코드의 예제가 가장 쉽게 이해가 되는 거 같다.   
이런식으로 유연하게 객체지향의 장점을 살릴 수 있다는 장점이 있다.    
`OCP, DIP, 합성, 다형성, 캡슐화 등` 의 총 집합이다.


## 참고
https://inpa.tistory.com/entry/GOF-💠-전략Strategy-패턴-제대로-배워보자   

