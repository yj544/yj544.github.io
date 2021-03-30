---
title: "Java Annotation"
categories:
  - java
tags:
  - java
  
toc: true
toc_icon: "cog"
toc_sticky: true
---

### Annotation
- 소스코드에 추가해서 사용할 수 있는 메타 데이터의 일종으로 보통 @를 앞에 붙여서 사용하는데 JDK 1.5부터 지원되기 시작했다. 

### Annotation 정의
```java
public @interface Warn /*implements java.lang.annotation.Annotation*/{

}
```
- 클래스를 정의할 때 class를 붙이듯 Annotation을 정의하기 위해서는 @interface를 붙이면 된다. 
- 이렇게 정의된 Annotation은 java.lang.annotation.Annotation를 implements한다. 


## Meta Annotation
- Meta Annotation은 Annotation에만 추가할 수 있는 Annotation이며, 따라서 @Target(ElementType.ANNOTATION_TYPE)가 추가되어 있다. 
- @Retention
  ```java
  @Documented
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.ANNOTATION_TYPE)
  public @interface Retention {
       RetentionPolicy value();
  }

  public enum RetentionPolicy {
      SOURCE, //Annotation이 컴파일 시점에 삭제됨
      CLASS, //Annotation이 컴파일러에 의해 Class File에 기록되지만 런타임에는 유지되지 않음
      RUNTIME //Annotation이 컴파일러에 의해 Class File에 기록되며, 런타임에서도 VM에 의해 유지됨
  }
  ```
  - Annotation이 보존되는 기간을 나타내며 @Retention이 없는 Annotation의 경우 기본 값은 RetentionPolicy.CLASS이다. 


- @Documented
  ```java
  @Documented
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.ANNOTATION_TYPE)
  public @interface Documented {
  }
  ```
  - JavaDoc을 만들 때 Annotation 정보를 포함한다. 
    ```java
    @Documented
    public @interface Warn /*implements java.lang.annotation.Annotation*/{

    }

    @Warn
    public class Base {
    }
    ```
    - [annotation-documented](/image/annotation-documented.PNG)

- @Target
  ```java
  @Documented
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.ANNOTATION_TYPE)
  public @interface Target {
      //Annotation이 적용될 수 있는 대상 배열을 리턴하는 함수
      ElementType[] value();
  }

  public enum ElementType {
      TYPE,
      FIELD, 
      METHOD, 
      PARAMETER,
      CONSTRUCTOR,
      LOCAL_VARIABLE,
      ANNOTATION_TYPE,
      PACKAGE,
      TYPE_PARAMETER,
      TYPE_USE
  }
  ```
  - Annotation이 적용될 수 있는 대상을 제한하며, 만약 Target에서 허용하는 ElementType 외의 대상에 Annotation을 사용하는 경우 컴파일 에러를 발생시킨다.
    - ``The annotation @Warn is disallowed for this location``

- @Inherited
  ```java
  @Documented
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.ANNOTATION_TYPE)
  public @interface Inherited {
  }  
  ```
    - 상속 관계에서도 Annotation이 유지됨

## Built In Annotaiton
- Java에 내장된 Annotation을 말한다.
- @Override
  ```java
  @Target(ElementType.METHOD)
  @Retention(RetentionPolicy.SOURCE)
  public @interface Override {
  }
  ```
  - 함수가 오버라이딩된 함수임을 알려주는 Annotation이다.
  - 부모 클래스에 없는 함수 또는 Object 클래스의 public method에 없는 함수를 선언할 때 @Override를 추가한 경우 컴파일 에러를 발생시킨다.  
    ```java
    public class Base {
      protected int getInt() {
        return 10;
      }
    }

    public class Child extends Base {
      
      // 부모 클래스 함수 오버라이딩
      @Override
      protected int getInt() {
        return 20;
      }

      // Object 클래스 함수 오버라이딩
      @Override
      public String toString() {
        // TODO Auto-generated method stub
        return super.toString();
      }

      //Error : The method getLong() of type Child must override or implement a supertype method
      //@Override
      //protected long getLong() {
      // return 10L;
      //}
    }
    ```

- @Deprecated
  ```java
  @Documented
  @Retention(RetentionPolicy.RUNTIME)
  @Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
  public @interface Deprecated {
  }
  ```
  - 대상이 사용할 때 위험하거나 다른 대안을 사용하는 것이 좋은 경우에 추가하는 Annotation
  - 컴파일러는 @Deprecated가 있는 대상을 사용하거나, 오버라이드할 때 경고한다. 

