# The Streams API

The Java stream API is constituted by a wonderful set of classes and methods that together make it possible to process, mutate, filter and collect (reduce) data(-objects) in a streaming fashion. By making use of parallel streams this is even possible in a multi-core manner. 

Streams in general have these three components:

1. **Stream creation**: One of several ways to instantiate a stream. This can be out of a normal collection such as a list, or from a file, or on demand.
2. **Stream processing**: filtering and mapping operations performed on each element (=object) in the stream. The stream will not end, but generate a new stream with the new or filtered objects.
3. **Stream termination**: All streams must end. This can be through writing each object to a new file, but usually you will collect 
the results in a new (reduced) form.

Together these operations are sometimes together named **map-filter-reduce**.

Here is a small example demonstrating all three components. 

```java
List<Integer> numbers = List.of(2,3,4,5,6,7);

int sum = numbers
        .stream()
        .map(n -> n * n)          // map number to squared number
        .filter(n -> n % 2 == 0)  // filter: only even numbers pass
        .reduce((x, y) -> x + y)  // reduce: sum them up
        .get();                    // extract the value from the optional
System.out.println("sum = " + sum);
```

gives 

<pre class="console_out">
sum = 56
</pre>

As you can see, it is possible to build a whole workflow in just a single statement. 

## Optionals
Many terminal operations produce `Optional`. It is like a container that may or may not hold a value. The purpose of the `Optional` class is to provide a "type-level solution" for representing optional values instead of null references.

Here are some common operations:

```java
String name = "Mike";
Optional<String> opt = Optional.of(name);
/* Alternative: opt = Optional.ofNullable(name);*/
System.out.println(opt.isPresent());
System.out.println(opt.get());
opt.ifPresent(n -> System.out.println(n));
System.out.println(opt.orElseGet(/*a Supplier for when no value is present*/() -> "John Doe"));

opt = Optional.empty();
System.out.println(opt.orElseGet(() -> "John Doe"));
opt.ifPresentOrElse(
        /*a Consumer: what to do if present*/ n -> System.out.println(n),
        /*a Runnable: what to do if NOT present*/ () -> System.out.println("John Doe"));
```
<pre class="console_out">
true
Mike
Mike
Mike
John Doe
John Doe
</pre>

