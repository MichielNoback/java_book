# Methods and Constructors

This chapter constitutes the link between basic syntax and design. Variables, operators and flow control form the basis of your algorithms. 
These algorithms have to be organized and combined into bigger systems, and building systems is done by combining methods in classes and these classes in packages and dependency graphs.  

Methods are pieces of code that are used as a "black box": you don't need to know how they work, as long as you know (and trust) what goes in and what comes out.

There exists quite a variety of method-like structures in Java:

- **_Instance methods_**: Methods that operate on objects and have access to both class and instance variables and methods. 
- **_class methods_**: Methods that operate on classes and have access only to class (static) variables and methods. 
- **_constructors_ (_object initializers_)**: Methods that specify how objects should be initialized.
- **_object initializers_**: Code blocks that specify how objects should be initialized. These run before any constructor but are parameterless (no input arguments);
- **_class initializers_**: Code blocks that specify how classes should be initialized. These run at class loading but are parameterless;
- **_Lambdas_**: In Java a lambda is a function which can be created without belonging to any class. It is a shorthand for an anonymous implementation of a Java Functional Interface (and thus it is unseen actually an anonymous local inner class); see [java.util.function](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/function/package-summary.html).

This chapter only deals with the first three of this list: class and instance methods and constructors. 

For completeness sake all function-like Java constructs are demonstrated in the class below.

```java
package snippets.syntax;

import java.util.ArrayList;
import java.util.List;

public class FunctionTypesDemo {
    static List<String> zoo;
    List<String> fruits = new ArrayList<>();

    /*CLASS INITIALIZER*/
    static{
        zoo = new ArrayList<>(List.of("Giraffe", "Mouse", "Scorpion", "Zebra"));
    }

    /*OBJECT INITIALIZER -- RUNS BEFORE ANY CONSTRUCTOR*/
    {
        /*A LAMBDA:    f -> fruits.add(f)*/
        List.of("Kiwi", "Orange", "Mango", "Pear")
                .stream()
                .forEach(f -> fruits.add(f));

        // of course this would have been better,
        // but then there would have been no lambda:
        // fruits.addAll(List.of("Kiwi", "Orange", "Mango", "Pear"));
    }

    /*OVERLOADED CONSTRUCTOR*/
    FunctionTypesDemo() {
        fruits.add("apple");
    }

    /*OVERLOADED CONSTRUCTOR*/
    FunctionTypesDemo(String fruit) {
        fruits.add(fruit);
    }

    /*INSTANCE METHOD*/
    String getFruit(int index) {
        return this.fruits.get(index);
    }

    /*A STATIC (CLASS) METHOD*/
    static void addAnimal(String animal) {
        FunctionTypesDemo.zoo.add(animal);
    }

    /*A SPECIAL CASE CLASS METHOD*/
    public static void main(String[] args) {
        FunctionTypesDemo demo = new FunctionTypesDemo();
        System.out.println(demo.fruits);
        System.out.println("Fruit at index 1 is " + demo.getFruit(1));
        FunctionTypesDemo.addAnimal("Platypus");
        System.out.println(FunctionTypesDemo.zoo);
    }
}
```

Methods define the behavior of an object or a class, modified by its state.
Methods can unseen serve the inner workings of an object, or be available to the outside world: this is something you decide upon, and which is an extremely important design aspect!

## Method signature

Every method has a **_signature_**. In Java, this is a binding contract.
The signature defines what can go in, what comes out, and what the scope (or visibility) of the method is.
Thus, the signature is the description of the method contract to the outside world: everything but its implementation (the method body).

Here, the method signature is shown in red and the method body in blue.

<pre style="color:darkblue;font-weight:bold;font-family:courier;font-size:1.2em;">
<span style="color:darkred;">public int addInts(int x, int y)</span> {
    int result = x + y;
    return result;
}
</pre>

Zooming in on the signature, these are the signature elements:

<pre style="color:darkblue;font-weight:bold;font-family:courier;font-size:1.2em;">
<span style="color:purple;font-style:italic;">access_modifier</span> <span style="color:darkgreen;font-style:italic;">scope</span> <span style="color:darkred;font-style:italic;">return_type</span> methodName(<span style="color:orange;font-style:italic;">parameter(s)</span>) { }
</pre>

