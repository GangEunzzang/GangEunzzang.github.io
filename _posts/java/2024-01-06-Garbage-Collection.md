---
title: 가비지 컬렉션 정리
date: 2024-01-01 23:30:00 +09:00
categories: [java]
tags:
  [
    java, 
    자바, 
    가비지 컬렉션,
    GC,
    Garbage Collector
  ]
---

* * *

 
## Garbage Collection(GC) 이란?
* * *
* JVM(자바 가상 머신)의 `Heap 영역`에서 필요 없게 된 객체를 삭제해주는 프로세스
* 객체가 접근 불가능한 상태(Unreachable)가 되었을 때, 메모리가 누적되므로 이를 수거해줌

그럼 이런 의문이 들 수도 있다.  
`왜 메모리 관리를 해줘야 돼?..`
다음의 예시를 보자

```java
for (int i = 0; i < 100_000_000_000; i++ ) {
	Test test = new Test();
	test.run();
}
```

만약 위와 같은 코드가 있다고 했을 때 자바에서 실행하면 어떻게 될까?    
아마 실행 시간은 오래 소요 되지만 정상적으로 동작이 될 것이다.    

반대로, GC가 동작을 하지 않아, 메모리를 계속 점유하고 있다고 가정해보자.  
각 인스턴스의 크기를 1KB 라고 가정하겠다.  

1000억 * 1KB = 1000억 KB
`1000억 KB = 976.562.500MB = 976.562.5GB = 약 953.67TB`

어마어마한 수치이다.. 메모리는 고사하고 하드웨어도 버티기 힘든 크기이다.
이래서 한정된 메모리를 효율적으로 사용하기 위해 GC가 필요한 것이다.  
<br>
GC 단점   
* 메모리를 해제 시점을 정확하게 알 수 없다.
* GC가 동작하는 동안 `Stop-The-World` 발생


<br><br>


## Garbage Collector (가비지 컬렉터)
* * * 
Java의 Garbage Collector는 동작 방식에 따라 매우 다양한 종류가 있으나 공통적으로 크게 다음 2가지 작업을 수행한다.
1. 힙(heap) 내의 객체 중에서 가비지(garbage)를 찾아낸다.
2. 찾아낸 가비지를 처리해서 힙의 메모리를 회수한다.

최초의 Java에서는 이들 GC 작업에 애플리케이션의 사용자 코드가 관여하지 않도록 구현되어 있었다. 그러나 위 2가지 작업에서 좀 더 다양한 방법으로 객체를 처리하려는 요구가 있었다. 이에 따라 **JDK 1.2부터는** java.lang.ref 패키지가 추가되어 제한적이나마 **사용자의 코드와 GC가 상호작용할 수 있게** 하고 있다.

Java의 java.lang.ref 패키지는 전형적인 객체 참조인 Strong reference 외에도 **Soft, weak, phantom 3가지의 새로운 참조 방식**을 각각의 Reference 클래스로 제공한다. 이 3가지 Reference 클래스를 어플리케이션에 사용하면 GC에 일정 부분 관여할 수 있고, LRU(Least Recently Used) 캐시 같이 특별한 작업을 하는 애플리케이션을 더 쉽게 작성할 수 있다. 이를 위해서는 GC에 대해서도 잘 이해해야하고, 이들 참조방식의 동작도 잘 이해해야 한다.
- **LRU 캐시**  
캐시는 빠르고 작은 단기기억 공간이며 Key-Value 관계를 가지고 있다. 캐시 오버플로우가 났을 때 데이터를 지워주는 규칙을 LRU라고한다. LRU는 Least Recently Used의 약어이며 가장 과거에 사용된 데이터를 삭제해주는 것 이다.

<br><br>

## GC와 Reachability
* * *
Java GC는 객체가 가비지인지 판별하기 위해서 **reachability**라는 개념을 사용한다. 어떤 객체에 유효한 참조가 있으면 `reachable`로 없으면 `unreachable`로 구별하고, `unreachable` 객체를 가비지로 간주해 GC를 수행한다.  

