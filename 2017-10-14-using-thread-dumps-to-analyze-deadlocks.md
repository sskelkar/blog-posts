## Using thread dumps to analyse deadlocks
In a multi-threaded Java application, a deadlock occurs when two threads wait forever attempting to acquire locks that are held by each other. Here’s a simple example to simulate a deadlock:
```java
public class Deadlock {
    private Object lock1;
    private Object lock2;

    public Deadlock(Object lock1, Object lock2) {
        this.lock1 = lock1;
        this.lock2 = lock2;
    }

    public void methodA() {
        System.out.println("trying to acquire lock1 from - " + Thread.currentThread().getName());
        synchronized (lock1) {
            someLongRunningTask();
            methodB();
        }
    }

    public void methodB() {
        System.out.println("trying to acquire lock2 from - " + Thread.currentThread().getName());
        synchronized (lock2) {
            someLongRunningTask();
            methodA();
        }
    }

    private void someLongRunningTask() {
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[]args) {
        Object lock1 = new Object();
        Object lock2 = new Object();

        new Thread(() -> new Deadlock(lock1, lock2).methodA()).start();
        new Thread(() -> new Deadlock(lock1, lock2).methodB()).start();
    }
}
```
First thread calls `methodA` and acquires `lock1`. Second thread calls `methodB` and acquires `lock2`. Then the first thread calls `methodB` while at the same time the second thread calls `methodA`. Both are trying to acquire a lock that is already held by another thread, so neither can proceed further.

Thread dumps can be very useful to identify the source of such deadlocks in an application. A typical thread dump gives us information, precise to  the method name and line number, where a thread is stuck trying to acquire a lock. To generate a thread dump, we can use the `jstack` utility provided by JVM.
```
jstack -l <pid>
```
We can use `jps` command to know the `pid` of the deadlocked Java process.

Here’s a snippet of the thread dump of the above code, that gives us the deadlock information:
```
Found one Java-level deadlock:
=============================
"Thread-1":
  waiting to lock monitor 0x00007fc89a037b58 (object 0x000000076abda8f0, a java.lang.Object),
  which is held by "Thread-0"
"Thread-0":
  waiting to lock monitor 0x00007fc89a0350b8 (object 0x000000076abda900, a java.lang.Object),
  which is held by "Thread-1"

Java stack information for the threads listed above:
===================================================
"Thread-1":
  at jcip.Deadlock.methodA(Deadlock.java:15)
  - waiting to lock <0x000000076abda8f0> (a java.lang.Object)
  at jcip.Deadlock.methodB(Deadlock.java:24)
  - locked <0x000000076abda900> (a java.lang.Object)
  at jcip.Deadlock.lambda$main$1(Deadlock.java:41)
  at jcip.Deadlock$$Lambda$2/1791741888.run(Unknown Source)
  at java.lang.Thread.run(Thread.java:745)
"Thread-0":
  at jcip.Deadlock.methodB(Deadlock.java:23)
  - waiting to lock <0x000000076abda900> (a java.lang.Object)
  at jcip.Deadlock.methodA(Deadlock.java:16)
  - locked <0x000000076abda8f0> (a java.lang.Object)
  at jcip.Deadlock.lambda$main$0(Deadlock.java:40)
  at jcip.Deadlock$$Lambda$1/363771819.run(Unknown Source)
  at java.lang.Thread.run(Thread.java:745)

Found 1 deadlock.
```