- ***access modifier*** can be one of four things: `public`, `protected`, `private` or nothing. Nothing, the default, is something in between private and protected: it has package visibility.
- ***scope*** can be `static` or nothing. Nothing means instance-level and `static` means class level.
- ***return type*** is the data type of the thing that is returned from the method. If it does not return anything, the return type is `void`. Constructors have the instance itself as return type. They have no return type in the signature, nor an allowed `return` statement in the body.
- ***method name*** can be any allowed Java name.
- ***parameters*** can be any number of any type, but need to have a defined type, as everywhere in Java.

### access modifier: public or private?

The access modifier defines the **_visibility_** of your method. There are four levels of visibility, but we'll start out with only two:

- **_`public`_** - accessible anywhere in the **_class path_**: all other code can "see" and access the method. 
- **_`private`_** - accessible only within the same class

We'll come back to access modifiers later (see [access modfiers](/04_oop/access_modifiers)), when discussing class design. This is because access modifiers can be applied not only to methods, but to classes and instance variables as well.

:::{tip}

The general rule is this:  
**_Make a method public when it is part of your class API and make it private when it only serves the inner workings of your class_**.
It is often much harder to change the signature of a public method tan of a private method.
:::

### Scope: static vs instance methods

Some methods live outside object scope and are only available as class methods: these can be recognized by the **_`static`_** keyword. Static methods can be called on a class and on an instance. Instance methods can only be called on an instance, usually because they are dependent on instance variable(s).

You have of course seen a `static` method several times: `public static void main(String[] args){}`

Here are two methods that do the same, but are be used in different contexts.

Consider this class:

```java

public class StaticVsInstance {
    /**
     * static / class method
     */
    static int addIntsStatic(int x, int y) {
        return x + y;
    }

    /**
     * instance method
     */
    int addIntsInstance(int x, int y) {
        return x + y;
    }
}
```

The only difference lies in the `static` keyword. These are the different ways these methods can (not) be called:

```java
//only static can be called on the class
StaticVsInstance.addIntsStatic(2, 5);
//WILL NOT COMPILE: can not be called on a class
//StaticVsInstance.addIntsInstance(3, 4);

StaticVsInstance statInst = new StaticVsInstance();
//BOTH can be called on an instance
statInst.addIntsStatic(5, 6);
statInst.addIntsInstance(3, 7);
```

### Return type

The return type defines what a method can (and must) return. This can be any Java type; primitive or reference type. There is a special case when the method does not return anything: this is the **_`void`_** return type:

```java
public void printWelcome (String name) {
    System.out.println("Hello, " + name + ", we hope you enjoy our app.");
}
```

**Returning multiple values?**  
Forget it! There is NO way you can return multiple values from a method.
Unless...you put these values in a composite form. The following shows a legal but not very nice solution.
Note the use of the **_varargs_** method parameter that enables you to let a method be called with a variable number of arguments of the same type.

```java
/**
* Calculates the sum and average of a series of numbers and
* returns this as an array with the sum at index 0 and the average at index 1
* @param numbers
* @return statistics
*/
static double[] doStatistics(int... numbers){ //VARARGS!
    int sum = 0;
    for (int n : numbers) sum += n;
    double average = (double)sum/numbers.length;
    double[] result = {(double)sum, average};
    return result;
}

//usage
System.out.println("doStatistics(1, 2, 3) = " + Arrays.toString(doStatistics(1, 2, 3)));
System.out.println("doStatistics(1, 2, 3, 4) = " + Arrays.toString(doStatistics(1, 2, 3, 4)));
//outputs
//doStatistics(1, 2, 3) = [6.0, 2.0]
//doStatistics(1, 2, 3, 4) = [10.0, 2.5]
```
Why is this not nice? You coerce an int into a double where this is not appropriate. 
Besides this, you need extensive comments (documentation) to explain what is going on.

**THINK OBJECTS!**

