---
title: "Java Class Loader"
categories:
  - java
tags:
  - java
---

# 1. Class Loader
Class Loader는 자바 클래스 파일을 컴파일 시점이 아닌 런타임 시점에 JVM에 로드하는 주체이며 JRE에 포함되어 있다.
자바 코드(.java)를 컴파일하면 JVM에서 실행 가능한 클래스 파일(.class)형태가 되며, JVM이 해당 클래스를 실행하기 위해서는 로딩하는 과정이 필요하다.

# 2. java.lang.ClassLoader (rt.jar) 
    자바에서 제공되는 모든 클래스 로더는 ClassLoader라는 추상 클래스를 통해 (또는 상속받아) 표현된다.
    모든 클래스는 getClassLoader라는 함수를 가지며, 이는 해당 클래스를 로드하는 클래스 로더를 참조한다.

    ```java
		System.out.println(ClassLoaderTest.class.getClassLoader()); //sun.misc.Launcher$AppClassLoader@4e0e2f2a
		System.out.println(Translator.class.getClassLoader()); //sun.misc.Launcher$ExtClassLoader@4e25154f
		System.out.println(Translator[].class.getClassLoader()); //sun.misc.Launcher$ExtClassLoader@4e25154f
		System.out.println(ArrayList.class.getClassLoader()); //null : bootstrap class loader
		System.out.println(int.class.getClassLoader()); //null
    ```
  ### (1) loadClass
    ```java
    public Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {};
    ```    
    클래스 로더의 진입점이며 클래스를 로딩하는 역할을 하는 함수다.
    name 파라미터로 로드 대상 클래스의 전체 이름을 전달받고 resolve 파라미터로 클래스 간의 참조 관계 설정 유무를 체크한다.
    다만 resolve 파라미터는 항상 true일 필요는 없다. 만약 단순히 클래스의 존재 유무만 확인하려는 경우 false로 설정되어도 무방하다.
    실제 ClassLoader 클래스의 loadClass 구현부는 아래와 같다.

    ```java
    protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name); //1. 이미 대상 클래스가 로드된 상태인지 확인한다.
            if (c == null) { //2. 대상 클래스가 로드되지 않는 상태이면 
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false); //2-1. 클래스 로더의 parent를 확인하여 null이 아니면 parent에게 위임
                    } else {
                        c = findBootstrapClassOrNull(name); //2-2. 클래스 로더의 parent가 null이면 최상위이므로 BootstrapClassloader를 찾음
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name); //3. 이후에도 못 찾은 경우 findClass 호출

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
    ```

  ### (2) defineClass
    ```java
    protected final Class<?> defineClass(String name, byte[] b, int off, int len, ProtectionDomain protectionDomain)
    ```
    바이트 배열을 클래스의 인스턴스로 변환해주는 역할을 하는 함수이며 final로 정의되어 재정의가 불가하다.
    클래스가 로드되기 전에 호출되어야 하는데, 바이트 배열에 유효한 클래스가 없으면 ClassFormatError를 던진다.

  ### (3) findClass
    ```java
    protected Class<?> findClass(String name) throws ClassNotFoundException {
      throw new ClassNotFoundException(name);
    }   
    ```
    name 파라미터로 전달받은 전체 클래스 명을 통해 클래스를 찾는 함수다.
    커스텀 클래스 로더를 작성하는 경우 반드시 해당 메서드를 오버라이딩 해야한다. 
    이는 클래스 로더의 특징인 위임 모델을 따르기 위함이며, loadClass에서 부모 클래스 로더로 클래스 로딩을 위임했을 때(2-1.) 클래스를 찾지 못하면 findClass를 호출한다. 

  ### (4) getParent
    ```java
      public final ClassLoader getParent() {
        if (parent == null)
            return null;
        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            // Check access to the parent class loader
            // If the caller's class loader is same as this class loader,
            // permission check is performed.
            checkClassLoaderPermission(parent, Reflection.getCallerClass());
        }
        return parent;
    }  
    ```
    부모 클래스 로더를 리턴하는 함수이며 클래스 로딩 과정을 위임하기 위해 호출한다. 

  ### (5) getResource
    ```java
    public URL getResource(String name) {
        URL url;
        if (parent != null) {
            url = parent.getResource(name); //부모 클래스 로더가 있는 경우 부모에게 리소스 찾는 작업을 위임
        } else {
            url = getBootstrapResource(name); //부모 클래스 로더가 없는 경우 최상위 클래스 로더이므로 BoootstrapClassLoader를 통해 수행
        }
        if (url == null) {
            url = findResource(name); //부모 클래스 로더에서 리소스를 못찾으면 findResource 호출
        }
        return url;
    }
    ```
    name 파라미터에 해당하는 리소스를 찾는 함수이다. (Java는 Class Path를 기준으로 리소스를 로드한다.)

