---
title: "Java Reflection"
categories:
  - java
tags:
  - java
  
toc: true
toc_icon: "cog"
toc_sticky: true
---

### [Reflection](https://www.oracle.com/technical-resources/articles/java/javareflection.html)
- Reflection은 클래스 및 인터페이스의 멤버변수, 함수 등의 런타임 속성을 검사하거나 수정할 수 있는 기능이다.
- 예를 들어 아래와 같은 코드를 통해 특정 클래스의 멤버 이름이나 함수 이름을 가져와서 출력할 수 있다.
  ```java
  import java.lang.reflect.Field;
  import java.lang.reflect.Method;

  public class Reflection {
      public static void main(String args[])
      {
        try {
            Class c = Class.forName(args[0]);
            Method m[] = c.getDeclaredMethods();
            Field f[] = c.getDeclaredFields();
            for (int i = 0; i < m.length; i++)
              System.out.println(m[i].toString());

            for (int i = 0; i < f.length; i++)
              System.out.println(f[i].toString()); 	  
        }
        catch (Throwable e) {
            System.err.println(e);
        }
      }
  }
  ```

- 클래스의 정보에 접근하려면 우선 대상 클래스에 대한 java.lang.Class 객체를 가져와야 한다. 
  ```
  // 방법 1
  Class c = Class.forName("java.lang.String");

  // 방법 2 (primitive type)
  Class c = int.class;
  Class c = Integer.TYPE;
  ```

### [Reflection API](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html)
- Class에서 제공되는 Reflection API를 통해 여러가지 정보를 얻을 수 있다. 
  ```
  //Class에서 제공하는 함수 중 일부...
  //Method, Field와 같은 reflection 관련 클래스는 java.lang.reflect 에 포함되어 있다.
  
  //클래스에 선언된 모든 public 함수(상속된 것도 public이면 포함) 배열을 리턴한다.
  Method[] getMethods()

  //클래스에 선언된 모든 함수(상속된 것은 제외) 배열을 리턴한다.
  Method[] getDeclaredMethods() 
  
  //클래스에 선언된 모든 생성자 배열을 리턴한다.
  Constructor<?>[] getDeclaredConstructors()

  //클래스에 선언된 모든 public 멤버변수(상속된 것도 public이면 포함) 배열을 리턴한다.
  Field[] getFields()

  //클래스에 선언된 모든 멤버변수(상속된 것은 제외) 배열을 리턴한다.
  Field[] getDeclaredFields() =>  Returns an array of objects reflecting all the fields declared by the class or interface represented by this object 
  ```   
  
### 주의사항
- Performance Overhead
  - Reflection은 동적으로 해결되는 타입이 포함되므로 특정 JVM에 최적화되지 않는다. 결과적으로 Reflection은 비Reflection에 비해 성능이 떨어진다. 
- Security Restrictions
  - Reflection은 실행 권한이 필요한데, security manager가 실행 중일 때는 이 권한이 없을 수 있다. 이는 애플릿 같이 제한된 보안 컨텍스트에서 실행해야 하는 코드에 대한 중요 고려 사항이다.
- Exposure of Internals
  - Reflection은 클래스의 private 멤버변수 및 함수에 접근할 수 있게 하여 코드가 제대로 동작하지 않는 등 예기치않은 부작용이 발생할 수 있다. 