한 객체는 여러 다른 객체를 참조하고, 참조 된 다른 객체들도 마찬가지로 또 다른 객체들을 참조할 수 있으므로 객체들은 참조 사슬을 이루게 된다. 이런 상황에서 유효한 참조 여부를 파악하려면 항상 **유효한 최초의 참조**가 있어야 하는데 이를 객체 참조의 **root set**이라고 한다.  

<br><br>


## 전체적인 JVM의 구조
* * *
![jvm-structer.PNG](..%2F..%2Fassets%2Fimg%2Fjvm-structer.PNG)
### JVM에서 런타임 영역(runtime data area, JVM Memory)의 구조
- 런타임 영역은 JVM이 Java Bytecode를 실행하기 위해 사용하는 **메모리 영역**
- ![runtime-data-area.PNG](..%2F..%2Fassets%2Fimg%2Fruntime-data-area.PNG)
<center>그림 1 런타임 데이터 영역(Oracle HotSpot VM 기준)</center><br>

런타임 데이터 영역 크게 세 부분으로 나눌 수 있다. 위 그림에서 객체에 대한 참조는 화살표로 표시되어 있다.  
1. **스레드**가 차지하는 영역 - 각 스레드 마다 존재  
 - **Java Stack**    
   1) 스레드 마다 1개만 존재하고, 스택 프레임은 메서드가 호출될 때 마다 생성된다. 메서드 실행이 끝나면 스택 프레임은 pop되어 스택에서 제거된다.  
   2) Stack Frame은 Local Variables Array, Operand stack, Frame Data를 갖는다.  
   3) Frame Data는 Constant Pool에 대한 참조, 이전 스택프레임에 대한 정보, 현재 메서드가 속한 클래스/객체에 대한 참조 등의 정보를 갖는다.
 - **PC Register(Program Counter)**: 각 스레드는 어떤 메서드를 항상 실행하고 있고, 그 때 PC는 그 메서드 안에서 Byte Code 몇 번째 줄을 실행하고 있는지를 나타내는 역할  
 - **Native Stack**  
   1) 성능향상을 위해 자바의 ByteCode가 아닌 C/C++로 작성된 코드를 컴파일해서 사용하는 경우가 있다. 그때 사용되는 메서드를 Natvie (method) Stack이라고 한다.  
   2) **JNI(Java Native Interface)**  
      자바는 특정 운영체제에 종속되지 않도록, JVM이라는 가상 머신 위에서 실행되게 끔 만들어진 언어이다.  
      운영체제에 맞는 JVM이 각기 존재하기 때문에, Java 개발자는 하나의 Java 파일만 만들면 운영체제와 상관없이 원하는 결과물을 쉽게 얻을 수 있다.  

      하지만 단점도 있다. 그 중 하나의 단점은 운영체제의 모든 기능을 JVM이 담지 못한다는 것이다. 따라서 구현하고 싶은 몇몇 기능들은 Java 언어 자체로도 해결 안되는 경우가 존재한다.  
      JNI는 **Java 언어 자체로 해결이 안되는 경우 대처할 수 있는 방법 중 하나**이다. 운영체제의 고유기능(Native)을 Java로 해결하는 것이 아닌 운영체제가 구현된 언어(보통 C, C++)로 운영체제의 고유 기능을 만든다.
2. **힙**: 프로그램을 실행하면서 생성한 모든 객체의 인스턴스 저장
3. 클래스 정보가 차지하는 영역인 **메서드 영역**
 - 클래스 로더가 클래스 파일을 읽어오면, 클래스 정보를 파싱해서 메서드 영역에 저장.
 - ex) 변수, 메서드, 정적 변수, 바이트 코드는 어떤게 있는가 등을 저장  

<br><br>


