## Memory profiling – simple examples
Recently I have been trying to learn different memory profiling tools to monitor Java applications. I have looked into the command line tools that are shipped as part of JDK like `jstat`, `jps`, `jvisualvm` etc. Licensed tools like `Yourkit` provide wholesome information about a running JVM including memory usage, CPU time, thread count etc. Running a java application with `-verbose:gc` option prints memory usage of each generation after every garbage collection event.

Profiler tools like `VisualVM` modify the bytecode of the application they are profiling, which may result in some peculiarities. For example, if we profile the following code in `VisualVM`, it shows heap usage increasing linearly over time even though no objects are being created.
```java
public class EmptyWhileLoop {
  public static void main(String[] arg) {
    while(true) {}
  }
}
```
![VisualVM output](https://github.com/sskelkar/blog-posts/raw/master/images/memory-profiling-1.png)

Alternatively, `jstat` can give us periodical information on a wide range of parameters like garbage collection statistics, heap capacities, class compilation etc.

Typically `jstat` can be used in following way:
```sh
jstat <option> <pid> <interval> <count>
```
So if we run the following on command prompt,
```sh
jstat -gcutil 410 500 10
```
It will print garbage collection statistics of the JVM running with `pid` 410 every 0.5 second, 10 times. If we don’t give the `count` option, `jstat` will run till the process exits. `pid` of an application can be found using `jps` command.

For above code, `jstat -util` will give the output as following:
![jstat output](https://github.com/sskelkar/blog-posts/raw/master/images/memory-profiling-1.png)
The meaning of each column can be found in the official [Oracle documentation](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html).

Now let's consider a simple example of creating an ArrayList with and without providing `initialCapacity`. If we do not provide `initialCapacity` to the ArrayList constructor, by default it creates an internal array of initial size 10. As we add elements to this ArrayList and exceed this size, a new internal array with size as 1.5 times the old size is created and values from old array are copied to the new one. If we add large number of values to an ArrayList but do not initialize it, the internal array will have to be resized several times over, and the discarded array objects go up for garbage collection. So an application that does not initialize ArrayList is going to consume much more memory than the one that does. We can verify this by running the JVM with `-verbose:gc` option.

Suppose we have a class like this:
```java
import java.util.*;

public class ArrayListProfiler {
  public static void main(String[] a) {
    List<String> array = new ArrayList<String>();
    
    for(int i=0; i<10000; i++) {
      array.add(String.valueOf(i));
    }
  }
}
```
I will compile and run it as following:
```cmd
javac ArrayListProfiler.java && java -Xmx1m -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:D:\gc.log ArrayListProfiler
```
First let's understand what each of these options means. `-Xmx1m` sets the maximum heap size for this application as 1MB. Limiting the heap size to some small value is important to simulate low memory condition. Otherwise, JVM won’t need to kick in the garbage collector. `-verbose:gc` enables logging of garbage collection information. `-XX:+PrintGCDetails` and `-XX:+PrintGCTimeStamps` give detailed GC information and generation wise heap usage. `-Xloggc:D:\gc.log` logs the `-verbose:gc` output to a file.

The output generated for above code is as following:
```sh
Java HotSpot(TM) 64-Bit Server VM (25.40-b25) for windows-amd64 JRE (1.8.0_40-b26), built on Mar  7 2015 13:51:59 by "java_re" with MS VC++ 10.0 (VS2010)
Memory: 4k page, physical 8365480k(1878384k free), swap 16729064k(4152084k free)
CommandLine flags: -XX:InitialHeapSize=1048576 -XX:MaxHeapSize=1048576 -XX:+PrintGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC 
0.077: [GC (Allocation Failure) [PSYoungGen: 512K->496K(1024K)] 512K->520K(1536K), 0.0007685 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
0.080: [GC (Allocation Failure) [PSYoungGen: 1008K->486K(1024K)] 1032K->806K(1536K), 0.0009741 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
0.081: [Full GC (Ergonomics) [PSYoungGen: 486K->486K(1024K)] [ParOldGen: 320K->288K(512K)] 806K->774K(1536K), [Metaspace: 2439K->2439K(1056768K)], 0.0064944 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
Heap
 PSYoungGen      total 1024K, used 517K [0x00000000ffe80000, 0x0000000100000000, 0x0000000100000000)
  eden space 512K, 6% used [0x00000000ffe80000,0x00000000ffe87bc0,0x00000000fff00000)
  from space 512K, 95% used [0x00000000fff80000,0x00000000ffff9a28,0x0000000100000000)
  to   space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
 ParOldGen       total 512K, used 288K [0x00000000ffe00000, 0x00000000ffe80000, 0x00000000ffe80000)
  object space 512K, 56% used [0x00000000ffe00000,0x00000000ffe481c8,0x00000000ffe80000)
 Metaspace       used 2445K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 266K, capacity 386K, committed 512K, reserved 1048576K
```
A detailed explanation of how to read this `-verbose:gc` output can be found [here](https://stackoverflow.com/questions/16794783/how-to-read-a-verbosegc-output/16797404#16797404). But for our purpose, we can see that there were three garbage collection events and old generation space usage is more than 50%.

Now lets initialize the ArrayList object and run the same test.
```java
import java.util.*;

public class ArrayListProfiler {
  public static void main(String[] a) {
    List<String> array = new ArrayList<String>(10000);

    for(int i=0; i<10000; i++) {
      array.add(String.valueOf(i));
    }
  }
}
```
The output in this case comes out to be like this:
```sh
Java HotSpot(TM) 64-Bit Server VM (25.40-b25) for windows-amd64 JRE (1.8.0_40-b26), built on Mar  7 2015 13:51:59 by "java_re" with MS VC++ 10.0 (VS2010)
Memory: 4k page, physical 8365480k(1929064k free), swap 16729064k(4231388k free)
CommandLine flags: -XX:InitialHeapSize=1048576 -XX:MaxHeapSize=1048576 -XX:+PrintGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC 
0.071: [GC (Allocation Failure) [PSYoungGen: 512K->487K(1024K)] 512K->503K(1536K), 0.0008072 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 PSYoungGen      total 1024K, used 907K [0x00000000ffe80000, 0x0000000100000000, 0x0000000100000000)
  eden space 512K, 82% used [0x00000000ffe80000,0x00000000ffee9250,0x00000000fff00000)
  from space 512K, 95% used [0x00000000fff00000,0x00000000fff79c50,0x00000000fff80000)
  to   space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
 ParOldGen       total 512K, used 16K [0x00000000ffe00000, 0x00000000ffe80000, 0x00000000ffe80000)
  object space 512K, 3% used [0x00000000ffe00000,0x00000000ffe04000,0x00000000ffe80000)
 Metaspace       used 2445K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 266K, capacity 386K, committed 512K, reserved 1048576K
```
Garbage collection was needed only once and old generation space usage is only 3% because unlike the previous case, the internal arrays didn’t have to be recreated.

