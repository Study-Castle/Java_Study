# **3주차 JVM**

## Heap Area란?

![image.png](attachment:e9d8fb31-7131-43d0-be04-4096071c85e4:image.png)

- **Instance(Object)**
- **Array 객체**

만 저장되는 메모리 영역이며, **모든 Thread가 공유**하는 영역

> 자바에서 new 키워드로 생성되는 모든 객체는 Heap에 저장된다.
> 
- JVM 시작 시 생성
- 애플리케이션 실행 동안 유지
- Garbage Collection의 **주 대상 영역**

## Heap Area의 특징

- **Thread 공유 영역**
- 런타임(Runtime) 데이터 저장
- 메모리 해제는 **GC만 가능**
- Java 코드나 Bytecode에는 **명시적 해제 명령이 존재하지 않음**

```
객체 생성 → Heap
객체 제거 → Garbage Collection
```

## Heap 크기 관련 옵션

- 초기 크기
    
    ```
    -Xms
    ```
    
- 최대 크기
    
    ```
    -Xmx
    ```
    

**참고**

- `XX:MaxPermSize` → **Java 8 이전 Perm 영역 전용**
- Java 8 이후에는 **Metaspace**로 대체됨

**Heap 크기**

- GC 전략에 따라 고정 / 가변 가능
- JVM이 실행 중 **동적으로 확장·축소**

## JVM 메모리 구조

❌ Java 메모리 구조 = Heap

⭕ Java 메모리 구조 = **여러 영역의 조합**

| 구분 | Heap Area | Stack Area | Method Area |
| --- | --- | --- | --- |
| 저장 정보 | Object, Array | 지역변수, 매개변수 | Class, Method, Bytecode |
| Thread 공유 | O | X (Thread별) | O |
| GC 대상 | O | X | O (Java 8 이후 일부) |

**Method Area의 일부가 GC 대상이 되는 이유**

Java 8부터 Method Area의 구현이 PermGen에서 Native 영역의 Metaspace로 변경되었지만, Static 객체는 원래부터 Heap에 저장되었으며 GC 여부는 참조 관계와 ClassLoader 생존 여부에 의해 결정된다.

Static 객체는 GC 대상이 될 수 있지만, 일반적으로 ClassLoader가 살아 있어 Reachable 상태이기 때문에 보통 회수되지 않는다.

## Garbage Collection이란?

**더 이상 참조되지 않는 객체(Unreachable Object)를**

**JVM이 자동으로 탐지하고 메모리를 회수하는 과정**

- C 언어: `free()`로 개발자가 직접 메모리 해제
- Java: JVM이 GC를 통해 자동 관리

**Java에서 메모리를 명시적으로 해제하는 방법**

- 객체 참조를 `null`로 변경
- `System.gc()` 호출

`System.gc()`는 **강제 GC 요청으로 성능에 심각한 영향을 주므로 사용 금지**

객체가 더 이상 참조되지 않으면 **GC 대상(Garbage)** 이 되고,

JVM의 Garbage Collector가 주기적으로 이를 탐지하여 제거한다.

## Minor GC와 Major GC

JVM Heap은 다음 두 가지 가정을 기반으로 설계

1. 대부분의 객체는 **금방 접근 불가능(Unreachable)** 상태가 된다.
2. **Old → Young 참조는 매우 드물다.**

이 특성을 활용해 Heap은 크게 두 영역으로 나뉨

![image.png](attachment:df143448-9f9b-4e77-8f03-9ca0b046b112:image.png)

### Young Generation (Minor GC)

- 새로 생성된 객체가 할당되는 영역
- 대부분의 객체가 여기서 생성되고 사라짐
- 이 영역에서 발생하는 GC → **Minor GC**

### Old Generation (Major / Full GC)

- Young 영역에서 살아남은 객체가 이동(Promotion)
- 크기가 크고 GC 빈도는 낮음
- 이 영역에서 발생하는 GC → **Major GC (또는 Full GC)**

