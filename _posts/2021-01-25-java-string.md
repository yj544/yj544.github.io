---
title: "Java String 그리고 String Pool"
categories:
  - java
tags:
  - java
  - string
  - string_pool
  
toc: true
toc_icon: "cog"
toc_sticky: true
---

### Java String
Java에서 아주 많이 쓰이는 자료형인 String은 StringPool이라는 영역에서 관리된다.  
이는 리터럴을 통해 생성한 경우에 해당하는 말이며, new 를 통해 직접 생성하는 경우에는 StringPool이 아닌 Heap 영역에 할당된다.

```java
	String literal1 = "string";
	String literal2 = "string";
	String object = new String("string");
	
	System.out.println(literal1 == literal2); //true
	System.out.println(literal1 == object); // false
	System.out.println(literal1.equals(object)); //true
```

따라서 literal1, literal2 는 StringPool에 생성된 동일한 string을 참조하면서 값과 주소값 모두 같다.  
단 직접 생성자를 호출해서 생성한 object의 경우는 별도의 Heap 영역에 생성되면서 값은 같지만 주소값은 기존 lteral1, literal2와 다르다.  

### Literal
[리터럴](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html) 자바 기본형에 해당하는 특정 값을 표현하기 위해 사용하며 정수, 실수, 문자, 논리, 문자열 리터럴이 포함된다.

```java
boolean result = true; //논리 리터럴
char capitalC = 'C'; //문자 리터럴 -> '로 표현한다.
byte b = 100; //정수 리터럴 (int로 인식되고 byte로 형변환되어 할당된다.)
short s = 10000; //정수 리터럴 (int로 인식되고 short로 형변환되어 할당된다.)
int i = 100000; //정수 리터럴
long l = 1000000000000000000L; //정수 리터럴 (범위에 포함되는 값이지만 L을 붙이지 않으면 int로 인식해서 오류가 발생한다.)
String s = "string"; //문자열 리터럴 -> "로 표현한다.
```

### String intern
String을 리터럴로 선언하는 경우 String의 intern 함수가 호출된다.  
intern는 native 함수라서 구현체는 알 수 없지만 해당 함수 명세를 참조하면 다음과 같다 .
```
intern 함수가 호출되었을 때 String Pool에 해당 String과 같은(equlas 함수로 비교) String 객체가 있으면 해당 String을 반환한다. 
그렇지 않은 경우 String Pool에 String 객체를 새로 생성한 후 해당 객체를 반환한다. 
즉 s.equlas(t) 가 true 라면 s.intern() == t.inturn() 도 true이다. 
```

### String Pool
Java에서 String Pool은 HashTable으로 관리된다. 
Java6까지 String Pool은 PermGen 영역에 존재했으나 많은 문자열을 생성했을 때 Out Of Memory 문제가 발생할 여지가 있었다.    
PermGen 영역은 런타임 시점에는 크기를 확장할 수 없고 GC에 영향을 받지 않기 때문이다.   
따라서 Java7부터는 String Pool을 가변 크기로 운영할 수 있으면서 GC도 수행될 수 있는 Heap 영역으로 옮겼다.    

Java6에서 유일하게 String Pool에 의한 OOM을 막기 위한 방법은 PermGen을 늘리는 것이었다.  
```
-XX:MaxPermSize=1G
```

Java7부터는 String Pool의 사이즈만 늘리는 것도 가능하다.  
```
-XX:StringTableSize=60013
``` 

단, 값은 Hash 성능 향상을 위해 소수로 기입할 것을 권한다고 한다.  

** 사이즈가 소수인 것이 어떻게 HashTable의 성능에 영향을 미치는 지 궁금해서 확인해보았다.  
Hash를 사용하는 자료구조는 들어가는 데이터의 해시를 통해 데이터를 넣을 index를 구하는 방식인 건 아는데, 사이즈는 어떤 영향을 주는 걸까?  
데이터를 넣을 때마다 중복되는 key가 있는지 확인하는 containsKey 함수에서 index를 구하는 방식은 다음과 같다.  
```java
    public synchronized boolean containsKey(Object key) {
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length; //여기에서 length를 사용
        for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
            if ((e.hash == hash) && e.key.equals(key)) {
                return true;
            }
        }
        return false;
    }
```
key의 해시 값과 0x7FFFFFFF를 & 연산한 후 해당 값을 HashTable의 크기와 % 연산을 한다.    
도출한 index 값의 중복을 피하기 위해 그런 것 같다.    

### StringBuffer / StringBuilder
String은 리터럴로 선언하는 족족 String Pool에 생성되며 immutable 속성을 가져 멀티 쓰레드 환경에서도 안정적으로 사용할 수 있는 장점이 있다.  
다만 문자열 연산이 필요할 때 String과 연산자를 사용하는 것은 성능상 문제를 발생시킬 수 있다.
연산자 +를 통해 문자열을 이어서 String을 만든다고 가정했을 때 총 5개의 문자열이 만들어지는 것이다.  
```java
		String str = "a" + "b" + "c" + "d" + "e";
    //a, ab, abc, abcd, abcde 생성 => GC가 동작할 때 메모리에서 지워짐
```
따라서 문자열에 잦은 연산이 필요한 경우 StringBuffer 또는 StringBuilder를 사용할 것을 권한다.
StringBuffer와 StringBuilder는 가변 길이 배열을 사용하기 때문에 String과 달리 원본 문자열을 수정할 수 있다. 
```java
String str = builder.append("a").append("b").append("c").append("d").append("e").toString();
```