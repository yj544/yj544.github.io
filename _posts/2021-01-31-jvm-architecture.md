---
title: "JVM Architecture"
categories:
  - java
tags:
  - java
  - jvm
  
toc: true
toc_icon: "cog"
toc_sticky: true
---

### JVM Architecture
![JVM Architecture](https://www.guru99.com/images/1/2.png)

- 1) ClassLoader 
   - 클래스 로더는 클래스 파일을 로딩하고 링킹하고 초기화하는 역할을 한다. 
- 2) Method Area
   - 클래스 파일에 포함되는 메타 데이터, 함수 코드, [constant runtime pool](https://docs.oracle.com/javase/specs/jvms/se6/html/ClassFile.doc.html#20080) 을 저장한다. 
- 3) Heap
   - 인스턴스 변수 및 배열과 관련된 모든 객체가 저장되는 공간으로 여러 쓰레드에서 공유되는 메모리다.
- 4) [JVM Language Stacks ](https://www.geeksforgeeks.org/java-virtual-machine-jvm-stack-area/)
   - 쓰레드는 생성될 때 자체 JVM Stack을 가진다.
   - 쓰레드가 함수를 호출할 때 파라미터, 로컬 변수, 중간 계산 값, 기타 데이터 등은 해당 스택에 저장되고, 함수 호출이 종료되면 스택에서 제거된다.
   - 모든 함수 호출이 완료되면 스택이 완전히 비워지고 메모리에서 제거된 이후 쓰레드는 종료된다.
- 5) PC Registers
   - 쓰레드는 생성될 때 자체 PC Register를 가진다.
   - PC Register에는 현재 수행되는 JVM instruction의 주소가 저장된다.
 - 6) Native Method Stacks
   - Java가 아닌 다른 언어로 작성된 네이티브 코드의 명령어를 저장한다.
- 7) Execution Engine
   - ClassLoader에 의해 로드된 바이트코드 실행을 담당한다.
   - Interpreter 방식과 JIT(Just In Time) 방식을 혼합해서 사용한다.
      - Interpreter : 바이트코드를 한 줄씩 해석해서 실행하는 방식으로 초기에 사용하며 속도가 느리다.
      - JIT : 런타임 시점에 바이트코드를 OS에 최적화된 NativeCode로 변환하는 방식으로 변환 시 비용이 비교적 크기 때문에 초기에는 Interpreter 방식을 사용하다가 일정 시간 이후 JIT으로 변경한다. 
- 8) Native Method Interface
   - JVM에서 실행되는 Java 코드가 네이티브 응용 프로그램, C/C++, 어셈블리 같은 다른 언어들로 작성된 라이브러리를 호출하거나 반대로 호출될 수 있게 하는 프레임워크이다.
- 9) Native Method Library
   - Execution Engine에서 필요한 Native Libraries (C/C++)의 모음이다.    

### Java 코드 컴파일 및 실행 과정
https://www.guru99.com/java-virtual-machine-jvm.html