- @SuppressWarnings
  ```java
  @Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
  @Retention(RetentionPolicy.SOURCE)
  public @interface SuppressWarnings {
      //deprecation : @Deprecated가 붙은 대상에 발생하는 경고
      //unchecked : 제네릭 타입을 지정하지 않을 때 발생하는 경고
      //rawtypes : 제네릭을 사용하지 않아서 발생하는 경고
      //varargs : 가변인자 타입이 제네릭 타입일 때 발생하는 경고
      String[] value();
  }
  ```
  - 컴파일러가 보여주는 경고를 강제로 보이지 않게함

- @SafeVarargs
  ```java
  @Documented
  @Retention(RetentionPolicy.RUNTIME)
  @Target({ElementType.CONSTRUCTOR, ElementType.METHOD})
  public @interface SafeVarargs {}
  ```
  - 가변 길이 매개변수를 사용하는 함수의 경우 컴파일러가 경고를 표시하는데, 해당 경고를 없애기 위해 사용한다.
    ```java
    public static <T> T[] unsafe(T... elements) { // Type safety: Potential heap pollution via varargs parameter elements
        return elements;
    }
    
    @SafeVarargs
    public static <T> T[] safe(T...  elements) {
      return elements; 
    }
    ```

- @FunctionalInterface
  ```java
  @Documented
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.TYPE)
  public @interface FunctionalInterface {}
  ```
  - 해당 인터페이스는 항상 함수형으로만 사용되는 인터페이스임을 명시하는 Annotation으로 1.8 부터 추가된 Annotation이다.
  - 함수형 인터페이스의 경우 개념상 하나의 abstract 함수만 포함해야 하는데, Object 클래스에 존재하는 함수의 abstract 선언은 있어도 카운팅하지 않는다. (구현체가 반드시 있기 때문에)
    ```java
    @FunctionalInterface
    public interface Base { 
      public abstract int getOne();
            
      //Invalid '@FunctionalInterface' annotation; Base is not a functional interface
      //public abstract int getTwo();
      
      @Override
      String toString();
    }
    ```
  - Runnable 인터페이스도 @FunctionalInterface가 추가되어 있다.
    ```java
    @FunctionalInterface
    public interface Runnable {
        public abstract void run();
    }
    ```