```java
//a simple data class
static class Statistics{
    int sum;
    double average;

    //makes printing objects a breeze!
    @Override
    public String toString() {
        return "Statistics{sum=" + sum + ", average=" + average + '}';
    }
}

/**
 * the refactored method
 * Calculates the sum and average of a series of numbers and
 * returns this as an stats object
 * @param numbers
 * @return statistics
 */
static Statistics doStatistics(int... numbers){
    int sum = 0;
    for (int n : numbers) sum += n;
    double average = (double)sum/numbers.length;
    Statistics statistics = new Statistics();
    statistics.sum = sum;
    statistics.average = average;
    return statistics;
}

//usage
System.out.println("doStatistics(1,2,5,6) = " + doStatisticsTheGoodWay(1, 2, 5, 6));
//outputs
//doStatistics(1,2,5,6) = Statistics{sum=14, average=3.5}
```

The `toString()` method is similar to the Python `__str__()` method. It will be discussed in detail later.


### Method name

The name can be any compilable Java identifier, but as stated earlier it is good practice to have method names 

- be a noun, in camelcase starting with a lowercase letter
- be as descriptive as possible


### Parameters (versus arguments)

The method parameters are the number and type of variables that go in. Again, these can be of any valid Java 
type.

Java is pass-by-value. This means: pass-by-copy. Passing a reference means: pass a copy of the reference.
This reference copy will **_point to the same object_** as the original reference. This is sometimes confusing for 
beginners:

```java
class MyDataClass {
    int x = 42;
}

void workWithClass(MyDataClass myDataClass) {
    myDataClass.x++;
}

//usage 
MyDataClass myDataClass = new MyDataClass();
System.out.println("myDataClass.x = " + myDataClass.x);
workWithClass(myDataClass);
System.out.println("myDataClass.x = " + myDataClass.x);

//outputs
//myDataClass.x = 42
//myDataClass.x = 43
```


:::{note}
**On the difference between parameters and arguments**  
A parameter is the variable which is part of the method's signature (method declaration)
An argument is an expression used when calling the method.
:::

### Cheating the contract

This is not possible: the compiler won't let you get away with it. Here are some 
examples showing what is legal and what isn't.

```java
public class MyMathTools{

    /*test the math function*/
    public static void main(String[] args){
        MyMathTools mt = new MyMathTools();
        //WON'T COMPILE: argument too little
        mt.addInts(3);

        //WON'T COMPILE: argument too many
        mt.addInts(1, 2, 3);

        //WON'T COMPILE: argument of wrong type
        mt.addInts(2, "foo");

        //WON'T COMPILE: double to int is loss-of-precision (unsafe)
        mt.addInts(1.66, 2.33);

        //WILL COMPILE; explicit cast (will return 1 + 2)
        mt.addInts((int)1.66, (int)2.33);

        //WILL COMPILE; int can be implicitly converted to double
        addDoubles(2, 4)
    }
    public int addInts(int x, int y) {
        return (x + y);
    }

    public double addDoubles(double x, double y) {
        return (x + y);
    }

    public int addIntsWrong(int x, int y){
        x + y;
        //WON'T COMPILE: should return an int
        return;
    }
}
```

## Interacting with state

The essence of Object-oriented programming is that methods often interact with an object's state: they display behavior that is dependent on the value of the objects' instance variables. Here is a simple class that uses its state to modify method behavior.

```java
class PowerUpper {
    int power = 2;

    int powerUp(int x) {
        return (int)Math.pow(x, power);
    }
}

//calling code
PowerUpper powerUpper = new PowerUpper();
System.out.println("4 ^ 2 = " + powerUpper.powerUp(4));
powerUpper.power = 3;
System.out.println("4 ^ 3 = " + powerUpper.powerUp(4));
```

outputs:

<pre class="console_out">
4 ^ 2 = 16
4 ^ 3 = 64
</pre>

## Method overloading

Java does not know about **_default parameter values_**. 
Fortunately, it can be implemented using a more flexible mechanism called **_method overloading_**. 
Method overloading is the implementation of multiple methods with the same name and return type, but with different (numbers of) arguments. 
It is a bit of a hassle, but more versatile at the same time since this not only supports default values but also extra pieces of algorithm code on top of default behavior.

Here is an adjusted version of the PowerUpper class, implementing different overloading techniques.

