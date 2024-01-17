---
title: ' Item 1 :: 생성자 대신 정적 팩터리 메서드를 고려하라'
layout: post
categories: java
tags: Effective-Java
comments: false
---

### 정적 팩터리 메서드의 장점
#### 1. 이름을 가질 수 있다.
정적 팩터리 메서드는 생성자보다 읽기 편하다.


하나의 시그니처로는 생성자를 하나만 만들 수 있다.
```java
    public Student(String studentID, int grade);     //가 생성된 상태에서
    public Student(String studentID, int GPA);  //이렇게 생성이 불가능. 컴파일 에러 발생!
```

물론 입력 매개변수들의 순서를 다르게 한 생성자를 새로 추가하는 식으로 아래 소스 같이 제한을 피해볼 수도 있으나 아주 좋지 않은 발상이다.
```java
    public Student(String studentID, int grade);     //가 생성된 상태에서
    public Student(int GPA, String studentID);  //이렇게 생성이 가능하며 에러 발생하지 않긴 하지만... 매우 비추천
```

하지만 정적 팩터리 메서드는 이러한 제약이 없다. 한 클래스에 시그니처가 같은 생성자가 여러개 필요할 것 같으면, 생성자를 정적 팩터리 메서드로 바꾼 다음 각각의 차이를 잘 드러내는 이름을 지어주면 된다.
```java
class Student{
    Student(String studentID, int grade, int GPA) {}
   
    private String studentID;
    private int grade;
    private double GPA;

    public static Student StudentWithIDandGrade(String studentID, int grade){
        return new Student(studentID, grade, 0);
    }

    public static Student StudentWithIDAndGPA(String studentID, int GPA){
        return new Student(studentID, 0, GPA);
    }
}
```

#### 2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
반드시 새로운 객체를 만들 필요가 없다. 플라이웨이트 패턴도 이와 비슷한 기법이다.

```java
package item1;

public class Foo {
    String name;
    String address;
   
    public Foo(){
    }
   
    //생성자
    public Foo(String name) {
        this.name = name;
    }
   
    //생성자 - 위의 생성자랑 시그니처가 같으므로 생성 불가
//  public Foo(String address) {
//      this.address = address;
//  }
   
    // static 팩토리 메서드
    public static Foo withName(String name){
        return new Foo(name);
    }
   
    public static Foo withAddress(String address) {
        Foo foo = new Foo(address);
        return foo;
    }
   
    private static final Foo GOOD_NIGHT = new Foo();
   
    public static Foo getFoo() {
        return GOOD_NIGHT;//객체를 아예 새로 생성하지 않고 미리 만들어놓은 것을 가져옴. 특히 생성비용이 큰 경우 성능을 상당히 끌어올려줌.
    }
   
    public static void main(String[] args){
        //장점 1: foo 코드보다는 foo2 의미가 더 읽기 편하다. Ara가 무슨 의미인지 알 수 있으므로.
        Foo foo = new Foo("Ara");
        Foo foo1 = Foo.withName("Ara");
       
        //장점 2: 반드시 새로운 객체를 만들 필요가 없다. 플라이웨이트 패턴도 이와 비슷한 기법임.
        Foo foo2 = Foo.getFoo();
    }
}
```
반복되는 요청에 같은 객체를 반환하는 식으로 정적 팩터리 방식의 클래스는 언제 어느 인스턴스를 살아있게 할지를 철저히 통제할 수 있다. 이를 **인스턴트 통제 클래스**라 한다. 즉, 값이 있으면 새로 만들지 않고 기존 객체를 재사용하는 클래스를 말한다.  


#### 3. 반환 타입의 하위 객체를 반환할 수 있는 능력이 있다.
반환할 객체의 클래스를 자유롭게 선택할 수 있게하는 유연성이 있다. 이는 인터페이스를 정적 팩터리 메서드의 반환타입으로 사용하는 `인터페이스 기반 프레임워크`의 핵심 기술이다.  

