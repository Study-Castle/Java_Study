# **2주차 JVM**

## JVM이란?

JVM(Java Virtual Machine)은 **Java 바이트코드(.class)를 실행하는 가상 머신이다.**

Java는 운영체제에 직접 실행되지 않고

![image.png](attachment:1e766782-3f79-4381-b520-4b7f73b2be7e:image.png)

위와 같은 구조를 가지기 때문에

**어느 운영체제에서도 동일한 Java 프로그램 실행이 가능**하다.

> Write Once, Run Anywhere의 핵심
> 

JVM은 JRE(Java Runtime Environment)의 핵심 구성 요소이며,

JRE = JVM + Java 표준 라이브러리라고 볼 수 있다.

## JVM 전체 구조

![image.png](attachment:12f37574-2c3d-4f52-b579-ffa338795993:image.png)

### Class Loader

- `.class` 파일을 **필요한 시점에 메모리로 로드**
- 클래스 로더 계층 구조 사용

### Runtime Data Area

- JVM이 실제로 데이터를 저장하고 실행하는 메모리 영역

### Execution Engine

- 바이트코드를 실행
- 인터프리터 + JIT 컴파일러 사용

### Native Method Interface (JNI)

- Java 코드에서 C/C++ 같은 네이티브 코드 호출 가능하게 함

## 자바 클래스 파일(.class) 구조

![image.png](attachment:f3ad8c1c-f134-481f-8166-f79e4ecfd6a0:image.png)

자바 클래스 파일은 JVM이 이해할 수 있는 **표준 포맷**을 가진다.

### 기본 구성 요소

| 항목 | 설명 |
| --- | --- |
| Magic Number | `0xCAFEBABE` – 자바 클래스 파일 식별 |
| 버전 정보 | 컴파일된 Java 버전 |
| Constant Pool | 클래스/메서드/필드 참조 저장 |
| Access Flags | public, abstract, final 등 |
| Class / Super Class | 자신과 부모 클래스 정보 |
| Interfaces | 구현한 인터페이스 |
| Fields | 멤버 변수 |
| Methods | 메서드 |
| Attributes | 디버깅 정보, 애노테이션 등 |

### Constant Pool이 중요한 이유

- 클래스, 메서드, 필드 이름은 **문자열이 아니라 인덱스로 참조**
- 런타임 시 **심볼릭 레퍼런스를 실제 메모리 주소로 연결**
- 이를 통해 프로그램이 메모리 내에서 객체나 클래스에 접근 가능

## 클래스 로딩 시점

JVM은 **모든 클래스를 한 번에 로딩하지 않는다.**

→ 클래스를 컴파일 시점이 아니라 **실행 중에 로딩**

→ **동적 클래스 로딩**

### 클래스는 언제 로딩될까?

- JVM 시작 시 main 클래스 로딩
- main 클래스가 **직접 참조하는 클래스**
- 상속/구현 관계에 있는 클래스
- static 필드 / static 메서드 참조
- `Class.forName()`
- `ClassLoader.loadClass()`
- 설정 파일 / DB / 사용자 입력 기반 클래스 선택
- Spring, JDBC, SPI, 플러그인 시스템

**지연 로딩(Lazy Loading)**

> 클래스가 **참조 가능 상태에 있어도**
실제로 사용되기 전까지 로드하지 않는 방식
> 
- JVM의 **메모리 최적화 전략**
- 로드할 수 있다 ≠ 지금 당장 로드한다
- **정말 필요해질 때까지 미룬다**

**왜 동적 로딩이 중요할까?**

- **필요한 클래스만** 메모리에 올림 → 메모리 효율
    - 만약 정적 로딩으로?
    - 프로그램 시작 시 모든 클래스, 라이브러리, 의존성 한번에 메모리 적재
    - **시작 속도 ↓, 메모리 사용량 ↑, 실제로 쓰지도 않는 클래스 낭비**
    - 동적 로딩 사용?
    - **힙 + 메타스페이스 사용량 ↓, GC 부담 ↓, 서버 안정성 ↑**
- 리플렉션, 플러그인, 프레임워크(Spring) 구현 가능
- 런타임 확장 가능

## 클래스 로딩 과정

![image.png](attachment:5e9abeb0-8135-46ca-9ccd-762987d3029b:image.png)

### 1. Loading

Loading 과정에서 파일을 업로드할 때 각각의 .class 파일들이 기본으로 제공받은 파일인지 개발자가 정의한 파일인지 등에 따라 맞는 클래스 로더에 따라 로딩

- `.class` 파일을 읽어 메모리에 적재
- `java.lang.Class` 객체 생성
- Method Area에 클래스 정보 저장

