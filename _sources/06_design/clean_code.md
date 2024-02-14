# General coding principles

## General

### Catch null as soon as possible

- Null values are the most common cause of bugs
- Catch them as soon as possible, preferably in the constructor
- Use `Objects.requireNonNull()`

```java
public Car(Wheel wh) {
    this.wheel = Objects.requireNonNull(wh, "Wheel should be non-null!");
}
```

### Error handling

- Use Exceptions rather than Error codes
- Use unchecked exceptions rather than checked exceptions
- Don’t return null but rather the SPECIAL CASE object or Optional
- Think well where you want to catch your errors


### Comments

In general, comment your API methods, as concise as possible.
These are useless comments (even the one for the public API):

```java
/**
 * The employee that is being dealt with
 */
private Employee employee;

/**
 * This method returns whether the employee is
 * eligible for full pension.
 * @return eligible
 */
public boolean isEligibleForFullPension() {
    return eligibleForFullPension;
}
```

For the rest: make your code self-explanatory and only provide comments when they are really necessary.. 

```java
if (employee.getWorkingYears() >= MINIMUM_COMPLETE_PENSION_PERIOD
        && employee.getFteSize() >= 0.95) { }

//OR this?

if (employee.isEligibleForFullPension()) { }
```

See below for more examples of this kind.

## Names and variables

This has been mentioned before, but for completeness' sake repeated here.

:::{important}
Java is **_CamelCase_** not **_snake_case_**.  
If a name needs a comment, it is not a good name.  
Names should reveal intention of its use and nature. They specify what is measured and the unit in which this happens.
:::

- **variables** are nouns, written lowercase-first. An exception is the `static final` constant, which is written `ALL_CAPS`.
- **methods** are verbs or verb-like names, lowercase-first.
- **classes** are nouns, uppercase-first.

Names _can_ be long. This is no problem with modern IDEs and tools like AI coding assistants.
Of course, it is still a good idea to have a loop counter named `i` because this is convention. Naming it
Here are a few bad examples each followed by a better alternative (without access modifiers): 

```java
//VARIABLES
int qs; //BAD - abbreviation
int quality_score; //BAD - snake_case
int QualityScore; //BAD - first character uppercase

int qualityScore; // GOOD

double dist; //BAD abbreviation
double distance; //BAD - no units

double distanceInMeters; //GOOD

//METHODS
double eucldistnc(){Point x, Point y} //BAD - abbreviation
double distance(Point x, Point y){} //BAD - not informative enough although the next would be OK
double distance(Point x, Point y, DistanceMetric metric){} // GOOD because the metric is a function parameter
double euclideanDistance(Point x, Point y){} //BAD - noun (although this usage is not that bad)
double getEuclideanDistance(Point x, Point y){} //GOOD - noun (although this usage is not that bad)

void process(){} //BAD not informative
void processGffElement(){} //GOOD

//CLASSES
FileParse{} //BAD - not a noun, not informative
Parser{} //BAD - not informative
gffFileParser{} //BAD not uppercase-first
GffFileParser{} //GOOD
```


Here is a special case, a variable holding a complex data structure:

```java
//HashMap that stores employees by ID and employee properties as HashMap as well
//BAD - not coded to interface (Map) and unintelligible
HashMap<String, HashMap<String, String>> employees;

//still BAD - unintelligible
Map<String, Map<String, String>> employees; 

//We have records now!
record Employee(String id, String name, String role){}

//MUCH better
Map<String, Employee> employees;

//maybe even better?
class EmployeeCollection{
    Map<String, Employee> employees;
    //class code to manage employees
}
```


### Avoid complex data structures

This principle has been shown in the code snippet above.
Whenever you have a data structure of two nested collections or more, you should consider refactoring this into a data class, preferably a record. 

:::{tip}
Refactor complex data structures into records.
:::

The section below also has two examples of the use of records. This is not so much to wrap complex data structures, but more to wrap multiple return values from a method.


## Methods

### General rules

- Methods have very descriptive names (verb), in `camelCase()` with lowercase-first.
- They do one thing only (SRP!)
- They preferably define no more than 2 or 3 parameters. This keeps testing at an acceptable level of difficulty. If you have more than three arguments, you can wrap them into a configuration object.

:::{important}
**keep methods testable**
:::

:::{tip}
Wrap multiple return values into a single record.
:::

below is an example of parameter wrapping.

```java
abstract Circle createCircle(double x, double y, double radius);

//better:
abstract Circle createCircle(Point center, double radius);
```