## Heap 메모리 구조
* * *
### <span style="color:red">jdk 1.7</span> 버전과 그 이전
![heap-before-jdk-1point7.PNG](..%2F..%2Fassets%2Fimg%2Fheap-before-jdk-1point7.PNG)
 - Eden: 새로 생성한 대부분의 객체가 위치하는 곳
 - Survival 0, Survival 1: Eden 영역에서 GC가 한번 발생한 후 살아남은 객체들이 존재하는 곳
 - Old Memory: Young Generation(Eden + S0, S1)에 대한 GC가 반복되는 과정속에서 살아남은 객체가 살아 남는 곳. 특정 회수 이상 참조되어 Old 영역으로 가기 위한 Age를 달성하였을 때 이동하게 된다.
 - Perm: Class / Method의 Meta 정보, static 변수 / 상수들이 저장되는 곳

우리가 흔히 아는 GC도 위와 같은 가정을 두고 언급한 것들이 정말 많다. 하지만 지금도 그럴까?

### jdk 1.8 버전 이후
![heap-after-jdk-1point8.PNG](..%2F..%2Fassets%2Fimg%2Fheap-after-jdk-1point8.PNG)  
`Perm` 영역이 `Metaspace`로 바뀌었다. 어떠한 부분들이 더 바뀌었을까?

|구분      |Perm                                         |Metaspace|
|---------|---------------------------------------------|---------|
| 저장정보 | 클래스 meta / 메소드 meta / static 변수, 상수 | 클래스 meta / 메소드 meta|
| 관리 포인트| Heap 영역 튜닝 + Perm 영역 별도 | Native 영역 동적 조정
| GC| Full FC | Full FC
| 메모리 측면 | -XX: PermSize / -XX: MaxPermSize | -XX:MetaSpaceSize / -XX: MaxMetaSpaceSize

가장 중요한 핵심은 `Perm` 영역이 `Heap`이 아니라 `Native` 영역으로 바뀌었다는 것이다.  
**Native** 영역의 가장 큰 특징 중 하나는 JVM에 의해서 크기가 강제되지 않고, **프로세스가 이용할 수 있는 메모리 자원을 최대로 활용할 수 있다.**  
만일 메모리 leak이 Classloader을 동작하는 코드에 발생하는 것으로 의심된다면, 이는 최대 메모리를 설정하지 않았기 때문이다.  


### Young 영역 (Young Generation)
- 새롭게 생성된 객체가 할당(Allocation) 되는 영역
- 대부분의 객체가 금방 Unreachable 상태가 되기 때문에, 많은 객체가 Young 영역에서 사라진다.
- Young 영역에 대한 GC를 `Minor GC` 라고 부른다.

### Old 영역 (Old Generation)
- Young영역에서 Reachable 상태를 유지하여 살아남은 객체가 복사되는 영역
- Young 영역보다 크게 할당되며, 영역의 크기가 큰 만큼 가비지는 적게 발생한다.
- Old 영역에 대한 가비지 컬렉션(Garbage Collection)을 `Major GC` 또는 `Full GC` 라고 부른다.

추가적으로 더욱 효율적인 GC를 위해 Young 영역을 3가지 영역(`Eden, survivor0, survivor1`) 으로 나눈다.
![youngGeneration.png](..%2F..%2Fassets%2Fimg%2FyoungGeneration.png)

### Eden
- new를 통해 새로 생성된 객체가 위치
- 정기적인 GC후 살아남은 객체들은 `Survivor` 영역으로 보냄

### Survivor0 / Survivor1
- 최소 1번의 GC에서 살아남은 객체가 존재하는 영역
- Survivor 영역에는 특별한 규칙이 있는데, `Survivor0` or `survivor1` 둘중 하나를 꼭 비어있어야 한다.

