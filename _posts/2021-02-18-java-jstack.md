---
title: "jstack JVM Thread Dump"
categories:
  - java
tags:
  - java
  - jvm
  - jstack
  - dump
  
toc: true
toc_icon: "cog"
toc_sticky: true
---

### [jstack](https://docs.oracle.com/javase/7/docs/technotes/tools/share/jstack.html)
Linux 환경에서 서비스가 응답하지 않는 현상으로 ``jstack -l``로 스택 정보를 수집하려고 했으나 아래와 같은 오류가 발생했다.
```
Unable to open socket file: target process not responding or HotSpot VM not loaded
The -F option can be used when the target process is not responding
```

위 내용에서 말하는 것처럼 ``jstack -F``로 스택 정보를 수집하니 정상적으로 수집이 되었는데, 수집된 내용이 약간 상이했다. 
그래서 수집 방식이 옵션에 따라 어떻게 달라지는지 정리해보았다.

### 사용 방법
- 명령어
```
jstack [ option ] pid (스택 정보를 수집하고자 하는 프로세스의 id)
jstack [ option ] executable core
jstack [ option ] [server-id@]remote-hostname-or-IP
```

- 옵션
   - F : l 옵션이 응답하지 않는 경우 강제로 덤프를 수집하는 옵션
   - l : 동기화 등 부가적인 정보를 함께 수집하는 옵션
   - m : mixed mode
   - h / help

### JStack main function
- 아래 내용은 Openjdk 1.8 Linux로 확인한 내용이다.
- 1) tools.jar > sun.tools.jstack.JStack
   ```java
   public class JStack
   {
   public static void main(String[] paramArrayOfString) throws Exception {
      //...
      int i = 0;
      while (i < paramArrayOfString.length) {
         String str = paramArrayOfString[i];
         if (!str.startsWith("-")) {
            break;
         }
         if (str.equals("-help") || str.equals("-h")) {
            usage(0);
         }
         else if (str.equals("-F")) {
            bool = true;
         }
         else if (str.equals("-m")) {
            bool1 = true;
         }
         else if (str.equals("-l")) {
            bool2 = true;
         } else {
            usage(1);
         } 
         i++;
      } 

      //...
      if (bool) {
         String[] arrayOfString = new String[j];
         for (int k = i; k < paramArrayOfString.length; k++) {
            arrayOfString[k - i] = paramArrayOfString[k];
         }
         runJStackTool(bool1, bool2, arrayOfString);
      } else {
         String arrayOfString[], str = paramArrayOfString[i];
         if (bool2) {
            arrayOfString = new String[] { "-l" };
         } else {
            arrayOfString = new String[0];
         } 
         runThreadDump(str, arrayOfString);
      } 
   }
   ```
   - 파라미터(l, F, m)에 따라서 구동되는 함수가 달라진다.
      - l => runThreadDump
      - F/m => runJStackTool

### option -l 
- 2) runThreadDump (sun.tools.jstack.JStack)
   ```java
   private static void runThreadDump(String paramString, String[] paramArrayOfString) throws Exception {
      int i;
      VirtualMachine virtualMachine = null;
      try {
         virtualMachine = VirtualMachine.attach(paramString); 
      } catch (Exception exception) {
         String str = exception.getMessage();
         if (str != null) {
            System.err.println(paramString + ": " + str);
         } else {
            exception.printStackTrace();
         } 
         if (exception instanceof com.sun.tools.attach.AttachNotSupportedException && loadSAClass() != null) {
            System.err.println("The -F option can be used when the target process is not responding");
         }
         System.exit(1);
      }
      InputStream inputStream = ((HotSpotVirtualMachine)virtualMachine).remoteDataDump((Object[])paramArrayOfString);
      byte[] arrayOfByte = new byte[256];
      
      do {
         i = inputStream.read(arrayOfByte); //덤프 내용을 읽어서 
         if (i <= 0) continue;  String str = new String(arrayOfByte, 0, i, "UTF-8");
         System.out.print(str); //출력
      }
      while (i > 0);
      inputStream.close();
      virtualMachine.detach();
   }
   ```