### Keep methods small

- Keep methods short and simple: not too deep nesting of flow control blocks (maximum three) and not too many method parameters. 
Prefer a small data class (record) as return value over complex data structures.
- Keep a maximum of two indentation levels.

Here follows a refactoring example to demonstrate the advantage of using small -even tiny- methods.  

Suppose we have this class, which is a reporting tool on FastQ sequence quality.

```java
package nl.bioinf.refactoring.fastqc;

import java.io.BufferedReader;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.io.File;

public class FastQC {

    public static void main(String[] args) {
        if (args.length != 1) {
            System.out.println("No file provided! \nUsage: java -jar fastqc.jar <fastq file>");
            return;
        }
        File fastqFile = Paths.get(args[0]).toFile();
        if(!fastqFile.exists() || !fastqFile.isFile() || !fastqFile.canRead()) {
            System.out.println("File " + fastqFile + " does not exist");
            return;
        }
        try(BufferedReader reader = Files.newBufferedReader(fastqFile.toPath())) {
            String line;
            String headerLine = "";
            long sequenceCount = 0;
            long failedSequenceCount = 0;
            boolean inQualityLine = false;
            while((line = reader.readLine()) != null) {
                // header lines start with @
                if (line.startsWith("@")) {
                    //System.out.println("Header line: " + line);
                    headerLine = line;
                    sequenceCount++;
                    continue;
                }
                // quality lines start with +
                if (line.startsWith("+")) {
                    //System.out.println("Next is quality line: " + line);
                    inQualityLine = true;
                    continue;
                }

                if (inQualityLine) {
                    //System.out.println("Quality line: " + line);

                    //read the quality line and determine the average quality score
                    //and number of low-quality bases
                    //if the average quality score is below 30, or the number of low-quality
                    //bases higher than 10, mark as failed
                    //score = ascii - 33
                    int lowCount = 0;
                    int summedScore = 0;
                    for (int i = 0; i < line.length(); i++) {
                        char qualityChar = line.charAt(i);
                        int qualityScore = qualityChar - 33;
                        summedScore += qualityScore;
                        if (qualityScore < 20) {
                            lowCount++;
                        }
                    }
                    double averageQuality = (double) summedScore / line.length();
                    //System.out.println("Average Quality: " + +averageQuality + ", low count: " + lowCount);
                    if (averageQuality < 30 || lowCount > 10) {
                        failedSequenceCount++;
                        //System.out.println("Warning: " + headerLine + " has low quality");
                    }
                    inQualityLine = false;
                }
            }
            //report results
            double fraction = (double) failedSequenceCount / sequenceCount * 100;
            System.out.println("Done reading file " + fastqFile);
            System.out.print("Number of failed sequences: " + failedSequenceCount
                    + " out of " + sequenceCount + " sequences ( " +
                    fraction + "%)");
        } catch (Exception e) {
            System.out.println("Error while reading file " + fastqFile);
            e.printStackTrace();
        }
    }
}
```

It has several major flaws:  
- everything is in static `main()`, effectively making this a script instead of reusable code
- its main() has a maximum of no less then 6 indentation levels
- it is not testable
- it is weakly readable code and therefore needs lots of comments
- there are many things done in the main() method that could be extracted and reused.
- because it has no tests, it needs a lot of print statements during development.
- (there are probably more)

Let's refactor.  

You can start either from the highest indentation level and work my way up, or the other way around.  
I choose the first, because this makes it possible to create JUnit tests along with it.  
Here is a snippet with a conditional that can be extracted into a tiny but highly testable method:

```java
if (averageQuality <= 30 || lowCount >= 10) {
    failedSequenceCount++;
    //System.out.println("Warning: " + headerLine + " has low quality");
}
```

While we're at it, let's also take out these within-code quality definitions (the numbers 30 and 10)!  
First define some static class variables:

```java
private static final int AVERAGE_QUALITY_CUTOFF = 30;
private static final int LOW_QUALITY_BASES_CUTOFF = 10;
```

and next use these constants in a tiny helper method:

```java
private static boolean isFailedSequence(double averageQuality, int lowCount) {
    return averageQuality <= AVERAGE_QUALITY_CUTOFF || lowCount >= LOW_QUALITY_BASES_CUTOFF;
}
```

Only for this example I include a test method (omitted for the subsequent refactored methods):