**Old 영역이 Young 영역보다 큰 이유**

- Young 객체는 수명이 짧음
- 큰 객체는 바로 Old 영역에 할당되기 때문

## Card Table (Old → Young 참조 문제)

![image.png](attachment:0edaff02-a789-42f8-a02c-7d9bf4f94801:image.png)

**Old 영역 객체가 Young 영역 객체를 참조하는 경우**도 존재

이를 효율적으로 처리하기 위해 **Card Table** 이 사용

- Old 영역을 **512byte 단위의 카드**로 관리
- Old → Young 참조 발생 시 해당 카드만 표시
- Minor GC 시 **Old 전체를 스캔하지 않고 Card Table만 확인**

→ Minor GC 성능 최적화를 위한 핵심 구조

**GC 후의 Card Table 정리**

- 참조가 여전히 있는건 유지
- 참조가 사라졌으면 해당 카드 클리어
- Young 객체가 Old로 Promotion되면 해당 카드 클리어

## Stop-The-World & Mark-Sweep

GC가 실행되면 JVM은 **애플리케이션 실행을 일시 중단**

### **Stop-The-World(STW)**

- GC 스레드를 제외한 모든 스레드 중단
- GC 완료 후 애플리케이션 재개

### GC의 기본 흐름

1. **Mark**
    - Reachable 객체 식별
2. **Sweep**
    - Mark되지 않은 객체 제거
3. (선택) **Compact**
    - 메모리 단편화 제거

→ **모든 GC 알고리즘에서 STW는 발생**

→ GC 튜닝의 핵심은 **STW 시간을 줄이는 것**

## Minor GC 동작 과정 (Young 영역)

Young 영역은 다음 3가지로 구성

- **Eden :** 새로 생성된 객체가 할당되는 영역
- **Survivor 1 :** 최소 1번의 GC이후 살아남은 객체가 존재하는 영역
- **Survivor 2**

### 동작 흐름

![image.png](attachment:c1bb6fd0-9861-432e-9eac-e801d527ce33:image.png)

1. 객체는 Eden에 생성
2. Eden이 가득 차면 Minor GC 발생
3. 살아남은 객체 → Survivor 중 하나로 이동
4. Survivor가 가득 차면 다른 Survivor로 복사 (이때 하나의 Survivor 영역은 빈 공간이 됨)
5. 여러 번 살아남은 객체 → Old 영역으로 Promotion

```
[객체 생성]
   ↓
Eden 부족
   ↓
Minor GC 발생
   ↓
Eden + Survivor 스캔
   ↓
To-Survivor로 복사 시도
   ↓
Survivor 공간 부족?
   ├─ No → 정상
   └─ Yes → Old로 Promotion
               └─ Old 부족 → Full GC
```

**Survivor을 두 개로 나누고 한 공간이 빈 공간이 이유**

만약 복사하려는 Survivor이 빈 공간이 아니라면

- 단편화 제거 힘듦
- 객체 섞임
- age 관리 복잡

Survivor이 하나라면

- 매 Minor GC마다
- 압축 수행 필요
- STW 증가

객체의 생존 횟수는 **age**로 관리됨 (Object Header에 저장)

- Survivor 0 과 1을 오갈 때마다 age++
- Survivor 영역은 **항상 하나만 사용**
- 두 Survivor 모두 사용 중이거나 비어 있으면 비정상 상태

→ **단편화를 제거하고 빠른 Minor GC와 정확한 객체 생존 판단**

**Minor GC 발생 조건**

Eden이 꽉 찼을 때만 발생

→ Minor GC가 이뤄지면서 Survivor이 공간이 부족하게 되는 것

**Survivor이 꽉 차게 되면?**

- Old Generation으로 Promotion
- age의 기준을 못미친다면?
- 조기 승격 가능
- Dynamic Age Determination이라고 하는 조건으로 특정 age 이상의 객체 합계가 Survivor의 일정 비율 초과하면 그 age부터 바로 Old 승격