- 3) attach (tools.jar > com.sun.tools.attach.VirtualMachine)
   ```java
   public static VirtualMachine attach(String paramString) throws AttachNotSupportedException, IOException {
      if (paramString == null) {
         throw new NullPointerException("id cannot be null");
      }
      List list = AttachProvider.providers(); //Provider를 구함
      if (list.size() == 0) {
         throw new AttachNotSupportedException("no providers installed");
      }
      AttachNotSupportedException attachNotSupportedException = null;
      for (AttachProvider attachProvider : list) {
         try {
            return attachProvider.attachVirtualMachine(paramString); //여기서 JVM에 attach
         } catch (AttachNotSupportedException attachNotSupportedException1) {
            attachNotSupportedException = attachNotSupportedException1;
         } 
      } 
      throw attachNotSupportedException;
   }
   ```

- 4) providers (tools.jar > com.sun.tools.attach.AttachProviders)
   ```java
   public static List<AttachProvider> providers() {
      synchronized (lock) {
         if (providers == null) {
         providers = new ArrayList();
         ServiceLoader serviceLoader = ServiceLoader.load(AttachProvider.class, AttachProvider.class.getClassLoader());
         Iterator iterator = serviceLoader.iterator();
            while (iterator.hasNext()) {
               try {
                  providers.add(iterator.next());
               } catch (Throwable throwable) {
                  if (throwable instanceof ThreadDeath) {
                     ThreadDeath threadDeath = (ThreadDeath)throwable;
                     throw threadDeath;
                  }  
                  System.err.println(throwable);
               } 
            } 
         } 
         return Collections.unmodifiableList(providers);
      } 
   }
   ```
   - ServiceLoader.load(AttachProvider..) 를 통해 Linux 환경에서는 LinuxAttachProvider가 로드된다. (자세한 내용은 아래 Java SPI 참고)

- 5) attachVirtualMachine (tools.jar > sun.tools.attach.LinuxAttachProvider)
   ```java
   public VirtualMachine attachVirtualMachine(String paramString) throws AttachNotSupportedException, IOException {
      checkAttachPermission(); //권한 있는지 확인해보고 
      testAttachable(paramString); //테스트도 해보고 
      return new LinuxVirtualMachine(this, paramString);
   }
   ```

- 6) LinuxVirtualMachine (sun.tools.attach)
   ```java
   LinuxVirtualMachine(AttachProvider paramAttachProvider, String paramString) throws AttachNotSupportedException, IOException {
      super(paramAttachProvider, paramString);
      try {
         i = Integer.parseInt(paramString);
      } catch (NumberFormatException numberFormatException) {
         throw new AttachNotSupportedException("Invalid process identifier");
      }
      this.path = findSocketFile(i); // /tmp 경로에 .java_pid{{pid}} 파일을 찾아봄 
      if (this.path == null) {
         file = createAttachFile(i); // /proc/{{i}}/cwd/.attach_pid{{pid}} 파일을 만들어보고 안 되면 /tmp 경로에 .java_pid{{pid}} 파일 만듦
         try {
            //....
            byte b = 0;
            long l = 200L;
            int k = (int)(attachTimeout() / l);
            do {
               try {
                  Thread.sleep(l);
               } catch (InterruptedException interruptedException) {}
               this.path = findSocketFile(i); 
               ++b;
            } while (b <= k && this.path == null);
               if (this.path == null) {
                  throw new AttachNotSupportedException("Unable to open socket file: target process not responding or HotSpot VM not loaded");
               }
         }
         finally { file.delete(); } 
      } 
      checkPermissions(this.path);
      j = socket();
      try {
         connect(j, this.path);
      } finally {
         close(j);
      } 
   }
   ```

- 7) remoteDataDump (HotSpotVirtualMachine)
   ```java
   public InputStream remoteDataDump(Object... paramVarArgs) throws IOException { return executeCommand("threaddump", paramVarArgs); }
   private InputStream executeCommand(String paramString, Object... paramVarArgs) throws AgentLoadException, IOException {
      try {
         return execute(paramString, paramVarArgs);
      } catch (AgentLoadException agentLoadException) {
         throw new InternalError("Should not get here", agentLoadException);
      } 
   }
   ```

