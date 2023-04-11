# Flow control

This is a straightforward listing of the Java flow control structures.

## if [else if] [else]

If/else is the basic decision-making mechanism in every programming language. 
In Java, both "else if" and "else" are optional.

<pre style="color:darkblue;font-weight:bold;font-family:courier;font-size:1.2em;">
if (<span style="color:darkred;font-style:italic;">condition</span>) {} 
<span style="color:darkgreen;">[</span>else if (<span style="color:darkred;font-style:italic;">condition</span>) {}<span style="color:darkgreen;">]*</span>
<span style="color:darkgreen;">[</span>else {}<span style="color:darkgreen;">]?</span>
</pre>

So, the "if" block is the required basis, "else if" can be used zero to many times - but only directly following the "if", and the entire statement is closed by an optional "else".

So this is not legal:

```java
if (foo) { }
else { }
else if (bar) { }
```

and neither is this, because the `int number = 2;` interrupts the if/else block:

```java
if (foo) { }
int number = 2;
else { }
```

## while

While has two variants:

<pre style="color:darkblue;font-weight:bold;font-family:courier;font-size:1.2em;">
while (<span style="color:darkred;font-style:italic;">condition</span>) { } 
do { } while (<span style="color:darkred;font-style:italic;">condition</span>);
</pre>

The difference is that `do{}while()` is guaranteed to be executed at least once, whereas `while(condition) {}` can be skipped entirely if "condition" is false at the first evaluation.

## for

Iterates over a collection or a defined series of steps:

<pre style="color:darkblue;font-weight:bold;font-family:courier;font-size:1.2em;">
for (<span style="color:darkred;font-style:italic;">init</span>; <span style="color:darkred;font-style:italic;">condition</span>; <span style="color:darkred;font-style:italic;">change</span>) { } 
</pre>

- "init" is the loop initialization; declaring a counter usually.
- "condition" is the test that determines whether the loop should run once more
- "change" is the iteration change; usually an increment of the init.

This is a typical for-loop:

```java
String[] nucleotides = {"Adenine", "Cytosine", "Guanine", "Thymine"};
for( int i = 0; i < nucleotides.length; i++) {
    System.out.println("nucleotide " + i + " is " + nucleotides[i]);
}
```

All three elements are optional. This, for instance, is a legal for-loop:

```java
String[] nucleotides = {"Adenine", "Cytosine", "Guanine", "Thymine"};
int i = nucleotides.length - 1;
for( ; i >= 0; --i) {
    System.out.println("nucleotide " + i + " is " + nucleotides[i]);
}
```

Note that the change does not need to be an increment.  
Even this one is legal:

```
for(;;) {
    System.out.println("Hello ");
}

//same as 
while(true) {
    System.out.println("Hello");
}
```

But it will run into eternity (or until `ctrl + c` has been typed). Actually, variations of the second form 
are quite often used to wait for user input in a terminal setting:

```java
 while(true) {
    int answer = getUserInput();
    if (answer == 42) break;
}
```

### foreach {-}

The for-loop also has a variant. It is called the **_foreach_** loop. The difference is that there is no increment or condition; it is simply used to iterate a collection (Arrays or Collection types dealt with later).

<pre style="color:darkblue;font-weight:bold;font-family:courier;font-size:1.2em;">
for (<span style="color:darkred;font-style:italic;">element</span> : <span style="color:darkred;font-style:italic;">collection</span>) { } 
</pre>

And this is a working example:

```java
String[] nucleotides = {"Guanine", "Adenine", "Cytosine", "Thymine"};
for( String nucleotide : nucleotides) {
    System.out.println("nucleotide = " + nucleotide);
}
```


## break and continue

All iteration structures (the while `while` variants and both `for` variants) have the possibility to leave the current iteration early, or the loop entirely.

- **`break`** - leave the loop (or the `switch` block)
- **`continue`** - abort current iteration and go to next iteration


## switch/case

The `switch/case` structure is a decision flow control structure that is always replaceable by if/else. However, 
`switch/case` is often more efficient, and better suited to deal with choosing between different discrete options.
Where `if/else` works with boolean conditions, `switch/case` works with discrete cases.

The formal description is

<pre style="color:darkblue;font-weight:bold;font-family:courier;font-size:1.2em;">
switch (<span style="color:darkred;font-style:italic;">actual case</span>) {
    <span style="color:darkgreen;">[</span>case <span style="color:darkred;font-style:italic;"> case n</span>: {<span style="color:darkred;font-style:italic;">case specific statement(s)</span>}<span style="color:darkgreen;">]*</span>
    <span style="color:darkgreen;">[</span>default: {<span style="color:darkred;font-style:italic;">default statement(s)</span>}<span style="color:darkgreen;">]?</span>
}
</pre>

So there are one to many possible case-specific actions and zero or one default actions. 

Here is an example method using a switch/case:

```java
void switchCase(String country) {
    switch (country) {
        case "Netherlands":
            System.out.println("Some weather, huh?");
            break;
        case "Belgium":
            System.out.println("It is Belgian fries!");
            break;
        default:
            System.out.println("Have a beer?");
    }
}
```

and when this is run like this

```java
FlowControlDemo demo = new FlowControlDemo();
demo.switchCase("Netherlands");
```

we get this output

<pre class="console_out">
Some weather, huh?
</pre>

and when called with an unknown country the `default` will be printed.

Note the `break` keyword in each case. The `break` is a very important aspect of the switch block. 
A switch block is **_fall-through_** unless a break is encountered.

So if the breaks are removed from the switch


```java
void switchCase(String country) {
    switch (country) {
        case "Netherlands":
            System.out.println("Some weather, huh?");
            //break;
        case "Belgium":
            System.out.println("It is Belgian fries!");
            //break;
        default:
            System.out.println("Have a beer?");
    }
}
```

we get this output

<pre class="console_out">
Some weather, huh?
It is Belgian fries!
Have a beer?
</pre>

Although this seems rather illogical, there are uses for this behavior. For example, consider this error
handler for a web application:

```java
String errorMessage(int errorCode) {
    String message;
    switch (errorCode) {
        case 401:
        case 402:
        case 403:
        case 405:
            message = "You are trying to do something that is not allowed";
            break;
        case 404:
        case 503:
            message = "Resource not found";
            break;
        default:
            message = "Some exotic error occurred. Try again later";
    }
    return message;
}
```

When called like this

```java
void errorMessageTest() {
    FlowControlDemo demo = new FlowControlDemo();
    String errorMessage = demo.errorMessage(402);
    System.out.println(errorMessage);
    errorMessage = demo.errorMessage(502);
    System.out.println(errorMessage);
}
```

We get these messages: 

<pre class="console_out">
You are trying to do something that is not allowed
Some exotic error occurred. Try again later
</pre>


