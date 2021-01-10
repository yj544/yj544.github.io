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

#### Java Serializable (정리 중..)
##### 자바 직렬화 
https://woowabros.github.io/experience/2017/10/17/java-serialize.html
https://docs.oracle.com/javase/6/docs/platform/serialization/spec/class.html#4100

자바 직렬화란 자바 시스템 내부에서 사용되는 객체 또는 데이터를 외부의 자바 시스템에서도 사용할 수 있도록 하는 기술을 말한다.
기존 데이터를 바이트 형태로 변환하는 직렬화 기술과 바이트를 데이터로 변환하는 역직렬화를 아우르는 개념이라고 보면 된다.

직렬화를 하기 위해서는 단순히 ``java.io.Serializable`` 인터페이스를 구현하면 된다.
```java
import java.io.Serializable;

public class Member implements Serializable{

	private static final long serialVersionUID = 2125778954912190570L;
	
	String name;
	int age;
	String phoneNumber;
	
	public Member(String name, int age, String phoneNumber) {
		this.name = name;
		this.age = age;
		this.phoneNumber = phoneNumber;
	}

	@Override
	public String toString() {
		return "Member [name=" + name + ", age=" + age + ", phoneNumber=" + phoneNumber + "]";
	}
}
```


```java
  Member member = new Member("John", 32, "010-0000-0000");
  
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    ObjectOutputStream oos = new ObjectOutputStream(baos);
            
    oos.writeObject(member);
    byte[] serializedMember = baos.toByteArray();
  
  System.out.println("serialized member : " + Base64.getEncoder().encodeToString(serializedMember));
  
  ByteArrayInputStream bais = new ByteArrayInputStream(serializedMember);
  ObjectInputStream ois = new ObjectInputStream(bais);
  
  Member deserializedMember = (Member)ois.readObject();
  System.out.println("deserialized member : " + deserializedMember.toString());
```

Member 클래스가 Serializable를 상속받지 않고 직렬화를 시도하는 경우 아래와 같은 오류가 발생함
Exception in thread "main" java.io.NotSerializableException: Member
	at java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1184)
	at java.io.ObjectOutputStream.writeObject(ObjectOutputStream.java:348)
	at CloneableTest.main(CloneableTest.java:30)

  