- 8) execute (LinuxVirtualMachine)
   ```java
   InputStream execute(String paramString, Object... paramVarArgs) throws AgentLoadException, IOException {
      int j;
      String str;
      assert paramVarArgs.length <= 3;
      synchronized (this) {
         if (this.path == null) {
         throw new IOException("Detached from target VM");
         }
         str = this.path;
      } 
      int i = socket();
      try {
         connect(i, str);
      } catch (IOException iOException1) {
         close(i);
         throw iOException1;
      } 
      
      IOException iOException = null;
      try {
         writeString(i, "1");
         writeString(i, paramString);
         
         for (byte b = 0; b < 3; b++) {
            if (b < paramVarArgs.length && paramVarArgs[b] != null) {
               writeString(i, (String)paramVarArgs[b]);
            } else {
               writeString(i, "");
            } 
         } 
      } catch (IOException iOException1) {
         iOException = iOException1;
      }

      SocketInputStream socketInputStream = new SocketInputStream(i);
      try {
         j = readInt(socketInputStream);
      } catch (IOException iOException1) {
         socketInputStream.close();
         if (iOException != null) {
         throw iOException;
         }
         throw iOException1;
      } 
      // ....
      return socketInputStream;
   }
   ```
   - 소켓을 통해 쓰레드 덤프를 요청하고 데이터를 받아옴

### option -F
- 2) runJStackTool (sun.tools.jstack.JStack)
   ```java
   private static void runJStackTool(boolean paramBoolean1, boolean paramBoolean2, String[] paramArrayOfString) throws Exception {
      Class clazz = loadSAClass(); // sun.jvm.hotspot.tools.JStack 클래스를 로드해서 main 함수를 호출함 
      if (clazz == null) { usage(1); }
      if (paramBoolean1) { paramArrayOfString = prepend("-m", paramArrayOfString); }
      if (paramBoolean2) { paramArrayOfString = prepend("-l", paramArrayOfString); }
      
      Class[] arrayOfClass = { String[].class };
      Method method = clazz.getDeclaredMethod("main", arrayOfClass);
      
      Object[] arrayOfObject = { paramArrayOfString };
      method.invoke(null, arrayOfObject);
   }
   ```

- 3) sun.jvm.hotspot.tools.JStack (sa-jdi.jar)
   ```java
   public class JStack extends Tool
   {
      private boolean mixedMode;
      private boolean concurrentLocks;
      
      public void run() {
         Tool tool = null;
         if (this.mixedMode) {
            tool = new PStack(false, this.concurrentLocks);
         } else {
            tool = new StackTrace(false, this.concurrentLocks);
         } 
         tool.setAgent(getAgent());
         tool.setDebugeeType(getDebugeeType());
         tool.run();
      }

      public JStack(boolean mixedMode, boolean concurrentLocks) {
         this.mixedMode = mixedMode;
         this.concurrentLocks = concurrentLocks;
      }

      public static void main(String[] args) {
         boolean mixedMode = false;
         boolean concurrentLocks = false;
         int used = 0;
         for (int i = 0; i < args.length; i++) {
            if (args[i].equals("-m")) {
               mixedMode = true;
               used++;
            } else if (args[i].equals("-l")) {
               concurrentLocks = true;
               used++;
            } 
         } 
         
         if (used != 0) {
            String[] newArgs = new String[args.length - used];
            for (int i = 0; i < newArgs.length; i++) {
               newArgs[i] = args[i + used];
            }
            args = newArgs;
         } 
         
         JStack jstack = new JStack(mixedMode, concurrentLocks);
         jstack.execute(args);
      }
   }
   ```

