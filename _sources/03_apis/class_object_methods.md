# Methods of class Object

## Introduction

Previously you have seen some collection types: List, Map, Set. 
One of the methods that is universal to all collection types is `contains()`. 
How does the JVM determine that two objects are equal?
This is through the `equals()` method.

**_When you use the statement `myCollection.contains(myObject)`, you get `true` returned if one of the elements in myCollection returns `true` for `element.equals(myObject)`._**

The method `equals()` is there, even though you did not implement it. This is because they are declared and implemented in class `Object`, and **_every class in Java extends from Object implicitly_**. Whenever you create a class, such as this:

```java
class User {

}
```

you get this implicitly:

```java
class User extends Object {

}
```

Since User extends Object, it inherits all Object properties and methods. If you create an object (of any type), type the variable name followed by a dot, IntelliJ will show you the methods of class Object:

![Methods of class object](figures/methods_of_Object.png)

The `notify..()` and `wait..()` methods have to do with multithreading and are not dealt with here. 
The ones of interest are

- `toString()` returns a String representation of the current object. Implemented in class Object as the objects' memory hash.
- `equals()` returns a boolean indicating whether the argument 
is logically the same as the current object. Implemented in class Object as: `true` if both refer to the same object.
- `hashCode()` returns the hashCode (an int) of the current object. Implemented in class Object to return an integer representing the objects memory address.
- `getClass()` returns the class of the current object

## `toString()`: a string representation of the object

If you want to see a nice textual representation of an object during software development, you use toString(). 
In IntelliJ, type `ctrl + O` if you want to override a super class method.

![Override methods](figures/override_methods.png)

If you choose toString(), you will get something like this:

```java
@Override
public String toString() {
    return super.toString();
}
```

Note the `@Override` **_annotation_**. 
Annotations are syntactic sugar in Java, and this one says to the compiler: 
please check whether this override is correct, else fail compilation.
Correct with respect to toString() means: take no arguments and return a String.
The statement `return super.toString();` simply states: call the super class toString (of Object in this case) and return its result. Which means you get the uninformative:

`snippets.apis.User@78b1cc93`

To make toString useful, you'll need te redefine its logic, for instance:

```java
@Override
public String toString() {
    return "A user with name " + name;
}
```

which outputs `A user with name Henk`

Although this is logically correct, and will compile just fine, there are some conventions used in Java to be informative. To get a toString() which follows this rule, use another IntelliJ shortcut: `ctrl + N` (for generate code), and select toString().

![Generate toString()](figures/generate_code_toString.png)

Then, select the instance variables you want included in the string representation, click OK and you have something like this

```java
@Override
public String toString() {
    return "User{" +
            "id=" + id +
            ", name='" + name + '\'' +
            '}';
}
```

outputting `User{id=15, name='Henk'}`, which is a good textual representation of the underlying object.

## `equals()` determining logical similarity between objects

Consider these three User objects, where the first constructor argument is the user ID and the second the username.

```java
User user1 = new User(15, "Henk");
//same, equal?
User user2 = user1;

User user3 = new User(21, "Dirk");
//same, or equal User as user3?
User user4 = new User(21, "Dirk");
```

Is `user1` equal to `user2`? Is `user1` equal to itself, `user1`? Is `user3` equal to `user4`?  
There is an important distinction to be made: **_equality versus sameness_**. 
Two references are considered the same if they both _point to a single object on the heap_. 
On the other hand, equality is a more fuzzy concept. In general, we consider 
two references equal if they point either to one single object,
or to two distinct objects that are logically similar.

So, in the above code snippet, `user1` is definitely the same as `user` two since they refer to the same object. 
But since they are the same, they must therefore also be equal.  
On the other hand, `user3` and `user4` are two distinct objects (the constructor has run twice!), 
but **_logically similar_** because they have exactly the same internal data.

In Java, `==` tests for sameness and `equals()` for logical similarity.

Therefore, we would like our User class to behave like this.