### Minor GC 과정
![minorCG.png](..%2F..%2Fassets%2Fimg%2FminorCG.png)
1. 처음 생성된 객체는 Eden 영역에 생성
2. 객체가 계속 생성되어 Eden 영역이 임계치에 도달하면 `Minor GC` 발생
   - Eden 영역에서 사용되지 않는 객체의 메모리가 해제된다.
   - Eden 영역에서 살아남은 객체는 활성화된 Survivor 영역으로 이동
3. 1~2번 과정이 반복되다가 Survivor 영역이 가득차면 해당 영역에서 다른 Survivor 영역으로 이동
4. 이러한 과정을 반복하여 계속해서 살아남은 객체는 Old 영역으로 이동


### Major GC 과정
* Young영역에서 객체의 age가 임계값에 도달하면 Old영역으로 이동된다. 이를 `promotion` 이라고 지칭 
* Old 영역이 가득차면 FullCG 발생
* Young 영역과, Old 영역을 동시에 처리한다.

`Full GC`는 일반적으로 `Minor GC`보다 10대 이상의 시간이 소요된다.  
기본적으로 메모리가 크고, 처리해야 될 양이 많기 때문이다.  
따라서 가비기 컬렉션 성능 개선을 위해 많은 알고리즘이 존재한다.  

### GC 알고리즘 종류
- `Serial GC`: Heap의 앞부분부터 확인하여 살아있는 것들만 남기고(Sweep), 객체들이 연속되도록 Compaction 하는 작업. 기본적으로 Mark-Sweep-Compaction 알고리즘에 해당. Serial GC는 적은 메모리와 CPU 코어 개수가 적을 때 가장 올바름(싱글 쓰레드)
- `Parallel GC`: Serial GC의 멀티쓰레드 버전
- `Parallel Old GC`: Parallel GC와 다른 점은 `Mark-Summary-Compaction` 단계를 거쳐서 객체를 식별 Summary에 해당하는 작업이 GC를 수행한 영역에 대해서 살아있는 객체를 식별한다는 작업
- `Concurrent Mark & Sweep GC(CMS GC)`: 다른 GC와는 다르게 `Compaction`을 진행하지는 않는다.
    - Initial Mark: 클래스 로더에서 가장 가까운 객체 중 살아있는 객체만 찾는다.
    - Concurrent Mark: 위에서 살아있다고 확인한 객체에서 참조되고 있는 객체를 확인한다.
    - Remark: 위 단계에서 새로 추가되거나 참조가 끊긴 객체를 확인한다.
    - Concurrent Sweep: 쓰레기를 정리한다.
    - Java9 ++ deprecated, Java14 사용 중지
- `G1 GC`: 아래에서 자세히 설명

<br><br>


## G1GC
* * *

- Java 9+ 버전의 default CG
- 4GC 이상의 힙 메모리, Stop the World 시간이 0.5초 정도 필요한 상황에 사용 `(Heap이 너무 작을경우 미사용 권장)`
- 기존의 GC 와 물리적으로 개념이다르다.


G1GC는 앞서 언급했던 `Eden`, `Survivor`, `Old` 영역이 존재하지만,   
해당 영역은 고정된 크기가 아니며 전체 Heap 메모리 영역을 `Region` 이라는 특정한 크기로 나눈것이고,   
`Region`의 상태에 따라 그 `Region`의 역할(Eden, Survivor, Old)가 동적으로 변동한다.  
`Rgion`은 기본적으로 ( 전체 Heap 메모리 ) / 2048로 default 값이 지정되어 있다.

![G1GC Heap.PNG](..%2F..%2Fassets%2Fimg%2FG1GC%20Heap.PNG)

### G1 GC의 효율성
  * 이전 GC들 처럼 일일히 메모리를 탐색해 객체들을 제거하지 않는다.
  * 메모리가 많이 차있는 영역(region)을 인식하는 기능을 통해 `메모리가 많이 차있는 영역을 우선적으로 GC`
  * 즉, Heap Memory 전체를 탐색하는 것이 아닌 `region`을 나눠 탐색하고 `region`별로 GC 발생
  * 이전 GC는 `Young 영역`에 있는 객체들이 GC가 돌때마다 살아남으면 `Eden -> Survivor0 -> Survivor1` 순으로 이동했지만, G1 GC는 순차적으로 이동하지 않음
  * 더욱 효율적이라고 생각하는 위치로 객체를 `Reallocate(재할당)` 시킨다.


