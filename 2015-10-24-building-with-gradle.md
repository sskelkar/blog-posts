## Building with Gradle
Gradle is a popular build tool to manage Java projects. Gradle’s build scripts are written in Groovy. The design of Gradle is aimed to be used as a language, not as a rigid framework. In this article, I want to give some basic idea about what a Gradle build script is composed of and some of the features provided by Gradle.

Gradle is based upon two basic concepts: **projects** and **tasks**. A project can be your application or a library that might be used by a different project. A project doesn’t have to be something that you build, it could be something to be performed, like deploying an application. A task represents a particular piece of work that is done in a build process, like compiling some classes or running unit tests.

Let's create a simple `hello world` build script in which we shall define a simple task named `hello`. The file name should be `build.gradle`. For normal Java projects, the build file must be present in the project’s root directory.
```gradle
task hello << {
  println "Hello World!"
}
```
Open the command prompt and navigate to the current working directory and execute this build script with `gradle hello` command. Alternatively, you can also use the command `gradle -q hello`. The **-q** option prevents the output from getting cluttered by Gradle’s log messages.

In the next section I will give a short introduction on Gradle tasks. But feel free to skip it because in actual development, you won’t normally need to write all the tasks yourself. Instead you will use the ready made tasks provided by plugins.

### Tasks

Tasks are written in Groovy and can be used to do fairly complex work, similar to what can be done using any scripting language. This is an important feature that distinguishes Gradle from other XML based build tools like Apache Ant.

Tasks can call upon each other, i.e. you can hook a task to be executed before or after another task.
```gradle
task hi << {
  print "Hi! "
}

task bye(dependsOn: hi) << {
  print "Bye "
}
```
If we run `gradle bye`, the output will be `Hi! Bye`. Because task `bye` depends on task `hi`, it is executed after `hi`.

Each task is available as a property of the build script. Meaning, if we have declared a task in the script, we can refer to it inside another task using its name and even refer to its properties in the second task.
```gradle
task task1 {
  ext.num = 100    //declaring a property
}

task task2 << {
  println "Using property defined in $task1.name: " + task1.num
}
```
If we run `gradle task2`, the output is: `Using property defined in task1: 100`.

Gradle is neatly integrated with Ant. So any Ant target can be used like a native Gradle task at runtime. Conversely, you can also declare dependencies on Gradle tasks in your build.xml.

### Plugins
Gradle ships with a number of plugins, each equipped to support different types of projects. So for example, with Java plugin, you get pre-built Gradle tasks to do stuff like building the project (`gradle build`), running unit tests (`gradle test`), creating jar (`gradle jar`) etc.

Similarly, to manage spring boot projects you need to use the **spring-boot plugin**. To add a plugin, you need to add a line like the following to your `build.gradle`.
```
apply plugin: 'java'
```
Plugins are convention based. Meaning, they expect to find your project files in certain locations. If you follow the convention imposed by a plugin, you don’t need to do much in your build script. But if you don’t want to or can’t follow the plugin convention, Gradle allows you to customize your project likewise.

### Dependencies
If your project has dependencies on external JARs, they can be declared in `dependencies` block in the build script. Dependencies are grouped into dependency configurations that represent how they are to be used in your project. Four common dependency configurations provided by Java plugin are: `compile`, `runtime`, `testCompile` and `testRuntime`.

Dependency JARs are identified using their group, name and version attributes. In the build script they are declared in the format: `group:name:version`.

For external dependencies, you also need to tell Gradle where to find them. They could be located in a Maven repository or the local file system. This can be declared in a `repositories` block in the build script. You can either use the Maven central repository or specify the location of a remote Maven repository or the Maven repository on your machine:
```gradle
apply plugin: 'spring-boot'

repositories {
  mavenCentral()    //Maven central repository
  
  maven {
    url 'http://repo.maven.apache.org/maven2'    //remote Maven repository  
  }
  
  mavenLocal()    //local Maven repository
}

dependencies {
  compile("org.springframework.boot:spring-boot-starter:${spring_boot_version}")
}
```
In the above example, the version has been declared as `${spring_boot_version}`. Instead of writing the version number in `build.gradle`, where the same version can be associated with multiple dependencies, we can configure them separately in `gradle.properties` file. So in case you need to change the version number, you only need to edit it in properties file.

### Reports
You can also see a detailed reports of some of your important tasks. You can use `--profile` option while building your application to see the time taken during various phases of the build process, including each individual task, in the profile report present at `build\reports\profile directory`. Similarly you can see a summary of the unit test by opening the `build/reports/tests/index.html` file.

### Gradle Wrapper
You have created your awesome app using Gradle and now you want to share it with others. Does everyone who clones your project need to have Gradle installed on their machine? The answer is no! Or, if someone has Gradle installed on their machine but its version is different than the one required to run your app, do they need to install and configure another version of Gradle just to run your app? The answer is again no. You just need to add a block like this in your `build.gradle`.
```gradle
task wrapper(type: Wrapper) {
    gradleVersion = '2.7'
}
```
And run `gradle wrapper` to download and initialize the wrapper scripts. This will add two new scripts to your root directory and the wrapper jar and properties file in a new folder `gradle/wrapper`.

Now even if Gradle is not installed on a machine, you can run all your build tasks using the wrapper script **gradlew** instead of **gradle**. That is, `gradlew build`, `gradlew test` and so on. When the wrapper is used for the first time, it downloads and caches the Gradle binaries of the version specified in `build.gradle` on the host machine, making it easier to distribute your app.

### Additional Tips
* Use `gradle --gui` to launch the Gradle GUI.
* Use `gradle -q tasks` to list all the executable tasks and `gradle -q properties` to list all the properties of the build script of a project, including those added by the plugins.