1. **자바 8 이전**  
인터페이스에 정적 메서드를 선언할 수 없었다. 그래서 A라는 인터페이스가 있고 그 안에 이름이 "Type"인 A 인터페이스를 반환하는 정적 메서드가 필요하다면, "Types"라는 인스턴스화 불가 동반 클래스를 만들어 그 안에 A객체를 반환하는 Type이라는 이름의 정적 메소드를 정의하는 것이 관례였다.  
2. **자바 8 이후**  
인터페이스도 정적 팩터리 메서드를 가질 수 있다.
    - 인스턴스화 불가 동반 클래스를 쓰지 않고, 동반 클래스 내 public 정적 멤버를 인터페이스 자체에 둘 수 있게됨.
    - But, 자바 8에서도 인터페이스에는 public 정적 멤버만 허용함. 그러므로 정적 메서드를 구현하기 위한 코드 중 많은 부분은 여전히 별도의 `package-private` 클래스에 두어야 가능  

```java
package item1;

public interface FooInterface {
   
    //자바 8부터는 인터페이스에도 정적 팩터리 메서드 추가 가능. private static은 자바 9부터 가능
    public static Foo getFoo() {
        return new Foo();
    }
}
```  
그럼 private Static은 어떨 때 사용하는가?  
보통 아래처럼 두개의 메소드(doSomething과 doSomethingTomorrow)에서 공통으로 겹치는 소스가 있는 경우 private 메소드를 생성하여 공통부분을 묶어 메소드 하나로 만든다. 예제 소스에서는 `Effecttive를공부하고잔다` 부분이다.  
이때 public static 메소드에서 `Effecttive를공부하고잔다`를 호출할려는 경우 static 타입이 아니면 호출이 불가능하다. 왜냐? static 메서드에서는 static만 호출 및 사용이 가능하기 때문이다. 따라서 private static이 필요한 이유는 public static이 필요한 이유와 같다.

```java
public class WhyNeedPrivateStatic {
    public static void doSomething() {//static 메서드에서는 static만 호출 및 사용이 가능하다.
        //Todo 근무를 한다
        //Todo 네이버 블로그 천지양꼬치를 포스팅한다
        //Todo 설거지를 한다
        Effecttive를공부하고잔다();
    }
   
    public void doSomethingTomorrow() {
        //강남역 토즈에가서 스터디에간다
        Effecttive를공부하고잔다();
    }
   
    private static void Effecttive를공부하고잔다(){//그러므로 private static이 필요한 이유는 pulbic static이 필요한 이유와 같음
        //Todo Effective Java Item 1을 공부한다
        //Todo 잔다
    }
}
```

3. **자바9**  
    private 정적 메서드까지 허용. 하지만 정적 필드와 정적 멤버 클래스는 여전히 private이어야 함

    자바 컬렉션 프레임 워크는 핵심 인터페이스들에 수정 불가나 동기화 등의 기능을 덧붙인 총 45개의 유틸리티 구현체를 제공한다. 이 구현체의 대부분을 **단 햐냐의 인스턴스화 불가 클래스인 `java.util.Collections`에서 정적 팩터리 메서드를 통해 얻도록** 했다.  
    프로그래머는 명시한 인터페이스대로 동작하는 객체를 얻을 것임을 알기에 굳이 별도 문서를 찾아가며 실제 구현 클래스가 무엇인지 알아보지 않아도 되며, 정적 팩터리 메서드를 사용하는 클라이언트는 얻은 객체를 인터페이스 만으로 다루게 된다. 이는 일반적으로 좋은 솝관이다.

#### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다. 심지어 다음 릴리즈 에서는 또 다른 클래스의 객체 반환이 가능하다.  

예를 들어 아래 소스코드처럼 Foo 객체라고해서 Foo를 무조건 리턴하지 않고 flag에 따라 다른 객체를 반환하도록 할 수 있다. true인 경우 Foo 객체를 반환하고 fale인 경우 Foo의 하위 클래스인 Barfoo 객체를 반환한다.  

