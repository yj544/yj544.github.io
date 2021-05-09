---
title: "Java Exception Handling"
categories:
  - java
tags:
  - java
  
toc: true
toc_icon: "cog"
toc_sticky: true
---

### Java Exception
- Checked Exception
	- 컴파일 시 확인할 수 있는 예외를 말하며, 당연히 Checked Exception이 코드에 있다면 컴파일 에러가 발생한다. 
	- 위 그림에서 RuntimeException, Error를 제외한 Throwable 클래스를 직접 상속하는 IOException, SQLException, ClassNotFouncException 등이 포함된다.
- Unchecked Exception
	- RuntimeException을 상속하는 클래스가 여기에 포함된다. Unchecked Exception은 컴파일 시점에 알 수 없으며 런타임 시점에 발생한다.
- Error
	- 시스템 구동에 영향을 주는 치명적인 오류로 코드 레벨에서 대응이 불가능하다.

### Java Exception classes
- ![Exception Hierarchy](/image/java-exception-classes-hierarchy.png)
- Java에서 Exception 클래스는 계층 구조로 구성되어 있다.
- java.lang.Throwable 클래스가 최상위 클래스이며 크게 Exception 클래스와 Error 클래스로 나뉜다. 

```java
//1. ArithmeticException
// Thrown when an exceptional arithmetic condition has occurred. 
// For example, an integer "divide by zero" throws an instance of this class. 
// ArithmeticException objects may be constructed by the virtual machine as if suppression were disabled and/or the stack trace was not writable.
int a=50/0;

//2. NullPointerException  
// 1) Thrown when an application attempts to use null in a case where an object is required. These include:
// 2) Calling the instance method of a null object.
// 3) Accessing or modifying the field of a null object.
// 4) Taking the length of null as if it were an array.
// 5) Accessing or modifying the slots of null as if it were an array.
// 6) Throwing null as if it were a Throwable value.
String s=null;  
System.out.println(s.length());

//3. NumberFormatException
// Thrown to indicate that the application has attempted to convert a string to one of the numeric types, but that the string does not have the appropriate format.
String s="aa";  
int i=Integer.parseInt(s);

//4. ArrayIndexOutOfBoundsException 
//Thrown to indicate that an array has been accessed with an illegal index. The index is either negative or greater than or equal to the size of the array.
int a[]=new int[5];  
a[10]=50;  
```
### try-catch-finally
- try-catch-finally 블록에서 예외가 발생한 경우 finally가 실행되는 것은 보장하지만, try 또는 catch에서 return 문이 있는 경우 finally 구문에서 해당 값을 변경하는 것은 보장하지 않는다. 
- try 또는 catch 에서 return을 만나면 해당 값을 임시 변수에 담고 finally 구문을 수행한 이후에 return한다. 
	```java
		public int divide(int i, int j) {
			int result = 0;
			try {
				System.out.println("try start..");
				i = i/j;
				result = 1;
				return result; //예외가 발생하지 않으면 여기에서 리턴된다. 
			} catch (ArithmeticException e) {
				System.out.println("catch start..");
				result = 2;
				return result; //예외가 발생하면 여기에서 리턴된다. 
			} finally {
				System.out.println("finally start..");
				result = 3;
				//만약 finally 구문에 return이 있다면 finally의 return만 수행된다. try와 catch의 return은 절대 수행되지 않는다.
				//return result; 
			}

			//try에 return이 있는 경우 아래 코드는 컴파일 에러가 발생한다 : Unreachable code 
			//result = 4; 
			//return result;
		}
	```
	- return되는 result의 값은 1이다.
	- finally에서 result에 3을 할당하고 있지만, 이미 try 또는 catch에서 return을 만나면 return 값을 임시 변수에 담아두기 때문에 반영되지 않는다.
- 위 예시는 return하는 대상이 primitive type이었는데, 주소값인 경우는 결과가 달라질 수 있다.
	```java
	public class StringWrapper {
		private String str;

		public StringWrapper(String str) {
			this.str = str;
		}
		
		public String getStr() {
			return this.str;
		}
		
		public void setStr(String str) {
			this.str = str;
		}
	}

	public StringWrapper divide(int i, int j) {
		StringWrapper sw = new StringWrapper("0");
		
		try {
			System.out.println("try start..");
			sw.setStr("1");
			return sw;
		} catch (ArithmeticException e) {
			System.out.println("catch start..");
			return sw;
		} finally {
			System.out.println("finally start..");
			sw.setStr("3");
		}
	}
	```
	- return 되는 StringWrapper에 할당된 str은 "3"이다.

