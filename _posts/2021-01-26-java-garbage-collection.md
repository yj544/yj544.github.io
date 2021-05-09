---
title: "Java Garbage Collection"
categories:
  - java
tags:
  - java
  - jvm
  - garbage_collection
  
toc: true
toc_icon: "cog"
toc_sticky: true
---

### Garbage Collector
Java는 개발자가 직접 메모리를 해제하지 않기 때문에 가비지 컬렉터(Garbage Collector)가 사용하지 않는 객체를 찾아 지우는 작업을 수행한다.   

가비지 컬렉터는 두 가지를 가정한다. (weak generational hypothesis)
  - 대부분의 객체가 금방 접근 불가능 상태가 된다.
  - 오래된 객체에서 젊은 객체로의 참조는 거의 없다. 

### Heap Memory 
Heap 메모리는 크게 Young, Old, PermGen(``JDK 8에서는 제외됨``) 세 가지 영역으로 구분된다.
- 1) Young
   - 새롭게 생성된 객체가 위치하는 곳.
   - 대부분의 객체가 금방 접근 불가능 상태가 되기 때문에 해당 영역에서 생성되었다가 사라진다.
   - Young 영역에서 객체가 사라지면 Minor GC가 발생한다고 한다.
   - Eden 영역과 2개의 Survivor 영역(S0, S1)으로 구분된다. 

- 2) Old
   - Young 영역에서 생성된 이후 접근 불가능 상태가 되지 않은 객체가 복사되는 곳.
   - Young 영역보다는 크게 할당되어 GC도 적게 발생한다.
   - Old 영역에서 객체가 사라질 때 Major GC 또는 Full GC가 발생한다고 한다.

- 3) PermGen (Permanent Generation) 
   - Young 및 Old 공간과는 분리된 Permanent Heap 

#### Heap Memory (~ JDK 7)
```
   <----- Java Heap -----> <-PermanentHeap-> <--- Native Memory --->
   +------+----+----+-----+----------------+--------+--------------+
   | Eden | S0 | S1 | Old |    Permanent   | C Heap | Thread Stack |
   +------+----+----+-----+----------------+--------+--------------+
```

- JDK 7까지는 Heap이 Young, Old Heap이 포함되는 Java Heap 영역과 PermGen이 포함되는 PermanentHeap 영역으로 구성되었다.  
- jmap -heap 결과  
   ```
   Heap Configuration:
      MinHeapFreeRatio = 40
      MaxHeapFreeRatio = 70
      MaxHeapSize      = 8558477312 (8162.0MB)
      NewSize          = 1310720 (1.25MB)
      MaxNewSize       = 17592186044415 MB
      OldSize          = 5439488 (5.1875MB)
      NewRatio         = 2
      SurvivorRatio    = 8
      PermSize         = 21757952 (20.75MB)
      MaxPermSize      = 134217728 (128.0MB)

   Heap Usage:
   PS Young Generation
   Eden Space:
      capacity = 534380544 (509.625MB)
      used     = 226094384 (215.6204071044922MB)
      free     = 308286160 (294.0045928955078MB)
      42.30962121255672% used
   From Space:
      capacity = 54132736 (51.625MB)
      used     = 15469264 (14.752639770507812MB)
      free     = 38663472 (36.87236022949219MB)
      28.57654192834443% used
   To Space:
      capacity = 54525952 (52.0MB)
      used     = 0 (0.0MB)
      free     = 54525952 (52.0MB)
      0.0% used
   PS Old Generation
      capacity = 356581376 (340.0625MB)
      used     = 53702664 (51.21485137939453MB)
      free     = 302878712 (288.84764862060547MB)
      15.060423122042133% used
   PS Perm Generation
      capacity = 29229056 (27.875MB)
      used     = 29117608 (27.768714904785156MB)
      free     = 111448 (0.10628509521484375MB)
      99.61870817860145% used
   ```