```java
package nl.bioinf.refactoring.fastqc;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class FastQCTest {
    @Test
    void isFailedSequence_true() {
        assertTrue(FastQC.isFailedSequence(20, 0));
        assertTrue(FastQC.isFailedSequence(29, 0));
        assertTrue(FastQC.isFailedSequence(35, 11));
        assertTrue(FastQC.isFailedSequence(30, 10));
    }

    @Test
    void isFailedSequence_false() {
        assertFalse(FastQC.isFailedSequence(31, 0));
        assertFalse(FastQC.isFailedSequence(40, 9));
    }
}
```

Finally, here is its usage:

```java
if (isFailedSequence(averageQuality, lowCount)) {
    failedSequenceCount++;
}
```

Why is this useful?

- the conditional is much more readable (without code comments)
- when quality criteria change (or are extracted into class parameters) it is easy to locate their usage
- the test suite guarantees that subsequent refactoring operations don't break this existing code base.


Another tiny, but useful extraction is this (again also using a constant, not shown):

```java
static int getQualityScore(char qualityChar) {
    return qualityChar - ILLUMINA_ENCODING;
}
```

And this one, with similar purpose. 

```java
static boolean isLowQualityBase(char qualityChar) {
    return qualityChar < LOW_QUALITY_SCORE;
}
```

Note the use of naming convention: the `is....()` prefix for methods that return a boolean and `get....()` for methods returning some value.  

Her is another type of refactoring: a small 'statistics' record as inner class:

```java
private record SequenceQuality(int lowCount, int summedScore) { }
```

Java records are extremely useful for passing (mixed-type) data structures. They hardly need any code to define them and give lots of functionality out of the box.  
Here is its use in a dedicated method determining these statistics:

```java
public static SequenceQuality getSequenceQuality(String line) {
    int lowCount = 0;
    int summedScore = 0;
    for (int i = 0; i < line.length(); i++) {
        char qualityChar = line.charAt(i);
        int qualityScore = getQualityScore(qualityChar);
        summedScore += qualityScore;
        if (isLowQualityBase(qualityChar)) {
            lowCount++;
        }
    }
    //return the record instance
    return new SequenceQuality(lowCount, summedScore);
}
```

Similarly, we can wrap the FastQ file statistics in this record:

```java
public record FastQfileQuality(long sequenceCount, long failedSequenceCount) {
    public double fractionFailed() {
        return (double) failedSequenceCount / sequenceCount * 100;
    }
}
```

It has a single additional method publishing a new property: `fractionFailed()`.  

But now it is high time to get out of the `main()` and static environment and get rid of all these `static` keywords.
Here is the end result:

```java
package nl.bioinf.refactoring.fastqc;

import java.io.BufferedReader;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.io.File;

public class FastQC {
    //static class variables
    private static final int AVERAGE_QUALITY_CUTOFF = 30;
    private static final int LOW_QUALITY_BASES_CUTOFF = 10;
    private static final int ILLUMINA_ENCODING = 33;
    private static final int LOW_QUALITY_SCORE = 20;

    public static void main(String[] args) {
        if (args.length != 1) {
            System.out.println("No file provided! \nUsage: java -jar fastqc.jar <fastq file>");
            return;
        }
        File fastqFile = Paths.get(args[0]).toFile();
        if (! fileIsOk(fastqFile)) {
            System.out.println("File " + fastqFile + " does not exis or is not readable");
            return;
        }

        FastQC fastQC = new FastQC();
        FastQfileQuality qualityReport = fastQC.readFile(fastqFile);
        System.out.println("Done reading file " + fastqFile);
        System.out.print("Number of failed sequences: " + qualityReport.failedSequenceCount()
                + " out of " + qualityReport.sequenceCount + " sequences (" +
                (qualityReport.fractionFailed()*100) + "%)");
    }

    private record SequenceQuality(int lowCount, int summedScore) { }
    public record FastQfileQuality(long sequenceCount, long failedSequenceCount) {
        public double fractionFailed() {
            return (double) failedSequenceCount / sequenceCount * 100;
        }
    }

    private  boolean fileIsOk(File fastqFile) {
        if(!fastqFile.exists() || !fastqFile.isFile() || !fastqFile.canRead()) {
            System.out.println("File " + fastqFile + " does not exist");
            return false;
        }
        return true;
    }

    public FastQfileQuality readFile(File fastqFile) {
        try(BufferedReader reader = Files.newBufferedReader(fastqFile.toPath())) {
            String line;
            long sequenceCount = 0;
            long failedSequenceCount = 0;
            boolean inQualityLine = false;
            while((line = reader.readLine()) != null) {
                if (isHeaderLine(line)) {
                    sequenceCount++;
                    continue;
                }
                if (isQualityLine(line)) {
                    inQualityLine = true;
                    continue;
                }

                if (inQualityLine) {
                    SequenceQuality sq = getSequenceQuality(line);
                    double averageQuality = (double) sq.summedScore() / line.length();
                    if (isFailedSequence(averageQuality, sq.lowCount())) {
                        failedSequenceCount++;
                    }
                    inQualityLine = false;
                }
            }
            return new FastQfileQuality(sequenceCount, failedSequenceCount);
        } catch (IOException e) {
            throw new RuntimeException("Error while reading file " + fastqFile, e);
        }
    }

    public SequenceQuality getSequenceQuality(String line) {
        int lowCount = 0;
        int summedScore = 0;
        for (int i = 0; i < line.length(); i++) {
            char qualityChar = line.charAt(i);
            int qualityScore = getQualityScore(qualityChar);
            summedScore += qualityScore;
            if (isLowQualityBase(qualityChar)) {
                lowCount++;
            }
        }
        return new SequenceQuality(lowCount, summedScore);
    }

    boolean isQualityLine(String line) {
        return line.startsWith("+");
    }

    boolean isHeaderLine(String line) {
        return line.startsWith("@");
    }

    boolean isLowQualityBase(char qualityChar) {
        return qualityChar < LOW_QUALITY_SCORE;
    }

    int getQualityScore(char qualityChar) {
        return qualityChar - ILLUMINA_ENCODING;
    }

    boolean isFailedSequence(double averageQuality, int lowCount) {
        return averageQuality <= AVERAGE_QUALITY_CUTOFF || lowCount >= LOW_QUALITY_BASES_CUTOFF;
    }
}
```
The longest method is still 30 lines long (`readFile()`) - the upper boundary for methods.
Note the use of the different kinds of access modifiers.