```java
package temp;

import java.util.EnumSet;

public class Foo {
    String name;
    String address;
   
    public static Foo getFoo(boolean flag) {
        return flag ? new Foo(): new Barfoo();
    }
   
    public static void main(String[] args){
        //장점 4: Foo객체라고해서 Foo를 무조건 리턴할 필요는 없다. flag에 따라 객체를 반환하게됨.
        Foo foo3 = Foo.getFoo(false);
        EnumSet<Color> colors = EnumSet.allOf(Color.class);
        EnumSet<Color> colors2 = EnumSet.of(Color.BLUE, Color.WHITE);//실제 인스턴스는 enum의 갯수에 따라 달라진다. 몇 개 이하이면 ReugularEnumSet으로 나오고, 그 이상이면 JumboEnumSet으로 리턴하는 객체가 달라진다.
    }
   
    static class Barfoo extends Foo{
    }
   
    enum Color {
        RED, BLUE, WHITE
    }
}
```
실제로 EnumSet 클래스 선언 시 실제 인스턴스는 enum 원소의 갯수에 따라 달라진다. enum 원소가 64개 이하이면 ReugularEnumSet으로 나오고, 그 이상이면 JumboEnumSet 인스턴스가 반환된다. 하지만 클라이언트는 팩터리가 건네주는 객체가 어느 클래스의 인스턴스인지 알 수도 없고 알 필요도 없다.

#### 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
대표적인 서비스 제공자 프레임워크인 JDBC(Java Database Connectivity)를 만드는 근간은 이런 유연함이다.  

![JDBC](/assets\img/JDBC.gif)
즉, `서비스 접근 API인 DriverManager.getConnection`를 작성하는 시점에는 `반환할 서비스 인터페이스인 Connection`의 하위 클래스가 아직 존재하지 않아도 동작이 가능하다
1. **제공자(Provider)**: 서비스의 구현체
2. **프레임워크**: 구현체를 클라이언트에 제공하는 역할 통제. 클라이언트와 구현체 분리.  
    **3개의 핵심 컴포넌트**  
    1) 서비스 인터페이스 ex) Connection  
    2) 제공자 등록 API ex) DriverManager.registerDriver  
    3) 서비스 접근 API ex) DriverManager.getConnection  
        : 공급자가 제공하는 것보다 더 풍부한 서비스 인터페이스를 클라이언트에 반환 가능 → **브리지 패턴**

    **종종 사용되는 컴포넌트**  
    4) 서비스 제공자 인터페이스 ex) Driver  
        : 서비스 인터페이스의 인스턴스를 생성하는 팩터리 객체를 설명해줌  
          서비스 제공자 인터페이스가 없다면 각 구현체를 인스턴스로 만들 때 **리플렉션**을 사용해야 함
3. **Client**

자바 6부터는 `java.util.ServiceLoader`라는 범용 서비스 제공자 프레임워크를 제공하지만, JDBC가 그 보다 이전에 만들어졌기 때문에 JDBC는 ServiceLoader를 사용하진 않는다.

### 정적 팩터리 메서드의 단점
#### 1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.  
컬렉션 프레임워크의 유틸리티 구현 클래스들은 **상속할 수 없다**. 하지만 상속보다 컴포지션을 사용하도록 유도하고 불변 타입으로 만들려면 이 제약을 지켜야하기때문에 오히려 장점이 될 수 있다.

#### 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
생성자는 Javadoc 상단에 모아서 보여주지만 static 팩토리 메소드는 API 문서에서 특별히 다뤄주지 않는다. 따라서 클래스나 인터페이스 문서 상단에 static 팩토리 메소드 방식 클래스를 인스턴스화 하는 방법 등의 내용을 담아 주석으로 표시하여 제공하는 것이 좋겠다.

### 결론
정적 팩터리 메서드와 pulbic 생성자는 각자의 쓰임새가 있으니 무조건 정적 팩터리 메서드가 옳다고 할 순 없다. 각자의 상대적인 장단점이 있을 뿐! 그럼에도 불구하고 대게 정적 팩터리를 사용하는 것이 유리한 경우가 더 많으므로 무조건 pulbic 생성자만을 제공하던 습관이 있다면 고치는게 좋다 :)



