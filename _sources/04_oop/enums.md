# Enums: constants on steroids

## Constants are everywhere  

Constants are everywhere: the number pi, the number e, the URL to a web service 
you use in your app. These are really simple constants that do not need anything 
special besides being defined as a  

```java
public static final String PI = 3.1416;
public static final String API_URL = "http://www.example.com";
```

Of course, any alert observer might state that pi is already defined in class 
Math as PI, and in much more detail: 3.141592653589793. And of course, URLs 
should not be defined as hard-coded strings, but in a neat little properties 
file. 

But this post is not about simple constants. This post is about sets of constants 
that all refer to a given property with your application. This is where enums come 
in. I love them and hope that, after reading this post, you will too. I will 
show you that Java enums are so much more than simple constants. In fact, enums 
are regular Java classes, but only special in the fact that only the instances 
defined in the enum class file will ever see the day of light!  

## A basic enum

For instance, the roles of visitors in your web application: `GUEST`, `LOGGED_IN_USER`, 
`ADMIN`, or pizza sizes in your online ordering form: `SMALL`, `MEDIUM`, `LARGE`, 
`LUNATIC`. Or the four letters of the DNA alphabet: `G`, `A`, `T` and `C`. Let’s look at an example using the nucleotides.  

```java
public enum Nucleotide {
    A,
    C, 
    G,
    T;
}
```

Here, I have defined an enum with four values: G, A, T, and C. As a convention, enum value names must always be all-caps.  
You can do some nice things with it already.  

```java
package enums;

public final class EnumDemo {
    public static void main(final String[] args) {
        EnumDemo enumDemo = new EnumDemo();
        enumDemo.start();
    }

    private void start() {
        Nucleotide nucA = Nucleotide.A;
        //print it
        System.out.println("nucA = " + nucA);
        //test its nature
        if (nucA == Nucleotide.A) {
            System.out.println("We've got an A!");
        }
        //switch on it!
        switchOnNucleotide(Nucleotide.G);
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
nucA = A
We've got an A!
It is an G
</pre>

## A custom constructor

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
The reverse is also a much-used aspect: you have some string value and want to create an enum from it.  
This is how it's done:  

```java
String letter = "G";
Nucleotide nuc = Nucleotide.valueOf(letter);
System.out.println("nuc = " + nuc);
```

This will output  

<pre class="console_out">
nuc = Guanine
</pre>

This `valueOf()` call returned the `Nucleotide.G` constant. Note it will not be created, since only one instance will exist during your app life cycle.

When you use a non-existing value you get an `IllegalArgumentException`:

```java
letter = "P";
nuc = Nucleotide.valueOf(letter);
```

This will output  

<pre class="console_out">
Exception in thread "main" java.lang.IllegalArgumentException: No enum constant enums.Nucleotide.P
	at java.lang.Enum.valueOf(Enum.java:238)
	at enums.Nucleotide.valueOf(Nucleotide.java:11)
	at enums.EnumDemo.start(EnumDemo.java:41)
	at enums.EnumDemo.main(EnumDemo.java:19)
</pre>

This is already pretty neat, yes? And we're only just getting started!  

## Added functionality

Let's add some functionality. First something simple: provide access to the 
nucleotides’ molecular weight. I’ll use a HashMap to store these values and 
define a simple method that serves them.

```java
    //rest of class omitted

    private static final HashMap<Nucleotide, Double> MOLECULAR_WEIGHTS;
    /**
     * static initializer to populate the map.
     */
    static {
        MOLECULAR_WEIGHTS = new HashMap<>();
        MOLECULAR_WEIGHTS.put(A, 313.2);
        MOLECULAR_WEIGHTS.put(C, 304.2);
        MOLECULAR_WEIGHTS.put(G, 329.2);
        MOLECULAR_WEIGHTS.put(T, 304.2);
    }
    /**
     * returns the molecular weight of this nucleotide in single-stranded DNA, in Daltons.
     * @return molecularWeight
     */
    public double getMolecularWeight() {
        return MOLECULAR_WEIGHTS.get(this);
    }
    //rest of class omitted
