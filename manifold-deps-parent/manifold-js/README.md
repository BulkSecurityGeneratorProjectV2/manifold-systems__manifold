# Manifold : Javascript

The JavaScript type manifold provides direct, type-safe access to JavaScript files
as if they were Java files.

Here we have JavaScript resource file, `abc/JsProgram.js`:
```javascript
var x = 1;

function nextNumber() {
  return x++;
}

function exampleFunction(x) {
  return x + " from Javascript";
}
```

The type manifold maps type information directly to Java's type system so that
this JavaScript program is accessible as a Java class:

```java
import abc.JsProgram;

...

  String hello = JsProgram.exampleFunction("Hello");
  System.out.println(hello); // prints 'Hello from JavaScript'
  
  double next = JsProgram.nextNumber();
  System.out.println(next); // prints '1'
  next = JsProgram.nextNumber();
  System.out.println(next); // prints '2'
```

In addition to JavaScript programs you can also access JavaScript _classes_ directly.

This JavaScript defines class `Person` from file `abc/Person.js':
```javascript
class Person {
  constructor(firstName, lastName) {
    this._f = firstName
    this._l = lastName
  }

  displayName() {
    return this._f + " " + this._l
  }

  get firstName() {
    return this._f
  }
  set firstName(s) {
    this._f = s
  }

  get lastName() {
    return this._l
  }
  set lastName(s) {
    this._l = s
  }
}
```

You can access this JavaScript class file directly as a Java class:
```java
import abc.Person;

Person person = new Person("Joe", "Bloe");
String first = person.getFirstName();
System.out.println(first);
```

You can also create simple, type-safe Javascript _templates_.  Javascript template files have a `.jst` extension
and work very similar to JSP syntax. To illustrate:

File `com/foo/MyTemplate.jst`:
```
<%@ params(names) %>
This template lists each name provided in the 'names' parameter
Names:
<%for (var i = 0; i < names.length; i++) { %>
-${names[i]}
<% } %>
The end
```
From Java we can use this template in a type-safe manner:
```java
import com.foo.MyTemplate;
...
String results = MyTemplate.renderToString(Arrays.asList("Orax", "Dynatron", "Lazerhawk", "FM-84"));
System.out.println(results);
``` 
This prints the following to the console:
```
This template lists each name provided in the 'names' parameter
Names:
-Orax
-Dynatron
-Lazerhawk
-FM-84
The end
```

For full-featured template engine functionality see project [ManTL](http://manifold.systems/manifold-templates.html).


# IDE Support 

Manifold is best experienced using [IntelliJ IDEA](https://www.jetbrains.com/idea/download).

## Install

Get the [Manifold plugin](https://plugins.jetbrains.com/plugin/10057-manifold) for IntelliJ IDEA directly from IntelliJ
via:

<kbd>Settings</kbd> ➜ <kbd>Plugins</kbd> ➜ <kbd>Marketplace</kbd> ➜ search: `Manifold`

<p><img src="http://manifold.systems/images/ManifoldPlugin.png" alt="echo method" width="60%" height="60%"/></p>

## Sample Project

Experiment with the [Manifold Sample Project](https://github.com/manifold-systems/manifold-sample-project) via:

<kbd>File</kbd> ➜ <kbd>New</kbd> ➜ <kbd>Project from Version Control</kbd> ➜ <kbd>Git</kbd>

<p><img src="http://manifold.systems/images/OpenSampleProjectMenu.png" alt="echo method" width="60%" height="60%"/></p>

Enter: <kbd>https://github.com/manifold-systems/manifold-sample-project.git</kbd>

<p><img src="http://manifold.systems/images/OpenSampleProject.png" alt="echo method" width="60%" height="60%"/></p>

Use the [plugin](https://plugins.jetbrains.com/plugin/10057-manifold) to really boost your productivity. Use code
completion to conveniently access javascript. Make changes to your Javascript files and use the changes immediately,
no compilation!  Find usages of any element in your Javascript files. Perform rename refactors to quickly and safely
make project-wide changes.

# Building

## Building this project

The `manifold-js` project is defined with Maven.  To build it install Maven and run the following command.

```
mvn compile
```

## Using this project

The `manifold-js` dependency works with all build tooling, including Maven and Gradle. It also works with Java versions
8 - 12.

Here are some sample build configurations references.

>Note you can replace the `manifold-js` dependency with [`manifold-all`](https://github.com/manifold-systems/manifold/tree/master/manifold-all) as a quick way to gain access to all of
Manifold's features.

## Gradle

Here is a sample `build.gradle` script. Change `targetCompatibility` and `sourceCompatibility` to your desired Java
version (8 - 12), the script takes care of the rest. 
```groovy
plugins {
    id 'java'
}

