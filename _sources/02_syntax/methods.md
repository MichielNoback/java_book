# Methods

This post constitutes the link between basic syntax and design. Variables, operators and flow control 
form the basis of your algorithms. These algorithms have to be organized and combined into bigger 
systems, and building systems is done using classes and their methods.

Methods are pieces of code that are used as a "black box": you don't need to know how they work, as long as you know (and trust) what goes in and what comes out.

In Java, there exists quite a variety of method-like structures:

- **_instance methods_**
- **_class methods_**
- **_constructors_**
- **_class initializers_**
- **_object initializers_**

This post only deals with the first two: class and instance methods.

Methods define the behavior of an object or a class, modified by its state.
Methods can unseen serve the inner workings of an object, or be available to the outside world: this is something you decide upon, and which is an extremely important design aspect!

## Method signature

Every method has a **_signature_**. In Java, this is a binding contract.
The signature defines what can go in, what comes out, and what the method scope or visibility is.
So, the signature is the description of the method contract to the outside world: everything but its implementation (the method body).

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

### access modifier: public or private?

The access modifier defines the **_visibility_** of your method. There are four levels of visibility, but we'll start out with only two:

- **_`public`_** - accessible anywhere in the **_class path_**: all other code can "see" and access the method. 
- **_`private`_** - accessible only within the same class

We'll come back to access modifiers later (see [access modfiers](/04_oop/access_modifiers)), when discussing class design. This is because access modifiers can be applied not only to methods, but to classes and instance variables as well.

The general rule is this:  
**_Make a method public when it is part of your class API and make it private when it only serves the inner workings of your class_**

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



**On the difference between parameters and arguments**  
A parameter is the variable which is part of the method's signature (method declaration)
An argument is an expression used when calling the method.

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

Java does not know about **_default parameter values_**. Fortunately, it can be implemented using a more flexible 
mechanism called **_method overloading_**. Method overloading is the implementation of multiple methods with the 
same name and return type, but with different (numbers of) arguments. It is a bit of a hassle, but more versatile at the same time, since this not only supports default values but also extra pieces of algorithm code on top of default behavior.

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

    //WON'T COMPILE! is not an overload, but a redefenition of "int powerUp(double x, double power)"
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
