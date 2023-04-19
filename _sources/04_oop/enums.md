# Enums: constants on steroids

## Constants are everywhere  

Constants are everywhere: the number pi, the number e, the URL to a web service 
you use in your app. Here are two if these. 

```java
public static final double PI = 3.1416;
public static final String API_URL = "http://www.example.com";
```
Constants are declared as `public static final` variables (sometimes `private`).  

Of course, an alert observer might state that pi is already defined in class 
Math as PI, and in much more detail: 3.141592653589793. 
And also that URLs should not be defined as hard-coded strings, but in a properties file. 

But this post is not about simple constants. This post is about related sets of constants 
that all refer to a given property with your application. This is where enums come 
in. 
Java enums are so much more than simple constants. 
In fact, enums are regular Java classes, but very special in the fact that only the instances defined in the enum class file will ever see the day of light!  


## Working with enums

:::{admonition} Definition
A Java enum is a class declared with the keyword `enum`. It defines all possible discrete values for tha class.
Here is an example:

```java
public enum Role {
    GUEST,
    USER, 
    ADMIN;
}
```
In this example the enum of type `Role` has been defined, with three possible values (instances) of the enum.
:::

Other examples of typical use cases for enums are the roles of visitors in your web application: `GUEST`, `LOGGED_IN_USER`, `ADMIN`, or pizza sizes in an online ordering form: `SMALL`, `MEDIUM`, `LARGE`, 
`LUNATIC`. 
An example in the bioinformatics domain is the letters of the DNA alphabet: `G`, `A`, `T` and `C`. 
Let’s look at this example in more detail.  

```java
public enum Nucleotide {
    A,
    C, 
    G,
    T;
}
```

Here, I have defined an enum with four values (possible instances): G, A, T, and C. 
As a convention, enum value names should be all-caps.  
You can already do a variety of things with this implementation . 
See the next snippet for examples of usage. 

```java
package snippets.enums;

public final class EnumDemo {
    public static void main(final String[] args) {
        EnumDemo enumDemo = new EnumDemo();
        enumDemo.start();
    }

    private void start() {
        // declare a specific instance of the type
        Nucleotide nucA = Nucleotide.A;

        Nucleotide otherNucA = Nucleotide.A;

        // both variables point to the same object!
        System.out.println("nucA and otherNucA are the same: " + (nucA == otherNucA));

        // print it to show the constant value (this can be overridden -- see below)
        System.out.println("nucA = " + nucA);

        // get its ordinal (the rank in the declaration list)
        System.out.println("nucA.ordinal() = " + nucA.ordinal());
        System.out.println("Nucleotide.G.ordinal() = " + Nucleotide.G.ordinal());

        // get all possible values
        System.out.println("Nucleotide.values() = " + Arrays.toString(Nucleotide.values()));

        // convert from string
        Nucleotide nuc = Nucleotide.valueOf("C");
        System.out.println("created from String with valueOf() = " + nuc);

        // switch on it
        switchOnNucleotide(nuc);
    }

    private void switchOnNucleotide(final Nucleotide nuc) {
        switch (nuc) {
            case A:
                System.out.println("It is an A");
                break;
            case C:
                System.out.println("It is a C");
                break;
            case G:
                System.out.println("It is a G");
                break;
            case T:
                System.out.println("It is a T");
                break;
        }
    }
}
```

When printed, an enum will simply be displayed as the name of its value:  

<pre class="console_out">
nucA and otherNucA are the same: true
nucA = A
nucA.ordinal() = 0
Nucleotide.G.ordinal() = 2
Nucleotide.values() = [A, C, G, T]
created from String with valueOf() = C
It is a C
</pre>

The _factory_ function `valueOf()` is an often overlooked utility, converting from string to enum.

```java
String letter = "G";
Nucleotide nuc = Nucleotide.valueOf(letter);
System.out.println("nuc = " + nuc);
```

This will output  

<pre class="console_out">
nuc = G
</pre>

This `valueOf()` call returned the `Nucleotide.G` constant. Note that no new instance will be created!
Only one instance will of each possible instance will exist during your app life cycle.

When you use a non-existing value you get an `IllegalArgumentException`:

```java
nuc = Nucleotide.valueOf("P");
```

This will output something like this

<pre class="console_out">
Exception in thread "main" java.lang.IllegalArgumentException: No enum constant enums.Nucleotide.P
	at java.lang.Enum.valueOf(Enum.java:238)
	at enums.Nucleotide.valueOf(Nucleotide.java:11)
	at enums.EnumDemo.start(EnumDemo.java:41)
	at enums.EnumDemo.main(EnumDemo.java:19)
</pre>



Also, the fact that you can do flow control on the basis of the ordinal value can be useful but hardly ever used. 
See below for an example:

```java
public enum PizzaSize {
    SMALL,
    MEDIUM,
    LARGE,
    LUNATIC;
}

PizzaSize pz = PizzaSize.LUNATIC;

if (pz.ordinal() >= PizzaSize.LARGE.ordinal()) {
    System.out.println("Are you sure? This size is for really hungry people.");
}
```