**인스턴트 클래스를 통제하는 이유**  
1. **싱글턴(Singleton)**으로 만들 수 있다.
    - **싱글턴**: 객체의 인스턴스가 오직 1개만 생성되는 패턴을 의미  
    싱글턴이 사용되는 <u>예시</u>로는 어플의 다크모드가 있다. 사용자가 어플의 설정에서 다크모드를 설정해놓으면 다른 페이지로 이동하더라도 이 다크모드가 그대로 유지되어야한다. 즉, 어떤 페이지에 있던 이 세팅을 관리하는 객체는 반드시 같은 것을 사용해야 한다. 그럼 이 객체의 인스턴스가 하나만 생성되어 있어야 하는 것이 좋은데 이럴 때 싱글턴 패턴을 사용한다.  
    아래는 기본적인 싱글턴 코드이며 멀티 쓰레드 환경 등에서 오류가 발생할 소지가 있다. 그러므로 여기서는 싱글턴을 이해하는 용도로 소스코드를 참조하고, 실제 사용 시에는 싱글턴을 보다 안전하게 사용할 수 있는 방법들을 검색 후 사용하는 것을 권장한다.

        - **Settings.java**

        ```java
        package singleton;

        public class Settings {

            private Settings() {};//private: Settings 클래스에서만 사용 가능. 외부 클래스에서 사용하려고 하면 에러 발생.
            
            /* static으로 선언된 것들(정적공간) : 객체가 얼마나 생성되든 메모리의 지정된 공간에 딱 하나만 존재한다. 
            * 								컴파일 할 때 부터 이 요소가 차지할 메모리 용량을 이미 알 수 있도록 딱 정해져 있기 때문에
            * 								동적요소들과 대비되는 개념으로 static, 정적이라고 불림
            */
            private static Settings settings = null;//★★★ 자기 자신인 Settings 타입의 객체를 static으로 생성. null로 초기화 ★★★

            // 정적 팩토리 메소드
            public static Settings getSettings() {
                //1. 다른곳에서 getSettings를 실행하기 전이라면
                if(settings == null){
                    settings = new Settings();//Settings 객체를 선언해서 settings 변수에 넣어주고 아래에서 반환
                }
                //2. settings가 이미 만들어진 상태라면 settings를 그대로 반환
                return settings;
            }
            
            private static boolean darkMode = false;
            private static int fontSize = 13;
            
            public static boolean getDarkMode(){//public: 누구든지 이 클래스의 메소드를 호출 가능
                return darkMode;
            }

            public static int getFontSize() {
                return fontSize;
            }
            
            /* static이 아닌 것들(동적공간) : 객체가 생성될 때마다 메모리의 공간을 새로 차지*/
            public void setDarkMode(boolean _darkMode) {
                darkMode = _darkMode;
            }

            public void setFontsize(int _fontSize) {
                fontSize = _fontSize;
            }

        }
        ```
        - **FirstPage.java**

        ```java
        package singleton;

        public class FirstPage {
            //Settings객체의 static 메소드는 이미 메모리의 정적 공간에 자리를 차지하고 있는 상태이기 때문에
            //해당 객체를 new로 생성하지 않고도 클래스에서 바로 불러낼 수 있음
            private Settings settings = Settings.getSettings();
            
            public void setAndPrintSetting(){
                settings.setDarkMode(true); //정적 settings 객체에 값을 넣음
                settings.setFontsize(15);
                
                System.out.println(settings.getDarkMode()+" "+settings.getFontSize());//그 내용들을 출력함
            }
        }

        ```
        - **SecondPage.java**

        ```java
        package singleton;

        public class SecondPage {
            //Settings객체의 static 메소드는 이미 메모리의 정적 공간에 자리를 차지하고 있는 상태이기 때문에
            //해당 객체를 new로 생성하지 않고도 클래스에서 바로 불러낼 수 있음
            private Settings settings = Settings.getSettings();
            
            public void setAndPrintSetting(){
                settings.setDarkMode(true); //정적 settings 객체에 값을 넣음
                settings.setFontsize(15);
                
                System.out.println(settings.getDarkMode()+" "+settings.getFontSize());//그 내용들을 출력함
            }
        }

        ```
        - **MyProgram.java**

        ```java
        package singleton;

        public class MyProgram {

            public static void main(String[] args) {
                new FirstPage().setAndPrintSetting();
                new SecondPage().printSettings();
            }

        }
        ```
    
    근데... 그냥 정적변수를 쓰지 왜 싱글턴을 쓸까? 그건 바로 interface의 사용, lazy loading 등 싱글턴으로 할 수 있는 것들이 더 많기 때문이다!