- 4) execute (sun.jvm.hotspot.tools.Tool)
   ```java
   protected void execute(String[] args) {
      int returnStatus = 1;
      try {
         returnStatus = start(args);
      } finally {
         stop();
      } 
      System.exit(returnStatus);
   }
   
   public void stop() {
      if (this.agent != null) { this.agent.detach(); }
   }

   private int start(String[] args) {
      // ...
      this.agent = new HotSpotAgent();
      // ...
      this.agent.attach(pid);
      // ...
      startInternal();
      return 0;
   }

   private void startInternal() {
     //...
      String version = vm.getVMRelease();
      if (version != null) {
         out.print("JVM version is ");
         out.println(version);
      }
      run();
   }
   ```
   - 코드가 길어서 생략했는데 HotSpotAgent.attach(pid) > startInternal() > Jstack.run() 를 호출하는 흐름이다.

- 5) HotSpotAgent (sun.jvm.hotspot.HotSpotAgent)
   - 마찬가지로 코드가 길어서 생략했는데 LinuxDebuggerLocal > attach(pid) 를 호출하는 흐름이다.

- 6) LinuxDebuggerLocal (sun.jvm.hotspot.debugger.linux.LinuxDebuggerLocal)  
   ```java
   public class LinuxDebuggerLocal extends DebuggerBase implements LinuxDebugger {
      private void checkAttached() throws DebuggerException {
         if (this.attached) {
            if (this.isCore) {
            throw new DebuggerException("attached to a core dump already");
            }
            throw new DebuggerException("attached to a process already");
         } 
      }

      private void findABIVersion() throws DebuggerException {
         if (lookupByName0("libjvm.so", "__vt_10JavaThread") != 0L) {
            this.useGCC32ABI = false;
         } else {
            this.useGCC32ABI = true;
         } 
      }

      public void attach(int processID) throws DebuggerException {
         checkAttached();
         this.threadList = new ArrayList();
         this.loadObjectList = new ArrayList();

         class AttachTask implements WorkerThreadTask { 
            int pid;
            
            public void doit(LinuxDebuggerLocal debugger) throws DebuggerException {
               debugger.attach0(this.pid);
               debugger.attached = true;
               debugger.isCore = false;
               LinuxDebuggerLocal.this.findABIVersion();
            } 
         };
         
         AttachTask task = new AttachTask();
         task.pid = processID;
         this.workerThread.execute(task); //LinuxDebuggerLocalWorkerThread
      }

   class LinuxDebuggerLocalWorkerThread extends Thread
   {
      public void run() throws DebuggerException {
         synchronized (LinuxDebuggerLocal.this.workerThread) {
            while (true) {
               if (this.task != null) {
                  this.lastException = null;
               try {
                  this.task.doit(this.debugger);
               } catch (DebuggerException exp) {
                  this.lastException = exp;
               } 
                  this.task = null;
                  LinuxDebuggerLocal.this.workerThread.notifyAll();
               } 
               try {
               LinuxDebuggerLocal.this.workerThread.wait();
               } catch (InterruptedException interruptedException) {}
            } 
         } 
      }
      public LinuxDebuggerLocal.WorkerThreadTask execute(LinuxDebuggerLocal.WorkerThreadTask task) throws DebuggerException {
         synchronized (LinuxDebuggerLocal.this.workerThread) {
            this.task = task;
            LinuxDebuggerLocal.this.workerThread.notifyAll(); //모든 쓰레드를 깨어나게 함
            while (this.task != null) {
               try {
                  LinuxDebuggerLocal.this.workerThread.wait();
               } catch (InterruptedException interruptedException) {}
            } 
            if (this.lastException != null) { 
               throw new DebuggerException(this.lastException); 
            }
            return task;
         } 
      }
   }
   ```