:::{admonition} Access modifiers

Be as restrictive as possible with access modifiers.  
Be particularly careful with `public` because these constitute your library public API, making it very hard to change later.  
Methods that need JUnit tests preferably have the 'default' access level (private methods can not be tested). 
:::


## Classes

- Classes should be small
- Classes should adhere to the Single Responsibility Principle (SRP)
- Classes should be cohesive
- Classes should encapsulate their state

:::{admonition} Classes should protect their state
:class: warning

Never deal out your (mutable) collection to the outside.  
:::

Instead of this, 

```java
private List<Integer> elements = new LinkedList<Integer>();
public List<Integer> getStackAsList() {
    return elements;
}
```

which creates a risk of data corruption, makes your design easier to break and harder to change (rigid & fragile), you should do this:

```java
private List<Integer> elements = new LinkedList<Integer>();
public List<Integer> getStackAsList() {
    return Collections.unmodifiableList(elements);;
}
```

### Classes should be cohesive 

:::{note}
Cohesion refers to the degree to which the elements inside a module belong together.
In a cohesive class, methods do only things related to each other and to the class' purpose,
and also act on the same instance variables.
:::

The class below is highly cohesive because two out of three methods deal with both instance variables and one method with one instance variable. Also, the methods are all serving a single purpose.

```java
public class Stack {
    private int topOfStack = 0;
    private List<Integer> elements = new LinkedList<Integer>();

    public List<Integer> getStackAsList() {
        return Collections.unmodifiableList(elements);
    }

    public int size() {
        return topOfStack;
    }

    public void push(int element) {
        topOfStack++;
        elements.add(element);
    }

    public int pop() throws EmptyStackException {
        if (topOfStack == 0) throw new EmptyStackException();
        int element = elements.get(--topOfStack);
        elements.remove(topOfStack);
        return element;
    }
}
```


### Place variables in a clear context

Names are usually more meaningfull when placed in a context.  
With the class below, you need to study carefully what the variables represent; for instance, is the variable `numberOfLogins` related to the `datasource` or something else? And what about `port`?

```java
public class UserManagement {
    private long id;
    private String name;
    private String street;
    private int number;
    private String numberPostfix;
    private String zipCode;
    private int numberOfLogins;
    private String dataSourceUrl;
    private int port;
    
    //much code
```

The code below represents the same data, now ordered in logical entities - context.

```java
public class UserManagement {
    private User user;
    private Address address;
    private DataSource dataSource;
    //much code
}

public class User {
    private long id;
    private String name;
    private int numberOfLogins;
}

public class Address {
    private String street;
    private int number;
    private String numberPostfix;
    private String zipCode;
}

public class DataSource {
    private String dataSourceUrl;
    private int port;
}
```

