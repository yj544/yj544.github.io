---
title: "EffectiveJava #9 Clone"
categories:
  - java
tags:
  - java
  - effectivejava
  
toc: true
toc_icon: "cog"
toc_sticky: true
---

#### Effective Java #13 Clone 재정의는 주의해서 진행하라

##### Cloneable
Cloneable은 어떤 객체가 clone을 허용한다는 것을 명시할 용도로 고안된 인터페이스이다.

- java.lang.Cloneable
```java
public interface Cloneable {
}
```

단, 실제 clone을 담당하는 함수는 Cloneable이 아닌 Object에 있으며 protected 이다.
```java
public class Object {
  ...
  protected native Object clone() throws CloneNotSupportedException;
  ...
}  
```

Object 클래스의 clone 함수에 적힌 명세는 다음과 같다.
- clone 함수가 호출된 객체가 Cloneable 인터페이스를 implement하지 않았으면 CloneNotSupportedException를 던진다. 
- clone 함수는 객체를 복사(여기서 복사의 의미는 클래스마다 다르다고 함)해서 리턴하는 함수이다.
- x.clone() != x 는 참이다.
- x.clone().getClass() == x.getClass() 는 대체로 참이겠지만 반드시 참일 필요는 없다.
- x.clone().equals(x) 는 대체로 참이겠지만 반드시 참일 필요는 없다.
- 깊은 복사가 아닌 얕은 복사를 한다. 따라서 멤버 변수의 메모리도 별도로 복사해야 할 수 있다.
- 어떤 생성자도 호출되지 않는다.

##### Cloneable 적용해보기
Cloneable 인터페이스를 구현한 클래스는 public clone 함수를 제공해야 한다.
따라서 예시로 CloneableItem 클래스를 만들고 clone을 오버라이드했다.

```java
public class CloneableItem implements Cloneable{
	String name;
	int count;
	ArrayList<String> place = new ArrayList<String>();

	public CloneableItem(String name, int count, ArrayList<String> place) {
		this.name = name;
		this.count = count;
		this.place = place;
	}
	
	@Override
	public CloneableItem clone() throws CloneNotSupportedException {
		return (CloneableItem)super.clone();
	}
	
	public void addPlace(String place) {
		this.place.add(place);
	}
		
	public void setName(String name){
		this.name = name;
	}
  
	@Override
	public String toString() {
		return "CloneableItem [name=" + name + ", count=" + count + ", place=" + place + "]";
	}
}
```
** Object의 clone 함수는 Object를 리턴하지만, CloneableItem 클래스는 CloneableItem를 리턴한다.
** 이는 오버라이딩 함수의 리턴 타입은 기존 함수의 리턴 타입의 하위 클래스도 될 수 있게 하는 제네릭의 일종인 Covariant return type 때문에 가능하다.

그리고 해당 클래스를 통해 몇가지 테스트를 해봤다.

```java
  ArrayList<String> place = new ArrayList<>();
  place.add("desk");
  place.add("bag");
  
  CloneableItem pen = new CloneableItem("pen", 3, place);
  CloneableItem anotherPen = (CloneableItem)pen.clone();
  
  System.out.println(pen.toString()); //CloneableItem [name=pen, count=3, place=[desk, bag]]
  System.out.println(anotherPen.toString()); //CloneableItem [name=pen, count=3, place=[desk, bag]]
  
  pen.addPlace("table");
  pen.setName("colorPen");
  
  System.out.println(pen.toString()); //CloneableItem [name=colorPen, count=3, place=[desk, bag, table]]
  System.out.println(anotherPen.toString()); //CloneableItem [name=pen, count=3, place=[desk, bag, table]]
```

pen 객체의 name과 place를 변경했는데, anotherPen의 place도 같이 변경이 됐다.
디버깅을 해보면 쉽게 알 수 있는 부분이긴 한데 pen과 anotherPen의 멤버변수인 place는 같은 객체를 참조한다. (얕은 복사)
**그 외 별도의 메모리를 할당하는 primitive type과 별도의 Pool로 관리되는 String은 정상적으로 하나의 객체만 변경됨
따라서 place에 대해서도 정상적으로 복사가 되게 하려면 ArrayList의 clone 함수를 별도로 호출해야 한다. 

```java
	@Override
	public CloneableItem clone() throws CloneNotSupportedException {
		CloneableItem cloneItem = (CloneableItem)super.clone();
		cloneItem.place = (ArrayList)this.place.clone();
		return cloneItem;
	}
```

하지만 이런 방식으로 깊은 복사를 만드는 경우 해당 필드 (위 예시에서는 place)가 final이 될 수 없다. 

마찬가지로 리스트와 같은 참조를 참조하는 구조를 복사해야 하는 경우, 재귀로 해당 원소의 clone을 호출하는 방식을 사용할 수 있는데 이는 좋은 방식이 아니다.
함수를 호출하는 것 자체가 스택을 하나 생성하는 행위이고, 만약 어마어마하게 긴 리스트를 재귀로 복사하게 되면 스택 오버플로가 발생할 수 있다.
이 때는 반복문을 통해 복사하는 것이 낫다.

책에서는 clone 함수는 위험해보인다고 표현하며, 이보다 복사 생성자나 복사 팩터리를 제공하는 것이 객체 복제를 지원하는 좋은 방법이라고 설명한다. 
(사실 clone은 어떻게 구현되어있는지 알 수 없을 뿐더러, 관련 명세도 많지 않다. 심지어 clone이 구현된 클래스는 복사 대상 클래스가 아니라는 것 등등 석연찮은 점이 꽤 있다.)

##### 복사 생성자와 복사 팩터리
복사 생성자는 클래스 객체를 파라미터로 전달받는 생성자를 말하며 아래와 같이 구현될 수 있다.
```java
	public CloneableItem(CloneableItem item) {
		this.name = item.name;
		this.count = item.count;
		this.place = new ArrayList<String>(item.place);
	}
```

기존의 예시를 똑같이 수행해보자.
당연한 말이지만 객체를 복사할 때 새로운 객체를 생성했기 때문에 정상적인 결과가 나온다.
```java
	public static void main(String[] args) throws Exception {
		ArrayList<String> place = new ArrayList<>();
		place.add("desk");
		place.add("bag");
		
		CloneableItem pen = new CloneableItem("pen", 3, place);
		CloneableItem anotherPen = new CloneableItem(pen);
		
		System.out.println(pen.toString()); //CloneableItem [name=pen, count=3, place=[desk, bag]]
		System.out.println(anotherPen.toString()); //CloneableItem [name=pen, count=3, place=[desk, bag]]

		pen.addPlace("table");
		pen.setName("colorPen");
		
		System.out.println(pen.toString()); //CloneableItem [name=colorPen, count=3, place=[desk, bag, table]]
		System.out.println(anotherPen.toString()); //CloneableItem [name=pen, count=3, place=[desk, bag]]
	}
```

복사 팩터리는 복사 생성자와 유사하게 클래스 객체를 파라미터로 전달받는 정적 팩터리 메서드이다.
```java
	public static CloneableItem newInstance(CloneableItem item) {
		CloneableItem copyItem = new CloneableItem();
		copyItem.name = item.name;
		copyItem.count = item.count;
		copyItem.place = new ArrayList<String>(item.place);
		return copyItem;
	}
```