```

See how you can use the reference to `this` to fetch the molecular weight? 
Here are two lines using this functionality:

```java
System.out.println("nuc A weight = " + nucA.getMolecularWeight());
System.out.println("nuc C weight = " + Nucleotide.C.getMolecularWeight());
```

But how about the complements of these values (you know, the letter on the other 
strand of DNA in the double helix, the one it pairs with)? “A” has as complement 
“T”, “G” has “C” etc. I want to be able to get a complement for all nucleotides, 
given its original value. Exactly the same as the molecular weights, but the 
value is another nucleotide.  

## Advanced stuff

Still this is not the whole story. The _piece de resistance_ has yet to come. As 
you may know, there is another nucleotide that does not occur in DNA but solely 
in RNA: U (Uracil). DNA has A, C, G and T while RNA has A, C, G, and U. Thus, U 
replaces T in RNA. I want to add that one to my Nucleotide enum, and also provide 
a method that will tell me whether a nucleotide only occurs in RNA. I could 
simply provide a map for that, but this is not as efficient (or cool) as 
another solution. In fact, ONLY the U is exclusive for RNA. Therefore, I could 
provide some default functionality and override that behavior for Uracil. 


```java
    //rest of enum omitted
    A("Adenine"),
    C("Guanine"),
    G("Cytosine"),
    T("Thymine"),
    U("Uracil") {
        /*yes! a CONSTANT-SPECIFIC CLASS BODY!*/
        @Override
        public boolean isExclusiveRNA() {
            //override for single RNA nucleotide
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
Java programmers seem to know about. And now you do, too. Its purpose is to 
provide an override for some generic functionality that applies to most of the 
other constants of the enum.

Below follows the complete code of the enum. Of course, there is at least one 
major flaw in the model with respect to adherence to molecular biology – there 
awaits eternal fame for you if you find it.

```java
package enums;

import java.util.HashMap;

public enum Nucleotide {
    A("Adenine"),
    C("Guanine"),
    G("Cytosine"),
    T("Thymine"),
    U("Uracil") {
        /*yes! a CONSTANT-SPECIFIC CLASS BODY!*/
        @Override
        public boolean isExclusiveRNA() {
            return true;
        }
    };
    private static final HashMap<Nucleotide, Double> MOLECULAR_WEIGHTS;
    private static final HashMap<Nucleotide, Nucleotide> REVERSE_COMPLEMENTS;

    static {
        //MOL WEIGHTS
        MOLECULAR_WEIGHTS = new HashMap<>();
        MOLECULAR_WEIGHTS.put(A, 331.2);
        MOLECULAR_WEIGHTS.put(C, 307.2);
        MOLECULAR_WEIGHTS.put(G, 347.2);
        MOLECULAR_WEIGHTS.put(T, 322.2);
        MOLECULAR_WEIGHTS.put(T, 324.2);
        //REV COMPLS
        REVERSE_COMPLEMENTS = new HashMap<>();
        REVERSE_COMPLEMENTS.put(A, T);
        REVERSE_COMPLEMENTS.put(C, G);
        REVERSE_COMPLEMENTS.put(G, C);
        REVERSE_COMPLEMENTS.put(T, A);
        REVERSE_COMPLEMENTS.put(U, A);
    }

    public double getMolecularWeight() {
        return MOLECULAR_WEIGHTS.get(this);
    }

    public Nucleotide complement() {
        return REVERSE_COMPLEMENTS.get(this);
    }

    public boolean isExclusiveRNA() {
        return false;
    }

    private final String name;

    private Nucleotide(final String fullName) {
        this.name = fullName;
    }

    @Override
    public String toString() {
        return name;
    }
}
```


