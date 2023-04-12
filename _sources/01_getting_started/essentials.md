# The essence of Java

This chapter deals with some key characteristics of the Java programming language. Java is a very popular platform for several reasons: it is "compile once, run everywhere", it is the basis of Android, it is embedded in many domestic appliances, and it has a nice object-oriented design which makes it very suitablefor building larger (server side) applications.

## Cross-platform

Java is cross-platform in the sense that once it is compiled it can be run on any operating system (OS) that has a Java Runtime installed. When you run your app, this OS-specific Java installation instantiates a Java Virtual Machine (**JVM**), also called a Java Runtime Environment (JRE) and passes your compiled code to it.

There are several parties distributing Java (both as JRE and JDK - Java Development Kit): Oracle en OpenJava are the main ones. It does not matter which one you install: they all adhere to the Java language definitions.

It is quite an extensive story, very well explained [here](https://javapapers.com/core-java/differentiate-jvm-jre-jdk-jit/) but outside the scope of this course.

The key thing to remember is: your Java source files (`.java`) are compiled into class files (`.class`) and packaged into a zip-like archive (`.jar` - java archive) which can be distributed as executable, compatible with any OS that has a JVM installed.


## Multithreaded

Creating multithreaded applications and algorithm is extremely easy because multithreading support is build into the language from version 1, and greatly enhanced with the introduction of the Streams API in Java 8.

## Object-Oriented

In Java you don't write and run a _script_. In fact, it is impossible to write Java code outside a containing class.  
Instead, you write a program where all logic you create resides in methods within **_classes_**. Classes reside in their own source file. Java projects usually have many source files! One of these classes must contain a `main()` method for the library to be executable. You let the JVM know which class has the main method you want executed through a file called `manifest.mf`.
Besides the raw -primitive- data types such as integer (`int`) and `double`, all types are represented by classes, which define the blueprint to construct `objects`. So objects are the representations (**_instances_**) of a single class - more formally called a **_type_**. Java provides many types, and you will routinely define your own types when programming.

:::{admonition} Classes versus Objects
A Class is a Type blueprint that is used to instantiate Objects
:::

These objects, the number of which can range from one to many millions within a running application, are the fabric of the program. They hold data (called **_instance variables_**) and **_methods_** that define their behavior and functionality. 
 
## Strongly typed and scoped

This aspect can at first be a bit daunting when coming from languages such as Python or R. Every variable you use must have a **_declared type_**. Moreover, every class/object variable or method (**_member_**) has **_keywords_** defining its scope, also called visibility. 

:::{note}
In more recent Java versions you can use the keyword `var` for locally-scoped variables. They do not need to be declared of a specific type; the compiler figures that out for you.
:::


Below is an example class that shows these aspects. Read the Javadoc comments (between `/**` and `*/`) to see what everything means. You do not have to remember or understand all these keywords yet, only get an idea of their role within the code.

```java
/**
 * The package defines the NAMESPACE of this class. 
 * There can be many classes of the same name, but the package scope makes them 
 * uniquely identifiable.
 */
package snippets;

/**
 * Class Snp models a Single Nucleotide Polymorphism.
 * The class is public, so any class within the project has access to it.
 */
public class Snp {
    /**
     * These are the INSTANCE VARIABLES that model the SNPs data.
     * They are PRIVATE, so they can not be accessed or modified directly from
     * outside this class.
     * The types are long (long integer) and character (single letter).
     */
    private long position;
    private char referenceNucleotide;
    private char alternativeNucleotide;

    /**
     * This is the CONSTRUCTOR method that forces client code to provide the three
     * essential properties of a Snp that every INSTANCE should have defined 
     * before being INSTANTIATED.
     * It is declared public, so it is accessible to all other code. Properties are
     * passed as arguments to the constructor and stored as INSTANCE VARIABLES.
     * @param position a long integer
     * @param reference a single character: A, C, G or T
     * @param alternative a single character: A, C, G or T
     * @throws IllegalArgumentException if the given position is negative
     */
    public Snp(long position, char reference, char alternative) {
        /* A check is performed on the first parameter.
        * An exception (error) is generated when the value is wrong.*/
        if (position < 1) {
            throw new IllegalArgumentException("position must be positive");
        }
        this.position = position;
        this.referenceNucleotide = reference;
        this.alternativeNucleotide = alternative;
    }

    /**
     * a GETTER for the position. Since the position field is private, this makes
     * it a READ_ONLY property.
     * @return position as long integer
     */
    public long getPosition() {
        return position;
    }

    public char getReferenceNucleotide() {
        return referenceNucleotide;
    }

    public char getAlternativeNucleotide() {
        return alternativeNucleotide;
    }

    /**
     * this method tells a client whether ths SNP is a transition or a transversion.
     * The return type is boolean (true or false).
     * @return isTransition
     */
    public boolean isTransition() {
        return ((referenceNucleotide == 'A' && alternativeNucleotide == 'G') 
                 || (referenceNucleotide == 'C' && alternativeNucleotide == 'T'));
    }
}
```

## Compiled

Java is a compiled language, which means you have to compile the source into **_byte code_** before it can be run. Fortunately, you have to compile it only once, for the JVM. This is unlike for instance C and C++ projects that need to be build for each OS you want to support.  

Whenever you run an application within IntelliJ, it is first automatically compiled: all source files (`.java` files) are compiled into byte code files that have a `.class` extension. You can find these in the `build` folder. To create a distributable application, you need to build it into a "Jar" - with `.jar` extension. You run these on the command line using the 
command `java -jar <my-app.jar>`. The Gradle toolbox has all kinds of Tasks to automate this for you. 

Related to this: Run-time syntax errors do not occur (as much) as in Python for instance - they are stopped by the compiler. The compiler will not compile erroneous code.
