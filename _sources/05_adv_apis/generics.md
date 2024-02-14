# Generics

Before embarking on the advanced APIs chapters you should know a little bit more about two topics that were 
not really discussed before but were sort of taken for granted. These topics are Generics and Annotations.

Generics is the solution in Java, a statically typed language, to deal with collections (in the broadest sense) that 
are designed to hold diverse types.  

In this chapter, I will make use of the `Animal` class and two derived classes, `Mammal` and `Bird`. All three are listed below.
Note that the record type cannot be used here because records are "frozen": they can't be subclassed.

```java
package demos.advanced.generics;

import java.util.Objects;

public abstract class Animal {
    public String name;
    public short legs;

    public Animal(String name, short legs) {
        this.name = name;
        this.legs = legs;
    }

    public void breathe() {
        System.out.println(getClass().getSimpleName() + ", breathing");
    }

    //getName(), getLegs(), toString(), equals() and hashCode() omitted
}

```

```java
package demos.advanced.generics;

public class Mammal extends Animal{

    public Mammal(String name) {
        super(name, (short)4);
    }
    
    public void giveBirth() {
        System.out.println("Giving birth to live young");
    }
}
```

```java
package demos.advanced.generics;

public class Bird extends Animal{

        public Bird(String name) {
            super(name, (short)2);
        }
        
        public void layEggs() {
            System.out.println("Laying eggs");
        }
}
```


Before generics was introduced, in Java version 5.0, programmers needed to type cast their objects when 
accessed from a collection. Proceeding with the animals data model from above, suppose we created a List with 
some animals in it:

```java
List animals = new ArrayList();

//adding some animals to the list
animals.add(new Mammal("Mouse"));
animals.add(new Mammal("Elephant"));
animals.add(new Bird("Buzzard"));
animals.add("Imposter"); //This is no animal!
```

In the snippet above, a List which holds no specific type of object is created.
Therefore, although you put specifically typed objects in it, when you try to fetch or access them,
all objects will be of type Object. 
So, this is not allowed:

```java
for (Animal animal: animals){
    System.out.println(animal);
}
```

It gives a compiler error. So, although we know that only Animal instances were added, 
this is the only allowed type reference:

```java
for (Object object: animals){
    System.out.println(object);
}
```

Note that although the reference is of type object, the instances themselves are `Bird` and `Mammal`.

:::{warning}
Instances never change type.  
Only the reference giving access to them may change type.  
Think of the reference as a remote control to the object in memory (the heap). The reference type 
defines which buttons are available to push on.
:::

If you want to do animal-ish things with them, you need to be sure of the type, otherwise 
you run into trouble:

```java
for (Object object: animals) {
    Animal animal = (Animal) object;
    animal.breathe();
}
```
<pre class="console_out">
Mammal, breathing
Mammal, breathing
Bird, breathing
java.lang.ClassCastException: class java.lang.String cannot be cast to class demos.advanced.generics.Animal (java.lang.String is in module java.base of loader 'bootstrap'; demos.advanced.generics.Animal is in unnamed module of loader 'app')
	at demos.advanced.generics.GenericsDemo.preGenericsDemo(GenericsDemo.java:23)
	at demos.advanced.generics.GenericsDemo.main(GenericsDemo.java:8)
</pre>

No compiler error this time, but a thing much worse; a runtime error! Since the list did not only contain
animals, but also a String, the type cast failed with a `ClassCastException`. The only way to be sure is to use a type check:

```java
for (Object object: animals) {
    if (object instanceof Animal) {
        Animal animal = (Animal) object;
        animal.breathe();
    } else {
        System.out.println("Not an animal: " + object.getClass().getSimpleName());
    }
}
```
<pre class="console_out">
Mammal, breathing
Mammal, breathing
Bird, breathing
Not an animal: String
</pre>

So, it should be clear that there was quite a need for type-safe collections.  
Enter generics.  

```java
List<Animal> animals = new ArrayList<>();
```

The diamond operators `<>` are used to specify which type your collection will hold. 
Other types will not be allowed by the compiler. So the statement `animals.add("Imposter");` will 
fail to compile and type checks and type casting are not required anymore.

```java
animals.add(new Mammal("Mouse"));
animals.add(new Mammal("Elephant"));
animals.add(new Bird("Buzzard"));
//animals.add("Imposter"); //not allowed anymore

for (Animal animal: animals) {
    animal.breathe();
}
```

Before delving into some more complex aspects of generics, let's first look at a variation of 
the `java.util.Optional` class: `MyOptional`.  