Have a look at [Baeldung](https://www.baeldung.com/java-optional) for a more in-depth discussion.


## Stream creation

A stream needs a data source. 
Three means to create streams are discussed in this section: from a factory, from collections, from files, and on demand.

### Factory

The simplest way is to use the Stream factory method `Stream.of()`:

```java
Stream.of(2,3,4,5,6,7)
        .map(n -> Math.sqrt(n))
        .forEach(x -> System.out.println(x));
```

However, besides example context, I do not use this method very often. 
Simply because data usually comes from files or collections.


### From collections

This section gives examples with Lists and Arrays, but most collection support the generation of Streams from their elements.Hav a look at the docs for details.

In the above example you have already seen how to stream elements of a List. Sets, Maps and other collections all have stream initialization functions. Maps have three ways to stream their contents: the keys, the values and the entry pairs:

```java
Map<String, Integer> map = new HashMap<>();
map.put("a", 1);
map.put("b", 2);
map.put("c", 3);

map.keySet().stream().forEach(n -> System.out.println(n));
map.values().stream().forEach(n -> System.out.println(n));
map.entrySet().stream().forEach(n -> System.out.println(n));
```

Arrays have a diverse way of streaming numbers for common use cases. This most generic is this one:

```java
int[] numbers = {3, 4, 5, 6, 7};
IntStream stream = Arrays.stream(numbers);
```

but what if you are interested in some descriptive statistics?

```java
double avg = stream.average()
        .getAsDouble();
System.out.println("avg = " + avg);
```


### From files

Reading a file may throw an Exception, so this constitutes a little but more work, but the `lines()` function of the `Files` class does the job:

```java
String fileName = "your/file/path";
try (Stream<String> stream = Files.lines(Paths.get(fileName))) {
        stream
                .map(l -> l.replace(',', ';'))
                .limit(4)
                .forEach(n -> System.out.println(n));
} catch (IOException e) {
        e.printStackTrace();
}
```

### On demand

To generate values on demand (i.e. not pre-calculated) you need the `Streams.generate()` method and pass it an implementer of the `Supplier` functional interface.
Here is an example generating random numbers between 50 and 100.


```java
Stream numbers = Stream.generate(new Supplier() {
        Random random = new Random();
        int max = 100;
        int min = 50;

        @Override
        public Integer get() {
                return random.nextInt(max - min) + min;
        }
});
numbers
        .limit(10)
        .forEach(n -> System.out.println(n));
```

There are more dedicated methods for generating number ranges, using the `IntStream` class.

```java
IntStream.range(0, 6).forEach(n -> System.out.println(n));
```

Note that the range is end-exclusive, so in this example the number 6 will not be generated.


### Parallel streams

All the above have parallel equivalents in the form 

```java
List<Integer> numbers = List.of(2,3,4,5,6,7);
numbers.parallelStream();
```

thus extending your workflow to multiple cores (managed by the JVM). This, however, makes it impossible to rely on order of
processing of constituent elements. For instance, this snippet

```java
List<Integer> numbers = List.of(2,3,4,5,6,7);
numbers
        .parallelStream()
        .forEach(x-> System.out.println(x));
```

gives (on my computer):

<pre class="console_out">
5
7
2
4
3
6
</pre>

## Streaming characters from a String

In bioinformatics, working with the individual characters of a DNA, RNA or protein sequence is a very common task. However, if you stream them in the obvious way, you will get numbers:

```java
String s = "GACT";
s.chars().forEach(n -> System.out.println(n));
```

<pre class="console_out">
71
65
67
84
</pre>

This is because `s.chars()` generates in `IntStream`. To get actual characters, you need to cast them.

```java
String s = "GACT";
s.chars()
        .mapToObj(n -> (char)n)
        .forEach(n -> System.out.println(n));
```

Note that this uses `mapToObj()` instead of `map()`. This is because an `IntStream` is a stream of primitive values.



## Intermediate stream operations

Intermediate operations are operations that produce a new Stream of values. These values can be of the same type (with `filter()` or `peek()`), but they can also be of another type, as with `map()` and `mapToObj()`.

First we'll look at streaming characters because this makes it possible to show some examples with biological sequences.

### Intermediate operations: `peek()`

Function `peek()` can be used to inspect or store intermediate results.

```java
        String s = "GACT";
        s.chars()
                .peek(n -> System.out.print(n + " "))
                .mapToObj(n -> (char)n)
                .forEach(n -> System.out.print(n + " "));
```

<pre class="console_out">
71 G 65 A 67 C 84 T
</pre>

This output also shows us that processing is really sequential; each element traverses the whole pipeline, one after the other, and not batch-wise. In the case of batch-wise processing, we would have seen this:

```
71 65 67 84 G A C T
```


### Intermediate operations: `map()`

Take for example this amino acid sequence: `MPFRST`.
Suppose you want to convert this into three-letter codes. This is a way to do that:

```java
Map<Character, String> aminoacids = Map.of(
        'M', "Met",
        'P', "Pro",
        'F', "Phe",
        'R', "Arg",
        'S', "Ser",
        'T', "Thr"); // other 18 omitted
String s = "MPFRST";
s.chars()
        .mapToObj(n -> (char)n)
        .map(aa -> aminoacids.get(aa))
        .forEach(n -> System.out.print(n));
```

<pre class="console_out">
MetProPheArgSerThr
</pre>


### Intermediate operations: `filter()`

The `filter()` function uses a Predicate to assess whether an element will be passed on or not.
A predicate simply returns a boolean value, given an element as input.

Here are a few.

```java
Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9)
        .filter(n -> n % 2 == 0)    //even
        .filter(n -> n > 5)         //greater than 5
        .forEach(n -> System.out.println(n));
```

Of course, these could also have been combined into one predicate:

```java
Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9)
        .filter(n -> (n % 2 == 0 && n > 5))    //even and greater than 5
        .forEach(n -> System.out.println(n));
```

### Intermediate operations: `distinct()`

This works as you would expect (first occurrence is kept):

```java
String s = "GAACGTCCTA";
s.chars()
        .distinct()
        .mapToObj(n -> (char)n)
        .forEach(n -> System.out.print(n + " "));
```
<pre class="console_out">
G A C T 
</pre>

Distinct is essentially a `filter()` that only passes the first occurrence of each value in the stream.

### Intermediate operations: There are more!

Have a look at the docs for a more complete overview. 
These include the dedicated methods to generate primitives or convert primitives to objects.


## Method references as functional interfaces

So far, I have skipped a topic that your IDE may have suggested for you. For instance, when I use this construct 

```java
n -> System.out.print(n + " ")
```

in IntelliJ Idea, it suggests "Lambda can be replaced with method reference ". When I agree, it replaces the above lambda with this:

```java
System.out::println
```

This expression is an example of a **Method reference**. It usually has the structure `<class or object>::<function name>`.

The function can be static or instance-bound.

Have a look at this class:

```java
class Wrapper{
    static String wrap1(String s){
        return "|" + s + "|";
    }

    String wrap2(String s){
        return "$" + s + "$";
    }

    static String wrap3(String s, char wrapper){
        return wrapper + s + wrapper;
    }
}
```

The `wrap2()` method is non-static, the other two are static. The `wrap3()` method takes two arguments, the first two only take a single argument.
Since in a stream operation only single values are passed (usually; `reduce()` is an exception), method `wrap3()` can NOT be used in such a context.
Here are the two allowed method reference usages and the allowed normal usage for the third:

```java
Stream.of("a", "b", "c")
        .map(Wrapper::wrap1)
        .map(new Wrapper()::wrap2)
        //.map(Wrapper()::wrap3)           // does not compile! Needs an extra argument
        .map(s -> Wrapper.wrap3(s, '*'))   // this works
        .forEach(System.out::println);
```

<pre class="console_out">
*$|a|$*
*$|b|$*
*$|c|$*
</pre>


:::{admonition} Method references
:class: note
In general, any method that can be treated as a functional interface in its stream context, can be referenced using the double colon construct `<class|object>::<method>`.
:::

## Terminal Stream operations

All streams must end. Without an end, there is no use creating them, is there?

These terminations can be categorized into

1. One operation per value
2. Collecting values into a collection (List, Map, Set) 
3. Grouping values (a special case of collecting)
4. Reducing to a single value

### One operation per value

We have seen many examples of this already: the `foreach()` function.
This operation does not return anything. It simply ends the journey of that value.

### Collecting (grouped) values

We'll keep things simple here. The most common use cases are collecting values into a List, Set or Map. Of these, the Map is the most versatile (and difficult to master).

Below are two examples for lists and sets.


```java
System.out.println(Stream.of("a", "b", "c", "a", "c", "d", "a")
        .map(s -> s.toUpperCase())
        .toList()); //shortcut for .collect(Collectors.toList())

System.out.println(Stream.of("a", "b", "c", "a", "c", "d", "a")
        .map(s -> s.toUpperCase())
        .collect(Collectors.toSet()));
```

Now for maps. A map needs keys and values.
Only some basic use cases will be demonstrated here. Let's start with the simplest: grouping by some property into a Map of Lists. Given this simple record class:

```java
record Employee (String name, String role) {
    @Override
    public String toString(){
        return name + "(" + role + ")";
    }
}
```

We could stream and group these objects like this:

```java
List<Employee> employees = List.of(
        new Employee("John", "Developer"),
        new Employee("Jane", "Developer"),
        new Employee("Jack", "Tester"),
        new Employee("Jill", "Tester"),
        new Employee("Jack", "Manager")
);

Map<String, List<Employee>> grouped = employees
    .stream()
    .collect(groupingBy(Employee::role));
System.out.println(grouped);
```
<pre class="console_out">
{Tester=[Jack(Tester), Jill(Tester)], Developer=[John(Developer), Jane(Developer)], Manager=[Jack(Manager)]}
</pre>


A slightly more challenging task is counting occurrences.

```java
System.out.println(Stream.of("a", "b", "c", "a", "c", "d", "a")
        .map(s -> s.toUpperCase())
        .collect(Collectors.groupingBy(Function.identity(), Collectors.counting())));
```
<pre class="console_out">
{A=3, B=1, C=2, D=1}
</pre>

The `groupingBy()` function needs a function that will generate the key and a function that does some downstream processing step (like counting). 

Here is another example, grouping by string length and collecting into lists:

```java
System.out.println(Stream.of("abc", "ca", "cd", "aaa", "bab")
        .collect(Collectors.groupingBy(String::length, Collectors.toList())));
        //x -> x.length() works as well instead of String::length
```
<pre class="console_out">
{2=[ca, cd], 3=[abc, aaa, bab]}
</pre>

### Reductions

The general form of reductions is that values are sequentially processed into a single result.
Here is an example.

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5);
Integer prod = numbers.stream().reduce(1, (a, b) -> a * b);
System.out.println("cumulative prod=" + prod);
```
<pre class="console_out">
cumulative prod=120
</pre>

The `reduce()` function is seeded with a start value, and all values are (in this case) multiplied with the result of the previous reduction. So, in the above example, the seed value 1 i multiplied by 2 (giving 2) and this is multiplied by 3 (giving 6) and the next values are 24 and 120 which is the final outcome.

When working with non-number values, you don't have the seed value:

```java
Stream.of("a", "b", "c", "d", "e")
        .reduce((a, b) -> a + '-' + b)
        .ifPresent(System.out::println);
```
<pre class="console_out">
a-b-c-d-e
</pre>


Since concatenating strings is a thing that is often done, there is a shortcut:

```java
List<String> strings = List.of("a", "b", "c", "d", "e");
String concat = strings.stream().collect(Collectors.joining(","));
System.out.println(concat);
```

For numeric reductions the `IntStream` and other number streams have several methods at your disposal:  

- **`count()`**
- **`sum()`**
- **`max()`**
- **`min()`**
- **`distinct()`**

The names are self-explanatory.

