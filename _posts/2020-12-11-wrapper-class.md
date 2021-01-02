---
title: "Java Wrapper Class"
categories:
  - java
tags:
  - java
---

# 1. Java Wrapper Class
static factory method 에 대한 간단한 예시를 보고 문득 궁금해서 이것 저것 찾아보다 정리하는 글.
Java의 자료형은 [기본 타입(primitive type)](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html)과 참조 타입(reference type)이 있다. 
간혹 기본 타입을 객체로 표현해야 하는 경우가 있는데, 이를 위해 Java는 내부에 기본 타입을 멤버변수로 가지는 Wrapper Class를 제공한다. 
또한 기본 타입과 래퍼 클래스 간의 AutoBoxing(기본 타입 -> 래퍼 클래스), AutoUnBoxing(래퍼 클래스 -> 기본 타입)도 지원하여 마치 Wapper Class가 기본 타입인 것처럼 간편하게 사용할 수 있다.  

# 2. Java Wrapper Class 구현체
```java
public static Boolean valueOf(boolean b){
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```
생각없이 자주 쓰던 valueOf의 구현체를 보니 매개변수로 받은 boolean의 값에 따라 Boolean을 리턴하고 있다.
그럼 Boolean의 TRUE와 FALSE 객체는 미리 만들어 두는건가? 

```java
public final class Boolean implements java.io.Serializable, Comparable<Boolean>
{
    public static final Boolean TRUE = new Boolean(true);
    public static final Boolean FALSE = new Boolean(false);
}
```
맞다. 두 가지 모두 final로 미리 만들어두고 valueOf를 호출하면 해당 객체를 리턴한다.
그러면 너무 당연할 것 같은 테스트 

```java
		Boolean[] testBoolean = {
				new Boolean(false), //각각 다른 객체가 만들어지고 해당 객체를 각각 참조한다
				new Boolean(false),
				new Boolean(false)
		};
		
		Boolean[] testValueOf = {
				Boolean.valueOf(false), //다 같은 Boolean(false)를 참조한다
				Boolean.valueOf(false),
				Boolean.valueOf(false)
		};

		System.out.println(testBoolean[0] == testBoolean[2] ? "같다" : "다르다"); //다르다
		System.out.println(testBoolean[0].booleanValue() == testBoolean[2].booleanValue() ? "같다" : "다르다"); //같다

		System.out.println(testValueOf[0] == testValueOf[2] ? "같다" : "다르다"); //같다
		System.out.println(testValueOf[0].booleanValue() == testValueOf[2].booleanValue() ? "같다" : "다르다"); //같다
```

Boolean의 경우는 valueOf를 통해 얻은 객체 주소도 같기 때문에 == 비교도 가능하다!
그러면 다른 기본형의 Wrappdr Class는 어떨까?
```java

public final class Integer extends Number implements Comparable<Integer> {
    @Native public static final int   MIN_VALUE = 0x80000000;

    /**
     * A constant holding the maximum value an {@code int} can
     * have, 2<sup>31</sup>-1.
     */
    @Native public static final int   MAX_VALUE = 0x7fffffff;
    
    private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }

    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
}
```
일정 부분을 생략한 Integer 클래스를 가져온 것이다. 
여기서 조금 특징이 되는 부분은 IntegerCache 이다. Integer 클래스는 기본적으로 -128 부터 127까지의 수를 (java.lang.Integer.IntegerCache.high 프로퍼티가 없다면) 미리 메모리에 올려두고 사용한다. 
```java
Integer[] testInteger = {
        Integer.valueOf(1),
        Integer.valueOf(1),
        Integer.valueOf(3),
        Integer.valueOf(3),
        Integer.valueOf(-100),
        Integer.valueOf(-100),
        Integer.valueOf(200),
        Integer.valueOf(200)
};

System.out.println(testInteger[0] == testInteger[1] ? "같다" : "다르다"); //같다
System.out.println(testInteger[2] == testInteger[3] ? "같다" : "다르다"); //같다
System.out.println(testInteger[4] == testInteger[5] ? "같다" : "다르다"); //같다
System.out.println(testInteger[6] == testInteger[7] ? "같다" : "다르다"); //다르다
System.out.println(testInteger[6].intValue() == testInteger[7].intValue() ? "같다" : "다르다"); //같다
}
```
그래서 위와 같이 캐싱 처리되는 int 값 범위에서는 == 비교가 가능하지만 그 외 범위의 경우 ==로 비교할 수 없어진다. 
캐싱 대상 범위가 -128 ~ 127 인 것은 사용되는 대부분의 int 범위가 포함되기 때문일까? 
아무튼 일정 범위를 미리 메모리에 올려두는 Integer 클래스의 동작은 흥미로웠다.

다만 예상했던 것처럼 Integer 외에 Double, Float은 캐싱처리를 하지 않았다. 
```java
public static Double valueOf(String s) throws NumberFormatException {
    return new Double(parseDouble(s));
}
```