```java
static class PowerUpper {
    /**
     * returns x ^ 2
     */
    int powerUp(int x) {
        System.out.println("method A");
        return powerUp(x, 2);
    }

    /**
     * returns x ^ power
     */
    int powerUp(int x, int power) {
        System.out.println("method B");

        //THIS WILL CAUSE A STACK OVERFLOW
        //return powerUp(x, power);

        //OK
        return powerUp((double)x, (double)power);
    }

    /**
     * returns x ^ power from two doubles, but converts the result to int
     */
    int powerUp(double x, double power) {
        System.out.println("method C");
        return (int)Math.pow(x, power);
    }

    //this is not an overload! a different return type
    double powerUp(int x, double power) {
        System.out.println("method D");
        return Math.pow(x, power);
    }

    //WON'T COMPILE! is not an overload, but a re-definition of "int powerUp(double x, double power)"
    //double powerUp(double x, double power) {
    //    return Math.pow(x, power);
    //}
}

//usage
PowerUpper powerUpper = new PowerUpper();
System.out.println("4 ^ 2 = " + powerUpper.powerUp(4));
System.out.println("4 ^ 3 = " + powerUpper.powerUp(4, 3));
System.out.println("4.0 ^ 3.0 = " + powerUpper.powerUp(4.0, 3.0));
System.out.println("4.7 ^ 3.9 = " + powerUpper.powerUp(4.7, 3.9));
```

Outputs:

<pre class="console_out">
method A
method B
method C
4 ^ 2 = 16
method B
method C
4 ^ 3 = 64
method C
4.0 ^ 3.0 = 64
method C
4.7 ^ 3.9 = 418
</pre>

## Constructors

Consider the case of Pizzas. Pizzas can have a normal or thick base, and one to many ingredients.
How would you construct such an object? This data design is quite straightforward:

```java
public class Pizza {
    private String base;
    private List<String> toppings = new ArrayList<>();
    
    @Override
    public String toString() {
        return "Pizza{" +
                "base='" + base + '\'' +
                ", toppings=" + toppings +
                '}';
    }
}
```

In order for a pizza to exist it really needs a base: see the test below:

```java
    @Test
    void construct() {
        Pizza emptyNoBase = new Pizza();
        System.out.println("emptyNoBase = " + emptyNoBase);
    }
```

<pre class="console_out">
emptyNoBase = Pizza{base='null', toppings=[]}
</pre>

Therefore, adding a constructor seems like a good idea:


```java
public class Pizza {
    private String base;
    private List<String> toppings = new ArrayList<>();


    public Pizza(String base) {
        this.base = base;
    }

    //toString() omitted
}
```

Now you must provide a base in order to construct a pizza:

```java
    @Test
    void construct() {
        Pizza baseOnly = new Pizza("thin");
        System.out.println("baseOnly = " + baseOnly);
    }
```

<pre class="console_out">
baseOnly = Pizza{base='thin', toppings=[]}
</pre>

### Telescoping constructors 

But wait! Most (sane) people will want their pizza base to be thin. Making it mandatory to always provide a constructor argument is silly. This is where ***constructor overloading*** comes in. We provide two alternative constructors where one calls the other with default argument(s):

```java
public class Pizza {
    private String base;
    private List<String> toppings = new ArrayList<>();

    public Pizza() {
        this("thin"); //CALLS THE OTHER CONSTRUCTOR!
    }

    public Pizza(String base) {
        this.base = base;
    }

    //toString() omitted
}
```

Providing a base is now optional, with a (very!) sensible default:

```java
    @Test
    void construct() {
        Pizza baseOnly = new Pizza();
        System.out.println("baseOnly default = " + baseOnly);

        Pizza baseOnlyThick = new Pizza("thick");
        System.out.println("baseOnly thick = " + baseOnlyThick);
    }
```

<pre class="console_out">
baseOnly default = Pizza{base='thin', toppings=[]}
baseOnly thick = Pizza{base='thick', toppings=[]}
</pre>

But now we have a pizza without toppings. That is no good. We could use one of these methods to add ingredients (toppings) to the pizza: 

```java
    /**
     * Adds a single ingredient.
     * @param ingredient
     */
    void add(String ingredient) {
        this.toppings.add(ingredient);
    }

    /**
     * adds multiple ingredients.
     * @param ingredients
     */
    void addIngredients(List<String> ingredients) {
        this.toppings.addAll(ingredients);
    }
```

