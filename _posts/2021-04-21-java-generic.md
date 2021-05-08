---
title: "Java Generic"
categories:
  - java
tags:
  - java
  - generic
  
toc: true
toc_icon: "cog"
toc_sticky: true
---

### [Generic](https://docs.oracle.com/javase/tutorial/java/generics/index.html)
- Generic이란 타입을 일반화한 것을 말한다. 
- Generic을 사용하면 클래스나 함수에서 사용할 내부 데이터 타입이 컴파일 시점에 지정된다.
  - 컴파일 에러가 발생하거나, 캐스팅에 의한 런타임 오류가 발생할 수 있음
  ```java
    List list = new LinkedList();
    list.add(new Integer(1)); 
    Integer i = list.iterator().next(); //ERROR : Type mismatch: cannot convert from Object to Integer
    Integer i = (Integer)list.iterator().next(); //명시적 캐스팅이 필요
  ```
  - 타입을 포함하는 <> 연산자를 통해 컴파일러는 컴파일 시점에 타입을 제한할 수 있음
  ```java
    List<Integer> list = new LinkedList<>();
    list.add(new Integer(1)); 
    Integer i = list.iterator().next();
  ```
- 컴파일러는 Generic이 포함된 코드에 대해 강한 타입 체크를 진행한다.

#### Type Parameter Naming Conventions
- 타입 파라미터의 이름은 일반적인 변수/클래스/인터페이스와 구분짓기 위해 단일 대문자로 작성한다. 
- 일반적으로 많이 사용하는 Type Parameter는 다음과 같다.
  - E : 요소 (Element)
  - K : 키
  - N : 숫자
  - T : 타입
  - V : 값
  - S, U, V : 두 번째, 세 번째, 네 번째에 선언된 타입

#### Invoking and Instantiating a Generic Type
- Generic 클래스를 참조하려면 구체적인 값을 전달해야 한다.
  ```java
  public class Box <T> { 
      private T t; 
      public void set (T t) {this.t = t; } 
      public T get () {return t; } 
  }

  Box <Integer> integerBox;
  ```

- Type Parameter와 Type Argument는 다른 개념이다.
  - List<T>의 T는 Type Parameter이고, List<String>의 String은 Type Argument이다.

#### [Type Inference](https://docs.oracle.com/javase/tutorial/java/generics/genTypeInference.html)
- Java Compiler는 Generic이 사용된 경우 함수를 호출부와 선언부를 확인하여 해당 변수에 실제로 적용될 타입을 결정한다.
- 아래 코드의 경우 pick 함수의 리턴 타입이 Serializable이기 때문에 두 번째 파라미터 a2가 Serializable인지 확인한다.
  ```java
  static <T> T pick(T a1, T a2) { return a2; }
  Serializable s = pick("d", new ArrayList<String>());
  ```
- Generic 함수의 타입 추론
  ```java
  public class BoxDemo {
    public static class Box<U> {
      U u;
      
      public void set(U u) {
        this.u = u;
      }
      
      public U get() {
        return u;
      }
    }

    public static <U> void addBox(U u, java.util.List<Box<U>> boxes) {
      Box<U> box = new Box<>();
      box.set(u);
      boxes.add(box);
    }

    public static <U> void outputBoxes(java.util.List<Box<U>> boxes) {
      int counter = 0;
      for (Box<U> box : boxes) {
        U boxContents = box.get();
        System.out.println("Box #" + counter + " contains [" + boxContents.toString() + "]");
        counter++;
      }
    }

    public static void main(String[] args) {
      java.util.ArrayList<Box<Integer>> listOfIntegerBoxes = new java.util.ArrayList<>();
      //Generic Method(addBox)를 호출할 때 타입 명시하지 않아도 listOfIntegerBoxes의 타입을 통해 Integer라는 것을 유추한다.
      BoxDemo.<Integer>addBox(Integer.valueOf(10), listOfIntegerBoxes);
      BoxDemo.addBox(Integer.valueOf(20), listOfIntegerBoxes);
      BoxDemo.addBox(Integer.valueOf(30), listOfIntegerBoxes);
      BoxDemo.outputBoxes(listOfIntegerBoxes);
    }
  }
  ```
- Generic Class의 타입 추론
  - Generic Class의 생성자를 호출할 때 type argument 대신 type parameter <> (diamond)를 사용할 수 있다.
    ```java
    //Java7 이전
    Map<String, List<String>> myMap = new HashMap<String, List<String>>();
    //Java7 부터
    Map<String, List<String>> myMap = new HashMap<>();
    ```
  - 다만 Generic Class의 객체를 생성할 때 타입 추론을 위해서는 <>를 사용해야 하며 그렇지 않은 경우 아래와 같은 경고를 띄운다.
    ```java
		//HashMap is a raw type. References to generic type HashMap<K,V> should be parameterized
		Map<String, List<String >> myMap = new HashMap();
    ```