## Customization and extension

### A custom constructor

If you want something else printed by default besides the name of the constant, 
you can provide this value through a constructor:  

```java
package enums;

public enum Nucleotide {
    A("Adenine"),
    C("Guanine"),
    G("Cytosine"),
    T("Thymine");

    private String name;

    private Nucleotide(final String fullName) {
        this.name = fullName;
    }

    @Override
    public String toString() {
        return name;
    }
}
```

Now, when you print an enum you will get its string value given via the constructor 
argument. Yes, this `A("Adenine")` constructs a Nucleotide enum constant of type `Nucleotide.A` with "name" property "Adenine". 

This is already pretty neat, yes? And we're only just getting started!  

## Added functionality

Let's add some functionality. 
First something simple: provide access to the nucleotides’ molecular weight. 
I’ll use a HashMap to store these values and define a simple method that serves them.

```java
package snippets.enums;

import java.util.Map;

public enum Nucleotide {
    A("Adenine"),
    C("Guanine"),
    G("Cytosine"),
    T("Thymine");

    private static final Map<Nucleotide, Double> MOLECULAR_WEIGHTS =
            Map.of(A, 313.2, C, 304.2, G, 329.2, T, 304.2);
    private String name;

    Nucleotide(final String fullName) {
        this.name = fullName;
    }

    @Override
    public String toString() {
        return name;
    }

    /**
     * returns the molecular weight of the nucleotide, in Daltons.
     * @return molecularWeight
     */
    public double getMolecularWeight() {
        return MOLECULAR_WEIGHTS.get(this);
    }
}
```

See how you can use the reference to `this` to fetch the molecular weight? 
Here are two lines using this functionality:

```java
System.out.println("nuc A weight = " + nucA.getMolecularWeight());
System.out.println("nuc C weight = " + Nucleotide.C.getMolecularWeight());
```

Note that this property could have been implemented in a constructor setting as well, just like the nucleotide name.

But how about the complements of these values (you know, the letter on the other strand of DNA in the double helix, the one it pairs with)? 
“A” has as complement “T”, “G” has “C” etc. 
I want to be able to get a complement for all nucleotides, given its original value. 
The solution is the same: use a constructor value or a Map-like solution as with the nucleotide name.


## Advanced stuff

Still this is not the whole story. The _piece de resistance_ has yet to come. As you may know, there is another nucleotide that does not occur in DNA but solely in RNA: U (Uracil). DNA has A, C, G and T while RNA has A, C, G, and U. 
Thus, U replaces T in RNA. I want to add that one to my Nucleotide enum, and also provide a method that will tell me whether a nucleotide only occurs in RNA. I could simply provide a map for that, but this is not as efficient (or cool) as another solution. In fact, ONLY the U is exclusive for RNA. Therefore, I could provide some default functionality and override that behavior for Uracil. 


```java
    //rest of enum omitted
    A("Adenine"),
    C("Guanine"),
    G("Cytosine"),
    T("Thymine"),
    U("Uracil") {
        /*yes! a CONSTANT-SPECIFIC CLASS BODY! An override for single nucleotide*/
        @Override
        public boolean isExclusiveRNA() {
            return true;
        }
    };

    //rest of enum omitted

    /**
     * returns whether this nucleotide is only found in RNA.
     *
     * @return isExclusiveRNA
     */
    public boolean isExclusiveRNA() {
        //the default value
        return false;
    }

    //rest of enum omitted
```

This is an example of a **_constant-specific class body_**, something only certified 
Java programmers seem to know about. And now you do, too. Its purpose is to provide an override for some generic functionality that applies to most of the other constants of the enum.

Below follows the complete code of the enum. Of course, there is at least one major flaw in the model with respect to adherence to molecular biology – there awaits eternal fame for you if you find it.

```java
package snippets.enums;

import java.util.Map;

public enum Nucleotide {
    A("Adenine"),
    C("Guanine"),
    G("Cytosine"),
    T("Thymine"),
    U("Uracil") {
        /*yes! a CONSTANT-SPECIFIC CLASS BODY! An override for single nucleotide*/
        @Override
        public boolean isExclusiveRNA() {
            return true;
        }
    };;

    private static final Map<Nucleotide, Double> MOLECULAR_WEIGHTS =
            Map.of(A, 313.2, C, 304.2, G, 329.2, T, 304.2);
    private String name;

    Nucleotide(final String fullName) {
        this.name = fullName;
    }

    @Override
    public String toString() {
        return name;
    }

    /**
     * returns the molecular weight of the nucleotide, in Daltons.
     * @return molecularWeight
     */
    public double getMolecularWeight() {
        return MOLECULAR_WEIGHTS.get(this);
    }

    /**
     * returns whether this nucleotide is only found in RNA.
     * @return isExclusiveRNA
     */
    public boolean isExclusiveRNA() {
        return false;
    }
}
```

That's it. You have now seen almost everything there is to know about enums in Java.