## Another refactoring example

Here is another worked example emphasising te principles above.

What does this code do?

```java
    private List<int[]> theList;

    public List<int[]> getThem() {
        List<int[]> list1 = new ArrayList<>();
        for (int[] x : theList) {
            if (x[0] == 4) {
                list1.add(x);
            }
        }
        return list1;
    }
```

The javadoc gives some information:

```java
/**
 * This is a Minesweeper game, where the board is
 * represented by a List of cells. The cells are
 * represented by int arrays, where index 0 holds
 * the cell status: 4 means “flagged”
 */
```

Renaming variables already improves readability a lot:

```java
    public List<int[]> getFlaggedCells() {
        List<int[]> flaggedCells = new ArrayList<>();
        for (int[] x : gameBoard) {
            if (x[0] == 4) {
                flaggedCells.add(x);
            }
        }
        return flaggedCells;
    }
```

Extracting constant provides a next step.

```java
    private List<int[]> gameBoard;
    private static final int STATUS_VALUE_INDEX = 0;
    private static final int STATUS_FLAGGED = 4;

    public List<int[]> getFlaggedCells() {
        List<int[]> flaggedCells = new ArrayList<>();
        for (int[] x : gameBoard) {
            if (x[STATUS_VALUE_INDEX] == STATUS_FLAGGED) {
                flaggedCells.add(x);
            }
        }
        return flaggedCells;
    }
```

But extracting classes gives the final boost.

```java
class Cell {
    static final int STATUS_FLAGGED = 4;
    private int status;

    public boolean isFlagged() {
        return status == STATUS_FLAGGED;
    }

    public Cell() {  }

    public Cell(int status) {
        this.status = status;
    }
}
```

Here is the final version (for now):

```java
    private List<Cell> gameBoard;
    private static final int STATUS_VALUE_INDEX = 0;

    public List<Cell> getFlaggedCells() {
        List<Cell> flaggedCells = new ArrayList<>();
        for (Cell cell : gameBoard) {
            if (cell.isFlagged()) {
                flaggedCells.add(cell);
            }
        }
        return flaggedCells;
    }
```


## Constructors

- Only specify constructor parameters for values that really need to be specify and for which no default is available
- Give reasonable defaults for constructor parameters whenever this is possible, and provide a means to pass custom values
- Don't specify more than three overloaded constructors
- If the constructor knows more than three parameters, use factory methods or the Builder Pattern (see Design Patterns).


### Prefer factory methods over complex constructors

With the Cell example above, these dedicated factory methods could have been used.  
It may have been opportune to even mark the constructors `private` to force usage of these factory methods:

```java
class Cell {
    static final int STATUS_FLAGGED = 4;
    private int status;

    public boolean isFlagged() {
        return status == STATUS_FLAGGED;
    }

    private Cell() {  }

    public static Cell createFlaggedCell() {
        Cell cell = new Cell();
        cell.status = STATUS_FLAGGED;
        return cell;
    }

    public static Cell fromStatusCode(int status) {
        Cell cell = new Cell();
        cell.status = status;
        return cell;
    }
}
```

## Refactoring techniques overview

Below is a (non-exhaustive) overview of possible problems and associated refactoring techniques. 

1. Use clear and descriptive names for all code elements: 
    * Nouns for variables and classes.
    * Verbs for methods.
1. Replace literal values with variables/constants/enums.
1. Extract deeply nested flow control structures into methods. Allow no more than 3 or 4 levels of indentation.
1. Split longer functions into several smaller ones.
1. Replace complex boolean expressions with a method call.
1. Replace complex data structures with data objects, preferably records.
1. Split complex classes into several simple ones.
1. Use factory methods or builder pattern for complex classes.
1. Catch `NullPointerException` as soon as possible.
1. Never spill your guts: catch exceptions and convert them to informative error messages to the user.
1. Use well-designed error flow control with the exception mechanism.
1. Have only one entry- and exit point for the application - prevent the use of `System.exit()` in production code.
1. Use `stdout` for program output and `stderr` for error messages.

And finally:

:::{admonition} Repeated code is a code-smell!  
:class: warning  

Repeated code (often with only minor variations) can be very diverse: 
* simple repeated operations - refactor to method call
* a repeated code block creating complex objects - refactor with Builder or Factory Method Pattern
* repeated if/else or switch blocks - refactor with Inheritance, Strategy Pattern or Template Method Pattern, or other
* Biolerplate class code - refactor with records

This is not a complete listing.
:::