- @Repeatable
  ```java
  @Documented
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.ANNOTATION_TYPE)
  public @interface Repeatable {
      /**
      * Indicates the <em>containing annotation type</em> for the
      * repeatable annotation type.
      * @return the containing annotation type
      */
      Class<? extends Annotation> value();
  }
  ```
    - Annotation을 반복적으로 정의할 수 있게 하는 Annotation ([Repeating Annotations](https://docs.oracle.com/javase/tutorial/java/annotations/repeating.html))

### Annotation Processor 
- [참고](https://www.baeldung.com/java-annotation-processing-builder)
- Annotation Processor는 컴파일 시점에 Annotation들을 찾아서 처리하는 기능을 한다. 

#### 1) AbstractProcessor 
- AbstractProcessor는 Annotation Processor를 만드는 데 도움을 주기위해 만들어진 추상 클래스이며, Annotation Processor는 AbstractProcessor를 상속받아야 한다.
  ```java
  // annotations : annotation 데이터들 
  // roundEnv : 현재 및 이전 라운드에 대한 정보
  // return : 
  public abstract boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv);
  ```
  - 컴파일러가 매칭되는 Annotation이 포함된 모든 소스파일에 대해 process 함수를 호출한다. 
  - Annotation Processor는 process에서 전달받은 annotation을 읽고 처리하여 출력파일을 생성하는 작업을 한다.

- lombok을 보면 lombok.launch.AnnotationProcessorHider의 AnnotationProcessor가 AbstractProcessor를 상속받는다.
  ```java
  class AnnotationProcessorHider {

    public static class AstModificationNotifierData {
      public volatile static boolean lombokInvoked = false;
    }
    
    public static class AnnotationProcessor extends AbstractProcessor {
      private final AbstractProcessor instance = createWrappedInstance();
      
      @Override public Set<String> getSupportedOptions() {
        return instance.getSupportedOptions();
      }
      
      @Override public Set<String> getSupportedAnnotationTypes() {
        return instance.getSupportedAnnotationTypes();
      }
      
      @Override public SourceVersion getSupportedSourceVersion() {
        return instance.getSupportedSourceVersion();
      }
      
      @Override public void init(ProcessingEnvironment processingEnv) {
        disableJava9SillyWarning();
        AstModificationNotifierData.lombokInvoked = true;
        instance.init(processingEnv);
        super.init(processingEnv);
      }
      
      @SuppressWarnings({"sunapi", "all"})
      private void disableJava9SillyWarning() {
        try {
          Field theUnsafe = Unsafe.class.getDeclaredField("theUnsafe");
          theUnsafe.setAccessible(true);
          Unsafe u = (Unsafe) theUnsafe.get(null);
          
          Class<?> cls = Class.forName("jdk.internal.module.IllegalAccessLogger");
          Field logger = cls.getDeclaredField("logger");
          u.putObjectVolatile(cls, u.staticFieldOffset(logger), null);
        } catch (Throwable t) {
          
        }
      }
      
      @Override public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        return instance.process(annotations, roundEnv);
      }
      
      @Override public Iterable<? extends Completion> getCompletions(Element element, AnnotationMirror annotation, ExecutableElement member, String userText) {
        return instance.getCompletions(element, annotation, member, userText);
      }
      
      private static AbstractProcessor createWrappedInstance() {
        ClassLoader cl = Main.getShadowClassLoader();
        try {
          Class<?> mc = cl.loadClass("lombok.core.AnnotationProcessor");
          return (AbstractProcessor) mc.getDeclaredConstructor().newInstance();
        } catch (Throwable t) {
          if (t instanceof Error) throw (Error) t;
          if (t instanceof RuntimeException) throw (RuntimeException) t;
          throw new RuntimeException(t);
        }
      }
    }

    @SupportedAnnotationTypes("lombok.*")
	  public static class ClaimingProcessor extends AbstractProcessor {
      @Override public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        return true;
      }
      
      @Override public SourceVersion getSupportedSourceVersion() {
        return SourceVersion.latest();
      }
    }
  }
  ```
    - 코드를 보면 실제 프로세서는 ``lombok.core.AnnotationProcessor``인 것 같은데, 해당 클래스가 포함된 lombok.core 패키지 하위 파일들은 모두 SCL.lombok 확장자를 가지고 있고, 파일 내용을 열어보면 바이트코드 같아보이는데, 확장자 때문인지 디컴파일된 코드가 보이지는 않는다.
    - [공식 문서](https://projectlombok.org/contributing/lombok-execution-path)를 참고하면 IDE에서 lombok의 코드가 보이는 것을 방지하기 위해 이런 별도의 확장자를 사용한다고 한다.

#### Annotation Processor 등록
- 만들어진 Annotation Processor를 컴파일 단계에서 사용하기 위한 방법에는 크게 세 가지가 있다.
- Annotation Processor Tool (APT) 사용 
  - 1.5부터 지원되다가 1.7 부터 사라진 기능이라서 제외
- 컴파일러 키 사용
  - ``-processor``를 통해 Annotation Processor를 지정할 수 있다. 
  ```
  javac -processor package1.Processor1,package2.Processor2 SourceFile.java
  ```
- Maven 사용
  - maven-compiler-plugin을 통해 Annotation Processor를 지정할 수 있다.
  ```xml
  <build>
      <plugins>
          <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-compiler-plugin</artifactId>
              <version>3.5.1</version>
              <configuration>
                  <source>1.8</source>
                  <target>1.8</target>
                  <encoding>UTF-8</encoding>
                  <generatedSourcesDirectory>${project.build.directory}
                    /generated-sources/</generatedSourcesDirectory>
                  <annotationProcessors>
                      <annotationProcessor>
                          com.annotation.processor.BuilderProcessor
                      </annotationProcessor>
                  </annotationProcessors>
              </configuration>
          </plugin>
      </plugins>
  </build>  
  ```
- Processor를 Classpath에 등록
  - Processor로 사용할 대상을 META-INF/services/javax.annotation.processing.Processor에 기입하는 방식
  - lombok의 경우 lombok-{version}.jar에 다음과 같이 설정되어 있다. 
    ```
    lombok.launch.AnnotationProcessorHider$AnnotationProcessor
    lombok.launch.AnnotationProcessorHider$ClaimingProcessor
    ```