- finally 구문에서는 return하면 안 된다. 
	```java
	public int divide(int i, int j) {
		int result = 0;
		try {
			i = i/j;
			result = i;
			return result; //1. return 되지 않음
		} catch (ArithmeticException e) {
			//...
			throw new RuntimeException(); //2. 예외가 던져지지 않음
		} finally {
			result = 3;
			return result; //정상 리턴처럼 보임
		}
	}
	```
	- 1. 정상적으로 처리되는 경우 i 값이 리턴되는 것을 예상했지만 finally에서 3을 리턴하기 때문에 i 값은 리턴되지 않는다.
	- 2. 특정 예외를 잡아 부가적인 처리를 하고, 커스텀 예외를 던지려고 했지만 finally구문의 return에 의해 예외는 전혀 발생하지 않는다. 

### try-with-resources 
- java7부터 지원되는 기능으로 AutoCloseable 인터페이스를 구현한 클래스에만 해당된다. 
	```java
	public interface AutoCloseable {	
		void close() throws Exception;
	}
	```
- 대부분의 Stream 클래스는 AutoCloseable을 구현한다.
	```java
	public class FileInputStream extends InputStream { //public abstract class InputStream implements Closeable
		//..
		public void close() throws IOException {
			synchronized (closeLock) {
				if (closed) {
					return;
				}
				closed = true;
			}
			if (channel != null) {
			channel.close();
			}

			fd.closeAll(new Closeable() {
				public void close() throws IOException {
				close0();
			}
			});
		}
	}
	```
- 객체를 예외 발생 유무에 관계없이 자동으로 close하는데, 결국 AutoCloseable의 close 함수를 호출해주는 것이다.
	```java
	public class CustomResources implements AutoCloseable {
		@Override
		public void close() throws Exception {
			System.out.println("auto closed..");
		}	
		
		public void read() {
			System.out.println("read...");
		}

		public static void main(String[] args) throws RuntimeException{
			try (CustomResources cr = new CustomResources();){
				cr.read();
				int i = 20/0;
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	}
	// [output]
	// read...
	// auto closed..
	```

- 컴파일러가 적절한 위치에 close를 추가해주는 것이다.
	```
  public static void main(java.lang.String[]) throws java.lang.RuntimeException;
    Code:
       0: new           #6                  // class com/demo/exception/CustomResources
       3: dup
       4: invokespecial #7                  // Method "<init>":()V
       7: astore_1
       8: aconst_null
       9: astore_2
      10: aload_1
      11: invokevirtual #8                  // Method read:()V
      14: bipush        20
      16: iconst_0
      17: idiv
      18: istore_3
      19: aload_1
      20: ifnull        90
      23: aload_2
      24: ifnull        43
      27: aload_1
      28: invokevirtual #9                  // Method close:()V
      31: goto          90
      34: astore_3
      35: aload_2
      36: aload_3
      37: invokevirtual #11                 // Method java/lang/Throwable.addSuppressed:(Ljava/lang/Throwable;)V
      40: goto          90
      43: aload_1
      44: invokevirtual #9                  // Method close:()V
      47: goto          90
      50: astore_3
      51: aload_3
      52: astore_2
      53: aload_3
      54: athrow
      55: astore        4
      57: aload_1
      58: ifnull        87
      61: aload_2
      62: ifnull        83
      65: aload_1
      66: invokevirtual #9                  // Method close:()V
      69: goto          87
      72: astore        5
      74: aload_2
      75: aload         5
      77: invokevirtual #11                 // Method java/lang/Throwable.addSuppressed:(Ljava/lang/Throwable;)V
      80: goto          87
      83: aload_1
      84: invokevirtual #9                  // Method close:()V
      87: aload         4
      89: athrow
      90: goto          98
      93: astore_1
      94: aload_1
      95: invokevirtual #13                 // Method java/lang/Exception.printStackTrace:()V
      98: return
	```