group 'systems.manifold'
version '1.0-SNAPSHOT'

targetCompatibility = 11
sourceCompatibility = 11

repositories {
    jcenter()
    maven { url 'https://oss.sonatype.org/content/repositories/snapshots/' }
}

dependencies {
    compile group: 'systems.manifold', name: 'manifold-js', version: '2019.1.25'
    testCompile group: 'junit', name: 'junit', version: '4.12'

    // Add manifold to -processorpath for javac
    annotationProcessor group: 'systems.manifold', name: 'manifold-js', version: '2019.1.25'
}

if (JavaVersion.current() != JavaVersion.VERSION_1_8 &&
    sourceSets.main.allJava.files.any {it.name == "module-info.java"}) {
    tasks.withType(JavaCompile) {
        // if you DO define a module-info.java file:
        options.compilerArgs += ['-Xplugin:Manifold', '--module-path', it.classpath.asPath]
    }
} else {
    tasks.withType(JavaCompile) {
        // If you DO NOT define a module-info.java file:
        options.compilerArgs += ['-Xplugin:Manifold']
    }
}

tasks.compileJava {
    classpath += files(sourceSets.main.output.resourcesDir) //adds build/resources/main to javac's classpath
    dependsOn processResources
}
tasks.compileTestJava {
    classpath += files(sourceSets.test.output.resourcesDir) //adds build/resources/test to test javac's classpath
    dependsOn processTestResources
}
```
Use with accompanying `settings.gradle` file:
```groovy
rootProject.name = 'MyProject'
```

## Maven

### Java 8

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>my-js-app</artifactId>
    <version>0.1-SNAPSHOT</version>

    <name>My Javascript App</name>

    <properties>
        <!-- set latest manifold version here --> 
        <manifold.version>2019.1.25</manifold.version>
    </properties>
    
    <dependencies>
        <dependency>
            <groupId>systems.manifold</groupId>
            <artifactId>manifold-js</artifactId>
            <version>${manifold.version}</version>
        </dependency>
    </dependencies>

    <!--Add the -Xplugin:Manifold argument for the javac compiler-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                    <encoding>UTF-8</encoding>
                    <compilerArgs>
                        <!-- Configure manifold plugin-->
                        <arg>-Xplugin:Manifold</arg>
                    </compilerArgs>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### Java 9 or later
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>my-js-app</artifactId>
    <version>0.1-SNAPSHOT</version>

    <name>My Javascript App</name>

    <properties>
        <!-- set latest manifold version here --> 
        <manifold.version>2019.1.25</manifold.version>
    </properties>
    
    <dependencies>
        <dependency>
            <groupId>systems.manifold</groupId>
            <artifactId>manifold-preprocessor</artifactId>
            <version>${manifold.js}</version>
        </dependency>
    </dependencies>

    <!--Add the -Xplugin:Manifold argument for the javac compiler-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
                <configuration>
                    <source>11</source>
                    <target>11</target>
                    <encoding>UTF-8</encoding>
                    <compilerArgs>
                        <!-- Configure manifold plugin-->
                        <arg>-Xplugin:Manifold</arg>
                    </compilerArgs>
                    <!-- Add the processor path for the plugin (required for Java 9+) -->
                    <annotationProcessorPaths>
                        <path>
                            <groupId>systems.manifold</groupId>
                            <artifactId>manifold-js</artifactId>
                            <version>${manifold.version}</version>
                        </path>
                    </annotationProcessorPaths>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

# License

## Open Source
Open source Manifold is free and licensed under the [Apache 2.0](http://www.apache.org/licenses/LICENSE-2.0) license.  

## Commercial
Commercial licenses for this work are available. These replace the above ASL 2.0 and offer 
limited warranties, support, maintenance, and commercial server integrations.

For more information, please visit: http://manifold.systems//licenses

Contact: admin@manifold.systems

# Versioning

For the versions available, see the [tags on this repository](https://github.com/manifold-systems/manifold/tags).

# Author

* [Scott McKinney](mailto:scott@manifold.systems)
