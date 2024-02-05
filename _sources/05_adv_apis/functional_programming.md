# Functional Programming

Functional programming is a programming paradigm that treats computation as the evaluation of functions and avoids state-changing operations and mutable data. In functional programming, functions are first-class citizens, which means they can be assigned to variables, passed as arguments to other functions, and returned as values from other functions.  


## Lambdas & functional interfaces
Java has a very extensive API for functional programming. At the heart of this are Lambdas. Since it is not possible to assign methods to variables, lambdas and functional interfaces have been introduced to solve this problem.

To understand what lambdas are we need to understand functional interfaces. Consider this interface:

```java
public interface NumberCombiner {
    int combine(int i, int j);
}
```

It is an interface with a single (non-default) method, `combine()`. You could create a fully fledged class implementing this interface, like this:

```java
public class AdditionCombiner implements NumberCombiner{
    @Override
    public int combine(int i, int j) {
        return i + j;
    }
}
```

This class can reside in its own source file (AdditionCombiner.java), or be defined as a (named) inner class.
You could also create an anonymous inner class, which will look like this:

```java
NumberCombiner multiplyCombiner = new NumberCombiner() {
    @Override
    public int combine(int i, int j) {
        return i * j;
    }
};
```

This can be a class member or even a method-local inner class.  

As you can see, this is quite a bit of code to define a single functionality. That is why lambdas were introduced. They represent anonymous implementers of interfaces with a single abstract method. This last part is essential because otherwise the compiler will not be able to "see" which abstract method of the interface is implemented. 

:::{admonition} Functional Interface
:class: note
In Java, interfaces with a single abstract method are called **functional interfaces**.  
They can be implemented using lambdas.  
Usually their class definitions have the `@FunctionalInterface` annotation.
Here is an example:

```java
@FunctionalInterface
public interface NumberCombiner {
    int combine(int i, int j);
}
```
:::

This is the lambda implementation version of the above example.

```java
NumberCombiner multiplyCombiner = (i, j) -> j * j;
```

:::{admonition} Lambda
:class: note
A lambda expression is a short block of code which takes in zero or more parameters and returns a single value.  They represent anonymous implementers of functional interfaces (FI).  
They have this form:  
`(<args>) -> <Expression to execute with args>`  
or, when more than one expressions are needed,  
`(<args>) -> {<Expression one; expression two; ...>}`  
:::


### Generics in Functional Interfaces

Since combining of arguments into a single result is something that is done quite often, with Integers, Doubles, even Strings, it would be very nice to define these interfaces so that all kinds of arguments and return values would be accepted.  
Fortunately, such a mechanism already exists in Java for quite some time, and it is called **Generics**. 
Generics is all over the Collections API, and you may have seen it elsewhere as well.     
Here is a generic version of the NumberCombiner.

```java
@FunctionalInterface
interface Combiner<E, R>{
    R combine(E x, E y);
}
```

It accepts two parameters of type `E` and returns a single value of type `R`. 

Below is some method that expects to be handed such a combiner, specifying that it is accepting 
only specific implementations that are working on Integers and returning a Double:

```java
void useAcombiner(Combiner<Integer, Double> combiner) {
    System.out.println("Combiner combines 2 and 4: "
            + combiner.combine(2, 4));
}
```

If we wanted to create a classic implementer of this _generic_ `Combiner` interface we could create something like this:

```java
class DivisionCombiner implements Combiner<Integer, Double> {
    @Override
    public Double combine(Integer x, Integer y) {
        return (double)x / y;
    }
}
```

and subsequently use it in this way:

```java
DivisionCombiner divisionCombiner = new DivisionCombiner();
useAcombiner(divisionCombiner);
```

Taking it just a bit less verbose we could achieve the same with an anonymous method-local inner class:

```java
useAcombiner(new Combiner<Integer, Double>() {
    @Override
    public Double combine(Integer x, Integer y) {
        return Double.valueOf(x) / y;
    }
});
```

But the lambda way is of course so much nicer:

```java
useAcombiner((x, y) -> (Double.valueOf(x) / y));
```

Here, the definition and implementation of the DivisionCombiner go hand in hand, but it is still what is happening: 
a specific implementer of the functional interface is defined and implemented in a single expression.


### Lambda syntax rules

1. **The argument list**. The parameters are comma-separated and enclosed by parentheses.
If a single parameter is defined, the parentheses are optional. On the other hand, when no parameters are specified they are required. The type of the parameters is usually optional, as this can mostly -but not always- be inferred by the compiler. Sometimes, for readability, they are included anyway. 

2. **The arrow `->`**. This is an essential part. Without it there is no lambda.

3. **The body**. The body only needs braces `{}` when multiple statements are included. When a single expression is present, the result of that expression is the return value. Otherwise, an explicit `return` statement is required when a return value is needed (i.e. no `void` type). So, this is a valid lambda:
    ```java
    (n -> {
        n = (int)(n * 10 * Math.random());
        return n % 2 == 0;
    })
    ```


### Package `java.util.function`

So far you have seen only custom interfaces in this chapter. However, operations like "combine two objects (of one type) and return another" are so common in functional programming that the Java developers have already defined a set that will serve you well over most use cases.  
For instance, there is the `Combiner` equivalent, `BiFunction<T, U, R>` that represents a function that accepts two arguments and produces a result. Of course, `T` and `U` and `R` can be of the same type.  

Have a look at [the `java.util.function` docs](https://docs.oracle.com/en/java/javase/18/docs/api/index.html) for a complete listing.

Here is a small listing of some of the most-used interfaces:  

* **`BiFunction<T,U,R>`** Represents a function that accepts two arguments and produces a result.
* **`BiPredicate<T,U>`** Represents a predicate (boolean-valued function) of two arguments.
* **`Consumer<T>`** Represents an operation that accepts a single input argument and returns no result.
* **`Function<T,R>`** Represents a function that accepts one argument and produces a result.
* **`Predicate<T>`** Represents a predicate (boolean-valued function) of one argument.
* **`upplier<T>`** Represents a supplier of results.
* **`UnaryOperator<T>`** Represents an operation on a single operand that produces a result of the same type as its operand.