#### Heap Memory (JDK 8~) 
```                   
   <----- Java Heap -----> <-------------- Native Memory --------->
   +------+----+----+-----+----------------+--------+--------------+
   | Eden | S0 | S1 | Old |   Metaspace    | C Heap | Thread Stack |
   +------+----+----+-----+----------------+--------+--------------+
```

- [PermGen 영역이 사라지고 Metaspace가 추가됨](https://blogs.oracle.com/poonam/about-g1-garbage-collector%2c-permanent-generation-and-metaspace)
- 기존에 PermGen 영역에 저장되던 클래스 메타 정보가 Metaspace에 저장되며 Metaspace는 Native Memory에 위치함 
- jmap -heap 결과
   ```
   Heap Configuration:
      MinHeapFreeRatio         = 0
      MaxHeapFreeRatio         = 100
      MaxHeapSize              = 8558477312 (8162.0MB)
      NewSize                  = 178782208 (170.5MB)
      MaxNewSize               = 2852651008 (2720.5MB)
      OldSize                  = 358088704 (341.5MB)
      NewRatio                 = 2
      SurvivorRatio            = 8
      MetaspaceSize            = 21807104 (20.796875MB)
      CompressedClassSpaceSize = 1073741824 (1024.0MB)
      MaxMetaspaceSize         = 17592186044415 MB
      G1HeapRegionSize         = 0 (0.0MB)

   Heap Usage:
   PS Young Generation
   Eden Space:
      capacity = 134742016 (128.5MB)
      used     = 16169288 (15.420234680175781MB)
      free     = 118572728 (113.07976531982422MB)
      12.00018263048699% used
   From Space:
      capacity = 22020096 (21.0MB)
      used     = 0 (0.0MB)
      free     = 22020096 (21.0MB)
      0.0% used
   To Space:
      capacity = 22020096 (21.0MB)
      used     = 0 (0.0MB)
      free     = 22020096 (21.0MB)
      0.0% used
   PS Old Generation
      capacity = 358088704 (341.5MB)
      used     = 0 (0.0MB)
      free     = 358088704 (341.5MB)
      0.0% used
   ```

#### PermGen to Metaspace
- PermGen 에 저장되던 정보들
   - 클래스의 메타 정보(이름, 생성 정보, 필드, 메서드 등)
   - Static Object
   - interned String Object
   - JVM 내부 
   - Method Area 라고도 하며 , 바이트 코드, JIT 정보 등이 저장되는 영역.
   - JDK7 이전에는 intern된 String 객체도 이 공간에 저장되었음.
   - Major GC가 발생할 수 있음


Old 영역에서 Young 영역의 참조를 체크하기 위해서 512byte 카드 테이블을 사용한다.   
Young 영역에서 GC를 실행할 때는 카드 테이블만 확인해서 GC 대상을 식별한다.  
카드 테이블은 write barrier로 관리하고 이를 통해 Minor GC를 좀 더 빠르게 수행할 수 있다.  
![카드 테이블](https://d2.naver.com/content/images/2015/06/helloworld-1329-2.png)

### [GC Process](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)
- 1) Eden 영역에 새로 생성된 대부분의 객체가 할당된다.
   - ![Object Allocation](/image/1-object-allocation.png)
- 2) Eden 영역이 가득차면 Minor GC가 발생한다. 
   - ![Filling The Eden Space](/image/2-filling-the-eden-space.png)
- 3) 살아남은 객체(Referenced Object)는 S0으로 옮겨지며 Eden 영역은 비워진다.
   - ![Copying Referenced Objects](/image/3-copying-referenced-objects.png)
- 4) Eden 영역이 다시 가득차면 Minor GC가 발생하면서 살아남은 객체는 S1으로 옮겨지고 Eden 영역은 비워진다. 이 때 S0에 있던 객체들이 S1으로 옮겨지고 age 값이 증가한다.
   - ![Object Aging](/image/4-object-aging.png)
- 5) 위 과정을 반복하다가 살아남은 객체의 age 값이 임계값을 넘어가면 (예시에서는 8) 해당 객체는 Old 영역으로 이동한다.
   - ![Promotion](/image/5-promotion.png)

