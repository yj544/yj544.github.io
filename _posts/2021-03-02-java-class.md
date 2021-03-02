---
title: "Java Class 잡다한 정리"
categories:
  - java
tags:
  - java
  
toc: true
toc_icon: "cog"
toc_sticky: true
---

### Constructor Chaining
- 같은 클래스에서의 Constructor Chaining : 인스턴스를 가리키는 this 를 통해 수행
```java
public class Test1 {
	public Test1() {
		System.out.println("Test1()");
	}

	public Test1(int x) {
		//super와 달리 this는 기본 생성자에 대해서 기본적인 chaining을 추가하지 않는다.
		this(); //호출 안 하면 Test1()은 호출되지 않음
		System.out.println(x);
	}

	public Test1(int x, int y) {
		this(x);
		System.out.println(x * y);
	}
	
	public static void main(String[] args) {
		Test1 test = new Test1(10,20);
		//결과 
		//Test1()
		//10
		//200
	}
}
```

- 대체로 매개변수가 적은 상위 생성자를 호출하는 방식이겠지만, Chaining의 방향이 정해진 것은 아니다.
```java
public class Test2 {
	public Test2() {
		this(5);
		System.out.println("Test2()");
	}

	public Test2(int x) {
		this(5, 15);
		System.out.println(x);
	}

	public Test2(int x, int y) {
		System.out.println(x * y);
	}
	
	public static void main(String[] args) {
		Test2 test = new Test2();
		//결과 
		//75
		//5
		//Test2()
	}
}
```

- 상속 관계의 클래스에서의 Constructor Chaining : super
```java
public class Child extends Parent{
	public Child() {
		//super(); 가 생략된 형태이다
		System.out.println("Child()");
	}

	public Child(String name) {
		//super의 생성자가 불리면 super();는 추가되지 않는다.
		super(name);
		System.out.println("Child(String name)"); 
	}
}

public class Parent {
	public String name;

	public Parent(){
		System.out.println("Parent()"); 
	}
	
	public Parent(String name){
		this.name = name;
		System.out.println("Parent(String name)"); 
	}

	public static void main(String[] args) {
		// 호출 순서 : Parent() -> Child()
		Child c = new Child();
		
		// 호출 순서 : Parent(String name) -> Child(String name)
		Child c2 = new Child("test");
	}
}
```

### static block / instance block
```java
public class Parent {
   public static int count;
	public String name;
	
	static {
      //당연한 얘기지만 이 블록은 클래스가 메모리에 올라갈 때 한 번만 수행된다.
		System.out.println("static initialize block");
      count = 20;
	}
	
	{
      //당연한 얘기지만 클래스의 인스턴스가 생성될 때마다 호출된다.
		System.out.println("instance initialize block");
      name = "parent1";
	}
	
	public Parent(){
		System.out.println("Parent()"); 
      this.parent = "parent2";
	}
```

### inner class
- InnerClass의 경우 부모클래스$대상클래스.class 의 형식으로 클래스 파일이 만들어진다.
- 아래와 같은 클래스는 빌드 시 다음과 같은 형태의 class 파일이 생성된다.
   - Parent$ParentInnerClass$ParentInnerInnerClass.class
	- Parent$ParentInnerClass.class
	- Parent.class
   ```java
   public class Parent {
      public class ParentInnerClass extends Parent{
         public class ParentInnerInnerClass extends ParentInnerClass{
         }
      }
   ```

### abstract class / final class
- 클래스의 객체 생성을 막으려는 경우 abstract를 사용한다.
   ```java
   public abstract class Parent {
      //...
      public static void main(String[] args) {
         //'Parent' is abstract; cannot be instantiated
         //Parent p = new Parent(); 
      }
   }
   ```

- 클래스의 상속을 막으려는 경우 final을 사용한다.
   ```java
   public final class Child extends Parent{
      //...
      //public class ChildChild extends Child {
      //	Cannot inherit from final 'Child'
      //}
   }	
   ```

### Class 클래스 
- Class 클래스의 인스턴스는 JVM에서 실행되는 모든 클래스 및 인스턴스에 대한 정보를 가진다.
- 여기에는 enum(클래스의 일종), annotation(인터페이스의 일종), 그리고 primitive type 및 void 키워드도 포함된다.
- ClassLoader의 로딩 과정에서도 Class 클래스를 사용한다. 
- class의 반환형은 Class 클래스다.
   ```java
   Class integerClass = Integer.class;
   ```
- Class 클래스를 통해서 현재 JVM에 로딩된 클래스들의 여러 정보를 얻을 수 있다.
   ```java
		// Parent
		System.out.println(Child.class.getSuperclass().getName());

		// java.lang.Object : 모든 클래스의 최상위 부모 클래스
		System.out.println(Parent.class.getSuperclass().getName());

      //클래스명.class 외에 Class.forName을 통해서도 얻을 수 있다.
		Class.forName("Parent").getSuperclass().getName();
   ```

###