- 7) StankTrace (sun.jvm.hotspot.tools) : Jstack.run() 내에서 호출되는 부분
   ```java
   public void run(PrintStream tty) {
      //....
         Threads threads = VM.getVM().getThreads();
         int i = 1;
         for (JavaThread cur = threads.first(); cur != null; cur = cur.next(), i++) {
         if (cur.isJavaThread()) {
            Address sp = cur.getLastJavaSP();
            tty.print("Thread ");
            cur.printThreadIDOn(tty);
            tty.print(": (state = " + cur.getThreadState());
            if (this.verbose) {
               tty.println(", current Java SP = " + sp);
            }
            tty.println(')');
            try {
               for (JavaVFrame vf = cur.getLastJavaVFrameDbg(); vf != null; vf = vf.javaSender()) {
               Method method = vf.getMethod();
               tty.print(" - " + method.externalNameAndSignature() + " @bci=" + vf.getBCI());

               int lineNumber = method.getLineNumberFromBCI(vf.getBCI());
               if (lineNumber != -1) {
                  tty.print(", line=" + lineNumber);
               }
               
               if (this.verbose) {
                  Address pc = vf.getFrame().getPC();
                  if (pc != null) {
                     tty.print(", pc=" + pc);
                  }
                  
                  tty.print(", Method*=" + method.getAddress());
               } 
               //....
         } 
      } catch (AddressException e) {
         System.err.println("Error accessing address 0x" + Long.toHexString(e.getAddress()));
         e.printStackTrace();
      } 
   }
   ```
   - VM.getVM().getThreads(); 을 통해서 직접 Thread의 정보를 구해 출력한다.

### 정리 : Option -F와 -l의 차이
- -l은 jstack이 소켓 통신을 통해 해당 프로세스에게 덤프 수집을 요청하고 데이터를 받아서 출력한다.
   - 따라서 프로세스가 너무 바쁘거나 데드락 등의 이유로 요청을 처리할 수 없는 경우 덤프를 생성하지 못한다.
- -F는 jstack이 직접 해당 프로세스의 쓰레드 정보를 받아서 출력한다. 
   - 따라서 어느 경우든 덤프를 생성할 수 있지만, 모든 쓰레드의 동작을 멈춘 이후에 수집하므로 쓰레스 스택 정보가 가공된 상태가 된다. 

### Java SPI (Service Provider Interface)
- 참고
   ```
   https://www.baeldung.com/java-spi
   https://docs.oracle.com/javase/7/docs/api/java/util/ServiceLoader.html
   ```
- Java6부터 제공되는 기능으로 주어진 인터페이스에 맞는 구현체를 찾아소 로딩하는 과정을 말한다.
- Component
   - 1) Service : 특정 애플리케이션 또는 기능에 대한 엑세스를 제공하는 인터페이스 또는 클래스 집합
   - 2) Service Provider Interface : Service의 인터페이스 또는 추상 클래스이다. Service가 인터페이스라면 동시에 SPI도 될 수 있다.
   - 3) Service Provider
      - SPI의 구현체로 Service를 구현하거나 확장하는 하나 이상의 클래스를 포함한다.
      - META-INF/services 경로의 Provider 설정 파일이 있는데, 파일명과 내용에 대한 형식은 정해져있다. ([참고](https://docs.oracle.com/javase/7/docs/api/java/util/ServiceLoader.html))
         - 파일명 : FQCN(Full Qualified Class Name) (인터페이스)
         - 내용 : FQCN(Full Qualified Class Name) (실제 구현체)
      - tools.jar의 Service Provider를 확인해보면 아래와 같다. 
         - ![tools.jar > spi](/image/spi.PNG)
         - 그리고 위 코드에서 쓰는 ``ServiceLoader.load(AttachProvider.class...)``에 관한 설정은 아마도 ``com.sun.tools.attach.spi.AttachProvider`` 인 것 같다.
            ```
            #[solaris]sun.tools.attach.SolarisAttachProvider
            #[windows]sun.tools.attach.WindowsAttachProvider
            sun.tools.attach.LinuxAttachProvider //이래서 Linux에서는 LinuxAttachProvider가 로드되는 것
            #[macosx]sun.tools.attach.BsdAttachProvider
            #[aix]sun.tools.attach.AixAttachProvider
            ```
   - 4) ServiceLoader
      - SPI의 핵심 클래스로 구현체를 찾아서 로딩하는 역할을 한다. context classpath를 사용해서 ServiceProvider를 찾아서 내부에 캐싱한다.
   