### Old 영역에서 GC
Old 영역은 기본적으로 데이터가 가득차면 GC를 수행하는데 동작 방식은 GC 방식에 따라 다르다. 

### 5가지 GC [동작 방식](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/collectors.html)
- 1) Serial Collector (-XX:+UseSerialGC)
   - Java 5,6에서 사용되는 기본 GC.
   - 단일 쓰레드로 모든 GC 작업을 하기 때문에 적은 메모리와 적은 CPU 코어 개수인 환경에서 적합한 방식.
   - Young 영역은 generation algorithm, Old 영역의 GC는 mark-sweep-compact algorithm을 사용함.

- 2) Parallel Collector (-XX:-UseParallelOldGC)
   - throughput collector 라고도 하며 Young 영역의 GC를 병렬로 수행하여 오버헤드를 줄임. (Old 영역은 그대로 단일 쓰레드 처리)
   - Full GC가 수행될 때마다 Compaction 작업을 진행하여 시간이 많이 소요되나, 이후에는 메모리를 더 빠르게 할당할 수 있음.
      - Compaction : 메모리 사이에 사용하지 않는 공간이 없도록 옮겨서 메모리 단편화를 제거하는 작업
   - 메모리가 충분하고 CPU 코어 개수가 많을 때 적합한 방식.
   - Young 영역은 generation algorithm, Old 영역의 GC는 mark-sweep-compact algorithm을 사용함.

- 3) Parallel Old Collector (-XX:-UseParallelOldGC) 
   - Parallel Collector와 비슷하지만 Young 영역과 Old 영역 모두 GC를 병렬로 수행함.
   - Young 영역은 generation algorithm, Old 영역의 GC는 mark-summary-compact algorithm을 사용함.

- 4) Concurrent Mark Sweep (CMS) Collector (-XX:+UseConcMarkSweepGC)
   - 어플리케이션 쓰레드와 GC 쓰레드를 동시에 실행하여 stop-the-world 시간을 최소화하는 방식.
      - stop-the-world : GC 수행을 위해 JVM이 어플리케이션을 멈추는 것을 말한다. 대개 GC 튜닝이란 stw 시간을 줄이는 것을 의미한다.
   - ![CMS GC](https://d2.naver.com/content/images/2015/06/helloworld-1329-5.png)
   - Compaction 작업을 하지 않아서 Full GC 수행 시 속도가 빠름.
   - 메모리 단편화로 인해 Concurrent mode failure가 발생할 수 있으며 이 때 Compaction 작업을 수행하나 Paraller GC보다 오래걸림.
   - CPU 리소스가 부족하거나 메모리 단편화로 인해 공간이 부족해지면 Serial GC 방식의 Full GC를 수행함.
   - Parallel GC에 비해 1~20% Heap 메모리를 더 사용하고 백그라운드로 GC 쓰레드가 항상 실행되어야 해서 CPU 리소스를 많이 사용함.

- 5) [Garbage-First Garbage Collector](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/g1_gc.html#garbage_first_garbage_collection)
   - G1 GC는 Heap 영역을 여러개의 Region으로 나누고 각 Region에 유동적으로 Young Generation 또는 Old Generation을 할당한다.
      - eden, survivor, old 외에 humongous(객체의 크기가 큰 경우 사용하는 영역), available / unused region(미사용 영역) 타입이 있다. 
   - Young Generation은 Parallel, CMS GC처럼 멀티 쓰레드로 정리하고 Old Generation은 CMS처럼 백그라운드 쓰레드로 정리한다.
   - Region이 꽉차면 GC를 수행하고 남은 객체를 다른 Region에 옮긴다. 옮기는 과정에서 Compacting이 되어 메모리 단편화가 생기지 않는다.
   - Young 영역은 evacution pause algorithm, Old 영역의 GC는 concurrent marking algorithm을 사용함.
   - 모든 GC 방식 중 가장 빠르다.