G1GC에 새롭게 추가된 Heap영역이 있다.
- Humonogous: Region 크기의 50%를 초과하는 큰 객체를 저장하기 위한 공간
- Available/Unused: 아직 사용되지 않은 Region

<span style="color:red">G1GC</span>에서도 마찬가지로 YoungG Generation 영역에서 객체가 사라질 때 발생하는 **Minor GC**가 존재하며, 요 과정에서 살아남은 객체들을 Survivor Region으로 옮기고, 새로운 생성한 대부분의 객체가 위치하는 곳인 Eden에 대한 영역을 사용가능한(Available)Region으로 돌리는 형태로 과정이 일어나게 된다. 반면 <span style="color:red">G1GC</span>에는 Old Generation 영역에서 발생하는 GC인 **Full GC**와 유사한 **Concurrent Cycle**이라는 과정이 존재한다. 해당 과정은 <span style="color:red">IHOP(InitiatingHeapOccupancyPercent)</span>에서 정한 수치를 초과하면 실행하게 된다.

![G1GC-process.PNG](..%2F..%2Fassets%2Fimg%2FG1GC-process.PNG)
<center>G1GC 과정</center><br>

- **Initial Mark**: Old Region에 존재하는 객체들이 참조하는 Survivor Region을 찾는다(STW)
- **Root Region Scan**: 위에서 찾은 Survivor 객체들에 대한 스캔 작업을 실시한다.
- **Concurrent Mark**: 전체 Heap의 scan 작업을 실시하고, GC 대상 객체가 발견되지 않은 Region은 이 단계를 제외한다.
- **Remark**: 애플리케이션을 멈추고(STW) 최종적으로 GC 대상에서 제외할 객체를 식별한다.
- **CleanUp**: 애플리케이션을 멈추고(STW) 살아있는 객체가 적은 Region에 대한 미사용 객체를 제거한다.
- **Copy**: GC대상의 Region이었지만, Cleanup 과정에서 완전히 비워지지 않은 Region의 살아남은 객체들을 새로운 Region(Available/Unused) Region에 복사하여 Compaction을 수행한다.  
살아있는 객체가 아주 적은 Old 영역에 대해 [GC pause(mixed)]를 로그로 표시하고, Young GC가 이루어질 떄 수집되도록 한다.

### 튜닝포인트
그럼 도대체 뭘 바꿔야할까?  
그럼 도대체 뭘 바꿔야할까? 
우선은 무엇인가를 바꾸기 전에 항상  
**성능테스트 + 로그 옵션**을 켜야한다.
<span style="color:red">-Xlog:gc*:gc.log</span> 옵션으로 로그를 활성화하여 파일로 옮기는 것도 좋다.  
튜닝을 한다는 목적은 GC에 걸리는 시간을 최소화하는 목적으로 하면 좋다.
- **-XX: InitiatingHeapOccupancyPercent**: IHOP 퍼센트 조절(Marking에 해당하는 최저 임계치)
- **-XX: G1HeapRegionSize**: Region 영역당 하나의 사이즈 (default는 최대 (heap)/2048)
- **-XX:G1ReservePercent=10**: 공간 overflow의 위험을 줄이구이해 항상 여유공간을 유지할 예비 메모리(백분율)
- **-XX:G1HeapWastePercent=10**: 낭비할 Heap의 공간에 대한 백분율

이 외에도 매우 많다. 필요한 것은 그때 그때 찾아서 하는 것을 추천!  
하지만 언제까지나 GC 튜닝은 정말 모든 것을 다 해보고 마지막에 하는 최종방안임을 잊지 말 것.  

