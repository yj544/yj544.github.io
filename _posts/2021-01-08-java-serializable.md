---
title: "Java Serializable (직렬화)"
categories:
  - java
tags:
  - java
  
toc: true
toc_icon: "cog"
toc_sticky: true
---

### [Java Object Serialization Specification](https://docs.oracle.com/javase/8/docs/platform/serialization/spec/serial-arch.html) 
- 객체를 바이트 형태의 데이터로 변환하거나(직렬화) 반대로 바이트 형태의 데이터를 객체로 변환하는 기술(역직렬화)
- 직렬화 조건
  - Serializable 인터페이스를 상속받고, serialVersionUID 를 가지고 있어야 한다.
    ```java
    public class Date implements java.io.Serializable, Cloneable, Comparable<Date> {
      private static final long serialVersionUID = 7523967970034938905L;
        //...
    }
    ```
  - serialVersionUID가 없는 경우 기본 값이 할당되긴 하지만 해당 값은 클래스 세부 정보나 컴파일러에 따라 달라질 수 있으므로 별도로 선언하는 것이 좋다. 
    ```
    However, it is strongly recommended that all serializable classes explicitly declare serialVersionUID values, 
    since the default serialVersionUID computation is highly sensitive to class details that may vary depending on compiler implementations, 
    and can thus result in unexpected
    ```

- 직렬화
  ```java
	public static class Test implements Serializable {
		public int i;
		public String s;
		public Double d;
		
		public Test(int i, String s, Double d) {
			this.i = i;
			this.s = s;
			this.d = d;
		}
	}

  byte[] serialized;
  Test t = new Test(1, "s", Double.valueOf(1.1d));
  try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
    try (ObjectOutputStream oos = new ObjectOutputStream(baos)) {
        oos.writeObject(t);
        serialized = baos.toByteArray();
        System.out.println(Base64.getEncoder().encodeToString(serialized));
    }
  }
  ```

  ```
  rO0ABXNyABZTZXJpYWxpemF0aW9uVGVzdCRUZXN0smn1DgW6qkkCAANJAAFpTAABZHQAEkxqYXZhL2xhbmcvRG91YmxlO0wAAXN0ABJMamF2YS9sYW5nL1N0cmluZzt4cAAAAAFzcgAQamF2YS5sYW5nLkRvdWJsZYCzwkopa/sEAgABRAAFdmFsdWV4cgAQamF2YS5sYW5nLk51bWJlcoaslR0LlOCLAgAAeHA/8ZmZmZmZmnQAAXM=
  ```

- 역직렬화
  ```java
  try (ByteArrayInputStream bais = new ByteArrayInputStream(Base64.getDecoder().decode(serialized))) {
    try (ObjectInputStream ois = new ObjectInputStream(bais)) {
      Test t = (Test)ois.readObject();
      System.out.println(t.i + "," + t.s + "," + t.d);
    }
  }
  ```

- 직렬화 대상 클래스가 Serializable 인터페이스를 상속받지 않는 경우 에러 발생
  ```
  java.io.NotSerializableException: SerializationTest$Test
    at java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1184)
    at java.io.ObjectOutputStream.writeObject(ObjectOutputStream.java:348)
    at SerializationTest.main(SerializationTest.java:20)
  ```
    
- 직렬화된 대상 클래스에 변경이 발생하면 serialVersionUID 값이 변경됨
  - 멤버변수 또는 상수를 추가, 제거하거나 생성자에 들어오는 파라미터 이름만 바뀌어도 serialVersionUID 값은 변경된다.
    ```java
    java.io.InvalidClassException: SerializationTest$Test; local class incompatible: 
    stream classdesc serialVersionUID = -5590668021829293495, local class serialVersionUID = 5801329592937560658
      at java.io.ObjectStreamClass.initNonProxy(ObjectStreamClass.java:699)
      at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:2003)
      at java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1850)
      at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2160)
      at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1667)
      at java.io.ObjectInputStream.readObject(ObjectInputStream.java:503)
      at java.io.ObjectInputStream.readObject(ObjectInputStream.java:461)
      at SerializationTest.main(SerializationTest.java:44)
    ```