# 3. 클래스 로더 종류
 - Bootstrap Class Loader
   Bootstrap 클래스 로더는 $JAVA_HOME/jre/lib 디렉터리에 존재하는 rt.jar를 포함한 핵심 라이브러리들을 로드하며, 모든 클래스 로더의 최상위 부모이기도 하다.
   Native Code로 작성되어 JVM안에 포함된 형태로 배포된다. 

   - rt.jar 
     Thread, ArrayList 등과 같은 핵심 자바 클래스를 포함하는 jar
     다른 클래스 파일에 하는 엄격한 보안 체크를 하지 않고, JVM으로부터 신뢰받는 클래스 파일만 포함된다고 가정함

 - Extention Class Loader
   Bootstrap 클래스 로더의 자식이며 $JAVA_HOME/lib/ext 경로에 있는 클래스 및 java.ext.dirs 시스템 변수에 있는 클래스를 로드한다.

 - System & Application Class Loader
   JVM 내 모든 어플리케이션 레벨 클래스 로딩을 담당한다. 
   System 클래스 : CLASSPATH에 정의되거나 JVM 옵션에서 -cp, -classpath에 지정된 클래스

# 4. 클래스 로딩 과정
  클래스 로더는 위임 모델을 통해 클래스를 찾는다. 
  각 클래스 로더 인스턴스는 연결된 부모 클래스 로더가 있고, 최상위 클래스 로더는 Bootstrap 클래스 로더가 된다.
  이 때, 클래스 로딩 요청이 발생하면 클래스 로더 인스턴스는 부모 클래스 로더에게 클래스 찾는 작업을 위임한다. 
  부모 클래스 로더가 클래스를 못 찾은 경우에 자식 클래스 로더가 클래스를 탐색하는 구조이며, 결과적으로 모든 자식 클래스 로더까지 해당 클래스를 못 찾는 경우 ``ClassNotFoundException``이 발생한다.

# 5. 특징
  ### (1) 위임 모델 (Delegation Model)
    클래스 로딩 과정에서 말한 것 같이 클래스 로더는 대상 클래스를 찾을 때 기본적으로 위임 모델을 사용한다.
    일반적인 환경에서는 클래스 로더가 Application Class Loader -> Extention Class Loader -> Bootstrap Class Loader 의 관계로 부모를 참조하고 있을 것이다.
    이 경우 클래스 로드 요청이 발생하면 각 클래스 로더는 자신의 부모에게 처리를 위임하고, 결국 최상위 부모인 Bootstrap 클래스 로더부터 클래스를 찾는다.

  ### (2) 클래스 유일성 보장 (Unique Classes) 
    위임 모델을 통해 하위 클래스 로더는 부모 클래스 로더가 대상 클래스를 찾지 못한 경우에만 클래스 로딩 작업을 진행하므로 결과적으로 클래스의 유일성을 보장해준다. 
  
  ### (3) 가시성 (Visibility)
    하위 클래스 로더는 상위 클래스 로더가 로딩한 클래스를 볼 수 있고, 상위 클래스 로더는 하위 클래스 로더가 로드한 클래스를 볼 수 없다.
    예를 들어, System 클래스 로더에서 로드된 클래스는 Extension 클래스 로더 또는 Bootstrap 클래스 로더에서 로드된 클래스를 볼 수 있지만 그 반대는 성립하지 않는다.

  ### (4) 언로드 불가
    클래스 로더로 로딩한 클래스는 언로딩 할 수 없으며, 오직 GC 동작에 의한 언로드 또는 WAS 재시작 시에만 초기화된다.

# 6. ServletContextClassLoader / ServletContainerClassLoader
웹의 경우 ServletContextClassLoader, ServletContainerClassLoader 최소 두 개의 클래스 로더가 추가로 존재한다.
ServletContextClassLoader의 경우 웹 애플리케이션 자체 API를 제공하기 위해 컨테이너를 로드하는 역할을 하는 클래스 로더이다.
ServletContainerClassLoader는 사용자가 추가한 JSP나 WAR 파일을 로드하기 위한 클래스 로더이며 WEB-INF/classes 파위 파일을 먼저 탐색하고 이후 WEB-INF/libs에 있는 jar 파일을 탐색한다. 