현재 **JDK 11**의 Default GC 알고리즘으로 별다른 옵션을 주지 않으면 <span style="color:red">G1GC</span>를 사용하게 되는데.. 더 놀라운 것은 앞으로 나올 GC가 **JDK**의 버전업과 함께 준비중에 있다는 사실이다.

### GC 옵션
![G1GC-option.png](..%2F..%2Fassets%2Fimg%2FG1GC-option.png)

<br><br>

## ZGC
* * *
ZGC는 현재 지금 계속 개발되고있는 GC 알고리즘이다.  
<span style="color:red">ZGC</span>는 **JDK 15버전에서 바로 Production Ready** 상태인데, 조금 더 큰 메모리(8MB ~ 16TB)에서 효율적으로 Grabage Collect 하기위한 알고리즘개발자가 이야기하기를
```
적은 메모리나 큰 메모리에서 STW 시간을 최대한 적게(10ms 이하로) 가져가기 위해 제작되었다.
```
라고 한다.  
실제로 <span style="color:red">STW</span> 시간을 줄이기 위해서 Marking 시간에만 <span style="color:red">STW</span>를 가져가도록 하고 있다.  
<span style="color:red">G1GC</span>와는 메모리 구조가 매우 유사하다.

![ZGC.PNG](..%2F..%2Fassets%2Fimg%2FZGC.PNG)

각각의 Region을 간단한 구조로 가져갔을음 볼 수 있다.
<span style="color:red">ZGC</span>의 핵심은 바로 **Colored pointers**와 **Load barriers**라는 주요한 2가지 알고리즘이 존재하는데

![Colored-Pointers.PNG](..%2F..%2Fassets%2Fimg%2FColored-Pointers.PNG)
객체를 가리키는 변수의 포인터에서 64bit를 활용하여, Marking을 한 것을 볼 수 있다.
- Finalizeable : finalizer을 통해서만 참조되는 Object의 Garbage
- Remapped : 재배치 여부를 판단하는 Mark
- Marked 1 / 0 : Live object

그렇기 때문에 ZGC 사용을 위해서는 반드시 64bit 운영체제여야한다.

![Load-Barriers.PNG](..%2F..%2Fassets%2Fimg%2FLoad-Barriers.PNG)

다음은 **Load Barriers**이다.  
<span style="color:red">ZGC</span>는 <span style="color:red">G1GC</span>와는 다르게 메모리를 재배치하는 과정에 위에서 언급한 bit를 바탕으로 **STW**없이 재배치를 한다.  
이때 RemapMark와 RellocationSet을 확인하면서 참조와 Mark를 업데이트 하게 된다.  
그래서 ZGC는 아래와 같은 Flow를 따르게 된다.
- Mark Start <span style="color:red">STW</span> : ZGC의 Root에서 가리키는 객체 Mark 표시
- Concurrent Mark/Rempap: 객체의 참조를 탐색하면서 모든 객체에 Mark 표시
- Mark End <span style="color:red">STW</span> : 새롭게 들어온 객체들에 대해 Mark 표시
- Concurrent Pereare for Relocate: 재배치하려는 영역을 찾아 Relocation Set에 배치
- Relocate Sart <span style="color:red">STW</span> : 모든 Root 참조의 재배치를 진행하고 업데이트
- Concurrent Relocate: 이후 **Load Barriers**를 사용하여 모든 객체를 재배치 및 참조 수정
 
<span style="color:red">G1GC</span>와의 차이점은 바로 Pointer를 이용해서 객체를 Marking하고 관리하는 것이 핵심이다.  
개발자분 말씀으로는 **어떠한 Heap 메모리 사이즈가 와도** 각각의 <span style="color:red">STW</span> 시간을 10ms 이하로 줄이는 것이 <span style="color:red">ZGC</span>의 궁극적인 목표라고 한다.