## Young GC 최적화 기술

### bump-the-pointer

- Eden 마지막 할당 주소 캐싱
- 메모리 탐색 없이 바로 할당

**단점** 

멀티스레드 환경에서 lock contention 발생

**동작 원리**

앞에서부터 차곡차곡 채워지는게 맞음

- GC 이후에는 어차피 살아남은 객체들을 Survivor 영역으로 이동시킴
- 그러고 나서 Eden 전체를 초기화시켜서 compact 단계가 필요가 없음

### TLAB (Thread-Local Allocation Buffer)

- 스레드마다 Eden 일부를 할당
- lock 없이 bump-the-pointer 사용 가능
- HotSpot JVM의 핵심 성능 최적화 기법

### **TLAB의 트레이드오프**

**TLAB 구조**

```
Eden
├─ TLAB (Thread A)
├─ TLAB (Thread B)
├─ TLAB (Thread C)
└─ Global Eden Area
```

- 스레드가 객체 할당을 끝내고 TLAB에 애매한 공간이 남으면
- 그 공간은 **사용 못 하고 Minor GC 때 같이 버려질 수 있음**

→ **공간 낭비(Overspill)는 실제로 존재**

**이 낭비를 무마할 만큼 효율이 좋아지나?**

→ 압도적으로 Yes

1. **객체 생성은 매우 자주 일어남**

서버 애플리케이션에서

- request DTO
- 임시 컬렉션
- 스트링
- 람다 캡처 객체

→ **초당 수십만~수백만 객체 생성**

이때마다

- Eden 전역 bump-pointer에 접근
- CAS / lock 필요

→ 이게 쌓이면 **CPU 캐시 깨지고 성능 급락**

1. **lock 경합 비용은 생각보다 훨씬 크다**

객체 할당에 lock이 들어가면

- 컨텍스트 스위칭
- CPU pipeline flush
- cache line invalidation
- 메모리 배리어 비용

**lock 하나 = 객체 몇 개 생성 비용보다 비싸다**

그래서 JVM은

- 메모리 몇 KB 낭비 ↔ 수백만 번의 lock 제거

→ 이 교환은 **무조건 이득**

**그럼 Eden 영역은 커야 하는 거 아냐?**

JVM의 실제 전략

1. **Eden은 작아도 된다**
- Eden은 **버리는 공간**
- Minor GC 때 통째로 리셋
- 빠른 할당과 빠른 폐기 전략

1. **TLAB 크기는 동적 조절된다**
- 스레드별 할당 패턴 관찰
- TLAB 크기 자동 조절

→ 낭비 최소화 시도는 계속 함

1. **TLAB을 못 쓰는 경우**
- 큰 객체

→ Global Eden Area에 할당

## Major GC 특징

- Old 영역이 가득 차면 발생
- 영역이 크고 참조 관계가 복잡
- Minor GC보다 **10배 이상 비용 (Minor는 0.5초 ~ 1초)**
- 애플리케이션에 큰 영향

## Garbage Collection 알고리즘

### Serial GC

![image.png](attachment:ffa07a44-d025-4035-bce3-2f52e3455b4d:image.png)

- 단일 GC 스레드
- Mark-Sweep-Compact (Old)
- 멀티코어 서버 환경에서는 부적합

**Compact**

Heap 영역 정리

- Mark & Sweep 후 Heap 영역 앞 부분부터 채워서 객체가 존재하는 부분과 존재하지 않는 부분으로 나눔

```
-XX:+UseSerialGC
```

### Parallel GC (Throughput GC)

![image.png](attachment:a2f78389-d7b9-4a8a-82da-dc1d29bfb64e:image.png)

- 여러 GC 스레드를 병렬 수행
- GC에서 발생하는 오버헤드 많이 줄여줌
- 처리량(Throughput) 중시
- 서버 환경에서 널리 사용

