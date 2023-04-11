# Naming and commenting stuff

Anyone who knows me (as a teacher) knows that (a) I really like Java and (b) I really really hate it when stuff gets named in the wrong way.

## Naming

Experienced programmers can read code as if it is a nice book. When doing this, we rely on 
some conventions, just like in real language. For instance, in regular human text, names start with a 
capital, and sentences have a certain grammatical structure that makes it possible to read quickly without too much confusion. Whenever texts do not adhere to these rules or conventions, reading becomes very difficult.  
Programming languages are no different, and naming and layout-conventions help human readers to makes sense of it really quickly. 

Java has a concise set of naming rules that you should adopt from your first endeavors. Here are the most important ones.

For all names -classes, methods, variables- the single most important rule is that they should **_describe what they model in a precise and concise way_**.  For instance, a valid counter for te number of logins in a system could be `c`, `count`, `counter` or `loginCounter`, but only the last is really informative to anyone reading te code.  

The second general rule is that Java identifiers are always in **_CamelCase_**, with the exception of certain types of constants. Also, never start names with a number and do not use special characters! These are illegal and will not compile: `1forTheMoney`, `k#ller`.

### Classes

Class names should be a noun, in camel case, with the first letter capitalized. Here are a few good and bad examples for names of a class reading GFF data from a file:

```java
class Read {} //bad - no noun and vague
class Reader {} //bad, vague
class DataReader {} //bad, still vague
class dataReader {} // bad - starts with lowercase
class Data_Reader {} // bad - underscore is not camelcase
class GffRead {} //bad, not a noun
class GffReader {} //good: a noun and specific
```

### Methods

Method names should be a verb, in camel case. Here some examples for a method name calculating the Euclidean distance between two Point objects:

```java
double d(Point otherPoint) {} //bad - no verb and vague
double distance(Point otherPoint) {} // bad - no verb and still slightly vague
double euclidean_distance(Point otherPoint) {} //bad, no camel case
double EuclideanDistance(Point otherPoint) {} // bad - starts with uppercase character
double euclideanDistance(Point otherPoint) {} // almost good, but no noun
double getEuclideanDistance(Point otherPoint) {} //finally OK
```

### Variables

Variable names should be nouns as well, but starting in lowercase:

```java
double r; //plain bad
double ratio; //bad - what ratio?
double Ratio; //bad - starts with uppercase
double screen_ratio; //bad - underscore
double screenAspectRatio; //good: noun and descriptive
```

## Commenting

There are three ways for commenting:  

- Javadoc style: `/**\<LINE(S)\>*/` these are the most important because they end up in your documentation!
- multiline comment `/*\<LINE(S)\>*/`
- single line comment `//\<SINGLE LINE\>`

Here are some examples:


```java
/**
 * This is a Javadoc comment for the class: it describes what the class models and should be used for.
 * Also, version and author information can be put here.
 * @author Michiel
 * @version 0.1
 * */
public class Duck {
    /**
     * This is a Javadoc comment for a method.
     * It should include tags for method arguments, exceptions and return types.
     * @param loudness
     */
    public void quack(int loudness) {
        //single-line logic comment
        System.out.println("Quacking at level " + loudness);
        /*I can write a multiline block of
        * comment like this*/
    }
}
```

Here is a screenshot of part of the the corresponding Javadoc html:

![Javadoc](figures/javadoc_duck_1.png)

I strongly encourage you to try this out for yourself: create a class, put some javadoc 
comments in it and run "Tasks -> documentation -> javadoc" from the Gradle tool window.
The javadoc folder will be located in the `build/docs/` folder.
The javadoc tool will issue warnings but proceed for some inconsistencies, but errors and abort for others. The warnings and errors are pretty clear, fortunately.