### ZGC와 G1GC 성능비교
<span style="color:red">ZGC</span>는 위의 설명에서도 알 수 있듯, 큰 메모리에 아주 적합한 GC 방식이다.  
그래서 성능적으로 이득을 보기 위해서는 메모리가 크면 클 수록 좋다!

![SPECjbb2015-pause-times.PNG](..%2F..%2Fassets%2Fimg%2FSPECjbb2015-pause-times.PNG)

위와 같은 테스트 환경은 Heap Size <span style="color:red">128G</span>, CPU <span style="color:red">Intel Xeon E5-2690 2.9GHz, 16core</span> 환경에서 성능을 측정한 결과이다.  
최악의 경우에는 <span style="color:red">G1GC</span>와 비교했을 때 거의 1000배의 **STW** 시간의 차이가 나는 것을 볼 수 있다.

## 힙에 있는 객체들에 대한 참조
1. 힙 내의 다른 객체에 의한 참조
2. Java 스택, 즉 Java 메서드 실행 시에 사용하는 지역 변수와 파라미터들에 의한 참조
3. 네이티브 스택, 즉 JNI(Java Native Interface)에 의해 생성된 객체에 대한 참조
4. 메서드 영역의 정적 변수에 의한 참조

이들 중 힙 내의 다른 객체에 의한 참조(1)를 제외한 나머지 3개는 유효한 최초의 참조(root set)이다. 그리고 이들은 객체에 유효한 참조가 있는지(reachability)를 판가름하는 기준이 된다.
reachability를 더 자세히 보기위해 root set과 힙 내의 객체를 중심으로 다시보면 다음과 같다.

![rootSet-Objects-and-unreachable-objects.PNG](..%2F..%2Fassets%2Fimg%2FrootSet-Objects-and-unreachable-objects.PNG)
<center>그림 2 Reachable 객체와 Unreachable 객체</center><br>

위 그림에서 보듯, root set으로 부터 시작한 참조 사슬에 속한 객체들은 **reachable 객체**이고, 이 참조 사슬과 무관한 객체들이 **unreachable 객체**로 GC 대상이다. 오른쪽 아래처럼 reachable 객체를 참조하더라고, 다른 reachable 객체가 이 객체를 참조하지 않는다면 이 객체는 unreachable 객체이다.  

이 그림에서 참조는 모두 **java.lang.ref 패키지를 사용하지 않은 일반적인 참조**이며, 이를 흔히 strong reference라 부른다.

## Soft, Weak, Phantom Reference
![java-lang-references.PNG](..%2F..%2Fassets%2Fimg%2Fjava-lang-references.PNG)

java.lang.ref는 soft reference, weak reference, phantom reference를 클래스 형태로 제공한다. 예를 들면, java.lang.WeakReference 클래스는 참조 대상인 객체를 캡슐화한 WeakReference객체를 생성한다. 이렇게 생성된 WeakReference객체는 다른 객체와 달리 Java GV가 특별하게 취급한다. 캡슐화된 내부 객체는 weak reference에 의해 참조된다.

### 참고
- [Java Reference와 GC](https://d2.naver.com/helloworld/329631)
- [코딩 테스트, 중급, LRU cache](https://www.youtube.com/watch?v=HpuIrGiHwTo)
- [[10분 테코톡] 🎅무민의 JVM Stack & Heap](https://www.youtube.com/watch?v=UzaGOXKVhwU)
- [JAVA - JNI 사용하기](https://mommoo.tistory.com/71)
- [JVM과 Garbage Collection - G1GC vs ZGC](https://huisam.tistory.com/entry/jvmgc)
- [Garbage Collection의 개념 및 동작 원리 (1/2)](https://mangkyu.tistory.com/118)
- [가비지 컬렉션 GC 동작원리 알고리즘 총정리](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%80%EB%B9%84%EC%A7%80-%EC%BB%AC%EB%A0%89%EC%85%98GC-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%F0%9F%92%AF-%EC%B4%9D%EC%A0%95%EB%A6%AC)