### 2. Linking

로드된 .class 파일들을 사용하기 위해 검증하고 준비하는 과정

**Verification(검증)**

JVM에서 사용이 가능한 형태인지 검증

- 바이트코드 검증
- JVM 규칙 위반 시 `VerifyError`

**Preparation(준비)**

클래스가 필요로 하는 메모리 할당, 클래스의 필드, 메서드, 인터페이스를 나타내는 데이터 구조 준비

- static 변수 메모리 할당
- 타입의 기본값으로 초기화 (`0`, `null`)

**Resolution(분석)**

JVM의 할당된 메모리 참조를 Method Area에 참조된 메모리 주소 값으로 바꾸는 단계

- Constant Pool의 심볼릭 참조 → 실제 참조로 변환

### 3. Initialization

.class 파일의 코드를 읽어 Java 코드에서의 class와 interface의 값들을 지정한 값들로 초기화하고 method 실행 

- static 변수에 **명시된 값으로 초기화**
- Multi Threading으로 이뤄져 동시에 여러 class가 초기화 진행

## 클래스 로더 구조

![image.png](attachment:7487e945-66a7-46b4-8f51-20957c6aa305:image.png)

### JVM 클래스 로더 계층

**Bootstrap Class Loader**

JVM을 실행시키기 위한 필수 라이브러리 로드

- `java.lang`, `java.util` 등 핵심 클래스
- C/C++로 구현됨

**Extension Class Loader**

Bootstrap ClassLoader를 부모로 갖는 클래스 로더로 확장 자바 클래스 로드

- `JAVA_HOME/lib/ext` 경로

**Application Class Loader**

어플리케이션 레벨의 클래스 로드

- classpath에 있는 사용자 클래스, 라이브러리
- 개발자가 작성한 클래스

**Custom Class Loader**

- 사용자가 직접 구현 가능
- 동적 로딩, 플러그인 구조 등에 활용

## 프로그램 실행 흐름

```
클래스 로딩
 → 링킹
 → 초기화
 → main() 실행
 → 객체 생성
 → 메서드 호출
 → 종료
```

### 객체 생성 과정

1. 힙 메모리에 객체 공간 할당
2. 부모 클래스 생성자 호출
3. 자신의 생성자 호출

### 메서드 실행과 스택

- 메서드 호출 시 **스택 프레임 생성**
- 로컬 변수, 매개변수, 반환 주소 저장
- 메서드 종료 시 스택 프레임 제거

## Runtime Data Area (JVM 메모리 구조)

Runtime Data Area는 JVM이 자바 프로그램을 실행하면서 사용하는 **메모리 영역**이다.

크게 **스레드 공유 영역**과 **스레드 전용 영역**으로 나뉜다.

![image.png](attachment:76ff6256-33ad-41a8-a702-15ef6390c368:image.png)

## 스레드 공유 vs 스레드 전용

### 모든 스레드가 공유

- **Heap**
- **Method Area (또는 Metaspace 포함 개념)**

### 스레드마다 따로 생성 (Thread-local)

- **JVM Stack**
- **PC Register**
- **Native Method Stack**

**구분이 중요한 이유**

- 공유 영역(Heap/Method Area)은 **동시성 이슈**(락, GC 등)와 연관이 크고
- 전용 영역(Stack/PC/Native Stack)은 **각 스레드의 실행 흐름**과 직결된다.

## Method Area / Metaspace

![image.png](attachment:ca51c59d-7799-42d4-9854-04ecf944c027:image.png)

### Method Area란?

JVM이 클래스(.class)를 로드할 때, **클래스 레벨 정보**를 저장하는 영역이다.

저장되는 대표 정보

- 클래스 구조 정보(필드/메서드 시그니처)
- static 변수(정적 필드)
- 런타임 상수 풀(Runtime Constant Pool)
- 메서드 바이트코드/메타데이터

> 흔히 Class Area, Static Area라고도 부른다.
> 

### PermGen / Metaspace 구분

- **Java 7까지**: Method Area 구현 중 하나로 PermGen(퍼머넌트 제너레이션)이 존재했음
- **Java 8부터**: PermGen 제거 → **Metaspace**로 대체됨

**차이 핵심**

- PermGen: JVM 힙 일부로 관리, 크기 제한 설정이 중요
- Metaspace: 네이티브 메모리(운영체제 메모리)를 사용, 필요하면 더 확장 가능(무한정은 아님)

