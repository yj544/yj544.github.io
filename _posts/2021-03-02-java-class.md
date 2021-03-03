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

### inner class / nested class
- 클래스 안에서 다른 클래스를 정의할 때 static으로 선언하는 경우 ``static nested class``, non-static인 경우 ``inner class``라고 한다.
	```java
	class OuterClass {
		static class StaticNestedClass {
			//
		}
		class InnerClass {
			//
		}
	}
	```
- {OuterClassName}${InnerClassName}.class 의 형식으로 클래스 파일이 만들어진다.

   ```java
   public class Parent {
      public class ParentInnerClass extends Parent{
         public class ParentInnerInnerClass extends ParentInnerClass{
         }
      }
   ```   
	- Parent$ParentInnerClass$ParentInnerInnerClass.class
	- Parent$ParentInnerClass.class
	- Parent.class
- 
	```java
	public class OuterClass {
		
		private String name;
		public int count;
		
		private void init() {
			// num = 20; // OuterClass에서 InnerClass의 멤버변수에 접근할 수 없다.
			InnerClass i = new InnerClass();
			i.num = 20;
		}
		
		static class StaticNestedClass {
			private void init() {
				// static nested class에서는 OuterClass의 멤버변수에 접근할 수 없다. 
				// name = "test"; //Cannot make a static reference to the non-static field name
			}
		}
		class InnerClass {
			// InnerClass에서는 static 변수를 선언할 수 없다. 
			// private static String name; //The field name cannot be declared static in a non-static inner type, unless initialized with a constant expression
			public int num;
			
			private void init() {
				// InnerClass는 OuterClass의 멤버변수에 접근제한자에 제한없이 접근할 수 있다 
				name = "test";
				count = 10;
			}
		}

		// LocalClass : 블럭 안에 정의되는 LocalClass는 지역변수와 마찬가지로 블럭이 끝나면 소멸된다. 
		public void testLocalClass(String s1, String s2) {
			class LocalClass {
				String s1;
				String s2;
				
				public LocalClass(String s1, String s2) {
					this.s1 = s1;
					this.s2 = s2;
				}
			}
			LocalClass lc = new LocalClass(s1, s2);
		}	

		// AnonymousClass : LocalClass와 유사, 익명 클래스로 단 하나의 객체만 생성하는 일회용 클래스이다. 	
		interface AnonymousClass {
			public void getName(String name);
		}
		
		public void testAnonymousClass() {
			AnonymousClass ac = new AnonymousClass() {
				String someone = "";
				@Override
				public void getName(String name) {
					this.someone = name;
					System.out.println(this.someone);
				}
			}; 
		}
	}
	```
- InnerClass를 사용하는 이유
	- 논리적인 그룹화 : 특정 클래스에서만 사용되는 클래스를 논리적으로 그룹화하여 패키지를 간소화시킬 수 있다.
	- 캡슐화 : A 클래스의 private 멤버변수에 접근해야 하는 B 클래스가 있다고 가정했을 때 A의 InnerClass로 B를 만들면 A 클래스의 멤버변수를 public 또는 protected 등으로 변경하지 않고서도 사용할 수 있다.
	- 읽기 쉽고 관리하기 쉬운 코드 작성이 가능함

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

### Dynamic Method Dispatch
- Method Dispatch는 어떤 함수가 호출되어야 하는 지 결정하는 과정으로 컴파일 시점에 수행되면 static method dispatch, 런타임 시점에 수행되면 dynamic method dispatch라고 한다.
- 여기서 dynamic method dispatch는 다형성을 허용하는 개념이다.
- 대부분의 언어는 Virtual Method Table를 사용하여 dynamic dispatch를 구현한다.
- Virtual Method Table에는 각 클래스에서 호출되어야 하는 실제 함수의 주소가 매핑되어 있다.
- 객체가 생성되는 시점에 해당 객체가 참조할 Virtual Method Table을 찾아 객체의 메모리에 저장하는데, 이를 Virtual Table Pointer 또는 VPTR이라고 한다. 

```java

public class Base {
	void doSomething() {
		System.out.println("base..");
	}
}

public class Derived extends Base{
	void doSomething() {
		System.out.println("derived..");
	}
}

public static void main(String[] args) {
	Derived d = new Derived();
	d.doSomething(); //derived..
	
	Base b = new Base();
	b.doSomething(); //base..

	b = d;
	b.doSomething(); //derived..
}
```
