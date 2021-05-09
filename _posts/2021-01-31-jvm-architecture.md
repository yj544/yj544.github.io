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
![JVM Architecture](/image/JVM-Architecture.png)

- 1) ClassLoader 
   - 클래스 파일을 로딩하고 링킹하고 초기화하는 역할
- 2) Method Area
   - 클래스 파일에 포함되는 메타 데이터, 함수 코드, [constant runtime pool](https://docs.oracle.com/javase/specs/jvms/se6/html/ClassFile.doc.html#20080)이 저장되는 영역 
- 3) Heap
   - 인스턴스 변수 및 배열과 관련된 모든 객체가 저장되는 공간으로 여러 쓰레드에서 공유되는 메모리
- 4) [JVM Language Stacks](https://www.geeksforgeeks.org/java-virtual-machine-jvm-stack-area/)
   - 쓰레드에게 개별적으로 할당되는 영역으로 함수를 호출할 때 파라미터, 로컬 변수, 중간 계산 값, 기타 데이터 등은 해당 스택에 저장되고, 함수 호출이 종료되면 스택에서 제거된다.
   - 모든 함수 호출이 완료되면 스택이 완전히 비워지고 메모리에서 제거된 이후 쓰레드는 종료된다.
- 5) PC Registers
   - 쓰레드는 생성될 때 자체 PC Register를 가진다.
   - PC Register에는 현재 수행되는 JVM instruction의 주소가 저장된다.
 - 6) Native Method Stacks
   - Java가 아닌 다른 언어로 작성된 네이티브 코드(JNI를 사용하는 등)의 명령어를 저장한다.
   - JVM Language Stacks 와 마찬가지로 쓰레드 별로 고유의 Native Method Stacks를 할당받는다.
- 7) Execution Engine
   - ClassLoader에 의해 로드된 바이트코드 실행을 담당한다.
   - Interpreter 방식과 JIT(Just In Time) 방식을 혼합해서 사용한다.
      - Interpreter : 바이트코드를 한 줄씩 해석해서 실행하는 방식으로 초기에 사용하며 속도가 느리다.
      - JIT : 런타임 시점에 바이트코드를 OS에 최적화된 NativeCode로 변환하는 방식으로 변환 시 비용이 비교적 크기 때문에 초기에는 Interpreter 방식을 사용하다가 일정 시간 이후 JIT으로 변경한다. 
- 8) Native Method Interface
   - JVM에서 실행되는 Java 코드가 네이티브 응용 프로그램, C/C++, 어셈블리 같은 다른 언어들로 작성된 라이브러리를 호출하거나 반대로 호출될 수 있게 하는 프레임워크이다.
- 9) Native Method Library
   - Execution Engine에서 필요한 Native Libraries (C/C++)의 모음이다.    

#### JVM Stack
- JVM이 쓰레드 생성 시 개별적으로 할당하는 스택 영역으로 다른 쓰레드에서는 접근 불가
- 쓰레드에서 함수를 호출할 때 파라미터, 로컬 변수, 중간 계산 값, 기타 데이터 등은 해당 스택에 저장됨
- 쓰레드 내 모든 함수 호출이 완료되면 스택은 비워지고 쓰레드를 종료하기 전 JVM에 의해 빈 스택은 파괴됨

##### Stack Frame
- 쓰레드 스택에서 함수 호출 단위로 스택 프레임이 구성되며 각 스택 프레임은 Local Variable Array, Operand Stack, Frame Data로 구성됨
- JVM은 함수를 호출할 때 먼저 클래스 정보를 확인하여 함수에서 필요한 로컬 변수 배열, 피연산자 스택의 적절한 사이즈를 계산해서 스택 프레임을 생성하고 JVM 스택에 푸쉬함
- Local Variable Array (LVA)
   - 함수에 대한 모든 파라미터와 로컬 변수가 포함됨
   - 4byte 배열로 int,float 및 참조 변수의 경우 4byte를 차지함
   - double, long은 8byte(배열 2칸)를 차지함
   - 사이즈가 4byte보다 작은 byte, short, char은 저장 전 int로 변환되어 4byte를 차지함
   - boolean의 경우 JVM마다 다르지만 대체로 4byte를 차지함 
- Operand Stack
   - JVM이 중간 계산 결과를 저장하는 등 작업 공간으로 사용
   - 배열의 형태이지만 인덱스를 통해 접근하지는 않고 스택처럼 push 또는 pop 명령으로 사용함
- Frame Data 
   - Constant Pool Resolution 정보, 함수의 리턴 정보 또는 예외 발생 시의 Exception 정보 등이 저장되는 공간 
   - 함수가 정상 완료되면 JVM은 Frame Data를 통해 기존 스택 프레임을 복원한다.
   - 함수에서 예외가 발생하면 JVM은 Frame Data에서 참조하는 예외 테이블을 통해 예외 처리 방법을 결정한다.

##### Stack Frame의 동작
   ```java
      int i = 20;
      int j = 14;
      int k = i - j;
   ```
   위와 같은 코드를 컴파일하면 아래와 같은 바이트 코드가 생성되고, 
   ```
      //bipush : Push byte, The immediate byte is sign-extended to an int value. That value is pushed onto the operand stack. (int인데 값이 작아서인지 byte로 푸쉬된다.)
      //istore_<n> : Store int into local variable, The <n> must be an index into the local variable array of the current frame. It is popped from the operand stack, and the value of the local variable at <n> is set to value.
      //iload_<n> : Load int from local variable, The <n> must be an index into the local variable array of the current frame. The value of the local variable at <n> is pushed onto the operand stack.
      //isub : Subtract int. Both value1 and value2 must be of type int. The values are popped from the operand stack. The int result is value1 - value2
      ...
      Code:
      stack=2, locals=4, args_size=1
         0: bipush        20
         2: istore_0          
         3: bipush        14
         5: istore_1
         6: iload_0
         7: iload_1
         8: isub
         9: istore_2
         10: return
      ...
   ```
   각 과정에서 스택 프레임의 동작은 다음과 같을 것이다.
   ![Stack Frame](/image/stackframe.png)