```java
package demos.advanced.generics;

import java.util.function.Supplier;

public class MyOptional<T> {
    private T value;

    private MyOptional(T value) {
        this.value = value;
    }

    //factory method with value
    static <T> MyOptional<T> of(T value) {
        return new MyOptional<T>(value);
    }

    //factory method when no value is available
    static <T> MyOptional<T> empty() {
        return new MyOptional<T>(null);
    }

    public T getValueOrDefault(Supplier<T> supplier) {
        if (value != null) {
            return value;
        } else {
            return supplier.get();
        }
    }
}
```

This class definition states that instances of it are going to hold and process a `value`. 
The type of the value is not known beforehand, but you have to (or, more correct, should) 
pass this type at declaration time. 
Once you have an instance of this class, all its methods operate on the type that was 
passed in the diamond operators at declaration time.  

The `java.util.function.Supplier` interface is something of the Functional programming API, 
discussed in more detail in a following chapter.  
Here is some usage of this class.

```java
MyOptional<String> stringOptional = MyOptional.of("Hello, ");
String valueOrDefault1 = stringOptional.getValueOrDefault(() -> "World");
System.out.println(valueOrDefault1.toUpperCase()); // No casting required because of generic declaration

MyOptional<String> emptyOptional = MyOptional.empty();
String valueOrDefault2 = emptyOptional.getValueOrDefault(() -> "World");
System.out.println(valueOrDefault2.toUpperCase());
```
<pre class="console_out">
HELLO, 
WORLD
</pre>

Note that `() -> "World"` is a lambda expression that is exactly the same as 

```java
new Supplier<String>() {
    @Override
    public String get() {
        return "World";
    }
}
```

Let's extend this to a slightly more complex example. 
Suppose you want to have a utility method that is intended to be used to 
let all animals in a given list breathe.  

This is your first attempt:

```java
package demos.advanced.generics;
import java.util.List;

public class Animals {
    static void breather(List<Animal> animals) {
        for (Animal animal : animals) {
            animal.breathe();
        }
    }
}
```
Nothing funny going on here. Let's try this out:  

```java
List<Animal> animals = new ArrayList<>();
animals.add(new Mammal("Mouse"));
animals.add(new Mammal("Elephant"));
animals.add(new Bird("Buzzard"));

Animals.breather(animals); //OK
```

Still no problem. But then you try to pass it another list, this time only containing Mammals, 
and also declared as such:

```java
List<Mammal> mammals = new ArrayList<>();
mammals.add(new Mammal("Mouse"));
mammals.add(new Mammal("Elephant"));
Animals.breather(mammals); //Compile error
```

You get a compiler error! `Required type: List<Animal> Provided: List<Mammal>`. This is because 
`breather()` only accepts Lists declared as containing Animal instances.  
A simple refactor to generics solves this problem:

```java
package demos.advanced.generics;
import java.util.List;

public class Animals {
    static <T extends Animal> void breather(List<T> animals) {
        for (Animal animal : animals) {
            animal.breathe();
        }
    }
}
```

The method signature of this generic version states that it accepts lists (or more specifically: subtypes of `List`), 
where the declared type `T` of the elements it contains is `Animal` or any of its subtypes. The specification of the 
**bounded generics** (`<T extends Animal>`) is located before the return statement.

We call the use of `<T extends Animal>` an upper-bounded type, because only Animals and its subtypes are allowed.

The same method could also have been defined using the wildcard character `?`:

```java
package demos.advanced.generics;

import java.util.List;

public class Animals {

    static void breather(List<? extends Animal> animals) {
        for (Animal animal : animals) {
            animal.breathe();
        }
    }
}
```

Finally we get to the end of the line. The `sort()` function from the `Collections` class shows all elements of generics:


```java
static <T extends Comparable<? super T>> void sort(List<T> list) {
    //sorting logic
}
```
The function parameter simply states `List<T>`, so a List holding type `T`. But the bounded 
generics declaration states `<T extends Comparable<? super T>>`. Let's dissect.

* `T extends Comparable` means all elements in List need to be of type `Comparable` or its subclasses. 
In other words, since Comparable is an interface, they should all be implementers of Comparable.  
* The Comparable interface is defined generically itself (`Interface Comparable<T> {...}`). So the expression 
`<? super T>` states that when the passed list holds elements of type T, the implemented interface by these 
elements should also deal with type T, **_or any of its supertypes_**! The use of `super` is the opposite of
`extends` in this context.