```java
System.out.println("user1 == user1 -- " + (user1 == user1)); //should be true
System.out.println("user1.equals(user1) -- " + user1.equals(user1)); //should be true
System.out.println("user1 == user2 -- " + (user1 == user2)); //should be true
System.out.println("user1.equals(user2) -- " + user1.equals(user2)); //should be true
System.out.println("user1.equals(user3) -- " + user1.equals(user3)); //should be false
System.out.println("user3.equals(user4) -- " + user3.equals(user4)); //should be true!
```

but in actuality this is what we get

<pre class="console_out">
user1 == user1 -- true
user1.equals(user1) -- true
user1 == user2 -- true
user1.equals(user2) -- true
user1.equals(user3) -- false
user3.equals(user4) -- false
</pre>

Up to the last test, everything works out fine. Why does the last test return false? Because Java doesn't not know about logical similarity between User objects! We have to tell it. Therefore, by default this is what the `Object.equals()` method does:

```java
@Override
public boolean equals(Object other) {
    return (this == other);
}
```

Let's change the `equals()` method to reflect (my opinion of) similarity:

```java
@Override
public boolean equals(Object other) {
    User otherUser = (User) other;
    return (this.id == otherUser.id && this.name == otherUser.name);
}
```

There is a lot to say about this implementation. Can you spot the flaws?

There are several, so maybe it is better to let IntelliJ handle this for us, via `ctrl + N` (generate) -> `equals() and hashCode()` -> choose template (Java 7+), choose name and ID --> next, next next, Finish:

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    User user = (User) o;
    return id == user.id &&
            Objects.equals(name, user.name);
}
```

Study this carefully! Why is each statement inserted?

The `equals()` method must exhibit the following properties:  
- **_Symmetry_**: For two references, a and b, a.equals(b) if and only if b.equals(a) as well
- **_Reflexivity_**: For all non-null references, a.equals(a)
- **_Transitivity_**: If a.equals(b) and b.equals(c), then a.equals(c)
- **_Consistency with hashCode()_**: Two equal objects must have the same hashCode() value


## `hashCode()`: required when `equals()` is implemented

The `hashCode()` method is kind of the mysterious sister of `equals()`. You implement them together, but only equals() is easily understood. 

The `hashCode()` method is used for **_bucketing_** in Hash implementations like HashMap, HashTable, HashSet, etc.
The value received from `hashCode()` is used as the bucket number for storing elements of the set/map. 
This bucket number is the address of the element inside the set/map.
When you do `contains()` it will take the hash code of the element, then look for the bucket where hash code points to. 
If more than 1 element is found in the same bucket (multiple objects can have the same hash code), 
then it uses the `equals()` method to evaluate if the objects are equal, and then decide 
if `contains()` is true or false, or decide if element could be added in the set or not.
(_This paragraph is copied from a post on [Stackoverflow](https://stackoverflow.com/questions/3563847/what-is-the-use-of-hashcode-in-java)_)

The general contract of hashCode() states:  

- Whenever it is invoked on the same object more than once, `hashCode()` must consistently return the same value, provided no information used in equals comparisons on the object is modified. This value needs not remain consistent from one execution of an application to another execution of the same application

- If two objects are equal according to the `equals(Object)` method, then calling the `hashCode()` method on each of the two objects must produce the same value

- It is not required that if two objects are unequal according to the `equals(java.lang.Object)` method, then calling the hashCode method on each of the two objects must produce distinct integer results. However, developers should be aware that producing distinct integer results for unequal objects improves the performance of hash tables

Of course, we let IntelliJ do the hard work (`ctrl + N`):

```java
@Override
public int hashCode() {
    //uses a utility method from class Objects
    return Objects.hash(id, name);
}
```

## `getClass()` gives meta-information the objects class

Example code is best here:

```java
System.out.println("user1.getClass().getSimpleName() = " + user1.getClass().getSimpleName());
System.out.println("user1.getClass().getName() = " + user1.getClass().getName());
System.out.println("user1.getClass().getPackageName() = " + user1.getClass().getPackageName());
```

outputs

<pre class="console_out">
user1.getClass().getSimpleName() = User
user1.getClass().getName() = snippets.apis.User
user1.getClass().getPackageName() = snippets.apis
</pre>

There are a lot more methods in class `java.lang.Class`, but they serve difficult stuff, like introspection.

