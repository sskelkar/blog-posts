## Introduction to jdb
`jdb` (Java Debugger) is a simple command-line debugger for Java classes that is provided as part of the JDK tools and utilities.

`jdb` is based on a server-client model. While debugging, you have one JVM where the code is executed and another JVM where debugger runs. Either VMs can act as the server. There are two ways to start the debugger. You can directly fire up the debugger by giving the main class name with the jdb command.
```
jdb MainClassName
```
You may of course need to give a relative path depending on the current directory of the command prompt. When debugger is started this way, jdb internally invokes a second JVM, loads the specified class and suspends the execution before that class’s first instruction.

A more common way to use debugger is to attach it to a running JVM whose code has to be debugged. This connection is build upon Java Platform Debugger Architecture. To quote Oracle documentation:
> A JDPA Transport is a method of communication between debugger and the target VM. The communication is connection oriented – one side acts as a server, listening for a connection. The other side acts as a client and connects to the server. JPDA allows either the debugger application or the target VM to act as the server. Transport implementations can allow communications between processes running on a single machine, on different machines, or either. When establishing a connection a transport addresses is used to identify the end-point of the connection. The format of a transport address depends on the type of transport.

The mode of transport can be based on shared memory or socket. Socket transport is more commonly used. A JVM that has to be debugged with `jdb` must be started with the following options:
```
-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=1044
```
This will start the JVM and suspend the execution and listen on port `1044`. To attach a debugger to this JVM, you need to fire `jdb` with following options:
```
jdb -connect com.sun.jdi.SocketAttach:hostname=localhost,port=1044
```
If you are using Gradle to build your application, you can add following block to your build file to add debugging options to the run task:
```sh
run {
    if(System.getProperty('DEBUG', 'false') == 'true') {
        jvmArgs '-Xdebug',
            '-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=1044'
    }
}
```
To run the application in debug mode, you can use the following command:
```
gradle -DDEBUG=true run
```
To attach debugger to this VM, open another command prompt and run the `jdb` command with connect option as stated above.

When your debugger is running, you can set breakpoints in your code by either specifying the line number or you can ask the debugger to pause execution when the control enters a particular method:
```
stop at package.ClassName:lineNumber
```
```
stop in package.ClassName.methodName
```
After setting the breakpoints give `run` command in the debugger window to start the suspended application. The code will execute till it hits a breakpoint, where it will suspend again.

To see the values of primitive types or contents of any object at the breakpoint, you can use following commands:
```
dump objectName
print primitiveVariableName
```
Note that to see print local variables, the classes must be compiled with `-g` option. Eg, `javac -g ClassName.java`. Using print command on objects gives their object reference.

If you have attached source code to `jdb` by running it with `-sourcepath` option, you can use `list` command to see at which line of code the execution is currently paused.

To step through the code you have `step` command. If the current line has a function invocation, this command will step into that function. If you want to go to the next line of code in the current stack frame without stepping into the function, use `next` command. To go back up the call stack use `step up`. You can use `stepi` to execute the next instruction in the code.

To *unpause* the program execution till the next breakpoint or end, you can use `cont`. To delete breakpoint use `clear`.

`threads` will list all the running threads grouped into system and main. Thread numbers are displayed in hexadecimal. To switch to a particular thread, use `thread <threadNumber>`, where thread number has to be given in decimal format. `where all` will provide information on which points in all the threads the execution is paused.

Using `jdb` is time consuming and involves a lot of manual work. But if for some reason you are stuck with an unfamiliar IDE, `jdb` can get you going because it is shipped with the JDK. Not to mention that working on command line is kind of badass in itself!