2. **인스턴스화 불가**로 만들 수 있다.  
   ex) 위 소스코드의 FirstPage 클래스에서 `private Settings settings = new Settings();`로 인스턴스화 불가
3. 불변 값 클래스에서 **동치인 인스턴트가 하나임을 보장**할 수 있다. (`a==b`일 때, `a.equlals(b)`가 성립)  
   인스턴트 통제는 플라이웨이트 패턴의 근간이 되며, 열거타입은 인스턴스가 하나만 만들어짐을 보장한다.

##### 플라이웨이트 패턴(Flyweight pattern) 
어떤 클래스의 인스턴스 한 개만 가지고 여러 개의 "가상 인스턴스"를 제공하고 싶을 때 사용하는 패턴.  
즉, 인스턴스를 가능한 대로 공유시켜 쓸데없이 new 연산자를 통한 메모리 낭비를 줄이는 방식.  
디자인 패턴 GoF 중 `공유(Sharing)을 통하여 대량의 객체를 효과적으로 지원하는 방법`

대표적인 예 - Java의 String Pool
- 동일한 문자열에 대해서는 다시 사용될 때에 새로운 메모리를 할당하는 것이 아니라 String Pool에 있는지 검사해서 가져옴.
- String을 생성할 때 new를 통해 String을 생성하면 Heap영역에 존재하게 되고 리터럴을 이용할 경우 String constant pool이라는 영역에 존재하게 된다.

1.New 연산자를 이용한 방식

```java
public static void main(String[] args) {
    String a = new String("A");
    String b = new String("A");

    System.out.println(a == b);      //주소값 비교
    System.out.println(a.equals(b)); //주소 안에 들어 있는 값 비교
}

false
true
```
**<span style="color:red">new 연산자</span>**를 사용하여 새로운 String 객체를 생성하면 JVM에서 **<span style="color:red">Heap영역</span>**에 String 객체를 생성하게 된다.
그렇기 때문에 a를 Heap 메모리에 생성하고, b를 또 Heap 메모리에 생성하고 이 둘은 다른 객체가 된다.

2.리터럴을 이용한 방식

```java
public static void main(String[] args) {
    String a = "A";
    String b = "A";

    System.out.println(a == b);      //주소값 비교
    System.out.println(a.equals(b)); //주소 안에 들어 있는 값 비교

true
true
```
new 연산자가 아닌 **<span style="color:red">리터럴("")</span>로 String 객체를 생성하면 JVM은 우선 <span style="color:red">String Contstrant Pool 영역</span>을 방문하게 됩니다. 거기서 같은 값을 가진 String 객체를 찾으면 그 객체의 주소 값을 반환하여 참조하게 됩니다. 찾지 못하면 String Constrant Pool에 해당 값을 가진 String 객체를 생성하고 그 주소 값을 반환하게 됩니다.** String constrant Pool 영역은 Heap 영역 내부에서 String 객체를 위해 별도로 관리하는 저장소입니다.  

리터럴로 a를 생성할 때 String Constrant Pool에 "A"라는 값을 가진 String 객체가 생성되었고, b 또한 리터럴로 생성되었기 때문에 String Constrant Pool에 방문해보았더니 a를 생성할 때 만들어준 "A"라는 값을 가진 String 객체를 발견한 것 입니다.  
결과적으로 a와 b는 같은 객체를 참조하고 있는 것 입니다.

구글링 중 너무 이미지로 잘 정리해주신 [by hyeraan님의 글](https://hyeran-story.tistory.com/123)이 있어 공유합니다.

![constrantPool](/assets\img/constrantPool.PNG)

#### 3. 반환 타입의 하위 객체를 반환할 수 있는 능력이 있다.
반환할 객체의 클래스를 자유롭게 선택할 수 있게하는 유연성이 있다.

### 참고
- [[디자인패턴/Design Pattern] Flyweight Pattern/플라이웨이트 패턴](https://lee1535.tistory.com/106)
- [객체지향 디자인패턴 1](https://www.youtube.com/watch?v=lJES5TQTTWE&t=91s)
- [[Java] equals()과 == 차이점, String Contrant Pool(상수풀)](https://hyeran-story.tistory.com/123)
