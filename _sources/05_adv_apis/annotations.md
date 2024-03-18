# Annotations

In Java, annotations provide metadata about a program, which can be used by the compiler 
or at runtime to perform specific actions or validations. Annotations start with the 
`@` symbol followed by the annotation name.

Annotations can be applied to various elements such as classes, methods, fields, 
parameters, and packages. They provide a way to add extra information to these elements 
without affecting their core functionality.


## Existing annotations 

The Java core language already has a few annotations. Here are the main ones.

* `@Override` Is used to indicate that a method will be overriding the method with the same 
signature in a parent class. It is used to throw compile time errors when the implementation 
is not a valid override.  
* `@Deprecated` marks a method or class as deprecated: replaced by newer functionality. Used by IDEs
* `@SuppressWarnings`. Used to indicate that warnings on code compilation should be ignored. 
Note that compile errors can never be ignored!
* `@FunctionalInterface` Is used for interfaces that can be implemented as lambdas. It indicates
that an interface cannot have more than one abstract method. The compiler will throw an error in 
case there is more than one.

Annotations are extensively used in Java frameworks like Spring, JPA (Java Persistence API), 
and JUnit for various purposes such as dependency injection, ORM mapping, and unit testing.  
They provide a way to configure and customize the behaviour of these frameworks without 
cluttering the codebase with configuration details.

It is best understood how they work through creating and using your own annotation. The next section 
describes the scenario of a simple Serializable use case: all objects marked as Serializable 
should be written to file at application shutdown time.

## Custom annotations

### Declaration
Annotations are declared using the `@interface` keyword

```java
public @interface Serializable {

}
```

### Retention Policy
Annotations can be retained at compile time, runtime, or discarded altogether. This is specified using an 
annotation from the `java.lang.annotation` package, the `@Retention` annotation. Our `@Serializable` 
annotation is supposed to be used at runtime:

```java
package demos.advanced.annotations;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME)
public @interface Serializable {

}
```

### Target

Annotations can be applied to various **_targets_** like `TYPE` (class, interface, enum), 
`METHOD` (a.k.a. functions), `FIELD`, etc. 
This is also specified using an annotation from the `java.lang.annotation` package, the `@Target` annotation. 
In this case, the `@Serializable` annotation is meant to annotate classes as being serializable, so the target is `TYPE`.

```java
package demos.advanced.annotations;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Serializable {

}
```

Besides being marked as Serializable, we need to specify which fields (a.k.a. instance variables) are 
going to be serialized. To this end, a second annotation is defined:

```java
package demos.advanced.annotations;
//same imports

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface SerializableField {
    String name() default "";
}
```

### Usage
Annotations are used by prefixing them with the `@` symbol followed by the annotation name. They can be used wherever applicable with the given `@Target`in the code. For example, here is a class where the above two annotations were used:

```java
package demos.advanced.annotations;

@Serializable // -> @Target(ElementType.TYPE)
public class Student {
    @SerializableField // -> @Target(ElementType.FIELD)
    private String firstName;

    @SerializableField
    private String lastName;

    @SerializableField(name="student-ID")
    private int studentId;

    //not interested in serializing this field
    private String email;

    public Student(String firstName, String lastName, int studentId) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.studentId = studentId;
    }

    //getters omitted
}

```

### Extracting annotation

So now we have some annotations defined and used, this is how we can 
extract and use these at runtime.

```java
package demos.advanced.annotations;

import java.lang.reflect.Field;

public class AnnotationProcessor {
    public static void main(String[] args) {
        Student s = new Student("John", "Doe", 12345);
        try {
            serialize(s);
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }

    private static void serialize(Object o) throws IllegalAccessException {
        Class clazz = o.getClass();
        if (clazz.isAnnotationPresent(Serializable.class)) {
            System.out.println("Class: " + clazz.getName());
            for (Field field : clazz.getDeclaredFields()) {
                if (field.isAnnotationPresent(SerializableField.class)) {
                    //make private fields accessible
                    field.setAccessible(true);
                    System.out.println("Field! name= '" + field.getName() +
                            "' annotation name='" + field.getAnnotation(SerializableField.class).name() +
                            "' value='" + field.get(o) + "'");
                }
            }
        } else {
            System.out.println("Class: " + clazz.getName() + " is not serializable");
        }
    }
}
```
<pre class="console_out">
Class: demos.advanced.annotations.Student
Serializable field! name= 'firstName' annotation name='' value='John'
Serializable field! name= 'lastName' annotation name='' value='Doe'
Serializable field! name= 'studentId' annotation name='student-ID' value='12345'
</pre>

### Finding all annotated classes (optional)

Usually, as a regular Java developer, you will simply use existing frameworks that have their 
own annotation processors. When you are creating your own framework you could use existing libraries for this 
task. The code below is simply for demonstration purposes.

To get back to the use case of writing all annotated objects to some external 
location, this is how you could do that in native Java.

```java
package demos.advanced.annotations;

import java.io.File;
import java.io.IOException;
import java.net.URL;
import java.util.ArrayList;
import java.util.Enumeration;
import java.util.List;

public class ObjectSerializer {
    public static void main(String[] args) {
        Class<?>[] classes = getClasses("demos.advanced.annotations");
        for (Class<?> clazz : classes) {
            if (clazz.isAnnotationPresent(Serializable.class)) {
                System.out.println("Class: " + clazz.getName());
                for (java.lang.reflect.Field field : clazz.getDeclaredFields()) {
                    if (field.isAnnotationPresent(SerializableField.class)) {
                        System.out.println("Field: " + field.getName());
                    }
                }
            }
        }
    }

    private static List<Class<?>> findClasses(File directory, String packageName) {
        List<Class<?>> classes = new ArrayList<Class<?>>();
        if (!directory.exists())
            return classes;

        File[] files = directory.listFiles();
        for (File file : files) {
            if (file.isDirectory()) {
                assert !file.getName().contains(".");
                classes.addAll(findClasses(file,
                        (!packageName.equals("") ? packageName + "." : packageName) + file.getName()));
            } else if (file.getName().endsWith(".class"))
                try {
                    classes.add(Class
                            .forName(packageName + '.' + file.getName().substring(0, file.getName().length() - 6)));
                } catch (ClassNotFoundException e) {
                    System.err.println(e.getMessage());
                }
        }
        return classes;
    }

    public static Class<?>[] getClasses(String packageName) {
        ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
        assert classLoader != null;
        String path = packageName.replace('.', '/');
        Enumeration<URL> resources = null;
        try {
            resources = classLoader.getResources(path);
        } catch (IOException e) {
            System.err.println(e.getMessage());
        }
        List<File> dirs = new ArrayList<File>();
        while (resources.hasMoreElements()) {
            URL resource = resources.nextElement();
            dirs.add(new File(resource.getFile()));
        }
        List<Class<?>> classes = new ArrayList<Class<?>>();
        for (File directory : dirs)
            classes.addAll(findClasses(directory, packageName));

        return classes.toArray(new Class[classes.size()]);
    }
}

```
<pre class="console_out">
Class: demos.advanced.annotations.Teacher
Field: firstName
Field: lastName
Field: teacherId
Class: demos.advanced.annotations.Student
Field: firstName
Field: lastName
Field: studentId
</pre>