But this is a little lame. Now we need to keep constructing the pizza after it has been constructed!
The solution is the use of ***varargs*** arguments in the constructor! Here it is:

```java
    public Pizza(String base, String ... ingredients) {
        this.base = base;
        System.out.println("ingredients = " + ingredients);
        for (String topping : ingredients) {
            toppings.add(topping);
        }
    }
```

:::{admonition} Varargs
Varargs is the use of an optional number of arguments of the same type.
They are defined using the syntax `Type... nameOfCollection`.
They can be applied to any type of function: constructors and (static) methods.
Internally they are translated into an `array`.
:::

The alert reader will have noticed that the default `base` value has disappeared. That is because if we also want to overload this variant, the compiler does not know to distinguish between constructors anymore. A solution would be to extract the base into an enum constant:

```java
public enum PizzaBase {
    THIN,
    THICK;
}
```

So now we have a Pizza class with four constructors which makes for a very versatile solution.

```java
package snippets.syntax;

import java.util.ArrayList;
import java.util.List;

public class Pizza {
    private PizzaBase base;
    private List<String> toppings = new ArrayList<>();

    public Pizza() {
        this(PizzaBase.THIN);
    }

    public Pizza(PizzaBase base) {
        this.base = base;
    }

    public Pizza(String... ingredients) {
        this(PizzaBase.THIN, ingredients);
    }

    public Pizza(PizzaBase base, String... ingredients) {
        this.base = base;
        for (String topping : ingredients) {
            toppings.add(topping);
        }
    }

    //toString() omitted
}

```


In principle this concludes the discussion of constructors. However, since this pattern of telescoping constructors is sometimes frowned upon, I present two alternatives below.

### Factory methods

Factory methods allow for the definition of an API to create complex objects without the client having to intervene with the details of this creation.

Here are two factory methods for creating pizzas:

```java
    //STATIC!
    public static Pizza createThinPizza(String... ingredients) {
        return createPizza(PizzaBase.THIN, ingredients);
    }

    public static Pizza createThickPizza(String... ingredients) {
        return createPizza(PizzaBase.THICK, ingredients);
    }

    //PRIVATE
    private static Pizza createPizza(PizzaBase base, String... ingredients) {
        Pizza p = new Pizza(base);
        Arrays.stream(ingredients).forEach(i -> p.addIngredient(i));
        return p;
    }
    
    //PRIVATE
    private void addIngredient(String addIngredient) {
        this.toppings.add(ingredient);
    }
```

### Builder pattern

For cases like this, the design pattern called the Builder Pattern is an excellent use case. This is of course out of scope for this course, but I like it so much I wanted to show it here as well.

```java
package nl.bioinf;

import java.util.ArrayList;
import java.util.List;
import java.util.Objects;

public class Pizza {
    private List<String> ingredients;
    private PizzaBase base;

    //A PRIVATE CONSTRUCTOR!
    private Pizza(Builder builder) {
        this.base = builder.base;
        this.ingredients = new ArrayList<>();
        for (String ingredient : builder.ingredients) {
            this.ingredients.add(ingredient);
        }
    }

    //INNER CLASS
    public static class Builder {
        private final PizzaBase base;
        private List<String> ingredients = new ArrayList<>();

        private Builder(PizzaBase base) {
            this.base = base;
        }

        public Builder ingredient(String ingredient) {
            this.ingredients.add(ingredient);
            return this;
        }

        public Pizza build() {
            if (this.ingredients.isEmpty()) {
                throw new IllegalArgumentException("there should be at least one ingredient");
            }
            return new Pizza(this);
        }
    }

    //FACTORY-LIKE METHOD
    public static Builder builder(PizzaBase base) {
        try {
            Objects.requireNonNull(base);
        } catch (NullPointerException ex) {
            throw new IllegalArgumentException("base should be provided");
        }

        return new Builder(base);
    }


    public static void main(String[] args) {
        Pizza pizza = Pizza
                .builder(null)
                .ingredient("cheese")
                .ingredient("onions")
                .ingredient("peppers")
                .ingredient("gorgonzola")
                .build();
        System.out.println("pizza = " + pizza);
    }

}
```