- Generic 생성자(generic parameter가 하나 이상인 생성자)의 타입 추론
  - 클래스의 Generic 유무에 상관 없이 생성자 또한 Generic이 될 수 있다.
    ```java 
    public class GenericBox <U> {
      public <T> Box(T t){
        //별도의 type parameter를 가지는 Generic 생성자
      }
    }

    //<>를 통해 Generic 클래스의 타입은 Integer, Generic 생성자의 타입은 String이라는 것을 유추
    GenericBox<Integer> box = new GenerixBox<>("box");

    public class Box {
      public <T> Box (T t){
        // Generic Class가 아닌 클래스의 Generic 생성자
      }
    }
    ```
- TargetType
  - 컴파일러는 TargetType을 Generic Method 호출 시 타입 추론을 할 때 사용한다. 
  ```java
    public static final <T> List<T> emptyList() {
      return (List<T>) EMPTY_LIST;
    }

    //TargetType이 List<String>이 되며 emptyList의 type argument는 String으로 추론됨
    //List<String> listOne = Collections.<String>emptyList();
    List<String> listOne = Collections.emptyList();
  ```
  - Java SE 7 이후로는 TargetType의 범위가 확장되었다.
  ```java
	  static void processStringList(List<String> stringList) {
	    // process stringList
	  }

    //Java SE 7 Error : The method processStringList(List<String>) is not applicable for the arguments (List<Object>)
    //emptyList의 T가 추론될 수 있는 TargetType이 없어서 Object로 생성됨
    processStringList(Collections.emptyList());
  
    //TargetType이 함수 인자까지 확장되어 T는 String으로 추론됨
    processStringList(Collections.<String>emptyList());
  ```

#### Bound Type Parameter
- Generic을 특정 타입의 서브 타입으로 제한하는 타입이다.
  ```java
  // T는 Number의 하위 클래스만 가능
  public <T extends Number> List<T> fromArrayToList(T[] a) {
      //...
  }
  ```
- Multiple Bounds
  ```java
	class A { /* ... */ }
	interface B { /* ... */ }
	interface C { /* ... */ }

	class D <T extends A & B & C> { /* ... */ } // 타입 중에 클래스가 있는 경우 해당 값을 먼저 써줘야 한다. 
	class D <T extends B & A & C> { /* ... */ } // Error : Duplicate nested type D
  ```

#### [WildCard With Generics](https://docs.oracle.com/javase/tutorial/java/generics/wildcards.html)
- 와일드 카드는 ?로 나타내며 알수 없는 유형을 참조할 때 사용한다.
- Upper Bounded Wildcards
   - 특정 타입의 서브 타입으로 제한
  ```java
  public static void process (List <? extends Foo> list) {/ * ... * /}
  ```
- Unbounded Wildcards
  - ?만 사용하여 지정하는 방식으로 Object 클래스의 함수로 구현했거나, 코드가 타입에 의존하지 않는 경우 유용하다.
    ```java
    public static void printList (List <?> list) {
        for (Object elem : list)
            System.out.print (elem + "");
        System.out.println ();
    }
    ```
- Lower Bounded Wildcards
  - 특정 타입의 상위 타입 클래스로 제한
    ```java
    public static void addNumbers (List <? super Integer> list) {
        for (int i = 1; i <= 10; i ++) {
            list.add (i);
        }
    }
    ```

### Type Erasure
- 이전 소스와의 호환성을 위해 Generic Type은 컴파일 시점에 검사된 이후 런타임 시점에는 제거된다. 
- Type Paremeter는 Object로 바뀐다.
  ```java
  public class Node<T> {

      private T data;
      private Node<T> next;

      public Node(T data, Node<T> next) {
          this.data = data;
          this.next = next;
      }

      public T getData() { return data; }
      // ...
  }
  //컴파일 시 type parameter T를 Object(생략됨)로 바꿈
  public class Node {

    private Object data;
    private Node next;

    public Node(Object data, Node next) {
        this.data = data;
        this.next = next;
    }

    public Object getData() { return data; }
    // ...
  }
  ```
  - Bounded Type Parameter의 경우 해당 타입으로 바뀐다.
    ```java
    public class Node<T extends Comparable<T>> {

        private T data;
        private Node<T> next;

        public Node(T data, Node<T> next) {
            this.data = data;
            this.next = next;
        }

        public T getData() { return data; }
        // ...
    }

    public class Node {

        private Comparable data;
        private Node next;

        public Node(Comparable data, Node next) {
            this.data = data;
            this.next = next;
        }

        public Comparable getData() { return data; }
        // ...
    }
    ```
- [Effects of Type Erasure and Bridge Methods](https://docs.oracle.com/javase/tutorial/java/generics/bridgeMethods.html)