```
-XX:+UseParallelGC
-XX:ParallelGCThreads=N
-XX:MaxGCPauseMillis=N
```

### Parallel Old GC

JDK 5 update 6부터 제공

- Parallel GC와 Old 영역에서의 차이점이 있음
- Mark - Summary - Compaction 사용
- Summary 단계에서 앞서 GC를 수행한 영역에 대해 별도로 살아있는 객체를 식별, 약간 복잡

### CMS GC (Deprecated)

![image.png](attachment:0ba9fd9d-c85f-445c-bd11-ae843a3eefb3:image.png)

- 여러 개의 스레드 사용, 기존과는 다른 Concurrent Mark & Sweep
- STW 시간 최소화, 모든 어플리케이션의 응답 속도가 중요할 때 사용
- Compaction 미수행 → 메모리 단편화
- Java 9 비권장 / Java 14 제거

**동작 순서**

1. Initial Mark 단계에서는 살아있는 객체를 찾는 단계로 단순히 객체만 찾고 끝낸다. 이때 Stop the World가 발생하는데 단순히 살아있는 객체만 찾고 끝내기 때문에 STW가 매우 짧다.
2. Concurrent Mark 단계에서는 방금 찾은 살아있는 객체에서 참조하고 있는 객체들을 따라가며 확인한다. 다른 스레드들이 실행중인 상태로 동시에 실행된다.
3. Remark 단계에서는 Concurrent Mark 단계에서 새로 추가되거나 참조가 끊긴 객체를 확인한다. 이때도 STW가 발생하며 마찬가지로 매우 짧다.
4. 마지막 Concurrent Sweep 단계에서는 쓰레기를 정리하는 작업을 진행하며 다른 스레드가 실행되고 있는 상황에서 진행된다.

### G1 GC (Garbage First)

![image.png](attachment:c35b9f87-eb73-4d1a-9dbf-22cb97f9775a:image.png)

장기적으로 서비스를 운영할 때 CMS GC에서 생기는 메모리 단편화의 문제를 위해 개발됨

- Heap을 동일한 영역으로 나누고 각 영역에 객체 할당 및 GC 실행
- 각 영역이 가득차면 다른 영역에 객체 할당 및 GC 실행
- 기존 Young → Old 이동이 사라짐
- 기존 Eden, Survivor, Old + Available/Unused(사용되지 않은 영역), Humonogous(영역 크기의 50% 초과 객체 저장)
- Garbage가 많은 영역부터 GC

**기존 Young → Old 이동의 변경점**

G1 GC에서는 Young과 Old가 고정된 메모리 영역이 아니라는 뜻

동일한 영역에 부여되는 논리적 역할은 그대로

Young과 Old의 물리적 경계를 없앤 것이지 객체 생명주기 개념 자체를 없앤 건 아님.

**Minor GC**

한 영역에 객체를 할당하다가 가득 차면 다른 영역에 객체를 할당하고 Minor GC 수행

- G1 GC는 각 영역을 추적하고 있어서 Garbage가 가장 많은 영역을 찾아 Mark & Sweep 수행
- Eden 영역에서 GC가 실행됐을 때, 살아남은 객체들은 다른 영역으로 이동
- 이때, 복제되는 영역이 Available/Unused라면 해당 영역을 Survivor 영역이 되고 Eden은 비었기 때문에 Available/Unused 영역으로 바뀜

**Major GC (Full GC)**

시스템 운영 중 객체가 너무 많아져 빠르게 메모리를 회수할 수 없을 때

- 기존 알고리즘은 실행 시간이 오래 걸림
- G1 GC는 추적하고 있어서 어느 영역의 Garbage가 많은지 알고 있고 GC를 실행할 영역을 조합해서 해당 영역에서만 실행
- Concurrent하게 실행되어 어플리케이션의 지연 최소화

```
-XX:+UseG1GC
```

→ 대용량 Heap + 멀티코어 서버에 최적

→ Java 9부터 기본 GC