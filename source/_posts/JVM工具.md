---
title: JVM工具
date: 2018-10-25 01:12:00
tags: 
    - JVM
    - 工具
categories:
    - JVM
---
# 命令行工具
##  jps
即JVM Process Status Tools，用于查看虚拟机进程状态。  
命令格式为
```bash
jps [option]
```
<!--more-->
### jps
列出正在运行的虚拟机进程id以及它们执行的主类。
```bash
C:\Users\14225\IdeaProjects\Code>jps
14804 Test
6372
14696 Jps
18344 RemoteMavenServer
3244 Launcher
```
### jps -q
省略虚拟机进程执行的主类，只列出正在运行的虚拟机进程id。
```bash
C:\Users\14225\IdeaProjects\Code>jps -q
14804
6372
18344
19596
3244
```
### jps -l
在jps的基础上，显示主类的全名。
```bash
14804 priv.shen.javassist.Test
21876 sun.tools.jps.Jps
6372
18344 org.jetbrains.idea.maven.server.RemoteMavenServer
3244 org.jetbrains.jps.cmdline.Launcher

```
### jps -m
在jps的基础上补充虚拟机进程启动时传递给主类的参数。
```bash
14804 Test
6372
16888 Jps -m
18344 RemoteMavenServer
3244 Launcher C:/Program Files/JetBrains/IntelliJ IDEA 2018.1/lib/annotations.jar;C:/Program Files/JetBrains/IntelliJ IDEA 2018.1/lib/httpclient-4
.5.2.jar;C:/Program Files/JetBrains/IntelliJ IDEA 2018.1/lib/slf4j-api-1.7.10.jar;C:/Program Files/JetBrains/IntelliJ IDEA 2018.1/lib/log4j.jar;C:
/Program Files/JetBrains/IntelliJ IDEA 2018.1/lib/snappy-in-java-0.5.1.jar;C:/Program Files/JetBrains/IntelliJ IDEA 2018.1/lib/util.jar;C:/Program
 Files/JetBrains/IntelliJ IDEA 2018.1/lib/jna.jar;C:/Program Files/JetBrains/IntelliJ IDEA 2018.1/lib/idea_rt.jar;C:/Program Files/JetBrains/Intel
liJ IDEA 2018.1/lib/jps-model.jar;C:/Program Files/JetBrains/IntelliJ IDEA 2018.1/lib/jna-platform.jar;C:/Program Files/JetBrains/IntelliJ IDEA 20
18.1/lib/oro-2.0.8.jar;C:/Program Files/JetBrains/IntelliJ IDEA 2018.1/lib/aether-dependency-resolver.jar;C:/Program Files/JetBrains/IntelliJ IDEA
 2018.1/lib/httpcore-4.4.5.jar;C:/Program Files/JetBrains/IntelliJ IDEA 2018.1/lib/netty-all-4.1.13.Final.jar;C:/Program Files/
```
### jps -v
在jps的基础上，补充虚拟机进程启动时的JVM参数
```bash
C:\Users\14225\IdeaProjects\Code>jps -v
14804 Test -javaagent:C:\Program Files\JetBrains\IntelliJ IDEA 2018.1\lib\idea_rt.jar=56883:C:\Program Files\JetBrains\IntelliJ IDEA 2018.1\bin -D
file.encoding=UTF-8
6372  -Xms128m -Xmx750m -XX:ReservedCodeCacheSize=240m -XX:+UseConcMarkSweepGC -XX:SoftRefLRUPolicyMSPerMB=50 -ea -Dsun.io.useCanonCaches=false -D
java.net.preferIPv4Stack=true -XX:+HeapDumpOnOutOfMemoryError -XX:-OmitStackTraceInFastThrow -Djb.vmOptionsFile=C:\Program Files\JetBrains\Intelli
J IDEA 2018.1\bin\idea64.exe.vmoptions -Didea.jre.check=true -Dide.native.launcher=true -Didea.paths.selector=IntelliJIdea2018.1 -XX:ErrorFile=C:\
Users\14225\java_error_in_idea_%p.log -XX:HeapDumpPath=C:\Users\14225\java_error_in_idea.hprof
18344 RemoteMavenServer -Djava.awt.headless=true -Didea.version==2018.1 -Xmx1024m -Didea.maven.embedder.version=3.3.9 -Dfile.encoding=GBK
1756 Jps -Denv.class.path=C:\Program Files\Java\jdk1.8.0_161\lib\tools.jar; -Dapplication.home=C:\Program Files\Java\jdk1.8.0_161 -Xms8m
3244 Launcher -Xmx700m -Djava.awt.headless=true -Djava.endorsed.dirs="" -Djdt.compiler.useSingleThread=true -Dpreload.project.path=C:/Users/14225/
IdeaProjects/Code -Dpreload.config.path=C:/Users/14225/.IntelliJIdea2018.1/config/options -Dexternal.project.config=C:\Users\14225\.IntelliJIdea20
18.1\system\external_build_system\code.982f4335 -Dcompile.parallel=false -Drebuild.on.dependency.change=true -Djava.net.preferIPv4Stack=true -Dio.
netty.initialSeedUniquifier=-5105987098425341563 -Dfile.encoding=GBK -Duser.language=zh -Duser.country=CN -Didea.paths.selector=IntelliJIdea2018.1
 -Didea.home.path=C:\Program Files\JetBrains\IntelliJ IDEA 2018.1 -Didea.config.path=C:\Users\14225\.IntelliJIdea2018.1\config -Didea.plugins.path
=C:\Users\14225\.IntelliJIdea2018.1\config\plugins -Djps.log.dir=C:/Users/14225/.IntelliJIdea2018.1/system/log/build-log -Djps.fallback.jdk.home=C
:/Program Files/JetBrains/IntelliJ IDEA 2018.1/jre64 -Djps.fallback.jdk.version=1.8.0_152-release -Dio.netty.noUnsafe=true -Djava.io.tmpdir=C:/Use
rs/14225/.Intell
```
## jstat
即JVM Statistics-Monitoring Tools，用于监控虚拟机运行信息，
包括类加载，内存，垃圾回收和运行时编译的状况。  
命令格式为
```bash
jstat option vimid [interval][s|ms] [count]
```
### jstat -class vmid
查看类加载，卸载的情况
```bash
C:\Users\14225\IdeaProjects\Code>jstat -class 14804
Loaded  Bytes  Unloaded   Bytes       Time
   558  1128.9        0     0.0       0.11
```
### jstat -gc vmid
查看各分区的容量以及使用情况，和GC耗时。
```bash
C:\Users\14225\IdeaProjects\Code>jstat -gc 14804
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
5120.0 5120.0  0.0    0.0   33280.0   5327.2   87552.0      0.0     4480.0 770.9  384.0   75.9       0    0.000   0      0.000    0.000

```
### jstat -gccapacity vmid
查看堆各分区的最大和最小容量以及GC次数。
```bash
C:\Users\14225\IdeaProjects\Code>jstat -gccapacity 14804
 NGCMN    NGCMX     NGC     S0C   S1C       EC      OGCMN      OGCMX       OGC         OC       MCMN     MCMX      MC     CCSMN    CCSMX     CCSC
 43520.0 689152.0  43520.0 5120.0 5120.0  33280.0    87552.0  1379328.0    87552.0    87552.0    0.0   1056768.0   4480.0   0.0   1048576.0  384.0
 YGC    FGC
  0      0
```
### jstat -gcutil vmid
查看各分区的使用百分比，以及GC的次数和耗时。
```bash
C:\Users\14225\IdeaProjects\Code>jstat -gcutil 14804
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00   0.00  16.01   0.00  17.21  19.76      0    0.000     0    0.000    0.000
```
### jstat -gccause vmid
在jstat -gcutil的基础上补充上次GC的原因。
```bash
C:\Users\14225\IdeaProjects\Code>jstat -gccause 14804
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT    LGCC                 GCC
  0.00   0.00  16.01   0.00  17.21  19.76      0    0.000     0    0.000    0.000 No GC                No GC
```
### jstat -gcnew vmid
查看新生代内存使用情况和GC情况。
```bash
C:\Users\14225\IdeaProjects\Code>jstat -gcnew 14804
 S0C    S1C    S0U    S1U   TT MTT  DSS      EC       EU     YGC     YGCT
5120.0 5120.0    0.0    0.0 15  15    0.0  33280.0   5327.2      0    0.000
```
### jstat -gcnewcapacity vmid
与jstat -gcnew类似，只不过监控的是新生代内存的最大最小空间。
```bash
C:\Users\14225\IdeaProjects\Code>jstat -gcnewcapacity 14804
  NGCMN      NGCMX       NGC      S0CMX     S0C     S1CMX     S1C       ECMX        EC      YGC   FGC
   43520.0   689152.0    43520.0 229376.0   5120.0 229376.0   5120.0   688128.0    33280.0     0     0
```
### jstat -gcold vmid
与jstat -gcnew类似，只不过是监控的老年代
```bash
C:\Users\14225\IdeaProjects\Code>jstat -gcold 14804
   MC       MU      CCSC     CCSU       OC          OU       YGC    FGC    FGCT     GCT
  4480.0    770.9    384.0     75.9     87552.0         0.0      0     0    0.000    0.000
```
### jstat -gcoldcapacity vmid
与jstat -gcnewcapacity类似，只不过是监控的是老年代
```bash
C:\Users\14225\IdeaProjects\Code>jstat -gcoldcapacity 14804
   OGCMN       OGCMX        OGC         OC       YGC   FGC    FGCT     GCT
    87552.0   1379328.0     87552.0     87552.0     0     0    0.000    0.000
```
### jstat -compiler vmid
输出JIT编译器编译过的方法以及耗时情况
```bash
C:\Users\14225\IdeaProjects\Code>jstat -compiler 14804
Compiled Failed Invalid   Time   FailedType FailedMethod
      62      0       0     0.04          0
```
### jstat -printcompilation vmid
输出JIT编译器编译过的方法
```bash
C:\Users\14225\IdeaProjects\Code>jstat -printcompilation 14804
Compiled  Size  Type Method
      62     63    1 java/lang/ThreadGroup threadTerminated
```
## jinfo
即Java Configuration Info，用于查看和修改虚拟机个各项配置参数。  
命令格式
```bash
jinfo [option] vmid
```
### jinfo -flags vmid
输出虚拟机进程的各项JVM参数（非默认，要想查看默认参数要用jinfo -flag name），而jps -v只能显示出显式指定的JVM参数。  
这个是jinfo -flags 显示的JVM参数
```bash
C:\Users\14225\IdeaProjects\Code>jinfo -flags 14804
Attaching to process ID 14804, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.161-b12
Non-default VM flags: -XX:CICompilerCount=4 -XX:InitialHeapSize=134217728 -XX:MaxHeapSize=2118123520 -XX:MaxNewSize=705691648 -XX:MinHeapDeltaByte
s=524288 -XX:NewSize=44564480 -XX:OldSize=89653248 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps -XX:-Use
LargePagesIndividualAllocation -XX:+UseParallelGC
Command line:  -javaagent:C:\Program Files\JetBrains\IntelliJ IDEA 2018.1\lib\idea_rt.jar=56883:C:\Program Files\JetBrains\IntelliJ IDEA 2018.1\bi
n -Dfile.encoding=UTF-8
```
而这个是jps -v 显示的JVM参数
```bash
14804 Test -javaagent:C:\Program Files\JetBrains\IntelliJ IDEA 2018.1\lib\idea_rt.jar=56883:C:\Program Files\JetBrains\IntelliJ IDEA 2018.1\bin -D
file.encoding=UTF-8
```
### jinfo -flag name=value vmid
修改虚拟机进程的某项参数，注意只有一部分参数允许运行时修改，这里修改的是使当OOM时做堆快照转储快照的参数生效。
```bash
C:\Users\14225\IdeaProjects\Code>jinfo -flag HeapDumpOnOutOfMemoryError=1 14804
```
### jinfo -systprops vmid
输出虚拟机进程的各项java系统属性  
这里比较多，只列出了一部分
```bash
:\Users\14225\IdeaProjects\Code>jinfo -sysprops 14804
Attaching to process ID 14804, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.161-b12
java.runtime.name = Java(TM) SE Runtime Environment
java.vm.version = 25.161-b12
sun.boot.library.path = C:\Program Files\Java\jdk1.8.0_161\jre\bin
java.vendor.url = http://java.oracle.com/
java.vm.vendor = Oracle Corporation
path.separator = ;
file.encoding.pkg = sun.io
java.vm.name = Java HotSpot(TM) 64-Bit Server VM
sun.os.patch.level =
sun.java.launcher = SUN_STANDARD
```
## jmap
即Java Memory Map，用于生成堆转储快照（headdump)，或者是查看finalize执行队列，堆、永久代的详细信息。
命令格式
```bash
jmap [option] vmid
```
### jmap -dump vimid
生成堆转储快照。  
命令格式
```
jmap -dump:[live],format=b,file=<filename> vmid
```
其中live表示只选择存活对象进行dump，format=b指以二进制文件的形式输出，file=<filename>指定输出的文件名称。
```
C:\Users\14225\IdeaProjects\Code>jmap -dump:live,format=b,file=dump.bin 14804
Dumping heap to C:\Users\14225\IdeaProjects\Code\dump.bin ...
Heap dump file created
```
后续需要用jhat来对dump进行分析，到时候一起讲。
### jmap -finalizerinfo vmid
查看finalize执行队列
```bash
C:\Users\14225\IdeaProjects\Code>jmap -finalizerinfo 14804
Attaching to process ID 14804, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.161-b12
Number of objects pending for finalization: 0
```
### jmap -heap vmid
查看堆的配置情况和使用情况
```bash
C:\Users\14225\IdeaProjects\Code>jmap -heap 14804
Attaching to process ID 14804, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.161-b12

using thread-local object allocation.
Parallel GC with 8 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 2118123520 (2020.0MB)
   NewSize                  = 44564480 (42.5MB)
   MaxNewSize               = 705691648 (673.0MB)
   OldSize                  = 89653248 (85.5MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 34078720 (32.5MB)
   used     = 0 (0.0MB)
   free     = 34078720 (32.5MB)
   0.0% used
From Space:
   capacity = 5242880 (5.0MB)
   used     = 0 (0.0MB)
   free     = 5242880 (5.0MB)
   0.0% used
To Space:
   capacity = 5242880 (5.0MB)
   used     = 0 (0.0MB)
   free     = 5242880 (5.0MB)
   0.0% used
PS Old Generation
   capacity = 35651584 (34.0MB)
   used     = 1378752 (1.31488037109375MB)
   free     = 34272832 (32.68511962890625MB)
   3.8672952090992645% used

1693 interned Strings occupying 153240 bytes.
```
### jmap -histo vmid
查看堆中的对象信息。  
这里只选择了存活的对象。
```bash
C:\Users\14225\IdeaProjects\Code>jmap -histo:live 14804

 num     #instances         #bytes  class name
----------------------------------------------
   1:          1169         439544  java.lang.Thread
   2:          3935         408408  [C
   3:          3785          90840  java.lang.String
   4:           641          73176  java.lang.Class
   5:          1174          46960  java.security.AccessControlContext
   6:           606          35136  [Ljava.lang.Object;
   7:            26          33672  [B
   8:           791          31640  java.util.TreeMap$Entry
   9:          1162          27888  [Ljava.security.ProtectionDomain;
  10:          1161          27864  priv.shen.javassist.Task
  11:          1245          19920  java.lang.Object
  12:           782          18768  java.util.LinkedList$Node
  13:           463          14816  java.util.HashMap$Node
  14:           394          12608  java.util.LinkedList
  15:             2           8240  [Ljava.lang.Thread;
  16:           202           8136  [Ljava.lang.String;
  17:            21           6032  [Ljava.util.HashMap$Node;
  18:           115           4904  [I
  19:            75           4800  java.net.URL
  20:           150           4800  java.util.Hashtable$Entry
  21:           103           4120  java.lang.ref.SoftReference
  22:           256           4096  java.lang.Integer
  23:            90           2880  java.util.concurrent.ConcurrentHashMap$Node
  24:            37           2072  sun.misc.URLClassPath$JarLoader
  25:            38           1824  sun.util.locale.LocaleObjectCache$CacheEntry
  26:            22           1760  java.lang.reflect.Constructor
  27:            40           1600  java.util.LinkedHashMap$Entry
  28:            37           1480  java.io.ObjectStreamField
  29:            18           1440  [Ljava.util.WeakHashMap$Entry;
  30:            27           1296  java.util.HashMap
  31:             8           1264  [Ljava.util.Hashtable$Entry;
  32:            27           1080  java.lang.ref.Finalizer
  33:             2           1064  [Ljava.lang.invoke.MethodHandle;
  34:             1           1040  [Ljava.lang.Integer;
  35:             1           1040  [[C
  36:            40            960  java.io.ExpiringCache$Entry
  37:            15            960  java.util.jar.JarFile
  38:            17            952  sun.nio.cs.UTF_8$Encoder
  39:             5            912  [Ljava.util.concurrent.ConcurrentHashMap$Node;
  40:            18            864  java.util.WeakHashMap
  41:             8            768  java.util.jar.JarFile$JarFileEntry
  42:            19            760  sun.util.locale.BaseLocale$Key
  43:            22            704  java.lang.ref.ReferenceQueue
  44:             8            640  [S
  45:            19            608  java.util.Locale
  46:            19            608  sun.util.locale.BaseLocale
  47:            15            480  java.util.zip.ZipCoder
  48:            19            456  java.util.Locale$LocaleKey
  49:             7            448  java.util.concurrent.ConcurrentHashMap
  50:            11            440  sun.nio.cs.UTF_8$Decoder
  51:            17            408  java.util.jar.Attributes$Name
  52:            13            392  [Ljava.io.ObjectStreamField;
  53:             1            384  com.intellij.rt.execution.application.AppMainV2$1
  54:            12            384  java.io.File
  55:             1            384  java.lang.ref.Finalizer$FinalizerThread
  56:            24            384  java.lang.ref.ReferenceQueue$Lock
  57:             6            384  java.nio.DirectByteBuffer
  58:             1            376  java.lang.ref.Reference$ReferenceHandler
  59:            15            360  java.util.ArrayDeque
  60:             6            336  java.nio.DirectLongBufferU
  61:            10            320  java.lang.OutOfMemoryError
  62:             3            312  [D
  63:            12            288  java.util.ArrayList
  64:            11            264  sun.misc.MetaIndex
  65:            11            264  sun.reflect.NativeConstructorAccessorImpl
  66:             5            200  java.util.WeakHashMap$Entry
  67:            12            192  [Ljava.lang.Class;
  68:             4            192  java.util.Hashtable
  69:             4            192  java.util.Properties
  70:             4            192  java.util.TreeMap
  71:             6            192  java.util.Vector
  72:            11            176  sun.reflect.DelegatingConstructorAccessorImpl
  73:             4            160  java.io.FileDescriptor
  74:             4            160  java.lang.ClassLoader$NativeLibrary
  75:             4            160  java.security.ProtectionDomain
  76:             3            144  java.nio.HeapByteBuffer
  77:             2            144  java.util.regex.Pattern
  78:             3            144  java.util.zip.Inflater
  79:             6            144  sun.misc.PerfCounter
  80:             6            144  sun.security.util.DisabledAlgorithmConstraints$Constraint$Operator
  81:             2            128  java.io.ExpiringCache$1
  82:             4            128  java.security.CodeSource
  83:             1            120  java.net.SocksSocketImpl
  84:             5            120  sun.misc.FloatingDecimal$PreparedASCIIToBinaryBuffer
  85:             2            112  java.lang.Package
  86:             2            112  java.util.LinkedHashMap
  87:             4             96  java.lang.RuntimePermission
  88:             2             96  java.lang.ThreadGroup
  89:             3             96  java.util.Stack
  90:             1             96  sun.misc.Launcher$AppClassLoader
  91:             2             96  sun.misc.URLClassPath
  92:             3             96  sun.net.spi.DefaultProxySelector$NonProxyInfo
  93:             2             96  sun.nio.cs.StreamEncoder
  94:             1             88  java.net.DualStackPlainSocketImpl
  95:             1             88  sun.misc.Launcher$ExtClassLoader
  96:             1             80  [Ljava.lang.ThreadLocal$ThreadLocalMap$Entry;
  97:             2             80  java.io.BufferedWriter
  98:             2             80  java.io.ExpiringCache
  99:             5             80  java.lang.ThreadLocal
 100:             2             80  sun.misc.FloatingDecimal$BinaryToASCIIBuffer
 101:             2             80  sun.security.util.DisabledAlgorithmConstraints$KeySizeConstraint
 102:             3             72  java.net.Proxy$Type
 103:             3             72  java.util.Collections$SynchronizedSet
 104:             3             72  java.util.concurrent.atomic.AtomicLong
 105:             3             72  java.util.zip.ZStreamRef
 106:             3             72  sun.misc.FloatingDecimal$ExceptionalBinaryToASCIIBuffer
 107:             3             72  sun.misc.JarIndex
 108:             1             64  [F
 109:             4             64  [Ljava.security.Principal;
 110:             2             64  java.io.FileOutputStream
 111:             2             64  java.io.FilePermission
 112:             2             64  java.io.PrintStream
 113:             2             64  java.lang.ClassValue$Entry
 114:             2             64  java.lang.VirtualMachineError
 115:             2             64  java.lang.ref.ReferenceQueue$Null
 116:             2             64  java.security.BasicPermissionCollection
 117:             2             64  java.security.Permissions
 118:             4             64  java.security.ProtectionDomain$Key
 119:             1             48  [J
 120:             2             48  java.io.BufferedOutputStream
 121:             1             48  java.io.BufferedReader
 122:             2             48  java.io.File$PathStatus
 123:             2             48  java.io.FilePermissionCollection
 124:             2             48  java.io.OutputStreamWriter
 125:             2             48  java.net.InetAddress$Cache
 126:             2             48  java.net.InetAddress$Cache$Type
 127:             1             48  java.net.SocketInputStream
 128:             1             48  java.nio.HeapCharBuffer
 129:             2             48  java.nio.charset.CoderResult
 130:             3             48  java.nio.charset.CodingErrorAction
 131:             2             48  java.util.regex.Pattern$SliceI
 132:             2             48  java.util.regex.Pattern$Start
 133:             2             48  sun.misc.NativeSignalHandler
 134:             2             48  sun.misc.Signal
 135:             1             48  sun.nio.cs.StreamDecoder
 136:             1             48  sun.nio.cs.US_ASCII$Decoder
 137:             2             48  sun.security.util.DisabledAlgorithmConstraints$DisabledConstraint
 138:             1             40  [Lsun.security.util.DisabledAlgorithmConstraints$Constraint$Operator;
 139:             1             40  [[Ljava.lang.String;
 140:             1             40  java.io.BufferedInputStream
 141:             1             40  sun.nio.cs.StandardCharsets$Aliases
 142:             1             40  sun.nio.cs.StandardCharsets$Cache
 143:             1             40  sun.nio.cs.StandardCharsets$Classes
 144:             1             40  sun.nio.cs.ext.ExtendedCharsets
 145:             1             32  [Ljava.lang.OutOfMemoryError;
 146:             2             32  [Ljava.lang.StackTraceElement;
 147:             1             32  [Ljava.lang.ThreadGroup;
 148:             1             32  [Ljava.net.Proxy$Type;
 149:             1             32  java.io.FileInputStream
 150:             1             32  java.io.WinNTFileSystem
 151:             1             32  java.lang.ArithmeticException
 152:             2             32  java.lang.Boolean
 153:             1             32  java.lang.NullPointerException
 154:             1             32  java.lang.ThreadLocal$ThreadLocalMap$Entry
 155:             1             32  java.net.InetAddress$InetAddressHolder
 156:             1             32  java.net.Socket
 157:             2             32  java.nio.ByteOrder
 158:             2             32  java.util.HashSet
 159:             2             32  java.util.concurrent.atomic.AtomicInteger
 160:             1             32  java.util.concurrent.atomic.AtomicReferenceFieldUpdater$AtomicReferenceFieldUpdaterImpl
 161:             1             32  java.util.regex.Pattern$Branch
 162:             1             32  sun.instrument.InstrumentationImpl
 163:             2             32  sun.net.www.protocol.jar.Handler
 164:             1             32  sun.nio.cs.StandardCharsets
 165:             1             24  [Ljava.io.File$PathStatus;
 166:             1             24  [Ljava.lang.ClassValue$Entry;
 167:             1             24  [Ljava.net.InetAddress$Cache$Type;
 168:             1             24  [Ljava.util.regex.Pattern$Node;
 169:             1             24  [Lsun.launcher.LauncherHelper;
 170:             1             24  java.io.InputStreamReader
 171:             1             24  java.lang.ClassValue$Version
 172:             1             24  java.lang.StringBuilder
 173:             1             24  java.lang.ThreadLocal$ThreadLocalMap
 174:             1             24  java.lang.invoke.MethodHandleImpl$4
 175:             1             24  java.lang.reflect.ReflectPermission
 176:             1             24  java.net.Inet4Address
 177:             1             24  java.net.Inet6AddressImpl
 178:             1             24  java.net.Proxy
 179:             1             24  java.util.BitSet
 180:             1             24  java.util.Collections$EmptyMap
 181:             1             24  java.util.Collections$SetFromMap
 182:             1             24  java.util.Collections$UnmodifiableRandomAccessList
 183:             1             24  java.util.Locale$Cache
 184:             1             24  java.util.regex.Pattern$Single
 185:             1             24  sun.instrument.TransformerManager
 186:             1             24  sun.launcher.LauncherHelper
 187:             1             24  sun.misc.URLClassPath$FileLoader
 188:             1             24  sun.nio.cs.ISO_8859_1
 189:             1             24  sun.nio.cs.ThreadLocalCoders$1
 190:             1             24  sun.nio.cs.ThreadLocalCoders$2
 191:             1             24  sun.nio.cs.US_ASCII
 192:             1             24  sun.nio.cs.UTF_16
 193:             1             24  sun.nio.cs.UTF_16BE
 194:             1             24  sun.nio.cs.UTF_16LE
 195:             1             24  sun.nio.cs.UTF_8
 196:             1             24  sun.nio.cs.ext.GBK
 197:             1             24  sun.security.util.DisabledAlgorithmConstraints
 198:             1             24  sun.util.locale.BaseLocale$Cache
 199:             1             16  [Ljava.lang.Throwable;
 200:             1             16  [Ljava.security.cert.Certificate;
 201:             1             16  [Lsun.instrument.TransformerManager$TransformerInfo;
 202:             1             16  java.io.FileDescriptor$1
 203:             1             16  java.lang.CharacterDataLatin1
 204:             1             16  java.lang.ClassValue$Identity
 205:             1             16  java.lang.Runtime
 206:             1             16  java.lang.String$CaseInsensitiveComparator
 207:             1             16  java.lang.System$2
 208:             1             16  java.lang.Terminator$1
 209:             1             16  java.lang.invoke.MemberName$Factory
 210:             1             16  java.lang.invoke.MethodHandleImpl$2
 211:             1             16  java.lang.invoke.MethodHandleImpl$3
 212:             1             16  java.lang.ref.Reference$1
 213:             1             16  java.lang.ref.Reference$Lock
 214:             1             16  java.lang.reflect.ReflectAccess
 215:             1             16  java.net.InetAddress$2
 216:             1             16  java.net.URLClassLoader$7
 217:             1             16  java.nio.Bits$1
 218:             1             16  java.nio.charset.CoderResult$1
 219:             1             16  java.nio.charset.CoderResult$2
 220:             1             16  java.security.ProtectionDomain$2
 221:             1             16  java.security.ProtectionDomain$JavaSecurityAccessImpl
 222:             1             16  java.util.Collections$EmptyList
 223:             1             16  java.util.Collections$EmptySet
 224:             1             16  java.util.Hashtable$EntrySet
 225:             1             16  java.util.WeakHashMap$KeySet
 226:             1             16  java.util.concurrent.atomic.AtomicBoolean
 227:             1             16  java.util.jar.JavaUtilJarAccessImpl
 228:             1             16  java.util.regex.Pattern$4
 229:             1             16  java.util.regex.Pattern$BranchConn
 230:             1             16  java.util.regex.Pattern$LastNode
 231:             1             16  java.util.regex.Pattern$Node
 232:             1             16  java.util.zip.ZipFile$1
 233:             1             16  sun.misc.ASCIICaseInsensitiveComparator
 234:             1             16  sun.misc.FloatingDecimal$1
 235:             1             16  sun.misc.Launcher
 236:             1             16  sun.misc.Launcher$Factory
 237:             1             16  sun.misc.Perf
 238:             1             16  sun.misc.Unsafe
 239:             1             16  sun.net.spi.DefaultProxySelector
 240:             1             16  sun.net.www.protocol.file.Handler
 241:             1             16  sun.reflect.ReflectionFactory
 242:             1             16  sun.security.util.AlgorithmDecomposer
 243:             1             16  sun.security.util.DisabledAlgorithmConstraints$Constraints
Total         19306        1371096
```
## jhat
即Java Heap Analysis Tool，一般与jmap配合使用来分析堆转储快照。  
命令格式  
```bash
jhat <filename>
```
```bash
C:\Users\14225\IdeaProjects\Code>jhat dump.bin
Reading from dump.bin...
Dump file created Wed Oct 24 18:43:51 CST 2018
Snapshot read, resolving...
Resolving 19387 objects...
Chasing references, expect 3 dots...
Eliminating duplicate references...
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.

```
jhat会把分析结果以html的形式输出，打开http://localhost:7000即可查看
## jstack
即Java Stack Trace，用于生成虚拟机进程当前时刻的线程快照，分析线程快照中线程的调用堆栈来快速定位线程长时间停顿的原因，可以用来检测死锁（一般是死锁，死循环，请求外部资源时长时间等待）。  
命令格式  
```
jstack [option] vmid
```
### jstack -l vimid
显示线程的调用堆栈，并且显示关于锁的信息。  
我这里特地查看了一个死锁的程序的线程快照。
```bash
C:\Users\14225\IdeaProjects\Code>jstack -l 3516
2018-10-24 19:29:51
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.161-b12 mixed mode):

"DestroyJavaVM" #16 prio=5 os_prio=0 tid=0x0000000019b48000 nid=0x3d44 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"Thread-3" #15 prio=5 os_prio=0 tid=0x0000000019b47800 nid=0x4d18 waiting for monitor entry [0x000000001aacf000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at priv.shen.javassist.Task.run(Task.java:13)
        - waiting to lock <0x00000000d60b7e90> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:748)

   Locked ownable synchronizers:
        - None

"Thread-2" #14 prio=5 os_prio=0 tid=0x0000000019b49000 nid=0x43f8 waiting for monitor entry [0x000000001a9cf000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at priv.shen.javassist.Task.run(Task.java:13)
        - waiting to lock <0x00000000d60b7e80> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:748)

   Locked ownable synchronizers:
        - None

"Thread-1" #13 prio=5 os_prio=0 tid=0x0000000019b46800 nid=0x2aec waiting for monitor entry [0x000000001a8cf000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at priv.shen.javassist.Task.run(Task.java:19)
        - waiting to lock <0x00000000d60b7e80> (a java.lang.Object)
        - locked <0x00000000d60b7e90> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:748)

   Locked ownable synchronizers:
        - None

"Thread-0" #12 prio=5 os_prio=0 tid=0x0000000019b46000 nid=0x541c waiting for monitor entry [0x000000001a7cf000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at priv.shen.javassist.Task.run(Task.java:19)
        - waiting to lock <0x00000000d60b7e90> (a java.lang.Object)
        - locked <0x00000000d60b7e80> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:748)

   Locked ownable synchronizers:
        - None

"Service Thread" #11 daemon prio=9 os_prio=0 tid=0x0000000019a34000 nid=0x5968 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"C1 CompilerThread3" #10 daemon prio=9 os_prio=2 tid=0x00000000199d0800 nid=0x4664 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"C2 CompilerThread2" #9 daemon prio=9 os_prio=2 tid=0x00000000199c1800 nid=0x2774 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"C2 CompilerThread1" #8 daemon prio=9 os_prio=2 tid=0x00000000199bd800 nid=0x1690 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"C2 CompilerThread0" #7 daemon prio=9 os_prio=2 tid=0x00000000199b8800 nid=0x3e20 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"Monitor Ctrl-Break" #6 daemon prio=5 os_prio=0 tid=0x0000000019999000 nid=0x4ebc runnable [0x000000001a0ce000]
   java.lang.Thread.State: RUNNABLE
        at java.net.SocketInputStream.socketRead0(Native Method)
        at java.net.SocketInputStream.socketRead(SocketInputStream.java:116)
        at java.net.SocketInputStream.read(SocketInputStream.java:171)
        at java.net.SocketInputStream.read(SocketInputStream.java:141)
        at sun.nio.cs.StreamDecoder.readBytes(StreamDecoder.java:284)
        at sun.nio.cs.StreamDecoder.implRead(StreamDecoder.java:326)
        at sun.nio.cs.StreamDecoder.read(StreamDecoder.java:178)
        - locked <0x00000000d6133710> (a java.io.InputStreamReader)
        at java.io.InputStreamReader.read(InputStreamReader.java:184)
        at java.io.BufferedReader.fill(BufferedReader.java:161)
        at java.io.BufferedReader.readLine(BufferedReader.java:324)
        - locked <0x00000000d6133710> (a java.io.InputStreamReader)
        at java.io.BufferedReader.readLine(BufferedReader.java:389)
        at com.intellij.rt.execution.application.AppMainV2$1.run(AppMainV2.java:64)

   Locked ownable synchronizers:
        - None

"Attach Listener" #5 daemon prio=5 os_prio=2 tid=0x0000000019909000 nid=0x1c70 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"Signal Dispatcher" #4 daemon prio=9 os_prio=2 tid=0x0000000019960800 nid=0x3bd8 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"Finalizer" #3 daemon prio=8 os_prio=1 tid=0x00000000198f0800 nid=0x35c0 in Object.wait() [0x0000000019dcf000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000000d5f08ec0> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:143)
        - locked <0x00000000d5f08ec0> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:164)
        at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:209)

   Locked ownable synchronizers:
        - None

"Reference Handler" #2 daemon prio=10 os_prio=2 tid=0x0000000002a8a000 nid=0x4510 in Object.wait() [0x00000000198ce000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000000d5f06b68> (a java.lang.ref.Reference$Lock)
        at java.lang.Object.wait(Object.java:502)
        at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
        - locked <0x00000000d5f06b68> (a java.lang.ref.Reference$Lock)
        at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

   Locked ownable synchronizers:
        - None

"VM Thread" os_prio=2 tid=0x0000000017a09000 nid=0x4e20 runnable

"GC task thread#0 (ParallelGC)" os_prio=0 tid=0x00000000029a9000 nid=0x3b2c runnable

"GC task thread#1 (ParallelGC)" os_prio=0 tid=0x00000000029aa800 nid=0x1918 runnable

"GC task thread#2 (ParallelGC)" os_prio=0 tid=0x00000000029ac000 nid=0x547c runnable

"GC task thread#3 (ParallelGC)" os_prio=0 tid=0x00000000029ad800 nid=0x8f8 runnable

"GC task thread#4 (ParallelGC)" os_prio=0 tid=0x00000000029b1000 nid=0x56b8 runnable

"GC task thread#5 (ParallelGC)" os_prio=0 tid=0x00000000029b2000 nid=0x3fd8 runnable

"GC task thread#6 (ParallelGC)" os_prio=0 tid=0x00000000029b5000 nid=0x3f74 runnable

"GC task thread#7 (ParallelGC)" os_prio=0 tid=0x00000000029b6800 nid=0x3a08 runnable

"VM Periodic Task Thread" os_prio=2 tid=0x0000000019b2e800 nid=0x78c waiting on condition

JNI global references: 17


Found one Java-level deadlock:
=============================
"Thread-3":
  waiting to lock monitor 0x0000000017a13728 (object 0x00000000d60b7e90, a java.lang.Object),
  which is held by "Thread-1"
"Thread-1":
  waiting to lock monitor 0x0000000017a12288 (object 0x00000000d60b7e80, a java.lang.Object),
  which is held by "Thread-0"
"Thread-0":
  waiting to lock monitor 0x0000000017a13728 (object 0x00000000d60b7e90, a java.lang.Object),
  which is held by "Thread-1"

Java stack information for the threads listed above:
===================================================
"Thread-3":
        at priv.shen.javassist.Task.run(Task.java:13)
        - waiting to lock <0x00000000d60b7e90> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:748)
"Thread-1":
        at priv.shen.javassist.Task.run(Task.java:19)
        - waiting to lock <0x00000000d60b7e80> (a java.lang.Object)
        - locked <0x00000000d60b7e90> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:748)
"Thread-0":
        at priv.shen.javassist.Task.run(Task.java:19)
        - waiting to lock <0x00000000d60b7e90> (a java.lang.Object)
        - locked <0x00000000d60b7e80> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.
```
如果发生了死锁，可以在最后查看分析结果来排查死锁原因。
# 图形工具
JDK自带的jconsle.exe和jvisualvm.exe这两个图形化工具，是将上述的命令工具可视化了，很容易上手，这里不做过多介绍。