| Method Area | Metaspace |
| --- | --- |
| JDK8 이후부터 삭제 | JDK8부터 도입 |
| 항상 최대 크기가 고정 | 기본 OS에 따라 자동으로 크기를 늘림 |
| 연속적인 Java Heap 메모리 | 기본 OS에서 제공하는 메모리 |
| 비효율적인 Garbage Collection | 효율적인 Garbage Collection |

### Runtime Constant Pool

Method Area 내부에 포함되는 클래스별 상수/참조 테이블 느낌이다.

**여기에 들어가는 것**

- 리터럴 상수(문자열, 숫자)
- 클래스/메서드/필드에 대한 **심볼릭 레퍼런스**

JVM은 실행 중 참조가 필요하면

- 상수 풀 인덱스 → 실제 메모리 대상(클래스, 메서드 등)으로 연결해서 사용

## Heap Area (힙)

힙은 런타임에 생성되는 **객체(인스턴스), 배열** 같은 참조 타입 데이터가 저장되는 영역이다.

- `new`로 생성되는 대부분은 Heap에 저장됨
- 모든 스레드가 공유함
- GC(Garbage Collection)의 주 무대

### 참조 관계 핵심

![image.png](attachment:fc7b35e3-6358-4d73-90ee-86735d662a18:image.png)

힙에 있는 객체 자체를 직접 다루는 게 아니라,

- **스택에 있는 참조 변수**가 힙 객체의 주소(참조)를 들고 있음

```java
Person p = new Person();
```

- `new Person()` 객체 → Heap
- `p` 참조 변수 → Stack (그리고 Heap 객체를 가리킴)

→ 참조가 끊기면 GC 대상

### Heap 세대 구분(Generational GC)

힙을 효율적으로 관리하기 위해 **세대(Generation)** 개념을 쓴다.

**Young Generation**

생명주기 짧은 객체가 많아서 **자주 GC**가 발생하는 영역

**Old Generation**

Young에서 오래 살아남은 객체가 올라오는 곳

여기는 GC가 상대적으로 덜 자주 일어나지만, 한 번 할 때 비용이 큼.

## JVM Stack (스택)

스택은 **메서드 호출 단위로 생성되는 실행 프레임**을 저장한다.

스레드마다 하나씩 존재하고, 스레드 종료 시 같이 사라진다.

### Stack Frame에 들어가는 것

메서드가 호출될 때마다 Frame이 생기고, 종료되면 제거된다.

Frame 안에는 보통

- 지역변수(Local Variables)
- 매개변수(Parameters)
- 중간 연산 값(Operand Stack)
- 반환 주소(Return Address) 등

### StackOverflowError

스택은 크기가 제한되어 있어서

재귀 호출이 너무 깊거나, 프레임이 과도하게 쌓이면 `StackOverflowError` 발생 가능.

## PC Register

PC Register는 각 스레드가 **지금 실행 중인 바이트코드 명령 위치**를 기억하는 곳이다.

**왜 필요?**

- JVM은 멀티스레드 환경에서 스레드가 번갈아 실행되는데
- 다시 돌아왔을 때 어디부터 실행해야 하는지를 알아야 한다.

→ PC Register는 JVM 관점의 실행 커서

## Native Method Stack

![image.png](attachment:d9d37f80-3dc4-4027-8411-6c9f046ec8a0:image.png)

Java 코드가 아니라 JNI를 통해 호출된 네이티브 코드(C/C++ 등)를 실행할 때

그 호출 스택을 관리하는 영역이다.

흐름 예시

- Java 메서드 실행 → JVM Stack 사용
- 내부에서 `native` 메서드 호출 → Native Method Stack 사용
- 네이티브 코드 끝나면 → 다시 JVM Stack으로 복귀

## Excecution Engine(실행 엔진)

실행 엔진을 통해 Runtime Data Area에 있는 바이트 코드를 기계어로 변환해서 실행

### Interperter(인터프리터)

바이트 코드를 한줄 한줄 해석하고 실행

- 명령어를 처음에 시작하는 속도는 컴파일러보다 빠르지만 전체적인 수행 속도는 느리다
- 또한 한줄 한줄 수행되다 보니 동일 바이트 코드를 번역하더라도 새로 해석해서 수행

### Just In Time(JIT)

인터프리터의 속도 문제를 해결하기 위해 디자인된 기능

- 바이트 코드를 한 번에 읽어 번역
- 처음 시작하는 속도는 느릴 수 있지만 전체적인 수행 속도는 빠르다.

JVM은 이 두 방식을 결합해서 사용

→ 프로그램이 처음 실행될 때는 인터프리터를 통해 빠르게 실행되지만

→ 일정한 코드 블록이나 메서드가 자주 실행되는 경우엔 JIT 컴파일러를 통해 최적화