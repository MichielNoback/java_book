# Access modifiers & Other keywords

Several access modifiers and keywords have already been introduced, explicitly or implicitly. This post gives an overview of the most important ones.

## Access Modifiers

Access modifiers are used in the declaration of a method, field, or inner class -collectively called **_members_**- to control access to them.
Using no modifier is an (implicit) modifier, giving **_default_** - but very distinct! - behavior.

- **`private`**
    Private members can only be accessed by other members of their own class.

- **`protected`**
    Protected members can only be accessed by members of their own class, that class's subclasses or classes from the same package.

- **`public`**
    Public members can be accessed by the members of any class.

- **_default_** access occurs when no explicit access modifier is attached to a member. Members with default access can be accessed by classes within the same package.

This is the same info in table form:

| Keyword             | Class | Package | Subclass | All |
|---------------------|-------|---------|----------|-----|
| public              | &#x2713;     | &#x2713;       | &#x2713;        | &#x2713;   |
| protected           | &#x2713;     | &#x2713;       | &#x2713;        |    |
| default/no modifier | &#x2713;     | &#x2713;       |         |    |
| private             | &#x2713;     |        |         |    |

Careful choice of access levels is fundamental to good Java design. Although there are no strict rules, fields (instance and class variables) should usually be private, where public (or default) getters and/or setters privide read and/or write access. API methods should be public. When classes in the same package work intimately together, default access can be appropriate. The same holds for inheritance relationships and the `protected` modifier. 

:::{admonition} Design Rule
Minimize the accessibility of class members (fields and methods).
:::

Changing public API methods is almost impossible since it will create backward incompatibility.


## Other keywords

This is a list of keywords in the Java programming language. I omitted the access modifiers (discussed separately), the primitive types, flow control keywords as well as some I do not deem essential in an introduction of Java. You cannot use any of the these as identifiers. Text was copied from [Wikipedia](https://en.wikipedia.org/wiki/List_of_Java_keywords) and edited. 

- **`abstract`**
    A method with no definition must be declared as abstract and the class containing it must be declared as abstract. Abstract classes cannot be instantiated. Abstract methods must be implemented in the first concrete subclass. Note that an abstract class isn't required to have an abstract method at all.

- **`assert`** (added in J2SE 1.4)
    Assert describes a predicate (a true–false statement) placed in a Java program to indicate that the developer thinks that the predicate is always true at that place. If an assertion evaluates to false at run-time, an assertion failure results, which typically causes execution to abort. 

- **`class`**
    A type that defines the implementation of a particular kind of object. A class definition specifies the interfaces the class implements and the immediate superclass of the class. If the superclass is not explicitly specified, the superclass is implicitly Object. The `class` keyword can also be used in the form `Class.class` to get a Class object without needing an instance of that class. For example, `String.class` can be used instead of doing new `String().getClass()`.

- **`enum`** (added in J2SE 5.0)[3]
    A Java keyword used to declare an enumerated type. Enumerations extend the base class Enum.

- **`extends`**
    Used in a class declaration to specify the superclass; used in an interface declaration to specify one or more superinterfaces. Class X extends class Y to add functionality, either by adding fields or methods to class Y, or by overriding methods of class Y. An interface Z extends one or more interfaces by adding methods. Class X is said to be a subclass of class Y; Interface Z is said to be a subinterface of the interfaces it extends.
    Also used to specify an upper bound on a type parameter in Generics.

- **`final`**
    Define an entity once that cannot be changed nor derived from later. More specifically: a final class cannot be subclassed, a final method cannot be overridden, and a final variable can occur at most once as a left-hand expression on an executed command. All methods in a final class are implicitly final.

- **`implements`**
    Included in a class declaration to specify one or more interfaces that are implemented by the current class. A class inherits the types and abstract methods declared by the interfaces.

- **`import`**
    Used at the beginning of a source file to specify classes or entire Java packages to be referred to later without including their package names in the reference. Since J2SE 5.0, import statements can import static members of a class.

- **`interface`**
    Used to declare a special type of class that only contains abstract or default methods, constant (static final) fields and static interfaces. It can later be implemented by classes that declare the interface with the `implements` keyword. 

- **`new`**
    Used to create an instance of a class or array object.

- **`package`**
    A Java package is a group of similar classes and interfaces. Packages are declared with the `package` keyword. Classes have a package declaration at the top of the source file.

- **`static`**
    Used to declare a field, method, or inner class as a class field. Classes maintain one copy of class fields regardless of how many instances exist of that class. `static` also is used to define a method as a class method. Class methods are bound to the class instead of a specific instance, and can only operate on class fields. (Classes and interfaces declared as static members of another class or interface are actually top-level classes and are not inner classes.)

- **`super`**
    Used to access members of a class inherited by the class in which it appears. Allows a subclass to access overridden methods and hidden members of its superclass. The super keyword is also used to forward a call from a constructor to a constructor in the superclass.
    Also used to specify a lower bound on a type parameter in Generics.

- **`this`**
    Used to represent an instance of the class in which it appears. `this` can be used to access class members and as a reference to the current instance. The `this` keyword is also used to forward a call from one constructor in a class to another constructor in the same class.

- **`throw`**
    Causes the declared exception instance to be thrown. This causes execution to continue with the first enclosing exception handler declared by the catch keyword to handle an assignment compatible exception type. If no such exception handler is found in the current method, then the method returns and the process is repeated in the calling method. If no exception handler is found in any method call on the stack, then the exception is passed to the thread's uncaught exception handler.

- **`throws`**
    Used in method declarations to specify which exceptions are not handled within the method but rather passed to the next higher level of the program. All uncaught exceptions in a method that are not instances of RuntimeException must be declared using the `throws` keyword.

- **`void`**
    The `void` keyword is used to declare that a method does not return any value.

